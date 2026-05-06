# 九州家族旅行 Website Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `index.html` travel diary website for the Kyushu family trip — interactive Google Map with landmark pins, 7-day scrollable timeline with photo uploads and author notes, responsive on mobile and desktop.

**Architecture:** Pure static HTML file with embedded `<style>` CSS and `<script>` JS. Google Maps JavaScript API renders colored pins; vanilla JS dynamically renders all timeline cards from a data array. Photos upload via `<input type="file">`, are read as base64 via FileReader, and persisted in `localStorage`. No build step, no npm, no frameworks.

**Tech Stack:** HTML5, CSS3 (custom properties, CSS Grid, Flexbox), Vanilla JavaScript ES6, Google Maps JavaScript API v3

---

## File Structure

| Path | Purpose |
|------|---------|
| `index.html` | Entire site: HTML skeleton + `<style>` CSS + `<script>` JS data & rendering |

---

## Task 1: HTML Skeleton + CSS Design Tokens + Nav Bar

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create `index.html` with design tokens, reset, and nav**

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>九州家族旅行 2025</title>
  <style>
    :root {
      --green: #748573;
      --white: #ffffff;
      --gray-bg: #f0f0f0;
      --gray-text: #888888;
      --gray-border: #dddddd;
      --font: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Helvetica Neue', sans-serif;
      --card-radius: 12px;
      --card-shadow: 0 1px 4px rgba(0,0,0,0.08);
    }

    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: var(--font);
      background: var(--white);
      color: #1a1a1a;
      -webkit-font-smoothing: antialiased;
    }

    /* ── Nav ─────────────────────────────── */
    .nav {
      position: sticky;
      top: 0;
      z-index: 200;
      background: var(--white);
      border-bottom: 1px solid var(--gray-border);
      padding: 14px 16px;
    }
    .nav-title {
      font-size: 15px;
      font-weight: 600;
      letter-spacing: 0.01em;
    }
  </style>
</head>
<body>

  <nav class="nav">
    <span class="nav-title">九州家族旅行 2025</span>
  </nav>

  <!-- Map and timeline will be added in later tasks -->

</body>
</html>
```

- [ ] **Step 2: Verify nav in browser**

Open `index.html` in a browser (double-click file, or `open index.html` in terminal).
Expected: white page with sticky nav showing "九州家族旅行 2025", 1px bottom border.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add HTML skeleton with design tokens and nav bar"
```

---

## Task 2: Google Maps Container + API Initialization

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add map container HTML and CSS**

Inside `<body>`, after `</nav>`, add:

```html
  <div class="main-layout">
    <div id="map" class="map-container"></div>
    <div id="timeline" class="timeline"></div>
  </div>
```

Inside `<style>`, add:

```css
    /* ── Layout ──────────────────────────── */
    .main-layout {
      display: flex;
      flex-direction: column;
    }

    /* ── Map ─────────────────────────────── */
    .map-container {
      height: 40vh;
      width: 100%;
      background: var(--gray-bg);
    }

    /* ── Timeline (mobile) ───────────────── */
    .timeline {
      padding: 0 16px 80px;
    }
```

- [ ] **Step 2: Add Google Maps API script tags**

At the bottom of `<body>`, before `</body>`, add:

```html
  <script>
    function initMap() {
      const map = new google.maps.Map(document.getElementById('map'), {
        center: { lat: 33.2, lng: 130.9 },
        zoom: 8,
        disableDefaultUI: true,
        zoomControl: true,
        mapTypeControl: false,
        styles: [
          { featureType: 'poi', elementType: 'labels', stylers: [{ visibility: 'off' }] },
          { featureType: 'transit', elementType: 'labels', stylers: [{ visibility: 'off' }] }
        ]
      });
      window._map = map;
      // Pins will be added in Task 4 after data is defined
    }
  </script>

  <!-- Replace YOUR_API_KEY with your Google Maps API key -->
  <script
    src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap"
    async defer>
  </script>
```

- [ ] **Step 3: Verify map loads**

Replace `YOUR_API_KEY` with a real key temporarily for testing, or observe the gray placeholder div if no key available.
Expected with key: Google Map centered on Kyushu renders in the top 40vh.
Expected without key: Gray `#f0f0f0` placeholder div appears — layout is correct.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Google Maps container and API initialization"
```

---

## Task 3: Landmark Data Array (All 7 Days)

**Files:**
- Modify: `index.html` (add JS data before the `initMap` function)

- [ ] **Step 1: Add LANDMARKS data array**

Inside the existing `<script>` block, add this BEFORE the `initMap` function:

```javascript
    const LANDMARKS = [
      // ── Day 1 · 福岡 ──────────────────────────────────────────
      {
        id: 'day1-hakata',
        day: 1, city: '福岡',
        name: '博多駅',
        hours: '全天開放',
        tags: ['推薦'],
        notes: [
          { author: 'E', text: '抵達！先去車站周邊逛逛，博多口和筑紫口都有好吃的' },
          { author: 'S', text: '博多阪急百貨 B1 食品區必逛，地下美食很多' }
        ],
        lat: 33.5898, lng: 130.4205
      },
      {
        id: 'day1-tenjin',
        day: 1, city: '福岡',
        name: '天神地下街',
        hours: '10:00–20:00',
        tags: ['必買', '推薦'],
        notes: [
          { author: 'E', text: '天神地下街超好買，雨天也不怕！藥妝店一間接一間' },
          { author: 'S', text: '帶小孩逛很方便，不用曬太陽' }
        ],
        lat: 33.5902, lng: 130.3985
      },

      // ── Day 2 · 太宰府 ────────────────────────────────────────
      {
        id: 'day2-sando',
        day: 2, city: '太宰府',
        name: '太宰府天滿宮參道',
        hours: '09:00–17:00',
        tags: ['必買', '推薦'],
        notes: [
          { author: 'E', text: '梅枝餅一定要現烤的！Starbucks 也在參道上，造型超特別' },
          { author: 'S', text: '試吃超多，小孩很開心，各種梅子口味零食' }
        ],
        lat: 33.5210, lng: 130.5320
      },
      {
        id: 'day2-dazaifu',
        day: 2, city: '太宰府',
        name: '太宰府天滿宮',
        hours: '06:30–19:00（依季節）',
        tags: ['推薦', '親子友善'],
        notes: [
          { author: 'E', text: '學業之神，來拜一下！牛像和梅花照片很美' },
          { author: 'S', text: '小孩很喜歡拜神儀式，氛圍很好' }
        ],
        lat: 33.5227, lng: 130.5337
      },

      // ── Day 3 · 福岡 ──────────────────────────────────────────
      {
        id: 'day3-nakasu',
        day: 3, city: '福岡',
        name: '中洲川端商店街',
        hours: '10:00–20:00',
        tags: ['推薦'],
        notes: [
          { author: 'E', text: '白天可以逛商店街，那珂川旁散步很舒服' },
          { author: 'S', text: '超適合親子，商店街寬敞好推車' }
        ],
        lat: 33.5938, lng: 130.4052
      },
      {
        id: 'day3-yatai',
        day: 3, city: '福岡',
        name: '中洲屋台',
        hours: '18:00–凌晨 01:00',
        tags: ['推薦', '必吃'],
        notes: [
          { author: 'E', text: '福岡最必去！豚骨拉麵配串燒，河邊氣氛超好' },
          { author: 'S', text: '帶小孩去要早點（18:00），人少比較好坐' }
        ],
        lat: 33.5925, lng: 130.4055
      },

      // ── Day 4 · 福岡 ──────────────────────────────────────────
      {
        id: 'day4-uminaka',
        day: 4, city: '福岡',
        name: '海の中道海浜公園',
        hours: '09:30–17:30（週一休）',
        tags: ['親子友善', '推薦'],
        notes: [
          { author: 'E', text: '超大公園，花田很美，租腳踏車繞一圈很舒服' },
          { author: 'S', text: '動物區孩子瘋了！可以摸小動物，必來！' }
        ],
        lat: 33.6402, lng: 130.4368
      },

      // ── Day 5 · 湯布院 ────────────────────────────────────────
      {
        id: 'day5-kinrin',
        day: 5, city: '湯布院',
        name: '金鱗湖',
        hours: '全天開放（清晨最美）',
        tags: ['推薦'],
        notes: [
          { author: 'E', text: '清晨霧氣升起像夢境，建議 7:00 前到，遊客少' },
          { author: 'S', text: '小孩看到大錦鯉超興奮，湖邊鴨子也很可愛' }
        ],
        lat: 33.2792, lng: 131.0558
      },
      {
        id: 'day5-yutsubo',
        day: 5, city: '湯布院',
        name: '湯の坪街道',
        hours: '10:00–18:00',
        tags: ['必買', '推薦'],
        notes: [
          { author: 'E', text: '布丁和霜淇淋超讚！各種湯布院限定土產在這裡買' },
          { author: 'S', text: '手作雜貨很多，買了好多伴手禮' }
        ],
        lat: 33.2770, lng: 131.0497
      },

      // ── Day 6 · 熊本 ──────────────────────────────────────────
      {
        id: 'day6-castle',
        day: 6, city: '熊本',
        name: '熊本城',
        hours: '09:00–17:00（最終入場 16:30）',
        tags: ['推薦'],
        notes: [
          { author: 'E', text: '整修中但外觀仍壯觀，天守閣重建完成可進入參觀' },
          { author: 'S', text: '歷史感很強，孩子對武士甲冑很感興趣' }
        ],
        lat: 32.8063, lng: 130.7058
      },
      {
        id: 'day6-kamidori',
        day: 6, city: '熊本',
        name: '上通アーケード',
        hours: '10:00–20:00',
        tags: ['必買'],
        notes: [
          { author: 'E', text: '熊本熊周邊在這裡買最齊全！官方 shop 就在這' },
          { author: 'S', text: '在地品牌很多，比東京便宜，買了一堆' }
        ],
        lat: 32.8038, lng: 130.7108
      },

      // ── Day 7 · 熊本 ──────────────────────────────────────────
      {
        id: 'day7-suizenji',
        day: 7, city: '熊本',
        name: '水前寺成趣園',
        hours: '08:30–17:00',
        tags: ['推薦', '親子友善'],
        notes: [
          { author: 'E', text: '縮小版富士山日式庭園，安靜優美，最後一天輕鬆散步' },
          { author: 'S', text: '小孩在庭園追鴿子玩得很開心，門票便宜值得來' }
        ],
        lat: 32.7990, lng: 130.7325
      }
    ];
```

- [ ] **Step 2: Verify data is accessible in browser console**

Open `index.html`, open DevTools Console, type `LANDMARKS.length`.
Expected output: `12`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add 12-landmark data array for all 7 days"
```

---

## Task 4: Render Map Pins from Data

**Files:**
- Modify: `index.html` (update `initMap` function, add `renderPins`)

- [ ] **Step 1: Replace `initMap` body and add `renderPins` function**

Replace the existing `initMap` function and add `renderPins` directly after it:

```javascript
    function initMap() {
      const map = new google.maps.Map(document.getElementById('map'), {
        center: { lat: 33.2, lng: 130.9 },
        zoom: 8,
        disableDefaultUI: true,
        zoomControl: true,
        mapTypeControl: false,
        styles: [
          { featureType: 'poi', elementType: 'labels', stylers: [{ visibility: 'off' }] },
          { featureType: 'transit', elementType: 'labels', stylers: [{ visibility: 'off' }] }
        ]
      });
      window._map = map;
      renderPins(map);
    }

    function renderPins(map) {
      LANDMARKS.forEach(lm => {
        const marker = new google.maps.Marker({
          position: { lat: lm.lat, lng: lm.lng },
          map,
          title: lm.name,
          icon: {
            path: google.maps.SymbolPath.CIRCLE,
            scale: 8,
            fillColor: '#748573',
            fillOpacity: 1,
            strokeColor: '#ffffff',
            strokeWeight: 2
          }
        });

        const infoWindow = new google.maps.InfoWindow({
          content: `<div style="font-family:-apple-system,sans-serif;font-size:13px;font-weight:500;padding:2px 4px">${lm.name}</div>`
        });

        marker.addListener('click', () => {
          infoWindow.open(map, marker);
          const card = document.getElementById(`card-${lm.id}`);
          if (card) {
            card.scrollIntoView({ behavior: 'smooth', block: 'center' });
            card.classList.add('card-highlight');
            setTimeout(() => card.classList.remove('card-highlight'), 1500);
          }
        });
      });
    }
```

- [ ] **Step 2: Add card-highlight CSS**

Inside `<style>`, add:

```css
    .card-highlight {
      outline: 2px solid var(--green);
      outline-offset: 2px;
      transition: outline 0.3s;
    }
```

- [ ] **Step 3: Verify pins appear on map (requires valid API key)**

Open `index.html` with a valid API key.
Expected: 12 sage-green circular pins appear across Fukuoka, Dazaifu, Yufuin, and Kumamoto regions.
Without a key: "For development purposes only" watermark; pins still render structurally correct.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: render map pins from landmark data with click-to-scroll"
```

---

## Task 5: Dynamic Timeline Cards Rendering

**Files:**
- Modify: `index.html` (add CSS for cards, add JS rendering functions)

- [ ] **Step 1: Add card CSS**

Inside `<style>`, add:

```css
    /* ── Day Header ──────────────────────── */
    .day-header {
      display: flex;
      align-items: baseline;
      gap: 8px;
      padding: 20px 0 10px;
      position: sticky;
      top: 49px;       /* height of nav */
      background: var(--white);
      z-index: 100;
      border-bottom: 1px solid var(--gray-border);
      margin-bottom: 12px;
    }
    .day-label {
      font-size: 13px;
      font-weight: 600;
      color: var(--green);
      text-transform: uppercase;
      letter-spacing: 0.05em;
    }
    .day-city {
      font-size: 16px;
      font-weight: 600;
      color: #1a1a1a;
    }

    /* ── Card ────────────────────────────── */
    .card {
      background: var(--white);
      border: 1px solid var(--gray-border);
      border-radius: var(--card-radius);
      box-shadow: var(--card-shadow);
      overflow: hidden;
      margin-bottom: 12px;
    }

    /* ── Card Photo ──────────────────────── */
    .card-photo {
      height: 180px;
      background: var(--gray-bg);
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      overflow: hidden;
      position: relative;
    }
    .card-photo img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
    .photo-placeholder {
      font-size: 13px;
      color: var(--gray-text);
    }

    /* ── Card Body ───────────────────────── */
    .card-body {
      padding: 12px 14px 14px;
    }
    .card-name {
      font-size: 15px;
      font-weight: 500;
      margin-bottom: 3px;
    }
    .card-hours {
      font-size: 12px;
      color: var(--gray-text);
      margin-bottom: 8px;
    }

    /* ── Tags ────────────────────────────── */
    .card-tags {
      display: flex;
      flex-wrap: wrap;
      gap: 6px;
      margin-bottom: 10px;
    }
    .tag {
      font-size: 11px;
      padding: 2px 7px;
      border-radius: 20px;
      border: 0.5px solid var(--gray-border);
      color: var(--gray-text);
    }
    .tag-green {
      border-color: var(--green);
      color: var(--green);
    }

    /* ── Notes ───────────────────────────── */
    .card-notes {
      display: flex;
      flex-direction: column;
      gap: 8px;
      border-left: 1.5px solid var(--green);
      padding-left: 10px;
    }
    .note {
      display: flex;
      align-items: flex-start;
      gap: 8px;
    }
    .note-avatar {
      width: 20px;
      height: 20px;
      border-radius: 50%;
      background: var(--gray-bg);
      color: var(--green);
      font-size: 10px;
      font-weight: 600;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-shrink: 0;
      margin-top: 1px;
    }
    .note-bubble {
      font-size: 12px;
      line-height: 1.5;
      color: #444;
      background: var(--gray-bg);
      border-radius: 8px;
      padding: 5px 9px;
    }

    /* ── Connector between cards ─────────── */
    .card-connector {
      width: 1px;
      height: 12px;
      background: var(--gray-border);
      margin: 0 auto;
    }
```

- [ ] **Step 2: Add JS rendering functions**

Inside the `<script>` block, after `renderPins`, add:

```javascript
    function renderTimeline() {
      const timeline = document.getElementById('timeline');
      timeline.innerHTML = '';

      // Group landmarks by day, preserving order
      const byDay = {};
      LANDMARKS.forEach(lm => {
        if (!byDay[lm.day]) byDay[lm.day] = { city: lm.city, items: [] };
        byDay[lm.day].items.push(lm);
      });

      Object.keys(byDay)
        .map(Number)
        .sort((a, b) => a - b)
        .forEach(day => {
          const { city, items } = byDay[day];

          // Day header
          const header = document.createElement('div');
          header.className = 'day-header';
          header.id = `day-${day}`;
          header.innerHTML = `<span class="day-label">Day ${day}</span><span class="day-city">${city}</span>`;
          timeline.appendChild(header);

          // Cards with connectors
          items.forEach((lm, idx) => {
            timeline.appendChild(createCard(lm));
            if (idx < items.length - 1) {
              const connector = document.createElement('div');
              connector.className = 'card-connector';
              timeline.appendChild(connector);
            }
          });
        });
    }

    function createCard(lm) {
      const card = document.createElement('div');
      card.className = 'card';
      card.id = `card-${lm.id}`;

      const savedPhoto = localStorage.getItem(`photo-${lm.id}`);
      const photoContent = savedPhoto
        ? `<img src="${savedPhoto}" alt="${lm.name}">`
        : `<span class="photo-placeholder">點擊上傳照片</span>`;

      const tagsHTML = lm.tags.map(t =>
        `<span class="tag ${t === '推薦' ? 'tag-green' : ''}">${t}</span>`
      ).join('');

      const notesHTML = lm.notes.map(n =>
        `<div class="note">
          <div class="note-avatar">${n.author}</div>
          <div class="note-bubble">${n.text}</div>
        </div>`
      ).join('');

      card.innerHTML = `
        <div class="card-photo" onclick="triggerUpload('${lm.id}')">
          ${photoContent}
          <input type="file" accept="image/*" id="file-${lm.id}"
                 style="display:none"
                 onchange="handlePhoto('${lm.id}', this)">
        </div>
        <div class="card-body">
          <h3 class="card-name">${lm.name}</h3>
          <p class="card-hours">${lm.hours}</p>
          <div class="card-tags">${tagsHTML}</div>
          <div class="card-notes">${notesHTML}</div>
        </div>`;
      return card;
    }
```

- [ ] **Step 3: Call `renderTimeline()` on page load**

Inside `<script>`, add at the very end (after all function definitions):

```javascript
    document.addEventListener('DOMContentLoaded', renderTimeline);
```

- [ ] **Step 4: Verify timeline renders**

Open `index.html` in browser.
Expected: 7 day headers (Day 1 福岡 … Day 7 熊本) with cards underneath, gray photo placeholders, outline tags, green-bordered notes with E/S avatars, thin connectors between cards.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: render full 7-day timeline from data with cards, tags, and notes"
```

---

## Task 6: Photo Upload + localStorage Persistence

**Files:**
- Modify: `index.html` (add JS photo functions)

- [ ] **Step 1: Add photo upload functions**

Inside `<script>`, after `createCard`, add:

```javascript
    function triggerUpload(id) {
      document.getElementById(`file-${id}`).click();
    }

    function handlePhoto(id, input) {
      const file = input.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = e => {
        const dataUrl = e.target.result;
        localStorage.setItem(`photo-${id}`, dataUrl);

        // Update the photo div in-place without re-rendering the whole card
        const photoDiv = document.querySelector(`#card-${id} .card-photo`);
        photoDiv.innerHTML = `
          <img src="${dataUrl}" alt="">
          <input type="file" accept="image/*" id="file-${id}"
                 style="display:none"
                 onchange="handlePhoto('${id}', this)">`;
      };
      reader.readAsDataURL(file);
    }
```

- [ ] **Step 2: Verify photo upload works**

Open `index.html`, click on any card's photo placeholder.
Expected: file picker opens. Select any image. The card's photo area updates immediately with the selected image.
Reload the page. Expected: the photo is still there (loaded from localStorage).

- [ ] **Step 3: Verify multiple cards work independently**

Upload different photos to two different cards. Reload.
Expected: each card shows its own photo.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: photo upload via FileReader with localStorage persistence"
```

---

## Task 7: Desktop Responsive Layout (≥ 768px)

**Files:**
- Modify: `index.html` (add CSS media query)

- [ ] **Step 1: Add desktop layout CSS**

Inside `<style>`, append at the end:

```css
    /* ── Desktop Layout (≥ 768px) ────────── */
    @media (min-width: 768px) {
      body {
        height: 100vh;
        display: flex;
        flex-direction: column;
        overflow: hidden;
      }

      .main-layout {
        flex: 1;
        display: grid;
        grid-template-columns: 1fr 1fr;
        overflow: hidden;
        min-height: 0;
      }

      .map-container {
        height: 100%;
        position: sticky;
        top: 0;
      }

      .timeline {
        overflow-y: auto;
        height: 100%;
        padding: 0 24px 80px;
      }

      .day-header {
        top: 0;       /* no nav offset needed; nav is outside scroll area */
      }
    }
```

- [ ] **Step 2: Verify desktop layout**

Open `index.html` and resize browser window to ≥ 768px wide.
Expected: left half shows the map (full height), right half shows the scrollable timeline. The map stays fixed as you scroll the right panel.

Resize to < 768px (or open DevTools mobile mode).
Expected: map appears at top 40vh, timeline scrolls below it — single-column layout.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: responsive desktop layout with fixed map and scrollable timeline"
```

---

## Task 8: Final Polish — Hover States + Meta Tags

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add hover states and photo overlay hint**

Inside `<style>`, add:

```css
    /* ── Hover states ────────────────────── */
    .card-photo:hover .photo-placeholder {
      color: var(--green);
    }
    .card-photo:hover::after {
      content: '📷';
      position: absolute;
      bottom: 8px;
      right: 10px;
      font-size: 16px;
      opacity: 0.7;
    }
    .tag:hover {
      opacity: 0.75;
    }
```

- [ ] **Step 2: Add OG meta tags for sharing**

Inside `<head>`, after `<title>`, add:

```html
  <meta name="description" content="Edna & 小姑 的九州家族旅行紀錄 — 福岡・太宰府・湯布院・熊本">
  <meta property="og:title" content="九州家族旅行 2025">
  <meta property="og:description" content="7天行程完整紀錄，附地圖、照片與旅遊筆記">
  <meta property="og:type" content="website">
```

- [ ] **Step 3: Final visual check**

Open `index.html`. Verify:
- [ ] Nav "九州家族旅行 2025" is visible and sticky
- [ ] Map area renders (gray placeholder or actual map)
- [ ] All 7 day headers visible while scrolling
- [ ] Cards have photo placeholder, landmark name, hours, tags, notes
- [ ] Tags: 推薦 in green outline, others in gray outline
- [ ] Notes: green left bar, gray-bg bubbles, E/S circle avatars
- [ ] Clicking photo placeholder opens file picker
- [ ] Desktop ≥768px: side-by-side map + timeline
- [ ] Mobile <768px: stacked map + timeline

- [ ] **Step 4: Final commit**

```bash
git add index.html
git commit -m "feat: add hover states, OG meta tags — site complete"
```

---

## Deployment

After all tasks complete, deploy to Netlify Drop:

1. Go to [https://app.netlify.com/drop](https://app.netlify.com/drop)
2. Drag and drop `index.html` onto the page
3. Netlify generates a public URL instantly
4. Share via IG Story or LINE group

> **Note on Google Maps API Key:** Before sharing publicly, restrict the API key to your Netlify domain in Google Cloud Console → APIs & Services → Credentials → HTTP referrers.

---

## Content Fill-in Checklist (Edna & 小姑)

After deployment, Edna and 小姑 fill in directly in the JS data array in `index.html`:
- [ ] Verify/update each landmark's `hours` field
- [ ] Update each `notes[].text` with real personal comments
- [ ] Update `tags` arrays (add/remove 推薦/必買/親子友善 per landmark)
- [ ] Upload photos by clicking each card's photo area
- [ ] Optionally add more landmarks by adding entries to `LANDMARKS` array

---

## Self-Review Against Spec

| Spec Requirement | Covered |
|-----------------|---------|
| 每天地標標示（地點、營業時間） | ✅ Task 3 data + Task 5 card rendering |
| 行程路線地圖 | ✅ Task 2 + Task 4 map pins |
| 兩人旅遊筆記（E & S） | ✅ Task 5 notes with avatars |
| Mobile-first 40vh 地圖 | ✅ Task 2 CSS + Task 7 media query |
| Desktop 左固定地圖右捲動 | ✅ Task 7 |
| 圖釘點擊 → 捲動到卡片 | ✅ Task 4 `renderPins` click listener |
| 卡片：照片上傳區 | ✅ Task 6 |
| 卡片：Outline tag 樣式 | ✅ Task 5 CSS |
| 推薦 tag 綠色，其他灰色 | ✅ Task 5 `tag-green` class |
| 筆記左側 #748573 線 | ✅ Task 5 `.card-notes` border-left |
| E/S 圓形頭像 淺灰底+綠字 | ✅ Task 5 `.note-avatar` styles |
| Sticky day header | ✅ Task 5 CSS `position: sticky` |
| localStorage 照片持久化 | ✅ Task 6 |
| 單一 index.html 靜態部署 | ✅ No build step required |
| System font stack | ✅ Task 1 CSS variables |
| 純白背景 + 鼠尾草綠重點色 | ✅ Task 1 design tokens |
