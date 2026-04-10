🧠 Core Capabilities
Feature	How It Works
Local AI Conversation	Uses the 77M‑parameter LaMini‑Flan‑T5 model via Transformers.js to generate responses directly in your browser.
Automatic Tool Selection	An AI‑powered router classifies every query as chat, code, or web and chooses the best handler.
Manual Tool Override	Buttons (💬 CHAT, 📟 CODE, 🌐 WEB, 🧠 MEMORY) let you force a specific tool for the next message.
Web Search + Auto‑Learn	Factual questions trigger a Wikipedia → DuckDuckGo search. The answer is displayed and automatically saved for future offline recall.
Code Search	Programming queries fetch snippets from Stack Overflow (with fallback local snippets). Each code block includes a copy button.
Personal Knowledge Base	Every 👍‑approved answer is stored permanently in IndexedDB (virtually unlimited capacity). The 🧠 MEMORY button recalls the best‑matching fact.
Short‑Term Memory	The last 30 conversation turns are saved and injected into the AI's prompt for contextual replies.
Offline Capability	After the initial model download (~80 MB), the entire app works without internet (except web/code searches).
🏗️ Technical Architecture

    Storage: IndexedDB for facts and conversation memory (unlimited), with a one‑time migration from older localStorage versions.

    AI Model: @xenova/transformers pipeline with Xenova/LaMini‑Flan‑T5‑77M.

    Web APIs:

        Wikipedia REST API (no CORS proxy needed)

        DuckDuckGo Instant Answer (via allorigins.win proxy)

        Stack Exchange API for code snippets

    UI: CRT‑style terminal with starfield, scanlines, glitch effects, three color themes, retro beeps, and fullscreen mode.

🎮 User Interaction

    Type a message and press Enter → the AI router decides the tool.

    Click any tool button to force the next response type.

    Use 👍 to save a fact, 👎 to correct it.

    ↑/↓ arrow keys navigate command history.

    Triple‑click the brand tag to export/import your entire knowledge base as JSON.

    📚 FORGET clears all learned facts; 🧠 CLEAR CHAT wipes conversation memory.

This is a complete, standalone offline‑first AI assistant that learns from every interaction and gives you full control over how it responds.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MYTHIC AI · Unlimited Memory (IndexedDB)</title>
    <link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
    <script type="importmap">
        {
            "imports": {
                "@xenova/transformers": "https://cdn.jsdelivr.net/npm/@xenova/transformers@2.17.2/dist/transformers.min.js"
            }
        }
    </script>
    <style>
        /* All CSS unchanged */
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'VT323', monospace; }
        body {
            background: #000;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
            padding: 20px;
            overflow-y: auto;
            position: relative;
        }
        #starfield {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            z-index: 0;
            pointer-events: none;
            opacity: 0.6;
        }
        .device-case {
            background: rgba(10, 10, 26, 0.85);
            backdrop-filter: blur(2px);
            padding: 25px;
            border-radius: 15px;
            box-shadow: 0 0 0 3px #2a2a4a, 0 0 30px rgba(0,150,255,0.3), 0 15px 35px rgba(0,0,0,0.9), inset 0 0 30px rgba(0,0,0,0.5);
            width: 100%;
            max-width: 1000px;
            position: relative;
            border: 1px solid #3a3a5a;
            margin: 20px 0;
            z-index: 2;
            transition: all 0.3s;
        }
        .device-case.fullscreen {
            position: fixed;
            top: 10px; left: 10px; right: 10px; bottom: 10px;
            max-width: none;
            width: auto;
            margin: 0;
            z-index: 100;
            background: rgba(5, 5, 20, 0.95);
        }
        .screw { position: absolute; width: 14px; height: 14px; background: linear-gradient(45deg, #4a4a6a, #2a2a3a); border-radius: 50%; box-shadow: inset 1px 1px 2px rgba(0,0,0,0.5); z-index: 5; }
        .screw::after { content: ''; position: absolute; top: 50%; left: 50%; width: 9px; height: 2px; background: #1a1a2a; transform: translate(-50%, -50%) rotate(45deg); }
        .screw::before { content: ''; position: absolute; top: 50%; left: 50%; width: 9px; height: 2px; background: #1a1a2a; transform: translate(-50%, -50%) rotate(-45deg); }
        .tl { top: 15px; left: 15px; } .tr { top: 15px; right: 15px; } .bl { bottom: 15px; left: 15px; } .br { bottom: 15px; right: 15px; }
        .terminal-container {
            background: #000a1a;
            border: 4px solid #001a3a;
            border-radius: 4px;
            box-shadow: inset 0 0 30px rgba(0,150,255,0.15), 0 0 20px rgba(0,100,200,0.3);
            overflow: hidden;
            position: relative;
            display: flex;
            flex-direction: column;
            height: 600px;
            transition: border-color 0.2s, box-shadow 0.2s;
        }
        .scanline { position: absolute; top: 0; left: 0; width: 100%; height: 8px; background: linear-gradient(180deg, transparent, rgba(0,200,255,0.15), transparent); opacity: 0.5; animation: scanline 4s linear infinite; pointer-events: none; z-index: 4; }
        .crt-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: repeating-linear-gradient(0deg, rgba(0,0,0,0.12), rgba(0,0,0,0.12) 1px, transparent 1px, transparent 2px); pointer-events: none; z-index: 3; box-shadow: inset 0 0 60px rgba(0,0,0,0.8); }
        @keyframes scanline { 0% { top: -10%; } 100% { top: 110%; } }
        .terminal-header {
            background: linear-gradient(90deg, #001a3a, #002a5a, #001a3a);
            padding: 10px 15px;
            border-bottom: 2px solid #004488;
            display: flex;
            justify-content: space-between;
            align-items: center;
            z-index: 2;
            flex-wrap: wrap;
            gap: 5px;
        }
        .brand-tag {
            font-size: 1.1rem;
            color: #00ccff;
            background: #000a1a;
            padding: 2px 10px;
            border: 1px solid #00ccff;
            font-weight: bold;
            letter-spacing: 2px;
            text-shadow: 0 0 8px #00ccff;
            cursor: pointer;
            user-select: none;
        }
        .temporal-status { display: flex; flex-direction: column; align-items: center; color: #00aaff; font-size: 0.9rem; text-shadow: 0 0 5px #00aaff; }
        .temporal-status .year { font-size: 1.4rem; color: #00ffff; text-shadow: 0 0 10px #00ffff; }
        .clock-status { font-size: 1rem; color: #88ddff; }
        .avatar-display { font-size: 12px; color: #00ccff; text-align: center; line-height: 1.1; text-shadow: 0 0 5px #00ccff; white-space: pre; font-family: 'VT323', monospace; min-width: 100px; transition: all 0.3s; }
        .status-container { display: flex; gap: 8px; }
        .status-light { width: 12px; height: 12px; border-radius: 50%; background: #1a1a2a; border: 1px solid #000; }
        .status-light.active-cyan { background: #00ffff; box-shadow: 0 0 12px #00ffff; animation: pulse 1s infinite alternate; }
        .status-light.active-blue { background: #0088ff; box-shadow: 0 0 10px #0088ff; }
        .status-light.active-amber { background: #ffaa00; box-shadow: 0 0 10px #ffaa00; animation: pulse 0.3s infinite alternate; }
        @keyframes pulse { from { opacity: 0.6; } to { opacity: 1; } }
        .terminal-body {
            flex: 1;
            padding: 15px;
            color: #00ddff;
            font-size: 1.4rem;
            line-height: 1.4;
            text-shadow: 0 0 5px rgba(0,220,255,0.7);
            overflow-y: auto;
            position: relative;
        }
        .message {
            margin-bottom: 12px;
            word-wrap: break-word;
            animation: fadeIn 0.3s;
            padding: 5px 10px;
            background: rgba(0,20,40,0.4);
            border-radius: 6px;
            border-left: 2px solid #0066aa;
            position: relative;
        }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
        .user-message { color: #00ffff; text-shadow: 0 0 8px #00ffff; border-left-color: #00aaff; }
        .system-message { color: #ffbb44; font-weight: bold; text-shadow: 0 0 6px rgba(255,170,0,0.7); }
        .error-message { color: #ff6666; text-shadow: 0 0 6px rgba(255,100,100,0.7); }
        .prediction-message { color: #88ffcc; text-shadow: 0 0 5px rgba(136,255,204,0.7); border-left: 3px solid #00aa88; }
        .mythic-message { color: #d0e8ff; border-left: 4px solid #aa88ff; padding-left: 14px; background: #00183050; text-shadow: 0 0 6px rgba(170,136,255,0.5); }
        .code-message {
            border-left: 4px solid #00ccaa;
            background: #0a1a2a;
            padding: 10px;
            font-family: 'VT323', monospace;
            white-space: pre-wrap;
            color: #aaffaa;
            margin: 8px 0;
            border-radius: 6px;
            position: relative;
        }
        .copy-code-btn {
            position: absolute;
            top: 5px;
            right: 5px;
            background: #1a3a4a;
            border: 1px solid #00ccaa;
            color: #aaffaa;
            font-size: 0.9rem;
            padding: 2px 6px;
            cursor: pointer;
            border-radius: 4px;
            opacity: 0.7;
            transition: opacity 0.2s;
            z-index: 5;
        }
        .copy-code-btn:hover { opacity: 1; background: #00ccaa; color: #000; }
        .learn-message { border-left: 4px solid #ffaa44; background: #1a2a0a; padding: 8px 12px; color: #ffffaa; }
        .feedback-buttons { display: inline-flex; gap: 8px; margin-top: 8px; margin-left: 10px; }
        .feedback-btn { background: #0a1a2a; border: 1px solid #00ccff; color: #00ccff; font-size: 1rem; padding: 2px 8px; border-radius: 16px; cursor: pointer; font-family: monospace; transition: all 0.2s; }
        .feedback-btn:hover { background: #00ccff; color: #000; box-shadow: 0 0 8px #0af; }
        .correction-input { display: inline-block; margin-left: 8px; background: #000a1a; border: 1px solid #ffaa44; color: #ffaa44; font-family: monospace; padding: 2px 5px; font-size: 0.9rem; width: 180px; }
        .prompt { opacity: 0.8; margin-right: 8px; font-weight: bold; color: #88ddff; }
        .alien-icon { display: inline-block; margin-right: 4px; filter: drop-shadow(0 0 4px #88ff88); }
        .terminal-input-area { background: #000a1a; padding: 12px; border-top: 2px solid #003a6a; display: flex; align-items: center; }
        #user-input { flex: 1; background: transparent; border: none; color: #00ffff; font-size: 1.5rem; outline: none; font-family: 'VT323', monospace; text-shadow: 0 0 5px #00ffff; }
        .controls-panel { margin-top: 15px; display: flex; justify-content: space-between; gap: 8px; padding: 10px; background: #0a0a1a; border-radius: 5px; border: 1px solid #2a2a4a; flex-wrap: wrap; }
        .hw-btn {
            flex: 1;
            background: linear-gradient(180deg, #2a2a4a, #1a1a2e);
            border: 1px solid #000;
            color: #9ac0dd;
            padding: 10px;
            font-family: 'VT323', monospace;
            font-size: 1.1rem;
            cursor: pointer;
            text-transform: uppercase;
            border-radius: 4px;
            box-shadow: 0 3px 0 #000;
            transition: all 0.1s;
            text-align: center;
            letter-spacing: 1px;
            font-weight: bold;
            min-width: 100px;
        }
        .hw-btn:active { transform: translateY(3px); box-shadow: 0 0 0 #000; color: #fff; }
        .hw-btn:hover {
            color: #00ffff;
            background: linear-gradient(180deg, #3a3a5a, #2a2a3e);
            box-shadow: 0 0 15px rgba(0,200,255,0.3);
            animation: glitch-hover 0.3s infinite;
        }
        @keyframes glitch-hover { 0%{transform:skew(0deg)}25%{transform:skew(2deg)}75%{transform:skew(-2deg)}100%{transform:skew(0deg)} }
        .danger-btn { color: #ff8888; border-color: #5a2a2a; background: linear-gradient(180deg, #3a1a1a, #2a0a0a); }
        .danger-btn:hover { color: #ff4444; text-shadow: 0 0 8px #ff4444; background: linear-gradient(180deg, #4a2a2a, #3a1a1a); }
        .special-btn { color: #eebb44; border-color: #5a4a00; background: linear-gradient(180deg, #3a3a1a, #2a2a0a); }
        .special-btn:hover { color: #ffcc00; text-shadow: 0 0 8px #ffcc00; background: linear-gradient(180deg, #4a4a2a, #3a3a1a); }
        .tool-btn { color: #aaddff; border-color: #2a6a9a; background: linear-gradient(180deg, #1a3a5a, #0a2a4a); }
        .tool-btn:hover { color: #aaffff; text-shadow: 0 0 8px #aaffff; background: linear-gradient(180deg, #2a4a6a, #1a3a5a); }
        .screen-shake { animation: shake 0.4s cubic-bezier(.36,.07,.19,.97) both; }
        @keyframes shake { 10%,90%{ transform: translate3d(-1px,0,0); } 20%,80%{ transform: translate3d(3px,0,0); } 30%,50%,70%{ transform: translate3d(-5px,0,0); } 40%,60%{ transform: translate3d(5px,0,0); } }
        .glitch-text { animation: glitch-anim 0.2s infinite; }
        @keyframes glitch-anim { 0%{ transform: translate(0) } 20%{ transform: translate(-2px,2px) } 40%{ transform: translate(-2px,-2px) } 60%{ transform: translate(2px,2px) } 80%{ transform: translate(2px,-2px) } 100%{ transform: translate(0) } }
        .timestamp { color: #88bbdd; font-size: 1rem; opacity: 0.9; font-weight: bold; }
        .attribution-link { color: #88aaff; font-size: 0.9rem; text-decoration: none; margin-left: 8px; }
        .attribution-link:hover { text-decoration: underline; }
        .scroll-lock-indicator {
            position: absolute;
            bottom: 10px;
            right: 20px;
            background: #0a1a2a;
            color: #ffaa00;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 0.9rem;
            border: 1px solid #ffaa00;
            opacity: 0;
            transition: opacity 0.3s;
            pointer-events: none;
            z-index: 10;
        }
        .progress-bar { display: inline-block; width: 150px; height: 16px; background: #001a2a; border: 1px solid #0af; margin-left: 8px; vertical-align: middle; }
        .progress-fill { height: 100%; width: 0%; background: #0af; box-shadow: 0 0 8px #0af; transition: width 0.1s; }
        body.scheme-green .terminal-container { border-color: #0a4a0a; }
        body.scheme-green .terminal-header { background: linear-gradient(90deg, #0a2a0a, #0a4a0a, #0a2a0a); }
        body.scheme-green .brand-tag { color: #0f0; border-color: #0f0; text-shadow: 0 0 8px #0f0; }
        body.scheme-green .terminal-body { color: #8f8; text-shadow: 0 0 5px #0f0; }
        body.scheme-green .user-message { color: #0f0; text-shadow: 0 0 8px #0f0; }
        body.scheme-green .mythic-message { border-left-color: #8f8; }
        body.scheme-green .code-message { border-left-color: #0f0; background: #0a1a0a; color: #afa; }
        body.scheme-amber .terminal-container { border-color: #8a5a0a; }
        body.scheme-amber .terminal-header { background: linear-gradient(90deg, #2a1a0a, #4a2a0a, #2a1a0a); }
        body.scheme-amber .brand-tag { color: #fb0; border-color: #fb0; text-shadow: 0 0 8px #fb0; }
        body.scheme-amber .terminal-body { color: #fc6; text-shadow: 0 0 5px #fb0; }
        body.scheme-amber .user-message { color: #fb0; text-shadow: 0 0 8px #fb0; }
        body.scheme-amber .mythic-message { border-left-color: #fc6; }
        body.scheme-amber .code-message { border-left-color: #fb0; background: #1a1a0a; color: #ffd; }
    </style>
</head>
<body class="scheme-cyan">
    <canvas id="starfield"></canvas>
    <div class="device-case" id="device-case">
        <div class="screw tl"></div><div class="screw tr"></div><div class="screw bl"></div><div class="screw br"></div>
        <div class="terminal-container" id="terminal-screen">
            <div class="scanline"></div><div class="crt-overlay"></div>
            <div class="terminal-header">
                <div class="brand-tag" id="brand-tag" title="Triple-click for Dev Tools">🧠 MYTHIC AI · UNLIMITED</div>
                <div class="temporal-status">
                    <div>SIGNAL FROM</div>
                    <div class="year" id="future-year">2157</div>
                    <div class="clock-status" id="live-clock">00:00:00</div>
                </div>
                <div class="avatar-display" id="avatar-face"></div>
                <div class="status-container">
                    <div class="status-light active-cyan" id="signal-light"></div>
                    <div class="status-light active-blue" id="sync-light"></div>
                    <div class="status-light" id="warn-light"></div>
                    <button id="mute-toggle" style="background:none;border:none;color:#0af;font-size:1.2rem;cursor:pointer;">🔊</button>
                </div>
            </div>
            <div class="terminal-body" id="output">
                <div class="message system-message">
                    <span class="timestamp">[INDEXEDDB STORAGE]</span><br>
                    ✨ Virtually unlimited memory – store millions of facts.<br>
                    <span id="loading-progress"></span>
                </div>
                <div class="message mythic-message">
                    <span class="prompt"><span class="alien-icon">🧠</span> MYTHIC></span> Ready. Use tools or let me decide.
                </div>
            </div>
            <div class="scroll-lock-indicator" id="scroll-lock-badge">🔒 SCROLL LOCK</div>
            <div class="terminal-input-area">
                <span class="prompt"><span class="alien-icon">👽</span> PRESENT></span>
                <input type="text" id="user-input" placeholder="Ask anything..." autocomplete="off">
            </div>
        </div>
        <div class="controls-panel">
            <button class="hw-btn tool-btn" id="btn-chat" title="Force chat (conversation)">💬 CHAT</button>
            <button class="hw-btn tool-btn" id="btn-code" title="Force code search">📟 CODE</button>
            <button class="hw-btn tool-btn" id="btn-web" title="Force web search">🌐 WEB</button>
            <button class="hw-btn tool-btn" id="btn-memory" title="Recall stored fact">🧠 MEMORY</button>
            <button class="hw-btn danger-btn" id="btn-sever">👽 SEVER</button>
            <button class="hw-btn" id="btn-clear-memory">🧠 CLEAR CHAT</button>
            <button class="hw-btn" id="btn-clear-facts">📚 FORGET</button>
            <button class="hw-btn" id="btn-color-scheme">🎨 THEME</button>
            <button class="hw-btn" id="btn-fullscreen">🖥️ FULL</button>
        </div>
    </div>

    <script type="module">
        import { pipeline, env } from '@xenova/transformers';
        env.allowLocalModels = false;
        env.useBrowserCache = true;

        // ---------- INDEXEDDB STORAGE (UNLIMITED) ----------
        const DB_NAME = 'mythic_ai_db';
        const DB_VERSION = 1;
        const STORE_FACTS = 'learned_facts';
        const STORE_MEMORY = 'conversation_memory';

        let db = null;

        async function openDatabase() {
            return new Promise((resolve, reject) => {
                const request = indexedDB.open(DB_NAME, DB_VERSION);
                request.onerror = () => reject(request.error);
                request.onsuccess = () => {
                    db = request.result;
                    resolve(db);
                };
                request.onupgradeneeded = (event) => {
                    const db = event.target.result;
                    if (!db.objectStoreNames.contains(STORE_FACTS)) {
                        db.createObjectStore(STORE_FACTS);
                    }
                    if (!db.objectStoreNames.contains(STORE_MEMORY)) {
                        db.createObjectStore(STORE_MEMORY);
                    }
                };
            });
        }

        async function saveFactsToDB(facts) {
            if (!db) await openDatabase();
            return new Promise((resolve, reject) => {
                const tx = db.transaction(STORE_FACTS, 'readwrite');
                const store = tx.objectStore(STORE_FACTS);
                store.put(facts, 'facts');
                tx.oncomplete = resolve;
                tx.onerror = () => reject(tx.error);
            });
        }

        async function loadFactsFromDB() {
            if (!db) await openDatabase();
            return new Promise((resolve, reject) => {
                const tx = db.transaction(STORE_FACTS, 'readonly');
                const store = tx.objectStore(STORE_FACTS);
                const request = store.get('facts');
                request.onsuccess = () => resolve(request.result || {});
                request.onerror = () => reject(request.error);
            });
        }

        async function saveMemoryToDB(memory) {
            if (!db) await openDatabase();
            return new Promise((resolve, reject) => {
                const tx = db.transaction(STORE_MEMORY, 'readwrite');
                const store = tx.objectStore(STORE_MEMORY);
                store.put(memory, 'memory');
                tx.oncomplete = resolve;
                tx.onerror = () => reject(tx.error);
            });
        }

        async function loadMemoryFromDB() {
            if (!db) await openDatabase();
            return new Promise((resolve, reject) => {
                const tx = db.transaction(STORE_MEMORY, 'readonly');
                const store = tx.objectStore(STORE_MEMORY);
                const request = store.get('memory');
                request.onsuccess = () => resolve(request.result || []);
                request.onerror = () => reject(request.error);
            });
        }

        async function clearFactsInDB() {
            if (!db) await openDatabase();
            return new Promise((resolve, reject) => {
                const tx = db.transaction(STORE_FACTS, 'readwrite');
                const store = tx.objectStore(STORE_FACTS);
                store.delete('facts');
                tx.oncomplete = resolve;
                tx.onerror = () => reject(tx.error);
            });
        }

        async function clearMemoryInDB() {
            if (!db) await openDatabase();
            return new Promise((resolve, reject) => {
                const tx = db.transaction(STORE_MEMORY, 'readwrite');
                const store = tx.objectStore(STORE_MEMORY);
                store.delete('memory');
                tx.oncomplete = resolve;
                tx.onerror = () => reject(tx.error);
            });
        }

        // Migration from old localStorage (one-time)
        async function migrateFromLocalStorage() {
            try {
                const oldFacts = localStorage.getItem('mythic_facts_v7');
                const oldMemory = localStorage.getItem('mythic_memory_v5');
                if (oldFacts) {
                    learnedFacts = JSON.parse(oldFacts);
                    await saveFactsToDB(learnedFacts);
                    localStorage.removeItem('mythic_facts_v7');
                }
                if (oldMemory) {
                    conversationMemory = JSON.parse(oldMemory);
                    await saveMemoryToDB(conversationMemory);
                    localStorage.removeItem('mythic_memory_v5');
                }
                // Also clear older versions
                for (let i = 1; i <= 6; i++) localStorage.removeItem(`mythic_facts_v${i}`);
                for (let i = 1; i <= 4; i++) localStorage.removeItem(`mythic_memory_v${i}`);
            } catch (e) {
                console.warn('Migration failed', e);
            }
        }

        // ---------- STATE ----------
        let learnedFacts = {};
        let conversationMemory = [];
        let generator = null, modelReady = false;
        let commandHistory = [];
        let historyIndex = -1;
        let audioCtx = null;
        let muted = false;
        let autoScroll = true;
        let colorSchemes = ['cyan', 'green', 'amber'];
        let currentScheme = 0;
        let forcedIntent = null;

        // DOM
        const output = document.getElementById('output');
        const input = document.getElementById('user-input');
        const futureYear = document.getElementById('future-year');
        const warnLight = document.getElementById('warn-light');
        const screen = document.getElementById('terminal-screen');
        const signalLight = document.getElementById('signal-light');
        const avatar = document.getElementById('avatar-face');
        const brandTag = document.getElementById('brand-tag');
        const liveClock = document.getElementById('live-clock');
        const scrollLockBadge = document.getElementById('scroll-lock-badge');
        const deviceCase = document.getElementById('device-case');
        const muteBtn = document.getElementById('mute-toggle');

        const avatarFaces = {
            idle: `   .    .   \n .   .  .  \n  [*_*]   \n .   .  .  \n   .    .   `,
            processing: `   .    .   \n .   .  .  \n  [O_O]   \n .   .  .  \n   .    .   `,
            cosmos: `   .    .   \n .   .  .  \n  [^_^]   \n .   .  .  \n   .    .   `,
            happy: `   .    .   \n .   .  .  \n  [^_^]   \n .   .  .  \n   .    .   `,
            error: `   .    .   \n .   .  .  \n  [>_<]   \n .   .  .  \n   .    .   `
        };
        avatar.textContent = avatarFaces.idle;

        // Audio
        function initAudio() {
            if (audioCtx) return;
            try { audioCtx = new (window.AudioContext || window.webkitAudioContext)(); } catch(e){}
        }
        function beep(type) {
            if (muted || !audioCtx) return;
            if (audioCtx.state === 'suspended') audioCtx.resume();
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain); gain.connect(audioCtx.destination);
            let freq = 800, dur = 0.08;
            if (type === 'send') { freq = 600; dur = 0.06; }
            else if (type === 'receive') { freq = 1000; dur = 0.1; }
            else if (type === 'error') { freq = 300; dur = 0.15; }
            osc.frequency.value = freq;
            gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + dur);
            osc.start(); osc.stop(audioCtx.currentTime + dur);
        }
        muteBtn.addEventListener('click', () => {
            muted = !muted;
            muteBtn.textContent = muted ? '🔇' : '🔊';
            if (!muted) initAudio();
        });

        // Starfield (unchanged)
        const canvas = document.getElementById('starfield');
        const ctx = canvas.getContext('2d');
        let stars = [];
        function resizeCanvas() { canvas.width = window.innerWidth; canvas.height = window.innerHeight; }
        function initStars(count=200) {
            stars = [];
            for(let i=0; i<count; i++) {
                stars.push({
                    x: Math.random() * canvas.width,
                    y: Math.random() * canvas.height,
                    size: Math.random() * 2 + 0.5,
                    speed: Math.random() * 0.5 + 0.1
                });
            }
        }
        function drawStars() {
            ctx.clearRect(0,0,canvas.width,canvas.height);
            ctx.fillStyle = '#ffffff';
            stars.forEach(s => {
                ctx.beginPath();
                ctx.arc(s.x, s.y, s.size, 0, Math.PI*2);
                ctx.fill();
                s.y += s.speed;
                if(s.y > canvas.height) { s.y = 0; s.x = Math.random() * canvas.width; }
            });
            requestAnimationFrame(drawStars);
        }
        window.addEventListener('resize', () => { resizeCanvas(); initStars(); });
        resizeCanvas(); initStars(); drawStars();

        // Clock
        function updateClock() {
            const now = new Date();
            liveClock.textContent = now.toLocaleTimeString();
        }
        setInterval(updateClock, 1000); updateClock();

        // Scroll lock
        output.addEventListener('scroll', () => {
            const atBottom = output.scrollHeight - output.scrollTop - output.clientHeight < 30;
            autoScroll = atBottom;
            scrollLockBadge.style.opacity = autoScroll ? '0' : '1';
        });

        // Storage wrappers (now async, but we call them and assign to global state)
        async function loadLearnedFacts() {
            try {
                await openDatabase();
                await migrateFromLocalStorage();
                const facts = await loadFactsFromDB();
                learnedFacts = facts || {};
            } catch (e) {
                console.warn('IndexedDB failed, falling back to memory', e);
                learnedFacts = {};
            }
        }
        async function saveLearnedFacts() {
            try {
                await saveFactsToDB(learnedFacts);
            } catch (e) { console.warn('Save facts failed', e); }
        }
        async function loadMemory() {
            try {
                const mem = await loadMemoryFromDB();
                conversationMemory = mem || [];
            } catch (e) { conversationMemory = []; }
        }
        async function saveMemory() {
            try {
                await saveMemoryToDB(conversationMemory);
            } catch (e) { console.warn('Save memory failed', e); }
        }
        function addToMemory(role, content) {
            conversationMemory.push({role, content});
            if(conversationMemory.length > 30) conversationMemory = conversationMemory.slice(-30);
            saveMemory();
        }
        async function clearMemory() {
            conversationMemory = [];
            await clearMemoryInDB();
        }
        async function clearFacts() {
            learnedFacts = {};
            await clearFactsInDB();
        }

        // Helpers (unchanged)
        function scrollToBottom() { if(autoScroll) { output.scrollTop = output.scrollHeight; } }
        function flashWarning() { warnLight.classList.add('active-amber'); setTimeout(() => warnLight.classList.remove('active-amber'), 400); }
        function setAvatar(face) { avatar.textContent = avatarFaces[face] || avatarFaces.idle; }
        function escapeHtml(t) { return t.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }

        function addLine(text, type='future-message', isHTML=false, extraData=null) {
            const div = document.createElement('div'); div.className = `message ${type}`;
            const prompts = {'user-message':'PRESENT>','system-message':'SYS>','prediction-message':'PRED>','mythic-message':'MYTHIC>','code-message':'CODE>','error-message':'ERROR>','learn-message':'LEARN>'};
            const prompt = prompts[type] || 'FUTURE>';
            if (type==='system-message'||type==='prediction-message'||type==='mythic-message'||type==='code-message'||type==='error-message'||type==='learn-message') div.innerHTML = text;
            else div.innerHTML = `<span class="prompt"><span class="alien-icon">👽</span> ${prompt}</span> ${text}`;
            output.appendChild(div);

            if(type === 'code-message') {
                const pre = div.querySelector('pre');
                if(pre) {
                    const btn = document.createElement('button');
                    btn.className = 'copy-code-btn';
                    btn.textContent = '📋 COPY';
                    btn.onclick = () => {
                        navigator.clipboard.writeText(pre.textContent).then(() => {
                            btn.textContent = '✓ COPIED';
                            setTimeout(() => btn.textContent = '📋 COPY', 1500);
                        });
                    };
                    div.style.position = 'relative';
                    div.appendChild(btn);
                }
            }

            if((type==='mythic-message'||type==='prediction-message'||type==='code-message') && extraData?.question) {
                const fbDiv = document.createElement('div');
                fbDiv.className = 'feedback-buttons';
                fbDiv.innerHTML = `<button class="feedback-btn up">👍</button><button class="feedback-btn down">👎</button>`;
                fbDiv.querySelector('.up').onclick = () => handleFeedback(true, extraData.question, extraData.answer, div, fbDiv);
                fbDiv.querySelector('.down').onclick = () => handleFeedback(false, extraData.question, extraData.answer, div, fbDiv);
                div.appendChild(fbDiv);
            }
            scrollToBottom(); flashWarning();
        }

        function handleFeedback(isCorrect, q, a, msgDiv, fbDiv) {
            if(isCorrect) {
                const subj = q.toLowerCase().replace(/[?,.!]/g,'').slice(0,50);
                learnedFacts[subj] = a.slice(0,200);
                saveLearnedFacts();
                addLine(`<span class="timestamp">[LEARN]</span> ✅ Stored: "${subj}..."`, 'learn-message');
                fbDiv.innerHTML = '<span style="color:#afa;">✓ Learned</span>';
                setAvatar('happy'); beep('receive');
            } else {
                const removed = learnedFacts[q.toLowerCase().slice(0,50)] ? true : false;
                if(removed) { delete learnedFacts[q.toLowerCase().slice(0,50)]; saveLearnedFacts(); }
                fbDiv.innerHTML = `<span style="color:#fa6;">✗ Correct?</span>
                    <input class="correction-input" placeholder="Type fix..."><button class="feedback-btn">Submit</button>`;
                const inp = fbDiv.querySelector('input');
                const sub = fbDiv.querySelector('button');
                sub.onclick = () => {
                    const corr = inp.value.trim();
                    if(corr) {
                        learnedFacts[q.toLowerCase().slice(0,50)] = corr;
                        saveLearnedFacts();
                        addLine(`<span class="timestamp">[LEARN]</span> ✅ Corrected: "${corr}"`, 'learn-message');
                        fbDiv.innerHTML = '<span style="color:#afa;">✓ Corrected</span>';
                        beep('receive');
                    } else {
                        fbDiv.innerHTML = '<span style="color:#f88;">No correction</span>';
                    }
                };
                beep('error');
            }
        }

        // -------- WEB SEARCH (AUTO‑LEARN) --------
        async function searchDuckDuckGo(query) {
            const proxy = 'https://api.allorigins.win/raw?url=';
            const apiUrl = `https://api.duckduckgo.com/?q=${encodeURIComponent(query)}&format=json&no_html=1&skip_disambig=1`;
            try {
                const resp = await fetch(proxy + encodeURIComponent(apiUrl));
                const data = await resp.json();
                if (data.Abstract) return { found: true, text: data.Abstract, heading: data.Heading };
                if (data.Definition) return { found: true, text: data.Definition, heading: data.Heading };
                if (data.Answer) return { found: true, text: data.Answer, heading: data.Heading };
                return { found: false };
            } catch { return { found: false }; }
        }

        async function searchWikipedia(query) {
            try {
                const url = `https://en.wikipedia.org/api/rest_v1/page/summary/${encodeURIComponent(query)}`;
                const resp = await fetch(url);
                const data = await resp.json();
                if (data.extract) return { found: true, text: data.extract, title: data.title };
                return { found: false };
            } catch { return { found: false }; }
        }

        async function fetchWebAnswer(query) {
            const wiki = await searchWikipedia(query);
            if (wiki.found) return { found: true, text: wiki.text, source: wiki.title };
            const ddg = await searchDuckDuckGo(query);
            if (ddg.found) return { found: true, text: ddg.text, source: ddg.heading };
            return { found: false };
        }

        // -------- CODE SEARCH (STACK OVERFLOW) --------
        const localSnippets = {
            "fetch api javascript": `fetch('https://api.example.com/data')\n  .then(response => response.json())\n  .then(data => console.log(data))\n  .catch(err => console.error(err));`,
            "css neon button": `.neon-button {\n  background: transparent;\n  border: 2px solid #0ff;\n  color: #0ff;\n  padding: 10px 20px;\n  text-shadow: 0 0 5px #0ff;\n  box-shadow: 0 0 10px #0ff;\n  transition: 0.3s;\n}\n.neon-button:hover {\n  background: #0ff;\n  color: #000;\n  box-shadow: 0 0 20px #0ff;\n}`,
            "html basic structure": `<!DOCTYPE html>\n<html>\n<head>\n    <title>My Page</title>\n</head>\n<body>\n    <h1>Hello World</h1>\n</body>\n</html>`
        };

        async function fetchStackOverflowCode(query) {
            const lowerQ = query.toLowerCase();
            for (const [key, snippet] of Object.entries(localSnippets)) {
                if (lowerQ.includes(key)) return { found: true, title: "Built‑in Snippet", code: snippet, link: null };
            }
            const searchUrl = `https://api.stackexchange.com/2.3/search/advanced?order=desc&sort=votes&q=${encodeURIComponent(query)}&site=stackoverflow&filter=withbody&pagesize=3`;
            try {
                const resp = await fetch(searchUrl);
                const data = await resp.json();
                if (data.error_id === 502) return { found: false, message: "Rate limit. Try later." };
                if (!data.items?.length) return { found: false, message: "No matches." };
                for (const item of data.items) {
                    const matches = [...item.body.matchAll(/<pre[^>]*><code[^>]*>([\s\S]*?)<\/code><\/pre>/gi)];
                    if (matches.length) {
                        let snippet = '';
                        for (const m of matches) {
                            let code = m[1].replace(/&lt;/g,'<').replace(/&gt;/g,'>').replace(/&amp;/g,'&').replace(/&quot;/g,'"').replace(/&#39;/g,"'").replace(/<[^>]*>/g,'');
                            snippet += code + '\n\n';
                        }
                        return { found: true, title: item.title, code: snippet.trim(), link: item.link };
                    }
                }
                return { found: false, message: "No code extracted." };
            } catch { return { found: false, message: "API error." }; }
        }

        // -------- LOCAL AI (TRANSFORMERS) --------
        async function loadModel() {
            if (generator) return true;
            const progSpan = document.getElementById('loading-progress');
            progSpan.innerHTML = `Loading model <span class="progress-bar"><span class="progress-fill" id="prog-fill"></span></span>`;
            const fill = document.getElementById('prog-fill');
            setAvatar('processing');
            try {
                let w = 0;
                const intv = setInterval(() => { w = Math.min(w+5, 90); fill.style.width = w+'%'; }, 200);
                generator = await pipeline('text2text-generation', 'Xenova/LaMini-Flan-T5-77M');
                clearInterval(intv);
                fill.style.width = '100%';
                setTimeout(() => progSpan.innerHTML = '', 500);
                modelReady = true;
                addLine(`<span class="timestamp">[ONLINE]</span> Neural core ready.`, 'system-message');
                setAvatar('cosmos'); beep('receive');
                return true;
            } catch(e) {
                modelReady = false;
                addLine(`<span class="timestamp">[ERROR]</span> Model failed. Using fallback.`, 'error-message');
                setAvatar('error'); beep('error');
                return false;
            }
        }

        function getRelevantFacts(query) {
            const lowerQuery = query.toLowerCase();
            const relevant = [];
            for (const [subj, val] of Object.entries(learnedFacts)) {
                if (lowerQuery.includes(subj.slice(0,30)) || subj.includes(lowerQuery.split(' ')[0])) {
                    relevant.push(`${subj} = ${val}`);
                }
            }
            return relevant;
        }

        function buildPromptWithFacts(userMessage) {
            const relevantFacts = getRelevantFacts(userMessage);
            let factSection = "";
            if (relevantFacts.length) {
                factSection = "Relevant facts you have learned:\n" + relevantFacts.map(f => `- ${f}`).join('\n') + "\n\n";
            }
            let p = factSection + "You are Mythic AI from 2157 with self‑learning memory. Use facts above if helpful.\n";
            conversationMemory.slice(-12).forEach(t => p += `${t.role==='user'?'Human':'AI'}: ${t.content}\n`);
            return p + `Human: ${userMessage}\nAI:`;
        }

        async function generateWithFacts(msg) {
            if (!modelReady) return "I cannot generate right now. Try a web search question.";
            try {
                const prompt = buildPromptWithFacts(msg);
                const res = await generator(prompt, {max_new_tokens:80, temperature:0.85, do_sample:true});
                let gen = res[0].generated_text.trim().replace(/^AI:\s*/i,'');
                return gen || "The future holds wonders.";
            } catch { return "Generation error."; }
        }

        // -------- AI INTENT CLASSIFIER (ROUTER) --------
        async function classifyIntent(userMessage) {
            if (!modelReady) {
                const lower = userMessage.toLowerCase();
                if (lower.includes('code') || lower.includes('html') || lower.includes('css') || lower.includes('javascript')) return 'code';
                if (lower.includes('who') || lower.includes('what') || lower.includes('when') || lower.includes('where') || lower.endsWith('?')) return 'web';
                return 'chat';
            }

            const prompt = `You are an intent classifier. Classify the following user message into exactly one of these categories: "chat", "code", "web". Reply with only the category word.

Examples:
User: "Hello" → chat
User: "Write a CSS button" → code
User: "Who is the president?" → web
User: "What's the weather?" → web
User: "Tell me a joke" → chat
User: "How do I center a div?" → code
User: "What is the meaning of life?" → web
User: "Good morning" → chat
User: "Create a JavaScript function to fetch data" → code
User: "When was Einstein born?" → web

User: "${userMessage}"
Category:`;

            try {
                const res = await generator(prompt, { max_new_tokens: 5, temperature: 0.1, do_sample: false });
                const category = res[0].generated_text.trim().toLowerCase();
                if (category.includes('code')) return 'code';
                if (category.includes('web')) return 'web';
                return 'chat';
            } catch {
                return 'chat';
            }
        }

        // -------- CORE PROCESSING (WITH FORCED INTENT) --------
        async function processUserInput(txt) {
            addLine(txt, 'user-message');
            addToMemory('user', txt);
            commandHistory.push(txt); historyIndex = commandHistory.length;
            screen.classList.add('screen-shake'); setTimeout(() => screen.classList.remove('screen-shake'), 400);
            setAvatar('processing'); signalLight.classList.remove('active-cyan'); signalLight.style.backgroundColor = '#0088ff';
            beep('send');

            let response = '', type = 'mythic-message';
            const cleanQuery = txt.toLowerCase().replace(/[?.,!]/g, '').trim();

            // 1. Check learned facts first
            if (learnedFacts[cleanQuery]) {
                response = learnedFacts[cleanQuery];
                type = 'mythic-message';
            }
            else {
                let intent = forcedIntent;
                if (!intent) {
                    intent = await classifyIntent(txt);
                    addLine(`<span class="timestamp">[AI ROUTER → ${intent.toUpperCase()}]</span>`, 'system-message');
                } else {
                    addLine(`<span class="timestamp">[MANUAL TOOL → ${intent.toUpperCase()}]</span>`, 'system-message');
                    forcedIntent = null;
                }

                if (intent === 'code') {
                    const cr = await fetchStackOverflowCode(txt);
                    if (cr.found) {
                        const escapedCode = escapeHtml(cr.code);
                        let linkHtml = cr.link ? `<br><a href="${cr.link}" target="_blank" class="attribution-link">↗ View on Stack Overflow</a>` : '';
                        response = `<span class="timestamp">[CODE · ${cr.title}]</span><br><pre class="code-message">${escapedCode}</pre>${linkHtml}`;
                        type = 'code-message';
                    } else {
                        response = `<span class="timestamp">[CODE SEARCH]</span> ${cr.message}`;
                        type = 'error-message';
                    }
                }
                else if (intent === 'web') {
                    const web = await fetchWebAnswer(txt);
                    if (web.found) {
                        response = `<span class="timestamp">[WEB · ${web.source || 'Result'}]</span><br>${web.text}`;
                        type = 'mythic-message';
                        learnedFacts[cleanQuery] = web.text.slice(0, 200);
                        await saveLearnedFacts();
                        addLine(`<span class="timestamp">[AUTO‑LEARN]</span> ✅ Answer saved for offline use.`, 'learn-message');
                    } else {
                        response = modelReady ? await generateWithFacts(txt) : "I couldn't find an answer online. Try rephrasing.";
                        if (!modelReady) type = 'error-message';
                    }
                }
                else if (intent === 'memory') {
                    const facts = Object.entries(learnedFacts);
                    let bestMatch = null, bestScore = 0;
                    for (const [key, val] of facts) {
                        if (cleanQuery.includes(key) || key.includes(cleanQuery)) {
                            const score = key.length;
                            if (score > bestScore) { bestScore = score; bestMatch = val; }
                        }
                    }
                    if (bestMatch) {
                        response = bestMatch;
                        type = 'mythic-message';
                        addLine(`<span class="timestamp">[MEMORY RECALL]</span> Found stored fact.`, 'learn-message');
                    } else {
                        response = modelReady ? await generateWithFacts(txt) : "No matching memory found. Using chat.";
                        if (!modelReady) type = 'prediction-message';
                    }
                }
                else { // 'chat'
                    response = modelReady ? await generateWithFacts(txt) : "Chat mode: I'm still loading.";
                    if (!modelReady) type = 'prediction-message';
                }
            }

            if (!response) response = "The cosmic signal is faint. Please try again.";

            const cleanAnswer = response.replace(/<[^>]*>/g, '').substring(0, 300);
            addToMemory('assistant', cleanAnswer);
            signalLight.style.backgroundColor = ''; signalLight.classList.add('active-cyan');
            setAvatar('cosmos'); beep('receive');
            addLine(response, type, true, { question: txt, answer: cleanAnswer });
            futureYear.textContent = Math.random() > 0.5 ? '2157' : ['2140','2165','2180'][Math.floor(Math.random()*3)];
        }

        // -------- EVENT LISTENERS --------
        input.addEventListener('keydown', (e) => {
            if(e.key === 'ArrowUp') {
                e.preventDefault();
                if(historyIndex > 0) { historyIndex--; input.value = commandHistory[historyIndex]; }
            } else if(e.key === 'ArrowDown') {
                e.preventDefault();
                if(historyIndex < commandHistory.length-1) { historyIndex++; input.value = commandHistory[historyIndex]; }
                else { historyIndex = commandHistory.length; input.value = ''; }
            }
        });
        input.addEventListener('keypress', async (e) => {
            if(e.key==='Enter' && input.value.trim()) {
                const t = input.value.trim(); input.value = '';
                await processUserInput(t);
            }
        });
        document.addEventListener('click', ()=>input.focus());

        // Tool buttons
        document.getElementById('btn-chat').addEventListener('click', () => {
            forcedIntent = 'chat';
            addLine(`<span class="timestamp">[TOOL]</span> Chat mode forced for next query.`, 'system-message');
            input.focus();
        });
        document.getElementById('btn-code').addEventListener('click', () => {
            forcedIntent = 'code';
            addLine(`<span class="timestamp">[TOOL]</span> Code search forced for next query.`, 'system-message');
            input.focus();
        });
        document.getElementById('btn-web').addEventListener('click', () => {
            forcedIntent = 'web';
            addLine(`<span class="timestamp">[TOOL]</span> Web search forced for next query.`, 'system-message');
            input.focus();
        });
        document.getElementById('btn-memory').addEventListener('click', () => {
            forcedIntent = 'memory';
            addLine(`<span class="timestamp">[TOOL]</span> Memory recall forced for next query.`, 'system-message');
            input.focus();
        });

        document.getElementById('btn-sever').addEventListener('click', ()=>{
            screen.classList.add('glitch-text'); addLine(`<span class="timestamp">[SEVERING...]</span>`,'error-message');
            setTimeout(()=>{ screen.classList.remove('glitch-text'); addLine("Link restored.",'mythic-message'); },2000);
            beep('error');
        });
        document.getElementById('btn-clear-memory').addEventListener('click', async ()=>{
            await clearMemory();
            addLine(`<span class="timestamp">[MEMORY PURGED]</span>`,'system-message');
            addToMemory('assistant', "Memory reset.");
        });
        document.getElementById('btn-clear-facts').addEventListener('click', async ()=>{
            await clearFacts();
            addLine(`<span class="timestamp">[FACTS FORGOTTEN]</span> All learned facts erased.`, 'system-message');
        });
        document.getElementById('btn-color-scheme').addEventListener('click', ()=>{
            currentScheme = (currentScheme + 1) % colorSchemes.length;
            document.body.className = `scheme-${colorSchemes[currentScheme]}`;
        });
        document.getElementById('btn-fullscreen').addEventListener('click', ()=>{
            deviceCase.classList.toggle('fullscreen');
        });

        // Dev tools (import/export now works with IndexedDB data)
        brandTag.addEventListener('click', (e) => {
            if(e.detail === 3) {
                const data = { facts: learnedFacts, memory: conversationMemory };
                const json = JSON.stringify(data);
                const action = prompt('DEV: Type "export" to copy JSON, or "import" to paste JSON');
                if(action === 'export') {
                    navigator.clipboard.writeText(json);
                    alert('JSON copied');
                } else if(action === 'import') {
                    const inp = prompt('Paste JSON:');
                    try {
                        const imp = JSON.parse(inp);
                        learnedFacts = imp.facts || {};
                        conversationMemory = imp.memory || [];
                        saveLearnedFacts();
                        saveMemory();
                        location.reload();
                    } catch(e) { alert('Invalid JSON'); }
                }
            }
        });

        // Random glitch
        setInterval(() => {
            if(Math.random() > 0.7) {
                screen.classList.add('glitch-text');
                setTimeout(() => screen.classList.remove('glitch-text'), 150);
            }
        }, 8000);

        // Init – now async
        window.addEventListener('load', async () => {
            input.focus();
            await openDatabase();
            await migrateFromLocalStorage();
            await loadLearnedFacts();
            await loadMemory();
            loadModel();
            if(conversationMemory.length) addLine(`<span class="timestamp">[MEMORY]</span> ${conversationMemory.length} exchanges loaded.`, 'system-message');
            if(Object.keys(learnedFacts).length) addLine(`<span class="timestamp">[FACTS]</span> ${Object.keys(learnedFacts).length} facts remembered.`, 'system-message');
            initAudio();
        });
    </script>
</body>
</html>
