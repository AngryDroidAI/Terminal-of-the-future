MYTHIC AI · DUAL MIND
A browser-based AI chat interface with dual neural models and interactive knowledge visualization.
Core Features
Dual AI Models:
- Primary Mind - Flan-T5 (text2text-generation) for intent classification and generation
- Second Mind - GPT-2 for creative refinement and collaboration
- Toggle "Dual Mind" for synthesized responses from both models
Knowledge System:
- IndexedDB persistence for facts and conversation history
- Facts auto-saved when using "Web" search mode
- Conversation memory persists across sessions
Interactive Neural Network:
- 3-layer visualization (INPUT → HIDDEN → OUTPUT)
- Facts distributed across layers based on order
- Animated synaptic connections between related concepts
- Hover tooltips show stored information
- Click neurons to recall facts
- Signal particles flow through active pathways
Tools:
- CHAT / CODE / WEB mode forcing
- RECALL - view stored facts
- SEVER / CLEAR CHAT / FORGET - reset functionality
- DEMO - populate test data
- Fullscreen / Theme toggle
UI:
- Cyberpunk aesthetic with cyan/purple accents
- Backdrop blur panels
- Animated particle background
- Responsive layout
Technical Stack
- Pure HTML/CSS/JS (no build)
- @xenova/transformers for in-browser ML
- Canvas 2D API for neural network visualization
- IndexedDB for persistence
- Web Audio API for sound feedback
How It Works
1. Load page → models download from CDN (~50MB)
2. Ask questions → intent is classified
3. Web search → results saved as facts → neurons added
4. Chat responses → learn from conversation
5. Facts form synaptic connections in neural view

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MYTHIC AI · DUAL MIND (Neural Network)</title>
    <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600&family=JetBrains+Mono&display=swap" rel="stylesheet">
    <script type="importmap">
        {
            "imports": {
                "@xenova/transformers": "https://cdn.jsdelivr.net/npm/@xenova/transformers@2.17.2/dist/transformers.min.js"
            }
        }
    </script>
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
        .dot-orange { background: #f0b37b; box-shadow: 0 0 10px #f0b37b; }
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
        .mind1-msg { border-right-color: #0cf; }
        .mind2-msg { border-right-color: #f0b37b; }
        .system-msg {
            align-self: center;
            background: #111c2e;
            color: #94a3b8;
            font-size: 0.9rem;
            border: 1px dashed #3b82f6;
            padding: 6px 16px;
        }
        .code-block {
            background: #0a111f;
            border: 1px solid #2dd4bf;
            border-radius: 12px;
            padding: 14px;
            font-family: 'JetBrains Mono', monospace;
            white-space: pre-wrap;
            color: #b9f3f0;
            position: relative;
            max-width: 95%;
            overflow-x: auto;
        }
        .copy-btn {
            position: absolute;
            top: 8px;
            right: 8px;
            background: #1e3a5f;
            border: 1px solid #2dd4bf;
            color: #b9f3f0;
            border-radius: 6px;
            padding: 4px 10px;
            font-size: 0.8rem;
            cursor: pointer;
            transition: background 0.2s;
        }
        .copy-btn:hover { background: #2dd4bf; color: #0a111f; }
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
        .dual-toggle {
            background: #0f1a2b;
            border: 1px solid #a78bfa;
            color: #c4b5fd;
            padding: 8px 16px;
            border-radius: 40px;
            cursor: pointer;
            font-weight: 500;
            transition: all 0.2s;
        }
        .dual-toggle.active {
            background: #6b21a8;
            border-color: #d8b4fe;
            color: white;
            box-shadow: 0 0 15px #a855f7;
        }
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
                    <span class="logo-icon" aria-hidden="true">🔮</span>
                    <span class="brand-text">MYTHIC AI</span>
                    <span class="model-badge">DUAL MIND · NEURAL</span>
                </div>
                <div class="status-row" aria-label="System status">
                    <span class="status-dot" id="mind1Led" title="Primary Mind (Flan-T5)" aria-label="Primary mind status"></span>
                    <span class="status-dot" id="mind2Led" title="Second Mind (GPT-2)" aria-label="Second mind status"></span>
                    <span class="status-dot dot-purple" id="graphLed" aria-label="Neural network status"></span>
                    <span id="live-clock" aria-live="polite">--:--:--</span>
                </div>
            </div>
            <div class="chat-messages" id="chatOutput" role="log" aria-live="polite" aria-label="Chat messages">
                <div class="message system-msg" role="status">✨ MYTHIC AI · DUAL MIND active. Neural network ready.</div>
                <div class="message assistant-msg mind1-msg">🧠 Primary Mind online. Toggle Second Mind for collaboration.</div>
            </div>
            <div class="input-area">
                <input type="text" id="user-input" placeholder="Ask Mythic AI..." autocomplete="off" aria-label="Type your message" />
                <button class="dual-toggle" id="dualMindToggle" title="Toggle Second Mind (GPT-2) collaboration" aria-pressed="false">🧠 Dual Mind</button>
                <button class="send-btn" id="sendBtn" aria-label="Send message">➤ SEND</button>
                <button id="muteToggle" aria-label="Toggle sound" style="background:none;border:1px solid #3b82f6;border-radius:40px;padding:8px 16px;color:#9cc;cursor:pointer;">🔊</button>
            </div>
        </div>
        <div class="graph-panel" role="region" aria-label="Neural Network">
            <div class="graph-header">
                <div class="graph-title">
                    <span aria-hidden="true">🔮</span>
                    <span>NEURAL NETWORK</span>
                    <span style="font-size:0.8rem;">(synaptic connections)</span>
                </div>
                <div id="graphStats">neurons: 0 · synapses: 0</div>
            </div>
            <div class="canvas-wrapper">
                <canvas id="neuralCanvas" aria-label="Interactive neural network visualization"></canvas>
            </div>
        </div>
    </div>
    <div class="toolbar" role="toolbar" aria-label="Tools">
        <button class="tool-btn" id="btnChat" aria-pressed="false">💬 CHAT</button>
        <button class="tool-btn" id="btnCode" aria-pressed="false">📟 CODE</button>
        <button class="tool-btn" id="btnWeb" aria-pressed="false">🌐 WEB</button>
        <button class="tool-btn" id="btnMemory">🧠 RECALL</button>
        <button class="tool-btn danger-btn" id="btnSever">⚠️ SEVER</button>
        <button class="tool-btn" id="btnClearChat">🧹 CLEAR CHAT</button>
        <button class="tool-btn" id="btnForgetFacts">📚 FORGET</button>
        <button class="tool-btn" id="btnTheme">🎨 THEME</button>
        <button class="tool-btn" id="btnFullscreen">🖥️ FULL</button>
        <button class="tool-btn" id="btnDemo">🎭 DEMO</button>
    </div>
    <div class="footer-note">
        <span>🔮 neural pathways strengthen with use</span>
        <span id="futureTag">SYNC · 2157</span>
    </div>
</div>
<script type="module">
    import { pipeline, env } from '@xenova/transformers';
    env.allowLocalModels = false;
    env.useBrowserCache = true;
    const DB_NAME = 'mythic_neural_db';
    const STORE_FACTS = 'facts';
    const STORE_MEM = 'memory';
    let db;
    let learnedFacts = {};
    let conversationMemory = [];
    const openDB = () => new Promise((resolve, reject) => {
        const request = indexedDB.open(DB_NAME, 1);
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
    let generator1 = null;
    let generator2 = null;
    let model1Ready = false;
    let model2Ready = false;
    let dualMindActive = false;
    let forcedIntent = null;
    let audioCtx = null;
    let muted = false;
    const outputDiv = document.getElementById('chatOutput');
    const inputField = document.getElementById('user-input');
    const neuralCanvas = document.getElementById('neuralCanvas');
    const ctx = neuralCanvas.getContext('2d');
    const graphStats = document.getElementById('graphStats');
    const futureTag = document.getElementById('futureTag');
    const mind1Led = document.getElementById('mind1Led');
    const mind2Led = document.getElementById('mind2Led');
    const dualToggle = document.getElementById('dualMindToggle');
    let hoveredNeuron = null;
    let activeNeurons = new Set();
    let signalParticles = [];
    function escapeHtml(text) {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }
    function addMessage(text, type = 'assistant', mind = 1) {
        const div = document.createElement('div');
        if (type === 'code') {
            div.className = 'code-block';
            div.innerHTML = `<pre>${escapeHtml(text)}</pre>`;
            const btn = document.createElement('button');
            btn.className = 'copy-btn';
            btn.textContent = '📋';
            btn.setAttribute('aria-label', 'Copy code');
            btn.onclick = () => {
                navigator.clipboard.writeText(text).then(() => {
                    btn.textContent = '✓';
                    setTimeout(() => btn.textContent = '📋', 1000);
                });
            };
            div.appendChild(btn);
        } else if (type === 'user') {
            div.className = 'message user-msg';
            div.textContent = text;
        } else if (type === 'system') {
            div.className = 'message system-msg';
            div.textContent = text;
        } else {
            div.className = `message assistant-msg mind${mind}-msg`;
            div.textContent = text;
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
            const frequency = type === 'send' ? 800 : type === 'receive' ? 1000 : 600;
            oscillator.frequency.value = frequency;
            oscillator.type = 'sine';
            gainNode.gain.setValueAtTime(0.1, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.1);
            oscillator.start();
            oscillator.stop(audioCtx.currentTime + 0.1);
        } catch (e) { console.warn('Audio error:', e); }
    }
    function updateClock() {
        document.getElementById('live-clock').textContent = new Date().toLocaleTimeString();
    }
    function buildNeuralFromFacts() {
        const facts = Object.entries(learnedFacts);
        const neurons = [];
        facts.forEach(([key, val], i) => {
            neurons.push({
                id: `n${i}`,
                label: key.slice(0, 20),
                key,
                value: val.slice(0, 60),
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
            ctx.fillText('🧠 Train the neural network', w / 2, h / 2);
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
        if (signalParticles.length > 0) {
            signalParticles.forEach(p => {
                ctx.beginPath();
                ctx.arc(p.x, p.y, 3, 0, Math.PI * 2);
                ctx.fillStyle = '#00ffcc';
                ctx.shadowColor = '#00ffaa';
                ctx.shadowBlur = 10;
                ctx.fill();
                ctx.shadowBlur = 0;
            });
            signalParticles = signalParticles.filter(p => {
                const target = neurons.find(n => n.id === p.to);
                if (!target) return false;
                const dx = target.x - p.x;
                const dy = target.y - p.y;
                const dist = Math.hypot(dx, dy);
                p.x += (dx / dist) * 3;
                p.y += (dy / dist) * 3;
                return dist > 5;
            });
        }
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
            if (isActive || isHovered) {
                ctx.shadowBlur = 15;
            }
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
    async function loadModels() {
        try {
            addMessage('🔵 Loading Primary Mind (Flan-T5)...', 'system');
            generator1 = await pipeline('text2text-generation', 'Xenova/LaMini-Flan-T5-77M');
            model1Ready = true;
            mind1Led.classList.add('dot-cyan');
            addMessage('✅ Primary Mind online.', 'system');
        } catch (e) {
            console.error('Primary model error:', e);
            addMessage('⚠️ Primary model failed to load.', 'system');
        }
        try {
            addMessage('🟠 Loading Second Mind (GPT-2)...', 'system');
            generator2 = await pipeline('text-generation', 'Xenova/gpt2');
            model2Ready = true;
            mind2Led.classList.add('dot-orange');
            addMessage('✅ Second Mind online. Toggle Dual Mind to collaborate.', 'system');
        } catch (e) {
            console.error('Secondary model error:', e);
            addMessage('⚠️ Second model unavailable.', 'system');
        }
    }
    async function classifyIntent(message) {
        if (!model1Ready) {
            const lower = message.toLowerCase();
            if (lower.includes('code') || lower.includes('function') || lower.includes('script')) return 'code';
            if (lower.includes('search') || lower.includes('find') || lower.includes('?') || lower.includes('what is') || lower.includes('who is') || lower.includes('how to') || lower.includes('define')) return 'web';
            return 'chat';
        }
        const prompt = `Classify: "chat", "code", or "web".\nUser: ${message}\nCategory:`;
        try {
            const result = await generator1(prompt, { max_new_tokens: 5, temperature: 0.1 });
            const category = result[0].generated_text.toLowerCase();
            if (category.includes('code')) return 'code';
            if (category.includes('web')) return 'web';
            return 'chat';
        } catch (e) {
            console.error('Intent classification error:', e);
            return 'chat';
        }
    }
    async function wikipediaSearch(query) {
        try {
            const searchUrl = `https://en.wikipedia.org/w/api.php?action=opensearch&search=${encodeURIComponent(query)}&limit=3&format=json&origin=*`;
            const searchRes = await fetch(searchUrl);
            const searchData = await searchRes.json();
            if (!searchData[1] || searchData[1].length === 0) return { found: false };
            const title = searchData[1][0];
            const summaryUrl = `https://en.wikipedia.org/w/api.php?action=query&prop=extracts&exintro=true&explaintext=true&titles=${encodeURIComponent(title)}&format=json&origin=*`;
            const summaryRes = await fetch(summaryUrl);
            const summaryData = await summaryRes.json();
            const pages = summaryData.query?.pages;
            if (pages) {
                const page = Object.values(pages)[0];
                if (page.extract) {
                    return {
                        found: true,
                        text: page.extract.slice(0, 500),
                        title: title,
                        url: `https://en.wikipedia.org/wiki/${encodeURIComponent(title.replace(/ /g, '_'))}`
                    };
                }
            }
            return { found: false };
        } catch (e) {
            console.error('Wikipedia error:', e);
            return { found: false };
        }
    }
    async function duckDuckGoSearch(query) {
        try {
            const response = await fetch(
                `https://api.allorigins.win/raw?url=${encodeURIComponent('https://api.duckduckgo.com/?q=' + encodeURIComponent(query) + '&format=json&no_html=1')}`
            );
            const data = await response.json();
            if (data.Abstract) return { found: true, text: data.Abstract, source: 'DuckDuckGo' };
            if (data.RelatedTopics?.length > 0 && data.RelatedTopics[0]?.Text) {
                return { found: true, text: data.RelatedTopics[0].Text, source: 'DuckDuckGo' };
            }
            return { found: false };
        } catch (e) {
            return { found: false };
        }
    }
    async function smartWebSearch(query) {
        addMessage('<span class="searching">🔍 Searching Wikipedia...</span>', 'system');
        const wikiResult = await wikipediaSearch(query);
        if (wikiResult.found) {
            addMessage(`📚 Found on Wikipedia: ${wikiResult.title}`, 'system');
            return { ...wikiResult, source: 'Wikipedia' };
        }
        addMessage('🔍 Trying DuckDuckGo...', 'system');
        const ddgResult = await duckDuckGoSearch(query);
        if (ddgResult.found) {
            return ddgResult;
        }
        return { found: false };
    }
    const codeSnippets = {
        'fetch api': "fetch(url).then(r => r.json()).then(console.log)",
        'css grid': ".grid { display: grid; gap: 1rem; }",
        'async function': "async function name() {\n  const data = await fetch(url);\n  return data.json();\n}",
        'event listener': "element.addEventListener('click', (e) => {\n  console.log(e.target);\n});"
    };
    async function codeSearch(query) {
        const lower = query.toLowerCase();
        for (const [key, code] of Object.entries(codeSnippets)) {
            if (lower.includes(key)) return { found: true, title: key, code };
        }
        return { found: false };
    }
    async function generateResponse(prompt, useDual = dualMindActive) {
        if (!model1Ready) return "Primary mind offline. Please refresh the page.";
        try {
            const result1 = await generator1(prompt, { max_new_tokens: 80, temperature: 0.85 });
            let primary = result1[0].generated_text.trim();
            if (!useDual || !model2Ready) return primary;
            const secondaryPrompt = `Rewrite creatively: ${primary}\nAlternative:`;
            try {
                const result2 = await generator2(secondaryPrompt, { max_new_tokens: 60, temperature: 0.9 });
                const secondary = result2[0].generated_text.trim();
                addMessage(`🧠 Primary: ${primary}`, 'assistant', 1);
                addMessage(`🌀 Second Mind: ${secondary}`, 'assistant', 2);
                return `**Synthesized**: ${primary} (Second mind suggests: ${secondary})`;
            } catch (e2) { return primary; }
        } catch (e) { return "Error generating response."; }
    }
    async function processInput(text) {
        addMessage(text, 'user');
        conversationMemory.push({ role: 'user', content: text });
        await saveMemory(conversationMemory);
        beep('send');
        const cleanKey = text.toLowerCase().replace(/[?.,!]/g, '').trim();
        if (learnedFacts[cleanKey]) {
            addMessage(learnedFacts[cleanKey], 'assistant', 1);
            const keys = Object.keys(learnedFacts);
            const idx = keys.indexOf(cleanKey);
            if (idx >= 0) activeNeurons.add(`n${idx}`);
            setTimeout(() => activeNeurons.clear(), 1500);
            conversationMemory.push({ role: 'assistant', content: learnedFacts[cleanKey] });
            await saveMemory(conversationMemory);
            beep('receive');
            return;
        }
        let intent = forcedIntent || await classifyIntent(text);
        forcedIntent = null;
        addMessage(`🔀 Router → ${intent.toUpperCase()}`, 'system');
        let response = '';
        if (intent === 'code') {
            const codeResult = await codeSearch(text);
            if (codeResult.found) {
                response = codeResult.code;
                addMessage(response, 'code');
            } else {
                response = "No matching code snippet found.";
                addMessage(response, 'assistant', 1);
            }
        } else if (intent === 'web') {
            const webResult = await smartWebSearch(text);
            if (webResult.found) {
                response = webResult.text;
                learnedFacts[cleanKey] = `${response.slice(0, 150)} [${webResult.source || 'Web'}]`;
                await saveFacts(learnedFacts);
                refreshNeural();
                addMessage(`📖 ${webResult.title || 'Result'}: ${response}`, 'assistant', 1);
            } else {
                const prompt = `Answer concisely: ${text}`;
                response = await generateResponse(prompt);
                addMessage(response, 'assistant', 1);
            }
        } else {
            const factsStr = Object.entries(learnedFacts).map(([k, v]) => `${k}=${v}`).join('; ');
            const prompt = factsStr ? `Facts: ${factsStr}\nUser: ${text}\nAI:` : text;
            response = await generateResponse(prompt);
            addMessage(response, 'assistant', 1);
        }
        if (!learnedFacts[cleanKey] && response) {
            learnedFacts[cleanKey] = response.slice(0, 200);
            await saveFacts(learnedFacts);
        }
        conversationMemory.push({ role: 'assistant', content: response });
        await saveMemory(conversationMemory);
        beep('receive');
        futureTag.textContent = `SYNC · ${2157 + Math.floor(Math.random() * 30)}`;
        setTimeout(() => {
            const keys = Object.keys(learnedFacts);
            if (keys.length > 0) {
                const lastKey = keys[keys.length - 1];
                const idx = keys.indexOf(lastKey);
                if (idx >= 0) {
                    activeNeurons.add(`n${idx}`);
                    signalParticles.push({ x: 50, y: 100, to: `n${idx}` });
                }
            }
        }, 300);
        refreshNeural();
    }
    document.getElementById('sendBtn').addEventListener('click', async () => {
        const value = inputField.value.trim();
        if (!value) return;
        inputField.value = '';
        await processInput(value);
    });
    inputField.addEventListener('keypress', (e) => {
        if (e.key === 'Enter') document.getElementById('sendBtn').click();
    });
    dualToggle.addEventListener('click', () => {
        dualMindActive = !dualMindActive;
        dualToggle.classList.toggle('active', dualMindActive);
        dualToggle.setAttribute('aria-pressed', dualMindActive);
        dualToggle.innerHTML = dualMindActive ? '🧠 Dual Mind ON' : '🧠 Dual Mind';
        addMessage(dualMindActive ? '🌀 Second Mind activated. Collaborating...' : '🔵 Single Mind mode.', 'system');
        if (dualMindActive && !model2Ready) addMessage('⚠️ Second model not loaded yet.', 'system');
    });
    document.getElementById('btnChat').onclick = () => { forcedIntent = 'chat'; addMessage('💬 Chat mode', 'system'); };
    document.getElementById('btnCode').onclick = () => { forcedIntent = 'code'; addMessage('📟 Code mode', 'system'); };
    document.getElementById('btnWeb').onclick = () => { forcedIntent = 'web'; addMessage('🌐 Web search', 'system'); };
    document.getElementById('btnMemory').onclick = () => {
        const keys = Object.keys(learnedFacts);
        if (keys.length) addMessage(`🧠 Neural pathway: ${keys.length} neurons`, 'system');
        else addMessage('🧠 No neural pathways yet.', 'system');
    };
    document.getElementById('btnSever').onclick = () => { addMessage('⚠️ Severing link...', 'system'); beep('error'); };
    document.getElementById('btnClearChat').onclick = async () => {
        await clearMemoryDB();
        conversationMemory = [];
        outputDiv.innerHTML = '';
        addMessage('Chat cleared', 'system');
    };
    document.getElementById('btnForgetFacts').onclick = async () => {
        await clearFactsDB();
        learnedFacts = {};
        activeNeurons.clear();
        signalParticles = [];
        refreshNeural();
        addMessage('Neural pathways pruned', 'system');
    };
    document.getElementById('btnTheme').onclick = () => { document.body.style.filter = document.body.style.filter ? '' : 'hue-rotate(180deg)'; };
    document.getElementById('btnFullscreen').onclick = () => { document.getElementById('studioContainer').classList.toggle('fullscreen'); };
    document.getElementById('btnDemo').onclick = async () => {
        const demoFacts = {
            'python': 'Python is a high-level programming language created by Guido van Rossum.',
            'javascript': 'JavaScript is the scripting language for web browsers.',
            'machine learning': 'ML enables computers to learn from data without explicit programming.',
            'neural network': 'Neural networks are inspired by biological brain structures.',
            'tensorflow': 'TensorFlow is an open-source ML framework by Google.',
            'deep learning': 'Deep learning uses layered neural networks for feature extraction.'
        };
        for (const [k, v] of Object.entries(demoFacts)) { learnedFacts[k] = v; }
        await saveFacts(learnedFacts);
        refreshNeural();
        addMessage(`🎭 Neural network trained with ${Object.keys(demoFacts).length} pathways!`, 'system');
    };
    document.getElementById('muteToggle').addEventListener('click', () => {
        muted = !muted;
        document.getElementById('muteToggle').textContent = muted ? '🔇' : '🔊';
        if (!muted && !audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    });
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
            addMessage(`🧠 ${hoveredNeuron.key}: ${hoveredNeuron.value}`, 'system');
            const idx = parseInt(hoveredNeuron.id.slice(1));
            activeNeurons.add(hoveredNeuron.id);
            setTimeout(() => activeNeurons.clear(), 1500);
            beep('receive');
        }
    });
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
        await loadModels();
        inputField.focus();
        if (conversationMemory.length) addMessage(`📀 ${conversationMemory.length} messages loaded`, 'system');
        if (Object.keys(learnedFacts).length) addMessage(`🧠 ${Object.keys(learnedFacts).length} neural pathways ready`, 'system');
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        console.log('✨ MYTHIC AI · DUAL MIND — Neural network active');
    });
</script>
</body>
</html>
