<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HELLDIVERS 2 // STRATAGEM DRAFT</title>
    
    <link href="https://fonts.googleapis.com/css2?family=Exo+2:wght@400;700;900&family=Share+Tech+Mono&display=swap" rel="stylesheet">
    
    <script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-database-compat.js"></script>

    <style>
        :root {
            --bg: #08090b;
            --panel: #121418;
            --accent: #f0c040;
            --cyan: #00c8ff;
            --green: #27ae60;
            --blue: #2196f3;
            --red: #e74c3c;
            --border: #2a2d35;
        }

        * { box-sizing: border-box; }
        body {
            margin: 0;
            background-color: var(--bg);
            background-image: 
                repeating-linear-gradient(0deg, transparent, transparent 1px, rgba(255,255,255,0.02) 1px, rgba(255,255,255,0.02) 2px),
                repeating-linear-gradient(90deg, transparent, transparent 1px, rgba(255,255,255,0.02) 1px, rgba(255,255,255,0.02) 2px);
            background-size: 20px 20px;
            color: #eee;
            font-family: 'Exo 2', sans-serif;
            overflow-x: hidden;
        }

        h1, h2, h3 { font-weight: 900; text-transform: uppercase; letter-spacing: 1px; color: var(--accent); margin: 0; }
        .mono { font-family: 'Share Tech Mono', monospace; }

        /* Layout */
        #app { max-width: 1200px; margin: 0 auto; padding: 20px; min-height: 100vh; display: flex; flex-direction: column; }
        header { 
            display: flex; justify-content: space-between; align-items: center; 
            border-bottom: 2px solid var(--accent); padding-bottom: 10px; margin-bottom: 20px;
        }

        /* Progress Dots */
        .progress { display: flex; gap: 10px; }
        .dot { width: 12px; height: 12px; border: 2px solid var(--border); border-radius: 50%; }
        .dot.active { background: var(--accent); border-color: var(--accent); box-shadow: 0 0 10px var(--accent); }

        /* Buttons */
        .btn {
            background: var(--accent); color: #000; border: none; padding: 10px 25px;
            font-family: 'Exo 2', sans-serif; font-weight: 900; cursor: pointer;
            clip-path: polygon(10% 0%, 100% 0%, 90% 100%, 0% 100%);
            transition: all 0.2s; text-transform: uppercase;
        }
        .btn:hover { transform: scale(1.05); filter: brightness(1.1); }
        .btn:disabled { background: #444; color: #888; cursor: not-allowed; }
        .btn-cyan { background: var(--cyan); }
        .btn-outline { background: transparent; border: 1px solid var(--accent); color: var(--accent); }

        /* Panels */
        .card { 
            background: var(--panel); border: 1px solid var(--border); padding: 15px; 
            position: relative; transition: 0.3s;
        }
        .card:hover { border-color: var(--accent); box-shadow: 0 0 15px rgba(240,192,64,0.1); }
        
        /* Hexagon Avatar */
        .hex {
            width: 60px; height: 60px; background: var(--accent);
            clip-path: polygon(25% 0%, 75% 0%, 100% 50%, 75% 100%, 25% 100%, 0% 50%);
            display: flex; align-items: center; justify-content: center; color: #000; font-weight: bold;
        }

        /* Grids */
        .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; }
        .flex-center { display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center; flex-grow: 1; }

        /* Forms */
        input, select {
            background: #1a1d23; border: 1px solid var(--border); color: white;
            padding: 10px; font-family: 'Share Tech Mono'; width: 100%; margin-bottom: 10px;
        }

        /* Phase Specifics */
        .strat-picker { display: grid; grid-template-columns: repeat(auto-fill, minmax(140px, 1fr)); gap: 5px; max-height: 400px; overflow-y: auto; }
        .strat-item { font-size: 0.8rem; padding: 5px; border: 1px solid #333; cursor: pointer; }
        .strat-item.selected { border-color: var(--cyan); background: rgba(0,200,255,0.1); }
        
        .drafted-overlay {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.8); display: flex; align-items: center; justify-content: center;
            color: var(--red); font-weight: 900; font-size: 1.5rem; transform: rotate(-15deg);
        }

        .team-alpha { border-top: 4px solid var(--accent); }
        .team-bravo { border-top: 4px solid var(--blue); }

        .hidden { display: none !important; }
    </style>
</head>
<body>

    <div id="app">
        <header>
            <div>
                <h1 id="room-display">HELLDIVERS 2 // DRAFT</h1>
                <div class="mono" id="room-code-label">OFFLINE</div>
            </div>
            <div class="progress" id="progress-dots">
                <div class="dot active"></div>
                <div class="dot"></div>
                <div class="dot"></div>
                <div class="dot"></div>
            </div>
        </header>

        <div id="phase-join" class="flex-center">
            <h2 style="font-size: 3rem; margin-bottom: 20px;">ESTABLISH UPLINK</h2>
            <div style="width: 300px;">
                <button class="btn" style="width: 100%; margin-bottom: 10px;" onclick="createRoom()">Create New Operation</button>
                <div style="margin: 20px 0; color: #555;">— OR —</div>
                <input type="text" id="join-code" placeholder="ENTER 4-LETTER CODE" maxlength="4" style="text-align: center; text-transform: uppercase;">
                <button class="btn btn-cyan" style="width: 100%;" onclick="joinRoom()">Join Operation</button>
            </div>
        </div>

        <div id="phase-lobby" class="hidden">
            <div class="flex-center" id="lobby-entry">
                <h2>IDENTIFY YOURSELF</h2>
                <input type="text" id="player-name" placeholder="CALLSIGN (e.g. Skull Admiral)" style="width: 300px;">
                <button class="btn" onclick="submitName()">JOIN LOBBY</button>
            </div>
            <div id="lobby-list-container" class="hidden">
                <h2 style="margin-bottom: 20px;">RECRUITS IN NEBULA</h2>
                <div id="player-list" class="grid"></div>
                <div style="margin-top: 40px; text-align: center;">
                    <button class="btn" id="start-loadouts-btn" onclick="updatePhase('loadout')">INITIALIZE LOADOUTS</button>
                </div>
            </div>
        </div>

        <div id="phase-loadout" class="hidden">
            <div class="grid">
                <div class="card">
                    <h3>CONFIGURE EQUIPMENT</h3>
                    <label class="mono">PRIMARY WEAPON</label>
                    <select id="sel-primary"></select>
                    <label class="mono">SECONDARY</label>
                    <select id="sel-secondary"></select>
                    <label class="mono">GRENADE</label>
                    <select id="sel-grenade"></select>
                    
                    <h3 style="margin-top: 20px; color: var(--cyan);">STRATAGEMS (PICK 4)</h3>
                    <div id="strat-list" class="strat-picker"></div>
                    <button class="btn" id="lock-btn" style="width: 100%; margin-top: 20px;" onclick="lockLoadout()">LOCK IN SELECTIONS</button>
                </div>
                <div class="card">
                    <h3>READY STATUS</h3>
                    <div id="ready-list"></div>
                    <div id="all-ready-container" class="hidden" style="margin-top: 20px;">
                        <button class="btn btn-cyan" onclick="updatePhase('vote')">ALL RECRUITS READY - START VOTE</button>
                    </div>
                </div>
            </div>
        </div>

        <div id="phase-vote" class="hidden">
            <h2 style="text-align: center; margin-bottom: 30px;">ELECT CAPTAINS</h2>
            <div class="grid" id="vote-grid"></div>
            <div style="text-align: center; margin-top: 30px;">
                <button class="btn btn-outline" onclick="manualCaptains()">MANUAL OVERRIDE (HOST)</button>
                <div id="start-draft-container" class="hidden">
                    <button class="btn btn-cyan" style="margin-top: 20px;" onclick="finalizeCaptains()">START DRAFTING</button>
                </div>
            </div>
        </div>

        <div id="phase-draft" class="hidden">
            <div id="draft-banner" class="mono" style="background: var(--accent); color: #000; padding: 10px; text-align: center; font-weight: bold; margin-bottom: 20px;">
                WAITING FOR COMMAND...
            </div>
            
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 30px;">
                <div class="card team-alpha">
                    <h3 id="cap1-name">TEAM ALPHA</h3>
                    <div id="team-0-list"></div>
                </div>
                <div class="card team-bravo" style="border-top-color: var(--blue);">
                    <h3 id="cap2-name" style="color: var(--blue);">TEAM BRAVO</h3>
                    <div id="team-1-list"></div>
                </div>
            </div>

            <h2>AVAILABLE HELLDIVERS</h2>
            <div id="draft-pool" class="grid"></div>
        </div>

        <div id="phase-results" class="hidden">
            <h2 style="text-align: center; font-size: 3rem; margin-bottom: 30px;">DRAFT COMPLETE</h2>
            <div class="grid" id="final-results"></div>
            <div style="text-align: center; margin-top: 40px;">
                <button class="btn" onclick="resetApp()">NEW OPERATION</button>
            </div>
        </div>
    </div>

    <script>
        // --- FIREBASE CONFIG ---
        const firebaseConfig = {
        apiKey: "AIzaSyCXVl_VIfuu-tdxtBurJmoNvpTqlu-XUQ0",
        authDomain: "helldivers-draft.firebaseapp.com",
        projectId: "helldivers-draft",
        storageBucket: "helldivers-draft.firebasestorage.app",
        messagingSenderId: "70043162066",
        appId: "1:70043162066:web:aa7f4272d28310c354e40b",
        measurementId: "G-QLBJ2F9WFR"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        // --- GAME DATA ---
        const PRIMARIES = ["AR-23 Liberator", "SG-8 Punisher", "R-63 Diligence", "SMG-37 Defender", "SG-225 Breaker", "LAS-5 Scythe", "JAR-5 Dominator", "PLAS-1 Scorcher", "SG-225SP Breaker Spray & Pray", "SG-225IE Breaker Incendiary"];
        const SECONDARIES = ["P-2 Peacemaker", "P-19 Redeemer", "P-4 Senator", "LAS-7 Dagger"];
        const GRENADES = ["G-12 High Explosive", "G-6 Frag", "G-16 Impact", "G-10 Incendiary", "G-3 Smoke", "G-23 Stun"];
        const STRATAGEMS = [
            {n:"Eagle Airstrike", c:"Eagle"}, {n:"Eagle Cluster Bomb", c:"Eagle"}, {n:"Eagle 500kg Bomb", c:"Eagle"}, {n:"Eagle Napalm Strike", c:"Eagle"},
            {n:"Orbital Precision Strike", c:"Orbital"}, {n:"Orbital Laser", c:"Orbital"}, {n:"Orbital Railcannon", c:"Orbital"}, {n:"Orbital 380mm HE Barrage", c:"Orbital"},
            {n:"Stalwart", c:"Support"}, {n:"Anti-Materiel Rifle", c:"Support"}, {n:"Railgun", c:"Support"}, {n:"Autocannon", c:"Support"}, {n:"Grenade Launcher", c:"Support"},
            {n:"Shield Generator Pack", c:"Backpack"}, {n:"Supply Pack", c:"Backpack"}, {n:"Jump Pack", c:"Backpack"}, {n:"Rover Guard Dog", c:"Backpack"},
            {n:"Mortar Sentry", c:"Sentry"}, {n:"Autocannon Sentry", c:"Sentry"}, {n:"Gatling Sentry", c:"Sentry"}, {n:"Rocket Sentry", c:"Sentry"},
            {n:"HMG Emplacement", c:"Emplacement"}, {n:"Shield Generator Relay", c:"Emplacement"}, {n:"Tesla Tower", c:"Emplacement"}
        ];

        // --- STATE ---
        let roomCode = localStorage.getItem('hd_roomCode') || null;
        let myName = localStorage.getItem('hd_playerName') || null;
        let selectedStrats = [];
        let roomData = null;

        // --- INITIALIZATION ---
        function initSelects() {
            const fill = (id, list) => {
                const el = document.getElementById(id);
                list.forEach(item => {
                    const opt = document.createElement('option');
                    opt.value = opt.textContent = item;
                    el.appendChild(opt);
                });
            };
            fill('sel-primary', PRIMARIES);
            fill('sel-secondary', SECONDARIES);
            fill('sel-grenade', GRENADES);

            const sList = document.getElementById('strat-list');
            STRATAGEMS.forEach(s => {
                const div = document.createElement('div');
                div.className = 'strat-item card';
                div.innerHTML = `<div class="mono" style="font-size:10px; color:var(--cyan)">${s.c}</div><div>${s.n}</div>`;
                div.onclick = () => toggleStrat(s.n, div);
                sList.appendChild(div);
            });
        }
        initSelects();

        if (roomCode) {
            connectToRoom(roomCode);
        }

        // --- CORE FUNCTIONS ---
        function createRoom() {
            const code = Math.random().toString(36).substring(2, 6).toUpperCase();
            db.ref('rooms/' + code).set({
                phase: 'lobby',
                createdAt: Date.now()
            }).then(() => connectToRoom(code));
        }

        function joinRoom() {
            const code = document.getElementById('join-code').value.toUpperCase();
            if (code.length !== 4) return alert("Invalid Code");
            connectToRoom(code);
        }

        function connectToRoom(code) {
            roomCode = code;
            localStorage.setItem('hd_roomCode', code);
            document.getElementById('room-code-label').textContent = `ROOM: ${code}`;
            document.getElementById('phase-join').classList.add('hidden');
            document.getElementById('phase-lobby').classList.remove('hidden');

            db.ref('rooms/' + code).on('value', snapshot => {
                roomData = snapshot.val();
                if (!roomData) {
                    resetApp();
                    return;
                }
                renderPhase();
            });
        }

        function submitName() {
            const nameInput = document.getElementById('player-name').value.trim();
            if (!nameInput) return;
            myName = nameInput;
            localStorage.setItem('hd_playerName', myName);
            db.ref(`rooms/${roomCode}/players/${myName}`).update({
                joined: true,
                locked: false
            });
        }

        function updatePhase(p) {
            db.ref(`rooms/${roomCode}`).update({ phase: p });
        }

        // --- LOGIC ---
        function toggleStrat(name, el) {
            if (selectedStrats.includes(name)) {
                selectedStrats = selectedStrats.filter(s => s !== name);
                el.classList.remove('selected');
            } else if (selectedStrats.length < 4) {
                selectedStrats.push(name);
                el.classList.add('selected');
            }
        }

        function lockLoadout() {
            if (selectedStrats.length < 4) return alert("Select 4 Stratagems!");
            db.ref(`rooms/${roomCode}/players/${myName}`).update({
                locked: true,
                loadout: {
                    primary: document.getElementById('sel-primary').value,
                    secondary: document.getElementById('sel-secondary').value,
                    grenade: document.getElementById('sel-grenade').value,
                    strats: selectedStrats
                }
            });
            document.getElementById('lock-btn').disabled = true;
            document.getElementById('lock-btn').textContent = "LOCKED IN";
        }

        function castVote(targetName) {
            if (targetName === myName) return;
            db.ref(`rooms/${roomCode}/votes/${myName}`).set(targetName);
        }

        function finalizeCaptains() {
            const votes = roomData.votes || {};
            const tally = {};
            Object.values(votes).forEach(v => tally[v] = (tally[v] || 0) + 1);
            
            const sorted = Object.keys(roomData.players).sort((a,b) => (tally[b]||0) - (tally[a]||0));
            const caps = [sorted[0], sorted[1]];
            
            db.ref(`rooms/${roomCode}`).update({
                captains: caps,
                draftTurn: 0,
                phase: 'draft'
            });
        }

        function pickPlayer(targetName) {
            const caps = roomData.captains;
            const turn = roomData.draftTurn;
            const capIdx = getTurnOwnerIdx(turn);
            
            // Check if it's my turn
            if (caps[capIdx] !== myName) return;

            db.ref(`rooms/${roomCode}/draftPicks/${capIdx}`).push(targetName);
            
            const totalPlayers = Object.keys(roomData.players).length - 2;
            const currentPicks = (roomData.draftPicks ? Object.values(roomData.draftPicks).flatMap(x => Object.values(x)).length : 0) + 1;

            if (currentPicks >= totalPlayers) {
                setTimeout(() => updatePhase('results'), 1500);
            }

            db.ref(`rooms/${roomCode}/draftTurn`).set(turn + 1);
        }

        function getTurnOwnerIdx(turn) {
            // Snake: 0, 1, 1, 0, 0, 1, 1...
            const pattern = [0, 1, 1, 0];
            return pattern[turn % 4];
        }

        // --- RENDERING ---
        function renderPhase() {
            const p = roomData.phase;
            const sections = ['phase-lobby', 'phase-loadout', 'phase-vote', 'phase-draft', 'phase-results'];
            sections.forEach(s => document.getElementById(s).classList.add('hidden'));
            document.getElementById(`phase-${p}`).classList.remove('hidden');

            // Dots
            const dots = document.querySelectorAll('.dot');
            const pIdx = ['lobby', 'loadout', 'vote', 'draft', 'results'].indexOf(p);
            dots.forEach((d, i) => d.className = i <= pIdx ? 'dot active' : 'dot');

            if (p === 'lobby') {
                if (myName && roomData.players?.[myName]) {
                    document.getElementById('lobby-entry').classList.add('hidden');
                    document.getElementById('lobby-list-container').classList.remove('hidden');
                    const list = document.getElementById('player-list');
                    list.innerHTML = '';
                    const players = Object.keys(roomData.players || {});
                    players.forEach(name => {
                        list.innerHTML += `<div class="card flex-center"><div class="hex">${name[0]}</div><div class="mono" style="margin-top:10px">${name}</div></div>`;
                    });
                    document.getElementById('start-loadouts-btn').style.display = players.length >= 2 ? 'inline-block' : 'none';
                }
            }

            if (p === 'loadout') {
                const readyList = document.getElementById('ready-list');
                readyList.innerHTML = '';
                let allLocked = true;
                Object.entries(roomData.players).forEach(([name, data]) => {
                    const status = data.locked ? `<span style="color:var(--green)">READY</span>` : `<span style="color:#555">EQUIPPING...</span>`;
                    readyList.innerHTML += `<div class="mono" style="margin-bottom:5px">${name}: ${status}</div>`;
                    if (!data.locked) allLocked = false;
                });
                document.getElementById('all-ready-container').className = allLocked ? '' : 'hidden';
            }

            if (p === 'vote') {
                const grid = document.getElementById('vote-grid');
                grid.innerHTML = '';
                const votes = roomData.votes || {};
                Object.keys(roomData.players).forEach(name => {
                    const tally = Object.values(votes).filter(v => v === name).length;
                    const votedForMe = votes[myName] === name;
                    grid.innerHTML += `
                        <div class="card" onclick="castVote('${name}')" style="cursor:pointer; border-color:${votedForMe ? 'var(--accent)' : ''}">
                            <div class="mono" style="font-size:2rem">${tally}</div>
                            <div>${name}</div>
                            ${name === myName ? '<div class="mono" style="font-size:10px">(CANNOT VOTE SELF)</div>' : ''}
                        </div>
                    `;
                });
                if (Object.keys(votes).length === Object.keys(roomData.players).length) {
                    document.getElementById('start-draft-container').classList.remove('hidden');
                }
            }

            if (p === 'draft') {
                const capIdx = getTurnOwnerIdx(roomData.draftTurn);
                const activeCap = roomData.captains[capIdx];
                const banner = document.getElementById('draft-banner');
                banner.textContent = `${activeCap.toUpperCase()} IS CHOOSING...`;
                banner.style.background = capIdx === 0 ? 'var(--accent)' : 'var(--blue)';

                document.getElementById('cap1-name').textContent = `TEAM ALPHA (${roomData.captains[0]})`;
                document.getElementById('cap2-name').textContent = `TEAM BRAVO (${roomData.captains[1]})`;

                // Render Teams
                const picks0 = roomData.draftPicks?.[0] ? Object.values(roomData.draftPicks[0]) : [];
                const picks1 = roomData.draftPicks?.[1] ? Object.values(roomData.draftPicks[1]) : [];
                document.getElementById('team-0-list').innerHTML = picks0.map(n => `<div class="mono">• ${n}</div>`).join('');
                document.getElementById('team-1-list').innerHTML = picks1.map(n => `<div class="mono">• ${n}</div>`).join('');

                // Render Pool
                const pool = document.getElementById('draft-pool');
                pool.innerHTML = '';
                const allDrafted = [...picks0, ...picks1, ...roomData.captains];
                Object.entries(roomData.players).forEach(([name, data]) => {
                    if (roomData.captains.includes(name)) return;
                    const isDrafted = allDrafted.includes(name);
                    pool.innerHTML += `
                        <div class="card" onclick="${!isDrafted ? `pickPlayer('${name}')` : ''}" style="opacity:${isDrafted ? 0.4 : 1}">
                            ${isDrafted ? '<div class="drafted-overlay">DRAFTED</div>' : ''}
                            <h3 style="color:white">${name}</h3>
                            <div class="mono" style="font-size:11px">
                                <div>PRI: ${data.loadout.primary}</div>
                                <div>SEC: ${data.loadout.secondary}</div>
                                <div style="color:var(--cyan)">${data.loadout.strats.join(', ')}</div>
                            </div>
                        </div>
                    `;
                });
            }

            if (p === 'results') {
                const res = document.getElementById('final-results');
                res.innerHTML = '';
                [0, 1].forEach(idx => {
                    const cap = roomData.captains[idx];
                    const picks = roomData.draftPicks?.[idx] ? Object.values(roomData.draftPicks[idx]) : [];
                    const members = [cap, ...picks];
                    let html = `<div class="card ${idx===0?'team-alpha':'team-bravo'}"><h3>TEAM ${idx===0?'ALPHA':'BRAVO'}</h3>`;
                    members.forEach(m => {
                        const d = roomData.players[m];
                        html += `<div style="margin-top:15px; border-bottom:1px solid #222; padding-bottom:10px;">
                            <div class="mono" style="color:var(--accent)">${m} ${m===cap?'(CAPTAIN)':''}</div>
                            <div style="font-size:12px">${d.loadout.primary} / ${d.loadout.strats.slice(0,2).join(', ')}...</div>
                        </div>`;
                    });
                    html += `</div>`;
                    res.innerHTML += html;
                });
            }
        }

        function resetApp() {
            if (roomCode) db.ref(`rooms/${roomCode}`).remove();
            localStorage.clear();
            location.reload();
        }

        function manualCaptains() {
            const names = Object.keys(roomData.players);
            if (names.length < 2) return;
            db.ref(`rooms/${roomCode}`).update({
                captains: [names[0], names[1]],
                draftTurn: 0,
                phase: 'draft'
            });
        }
    </script>
</body>
</html>
