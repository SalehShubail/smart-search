<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>البحث عن الحسابات الاجتماعية</title>
  <style>
    body { font-family: Tahoma, Arial, sans-serif; margin:0; padding:0; background:#f9f9f9;}
    header { background:#3a86ff; color:#fff; padding:15px; text-align:center;}
    .container { max-width:720px; margin:30px auto; padding:20px; background:#fff; border-radius:12px; box-shadow:0 4px 10px rgba(0,0,0,.08);}
    .search-types { display:flex; gap:10px; margin-bottom:20px; flex-wrap:wrap;}
    .search-types button { flex:1; padding:10px; border:none; border-radius:8px; cursor:pointer; background:#eee;}
    .search-types button.active { background:#3a86ff; color:#fff;}
    input { width:100%; padding:12px; border:1px solid #ccc; border-radius:10px; margin-bottom:10px;}
    button.submit-btn { padding:12px 18px; border:none; border-radius:10px; background:#3a86ff; color:#fff; cursor:pointer; width:100%;}
    button.submit-btn:hover { background:#265b9b;}
    .status { margin-top:14px; font-size:14px;}
    .error { color:#c02626;}
    .ok { color:#0a7b32;}
    pre { margin-top:16px; background:#111; color:#0f0; padding:12px; border-radius:10px; overflow-x:auto; white-space:pre-wrap; direction:ltr;}
  </style>
</head>
<body>
  <header>
    <h1>🔍 البحث عن الحسابات الاجتماعية</h1>
  </header>

  <div class="container">
    <div class="search-types">
      <button class="active" data-type="name">بحث بالاسم</button>
      <button data-type="username">بحث باليوزر</button>
      <button data-type="email">بحث بالإيميل</button>
      <button data-type="phone">بحث برقم الهاتف</button>
    </div>

    <form id="searchForm">
      <input type="text" id="queryInput" placeholder="أدخل البيانات هنا..." required>
      <button type="submit" class="submit-btn">ابحث الآن</button>
    </form>

    <div id="status" class="status"></div>
    <pre id="out" hidden></pre>
  </div>

  <script>
    const API_KEY = "rnd_wJwceXXi4OrcPMycmK1QyWShsmIA"; // مفتاحك الجديد
    const API_HOST = "social-links-search.p.rapidapi.com";

    const qEl = document.getElementById("queryInput");
    const form = document.getElementById("searchForm");
    const statusEl = document.getElementById("status");
    const outEl = document.getElementById("out");
    const searchButtons = document.querySelectorAll(".search-types button");

    let activeType = "name";
    let currentController = null;

    // تبديل نوع البحث
    searchButtons.forEach(btn => {
      btn.addEventListener("click", () => {
        searchButtons.forEach(b => b.classList.remove("active"));
        btn.classList.add("active");
        activeType = btn.dataset.type;
        qEl.placeholder = "أدخل " + btn.textContent + "...";
      });
    });

    form.addEventListener("submit", async (e) => {
      e.preventDefault();
      const query = qEl.value.trim();
      outEl.hidden = true;
      outEl.textContent = "";

      if (!query) {
        statusEl.innerHTML = "<span class='error'>اكتب قيمة للبحث.</span>";
        return;
      }

      if (currentController) currentController.abort();
      currentController = new AbortController();

      statusEl.textContent = "⏳ جاري البحث...";

      try {
        // URL ديناميكي حسب نوع البحث
        const url = `https://${API_HOST}/search?query=${encodeURIComponent(query)}&type=${encodeURIComponent(activeType)}`;

        const res = await fetch(url, {
          method: "GET",
          headers: {
            "x-rapidapi-key": API_KEY,
            "x-rapidapi-host": API_HOST
          },
          signal: currentController.signal
        });

        if (!res.ok) {
          const errText = await res.text();
          statusEl.innerHTML = `<span class='error'>خطأ HTTP ${res.status}: ${escapeHtml(errText)}</span>`;
          return;
        }

        const data = await res.json();
        statusEl.innerHTML = "<span class='ok'>✅ تم — النتائج بالأسفل.</span>";
        outEl.hidden = false;
        outEl.textContent = JSON.stringify(data, null, 2);

      } catch (e) {
        if (e.name === "AbortError") return;
        statusEl.innerHTML = `<span class='error'>فشل الطلب: ${escapeHtml(e.message)}</span>`;
      }
    });

    function escapeHtml(str) {
      return String(str)
        .replaceAll("&", "&amp;")
        .replaceAll("<", "&lt;")
        .replaceAll(">", "&gt;")
        .replaceAll('"', "&quot;")
        .replaceAll("'", "&#039;");
    }
  </script>
</body>
</html>
