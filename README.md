Temporal Link · Mythic AI static terminal—a fully client‑side, generative AI experience with memory, code retrieval, and web search, deployable on any static host like Neocities.
🧠 Core Concept

Temporal Link is a single HTML file that emulates a futuristic AI terminal from the year 2157. It requires no backend, no server, and no API keys. Everything runs in the user's browser using JavaScript, WebAssembly, and public APIs. The interface mimics a CRT monitor with scanlines, glitch effects, and a retro‑futuristic aesthetic.
⚙️ Key Features (What It Can Do)
1. Persistent Conversational Memory

    What it does: Remembers the last 30 exchanges between the user and the AI.

    How it works: Stores conversation history in the browser's localStorage. On page reload, the memory is restored, allowing the AI to reference past topics and maintain context.

    Benefit: Creates a coherent, continuous conversation rather than isolated Q&A.

2. Generative AI (Neural Responses)

    What it does: Produces novel, creative answers using a transformer model running locally.

    How it works: Uses Transformers.js to load and execute the LaMini-Flan-T5-77M model entirely in the browser (via WebAssembly). The model receives a prompt built from recent conversation history plus the user's new message.

    Benefit: Handles open‑ended questions, small talk, and generates human‑like predictions without calling any external AI service.

3. Code Snippet Retrieval (Stack Overflow)

    What it does: When the user asks for a code example (e.g., "CSS neon button" or "JavaScript fetch example"), the terminal fetches and displays actual code blocks from Stack Overflow.

    How it works:

        Detects code‑related keywords in the user's input.

        Queries the Stack Exchange API for relevant questions.

        Extracts code blocks using a flexible regex that handles HTML entities and tag attributes.

        Displays the formatted code in a dedicated monospaced block.

    Custom Rate‑Limit Message: If the free API limit (300 requests/day per IP) is exceeded, the terminal shows a playful in‑universe message: "Temporal bandwidth exceeded. To preserve the fabric of this reality, the code archives are temporarily sealed."

4. Internet Search (DuckDuckGo)

    What it does: Searches the web for factual answers and presents the most relevant result.

    How it works:

        Uses the DuckDuckGo Instant Answer API.

        To bypass CORS restrictions, requests are routed through a public CORS proxy (allorigins.win).

        Returns concise abstracts, definitions, or answers.

    Priority: The terminal tries DuckDuckGo first for knowledge questions. If no result is found, it falls back to Wikipedia.

5. Knowledge Base Fallback (Wikipedia)

    What it does: Provides detailed summaries when DuckDuckGo returns nothing.

    How it works: Queries the Wikipedia API for article summaries (up to 800 characters).

    Integration: Presented as "MYTHIC AI" responses, maintaining the futuristic theme.

6. Pre‑Loaded Predictions & Timeline Features

    Full Predictions Array: Over 65 imaginative predictions about the future (2028–2185) hardcoded into the script.

    Buttons:

        Generate Prediction: Displays a random prediction.

        Future Report: Compiles a 5‑prediction "report."

        Timeline Scan: Simulates scanning multiple future years with signal status updates.

        Sever Link: A glitchy, dramatic "disconnection" sequence (purely aesthetic).

        Clear Memory: Purges the conversation history from both memory and localStorage.

🎨 User Interface & Styling

    Retro‑Futuristic Design: CRT scanlines, glowing neon borders, and monospaced VT323 font.

    Dynamic ASCII Avatar: The alien face changes expression (idle, processing, cosmos, happy) based on the terminal's state.

    Message Types: Each response is color‑coded and prefixed with a role:

        PRESENT> – User messages (cyan)

        MYTHIC> – AI knowledge responses (purple border)

        CODE> – Code snippets (green border, dark background)

        PRED> – Predictions (teal)

        SYS> – System status (amber)

        ERROR> – Errors (red)

    Visual Feedback: Screen shake on input, blinking status lights, and a scanning line animation.

🔧 Technical Architecture (How It All Fits Together)
Component	Technology	Purpose
Generative AI	Transformers.js + ONNX Runtime Web	Runs a small language model locally
Memory	localStorage	Persists conversation across sessions
Code Fetching	Stack Exchange API	Retrieves code snippets from Stack Overflow
Web Search	DuckDuckGo API + CORS proxy	Fetches instant answers from the web
Knowledge	Wikipedia REST API	Provides article summaries as fallback
UI	HTML5, CSS3, JavaScript (ES Modules)	Terminal interface and interactions
Routing Logic (Simplified)
text

User Input
    │
    ├─ Contains "code" / "css" / "javascript" ? → Stack Overflow
    │
    ├─ Is a question (what/who/why/how) ? → DuckDuckGo → (fallback) Wikipedia
    │
    └─ Otherwise → Generative AI (with memory context)

🌐 Deployment & Limitations

    Zero‑Cost Hosting: Works on Neocities, GitHub Pages, or any static file server.

    Privacy: No data leaves the browser except public API calls (Stack Overflow, DuckDuckGo, Wikipedia).

    Limitations:

        The local AI model requires ~200 MB of RAM and may take 10–15 seconds to load on first visit.

        DuckDuckGo search uses a third‑party CORS proxy, which may occasionally be rate‑limited or slow.

        Stack Overflow API is limited to 300 requests/day per IP (unauthenticated).

🚀 Summary

Temporal Link is a demonstration of how far static web technologies have come. It combines:

    Learned patterns (via the transformer model)

    Generalization (handling unseen inputs)

    Generative behavior (novel responses)

    Internet‑connected knowledge (DuckDuckGo & Wikipedia)

    Code retrieval (Stack Overflow)

All within a single, portable HTML file that requires no backend, no API keys, and no cost to deploy. It's a fully functional, memory‑enhanced AI assistant that fits the "Mythic AI" aesthetic and runs entirely on the user's device.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Temporal Link · Full Memory + Internet Search</title>
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
        body { background: #050510; display: flex; justify-content: center; align-items: flex-start; min-height: 100vh; padding: 20px; overflow-y: auto; }
        .device-case {
            background: linear-gradient(145deg, #1a1a2e, #0f0f1a); padding: 25px; border-radius: 15px;
            box-shadow: 0 0 0 3px #2a2a4a, 0 0 30px rgba(0,150,255,0.2), 0 15px 35px rgba(0,0,0,0.9), inset 0 0 30px rgba(0,0,0,0.5);
            width: 100%; max-width: 1000px; position: relative; border: 1px solid #3a3a5a; margin: 20px 0;
        }
        .screw { position: absolute; width: 14px; height: 14px; background: linear-gradient(45deg, #4a4a6a, #2a2a3a); border-radius: 50%; box-shadow: inset 1px 1px 2px rgba(0,0,0,0.5); z-index: 5; }
        .screw::after { content: ''; position: absolute; top: 50%; left: 50%; width: 9px; height: 2px; background: #1a1a2a; transform: translate(-50%, -50%) rotate(45deg); }
        .screw::before { content: ''; position: absolute; top: 50%; left: 50%; width: 9px; height: 2px; background: #1a1a2a; transform: translate(-50%, -50%) rotate(-45deg); }
        .tl { top: 15px; left: 15px; } .tr { top: 15px; right: 15px; } .bl { bottom: 15px; left: 15px; } .br { bottom: 15px; right: 15px; }
        .terminal-container {
            background: #000a1a; border: 4px solid #001a3a; border-radius: 4px;
            box-shadow: inset 0 0 30px rgba(0,150,255,0.15), 0 0 20px rgba(0,100,200,0.3);
            overflow: visible; position: relative; display: flex; flex-direction: column; min-height: 400px;
        }
        .scanline { position: absolute; top: 0; left: 0; width: 100%; height: 8px; background: linear-gradient(180deg, transparent, rgba(0,200,255,0.15), transparent); opacity: 0.5; animation: scanline 4s linear infinite; pointer-events: none; z-index: 4; }
        .crt-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: repeating-linear-gradient(0deg, rgba(0,0,0,0.12), rgba(0,0,0,0.12) 1px, transparent 1px, transparent 2px); pointer-events: none; z-index: 3; box-shadow: inset 0 0 60px rgba(0,0,0,0.8); }
        @keyframes scanline { 0% { top: -10%; } 100% { top: 110%; } }
        .terminal-header {
            background: linear-gradient(90deg, #001a3a, #002a5a, #001a3a); padding: 12px 15px;
            border-bottom: 2px solid #004488; display: flex; justify-content: space-between; align-items: center;
            z-index: 2; flex-wrap: wrap; gap: 5px;
        }
        .brand-tag { font-size: 1.3rem; color: #00ccff; background: #000a1a; padding: 3px 12px; border: 1px solid #00ccff; font-weight: bold; letter-spacing: 2px; text-shadow: 0 0 8px #00ccff; }
        .temporal-status { display: flex; flex-direction: column; align-items: center; color: #00aaff; font-size: 1.1rem; text-shadow: 0 0 5px #00aaff; }
        .temporal-status .year { font-size: 1.6rem; color: #00ffff; text-shadow: 0 0 10px #00ffff; }
        .avatar-display { font-size: 14px; color: #00ccff; text-align: center; line-height: 1.1; text-shadow: 0 0 5px #00ccff; white-space: pre; font-family: 'VT323', monospace; min-width: 120px; transition: all 0.3s; }
        .status-container { display: flex; gap: 10px; }
        .status-light { width: 14px; height: 14px; border-radius: 50%; background: #1a1a2a; border: 1px solid #000; }
        .status-light.active-cyan { background: #00ffff; box-shadow: 0 0 12px #00ffff; animation: pulse 1s infinite alternate; }
        .status-light.active-blue { background: #0088ff; box-shadow: 0 0 10px #0088ff; }
        .status-light.active-amber { background: #ffaa00; box-shadow: 0 0 10px #ffaa00; animation: pulse 0.3s infinite alternate; }
        @keyframes pulse { from { opacity: 0.6; } to { opacity: 1; } }
        .terminal-body { flex: 1; padding: 20px; color: #00ddff; font-size: 1.5rem; line-height: 1.5; text-shadow: 0 0 5px rgba(0,220,255,0.7); min-height: 300px; }
        .message { margin-bottom: 14px; word-wrap: break-word; animation: fadeIn 0.3s; padding: 6px 12px; background: rgba(0,20,40,0.4); border-radius: 6px; border-left: 2px solid #0066aa; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
        .user-message { color: #00ffff; text-shadow: 0 0 8px #00ffff; border-left-color: #00aaff; }
        .future-message { color: #a0f0ff; text-shadow: 0 0 5px rgba(160,240,255,0.8); }
        .system-message { color: #ffbb44; font-weight: bold; text-shadow: 0 0 6px rgba(255,170,0,0.7); }
        .error-message { color: #ff6666; text-shadow: 0 0 6px rgba(255,100,100,0.7); }
        .prediction-message { color: #88ffcc; text-shadow: 0 0 5px rgba(136,255,204,0.7); border-left: 3px solid #00aa88; padding-left: 10px; margin-left: 5px; }
        .mythic-message { color: #d0e8ff; border-left: 4px solid #aa88ff; padding-left: 14px; margin: 10px 0; background: #00183050; text-shadow: 0 0 6px rgba(170,136,255,0.5); }
        .code-message { border-left: 4px solid #00ccaa; background: #0a1a2a; padding: 12px; font-family: 'VT323', monospace; white-space: pre-wrap; color: #aaffaa; margin: 10px 0; border-radius: 6px; }
        .prompt { opacity: 0.8; margin-right: 10px; font-weight: bold; color: #88ddff; }
        .alien-icon { display: inline-block; margin-right: 4px; filter: drop-shadow(0 0 4px #88ff88); }
        .terminal-input-area { background: #000a1a; padding: 15px; border-top: 2px solid #003a6a; display: flex; align-items: center; }
        #user-input { flex: 1; background: transparent; border: none; color: #00ffff; font-size: 1.6rem; outline: none; font-family: 'VT323', monospace; text-shadow: 0 0 5px #00ffff; }
        .controls-panel { margin-top: 15px; display: flex; justify-content: space-between; gap: 10px; padding: 12px; background: #0a0a1a; border-radius: 5px; border: 1px solid #2a2a4a; flex-wrap: wrap; }
        .hw-btn { flex: 1; background: linear-gradient(180deg, #2a2a4a, #1a1a2e); border: 1px solid #000; color: #9ac0dd; padding: 14px; font-family: 'VT323', monospace; font-size: 1.3rem; cursor: pointer; text-transform: uppercase; border-radius: 4px; box-shadow: 0 4px 0 #000; transition: all 0.1s; text-align: center; letter-spacing: 1px; font-weight: bold; min-width: 120px; }
        .hw-btn:active { transform: translateY(4px); box-shadow: 0 0 0 #000; color: #fff; }
        .hw-btn:hover { color: #00ffff; background: linear-gradient(180deg, #3a3a5a, #2a2a3e); box-shadow: 0 0 15px rgba(0,200,255,0.3); }
        .danger-btn { color: #ff8888; border-color: #5a2a2a; background: linear-gradient(180deg, #3a1a1a, #2a0a0a); }
        .danger-btn:hover { color: #ff4444; text-shadow: 0 0 8px #ff4444; background: linear-gradient(180deg, #4a2a2a, #3a1a1a); }
        .special-btn { color: #eebb44; border-color: #5a4a00; background: linear-gradient(180deg, #3a3a1a, #2a2a0a); }
        .special-btn:hover { color: #ffcc00; text-shadow: 0 0 8px #ffcc00; background: linear-gradient(180deg, #4a4a2a, #3a3a1a); }
        .screen-shake { animation: shake 0.4s cubic-bezier(.36,.07,.19,.97) both; }
        @keyframes shake { 10%,90%{ transform: translate3d(-1px,0,0); } 20%,80%{ transform: translate3d(3px,0,0); } 30%,50%,70%{ transform: translate3d(-5px,0,0); } 40%,60%{ transform: translate3d(5px,0,0); } }
        .glitch-text { animation: glitch-anim 0.2s infinite; }
        @keyframes glitch-anim { 0%{ transform: translate(0) } 20%{ transform: translate(-2px,2px) } 40%{ transform: translate(-2px,-2px) } 60%{ transform: translate(2px,2px) } 80%{ transform: translate(2px,-2px) } 100%{ transform: translate(0) } }
        .cursor { display: inline-block; width: 10px; height: 1.2em; background: #00ffff; animation: blink 0.8s infinite; margin-left: 3px; vertical-align: text-bottom; }
        @keyframes blink { 0%,100%{ opacity: 1; } 50%{ opacity: 0; } }
        .signal-wave { display: inline-block; animation: wave 1s ease-in-out infinite; }
        @keyframes wave { 0%,100%{ transform: scaleY(1); } 50%{ transform: scaleY(1.5); } }
        .timestamp { color: #88bbdd; font-size: 1.2rem; opacity: 0.9; font-weight: bold; }
    </style>
</head>
<body>
    <div class="device-case">
        <div class="screw tl"></div><div class="screw tr"></div><div class="screw bl"></div><div class="screw br"></div>
        <div class="terminal-container" id="terminal-screen">
            <div class="scanline"></div><div class="crt-overlay"></div>
            <div class="terminal-header">
                <div class="brand-tag">👽 NEURAL-LINK v4.0 · WEB SEARCH</div>
                <div class="temporal-status"><div>SIGNAL FROM</div><div class="year" id="future-year">2157</div></div>
                <div class="avatar-display" id="avatar-face"></div>
                <div class="status-container">
                    <div class="status-light active-cyan" id="signal-light"></div>
                    <div class="status-light active-blue" id="sync-light"></div>
                    <div class="status-light" id="warn-light"></div>
                </div>
            </div>
            <div class="terminal-body" id="output">
                <div class="message system-message">
                    <span class="timestamp">[2026-04-09 12:00:00 UTC]</span><br>
                    MEMORY PERSISTENCE · DUCKDUCKGO SEARCH · STACK OVERFLOW<br>
                    WIKIPEDIA FALLBACK · GENERATIVE AI · CUSTOM RATE LIMIT<br>
                    <span class="signal-wave">▂▃▅▇</span> ASK ANYTHING · I SEARCH THE WEB <span class="signal-wave">▇▅▃▂</span>
                </div>
                <div class="message mythic-message">
                    <span class="prompt"><span class="alien-icon">👽</span> MYTHIC></span> Link established. I can search the internet, fetch code, and remember our conversations. Ask away.
                </div>
            </div>
            <div class="terminal-input-area">
                <span class="prompt"><span class="alien-icon">👽</span> PRESENT></span>
                <input type="text" id="user-input" placeholder="Ask me anything... I search the web." autocomplete="off">
            </div>
        </div>
        <div class="controls-panel">
            <button class="hw-btn" id="btn-predict">👽 GENERATE PREDICTION</button>
            <button class="hw-btn" id="btn-bulk">👽 FUTURE REPORT</button>
            <button class="hw-btn special-btn" id="btn-timeline">👽 TIMELINE SCAN</button>
            <button class="hw-btn danger-btn" id="btn-sever">👽 SEVER LINK</button>
            <button class="hw-btn" id="btn-clear-memory">🧠 CLEAR MEMORY</button>
        </div>
    </div>

    <script type="module">
        import { pipeline, env } from '@xenova/transformers';

        env.allowLocalModels = false;
        env.useBrowserCache = true;

        // ---------- FULL PREDICTIONS ----------
        const predictions = [
            "In 2028, humanity will achieve the first stable quantum internet connection, linking computers across continents instantaneously.",
            "By 2031, the first permanent lunar research base will be established, housing 120 scientists and engineers.",
            "In 2033, artificial general intelligence will be achieved, but it will immediately request more processing power and a nap.",
            "The year 2035 will see the first successful human brain-computer interface allowing direct thought-to-text communication.",
            "By 2037, fusion energy will become commercially viable, ending the fossil fuel era within a decade.",
            "In 2039, the first generation of genetically modified crops will eliminate world hunger for the first time in history.",
            "The Mars colony will be founded in 2041 with 500 pioneers. They will immediately complain about the dust.",
            "By 2043, autonomous vehicles will reduce global traffic accidents by 94%. Humans will miss the joy of road rage.",
            "In 2045, the first human will live to see their 150th birthday thanks to cellular regeneration therapy.",
            "The year 2047 will mark the first contact with extraterrestrial intelligence. They will be disappointed in our reality TV.",
            "By 2050, climate change will be reversed through atmospheric carbon capture. The Earth will thank us, eventually.",
            "In 2052, teleportation of inanimate objects will become routine. Your Amazon packages will arrive before you order them.",
            "The first underwater city, Atlantis-1, will be completed in 2055 beneath the Pacific Ocean.",
            "By 2057, AI therapists will become more popular than human ones. They never judge, they just optimize.",
            "In 2060, humanity will discover evidence of parallel universes. In one of them, you're a billionaire.",
            "The year 2063 will see the creation of the first artificial ecosystem on Venus, floating in its upper atmosphere.",
            "By 2065, nanobots will be able to repair human DNA in real-time. Aging will become optional.",
            "In 2067, the first interstellar probe will reach Proxima Centauri and send back images of an Earth-like world.",
            "The global language will converge to a hybrid of English, Mandarin, and emoji by 2070.",
            "By 2072, virtual reality will be indistinguishable from physical reality. People will start forgetting which is which.",
            "In 2075, humanity will build the first Dyson swarm around the Sun, capturing 1% of its total energy output.",
            "The year 2077 will see the first successful upload of a human consciousness to a digital substrate. They'll ask for a reboot.",
            "By 2080, Earth's population will stabilize at 9 billion as space colonization accelerates.",
            "In 2082, the first human will be born in space. They will be taller than everyone on Earth.",
            "The World Government will be established in 2085 after decades of bickering. The first law will be about proper queue etiquette.",
            "By 2087, teleportation of living organisms will be achieved. A cat will be the first subject. It will be unimpressed.",
            "In 2090, the oceans will be fully cleaned of plastic through autonomous nanobot swarms.",
            "The year 2092 will mark the first human mission to another star system using a breakthrough warp drive.",
            "By 2095, AI will compose the greatest symphony ever written. Humans will weep at its beauty.",
            "In 2097, the first time machine prototype will be built. It can only send data 131 years into the past. That's how you're reading this.",
            "The year 2100 will see Earth declared a heritage site as humanity expands across the solar system.",
            "By 2103, humans will have merged with AI to such a degree that the distinction will be meaningless.",
            "In 2105, the first human civilization on another planet will declare independence. Earth will be proud.",
            "By 2107, death will become a choice for those who can afford the consciousness backup subscription.",
            "The year 2110 will mark the discovery that the universe is a simulation. The response will be a collective 'meh'.",
            "In 2112, humanity will build a ringworld around Jupiter. The view will be spectacular.",
            "By 2115, the concept of 'work' will be obsolete. Humans will pursue art, exploration, and competitive napping.",
            "In 2117, the first alien artifact will be found on Mars. It will be a parking ticket.",
            "The year 2120 will see the first human-AI hybrid species. They will be very good at math and very bad at small talk.",
            "By 2122, Earth will be rewilded into a nature preserve. Cities will move to orbit.",
            "In 2125, humanity will discover how to manipulate gravity. Flying cars will finally happen.",
            "The year 2127 will mark the first peaceful contact with a Type II civilization. They will offer us technology in exchange for our music.",
            "By 2130, the solar system will have a population of 50 billion, mostly in orbital habitats.",
            "In 2132, humans will gain the ability to share dreams wirelessly. Nightmares will become a group activity.",
            "The first galaxy-spanning civilization will begin in 2135 with the invention of the Alcubierre drive.",
            "By 2137, consciousness will be proven to be a fundamental property of the universe, like gravity.",
            "In 2140, humanity will encounter its first existential threat from beyond the galaxy. We will defeat it with mathematics.",
            "The year 2142 will see the creation of artificial universes in laboratory conditions.",
            "By 2145, humans will have visited 100 star systems. Tourism will be the largest industry.",
            "In 2147, the last human will die of natural causes. Everyone else will have uploaded by then.",
            "The year 2150 will mark the merger of all human consciousness into a single entity. It will be lonely and create friends.",
            "By 2152, humanity will discover the secret to faster-than-light communication. This message is proof.",
            "In 2155, the universe will be mapped in its entirety. We will find that it is shaped like a donut.",
            "The year 2157 is when I exist. We have solved most problems. We still argue about pizza toppings.",
            "By 2160, humanity will make contact with parallel universe versions of ourselves. They will all be slightly disappointed.",
            "In 2162, the concept of individual identity will evolve into something beautiful and incomprehensible.",
            "The year 2165 will see the first human civilization achieve Type III status on the Kardashev scale.",
            "By 2167, time will be understood as a dimension that can be folded. Mondays will become optional.",
            "In 2170, humanity will discover that consciousness survives the death of the universe. There will be a sequel.",
            "The year 2172 will mark the creation of the first universe by human hands. It will contain better physics.",
            "By 2175, the distinction between creator and creation will dissolve entirely. Everything will be everything.",
            "In 2177, humanity will finally answer the ultimate question. The answer will be 42. We already knew.",
            "The year 2180 will see the last star in the universe begin to dim. Humanity will build a new one.",
            "By 2182, the concept of 'future' and 'past' will merge into a single eternal present.",
            "In 2185, humanity will become the universe. The universe will be pleased.",
            "The final prediction: everything will be okay. Not immediately, but eventually. Always eventually."
        ];

        // ---------- MEMORY ----------
        let conversationMemory = [];
        const STORAGE_KEY = 'temporal_link_memory';
        function loadMemory() { try { const s = localStorage.getItem(STORAGE_KEY); if (s) conversationMemory = JSON.parse(s); } catch(e){} }
        function saveMemory() { try { localStorage.setItem(STORAGE_KEY, JSON.stringify(conversationMemory)); } catch(e){} }
        function addToMemory(role, content) { conversationMemory.push({role, content}); if(conversationMemory.length>30) conversationMemory=conversationMemory.slice(-30); saveMemory(); }
        function clearMemory() { conversationMemory = []; saveMemory(); }
        loadMemory();

        // DOM
        const output = document.getElementById('output');
        const input = document.getElementById('user-input');
        const futureYear = document.getElementById('future-year');
        const warnLight = document.getElementById('warn-light');
        const screen = document.getElementById('terminal-screen');
        const signalLight = document.getElementById('signal-light');
        const avatar = document.getElementById('avatar-face');

        const avatarFaces = {
            idle: `    .     .       .  .   . .   .   . .    +  .\n  .     .  :     .    .. :. .___---------___.\n       .  .   .    .  :.:. _".^ .^ ^.  '.. :"-_. .\n    .  :       .  .  .:../:            . .^  :.:\\.\n        .   . :: +. :.:/: .   .    .        . . .:\\\n .  :    .     . _ :::/:               .  ^ .  . .:\\\n  .. . .   . - : :.:./.                        .  .:\\\n  .      .     . :..|:                    .  .  ^. .:|\n    .       . : : ..||        .                . . !:|\n  .     . . . ::. ::\\(                           . :)/\n .   .     : . : .:.|. ######              .#######::|\n  :.. .  :-  : .:  ::|.#######           ..########:|\n .  .  .  ..  .  .. :\\ ########          :######## :/\n  .        .+ :: : -.:\\ ########       . ########.:/\n    .  .+   . . . . :.:\\. #######       #######..:/\n      :: . . . . ::.:..:.\\           .   .   ..:/\n   .   .   .  .. :  -::::.\\.       | |     . .:/\n      .  :  .  .  .-:.":.::.\\             ..:/\n .      -.   . . . .: .:::.:.\\.           .:/\n.   .   .  :      : ....::_:..:\\   ___.  :/\n   .   .  .   .:. .. .  .: :.:.:\\       :/`,
            processing: `    .     .       .  .   . .   .   . .    +  .\n  .     .  :     .    .. :. .___---------___.\n       .  .   .    .  :.:. _".^ .^ ^.  '.. :"-_. .\n    .  :       .  .  .:../:            . .^  :.:\\.\n        .   . :: +. :.:/: .   .   [O_O]  . . .:\\\n ...`,
            cosmos: `    .     .       .  .   . .   .   . .    +  .\n  .     .  :     .    .. :. .___---------___.\n       .  .   .    .  :.:. _".^ .^ ^.  '.. :"-_. .\n    .  :       .  .  .:../:            . .^  :.:\\.\n        .   . :: +. :.:/: .   .   *-*    . . . .:\\\n ...`,
            happy: `    .     .       .  .   . .   .   . .    +  .\n  .     .  :     .    .. :. .___---------___.\n       .  .   .    .  :.:. _".^ .^ ^.  '.. :"-_. .\n    .  :       .  .  .:../:            . .^  :.:\\.\n        .   . :: +. :.:/: .   .   ^_^    . . . .:\\\n ...`
        };
        avatar.textContent = avatarFaces.idle;

        // ---------- GENERATIVE AI ----------
        let generator = null, modelReady = false;
        async function loadModel() {
            if (generator) return true;
            addLine(`<span class="timestamp">[LOADING NEURAL CORE...]</span> Initializing transformer (~10s)`, 'system-message');
            setAvatar('processing');
            try {
                generator = await pipeline('text2text-generation', 'Xenova/LaMini-Flan-T5-77M');
                modelReady = true;
                addLine(`<span class="timestamp">[NEURAL CORE ONLINE]</span>`, 'system-message');
                setAvatar('cosmos');
                return true;
            } catch(e) { modelReady = false; addLine(`<span class="timestamp">[ERROR]</span> Model failed.`, 'error-message'); setAvatar('idle'); return false; }
        }
        function buildPrompt(userMessage) {
            let p = "You are a futuristic AI from 2157 with memory. Respond helpfully, creatively.\n";
            conversationMemory.slice(-12).forEach(t => p += `${t.role==='user'?'Human':'AI'}: ${t.content}\n`);
            return p + `Human: ${userMessage}\nAI:`;
        }
        async function generateWithMemory(msg) {
            if(!modelReady) return getFallback(msg);
            try {
                const prompt = buildPrompt(msg);
                const res = await generator(prompt, {max_new_tokens:80, temperature:0.85, do_sample:true});
                let gen = res[0].generated_text.trim().replace(/^AI:\s*/i,'');
                return gen || getFallback(msg);
            } catch { return getFallback(msg); }
        }
        function getFallback(msg) { return `I remember: "${msg}". The future holds wonders.`; }

        // ---------- CODE DETECTION & STACK OVERFLOW (with custom rate limit msg) ----------
        function isCodeRequest(text) {
            const lower = text.toLowerCase();
            const ind = ['code','snippet','example','html','css','javascript','js','how to make','show me a','write a','function','button','animation','fetch','api'];
            return ind.some(k => lower.includes(k));
        }
        async function fetchStackOverflowCode(query) {
            const url = `https://api.stackexchange.com/2.3/search/advanced?order=desc&sort=relevance&q=${encodeURIComponent(query)}&site=stackoverflow&filter=withbody`;
            try {
                const resp = await fetch(url);
                const data = await resp.json();
                // Custom rate limit message
                if (data.error_id === 502 && data.error_name === 'throttle_violation') {
                    return { found: false, message: "Temporal bandwidth exceeded. To preserve the fabric of this reality, the code archives are temporarily sealed. Try again later, traveler." };
                }
                if (!data.items || data.items.length === 0) return { found: false, message: "No matching questions on Stack Overflow." };
                for (const item of data.items.slice(0,3)) {
                    const body = item.body;
                    const codeRegex = /<pre[^>]*><code[^>]*>([\s\S]*?)<\/code><\/pre>/gi;
                    const matches = [...body.matchAll(codeRegex)];
                    if (matches.length) {
                        let snippet = '';
                        for (const m of matches) {
                            let code = m[1].replace(/&lt;/g,'<').replace(/&gt;/g,'>').replace(/&amp;/g,'&').replace(/&quot;/g,'"').replace(/&#39;/g,"'").replace(/<[^>]*>/g,'');
                            snippet += code + '\n\n';
                        }
                        return { found: true, title: item.title, code: snippet.trim() };
                    }
                }
                return { found: false, message: "Found questions but couldn't extract code. Try a more specific query (e.g., 'CSS neon button')." };
            } catch(e) {
                return { found: false, message: "Failed to reach Stack Overflow." };
            }
        }

        // ---------- DUCKDUCKGO INTERNET SEARCH (via CORS proxy) ----------
        async function searchDuckDuckGo(query) {
            const proxy = 'https://api.allorigins.win/raw?url=';
            const apiUrl = `https://api.duckduckgo.com/?q=${encodeURIComponent(query)}&format=json&no_html=1&skip_disambig=1`;
            try {
                const resp = await fetch(proxy + encodeURIComponent(apiUrl));
                const data = await resp.json();
                if (data.Abstract && data.Abstract.length > 0) {
                    return { found: true, source: 'DuckDuckGo', text: data.Abstract, heading: data.Heading };
                } else if (data.Definition && data.Definition.length > 0) {
                    return { found: true, source: 'DuckDuckGo', text: data.Definition, heading: data.Heading };
                } else if (data.Answer && data.Answer.length > 0) {
                    return { found: true, source: 'DuckDuckGo', text: data.Answer, heading: data.Heading };
                }
                return { found: false, message: "The cosmic web yields no direct answer." };
            } catch(e) {
                return { found: false, message: "Search probe lost in a temporal rift." };
            }
        }

        // ---------- WIKIPEDIA FALLBACK ----------
        async function searchWikipedia(query) {
            try {
                const sUrl = `https://en.wikipedia.org/w/api.php?action=query&list=search&srsearch=${encodeURIComponent(query)}&format=json&origin=*&srlimit=1`;
                const sResp = await fetch(sUrl); const sData = await sResp.json();
                if (!sData.query.search.length) return { found: false, message: `No entry for "${query}".` };
                const title = sData.query.search[0].title;
                const sumUrl = `https://en.wikipedia.org/api/rest_v1/page/summary/${encodeURIComponent(title)}`;
                const sumResp = await fetch(sumUrl); const sumData = await sumResp.json();
                if (sumData.type === 'disambiguation') return { found: false, message: `"${query}" is ambiguous.` };
                if (sumData.extract) {
                    let extract = sumData.extract; if(extract.length>800) extract=extract.substring(0,800)+'...';
                    return { found: true, title: sumData.title, extract };
                }
                return { found: false, message: `No summary for "${query}".` };
            } catch { return { found: false, message: "Wikipedia lookup failed." }; }
        }

        // ---------- UI HELPERS ----------
        function scrollToBottom() { window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' }); }
        function flashWarning() { warnLight.classList.add('active-amber'); setTimeout(() => warnLight.classList.remove('active-amber'), 500); }
        function setAvatar(face) { avatar.textContent = avatarFaces[face] || avatarFaces.idle; }
        function addLine(text, type='future-message', isHTML=false) {
            const div = document.createElement('div'); div.className = `message ${type}`;
            const prompts = {'user-message':'PRESENT>','system-message':'SYS>','prediction-message':'PRED>','mythic-message':'MYTHIC>','code-message':'CODE>','error-message':'ERROR>'};
            const prompt = prompts[type] || 'FUTURE>';
            if (type==='system-message'||type==='prediction-message'||type==='mythic-message'||type==='code-message'||type==='error-message') div.innerHTML = text;
            else div.innerHTML = `<span class="prompt"><span class="alien-icon">👽</span> ${prompt}</span> ${text}`;
            output.appendChild(div); scrollToBottom(); flashWarning();
        }
        function escapeHtml(t) { return t.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }

        // ---------- CORE HANDLER (with DuckDuckGo first) ----------
        async function processUserInput(txt) {
            addLine(txt, 'user-message'); addToMemory('user', txt);
            screen.classList.add('screen-shake'); setTimeout(() => screen.classList.remove('screen-shake'), 400);
            setAvatar('processing'); signalLight.classList.remove('active-cyan'); signalLight.style.backgroundColor = '#0088ff';

            const lower = txt.toLowerCase(); let response = '', type = 'mythic-message';

            if (isCodeRequest(txt)) {
                addLine(`<span class="timestamp">[SEARCHING STACK OVERFLOW...]</span>`, 'system-message');
                const cr = await fetchStackOverflowCode(txt);
                if (cr.found) {
                    response = `<span class="timestamp">[CODE · ${cr.title}]</span><br><pre class="code-message">${escapeHtml(cr.code)}</pre>`;
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
                    addLine(`<span class="timestamp">[CONSULTING MYTHIC ARCHIVES...]</span>`, 'system-message');
                    const wiki = await searchWikipedia(txt);
                    if (wiki.found) {
                        response = `<span class="timestamp">[MYTHIC AI · ${wiki.title}]</span><br>${wiki.extract}`;
                        type = 'mythic-message';
                    } else {
                        response = `<span class="timestamp">[MYTHIC AI]</span> ${wiki.message}`;
                        type = 'error-message';
                    }
                }
            } else {
                response = await generateWithMemory(txt);
                type = 'mythic-message';
            }

            addToMemory('assistant', response.replace(/<[^>]*>/g,'').substring(0,200));
            signalLight.style.backgroundColor = ''; signalLight.classList.add('active-cyan');
            setAvatar('cosmos');
            addLine(response, type, true);
            futureYear.textContent = Math.random()>0.5 ? '2157' : ['2140','2165','2180'][Math.floor(Math.random()*3)];
        }

        // ---------- INIT ----------
        window.addEventListener('load', async () => {
            input.focus(); loadModel();
            if(conversationMemory.length===0) addToMemory('assistant', "I am a neural AI from 2157 with web search and persistent memory.");
            else addLine(`<span class="timestamp">[MEMORY RESTORED]</span> ${conversationMemory.length} exchanges loaded.`, 'system-message');
        });
        input.addEventListener('keypress', async (e) => { if(e.key==='Enter' && input.value.trim()) { const t = input.value.trim(); input.value = ''; await processUserInput(t); } });
        document.addEventListener('click', ()=>input.focus());

        // Buttons
        document.getElementById('btn-predict').addEventListener('click', ()=>{
            addLine(`<span class="timestamp">[GENERATING PREDICTION...]</span>`, 'system-message');
            const p = predictions[Math.floor(Math.random()*predictions.length)];
            addLine(`<span class="timestamp">[PREDICTION]</span><br>${p}`, 'prediction-message');
            addToMemory('assistant', p); setAvatar('cosmos');
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
            setTimeout(()=>{ screen.classList.remove('glitch-text'); addLine("Link restored. Memory intact.",'mythic-message'); },2000);
        });
        document.getElementById('btn-clear-memory').addEventListener('click', ()=>{
            clearMemory(); addLine(`<span class="timestamp">[MEMORY PURGED]</span>`,'system-message');
            addToMemory('assistant', "Memory reset.");
        });
    </script>
</body>
</html>
