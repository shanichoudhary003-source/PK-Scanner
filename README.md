[index (11).html](https://github.com/user-attachments/files/26749357/index.11.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PK Wire — Live Pakistan News</title>
<meta name="description" content="Live aggregated news feed from Pakistan's top sources">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Newsreader:ital,opsz,wght@0,6..72,300;0,6..72,400;0,6..72,600;0,6..72,700;1,6..72,400&family=JetBrains+Mono:wght@400;500;600&family=DM+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0A0C10;
    --card-bg: rgba(255,255,255,0.04);
    --card-hover: rgba(255,255,255,0.07);
    --text: #E8E6E1;
    --text-muted: #7A7B80;
    --accent: #E63946;
    --accent-glow: rgba(230,57,70,0.15);
    --border: rgba(255,255,255,0.06);
    --shimmer-bg: rgba(255,255,255,0.06);
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }
  
  body {
    min-height: 100vh;
    background: var(--bg);
    color: var(--text);
    font-family: 'Newsreader', 'Georgia', serif;
    position: relative;
    overflow-x: hidden;
  }

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
  @keyframes tickerScroll {
    0% { transform: translateX(0); }
    100% { transform: translateX(-50%); }
  }
  @keyframes gradientShift {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
  }

  .ambient-bg {
    position: fixed; top: 0; left: 0; right: 0; bottom: 0;
    background: radial-gradient(ellipse at 20% 0%, rgba(230,57,70,0.06) 0%, transparent 50%),
                radial-gradient(ellipse at 80% 100%, rgba(29,53,87,0.08) 0%, transparent 50%);
    pointer-events: none; z-index: 0;
  }

  .app { position: relative; z-index: 1; }

  /* Header */
  .header {
    padding: 28px 32px 20px;
    display: flex; flex-wrap: wrap;
    align-items: center; justify-content: space-between;
    gap: 16px;
    border-bottom: 1px solid var(--border);
  }
  .header-left { display: flex; align-items: baseline; gap: 16px; }
  .logo {
    font-family: 'Newsreader', serif;
    font-size: 28px; font-weight: 700;
    letter-spacing: -0.5px; color: var(--text); line-height: 1;
  }
  .logo span { color: var(--accent); }
  .live-badge { display: inline-flex; align-items: center; gap: 6px; }
  .live-dot {
    width: 8px; height: 8px; border-radius: 50%;
    background: #E63946; animation: pulse 1.5s infinite;
    box-shadow: 0 0 6px #E63946;
  }
  .live-text {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px; font-weight: 600;
    letter-spacing: 2px; text-transform: uppercase; color: #E63946;
  }
  .header-right { display: flex; align-items: center; gap: 16px; }
  .search-wrap { position: relative; }
  .search-icon {
    position: absolute; left: 14px; top: 50%; transform: translateY(-50%); opacity: 0.4;
  }
  .search-input {
    background: rgba(255,255,255,0.05);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 12px 16px 12px 42px;
    color: var(--text);
    font-family: 'DM Sans', sans-serif;
    font-size: 14px; width: 100%; max-width: 320px;
    outline: none; transition: all 0.25s;
  }
  .search-input:focus {
    border-color: rgba(230,57,70,0.4);
    background: rgba(255,255,255,0.07);
    box-shadow: 0 0 0 3px rgba(230,57,70,0.1);
  }
  .search-input::placeholder { color: var(--text-muted); }
  .updated-time {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px; color: var(--text-muted); white-space: nowrap;
  }

  /* Ticker */
  .ticker-strip {
    background: linear-gradient(90deg, #E63946 0%, #c0392b 50%, #E63946 100%);
    background-size: 200% 100%;
    animation: gradientShift 8s ease infinite;
    padding: 10px 0; overflow: hidden; position: relative;
  }
  .ticker-content {
    display: flex; animation: tickerScroll 60s linear infinite; white-space: nowrap;
  }
  .ticker-item {
    font-family: 'DM Sans', sans-serif;
    font-size: 13px; font-weight: 600; color: white;
    padding: 0 40px; display: flex; align-items: center; gap: 12px;
  }
  .ticker-dot { width: 4px; height: 4px; background: rgba(255,255,255,0.6); border-radius: 50%; }

  /* Categories */
  .categories {
    padding: 18px 32px; display: flex; gap: 8px;
    overflow-x: auto; border-bottom: 1px solid var(--border);
    align-items: center;
  }
  .cat-btn {
    padding: 8px 18px; border-radius: 100px;
    border: 1px solid var(--border); background: transparent;
    color: var(--text-muted); font-family: 'DM Sans', sans-serif;
    font-size: 13px; font-weight: 500; cursor: pointer;
    transition: all 0.25s; white-space: nowrap;
  }
  .cat-btn:hover { border-color: rgba(255,255,255,0.2); color: var(--text); }
  .cat-btn.active {
    background: var(--accent); border-color: var(--accent);
    color: #fff; box-shadow: 0 2px 12px var(--accent-glow);
  }
  .story-count {
    margin-left: auto;
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px; color: var(--text-muted);
  }

  /* Main */
  .main { padding: 24px 32px 60px; max-width: 900px; margin: 0 auto; }

  /* Breaking Card */
  .breaking-card {
    background: linear-gradient(135deg, rgba(230,57,70,0.12), rgba(230,57,70,0.03));
    border: 1px solid rgba(230,57,70,0.2);
    border-radius: 18px; padding: 32px;
    position: relative; overflow: hidden;
    cursor: pointer; transition: all 0.3s;
    margin-bottom: 28px; animation: fadeInUp 0.5s both;
    text-decoration: none; color: inherit; display: block;
  }
  .breaking-card:hover {
    border-color: rgba(230,57,70,0.4);
    box-shadow: 0 12px 48px rgba(230,57,70,0.15);
  }
  .breaking-card::after {
    content: ''; position: absolute; top: -50%; right: -20%;
    width: 300px; height: 300px;
    background: radial-gradient(circle, rgba(230,57,70,0.08), transparent 70%);
    pointer-events: none;
  }
  .breaking-label {
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px; font-weight: 700;
    letter-spacing: 2px; text-transform: uppercase;
    color: var(--accent); padding: 3px 8px;
    background: rgba(230,57,70,0.15); border-radius: 4px;
  }
  .breaking-title {
    font-family: 'Newsreader', serif;
    font-size: 26px; font-weight: 700;
    line-height: 1.3; margin-bottom: 12px; color: var(--text);
  }
  .breaking-desc {
    font-family: 'DM Sans', sans-serif;
    font-size: 15px; line-height: 1.6;
    color: var(--text-muted); max-width: 700px;
  }

  /* News Card */
  .news-card {
    background: var(--card-bg);
    border-radius: 14px; padding: 28px 24px;
    cursor: pointer; transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
    border: 1px solid transparent;
    position: relative; overflow: hidden;
    text-decoration: none; color: inherit; display: block;
  }
  .news-card .card-line {
    position: absolute; top: 0; left: 0;
    width: 3px; height: 100%;
    border-radius: 3px 0 0 3px;
    opacity: 0; transition: opacity 0.3s;
  }
  .news-card:hover {
    background: var(--card-hover);
    border-color: var(--border);
    transform: translateY(-2px);
    box-shadow: 0 8px 32px rgba(0,0,0,0.3);
  }
  .news-card:hover .card-line { opacity: 1; }
  .card-meta { display: flex; align-items: center; gap: 10px; margin-bottom: 12px; flex-wrap: wrap; }
  .source-tag {
    display: inline-block; padding: 4px 10px; border-radius: 6px;
    font-family: 'DM Sans', sans-serif;
    font-size: 11px; font-weight: 600;
    letter-spacing: 0.5px; text-transform: uppercase;
  }
  .time-tag {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px; color: var(--text-muted);
  }
  .cat-tag {
    font-family: 'DM Sans', sans-serif;
    font-size: 11px; color: var(--text-muted);
    background: rgba(255,255,255,0.04);
    padding: 2px 8px; border-radius: 4px; text-transform: capitalize;
  }
  .card-title {
    font-family: 'Newsreader', serif;
    font-size: 19px; font-weight: 600;
    line-height: 1.35; margin-bottom: 8px; color: var(--text);
  }
  .card-desc {
    font-family: 'DM Sans', sans-serif;
    font-size: 14px; line-height: 1.55;
    color: var(--text-muted);
    display: -webkit-box; -webkit-line-clamp: 2;
    -webkit-box-orient: vertical; overflow: hidden;
  }

  /* Feed */
  .feed { display: flex; flex-direction: column; gap: 12px; }

  /* Shimmer */
  .shimmer-card {
    background: var(--card-bg);
    border-radius: 14px; padding: 28px 24px; overflow: hidden;
  }
  .shimmer-bar {
    border-radius: 6px; background: var(--shimmer-bg);
    animation: shimmer 1.5s infinite;
  }

  /* Empty */
  .empty-state {
    text-align: center; padding: 60px 20px; animation: fadeInUp 0.4s both;
  }
  .empty-state .icon { font-size: 48px; margin-bottom: 16px; opacity: 0.3; }
  .empty-state p {
    font-family: 'DM Sans', sans-serif;
    font-size: 16px; color: var(--text-muted);
  }

  /* Footer */
  .footer {
    padding: 20px 32px;
    border-top: 1px solid var(--border);
    display: flex; justify-content: space-between; align-items: center;
    font-family: 'DM Sans', sans-serif;
    font-size: 12px; color: var(--text-muted);
    flex-wrap: wrap; gap: 8px;
  }

  /* Scrollbar */
  ::-webkit-scrollbar { width: 6px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.1); border-radius: 3px; }
  ::-webkit-scrollbar-thumb:hover { background: rgba(255,255,255,0.2); }

  /* Mobile */
  @media (max-width: 640px) {
    .header { padding: 20px 16px 16px; }
    .categories { padding: 14px 16px; }
    .main { padding: 16px 16px 40px; }
    .footer { padding: 16px; }
    .logo { font-size: 22px; }
    .breaking-title { font-size: 20px; }
    .breaking-card { padding: 24px 20px; }
    .search-input { max-width: 100%; }
    .header-right { width: 100%; }
    .search-wrap { flex: 1; }
  }
</style>
</head>
<body>
<div class="ambient-bg"></div>
<div class="app">

  <!-- Header -->
  <header class="header">
    <div class="header-left">
      <h1 class="logo"><span>PK</span> Wire</h1>
      <span class="live-badge">
        <span class="live-dot"></span>
        <span class="live-text">Live</span>
      </span>
    </div>
    <div class="header-right">
      <div class="search-wrap">
        <svg class="search-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/>
        </svg>
        <input class="search-input" id="searchInput" placeholder="Search stories..." />
      </div>
      <span class="updated-time" id="updatedTime"></span>
    </div>
  </header>

  <!-- Ticker -->
  <div class="ticker-strip" id="tickerStrip" style="display:none;">
    <div class="ticker-content" id="tickerContent"></div>
  </div>

  <!-- Categories -->
  <div class="categories" id="categories">
    <button class="cat-btn active" data-cat="all">All</button>
    <button class="cat-btn" data-cat="politics">Politics</button>
    <button class="cat-btn" data-cat="business">Business</button>
    <button class="cat-btn" data-cat="sports">Sports</button>
    <button class="cat-btn" data-cat="technology">Tech</button>
    <button class="cat-btn" data-cat="world">World</button>
    <span class="story-count" id="storyCount"></span>
  </div>

  <!-- Main -->
  <main class="main">
    <div id="breakingSlot"></div>
    <div class="feed" id="feed"></div>
    <div id="loading">
      <div style="display:flex;flex-direction:column;gap:16px;">
        <div class="shimmer-card"><div class="shimmer-bar" style="width:70px;height:18px;margin-bottom:16px;"></div><div class="shimmer-bar" style="width:90%;height:20px;margin-bottom:10px;"></div><div class="shimmer-bar" style="width:70%;height:20px;margin-bottom:16px;"></div><div class="shimmer-bar" style="width:100%;height:14px;margin-bottom:6px;"></div><div class="shimmer-bar" style="width:85%;height:14px;"></div></div>
        <div class="shimmer-card"><div class="shimmer-bar" style="width:90px;height:18px;margin-bottom:16px;"></div><div class="shimmer-bar" style="width:80%;height:20px;margin-bottom:10px;"></div><div class="shimmer-bar" style="width:60%;height:20px;margin-bottom:16px;"></div><div class="shimmer-bar" style="width:95%;height:14px;margin-bottom:6px;"></div><div class="shimmer-bar" style="width:75%;height:14px;"></div></div>
        <div class="shimmer-card"><div class="shimmer-bar" style="width:80px;height:18px;margin-bottom:16px;"></div><div class="shimmer-bar" style="width:85%;height:20px;margin-bottom:10px;"></div><div class="shimmer-bar" style="width:65%;height:20px;margin-bottom:16px;"></div><div class="shimmer-bar" style="width:90%;height:14px;margin-bottom:6px;"></div><div class="shimmer-bar" style="width:80%;height:14px;"></div></div>
      </div>
    </div>
  </main>

  <!-- Footer -->
  <footer class="footer">
    <span>Sources: Dawn · The News · Express Tribune · Dunya News · Pakistan Today</span>
    <span>Auto-refreshes every 2 minutes</span>
  </footer>
</div>

<script>
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

const FEEDS = [
  { url: "https://www.dawn.com/feeds/home", source: "Dawn" },
  { url: "https://www.thenews.com.pk/rss/1/1", source: "The News International" },
  { url: "https://tribune.com.pk/feed", source: "Express Tribune" },
  { url: "https://dunyanews.tv/feed", source: "Dunya News" },
  { url: "https://www.pakistantoday.com.pk/feed/", source: "Pakistan Today" },
];

let allArticles = [];
let activeCategory = "all";
let searchQuery = "";

function categorize(title, desc) {
  const t = `${title} ${desc}`.toLowerCase();
  if (/cricket|psl|match|pcb|batting|bowling|icc|hockey|football|fifa|sport/.test(t)) return "sports";
  if (/election|parliament|pm |pti|pmln|ppp|senate|govt|government|minister|nawaz|imran|bilawal|political|court|judiciary/.test(t)) return "politics";
  if (/stock|market|rupee|economy|gdp|trade|export|import|sbp|inflation|fiscal|investment|bank/.test(t)) return "business";
  if (/tech|ai |startup|software|digital|cyber|app |internet|5g|mobile|smartphone/.test(t)) return "technology";
  if (/us |china|india|iran|saudi|russia|ukraine|biden|trump|un |nato|global|international/.test(t)) return "world";
  return "all";
}

function timeAgo(dateStr) {
  const now = new Date();
  const date = new Date(dateStr);
  const mins = Math.floor((now - date) / 60000);
  if (mins < 1) return "Just now";
  if (mins < 60) return `${mins}m ago`;
  const hrs = Math.floor(mins / 60);
  if (hrs < 24) return `${hrs}h ago`;
  return `${Math.floor(hrs / 24)}d ago`;
}

function stripHtml(s) { const d = document.createElement("div"); d.innerHTML = s; return d.textContent || ""; }

async function fetchNews() {
  const proxy = "https://api.rss2json.com/v1/api.json?rss_url=";

  const results = await Promise.allSettled(
    FEEDS.map(async (feed) => {
      const res = await fetch(`${proxy}${encodeURIComponent(feed.url)}`);
      if (!res.ok) throw new Error();
      const data = await res.json();
      if (data.status !== "ok" || !data.items) return [];
      return data.items.map((item, i) => ({
        id: `${feed.source}-${i}-${item.pubDate}`,
        title: stripHtml(item.title || "").trim(),
        description: stripHtml(item.description || "").substring(0, 280).trim(),
        source: feed.source,
        url: item.link,
        publishedAt: item.pubDate,
        category: categorize(item.title || "", item.description || ""),
      }));
    })
  );

  const articles = results
    .filter(r => r.status === "fulfilled")
    .flatMap(r => r.value)
    .filter(a => a.title && a.title.length > 10)
    .sort((a, b) => new Date(b.publishedAt) - new Date(a.publishedAt));

  if (articles.length > 0) {
    allArticles = articles;
  } else if (allArticles.length === 0) {
    allArticles = getDemoArticles();
  }

  document.getElementById("loading").style.display = "none";
  const now = new Date();
  document.getElementById("updatedTime").textContent =
    `Updated ${now.toLocaleTimeString("en-GB", { hour: "2-digit", minute: "2-digit" })}`;

  buildTicker();
  render();
}

function buildTicker() {
  const strip = document.getElementById("tickerStrip");
  const content = document.getElementById("tickerContent");
  if (allArticles.length === 0) { strip.style.display = "none"; return; }
  strip.style.display = "block";
  const items = [...allArticles.slice(0, 10), ...allArticles.slice(0, 10)];
  content.innerHTML = items.map(a =>
    `<span class="ticker-item"><span class="ticker-dot"></span>${a.title.substring(0, 80)}${a.title.length > 80 ? "..." : ""}</span>`
  ).join("");
}

function render() {
  const filtered = allArticles.filter(a => {
    const matchCat = activeCategory === "all" || a.category === activeCategory;
    const matchSearch = !searchQuery ||
      a.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
      a.source.toLowerCase().includes(searchQuery.toLowerCase());
    return matchCat && matchSearch;
  });

  document.getElementById("storyCount").textContent = `${filtered.length} stories`;

  const breakingSlot = document.getElementById("breakingSlot");
  const feed = document.getElementById("feed");

  let startIdx = 0;

  // Breaking card
  if (filtered.length > 0 && activeCategory === "all" && !searchQuery) {
    const b = filtered[0];
    const sc = SOURCES[b.source] || "#444";
    breakingSlot.innerHTML = `
      <a href="${b.url}" target="_blank" rel="noopener noreferrer" class="breaking-card">
        <div style="display:flex;align-items:center;gap:10px;margin-bottom:14px;">
          <span class="breaking-label">Breaking</span>
          <span class="source-tag" style="background:${sc};color:#fff;">${b.source}</span>
        </div>
        <h2 class="breaking-title">${b.title}</h2>
        ${b.description ? `<p class="breaking-desc">${b.description}</p>` : ""}
        <span class="time-tag" style="margin-top:14px;display:block;">${timeAgo(b.publishedAt)}</span>
      </a>`;
    startIdx = 1;
  } else {
    breakingSlot.innerHTML = "";
  }

  const cards = filtered.slice(startIdx);
  if (cards.length === 0 && startIdx === 0) {
    feed.innerHTML = `<div class="empty-state"><div class="icon">📰</div><p>No stories found for this filter</p></div>`;
    return;
  }

  feed.innerHTML = cards.map((a, i) => {
    const sc = SOURCES[a.source] || "#666";
    return `
      <a href="${a.url}" target="_blank" rel="noopener noreferrer" class="news-card" style="animation:fadeInUp 0.4s ${Math.min(i * 0.05, 0.5)}s both;">
        <div class="card-line" style="background:${sc};"></div>
        <div class="card-meta">
          <span class="source-tag" style="background:${sc}22;color:${sc};">${a.source}</span>
          <span class="time-tag">${timeAgo(a.publishedAt)}</span>
          ${a.category !== "all" ? `<span class="cat-tag">${a.category}</span>` : ""}
        </div>
        <h3 class="card-title">${a.title}</h3>
        ${a.description ? `<p class="card-desc">${a.description}</p>` : ""}
      </a>`;
  }).join("");
}

// Category buttons
document.getElementById("categories").addEventListener("click", (e) => {
  const btn = e.target.closest(".cat-btn");
  if (!btn) return;
  activeCategory = btn.dataset.cat;
  document.querySelectorAll(".cat-btn").forEach(b => b.classList.remove("active"));
  btn.classList.add("active");
  render();
});

// Search
document.getElementById("searchInput").addEventListener("input", (e) => {
  searchQuery = e.target.value;
  render();
});

// Demo fallback
function getDemoArticles() {
  const now = Date.now();
  return [
    { id:"d1", title:"National Assembly passes federal budget amendments amid opposition walkout", description:"The lower house approved several amendments to the Finance Bill after heated debate, with opposition members staging a walkout over disputed clauses.", source:"Dawn", url:"https://www.dawn.com", publishedAt:new Date(now-300000).toISOString(), category:"politics" },
    { id:"d2", title:"Pakistan Stock Exchange surges past 85,000 milestone on strong institutional buying", description:"The KSE-100 index rallied sharply driven by blue-chip stocks in banking and energy sectors.", source:"The News International", url:"https://www.thenews.com.pk", publishedAt:new Date(now-900000).toISOString(), category:"business" },
    { id:"d3", title:"PCB announces squad for upcoming bilateral series against South Africa", description:"The Pakistan Cricket Board revealed a 16-member squad including three uncapped players for the upcoming tour.", source:"Express Tribune", url:"https://tribune.com.pk", publishedAt:new Date(now-1800000).toISOString(), category:"sports" },
    { id:"d4", title:"PM outlines new digital transformation initiative for government services", description:"The Prime Minister announced a comprehensive plan to digitise public services across all provinces within the next two years.", source:"Dunya News", url:"https://dunyanews.tv", publishedAt:new Date(now-3600000).toISOString(), category:"technology" },
    { id:"d5", title:"Pakistan and China agree to expand CPEC corridor with new infrastructure projects", description:"Both nations signed memoranda of understanding covering rail, port, and renewable energy projects valued at several billion dollars.", source:"Dawn", url:"https://www.dawn.com", publishedAt:new Date(now-5400000).toISOString(), category:"world" },
    { id:"d6", title:"SBP holds policy rate steady as inflation shows signs of easing", description:"The State Bank of Pakistan maintained its benchmark interest rate, citing cautious optimism on the inflation trajectory.", source:"Pakistan Today", url:"https://www.pakistantoday.com.pk", publishedAt:new Date(now-7200000).toISOString(), category:"business" },
    { id:"d7", title:"Heavy monsoon rains forecast for Sindh and Balochistan from next week", description:"The Met Office has issued an advisory warning of above-normal rainfall expected across southern provinces.", source:"Express Tribune", url:"https://tribune.com.pk", publishedAt:new Date(now-10800000).toISOString(), category:"all" },
    { id:"d8", title:"Supreme Court takes up constitutional review petition on fundamental rights", description:"A larger bench has been constituted to hear arguments on the scope of Article 25 protections.", source:"The News International", url:"https://www.thenews.com.pk", publishedAt:new Date(now-14400000).toISOString(), category:"politics" },
  ];
}

// Init
fetchNews();
setInterval(fetchNews, 120000);
</script>
</body>
</html>
