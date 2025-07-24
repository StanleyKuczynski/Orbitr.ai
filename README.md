# Orbitr.ai
An offline secure Bluetooth messaging app powered by bluetooth.
/*
Orbitr Starter Code
Cross-platform encrypted Bluetooth chat using Electron and Node.js
*/

// === main.js ===
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
      contextIsolation: false
    }
  });

  win.loadFile('index.html');
}

app.whenReady().then(createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});


// === index.html ===
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Orbitr</title>
</head>
<body>
  <h1>🔒 Orbitr</h1>
  <input type="text" id="msgInput" placeholder="Type a message" />
  <button onclick="sendMessage()">Send</button>
  <div id="log"></div>

  <script src="renderer.js"></script>
</body>
</html>


// === renderer.js ===
const noble = require('@abandonware/noble');
const crypto = require('crypto');

function log(msg) {
  const logDiv = document.getElementById('log');
  logDiv.innerHTML += `<p>${msg}</p>`;
}

// Generate encryption key pair (for demo only, real app would store and reuse securely)
const { publicKey, privateKey } = crypto.generateKeyPairSync('rsa', {
  modulusLength: 2048,
});

function encryptMessage(msg, pubKey) {
  return crypto.publicEncrypt(pubKey, Buffer.from(msg));
}

function decryptMessage(ciphertext) {
  return crypto.privateDecrypt(privateKey, ciphertext).toString();
}

// Scan for devices
noble.on('stateChange', state => {
  if (state === 'poweredOn') {
    log('Bluetooth on. Scanning for devices...');
    noble.startScanning();
  } else {
    noble.stopScanning();
    log('Bluetooth off or unavailable.');
  }
});

noble.on('discover', peripheral => {
  log(`Found: ${peripheral.advertisement.localName || peripheral.id}`);
});

// Placeholder message sender
function sendMessage() {
  const msg = document.getElementById('msgInput').value;
  const encrypted = encryptMessage(msg, publicKey); // In real app, you'd use recipient's key
  const decrypted = decryptMessage(encrypted);

  log(`Original: ${msg}`);
  log(`Decrypted: ${decrypted}`);
}

console.log('Renderer script loaded.');
