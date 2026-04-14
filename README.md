
import { useState, useEffect, useCallback, useRef } from "react";

const CATEGORIES = [
  { id: "all", label: "All" },
  { id: "politics", label: "Politics" },
  { id: "business", label: "Business" },
  { id: "sports", label: "Sports" },
  { id: "technology", label: "Tech" },
  { id: "world", label: "World" },
];

const SOURCES = {
  "Dawn": "#E63946",
  "Geo News": "#1D3557",
  "The News International": "#457B9D",
  "ARY News": "#2A9D8F",
  "Express Tribune": "#E9C46A",
  "Samaa TV": "#F4A261",
  "Dunya News": "#264653",
  "Pakistan Today": "#6A4C93",
};

// Shimmer loading card
const ShimmerCard = ({ delay = 0 }) => (
  <div
    style={{
      background: "var(--card-bg)",
      borderRadius: 14,
      padding: "28px 24px",
      animation: `fadeInUp 0.5s ${delay}s both`,
      overflow: "hidden",
      position: "relative",
    }}
  >
    <div style={{ display: "flex", gap: 10, marginBottom: 16 }}>
      <div style={{ width: 70, height: 18, borderRadius: 9, background: "var(--shimmer-bg)", animation: "shimmer 1.5s infinite" }} />
      <div style={{ width: 50, height: 18, borderRadius: 9, background: "var(--shimmer-bg)", animation: "shimmer 1.5s 0.2s infinite" }} />
    </div>
    <div style={{ width: "90%", height: 20, borderRadius: 6, background: "var(--shimmer-bg)", marginBottom: 10, animation: "shimmer 1.5s 0.1s infinite" }} />
    <div style={{ width: "70%", height: 20, borderRadius: 6, background: "var(--shimmer-bg)", marginBottom: 16, animation: "shimmer 1.5s 0.3s infinite" }} />
    <div style={{ width: "100%", height: 14, borderRadius: 4, background: "var(--shimmer-bg)", marginBottom: 6, animation: "shimmer 1.5s 0.2s infinite" }} />
    <div style={{ width: "85%", height: 14, borderRadius: 4, background: "var(--shimmer-bg)", animation: "shimmer 1.5s 0.4s infinite" }} />
  </div>
);

const LivePulse = () => (
  <span style={{ display: "inline-flex", alignItems: "center", gap: 6 }}>
    <span style={{
      width: 8, height: 8, borderRadius: "50%",
      background: "#E63946",
      animation: "pulse 1.5s infinite",
      boxShadow: "0 0 6px #E63946",
    }} />
    <span style={{
      fontFamily: "'JetBrains Mono', monospace",
      fontSize: 11,
      fontWeight: 600,
      letterSpacing: 2,
      textTransform: "uppercase",
      color: "#E63946",
    }}>Live</span>
  </span>
);

const formatTimeAgo = (dateStr) => {
  const now = new Date();
  const date = new Date(dateStr);
  const diffMs = now - date;
  const diffMins = Math.floor(diffMs / 60000);
  if (diffMins < 1) return "Just now";
  if (diffMins < 60) return `${diffMins}m ago`;
  const diffHrs = Math.floor(diffMins / 60);
  if (diffHrs < 24) return `${diffHrs}h ago`;
  const diffDays = Math.floor(diffHrs / 24);
  return `${diffDays}d ago`;
};

const categorizeArticle = (title, description) => {
  const text = `${title} ${description}`.toLowerCase();
  if (/cricket|psl|match|pcb|batting|bowling|icc|hockey|football|fifa|sport/.test(text)) return "sports";
  if (/election|parliament|pm |pti|pmln|ppp|senate|govt|government|minister|nawaz|imran|bilawal|political|court|judiciary/.test(text)) return "politics";
  if (/stock|market|rupee|economy|gdp|trade|export|import|sbp|inflation|fiscal|investment|bank/.test(text)) return "business";
  if (/tech|ai |startup|software|digital|cyber|app |internet|5g|mobile|smartphone/.test(text)) return "technology";
  if (/us |china|india|iran|saudi|russia|ukraine|biden|trump|un |nato|global|international/.test(text)) return "world";
  return "all";
};

export default function PakistanNewsLive() {
  const [articles, setArticles] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [activeCategory, setActiveCategory] = useState("all");
  const [lastUpdated, setLastUpdated] = useState(null);
  const [searchQuery, setSearchQuery] = useState("");
  const [expandedId, setExpandedId] = useState(null);
  const intervalRef = useRef(null);

  const fetchNews = useCallback(async () => {
    try {
      // Using multiple RSS-to-JSON proxies for Pakistani news sources
      const feeds = [
        { url: "https://www.dawn.com/feeds/home", source: "Dawn" },
        { url: "https://www.thenews.com.pk/rss/1/1", source: "The News International" },
        { url: "https://tribune.com.pk/feed", source: "Express Tribune" },
        { url: "https://dunyanews.tv/feed", source: "Dunya News" },
        { url: "https://www.pakistantoday.com.pk/feed/", source: "Pakistan Today" },
      ];

      const proxyBase = "https://api.rss2json.com/v1/api.json?rss_url=";

      const results = await Promise.allSettled(
        feeds.map(async (feed) => {
          const res = await fetch(`${proxyBase}${encodeURIComponent(feed.url)}`);
          if (!res.ok) throw new Error(`Failed: ${feed.source}`);
          const data = await res.json();
          if (data.status !== "ok" || !data.items) return [];
          return data.items.map((item, i) => ({
            id: `${feed.source}-${i}-${item.pubDate}`,
            title: item.title?.replace(/<[^>]*>/g, "").trim(),
            description: item.description?.replace(/<[^>]*>/g, "").substring(0, 280).trim(),
            source: feed.source,
            url: item.link,
            publishedAt: item.pubDate,
            thumbnail: item.thumbnail || item.enclosure?.link || null,
            category: categorizeArticle(item.title || "", item.description || ""),
          }));
        })
      );

      const allArticles = results
        .filter((r) => r.status === "fulfilled")
        .flatMap((r) => r.value)
        .filter((a) => a.title && a.title.length > 10)
        .sort((a, b) => new Date(b.publishedAt) - new Date(a.publishedAt));

      if (allArticles.length > 0) {
        setArticles(allArticles);
        setError(null);
      } else if (articles.length === 0) {
        // Fallback demo data if all feeds fail
        setArticles(getDemoArticles());
      }
      setLastUpdated(new Date());
      setLoading(false);
    } catch (err) {
      console.error(err);
      if (articles.length === 0) {
        setArticles(getDemoArticles());
      }
      setLoading(false);
      setLastUpdated(new Date());
    }
  }, []);

  useEffect(() => {
    fetchNews();
    intervalRef.current = setInterval(fetchNews, 120000); // refresh every 2 min
    return () => clearInterval(intervalRef.current);
  }, [fetchNews]);

  const filtered = articles.filter((a) => {
    const matchesCat = activeCategory === "all" || a.category === activeCategory;
    const matchesSearch = !searchQuery || 
      a.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
      a.source.toLowerCase().includes(searchQuery.toLowerCase());
    return matchesCat && matchesSearch;
  });

  const breakingNews = articles.slice(0, 1)[0];

  return (
    <div style={{
      "--bg": "#0A0C10",
      "--card-bg": "rgba(255,255,255,0.04)",
      "--card-hover": "rgba(255,255,255,0.07)",
      "--text": "#E8E6E1",
      "--text-muted": "#7A7B80",
      "--accent": "#E63946",
      "--accent-glow": "rgba(230,57,70,0.15)",
      "--border": "rgba(255,255,255,0.06)",
      "--shimmer-bg": "rgba(255,255,255,0.06)",
      minHeight: "100vh",
      background: "var(--bg)",
      color: "var(--text)",
      fontFamily: "'Newsreader', 'Georgia', serif",
      position: "relative",
      overflow: "hidden",
    }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Newsreader:ital,opsz,wght@0,6..72,300;0,6..72,400;0,6..72,600;0,6..72,700;1,6..72,400&family=JetBrains+Mono:wght@400;500;600&family=DM+Sans:wght@400;500;600;700&display=swap');

        * { box-sizing: border-box; margin: 0; padding: 0; }
        
        @keyframes fadeInUp {
          from { opacity: 0; transform: translateY(20px); }
          to { opacity: 1; transform: translateY(0); }
        }
        @keyframes pulse {
          0%, 100% { opacity: 1; transform: scale(1); }
          50% { opacity: 0.4; transform: scale(0.85); }
        }
        @keyframes shimmer {
          0% { opacity: 0.4; }
          50% { opacity: 0.8; }
          100% { opacity: 0.4; }
        }
        @keyframes slideIn {
          from { opacity: 0; transform: translateX(-10px); }
          to { opacity: 1; transform: translateX(0); }
        }
        @keyframes tickerScroll {
          0% { transform: translateX(0); }
          100% { transform: translateX(-50%); }
        }
        @keyframes gradientShift {
          0% { background-position: 0% 50%; }
          50% { background-position: 100% 50%; }
          100% { background-position: 0% 50%; }
        }

        .news-card {
          background: var(--card-bg);
          border-radius: 14px;
          padding: 28px 24px;
          cursor: pointer;
          transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
          border: 1px solid transparent;
          position: relative;
          overflow: hidden;
        }
        .news-card::before {
          content: '';
          position: absolute;
          top: 0; left: 0;
          width: 3px; height: 100%;
          border-radius: 3px 0 0 3px;
          opacity: 0;
          transition: opacity 0.3s;
        }
        .news-card:hover {
          background: var(--card-hover);
          border-color: var(--border);
          transform: translateY(-2px);
          box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        .news-card:hover::before { opacity: 1; }

        .cat-btn {
          padding: 8px 18px;
          border-radius: 100px;
          border: 1px solid var(--border);
          background: transparent;
          color: var(--text-muted);
          font-family: 'DM Sans', sans-serif;
          font-size: 13px;
          font-weight: 500;
          cursor: pointer;
          transition: all 0.25s;
          white-space: nowrap;
        }
        .cat-btn:hover { border-color: rgba(255,255,255,0.2); color: var(--text); }
        .cat-btn.active {
          background: var(--accent);
          border-color: var(--accent);
          color: #fff;
          box-shadow: 0 2px 12px var(--accent-glow);
        }

        .search-input {
          background: rgba(255,255,255,0.05);
          border: 1px solid var(--border);
          border-radius: 12px;
          padding: 12px 16px 12px 42px;
          color: var(--text);
          font-family: 'DM Sans', sans-serif;
          font-size: 14px;
          width: 100%;
          max-width: 320px;
          outline: none;
          transition: all 0.25s;
        }
        .search-input:focus {
          border-color: rgba(230,57,70,0.4);
          background: rgba(255,255,255,0.07);
          box-shadow: 0 0 0 3px rgba(230,57,70,0.1);
        }
        .search-input::placeholder { color: var(--text-muted); }

        .ticker-strip {
          background: linear-gradient(90deg, #E63946 0%, #c0392b 50%, #E63946 100%);
          background-size: 200% 100%;
          animation: gradientShift 8s ease infinite;
          padding: 10px 0;
          overflow: hidden;
          position: relative;
        }
        .ticker-content {
          display: flex;
          animation: tickerScroll 60s linear infinite;
          white-space: nowrap;
        }
        .ticker-item {
          font-family: 'DM Sans', sans-serif;
          font-size: 13px;
          font-weight: 600;
          color: white;
          padding: 0 40px;
          display: flex;
          align-items: center;
          gap: 12px;
        }
        .ticker-dot {
          width: 4px; height: 4px;
          background: rgba(255,255,255,0.6);
          border-radius: 50%;
        }

        .source-tag {
          display: inline-block;
          padding: 4px 10px;
          border-radius: 6px;
          font-family: 'DM Sans', sans-serif;
          font-size: 11px;
          font-weight: 600;
          letter-spacing: 0.5px;
          text-transform: uppercase;
        }

        .breaking-card {
          background: linear-gradient(135deg, rgba(230,57,70,0.12), rgba(230,57,70,0.03));
          border: 1px solid rgba(230,57,70,0.2);
          border-radius: 18px;
          padding: 32px;
          position: relative;
          overflow: hidden;
          cursor: pointer;
          transition: all 0.3s;
        }
        .breaking-card:hover {
          border-color: rgba(230,57,70,0.4);
          box-shadow: 0 12px 48px rgba(230,57,70,0.15);
        }
        .breaking-card::after {
          content: '';
          position: absolute;
          top: -50%; right: -20%;
          width: 300px; height: 300px;
          background: radial-gradient(circle, rgba(230,57,70,0.08), transparent 70%);
          pointer-events: none;
        }

        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.1); border-radius: 3px; }
        ::-webkit-scrollbar-thumb:hover { background: rgba(255,255,255,0.2); }
      `}</style>

      {/* Ambient background */}
      <div style={{
        position: "fixed", top: 0, left: 0, right: 0, bottom: 0,
        background: "radial-gradient(ellipse at 20% 0%, rgba(230,57,70,0.06) 0%, transparent 50%), radial-gradient(ellipse at 80% 100%, rgba(29,53,87,0.08) 0%, transparent 50%)",
        pointerEvents: "none", zIndex: 0,
      }} />

      <div style={{ position: "relative", zIndex: 1 }}>
        {/* Header */}
        <header style={{
          padding: "28px 32px 20px",
          display: "flex",
          flexWrap: "wrap",
          alignItems: "center",
          justifyContent: "space-between",
          gap: 16,
          borderBottom: "1px solid var(--border)",
        }}>
          <div style={{ display: "flex", alignItems: "baseline", gap: 16 }}>
            <h1 style={{
              fontFamily: "'Newsreader', serif",
              fontSize: 28,
              fontWeight: 700,
              letterSpacing: -0.5,
              color: "var(--text)",
              lineHeight: 1,
            }}>
              <span style={{ color: "var(--accent)" }}>PK</span> Wire
            </h1>
            <LivePulse />
          </div>

          <div style={{ display: "flex", alignItems: "center", gap: 16 }}>
            <div style={{ position: "relative" }}>
              <svg style={{ position: "absolute", left: 14, top: "50%", transform: "translateY(-50%)", opacity: 0.4 }} width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2">
                <circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/>
              </svg>
              <input
                className="search-input"
                placeholder="Search stories..."
                value={searchQuery}
                onChange={(e) => setSearchQuery(e.target.value)}
              />
            </div>
            {lastUpdated && (
              <span style={{
                fontFamily: "'JetBrains Mono', monospace",
                fontSize: 11,
                color: "var(--text-muted)",
                whiteSpace: "nowrap",
              }}>
                Updated {lastUpdated.toLocaleTimeString("en-GB", { hour: "2-digit", minute: "2-digit" })}
              </span>
            )}
          </div>
        </header>

        {/* Ticker */}
        {articles.length > 0 && (
          <div className="ticker-strip">
            <div className="ticker-content">
              {[...articles.slice(0, 10), ...articles.slice(0, 10)].map((a, i) => (
                <span key={i} className="ticker-item">
                  <span className="ticker-dot" />
                  {a.title?.substring(0, 80)}{a.title?.length > 80 ? "..." : ""}
                </span>
              ))}
            </div>
          </div>
        )}

        {/* Category Filters */}
        <div style={{
          padding: "18px 32px",
          display: "flex",
          gap: 8,
          overflowX: "auto",
          borderBottom: "1px solid var(--border)",
        }}>
          {CATEGORIES.map((cat) => (
            <button
              key={cat.id}
              className={`cat-btn ${activeCategory === cat.id ? "active" : ""}`}
              onClick={() => setActiveCategory(cat.id)}
            >
              {cat.label}
            </button>
          ))}
          <div style={{ marginLeft: "auto", display: "flex", alignItems: "center", gap: 6 }}>
            <span style={{
              fontFamily: "'JetBrains Mono', monospace",
              fontSize: 12,
              color: "var(--text-muted)",
            }}>
              {filtered.length} stories
            </span>
          </div>
        </div>

        {/* Main Content */}
        <main style={{ padding: "24px 32px 60px", maxWidth: 900, margin: "0 auto" }}>
          {loading ? (
            <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
              {[0, 1, 2, 3, 4].map((i) => <ShimmerCard key={i} delay={i * 0.1} />)}
            </div>
          ) : (
            <>
              {/* Breaking News Hero */}
              {breakingNews && activeCategory === "all" && !searchQuery && (
                <a
                  href={breakingNews.url}
                  target="_blank"
                  rel="noopener noreferrer"
                  style={{ textDecoration: "none", color: "inherit", display: "block", marginBottom: 28, animation: "fadeInUp 0.5s both" }}
                >
                  <div className="breaking-card">
                    <div style={{ display: "flex", alignItems: "center", gap: 10, marginBottom: 14 }}>
                      <span style={{
                        fontFamily: "'JetBrains Mono', monospace",
                        fontSize: 10, fontWeight: 700,
                        letterSpacing: 2, textTransform: "uppercase",
                        color: "var(--accent)",
                        padding: "3px 8px",
                        background: "rgba(230,57,70,0.15)",
                        borderRadius: 4,
                      }}>Breaking</span>
                      <span className="source-tag" style={{ background: SOURCES[breakingNews.source] || "#444", color: "#fff" }}>
                        {breakingNews.source}
                      </span>
                    </div>
                    <h2 style={{
                      fontFamily: "'Newsreader', serif",
                      fontSize: 26,
                      fontWeight: 700,
                      lineHeight: 1.3,
                      marginBottom: 12,
                      color: "var(--text)",
                    }}>{breakingNews.title}</h2>
                    {breakingNews.description && (
                      <p style={{
                        fontFamily: "'DM Sans', sans-serif",
                        fontSize: 15,
                        lineHeight: 1.6,
                        color: "var(--text-muted)",
                        maxWidth: 700,
                      }}>{breakingNews.description}</p>
                    )}
                    <span style={{
                      fontFamily: "'JetBrains Mono', monospace",
                      fontSize: 11,
                      color: "var(--text-muted)",
                      marginTop: 14,
                      display: "block",
                    }}>{formatTimeAgo(breakingNews.publishedAt)}</span>
                  </div>
                </a>
              )}

              {/* News Feed */}
              <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
                {filtered.slice(activeCategory === "all" && !searchQuery ? 1 : 0).map((article, i) => (
                  <a
                    key={article.id}
                    href={article.url}
                    target="_blank"
                    rel="noopener noreferrer"
                    style={{
                      textDecoration: "none",
                      color: "inherit",
                      animation: `fadeInUp 0.4s ${Math.min(i * 0.05, 0.5)}s both`,
                    }}
                  >
                    <div
                      className="news-card"
                      style={{ "--accent-line": SOURCES[article.source] || "#666" }}
                      onMouseEnter={(e) => e.currentTarget.querySelector(".card-line").style.opacity = 1}
                      onMouseLeave={(e) => e.currentTarget.querySelector(".card-line").style.opacity = 0}
                    >
                      <div
                        className="card-line"
                        style={{
                          position: "absolute", top: 0, left: 0,
                          width: 3, height: "100%",
                          background: SOURCES[article.source] || "#666",
                          borderRadius: "3px 0 0 3px",
                          opacity: 0, transition: "opacity 0.3s",
                        }}
                      />
                      <div style={{ display: "flex", alignItems: "center", gap: 10, marginBottom: 12 }}>
                        <span className="source-tag" style={{
                          background: `${SOURCES[article.source] || "#444"}22`,
                          color: SOURCES[article.source] || "#aaa",
                        }}>
                          {article.source}
                        </span>
                        <span style={{
                          fontFamily: "'JetBrains Mono', monospace",
                          fontSize: 11,
                          color: "var(--text-muted)",
                        }}>{formatTimeAgo(article.publishedAt)}</span>
                        {article.category !== "all" && (
                          <span style={{
                            fontFamily: "'DM Sans', sans-serif",
                            fontSize: 11,
                            color: "var(--text-muted)",
                            background: "rgba(255,255,255,0.04)",
                            padding: "2px 8px",
                            borderRadius: 4,
                            textTransform: "capitalize",
                          }}>{article.category}</span>
                        )}
                      </div>
                      <h3 style={{
                        fontFamily: "'Newsreader', serif",
                        fontSize: 19,
                        fontWeight: 600,
                        lineHeight: 1.35,
                        marginBottom: 8,
                        color: "var(--text)",
                      }}>{article.title}</h3>
                      {article.description && (
                        <p style={{
                          fontFamily: "'DM Sans', sans-serif",
                          fontSize: 14,
                          lineHeight: 1.55,
                          color: "var(--text-muted)",
                          display: "-webkit-box",
                          WebkitLineClamp: 2,
                          WebkitBoxOrient: "vertical",
                          overflow: "hidden",
                        }}>{article.description}</p>
                      )}
                    </div>
                  </a>
                ))}
              </div>

              {filtered.length === 0 && (
                <div style={{
                  textAlign: "center",
                  padding: "60px 20px",
                  animation: "fadeInUp 0.4s both",
                }}>
                  <div style={{ fontSize: 48, marginBottom: 16, opacity: 0.3 }}>📰</div>
                  <p style={{
                    fontFamily: "'DM Sans', sans-serif",
                    fontSize: 16,
                    color: "var(--text-muted)",
                  }}>No stories found for this filter</p>
                </div>
              )}
            </>
          )}
        </main>

        {/* Footer */}
        <footer style={{
          padding: "20px 32px",
          borderTop: "1px solid var(--border)",
          display: "flex",
          justifyContent: "space-between",
          alignItems: "center",
          fontFamily: "'DM Sans', sans-serif",
          fontSize: 12,
          color: "var(--text-muted)",
        }}>
          <span>Sources: Dawn, The News, Express Tribune, Dunya News, Pakistan Today</span>
          <span>Auto-refreshes every 2 minutes</span>
        </footer>
      </div>
    </div>
  );
}

function getDemoArticles() {
  const now = new Date();
  return [
    { id: "d1", title: "National Assembly passes federal budget amendments amid opposition walkout", description: "The lower house approved several amendments to the Finance Bill after heated debate, with opposition members staging a walkout over disputed clauses.", source: "Dawn", url: "#", publishedAt: new Date(now - 300000).toISOString(), category: "politics", thumbnail: null },
    { id: "d2", title: "Pakistan Stock Exchange surges past 85,000 milestone on strong institutional buying", description: "The KSE-100 index rallied sharply driven by blue-chip stocks in banking and energy sectors.", source: "The News International", url: "#", publishedAt: new Date(now - 900000).toISOString(), category: "business", thumbnail: null },
    { id: "d3", title: "PCB announces squad for upcoming bilateral series against South Africa", description: "The Pakistan Cricket Board revealed a 16-member squad including three uncapped players for the upcoming tour.", source: "Express Tribune", url: "#", publishedAt: new Date(now - 1800000).toISOString(), category: "sports", thumbnail: null },
    { id: "d4", title: "PM outlines new digital transformation initiative for government services", description: "The Prime Minister announced a comprehensive plan to digitise public services across all provinces within the next two years.", source: "Dunya News", url: "#", publishedAt: new Date(now - 3600000).toISOString(), category: "technology", thumbnail: null },
    { id: "d5", title: "Pakistan and China agree to expand CPEC corridor with new infrastructure projects", description: "Both nations signed memoranda of understanding covering rail, port, and renewable energy projects valued at several billion dollars.", source: "Dawn", url: "#", publishedAt: new Date(now - 5400000).toISOString(), category: "world", thumbnail: null },
    { id: "d6", title: "SBP holds policy rate steady as inflation shows signs of easing", description: "The State Bank of Pakistan maintained its benchmark interest rate, citing cautious optimism on the inflation trajectory.", source: "Pakistan Today", url: "#", publishedAt: new Date(now - 7200000).toISOString(), category: "business", thumbnail: null },
    { id: "d7", title: "Heavy monsoon rains forecast for Sindh and Balochistan from next week", description: "The Met Office has issued an advisory warning of above-normal rainfall expected across southern provinces.", source: "Express Tribune", url: "#", publishedAt: new Date(now - 10800000).toISOString(), category: "all", thumbnail: null },
    { id: "d8", title: "Supreme Court takes up constitutional review petition on fundamental rights", description: "A larger bench has been constituted to hear arguments on the scope of Article 25 protections.", source: "The News International", url: "#", publishedAt: new Date(now - 14400000).toISOString(), category: "politics", thumbnail: null },
  ];
}
