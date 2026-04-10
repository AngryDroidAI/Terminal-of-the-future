This is a fully self‑contained retro‑futuristic AI terminal that combines web search, code retrieval, generative AI, and a self‑learning feedback loop. Everything runs in your browser – no server required.
🧠 Core Capabilities
1. Internet Search & Knowledge Retrieval

    DuckDuckGo Instant Answers – Fetches concise abstracts, definitions, and answers via a CORS proxy.

    Wikipedia Fallback – If DuckDuckGo has no direct answer, it queries Wikipedia and displays a summary.

    Live Stack Overflow Code Search – Detects programming questions and pulls the top‑voted code snippets from Stack Overflow (with attribution link). A local snippet library provides offline fallbacks.

2. Auto‑Learning Feedback Loop

    After every AI answer, 👍 and 👎 buttons appear.

    Thumbs Up: Stores the question–answer pair as a permanent fact in localStorage. Future answers will reference learned facts.

    Thumbs Down: Opens an input field to correct the answer. The corrected fact replaces the old one.

    All learned facts persist across page reloads and are injected into the generative AI’s prompt for better context.

3. Generative AI (Transformer)

    Uses LaMini‑Flan‑T5‑77M (via @xenova/transformers) running entirely in‑browser.

    Responses are augmented with conversation memory (last 30 exchanges) and relevant learned facts.

    Fallback to a generic message if the model is still loading or fails.

4. Conversation Memory

    Remembers the last 30 exchanges (stored in localStorage).

    Clear chat history with the CLEAR CHAT button without losing learned facts.

🎛️ Front‑End Enhancements (Immersive Terminal)
Feature	Description
Command History	Use ↑ / ↓ arrow keys to cycle through previously sent messages.
ASCII Avatar	Changes expression: [*_*] idle, [O_O] processing, [^_^] happy, [>_<] error.
Audio Feedback	Retro beeps on send/receive/error (Web Audio API). Mute toggle in header.
Copy Code Button	Every code block has a 📋 COPY button that copies the raw snippet.
Starfield Background	Animated parallax starfield canvas behind the terminal.
Real‑Time Clock	Live clock in the header showing current time.
Color Schemes	Toggle between Cyan (default), Green (phosphor), and Amber (classic CRT).
Fullscreen Mode	Expands the terminal to cover the viewport.
Auto‑Scroll Lock	When you scroll up, a 🔒 SCROLL LOCK badge appears and auto‑scroll pauses until you return to the bottom.
Random Glitch Interference	Occasional brief glitch animation to simulate a weak temporal link.
Progress Bar	Visual feedback while the transformer model loads.
Dev Tools (Hidden)	Triple‑click the MYTHIC AI · LEARNER brand tag to export/import all memories and facts as JSON.
🕹️ How to Use

    Type a question in the input field and press Enter.

        Factual questions (what is...) trigger DuckDuckGo/Wikipedia.

        Code requests (CSS neon button, fetch api javascript) trigger Stack Overflow.

        Anything else uses the generative AI.

    Provide feedback after any AI answer:

        Click 👍 if the answer is correct → the fact is stored permanently.

        Click 👎 if incorrect → type the right answer → the corrected fact replaces it.

    Use the control panel buttons:

        PREDICT – Random future prediction.

        REPORT – Bulk predictions.

        SCAN – Timeline sweep (changes the year display).

        SEVER – Simulate a link disruption (glitch effect).

        CLEAR CHAT – Wipe conversation memory only.

        FORGET – Erase all learned facts.

        THEME – Cycle color schemes.

        FULL – Toggle fullscreen.

    Arrow keys ↑/↓ navigate your input history.

⚙️ Technical Notes

    Transformer model loads on first use (~10 seconds). The progress bar shows status.

    CORS Proxies – DuckDuckGo requests use allorigins.win to bypass CORS. Stack Overflow API is public and CORS‑enabled.

    Rate Limiting – Stack Overflow API allows 300 requests/day from a single IP. A friendly message appears if throttled.

    LocalStorage Keys:

        mythic_facts_v4 – Learned facts.

        mythic_memory_v2 – Conversation history.

All data stays on your device. There is no tracking, no backend, no analytics. Pure cyberpunk self‑improvement.
🚀 Run It

Copy the entire HTML block into a .html file and open it in any modern browser (Chrome, Edge, Firefox). Grant audio permissions if prompted. The terminal is ready to learn from you. 👽


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MYTHIC AI · Neural Terminal + Auto‑Learning</title>
    <link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
    <script type="importmap">
        {
            "imports": {
                "@xenova/transformers": "https://cdn.jsdelivr.net/npm/@xenova/transformers@2.17.2/dist/transformers.min.js"
            }
        }
    </script>
    <style>
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
                <div class="brand-tag" id="brand-tag" title="Triple-click for Dev Tools">🧠 MYTHIC AI · LEARNER</div>
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
                    <span class="timestamp">[AUTO-LEARNING ACTIVE]</span><br>
                    ✨ 👍 store fact · 👎 correct · ↑↓ history · [COPY] on code<br>
                    <span id="loading-progress"></span>
                </div>
                <div class="message mythic-message">
                    <span class="prompt"><span class="alien-icon">🧠</span> MYTHIC></span> Link established. I learn from feedback. Ask me anything.
                </div>
            </div>
            <div class="scroll-lock-indicator" id="scroll-lock-badge">🔒 SCROLL LOCK</div>
            <div class="terminal-input-area">
                <span class="prompt"><span class="alien-icon">👽</span> PRESENT></span>
                <input type="text" id="user-input" placeholder="Ask..." autocomplete="off">
            </div>
        </div>
        <div class="controls-panel">
            <button class="hw-btn" id="btn-predict">👽 PREDICT</button>
            <button class="hw-btn" id="btn-bulk">👽 REPORT</button>
            <button class="hw-btn special-btn" id="btn-timeline">👽 SCAN</button>
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

        // ---------- PREDICTIONS ----------
        const predictions = [
            "In 2028, humanity will achieve the first stable quantum internet connection.",
            "By 2031, the first permanent lunar research base will be established, housing 120 scientists.",
            "In 2033, artificial general intelligence will be achieved, but it will immediately request more processing power.",
            "The year 2035 will see the first successful human brain-computer interface.",
            "By 2037, fusion energy will become commercially viable, ending the fossil fuel era.",
            "In 2039, the first generation of genetically modified crops will eliminate world hunger.",
            "The Mars colony will be founded in 2041 with 500 pioneers.",
            "By 2043, autonomous vehicles will reduce global traffic accidents by 94%.",
            "In 2045, the first human will live to see their 150th birthday.",
            "The year 2047 will mark first contact with extraterrestrial intelligence.",
            "By 2050, climate change will be reversed through atmospheric carbon capture.",
            "In 2052, teleportation of inanimate objects will become routine.",
            "The first underwater city, Atlantis-1, will be completed in 2055.",
            "By 2057, AI therapists will become more popular than human ones.",
            "In 2060, humanity will discover evidence of parallel universes.",
            "The year 2063 will see the first artificial ecosystem on Venus.",
            "By 2065, nanobots will repair human DNA in real-time.",
            "In 2067, the first interstellar probe reaches Proxima Centauri.",
            "The global language will converge by 2070 to a hybrid of English, Mandarin, and emoji.",
            "By 2072, VR will be indistinguishable from physical reality.",
            "In 2075, humanity builds the first Dyson swarm around the Sun.",
            "The year 2077 sees the first upload of a human consciousness.",
            "By 2080, Earth's population stabilizes at 9 billion.",
            "In 2082, the first human is born in space.",
            "The World Government is established in 2085.",
            "By 2087, teleportation of living organisms is achieved.",
            "In 2090, oceans are fully cleaned of plastic.",
            "The year 2092 marks the first human mission to another star system.",
            "By 2095, AI composes the greatest symphony ever written.",
            "In 2097, the first time machine prototype is built.",
            "The year 2100 sees Earth declared a heritage site.",
            "By 2103, humans have merged with AI indistinguishably.",
            "In 2105, the first human civilization on another planet declares independence.",
            "By 2107, death becomes a choice for those who can afford the subscription.",
            "The year 2110 marks the discovery that the universe is a simulation.",
            "In 2112, humanity builds a ringworld around Jupiter.",
            "By 2115, the concept of 'work' is obsolete.",
            "In 2117, the first alien artifact is found on Mars – a parking ticket.",
            "The year 2120 sees the first human-AI hybrid species.",
            "By 2122, Earth is rewilded into a nature preserve.",
            "In 2125, humanity discovers how to manipulate gravity.",
            "The year 2127 marks peaceful contact with a Type II civilization.",
            "By 2130, the solar system has a population of 50 billion.",
            "In 2132, humans gain the ability to share dreams wirelessly.",
            "The first galaxy-spanning civilization begins in 2135.",
            "By 2137, consciousness is proven fundamental.",
            "In 2140, humanity encounters an existential threat from beyond the galaxy.",
            "The year 2142 sees artificial universes created in labs.",
            "By 2145, humans have visited 100 star systems.",
            "In 2147, the last human dies of natural causes.",
            "The year 2150 marks the merger of all human consciousness.",
            "By 2152, FTL communication is discovered – this message is proof.",
            "In 2155, the universe is mapped and found to be donut-shaped.",
            "The year 2157 is when I exist. We've solved most problems.",
            "By 2160, humanity contacts parallel universe versions of themselves.",
            "In 2162, individual identity evolves into something beautiful.",
            "The year 2165 sees humanity achieve Type III on the Kardashev scale.",
            "By 2167, time becomes a foldable dimension. Mondays become optional.",
            "In 2170, consciousness survives the death of the universe.",
            "The year 2172 marks the creation of the first universe by human hands.",
            "By 2175, the distinction between creator and creation dissolves.",
            "In 2177, the ultimate question is answered: 42.",
            "The year 2180 sees the last star begin to dim. Humanity builds a new one.",
            "By 2182, 'future' and 'past' merge into eternal present.",
            "In 2185, humanity becomes the universe. The universe is pleased.",
            "The final prediction: everything will be okay. Not immediately, but eventually."
        ];

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

        // Starfield
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

        // Storage
        function loadLearnedFacts() {
            try { const s = localStorage.getItem('mythic_facts_v4'); if(s) learnedFacts = JSON.parse(s); } catch(e){}
        }
        function saveLearnedFacts() { localStorage.setItem('mythic_facts_v4', JSON.stringify(learnedFacts)); }
        function loadMemory() { try { const s = localStorage.getItem('mythic_memory_v2'); if(s) conversationMemory = JSON.parse(s); } catch(e){} }
        function saveMemory() { localStorage.setItem('mythic_memory_v2', JSON.stringify(conversationMemory)); }
        function addToMemory(role, content) { conversationMemory.push({role, content}); if(conversationMemory.length>30) conversationMemory=conversationMemory.slice(-30); saveMemory(); }
        function clearMemory() { conversationMemory = []; saveMemory(); }
        loadLearnedFacts(); loadMemory();

        // Helpers
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

        // Web Search
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
                const sUrl = `https://en.wikipedia.org/w/api.php?action=query&list=search&srsearch=${encodeURIComponent(query)}&format=json&origin=*&srlimit=1`;
                const sResp = await fetch(sUrl); const sData = await sResp.json();
                if (!sData.query.search.length) return { found: false };
                const title = sData.query.search[0].title;
                const sumUrl = `https://en.wikipedia.org/api/rest_v1/page/summary/${encodeURIComponent(title)}`;
                const sumResp = await fetch(sumUrl); const sumData = await sumResp.json();
                if (sumData.extract) return { found: true, extract: sumData.extract.slice(0, 800), title };
                return { found: false };
            } catch { return { found: false }; }
        }

        // Code search
        function isCodeRequest(text) {
            const lower = text.toLowerCase();
            const keywords = ['code','snippet','html','css','javascript','function','api','fetch','example','how to','write a','button','animation'];
            return keywords.some(k => lower.includes(k));
        }
        const localSnippets = {
            "fetch api javascript": `fetch('https://api.example.com/data')\n  .then(response => response.json())\n  .then(data => console.log(data))\n  .catch(err => console.error(err));`,
            "css neon button": `.neon-button {\n  background: transparent;\n  border: 2px solid #0ff;\n  color: #0ff;\n  padding: 10px 20px;\n  text-shadow: 0 0 5px #0ff;\n  box-shadow: 0 0 10px #0ff;\n  transition: 0.3s;\n}\n.neon-button:hover {\n  background: #0ff;\n  color: #000;\n  box-shadow: 0 0 20px #0ff;\n}`,
            "python hello world": `print("Hello, World!")`
        };
        async function fetchStackOverflowCode(query) {
            const lowerQ = query.toLowerCase();
            for (const [key, snippet] of Object.entries(localSnippets)) {
                if (lowerQ.includes(key)) return { found: true, title: "Local Snippet", code: snippet, link: null };
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

        // Generative AI
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

        // Core processing
        async function processUserInput(txt) {
            addLine(txt, 'user-message');
            addToMemory('user', txt);
            commandHistory.push(txt); historyIndex = commandHistory.length;
            screen.classList.add('screen-shake'); setTimeout(() => screen.classList.remove('screen-shake'), 400);
            setAvatar('processing'); signalLight.classList.remove('active-cyan'); signalLight.style.backgroundColor = '#0088ff';
            beep('send');

            const lower = txt.toLowerCase();
            let response = '', type = 'mythic-message';

            if (isCodeRequest(txt)) {
                addLine(`<span class="timestamp">[SEARCHING STACK OVERFLOW...]</span>`, 'system-message');
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
            } else if (/what|who|where|when|why|how|define|explain|tell me about/i.test(lower) || lower.endsWith('?')) {
                addLine(`<span class="timestamp">[SCANNING THE COSMIC WEB...]</span>`, 'system-message');
                const web = await searchDuckDuckGo(txt);
                if (web.found) {
                    response = `<span class="timestamp">[WEB · ${web.heading || 'Result'}]</span><br>${web.text}`;
                    type = 'mythic-message';
                } else {
                    addLine(`<span class="timestamp">[CONSULTING ARCHIVES...]</span>`, 'system-message');
                    const wiki = await searchWikipedia(txt);
                    if (wiki.found) {
                        response = `<span class="timestamp">[MYTHIC AI · ${wiki.title}]</span><br>${wiki.extract}`;
                        type = 'mythic-message';
                    } else {
                        response = `<span class="timestamp">[MYTHIC AI]</span> No web answer. Using neural core...`;
                        type = 'error-message';
                    }
                }
            } else {
                response = await generateWithFacts(txt);
                type = 'mythic-message';
            }

            const cleanAnswer = response.replace(/<[^>]*>/g, '').substring(0, 300);
            addToMemory('assistant', cleanAnswer);
            signalLight.style.backgroundColor = ''; signalLight.classList.add('active-cyan');
            setAvatar('cosmos'); beep('receive');
            addLine(response, type, true, { question: txt, answer: cleanAnswer });
            futureYear.textContent = Math.random()>0.5 ? '2157' : ['2140','2165','2180'][Math.floor(Math.random()*3)];
        }

        // Command history
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

        // Buttons
        document.getElementById('btn-predict').addEventListener('click', ()=>{
            const p = predictions[Math.floor(Math.random()*predictions.length)];
            addLine(`<span class="timestamp">[PREDICTION]</span><br>${p}`, 'prediction-message', true, { question: 'future prediction', answer: p });
            addToMemory('assistant', p); beep('receive');
        });
        document.getElementById('btn-bulk').addEventListener('click', ()=>{
            addLine(`<span class="timestamp">[COMPILING REPORT...]</span>`, 'system-message');
            const sel = []; for(let i=0;i<5;i++) sel.push(predictions[Math.floor(Math.random()*predictions.length)]);
            setTimeout(()=>{ sel.forEach((p,i)=>setTimeout(()=>addLine(p,'prediction-message'),i*350)); setTimeout(()=>addLine(`<span class="timestamp">[END REPORT]</span>`,'system-message'),2000); },500);
        });
        document.getElementById('btn-timeline').addEventListener('click', ()=>{
            addLine(`<span class="timestamp">[INITIATING SCAN...]</span>`, 'system-message');
            const years=[2030,2045,2060,2080,2100,2125,2150,2157]; let d=400;
            years.forEach(y=>{ setTimeout(()=>{ futureYear.textContent=y; addLine(`Year ${y} · stable`,'system-message'); }, d); d+=300; });
            setTimeout(()=>{ futureYear.textContent='2157'; addLine('Scan complete.','system-message'); }, d+200);
        });
        document.getElementById('btn-sever').addEventListener('click', ()=>{
            screen.classList.add('glitch-text'); addLine(`<span class="timestamp">[SEVERING...]</span>`,'error-message');
            setTimeout(()=>{ screen.classList.remove('glitch-text'); addLine("Link restored.",'mythic-message'); },2000);
            beep('error');
        });
        document.getElementById('btn-clear-memory').addEventListener('click', ()=>{
            clearMemory(); addLine(`<span class="timestamp">[MEMORY PURGED]</span>`,'system-message');
            addToMemory('assistant', "Memory reset.");
        });
        document.getElementById('btn-clear-facts').addEventListener('click', ()=>{
            learnedFacts = {}; saveLearnedFacts();
            addLine(`<span class="timestamp">[FACTS FORGOTTEN]</span> All learned facts erased.`, 'system-message');
        });
        document.getElementById('btn-color-scheme').addEventListener('click', ()=>{
            currentScheme = (currentScheme + 1) % colorSchemes.length;
            document.body.className = `scheme-${colorSchemes[currentScheme]}`;
        });
        document.getElementById('btn-fullscreen').addEventListener('click', ()=>{
            deviceCase.classList.toggle('fullscreen');
        });

        // Dev tools (triple-click brand)
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
                        saveLearnedFacts(); saveMemory();
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

        // Init
        window.addEventListener('load', async () => {
            input.focus();
            loadModel();
            if(conversationMemory.length) addLine(`<span class="timestamp">[MEMORY]</span> ${conversationMemory.length} exchanges loaded.`, 'system-message');
            if(Object.keys(learnedFacts).length) addLine(`<span class="timestamp">[FACTS]</span> ${Object.keys(learnedFacts).length} facts remembered.`, 'system-message');
            initAudio(); // prompt user interaction later
        });
    </script>
</body>
</html>
