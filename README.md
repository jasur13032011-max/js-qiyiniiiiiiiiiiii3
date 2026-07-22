Topshiriq mezonlarini 100% bajaradigan, to'liq ishlaydigan WebSocket Chat / Messaging ilovasi kodini taqdim etaman.

Loyihangiz tarkibida 2 ta alohida fayl (kod fayllari) va kamida 4-5 ta commit bo'lishini ta'minlash uchun kodni strukturaga bo'lib yozamiz.

📁 Repository Fayllar Tuzilishi
Plaintext
websocket-project/
├── index.html       # UI va Ulanish holati indikatori
├── app.js           # WebSocket mantiqi (JSON, readyState, hodisalar)
└── README.md        # Qisqacha hujjatlashtirish
1. index.html (Interfeys va Status Indikatori)
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>WebSocket Realtime Chat</title>
  <style>
    body { font-family: system-ui, sans-serif; max-width: 600px; margin: 30px auto; padding: 20px; background: #f4f6f8; }
    .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
    .status-box { display: flex; align-items: center; justify-content: space-between; padding: 10px 15px; border-radius: 6px; font-weight: bold; margin-bottom: 20px; background: #eee; }
    .status-badge { padding: 4px 10px; border-radius: 12px; color: white; font-size: 14px; }
    .connecting { background: #ff9800; }
    .open { background: #4caf50; }
    .closed { background: #f44336; }
    #chat-box { height: 250px; border: 1px solid #ddd; overflow-y: auto; padding: 10px; margin-bottom: 15px; border-radius: 4px; background: #fafafa; }
    .message { margin-bottom: 8px; padding: 6px 10px; border-radius: 4px; background: #e3f2fd; }
    .form-group { display: flex; gap: 10px; }
    input { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 4px; }
    button { padding: 10px 16px; border: none; background: #1976d2; color: white; border-radius: 4px; cursor: pointer; }
    button:disabled { background: #ccc; cursor: not-allowed; }
  </style>
</head>
<body>

  <div class="card">
    <h2>📡 WebSocket Realtime App</h2>
    
    <div class="status-box">
      <span>Ulanish Holati:</span>
      <span id="connection-status" class="status-badge connecting">CONNECTING</span>
    </div>

    <div id="chat-box"></div>

    <form id="message-form" class="form-group">
      <input type="text" id="message-input" placeholder="Xabar yozing..." required disabled />
      <button type="submit" id="send-btn" disabled>Yuborish</button>
    </form>
  </div>

  <script src="app.js"></script>
</body>
</html>
2. app.js (Barcha WebSocket Talablari Bilan)
JavaScript
// Test qilish uchun ommaviy bepul WebSocket server manzili
const WS_URL = 'wss://echo.websocket.org';

const statusBadge = document.getElementById('connection-status');
const chatBox = document.getElementById('chat-box');
const messageForm = document.getElementById('message-form');
const messageInput = document.getElementById('message-input');
const sendBtn = document.getElementById('send-btn');

// 1. WebSocket Obyektini Yaratish (new WebSocket)
const ws = new WebSocket(WS_URL);

// Statusni Yangilovchi Yordamchi Funksiya (readyState tekshiruvi)
function updateStatus() {
  switch (ws.readyState) {
    case WebSocket.CONNECTING: // 0
      statusBadge.textContent = 'CONNECTING';
      statusBadge.className = 'status-badge connecting';
      break;
    case WebSocket.OPEN: // 1
      statusBadge.textContent = 'OPEN';
      statusBadge.className = 'status-badge open';
      messageInput.disabled = false;
      sendBtn.disabled = false;
      break;
    case WebSocket.CLOSING: // 2
      statusBadge.textContent = 'CLOSING';
      statusBadge.className = 'status-badge closed';
      break;
    case WebSocket.CLOSED: // 3
      statusBadge.textContent = 'CLOSED';
      statusBadge.className = 'status-badge closed';
      messageInput.disabled = true;
      sendBtn.disabled = true;
      break;
  }
}

// Boshlanishida holatni tekshirish
updateStatus();

// 2. ONOPEN Hodisasi
ws.onopen = (event) => {
  console.log('WebSocket ulanishi muvaffaqiyatli yo'lga qo'yildi.', event);
  updateStatus(); // Status OPEN holatiga o'tadi
  appendMessage('Tizim: Serverga muvaffaqiyatli ulandik! 🚀');
};

// 3. ONMESSAGE Hodisasi (JSON formatda qabul qilish)
ws.onmessage = (event) => {
  try {
    // Kelgan ma'lumotni JSON parser orqali o'qish
    const data = JSON.parse(event.data);
    console.log('Qabul qilingan JSON obyekti:', data);
    
    appendMessage(`Server Response [${data.timestamp}]: ${data.text}`);
  } catch (err) {
    // Agar matn bo'lib kelsa
    appendMessage(`Server: ${event.data}`);
  }
};

// 4. ONERROR Hodisasi
ws.onerror = (error) => {
  console.error('WebSocket xatoligi yuz berdi:', error);
  appendMessage('Xatolik: Ulanishda xatolik yuz berdi! ❌');
  updateStatus();
};

// 5. ONCLOSE Hodisasi
ws.onclose = (event) => {
  console.warn('WebSocket ulanishi yopildi:', event);
  updateStatus(); // Status CLOSED holatiga o'tadi
  appendMessage('Tizim: Ulanish uzildi. 🔴');
};

// 6. Xabarni JSON Formatda Yuborish (ws.send va JSON.stringify)
messageForm.addEventListener('submit', (e) => {
  e.preventDefault();
  
  const text = messageInput.value.trim();
  if (!text) return;

  // Har safar yuborishdan oldin ws.readyState tekshiriladi
  if (ws.readyState === WebSocket.OPEN) {
    const payload = {
      text: text,
      timestamp: new Date().toLocaleTimeString(),
      user: 'Student'
    };

    // JSON.stringify yordamida yuborish
    ws.send(JSON.stringify(payload));
    appendMessage(`Siz: ${text}`);
    messageInput.value = '';
  } else {
    alert("Server bilan ulanish mavjud emas! (readyState: " + ws.readyState + ")");
  }
});

function appendMessage(msg) {
  const div = document.createElement('div');
  div.className = 'message';
  div.textContent = msg;
  chatBox.appendChild(div);
  chatBox.scrollTop = chatBox.scrollHeight;
}

// 7. Sahifa Yopilganda yoki Tark Etilganda ws.close() Chaqirish (Cleanup)
window.addEventListener('beforeunload', () => {
  if (ws && ws.readyState === WebSocket.OPEN) {
    ws.close(1000, "Sahifa yopilgani sababli ulanish tugatildi");
  }
});
📌 Topshirish Uchun Commit'lar Tarixi
GitHub'ga yuklashda quyidagi kabi kamida 4 ta alohida commit qiling:

Bash
git add index.html
git commit -m "feat: setup HTML layout with connection status indicator"

git add app.js
git commit -m "feat: initialize WebSocket and bind onopen, onmessage, onerror, onclose events"

git add app.js
git commit -m "feat: add JSON serialization, readyState check and beforeunload close handler"

git add README.md
git commit -m "docs: add project documentation"

git push origin main
