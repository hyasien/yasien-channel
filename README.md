<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>البث المباشر | IPTV الرسمية</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/hls.js/1.4.12/hls.min.js"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" />
  <style>
    body { margin: 0; font-family: 'Cairo', sans-serif; background-color: #f0f2f5; color: #333; }
    header { background: #0f172a; color: white; padding: 2rem; text-align: center; box-shadow: 0 0 20px rgba(0,0,0,0.2); }
    .logo { font-size: 2.5rem; font-weight: 700; margin-bottom: 0.5rem; }
    .tagline { font-size: 1.1rem; color: #94a3b8; }
    nav, .controls { display: flex; justify-content: center; flex-wrap: wrap; gap: 1rem; margin: 1rem auto; max-width: 900px; }
    .btn { border: none; padding: 0.8rem 1.5rem; border-radius: 12px; font-weight: bold; cursor: pointer; transition: 0.3s; font-size: 1rem; color: white; }
    .btn:hover { opacity: 0.9; transform: scale(1.03); }
    .btn-green { background: linear-gradient(to right, #16a34a, #22c55e); }
    .btn-blue { background: linear-gradient(to right, #3b82f6, #6366f1); }
    select { padding: 0.8rem; border-radius: 8px; border: 2px solid #3b82f6; min-width: 220px; font-size: 1rem; font-weight: bold; background-color: #fff; color: #0f172a; }
    select:focus { border-color: red; outline: none; box-shadow: 0 0 5px rgba(255, 0, 0, 0.5); }
    #channel-grid { 
      display: grid; 
      gap: 1.5rem; 
      padding: 2rem; 
      justify-content: center;
      margin: 0 auto;
      max-width: 1200px;
      width: 95%;
    }
    .channel-card { 
      background: #fff; 
      border-radius: 16px; 
      overflow: hidden; 
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1); 
      position: relative; 
      aspect-ratio: 16/9; 
      width: 100%; 
      max-width: 100%; 
    }
    video { width: 100%; height: 100%; object-fit: cover; }
    footer { background: #0f172a; color: #e2e8f0; text-align: center; padding: 1.5rem; margin-top: 3rem; font-size: 0.9rem; }
    .category-wrap { display: flex; align-items: center; }
    .category-wrap label { margin-left: 0.5rem; white-space: nowrap; }
  </style>
</head>
<body>
  <header>
    <div class="logo">📡 IPTV الرسمية</div>
    <div class="tagline">مشاهدة مباشرة بجودة عالية وتصنيفات ذكية</div>
  </header>

  <section class="controls">
    <select id="channel-list">
      <option value="">-- اختر قناة --</option>
    </select>
    
    <div class="category-wrap">
      <label for="category-select"><strong>📁 التصنيف:</strong></label>
      <select id="category-select" onchange="filterByCategory()">
        <option value="all">الكل</option>
        <option value="لبنانية">قنوات لبنانية</option>
        <option value="إخبارية">قنوات إخبارية</option>
        <option value="إيرانية">قنوات إيرانية</option>
        <option value="دينية">قنوات دينية</option>
        <option value="وثائقية">قنوات وثائقية</option>
      </select>
    </div>
    
    <button class="btn btn-green" onclick="addChannel()">📺 تشغيل</button>
    <button class="btn btn-blue" onclick="clearChannels()">🔄 إعادة ضبط</button>
  </section>

  <main><div id="channel-grid"></div></main>

  <script>
    const channels = {
      "المنار": { name: "المنار", streamUrl: "https://edge.fastpublish.me/live/tracks-v2a1/mono.m3u8", category: "لبنانية", autoMute: true },
      "الصراط": { name: "الصراط", streamUrl: "https://softverse.b-cdn.net/Assirat/assiratobs/chunklist_w890757240.m3u8", category: "دينية", autoMute: true },
      "أم تي في": { name: "أم تي في", streamUrl: "https://hms.pfs.gdn/v1/broadcast/mtv/playlist.m3u8", category: "لبنانية", autoMute: true },
      "الجديد": { name: "الجديد", streamUrl: "https://samaflix.com:12103/channel7/tracks-v1a1/rewind-3600.m3u8", category: "لبنانية", autoMute: true },
      "ال بي سي": { name: "ال بي سي", streamUrl: "https://samaflix.com:12103/channel2/tracks-v4a1/rewind-3600.m3u8", category: "لبنانية", autoMute: true },
      "otv": { name: "otv", streamUrl: "https://samaflix.com:12103/channel3/tracks-v1a1/rewind-3600.m3u8", category: "لبنانية", autoMute: true },
      "nbn": { name: "nbn", streamUrl: "https://samaflix.com:12103/channel5/tracks-v1a1/rewind-3600.m3u8", category: "لبنانية", autoMute: true },
      "الميادين": { name: "الميادين", streamUrl: "https://mdnlv.cdn.octivid.com/almdn/smil:mpegts.stream.smil/playlist.m3u8", category: "إخبارية", autoMute: true },
      "الجزيرة": { name: "الجزيرة", streamUrl: "https://63b03f7689049.streamlock.net/live/_definst_/113/chunklist_w90548194.m3u8", category: "إخبارية", autoMute: true },
      "العربية": { name: "العربية", streamUrl: "https://live.alarabiya.net/alarabiapublish/alarabiya.smil/alarabiapublish/alarabiya_720p/chunks.m3u8", category: "إخبارية", autoMute: true },
      "العربي": { name: "العربي", streamUrl: "https://alarabyta.cdn.octivid.com/alaraby/smil:alaraby.stream.smil/chunklist_b1150000.m3u8", category: "إخبارية", autoMute: true },
      "سكاي نيوز": { name: "سكاي نيوز", streamUrl: "https://stream.skynewsarabia.com/ott/ott_1080.m3u8", category: "إخبارية", autoMute: true },
      "IRIB1": { name: "IRIB1", streamUrl: "https://lenz.splus.ir/PLTV/88888888/224/3221226637/1.m3u8", category: "إيرانية", autoMute: true },
      "IRIB2": { name: "IRIB2", streamUrl: "https://lenz.splus.ir/PLTV/88888888/224/3221226176/2.m3u8", category: "إيرانية", autoMute: true },
      "IRIB3": { name: "IRIB3", streamUrl: "https://lenz.splus.ir/PLTV/88888888/224/3221226868/1.m3u8", category: "إيرانية", autoMute: true },
      "IRIB4": { name: "IRIB4", streamUrl: "https://lenz.splus.ir/PLTV/88888888/224/3221226140/1.m3u8", category: "إيرانية", autoMute: true },
      "OfoghTV": { name: "OfoghTV", streamUrl: "https://lenz.splus.ir/PLTV/88888888/224/3221226143/1.m3u8", category: "إيرانية", autoMute: true },
      "IRIBNEWS": { name: "IRIBNEWS", streamUrl: "https://63b03f7689049.streamlock.net/live/_definst_/42/chunklist_w439220985.m3u8", category: "إيرانية", autoMute: true },
      "IRIB": { name: "IRIB", streamUrl: "https://lenz.splus.ir/PLTV/88888888/224/3221227020/4.m3u8", category: "إيرانية", autoMute: true },
      "Mostanad": { name: "Mostanad", streamUrl: "https://lenz.splus.ir/PLTV/88888888/224/3221226901/1.m3u8", category: "إيرانية", autoMute: true }
    };

    const grid = document.getElementById("channel-grid");
    const select = document.getElementById("channel-list");
    const categorySelect = document.getElementById("category-select");

    Object.entries(channels).forEach(([id, data]) => {
      const opt = document.createElement("option");
      opt.value = id;
      opt.textContent = data.name;
      opt.dataset.category = data.category;
      select.appendChild(opt);
    });

    function addChannel() {
      const selected = select.value;
      if (!selected) return;
      const data = channels[selected];
      const card = document.createElement("div");
      card.className = "channel-card";
      card.innerHTML = `<video controls autoplay muted></video>`;
      const video = card.querySelector("video");
      if (Hls.isSupported()) {
        const hls = new Hls({ autoStartLoad: true, startLevel: -1 });
        hls.loadSource(data.streamUrl);
        hls.attachMedia(video);
      } else if (video.canPlayType("application/vnd.apple.mpegurl")) {
        video.src = data.streamUrl;
      }
      if (data.autoMute) video.muted = true;
      grid.appendChild(card);
      updateGridLayout();
    }

   function updateGridLayout() {
  const count = grid.children.length;
  
  // تهيئة الإعدادات الافتراضية
  grid.style.gap = "1rem";
  
  if (count === 0) {
    // لا شيء للعرض
    return;
  } else if (count === 1) {
    // قناة واحدة - تأخذ مساحة كبيرة في الوسط
    grid.style.gridTemplateColumns = "1fr";
    grid.style.maxWidth = "80%";
    grid.style.margin = "0 auto";
  } else if (count === 2) {
    // قناتان - عرض أفقي بحجم كبير
    grid.style.gridTemplateColumns = "repeat(2, 1fr)";
    grid.style.maxWidth = "90%";
    grid.style.gap = "1.5rem";
  } else if (count <= 4) {
    // 3-4 قنوات - شبكة 2×2
    grid.style.gridTemplateColumns = "repeat(2, 1fr)";
    grid.style.maxWidth = "95%";
    grid.style.gap = "1.25rem";
  } else if (count <= 6) {
    // 5-6 قنوات - شبكة 3×2
    grid.style.gridTemplateColumns = "repeat(3, 1fr)";
    grid.style.maxWidth = "95%";
    grid.style.gap = "1rem";
  } else if (count <= 9) {
    // 7-9 قنوات - شبكة 3×3
    grid.style.gridTemplateColumns = "repeat(3, 1fr)";
    grid.style.maxWidth = "95%";
    grid.style.gap = "0.75rem";
  } else {
    // 10+ قنوات - شبكة 4×? متكيفة
    grid.style.gridTemplateColumns = "repeat(4, 1fr)";
    grid.style.maxWidth = "100%";
    grid.style.gap = "0.5rem";
  }
  
  // تعديل حجم القنوات وفقاً للعدد
  const cards = grid.querySelectorAll('.channel-card');
  cards.forEach(card => {
    if (count <= 2) {
      card.style.minHeight = "400px";
    } else if (count <= 4) {
      card.style.minHeight = "350px";
    } else if (count <= 9) {
      card.style.minHeight = "250px";
    } else {
      card.style.minHeight = "200px";
    }
  });
}

    function filterByCategory() {
      const selected = categorySelect.value;
      const options = select.querySelectorAll("option");
      options.forEach(opt => {
        if (!opt.value) return;
        const cat = opt.dataset.category;
        opt.style.display = selected === "all" || cat === selected ? "block" : "none";
      });
      select.value = "";
    }

    function clearChannels() {
      grid.innerHTML = "";
    }
  </script>
</body>
</html>
