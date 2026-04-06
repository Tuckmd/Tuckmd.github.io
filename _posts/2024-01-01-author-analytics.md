---
title: "Author Research Analytics"
date: 2024-01-01
permalink: /posts/2024/01/author-analytics/
tags:
  - tools
  - research
---

Type in any researcher's name to see their publication venues, research keywords, citation stats, and co-author network pulled live from OpenAlex. Results are filtered to peer-reviewed articles, reviews, book chapters, and conference papers only.

<style>
* { box-sizing: border-box; }
#search-row { display: flex; gap: 8px; margin-bottom: 1.5rem; }
#search-row input { flex: 1; padding: 8px 12px; border: 1px solid #ccc; border-radius: 6px; font-size: 14px; }
#search-row button { padding: 8px 16px; border: 1px solid #ccc; border-radius: 6px; cursor: pointer; font-size: 14px; }
#error { color: red; font-size: 13px; margin-top: 8px; min-height: 18px; }
#results { display: none; }
.stat-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin-bottom: 2rem; }
.stat { background: #f5f5f5; border-radius: 8px; padding: 1rem; }
.stat-label { font-size: 13px; color: #666; margin-bottom: 4px; }
.stat-value { font-size: 24px; font-weight: 500; }
.section { margin-bottom: 2rem; }
.section h2 { font-size: 16px; font-weight: 500; margin-bottom: 1rem; border-bottom: 1px solid #eee; padding-bottom: 8px; }
.bar-row { display: flex; align-items: center; gap: 10px; margin-bottom: 8px; }
.bar-label { font-size: 12px; color: #555; width: 220px; flex-shrink: 0; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.bar-track { flex: 1; background: #eee; border-radius: 3px; height: 18px; overflow: hidden; }
.bar-fill { height: 100%; border-radius: 3px; background: #378ADD; }
.bar-count { font-size: 12px; color: #666; width: 28px; text-align: right; }
.tag-cloud { display: flex; flex-wrap: wrap; gap: 8px; }
.tag { padding: 4px 10px; border-radius: 99px; font-size: 12px; border: 1px solid #ddd; color: #555; background: #f5f5f5; }
.tag.large { font-size: 14px; font-weight: 500; color: #0C447C; background: #E6F1FB; border-color: #B5D4F4; }
.tag.medium { font-size: 13px; color: #185FA5; background: #E6F1FB; border-color: #B5D4F4; }
.match-btn { display: block; width: 100%; text-align: left; padding: 10px 14px; margin-bottom: 6px; border-radius: 8px; border: 1px solid #ddd; background: white; cursor: pointer; font-size: 13px; }
.match-btn:hover { background: #f5f5f5; }
.match-name { font-weight: 500; }
.match-meta { font-size: 12px; color: #666; margin-top: 2px; }
#loading { font-size: 13px; color: #666; padding: 1rem 0; display: none; }
#author-header { margin-bottom: 1.5rem; }
#author-header h2 { font-size: 18px; font-weight: 500; border: none; padding: 0; margin-bottom: 4px; }
#author-header p { font-size: 13px; color: #666; }
.filter-note { font-size: 12px; color: #888; margin-bottom: 1.5rem; }
</style>

<div id="search-row">
  <input type="text" id="author-input" placeholder="Enter author name (e.g. Richard Caprioli)" />
  <button id="search-btn">Search</button>
</div>
<div id="error"></div>
<div id="loading">Searching OpenAlex...</div>
<div id="author-matches"></div>
<div id="results">
  <div id="author-header">
    <h2 id="author-name-display"></h2>
    <p id="author-affil-display"></p>
  </div>
  <p class="filter-note">Showing peer-reviewed articles, reviews, book chapters, and conference papers only.</p>
  <div class="stat-grid">
    <div class="stat"><div class="stat-label">Filtered works</div><div class="stat-value" id="stat-works">—</div></div>
    <div class="stat"><div class="stat-label">Total citations</div><div class="stat-value" id="stat-cites">—</div></div>
    <div class="stat"><div class="stat-label">h-index</div><div class="stat-value" id="stat-h">—</div></div>
    <div class="stat"><div class="stat-label">Active years</div><div class="stat-value" id="stat-years">—</div></div>
  </div>
  <div class="section">
    <h2>Publications per year</h2>
    <div style="position:relative;width:100%;height:200px;"><canvas id="year-chart"></canvas></div>
  </div>
  <div class="section">
    <h2>Top publication venues</h2>
    <div id="venues-list"></div>
  </div>
  <div class="section">
    <h2>Research keywords</h2>
    <div class="tag-cloud" id="keywords-cloud"></div>
  </div>
  <div class="section">
    <h2>Top co-authors</h2>
    <div id="coauthors-list"></div>
  </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<script>
let yearChart = null;
const TYPE_FILTER = 'article|review|book-chapter|book|proceedings-article';
const EMAIL = 'tuckmd1@gmail.com';

async function searchAuthors(name) {
  const url = `https://api.openalex.org/authors?search=${encodeURIComponent(name)}&per-page=5&mailto=${EMAIL}`;
  const res = await fetch(url);
  if (!res.ok) throw new Error('API error');
  const data = await res.json();
  return data.results || [];
}

async function fetchAuthorWorks(authorId) {
  const id = authorId.replace('https://openalex.org/', '');
  let allWorks = [], page = 1;
  while (true) {
    const url = `https://api.openalex.org/works?filter=author.id:${id},type:${TYPE_FILTER}&per-page=200&page=${page}&mailto=${EMAIL}`;
    const res = await fetch(url);
    if (!res.ok) break;
    const data = await res.json();
    allWorks = allWorks.concat(data.results || []);
    if (allWorks.length >= data.meta.count || data.results.length < 200) break;
    page++;
    if (page > 5) break;
  }
  return allWorks;
}

function renderAuthor(author) {
  document.getElementById('author-name-display').textContent = author.display_name;
  const inst = author.last_known_institutions?.[0]?.display_name ||
               author.last_known_institution?.display_name || '';
  document.getElementById('author-affil-display').textContent = inst;
  document.getElementById('stat-cites').textContent = (author.cited_by_count || 0).toLocaleString();
  document.getElementById('stat-h').textContent = author.summary_stats?.h_index ?? '—';
}

function renderWorks(works) {
  if (!works.length) {
    document.getElementById('error').textContent = 'No qualifying publications found for this author.';
    return;
  }

  document.getElementById('stat-works').textContent = works.length.toLocaleString();

  const years = works.map(w => w.publication_year).filter(Boolean);
  const minY = Math.min(...years), maxY = Math.max(...years);
  document.getElementById('stat-years').textContent = minY === maxY ? String(minY) : `${minY}–${maxY}`;

  const yearCounts = {};
  years.forEach(y => { yearCounts[y] = (yearCounts[y] || 0) + 1; });
  const sortedYears = Object.keys(yearCounts).sort();
  if (yearChart) yearChart.destroy();
  yearChart = new Chart(document.getElementById('year-chart'), {
    type: 'bar',
    data: {
      labels: sortedYears,
      datasets: [{
        data: sortedYears.map(y => yearCounts[y]),
        backgroundColor: '#378ADD',
        borderRadius: 3,
        borderSkipped: false
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: { grid: { display: false }, ticks: { font: { size: 11 }, autoSkip: true, maxRotation: 45 } },
        y: { grid: { color: 'rgba(0,0,0,0.06)' }, ticks: { font: { size: 11 }, stepSize: 1 } }
      }
    }
  });

  const venueCounts = {};
  works.forEach(w => {
    const v = w.primary_location?.source?.display_name;
    if (v) venueCounts[v] = (venueCounts[v] || 0) + 1;
  });
  const topVenues = Object.entries(venueCounts).sort((a, b) => b[1] - a[1]).slice(0, 10);
  const maxV = topVenues[0]?.[1] || 1;
  document.getElementById('venues-list').innerHTML = topVenues.map(([name, count]) =>
    `<div class="bar-row">
      <div class="bar-label" title="${name}">${name}</div>
      <div class="bar-track"><div class="bar-fill" style="width:${Math.round(count/maxV*100)}%"></div></div>
      <div class="bar-count">${count}</div>
    </div>`
  ).join('');

  const conceptCounts = {};
  works.forEach(w => {
    (w.concepts || []).forEach(c => {
      if (c.level >= 1 && c.level <= 3)
        conceptCounts[c.display_name] = (conceptCounts[c.display_name] || 0) + (c.score || 1);
    });
  });
  const topConcepts = Object.entries(conceptCounts).sort((a, b) => b[1] - a[1]).slice(0, 30);
  const maxC = topConcepts[0]?.[1] || 1;
  document.getElementById('keywords-cloud').innerHTML = topConcepts.map(([name, score]) => {
    const pct = score / maxC;
    const cls = pct > 0.5 ? 'large' : pct > 0.25 ? 'medium' : '';
    return `<span class="tag ${cls}">${name}</span>`;
  }).join('');

  const coauthorCounts = {};
  works.forEach(w => {
    (w.authorships || []).forEach(a => {
      const n = a.author?.display_name;
      if (n) coauthorCounts[n] = (coauthorCounts[n] || 0) + 1;
    });
  });
  const topCoauthors = Object.entries(coauthorCounts).sort((a, b) => b[1] - a[1]).slice(1, 11);
  const maxCo = topCoauthors[0]?.[1] || 1;
  document.getElementById('coauthors-list').innerHTML = topCoauthors.map(([name, count]) =>
    `<div class="bar-row">
      <div class="bar-label">${name}</div>
      <div class="bar-track"><div class="bar-fill" style="width:${Math.round(count/maxCo*100)}%;background:#1D9E75;"></div></div>
      <div class="bar-count">${count}</div>
    </div>`
  ).join('');
}

async function doSearch() {
  const name = document.getElementById('author-input').value.trim();
  if (!name) return;
  document.getElementById('error').textContent = '';
  document.getElementById('author-matches').innerHTML = '';
  document.getElementById('results').style.display = 'none';
  document.getElementById('loading').style.display = 'block';
  document.getElementById('loading').textContent = 'Searching OpenAlex...';
  try {
    const authors = await searchAuthors(name);
    document.getElementById('loading').style.display = 'none';
    if (!authors.length) {
      document.getElementById('error').textContent = 'No authors found. Try a different name.';
      return;
    }
    if (authors.length === 1) {
      await loadAuthor(authors[0]);
    } else {
      const box = document.getElementById('author-matches');
      box.innerHTML = '<div style="font-size:13px;color:#666;margin-bottom:8px;">Multiple authors found — select one:</div>';
      authors.forEach(a => {
        const inst = a.last_known_institutions?.[0]?.display_name ||
                     a.last_known_institution?.display_name || 'Unknown institution';
        const btn = document.createElement('button');
        btn.className = 'match-btn';
        btn.innerHTML = `<div class="match-name">${a.display_name}</div>
          <div class="match-meta">${inst} · ${(a.works_count || 0)} works · ${(a.cited_by_count || 0).toLocaleString()} citations</div>`;
        btn.onclick = () => { box.innerHTML = ''; loadAuthor(a); };
        box.appendChild(btn);
      });
    }
  } catch(e) {
    document.getElementById('loading').style.display = 'none';
    document.getElementById('error').textContent = 'Error connecting to OpenAlex. Please try again.';
  }
}

async function loadAuthor(author) {
  document.getElementById('loading').style.display = 'block';
  document.getElementById('loading').textContent = `Loading works for ${author.display_name}...`;
  renderAuthor(author);
  try {
    const works = await fetchAuthorWorks(author.id);
    renderWorks(works);
    document.getElementById('results').style.display = 'block';
  } catch(e) {
    document.getElementById('error').textContent = 'Error loading works.';
  }
  document.getElementById('loading').style.display = 'none';
}

document.getElementById('search-btn').onclick = doSearch;
document.getElementById('author-input').addEventListener('keydown', e => {
  if (e.key === 'Enter') doSearch();
});
</script>
