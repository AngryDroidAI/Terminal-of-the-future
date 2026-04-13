Core Concept

    Knowledge Model: Purely key‑value storage (no real language model).

    Learning: On a new topic, queries the Wikipedia API, extracts the intro paragraph, and saves it with proper attribution.

    Recall: Subsequent identical queries retrieve the stored answer without network calls.

    Storage Capacity: Can hold tens of thousands of article summaries before hitting browser storage limits.

Key Features
Feature	Description
Chat Interface	User asks questions; responses include Wikipedia extracts with clickable links to the full article and license info.
Neural Network Visualization	Dynamic canvas shows “neurons” (stored facts) and “synapses” (semantic connections between topics). Hover/click reveals details.
Persistent Memory	Facts and conversation history are saved in IndexedDB, surviving page reloads.
Demo Mode	Pre‑loads example facts about AI, neural networks, and Wikipedia.
Toolbar Controls	Search, recall stats, clear chat, forget facts, theme toggle, fullscreen, and a simulated “sever” action.
Sound & Visual Feedback	Subtle beeps for send/receive; active neurons glow; LED indicators show system status.
Attribution Compliant	Every Wikipedia snippet includes a link back to the article and the CC BY‑SA 4.0 license notice.
How It Works (Simplified)

    User enters a question → app checks local learnedFacts object (loaded from IndexedDB).

    If found → displays the stored HTML (with attribution) and briefly activates the corresponding neuron.

    If not found → calls Wikipedia’s opensearch and query APIs (with proper User-Agent), stores the extract + attribution HTML, then displays it.

    The neural canvas redraws nodes and edges based on keyword overlap between stored topics.

Technical Stack

    Frontend: HTML5, CSS3, vanilla JavaScript (ES6 modules).

    Storage: IndexedDB for facts and conversation history.

    APIs: Wikipedia (action=opensearch, action=query).

    Visuals: Canvas 2D with custom graph layout and particle background.

This app is a creative, educational demo that illustrates how an “LLM” could work if it simply memorized Wikipedia—and it does so with a slick cyberpunk aesthetic and full compliance with Wikimedia’s attribution requirements.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MYTHIC AI · WIKI LLM (Neural Network)</title>
    <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600&family=JetBrains+Mono&display=swap" rel="stylesheet">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            background: #0b0e14;
            font-family: 'Space Grotesk', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
            color: #e3e9ff;
        }
        #graph-bg {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            z-index: 0;
            opacity: 0.3;
            pointer-events: none;
        }
        .studio-container {
            position: relative;
            z-index: 5;
            width: 100%;
            max-width: 1600px;
            background: rgba(12, 20, 35, 0.75);
            backdrop-filter: blur(12px);
            border-radius: 32px;
            border: 1px solid rgba(0, 255, 255, 0.3);
            box-shadow: 0 20px 50px rgba(0,0,0,0.7), 0 0 0 1px rgba(0, 200, 255, 0.1) inset, 0 0 40px rgba(0, 180, 255, 0.2);
            padding: 24px;
        }
        .studio-container.fullscreen {
            position: fixed;
            inset: 0;
            max-width: 100vw;
            max-height: 100vh;
            border-radius: 0;
            overflow: auto;
        }
        .studio-main {
            display: flex;
            gap: 24px;
            height: 700px;
            min-height: 60vh;
            margin-bottom: 20px;
        }
        @media (max-width: 900px) {
            .studio-main {
                flex-direction: column;
                height: auto;
                min-height: 500px;
            }
            .studio-main .chat-panel,
            .studio-main .graph-panel {
                min-height: 350px;
            }
        }
        .chat-panel {
            flex: 1.2;
            display: flex;
            flex-direction: column;
            background: rgba(8, 15, 26, 0.7);
            border-radius: 24px;
            border: 1px solid rgba(64, 224, 255, 0.25);
            box-shadow: inset 0 0 30px rgba(0,0,0,0.5), 0 8px 20px rgba(0,0,0,0.3);
            overflow: hidden;
            backdrop-filter: blur(4px);
        }
        .chat-header {
            padding: 16px 20px;
            border-bottom: 1px solid #1e3a5f;
            display: flex;
            align-items: center;
            justify-content: space-between;
            background: rgba(10, 20, 40, 0.6);
            flex-wrap: wrap;
            gap: 10px;
        }
        .studio-brand {
            display: flex;
            align-items: center;
            gap: 12px;
        }
        .logo-icon {
            font-size: 28px;
            filter: drop-shadow(0 0 10px #0cf);
        }
        .brand-text {
            font-weight: 600;
            font-size: 1.6rem;
            letter-spacing: 1px;
            background: linear-gradient(135deg, #a0e9ff, #6b8cff, #c084fc);
            -webkit-background-clip: text;
            background-clip: text;
            color: transparent;
            text-shadow: 0 0 15px #3b82f6;
        }
        .model-badge {
            background: #0f1a2b;
            padding: 4px 12px;
            border-radius: 40px;
            border: 1px solid #2a6f9c;
            color: #7dd3fc;
            font-size: 0.9rem;
            font-family: 'JetBrains Mono', monospace;
        }
        .status-row {
            display: flex;
            align-items: center;
            gap: 12px;
        }
        .status-dot {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background: #1e3a5f;
        }
        .dot-cyan { background: #0ff; box-shadow: 0 0 12px #0ff; }
        .dot-purple { background: #a855f7; box-shadow: 0 0 10px #a855f7; }
        .chat-messages {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
            font-family: 'JetBrains Mono', monospace;
            font-size: 1rem;
            line-height: 1.5;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }
        .message {
            padding: 12px 16px;
            border-radius: 18px;
            max-width: 90%;
            word-wrap: break-word;
            backdrop-filter: blur(8px);
            animation: fadeSlide 0.2s ease-out;
            border: 1px solid rgba(255,255,255,0.05);
        }
        .message a {
            color: #7dd3fc;
            text-decoration: underline;
        }
        .message a:hover {
            color: #0cf;
        }
        @keyframes fadeSlide { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
        .user-msg {
            align-self: flex-end;
            background: linear-gradient(145deg, #1e3a5f, #0f1f33);
            border-bottom-right-radius: 4px;
            border-left: 3px solid #0cf;
            color: #e0f2fe;
        }
        .assistant-msg {
            align-self: flex-start;
            background: rgba(16, 30, 50, 0.9);
            border-bottom-left-radius: 4px;
            border-right: 3px solid #a78bfa;
            color: #d1e0ff;
        }
        .wiki-msg {
            border-right-color: #2dd4bf;
        }
        .system-msg {
            align-self: center;
            background: #111c2e;
            color: #94a3b8;
            font-size: 0.9rem;
            border: 1px dashed #3b82f6;
            padding: 6px 16px;
        }
        .attribution {
            font-size: 0.8rem;
            margin-top: 8px;
            padding-top: 8px;
            border-top: 1px dashed #2e5f8e;
            color: #9cb8d9;
        }
        .input-area {
            padding: 20px;
            border-top: 1px solid #1e3a5f;
            display: flex;
            align-items: center;
            gap: 12px;
            background: rgba(5, 12, 24, 0.6);
            flex-wrap: wrap;
        }
        #user-input {
            flex: 1;
            min-width: 150px;
            background: #0b1423;
            border: 1px solid #2a4a6a;
            border-radius: 40px;
            padding: 14px 20px;
            color: #e0f2fe;
            font-family: 'JetBrains Mono', monospace;
            font-size: 1rem;
            outline: none;
        }
        #user-input:focus {
            border-color: #0cf;
            box-shadow: 0 0 15px rgba(0, 204, 255, 0.3);
        }
        .send-btn {
            background: #1e4b6e;
            border: none;
            border-radius: 40px;
            padding: 12px 24px;
            color: white;
            font-weight: 600;
            cursor: pointer;
            border: 1px solid #3b82f6;
            transition: all 0.2s;
        }
        .send-btn:hover { background: #256f9e; box-shadow: 0 0 20px #0cf; }
        .graph-panel {
            flex: 0.9;
            background: rgba(8, 18, 28, 0.7);
            border-radius: 24px;
            border: 1px solid rgba(168, 85, 247, 0.4);
            backdrop-filter: blur(4px);
            display: flex;
            flex-direction: column;
            overflow: hidden;
            box-shadow: inset 0 0 40px rgba(0,0,0,0.4);
            min-width: 300px;
            min-height: 400px;
        }
        .graph-panel .canvas-wrapper {
            flex: 1;
            position: relative;
            height: 100%;
            min-height: 300px;
            background: radial-gradient(circle at 50% 50%, #0f1a2e, #050a14);
        }
        #neuralCanvas {
            position: absolute;
            top: 0; left: 0;
            width: 100%;
            height: 100%;
            cursor: pointer;
        }
        .graph-header {
            padding: 16px 20px;
            border-bottom: 1px solid #2a3a5a;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 8px;
        }
        .graph-title {
            font-weight: 600;
            color: #c4b5fd;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .toolbar {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-top: 16px;
            justify-content: center;
        }
        .tool-btn {
            background: rgba(20, 40, 70, 0.6);
            border: 1px solid #2e5f8e;
            color: #cbd5e1;
            padding: 10px 18px;
            border-radius: 40px;
            font-weight: 500;
            cursor: pointer;
            backdrop-filter: blur(5px);
            transition: all 0.2s;
        }
        .tool-btn:hover {
            background: #1e4b6e;
            border-color: #0cf;
            color: white;
            box-shadow: 0 0 12px #0cf;
        }
        .danger-btn { border-color: #b91c1c; color: #fca5a5; }
        .danger-btn:hover { background: #7f1d1d; border-color: #fca5a5; }
        .footer-note {
            display: flex;
            justify-content: space-between;
            margin-top: 12px;
            color: #7f8ea3;
            flex-wrap: wrap;
            gap: 10px;
        }
        #live-clock { font-family: monospace; color: #9cb8d9; }
        #graphStats { font-size: 0.85rem; color: #7dd3fc; }
        .searching {
            display: inline-block;
            animation: pulse 1s infinite;
        }
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
    </style>
</head>
<body>
<canvas id="graph-bg" aria-hidden="true"></canvas>
<div class="studio-container" id="studioContainer">
    <div class="studio-main">
        <div class="chat-panel" role="region" aria-label="Chat interface">
            <div class="chat-header">
                <div class="studio-brand" aria-label="Mythic AI Brand">
                    <span class="logo-icon" aria-hidden="true">📖</span>
                    <span class="brand-text">MYTHIC AI</span>
                    <span class="model-badge">WIKI LLM · NEURAL</span>
                </div>
                <div class="status-row" aria-label="System status">
                    <span class="status-dot dot-cyan" id="wikiLed" title="Wikipedia LLM active" aria-label="Wikipedia knowledge active"></span>
                    <span class="status-dot dot-purple" id="graphLed" aria-label="Neural network status"></span>
                    <span id="live-clock" aria-live="polite">--:--:--</span>
                </div>
            </div>
            <div class="chat-messages" id="chatOutput" role="log" aria-live="polite" aria-label="Chat messages">
                <div class="message system-msg" role="status">✨ MYTHIC AI · WIKI LLM active. Knowledge from Wikipedia (CC BY‑SA 4.0).</div>
                <div class="message assistant-msg wiki-msg">📚 Ask me anything. I'll search Wikipedia and build neural pathways.</div>
            </div>
            <div class="input-area">
                <input type="text" id="user-input" placeholder="Ask about anything on Wikipedia..." autocomplete="off" aria-label="Type your message" />
                <button class="send-btn" id="sendBtn" aria-label="Send message">➤ SEND</button>
                <button id="muteToggle" aria-label="Toggle sound" style="background:none;border:1px solid #3b82f6;border-radius:40px;padding:8px 16px;color:#9cc;cursor:pointer;">🔊</button>
            </div>
        </div>
        <div class="graph-panel" role="region" aria-label="Neural Network">
            <div class="graph-header">
                <div class="graph-title">
                    <span aria-hidden="true">🔮</span>
                    <span>NEURAL NETWORK</span>
                    <span style="font-size:0.8rem;">(synaptic memory)</span>
                </div>
                <div id="graphStats">neurons: 0 · synapses: 0</div>
            </div>
            <div class="canvas-wrapper">
                <canvas id="neuralCanvas" aria-label="Interactive neural network visualization"></canvas>
            </div>
        </div>
    </div>
    <div class="toolbar" role="toolbar" aria-label="Tools">
        <button class="tool-btn" id="btnWiki" aria-pressed="false">🌐 WIKI SEARCH</button>
        <button class="tool-btn" id="btnMemory">🧠 RECALL</button>
        <button class="tool-btn danger-btn" id="btnSever">⚠️ SEVER</button>
        <button class="tool-btn" id="btnClearChat">🧹 CLEAR CHAT</button>
        <button class="tool-btn" id="btnForgetFacts">📚 FORGET</button>
        <button class="tool-btn" id="btnTheme">🎨 THEME</button>
        <button class="tool-btn" id="btnFullscreen">🖥️ FULL</button>
        <button class="tool-btn" id="btnDemo">🎭 DEMO</button>
    </div>
    <div class="footer-note">
        <span>🧬 Content from Wikipedia · CC BY-SA 4.0</span>
        <span id="futureTag">SYNC · 2157</span>
    </div>
</div>
<script type="module">
    // ========================
    // WIKI LLM + NEURAL MEMORY
    // Attribution compliant: links to Wikipedia and license.
    // ========================

    const DB_NAME = 'mythic_neural_db';
    const STORE_FACTS = 'facts';
    const STORE_MEM = 'memory';
    let db;
    let learnedFacts = {};
    let conversationMemory = [];

    // Set a proper User-Agent as recommended by Wikimedia policy
    const USER_AGENT = 'MythicAI-WikiLLM/1.0 (https://example.com; your-email@example.com)';

    const openDB = () => new Promise((resolve, reject) => {
        const request = indexedDB.open(DB_NAME, 3);
        request.onerror = () => reject(request.error);
        request.onsuccess = () => { db = request.result; resolve(db); };
        request.onupgradeneeded = (event) => {
            const database = event.target.result;
            if (!database.objectStoreNames.contains(STORE_FACTS)) database.createObjectStore(STORE_FACTS);
            if (!database.objectStoreNames.contains(STORE_MEM)) database.createObjectStore(STORE_MEM);
        };
    });

    const saveFacts = async (facts) => {
        if (!db) await openDB();
        const tx = db.transaction(STORE_FACTS, 'readwrite');
        tx.objectStore(STORE_FACTS).put(facts, 'facts');
        return new Promise((resolve) => { tx.oncomplete = () => resolve(); });
    };

    const loadFacts = async () => {
        if (!db) await openDB();
        const tx = db.transaction(STORE_FACTS, 'readonly');
        return new Promise((resolve) => {
            const request = tx.objectStore(STORE_FACTS).get('facts');
            request.onsuccess = () => resolve(request.result || {});
            request.onerror = () => resolve({});
        });
    };

    const saveMemory = async (memory) => {
        if (!db) await openDB();
        const tx = db.transaction(STORE_MEM, 'readwrite');
        tx.objectStore(STORE_MEM).put(memory, 'memory');
        return new Promise((resolve) => { tx.oncomplete = () => resolve(); });
    };

    const loadMemory = async () => {
        if (!db) await openDB();
        const tx = db.transaction(STORE_MEM, 'readonly');
        return new Promise((resolve) => {
            const request = tx.objectStore(STORE_MEM).get('memory');
            request.onsuccess = () => resolve(request.result || []);
            request.onerror = () => resolve([]);
        });
    };

    const clearFactsDB = async () => {
        if (!db) await openDB();
        const tx = db.transaction(STORE_FACTS, 'readwrite');
        tx.objectStore(STORE_FACTS).delete('facts');
        return new Promise((resolve) => { tx.oncomplete = () => resolve(); });
    };

    const clearMemoryDB = async () => {
        if (!db) await openDB();
        const tx = db.transaction(STORE_MEM, 'readwrite');
        tx.objectStore(STORE_MEM).delete('memory');
        return new Promise((resolve) => { tx.oncomplete = () => resolve(); });
    };

    // UI elements
    const outputDiv = document.getElementById('chatOutput');
    const inputField = document.getElementById('user-input');
    const neuralCanvas = document.getElementById('neuralCanvas');
    const ctx = neuralCanvas.getContext('2d');
    const graphStats = document.getElementById('graphStats');
    const futureTag = document.getElementById('futureTag');
    const wikiLed = document.getElementById('wikiLed');

    let hoveredNeuron = null;
    let activeNeurons = new Set();
    let signalParticles = [];
    let audioCtx = null;
    let muted = false;

    function escapeHtml(text) {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }

    // Enhanced addMessage with HTML support for links
    function addMessage(text, type = 'assistant', extraClass = 'wiki-msg', isHtml = false) {
        const div = document.createElement('div');
        if (type === 'user') {
            div.className = 'message user-msg';
            div.textContent = text;
        } else if (type === 'system') {
            div.className = 'message system-msg';
            div.innerHTML = isHtml ? text : escapeHtml(text);
        } else {
            div.className = `message assistant-msg ${extraClass}`;
            if (isHtml) {
                div.innerHTML = text;
            } else {
                div.textContent = text;
            }
        }
        outputDiv.appendChild(div);
        outputDiv.scrollTop = outputDiv.scrollHeight;
    }

    function beep(type = 'send') {
        if (muted || !audioCtx) return;
        try {
            if (audioCtx.state === 'suspended') audioCtx.resume();
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);
            const frequency = type === 'send' ? 800 : 1000;
            oscillator.frequency.value = frequency;
            oscillator.type = 'sine';
            gainNode.gain.setValueAtTime(0.1, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.1);
            oscillator.start();
            oscillator.stop(audioCtx.currentTime + 0.1);
        } catch (e) {}
    }

    function updateClock() {
        document.getElementById('live-clock').textContent = new Date().toLocaleTimeString();
    }

    // Neural graph from facts
    function buildNeuralFromFacts() {
        const facts = Object.entries(learnedFacts);
        const neurons = [];
        facts.forEach(([key, val], i) => {
            neurons.push({
                id: `n${i}`,
                label: key.slice(0, 20),
                key,
                value: val.slice(0, 100),
                layer: i < 3 ? 0 : i < 6 ? 1 : 2
            });
        });
        const synapses = [];
        for (let i = 0; i < neurons.length; i++) {
            for (let j = i + 1; j < neurons.length; j++) {
                const words1 = new Set(neurons[i].key.toLowerCase().split(/\W+/));
                const words2 = neurons[j].key.toLowerCase().split(/\W+/);
                const overlap = [...words2].filter(w => words1.has(w) && w.length > 2).length;
                if (overlap) {
                    synapses.push({ from: neurons[i].id, to: neurons[j].id, weight: overlap, active: false });
                }
            }
        }
        return { neurons, synapses };
    }

    function drawNeuralNetwork() {
        const wrapper = neuralCanvas.parentElement;
        const w = neuralCanvas.width = wrapper.clientWidth;
        const h = neuralCanvas.height = wrapper.clientHeight;
        if (w === 0 || h === 0) {
            requestAnimationFrame(() => setTimeout(drawNeuralNetwork, 50));
            return;
        }
        const { neurons, synapses } = buildNeuralFromFacts();
        graphStats.textContent = `neurons: ${neurons.length} · synapses: ${synapses.length}`;
        ctx.clearRect(0, 0, w, h);
        if (!neurons.length) {
            ctx.fillStyle = '#4a6a8e';
            ctx.font = '16px "Space Grotesk", sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText('🧠 Ask Wikipedia to build neural memory', w / 2, h / 2);
            return;
        }
        const padding = 60;
        const layerGap = (w - padding * 2) / 2;
        const layerX = [padding, padding + layerGap, w - padding];
        const layerLabels = ['INPUT', 'HIDDEN', 'OUTPUT'];
        ctx.font = '12px "JetBrains Mono", monospace';
        for (let l = 0; l < 3; l++) {
            const layerNeurons = neurons.filter(n => n.layer === l);
            if (layerNeurons.length === 0) continue;
            ctx.fillStyle = l === 0 ? '#0cf' : l === 1 ? '#a78bfa' : '#22c55e';
            ctx.textAlign = 'center';
            ctx.fillText(layerLabels[l], layerX[l], 25);
            const neuronHeight = (h - padding * 2) / layerNeurons.length;
            layerNeurons.forEach((neuron, idx) => {
                const nx = layerX[l];
                const ny = padding + idx * neuronHeight + neuronHeight / 2;
                neuron.x = nx;
                neuron.y = ny;
                neuron.radius = l === 1 ? 16 : 14;
            });
        }
        synapses.forEach(synapse => {
            const fromNeuron = neurons.find(n => n.id === synapse.from);
            const toNeuron = neurons.find(n => n.id === synapse.to);
            if (!fromNeuron || !toNeuron) return;
            const isActive = activeNeurons.has(synapse.from) || activeNeurons.has(synapse.to);
            const isHovered = hoveredNeuron && (fromNeuron.id === hoveredNeuron.id || toNeuron.id === hoveredNeuron.id);
            ctx.beginPath();
            ctx.moveTo(fromNeuron.x, fromNeuron.y);
            ctx.lineTo(toNeuron.x, toNeuron.y);
            if (isActive) {
                ctx.strokeStyle = `rgba(0, 255, 200, ${0.4 + Math.sin(Date.now() / 100) * 0.2})`;
                ctx.lineWidth = 2 + synapse.weight;
            } else if (isHovered) {
                ctx.strokeStyle = 'rgba(168, 85, 247, 0.6)';
                ctx.lineWidth = 2;
            } else {
                ctx.strokeStyle = `rgba(100, 120, 180, ${0.15 + synapse.weight * 0.05})`;
                ctx.lineWidth = 1;
            }
            ctx.stroke();
        });
        neurons.forEach(neuron => {
            const isActive = activeNeurons.has(neuron.id);
            const isHovered = hoveredNeuron && neuron.id === hoveredNeuron.id;
            ctx.beginPath();
            ctx.arc(neuron.x, neuron.y, neuron.radius, 0, Math.PI * 2);
            if (isActive) {
                ctx.fillStyle = `rgba(0, 255, 200, ${0.6 + Math.sin(Date.now() / 150) * 0.3})`;
                ctx.shadowColor = '#00ffcc';
                ctx.shadowBlur = 20;
            } else if (neuron.layer === 0) {
                ctx.fillStyle = isHovered ? '#0cf' : '#0a7a9f';
            } else if (neuron.layer === 1) {
                ctx.fillStyle = isHovered ? '#a855f7' : '#6b21aa';
            } else {
                ctx.fillStyle = isHovered ? '#22c55e' : '#166534';
            }
            ctx.fill();
            ctx.shadowBlur = isActive || isHovered ? 15 : 0;
            ctx.strokeStyle = isHovered ? '#fff' : (neuron.layer === 0 ? '#0cf' : neuron.layer === 1 ? '#a78bfa' : '#22c55e');
            ctx.lineWidth = 2;
            ctx.stroke();
            ctx.shadowBlur = 0;
            ctx.fillStyle = '#e0f2fe';
            ctx.font = '9px "JetBrains Mono", monospace';
            ctx.textAlign = 'center';
            ctx.fillText(neuron.label, neuron.x, neuron.y + neuron.radius + 12);
        });
        if (hoveredNeuron) {
            ctx.fillStyle = 'rgba(0, 10, 20, 0.9)';
            ctx.fillRect(10, h - 70, w - 20, 60);
            ctx.strokeStyle = '#a78bfa';
            ctx.lineWidth = 1;
            ctx.strokeRect(10, h - 70, w - 20, 60);
            ctx.fillStyle = '#a0e9ff';
            ctx.font = 'bold 12px "JetBrains Mono", monospace';
            ctx.textAlign = 'left';
            ctx.fillText(hoveredNeuron.label, 20, h - 50);
            ctx.fillStyle = '#94a3b8';
            ctx.font = '11px "JetBrains Mono", monospace';
            const lines = hoveredNeuron.value.match(/.{1,50}/g) || [hoveredNeuron.value];
            lines.slice(0, 2).forEach((line, i) => { ctx.fillText(line, 20, h - 32 + i * 14); });
        }
        requestAnimationFrame(drawNeuralNetwork);
    }

    const refreshNeural = () => { drawNeuralNetwork(); };

    // Wikipedia search with proper attribution and User-Agent
    async function wikipediaSearch(query) {
        try {
            // First, search for the article
            const searchUrl = `https://en.wikipedia.org/w/api.php?action=opensearch&search=${encodeURIComponent(query)}&limit=3&format=json&origin=*`;
            const searchRes = await fetch(searchUrl, {
                headers: { 'User-Agent': USER_AGENT }
            });
            const searchData = await searchRes.json();
            if (!searchData[1] || searchData[1].length === 0) return { found: false };

            const title = searchData[1][0];
            const articleUrl = `https://en.wikipedia.org/wiki/${encodeURIComponent(title.replace(/ /g, '_'))}`;

            // Fetch the extract
            const summaryUrl = `https://en.wikipedia.org/w/api.php?action=query&prop=extracts&exintro=true&explaintext=true&titles=${encodeURIComponent(title)}&format=json&origin=*`;
            const summaryRes = await fetch(summaryUrl, {
                headers: { 'User-Agent': USER_AGENT }
            });
            const summaryData = await summaryRes.json();
            const pages = summaryData.query?.pages;
            if (pages) {
                const page = Object.values(pages)[0];
                if (page.extract) {
                    return {
                        found: true,
                        text: page.extract.slice(0, 600),
                        title: title,
                        url: articleUrl
                    };
                }
            }
            return { found: false };
        } catch (e) {
            console.error('Wikipedia error:', e);
            return { found: false };
        }
    }

    // Process user input: check local memory first, else query Wikipedia
    async function processInput(text) {
        addMessage(text, 'user');
        conversationMemory.push({ role: 'user', content: text });
        await saveMemory(conversationMemory);
        beep('send');

        const cleanKey = text.toLowerCase().replace(/[?.,!]/g, '').trim();

        // 1. Check neural memory
        if (learnedFacts[cleanKey]) {
            // Display stored fact (may contain HTML links if we stored them)
            addMessage(learnedFacts[cleanKey], 'assistant', 'wiki-msg', true);
            const keys = Object.keys(learnedFacts);
            const idx = keys.indexOf(cleanKey);
            if (idx >= 0) activeNeurons.add(`n${idx}`);
            setTimeout(() => activeNeurons.clear(), 1500);
            conversationMemory.push({ role: 'assistant', content: learnedFacts[cleanKey] });
            await saveMemory(conversationMemory);
            beep('receive');
            refreshNeural();
            return;
        }

        // 2. Wikipedia LLM (search)
        addMessage('<span class="searching">🔍 Searching Wikipedia...</span>', 'system', '', true);
        const wikiResult = await wikipediaSearch(text);

        let responseHtml = '';
        if (wikiResult.found) {
            const extract = wikiResult.text;
            const title = wikiResult.title;
            const url = wikiResult.url;
            const licenseLink = 'https://creativecommons.org/licenses/by-sa/4.0/';

            // Build HTML with proper attribution
            responseHtml = `
                <strong>📖 ${escapeHtml(title)}</strong><br><br>
                ${escapeHtml(extract)}<br><br>
                <div class="attribution">
                    🔗 <a href="${url}" target="_blank" rel="noopener">Read full article on Wikipedia</a><br>
                    📜 Content available under <a href="${licenseLink}" target="_blank" rel="noopener">CC BY-SA 4.0</a>
                </div>
            `;

            // Store the HTML representation for future recall
            learnedFacts[cleanKey] = responseHtml;
            await saveFacts(learnedFacts);
            refreshNeural();

            addMessage(responseHtml, 'assistant', 'wiki-msg', true);

            // Activate neuron
            const keys = Object.keys(learnedFacts);
            const idx = keys.indexOf(cleanKey);
            if (idx >= 0) {
                activeNeurons.add(`n${idx}`);
                setTimeout(() => activeNeurons.clear(), 2000);
            }
        } else {
            responseHtml = `I couldn't find a Wikipedia article on "<strong>${escapeHtml(text)}</strong>". Try rephrasing or asking about a broader topic.`;
            addMessage(responseHtml, 'assistant', 'wiki-msg', true);
        }

        conversationMemory.push({ role: 'assistant', content: responseHtml });
        await saveMemory(conversationMemory);
        beep('receive');
        futureTag.textContent = `SYNC · ${2157 + Math.floor(Math.random() * 30)}`;
        refreshNeural();
    }

    // Event listeners
    document.getElementById('sendBtn').addEventListener('click', async () => {
        const value = inputField.value.trim();
        if (!value) return;
        inputField.value = '';
        await processInput(value);
    });

    inputField.addEventListener('keypress', (e) => {
        if (e.key === 'Enter') document.getElementById('sendBtn').click();
    });

    // Toolbar actions
    document.getElementById('btnWiki').onclick = () => {
        addMessage('🌐 Wikipedia mode — ask anything. All content is CC BY-SA 4.0 licensed.', 'system', '', true);
        inputField.focus();
    };
    document.getElementById('btnMemory').onclick = () => {
        const keys = Object.keys(learnedFacts);
        addMessage(`🧠 Neural memory: ${keys.length} stored facts`, 'system');
    };
    document.getElementById('btnSever').onclick = () => { addMessage('⚠️ Severing link... (simulated)', 'system'); beep('send'); };
    document.getElementById('btnClearChat').onclick = async () => {
        await clearMemoryDB();
        conversationMemory = [];
        outputDiv.innerHTML = '';
        addMessage('✨ Chat cleared. Neural memory intact.', 'system');
    };
    document.getElementById('btnForgetFacts').onclick = async () => {
        await clearFactsDB();
        learnedFacts = {};
        activeNeurons.clear();
        signalParticles = [];
        refreshNeural();
        addMessage('📚 Neural pathways pruned.', 'system');
    };
    document.getElementById('btnTheme').onclick = () => { document.body.style.filter = document.body.style.filter ? '' : 'hue-rotate(180deg)'; };
    document.getElementById('btnFullscreen').onclick = () => { document.getElementById('studioContainer').classList.toggle('fullscreen'); };
    document.getElementById('btnDemo').onclick = async () => {
        const demoFacts = {
            'artificial intelligence': `<strong>📖 Artificial intelligence</strong><br><br>AI is intelligence demonstrated by machines.<br><br><div class="attribution">🔗 <a href="https://en.wikipedia.org/wiki/Artificial_intelligence" target="_blank">Wikipedia</a> · CC BY-SA 4.0</div>`,
            'neural network': `<strong>📖 Neural network</strong><br><br>Neural networks are computing systems inspired by biological brains.<br><br><div class="attribution">🔗 <a href="https://en.wikipedia.org/wiki/Neural_network" target="_blank">Wikipedia</a> · CC BY-SA 4.0</div>`,
            'wikipedia': `<strong>📖 Wikipedia</strong><br><br>Wikipedia is a free online encyclopedia.<br><br><div class="attribution">🔗 <a href="https://en.wikipedia.org/wiki/Wikipedia" target="_blank">Wikipedia</a> · CC BY-SA 4.0</div>`
        };
        for (const [k, v] of Object.entries(demoFacts)) { learnedFacts[k] = v; }
        await saveFacts(learnedFacts);
        refreshNeural();
        addMessage(`🎭 Demo neural network loaded with ${Object.keys(demoFacts).length} pathways!`, 'system');
    };

    document.getElementById('muteToggle').addEventListener('click', () => {
        muted = !muted;
        document.getElementById('muteToggle').textContent = muted ? '🔇' : '🔊';
        if (!muted && !audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    });

    // Canvas interactivity
    neuralCanvas.addEventListener('mousemove', (e) => {
        const rect = neuralCanvas.getBoundingClientRect();
        const mx = e.clientX - rect.left;
        const my = e.clientY - rect.top;
        const { neurons } = buildNeuralFromFacts();
        hoveredNeuron = null;
        for (const neuron of neurons) {
            const dist = Math.hypot(mx - neuron.x, my - neuron.y);
            if (dist < neuron.radius + 5) {
                hoveredNeuron = neuron;
                neuralCanvas.style.cursor = 'pointer';
                break;
            }
        }
        if (!hoveredNeuron) neuralCanvas.style.cursor = 'default';
    });

    neuralCanvas.addEventListener('click', () => {
        if (hoveredNeuron) {
            addMessage(`🧠 ${hoveredNeuron.key}: ${hoveredNeuron.value}`, 'system', '', true);
            activeNeurons.add(hoveredNeuron.id);
            setTimeout(() => activeNeurons.clear(), 1500);
            beep('receive');
        }
    });

    // Background animation
    const bgCanvas = document.getElementById('graph-bg');
    const bgCtx = bgCanvas.getContext('2d');
    function resizeBackground() {
        bgCanvas.width = window.innerWidth;
        bgCanvas.height = window.innerHeight;
    }
    window.addEventListener('resize', () => { resizeBackground(); refreshNeural(); });
    resizeBackground();
    const particles = [];
    for (let i = 0; i < 60; i++) {
        particles.push({
            x: Math.random() * bgCanvas.width,
            y: Math.random() * bgCanvas.height,
            vx: (Math.random() - 0.5) * 0.4,
            vy: (Math.random() - 0.5) * 0.4
        });
    }
    function animateBackground() {
        bgCtx.clearRect(0, 0, bgCanvas.width, bgCanvas.height);
        bgCtx.strokeStyle = '#2a4a7a';
        bgCtx.lineWidth = 0.5;
        particles.forEach((particle, i) => {
            particle.x += particle.vx;
            particle.y += particle.vy;
            if (particle.x < 0 || particle.x > bgCanvas.width) particle.vx *= -1;
            if (particle.y < 0 || particle.y > bgCanvas.height) particle.vy *= -1;
            particles.slice(i + 1).forEach(other => {
                const dx = particle.x - other.x;
                const dy = particle.y - other.y;
                const distance = Math.hypot(dx, dy);
                if (distance < 100) {
                    bgCtx.beginPath();
                    bgCtx.moveTo(particle.x, particle.y);
                    bgCtx.lineTo(other.x, other.y);
                    bgCtx.strokeStyle = `rgba(100, 180, 255, ${0.1 * (1 - distance / 100)})`;
                    bgCtx.stroke();
                }
            });
        });
        requestAnimationFrame(animateBackground);
    }
    animateBackground();

    // Initialize
    document.addEventListener('DOMContentLoaded', async () => {
        setInterval(updateClock, 1000);
        updateClock();
        await openDB();
        learnedFacts = await loadFacts() || {};
        conversationMemory = await loadMemory() || [];
        requestAnimationFrame(() => {
            setTimeout(() => {
                refreshNeural();
                new ResizeObserver(refreshNeural).observe(neuralCanvas.parentElement);
            }, 150);
        });
        inputField.focus();
        if (conversationMemory.length) addMessage(`📀 ${conversationMemory.length} messages loaded`, 'system');
        if (Object.keys(learnedFacts).length) addMessage(`🧠 ${Object.keys(learnedFacts).length} neural pathways ready`, 'system');
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        wikiLed.classList.add('dot-cyan');
        console.log('✨ MYTHIC AI · WIKI LLM — Neural network active (attribution compliant)');
    });
</script>
</body>
</html>
