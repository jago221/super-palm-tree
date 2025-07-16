# <!DOCTYPE html><html>
<head>
  <title>Secret Chat - Global</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
  <style>
    :root {
      --bg-color: #1a1a2e;
      --text-color: #fff;
      --box-color: #16213e;
    }
    body.light {
      --bg-color: #f1f1f1;
      --text-color: #000;
      --box-color: #fff;
    }
    body {
      font-family: Arial, sans-serif;
      background: var(--bg-color);
      color: var(--text-color);
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      transition: background 0.3s, color 0.3s;
    }
    header {
      background: #0f3460;
      width: 100%;
      padding: 15px;
      text-align: center;
      font-size: 1.5em;
      color: white;
    }
    #container {
      width: 95%;
      max-width: 500px;
      margin-top: 20px;
    }
    input, button {
      border: none;
      border-radius: 5px;
    }
    #pairID, #username, #pin {
      width: 100%;
      padding: 10px;
      margin: 5px 0;
      font-size: 1em;
    }
    #chatBox {
      height: 400px;
      overflow-y: scroll;
      background: var(--box-color);
      padding: 10px;
      border: 2px solid #0f3460;
      border-radius: 8px;
      margin-bottom: 10px;
    }
    #chatBox div {
      margin: 5px 0;
    }
    #messageInput {
      display: flex;
      gap: 5px;
    }
    #message {
      flex: 1;
      padding: 10px;
      font-size: 1em;
    }
    #sendBtn, #clearChatBtn, #toggleThemeBtn, #exportBtn {
      padding: 10px 15px;
      background: #e94560;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }
    #sendBtn:hover, #clearChatBtn:hover, #toggleThemeBtn:hover, #exportBtn:hover {
      background: #ff2e4c;
    }
    .timestamp {
      font-size: 0.7em;
      color: gray;
    }
  </style>
</head>
<body>
  <header>🔐 Secret Chat - Worldwide</header>
  <div id="container">
    <input id="pairID" placeholder="Enter your shared secret code..." />
    <input id="pin" placeholder="Enter PIN (Optional)" />
    <input id="username" placeholder="Enter your code name..." /><div id="chatBox"></div>

<div id="messageInput">
  <input id="message" placeholder="Type your secret message...">
  <button id="sendBtn">Send</button>
  <button id="clearChatBtn">🧹</button>
  <button id="exportBtn">📦</button>
  <button id="toggleThemeBtn">🌓</button>
</div>

  </div>  <audio id="ping">
    <source src="https://actions.google.com/sounds/v1/alarms/beep_short.ogg" type="audio/ogg">
  </audio>  <script>
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT.firebaseapp.com",
      databaseURL: "https://YOUR_PROJECT.firebaseio.com",
      projectId: "YOUR_PROJECT",
      storageBucket: "YOUR_PROJECT.appspot.com",
      messagingSenderId: "YOUR_ID",
      appId: "YOUR_APP_ID"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    const pairIDInput = document.getElementById('pairID');
    const pinInput = document.getElementById('pin');
    const usernameInput = document.getElementById('username');
    const messageInput = document.getElementById('message');
    const chatBox = document.getElementById('chatBox');
    const sendBtn = document.getElementById('sendBtn');
    const clearBtn = document.getElementById('clearChatBtn');
    const toggleThemeBtn = document.getElementById('toggleThemeBtn');
    const exportBtn = document.getElementById('exportBtn');
    const ping = document.getElementById('ping');

    let currentPairID = "";
    let pinCode = "";
    let chatHistory = [];

    toggleThemeBtn.onclick = () => {
      document.body.classList.toggle('light');
    };

    sendBtn.onclick = () => {
      const pairID = pairIDInput.value.trim();
      const pin = pinInput.value.trim();
      const username = usernameInput.value.trim();
      const message = messageInput.value.trim();
      if (!pairID || !username || !message) return;
      const time = new Date().toLocaleString();
      db.ref('chats/' + pairID).push({ user: username, text: message, time, pin });
      messageInput.value = '';
    };

    clearBtn.onclick = () => {
      if (confirm("Clear chat history for this code?")) {
        const pairID = pairIDInput.value.trim();
        if (pairID) db.ref('chats/' + pairID).remove();
        chatBox.innerHTML = "";
        chatHistory = [];
      }
    };

    exportBtn.onclick = () => {
      if (chatHistory.length === 0) return alert("Nothing to export.");
      const blob = new Blob([chatHistory.join("\n")], { type: 'text/plain' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = "secret-chat-log.txt";
      a.click();
      URL.revokeObjectURL(url);
    };

    pairIDInput.onchange = () => {
      const pairID = pairIDInput.value.trim();
      const pin = pinInput.value.trim();
      if (!pairID) return;
      chatBox.innerHTML = "";
      chatHistory = [];
      if (currentPairID) db.ref('chats/' + currentPairID).off();
      currentPairID = pairID;
      db.ref('chats/' + pairID).on('child_added', (snapshot) => {
        const msg = snapshot.val();
        if (pin && msg.pin && msg.pin !== pin) return; // Skip if PIN doesn't match
        const div = document.createElement('div');
        div.innerHTML = `<b>${msg.user}</b> <span class="timestamp">${msg.time}</span><br>${msg.text}`;
        chatBox.appendChild(div);
        chatBox.scrollTop = chatBox.scrollHeight;
        ping.play();
        chatHistory.push(`${msg.time} - ${msg.user}: ${msg.text}`);
      });
    };
  </script></body>
</html>-palm-tree