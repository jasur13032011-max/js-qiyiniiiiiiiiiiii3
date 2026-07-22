Repozitoriyada faqat README bo'lgan — index.html va script.js fayllari yuklanmagan.

threshold qiymati 0.3 bo'lishi va 10 ta rasm kartasida lazy loading ishlashi kerak.

ResizeObserver panel o'lchamlarini sahifada real-vaqt rejimida (pixelda) ko'rsatishi hamda o'lcham 400px dan kichik bo'lsa rangini o mezonida o'zgartirishi kerak.

MutationObserver bilan element qo'shish va o'chirish tugmalari sahifada bo'lishi hamda loglar aniq ko'rinishi kerak.

Barcha observer'lar uchun disconnect() (cleanup) va unobserve() mexanizmi to'liq ishlashi kerak.

Quyida ustozning barcha talablarini 100% bajaradigan tayyor loyiha fayllari berilgan.

📁 Repository Fayllar Tuzilishi
Loyihangiz papkasida 3 ta fayl yaratasiz:

Plaintext
my-observer-project/
├── index.html
├── script.js
└── README.md
1. index.html
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>Browser Observers Demo</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background-color: #f4f6f8;
      color: #333;
    }
    section {
      background: #fff;
      padding: 20px;
      margin-bottom: 30px;
      border-radius: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    /* Resize Panel Style */
    #resize-panel {
      width: 100%;
      height: 120px;
      background-color: #4caf50; /* Odatiy yashil rang */
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 18px;
      font-weight: bold;
      resize: horizontal;
      overflow: auto;
      border-radius: 6px;
      transition: background-color 0.3s ease;
    }
    /* Intersection Cards Style */
    .image-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
      gap: 15px;
    }
    .card {
      min-height: 200px;
      background: #e0e0e0;
      border-radius: 6px;
      display: flex;
      align-items: center;
      justify-content: center;
      overflow: hidden;
    }
    .card img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      display: block;
    }
    /* Mutation DOM Container */
    #mutation-container {
      border: 2px dashed #ccc;
      padding: 15px;
      min-height: 80px;
      margin-top: 10px;
      border-radius: 6px;
    }
    .item {
      padding: 8px;
      margin: 5px 0;
      background: #2196f3;
      color: white;
      border-radius: 4px;
    }
    #mutation-log {
      background: #222;
      color: #00ff00;
      padding: 10px;
      height: 120px;
      overflow-y: auto;
      font-family: monospace;
      margin-top: 10px;
      border-radius: 6px;
    }
    .btn {
      padding: 8px 16px;
      margin-right: 10px;
      cursor: pointer;
      border: none;
      border-radius: 4px;
      color: white;
      font-weight: bold;
    }
    .btn-add { background: #4caf50; }
    .btn-remove { background: #f44336; }
    .btn-cleanup { background: #ff9800; }
  </style>
</head>
<body>

  <h1>🔍 JavaScript Observer APIs Demo</h1>

  <section>
    <h2>1. ResizeObserver</h2>
    <p>Panelni o'ng burchagidan tortib o'lchamini o'zgartiring (400px dan kichik bo'lsa rang qizilga o'zgaradi):</p>
    <div id="resize-panel">
      O'lcham: <span id="resize-text">0px x 0px</span>
    </div>
  </section>

  <section>
    <h2>2. MutationObserver</h2>
    <button class="btn btn-add" id="btn-add">Element Qo'shish</button>
    <button class="btn btn-remove" id="btn-remove">Element O'chirish</button>
    <button class="btn btn-cleanup" id="btn-disconnect-mutation">Observer Disconnect</button>

    <div id="mutation-container">
      <div class="item">Boshlang'ich Element</div>
    </div>
    <div id="mutation-log">-- DOM o'zgarishlar logi --</div>
  </section>

  <section>
    <h2>3. IntersectionObserver (Lazy Loading)</h2>
    <p>Pastga scroll qiling. Threshold 0.3 (30% ko'ringanda) rasmlar yuklanadi va unobserve() qilinadi.</p>
    <div class="image-grid" id="image-grid">
      </div>
  </section>

  <script src="script.js"></script>
</body>
</html>
2. script.js
JavaScript
document.addEventListener('DOMContentLoaded', () => {

  // ==============================================================================
  // 1. RESIZE OBSERVER IMPLEMENTATION
  // ==============================================================================
  const resizePanel = document.getElementById('resize-panel');
  const resizeText = document.getElementById('resize-text');

  const resizeObserver = new ResizeObserver((entries) => {
    for (let entry of entries) {
      // Haqiqiy pixel o'lchamlarini olish
      const width = Math.round(entry.contentRect.width);
      const height = Math.round(entry.contentRect.height);

      // Sahifada real vaqtda ko'rsatish
      resizeText.textContent = `${width}px x ${height}px`;

      // 400px dan kichik bo'lsa rangini o'zgartirish (Talab bo'yicha)
      if (width < 400) {
        resizePanel.style.backgroundColor = '#f44336'; // Qizil
      } else {
        resizePanel.style.backgroundColor = '#4caf50'; // Yashil
      }
    }
  });

  // Observe() orqali qo'shish
  resizeObserver.observe(resizePanel);


  // ==============================================================================
  // 2. MUTATION OBSERVER IMPLEMENTATION
  // ==============================================================================
  const mutationContainer = document.getElementById('mutation-container');
  const mutationLog = document.getElementById('mutation-log');
  const btnAdd = document.getElementById('btn-add');
  const btnRemove = document.getElementById('btn-remove');
  const btnDisconnectMutation = document.getElementById('btn-disconnect-mutation');

  let itemCount = 1;

  // MutationObserver callbacks
  const mutationObserver = new MutationObserver((mutationsList) => {
    for (let mutation of mutationsList) {
      if (mutation.type === 'childList') {
        // Element qo'shilganini kuzatish
        mutation.addedNodes.forEach((node) => {
          if (node.nodeType === 1) { // Element Node
            logMutation(`[QO'SHILDI]: ${node.textContent}`);
          }
        });

        // Element o'chirilganini kuzatish
        mutation.removedNodes.forEach((node) => {
          if (node.nodeType === 1) {
            logMutation(`[O'CHIRILDI]: ${node.textContent}`);
          }
        });
      }
    }
  });

  function logMutation(message) {
    const logItem = document.createElement('div');
    logItem.textContent = `${new Date().toLocaleTimeString()} - ${message}`;
    mutationLog.appendChild(logItem);
    mutationLog.scrollTop = mutationLog.scrollHeight;
  }

  // Configurations: childList: true, subtree: true (Talab bo'yicha)
  mutationObserver.observe(mutationContainer, {
    childList: true,
    subtree: true
  });

  // DOM bilan ishlash tugmalari
  btnAdd.addEventListener('click', () => {
    itemCount++;
    const newItem = document.createElement('div');
    newItem.className = 'item';
    newItem.textContent = `Yangi Element #${itemCount}`;
    mutationContainer.appendChild(newItem);
  });

  btnRemove.addEventListener('click', () => {
    if (mutationContainer.lastElementChild) {
      mutationContainer.removeChild(mutationContainer.lastElementChild);
    }
  });

  // Disconnect() qilish
  btnDisconnectMutation.addEventListener('click', () => {
    mutationObserver.disconnect();
    logMutation("⚠️ MutationObserver to'xtatildi (disconnect qilindi).");
  });


  // ==============================================================================
  // 3. INTERSECTION OBSERVER IMPLEMENTATION (LAZY LOADING)
  // ==============================================================================
  const imageGrid = document.getElementById('image-grid');

  // 10 ta rasm kartasini dinamik yaratish
  for (let i = 1; i <= 10; i++) {
    const card = document.createElement('div');
    card.className = 'card';
    
    // Asl rasm manzili data-src atributida saqlanadi
    const img = document.createElement('img');
    img.dataset.src = `https://picsum.photos/300/200?random=${i}`;
    img.alt = `Rasm #${i}`;
    
    card.appendChild(img);
    imageGrid.appendChild(card);
  }

  // Threshold 0.3 qilib belgilandi (Talab bo'yicha)
  const intersectionObserver = new IntersectionObserver((entries, observer) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        const img = entry.target.querySelector('img');
        if (img && img.dataset.src) {
          img.src = img.dataset.src; // Lazily load image
          img.removeAttribute('data-src');
        }

        // Element yuklangach kuzatishni to'xtatish (unobserve)
        observer.unobserve(entry.target);
      }
    });
  }, {
    threshold: 0.3 // 30% ko'ringanda ishlaydi
  });

  // Barcha kartalarni observe() bilan qo'shish
  const cards = document.querySelectorAll('.card');
  cards.forEach((card) => intersectionObserver.observe(card));


  // ==============================================================================
  // CLEANUP (DISCONNECT ALL ON UNLOAD)
  // ==============================================================================
  window.addEventListener('beforeunload', () => {
    resizeObserver.disconnect();
    mutationObserver.disconnect();
    intersectionObserver.disconnect();
  });

});
3. README.md
Markdown
# 🔍 Browser Observer APIs Project

Ushbu loyihada JavaScript'ning uchta asosiy Observer API'lari amalda qo'llangan:

## 🛠 Ishlatilingan Texnologiyalar
- **IntersectionObserver**: Threshold `0.3` bilan sozlangan. 10 ta rasm kartasi uchun Lazy Loading va `unobserve()` qo'llangan.
- **ResizeObserver**: Panel o'lchamlarini real vaqtda pixelda ko'rsatadi. O'lcham `400px` dan kichik bo'lganda background rangini avtomatik o'zgartiradi.
- **MutationObserver**: `childList: true, subtree: true` sozlangan. DOM'ga element qo'shilishi va o'chirilishini kuzatib logga yozadi.

## 🚀 Ishga Tushirish
`index.html` faylini istalgan zamonaviy brauzerda oching.
📌 Oxirgi Git Commit va Topshirish:
Papka ichida ushbu 3 ta fayl ham borligiga ishonch hosil qiling.

Terminalda repository'ingizga push qiling:

Bash
git add .
git commit -m "feat: implement IntersectionObserver (threshold 0.3), ResizeObserver with color change and MutationObserver DOM logs"
git push origin main
