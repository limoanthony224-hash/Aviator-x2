<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Aviator Pro - Precision Flight</title>
    
    <link rel="manifest" id="pwa-manifest">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="theme-color" content="#0b0b0b">
     <link rel="apple-touch-icon" href="my-photo.jpg">

    <style>
        :root {
            --bg-dark: #0b0b0b;
            --panel-bg: #1c1c1c;
            --accent-gold: #ffca28;
            --accent-red: #e91e63;
            --success-green: #28a745;
            --mpesa-green: #49aa27;
            --text-main: #ffffff;
            --text-dim: #888;
        }

        body {
            background-color: var(--bg-dark); color: var(--text-main);
            font-family: 'Segoe UI', Roboto, sans-serif;
            margin: 0; padding: 0; display: flex; justify-content: center;
            height: 100vh; overflow: hidden;
        }

        .game-container {
            width: 100vw; height: 100vh; max-width: 400px;
            background: var(--panel-bg); display: flex; flex-direction: column;
            position: relative;
        }

        /* Notifications */
        #toast {
            position: absolute; top: -100px; left: 10px; right: 10px;
            background: #333; color: green; padding: 12px; border-radius: 8px;
            z-index: 1000; transition: 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            text-align: center; border-left: 5px solid var(--mpesa-green);
            box-shadow: 0 5px 15px rgba(0,0,0,0.5);
        }
        #toast.show { top: 20px; }

        .history-bar {
            background: #000; height: 30px; display: flex; align-items: center;
            gap: 8px; padding: 0 10px; overflow-x: auto; border-bottom: 1px solid #333;
        }
        .hist-val { font-size: 0.6rem; font-weight: bold; padding: 2px 6px; border-radius: 10px; background: #222; }

        .header {
            padding: 5px 12px; display: flex; justify-content: space-between;
            align-items: center; background: #1a1a1a; border-bottom: 1px solid #333;
        }

        .chart-area { flex: 1.5; background: #050505; position: relative; overflow: hidden; }

        #multiplier {
            position: absolute; top: 20%; width: 100%; text-align: center;
            font-size: 3rem; font-weight: 900; z-index: 10; text-shadow: 0 0 10px #000;
        }

        #next-round-timer {
            position: absolute; top: 40%; width: 100%; text-align: center; z-index: 30; display: none;
        }
        .progress-bar { width: 120px; height: 4px; background: #333; margin: 8px auto; border-radius: 5px; overflow: hidden; }
        #progress-fill { width: 100%; height: 100%; background: var(--mpesa-green); }

        #plane-wrapper {
            position: absolute; bottom: 0; left: 0; width: 45px; z-index: 25; display: none;
            margin-left: -5px; margin-bottom: -5px; 
        }
        .propeller {
            position: absolute; right: 0; top: 12px; width: 2px; height: 14px;
            background: #fff; animation: spin 0.1s linear infinite;
        }
        @keyframes spin { from { transform: rotateX(0deg); } to { transform: rotateX(360deg); } }

        #flight-curve { position: absolute; width: 115%; height: 100%; z-index: 20; }

        .bet-section { padding: 8px; background: #111; display: flex; flex-direction: column; gap: 6px; }
        .bet-row {
            display: grid; grid-template-columns: 1fr 1fr; gap: 8px;
            background: #1c1c1c; padding: 6px; border-radius: 8px; border: 1px solid #333;
        }
        .bet-btn { border-radius: 6px; border: none; font-weight: 900; color: white; height: 38px; cursor: pointer; }

        .players-box {
            flex: 1; background: #080808; margin: 0 8px 8px; border-radius: 8px;
            overflow-y: auto; padding: 8px; border: 1px solid #222;
        }
        .player-row { display: flex; justify-content: space-between; font-size: 0.65rem; padding: 3px 0; border-bottom: 1px solid #151515; }

        .modal {
            display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.9);
            z-index: 600; justify-content: center; align-items: center;
        }
    </style>
</head>
<body>

<div id="toast">Message goes here</div>

<div class="game-container">
    <div class="history-bar" id="history-list"></div>

    <div class="header">
        <div>
            <div style="font-size: 0.4rem; color: var(--text-dim);">WALLET BALANCE</div>
            <div id="balance-amount" style="color:var(--success-green); font-weight:800; font-size: 0.85rem;">KES 1,000.00</div>
        </div>
        <div style="display:flex; gap:5px;">
            <button onclick="triggerWithdraw()" style="background:#444; border:none; color:white; border-radius:4px; padding:5px 10px; font-weight:bold; font-size:0.6rem;">WITHDRAW</button>
            <button onclick="triggerMpesaPush()" style="background:var(--mpesa-green); border:none; color:white; border-radius:4px; padding:5px 12px; font-weight:bold; font-size:0.7rem;">DEPOSIT</button>
        </div>
    </div>

    <div class="chart-area" id="chart">
        <div id="next-round-timer">
            <div style="color: var(--accent-gold); font-weight: bold; font-size: 0.7rem;">WAITING FOR NEXT ROUND</div>
            <div class="progress-bar"><div id="progress-fill"></div></div>
        </div>
        
        <svg id="flight-curve" viewBox="0 0 400 300" preserveAspectRatio="none">
            <path id="curve-path" d="M 0 300" fill="none" stroke="#e91e63" stroke-width="5" stroke-linecap="round"/>
        </svg>

        <div id="multiplier">1.00x</div>
        
        <div id="plane-wrapper">
            <div class="propeller"></div>
            <img src="https://img.icons8.com/ios-filled/100/ff0000/fighter-jet.png" style="width:100%; transform: rotate(-15deg);">
        </div>
    </div>

    <div class="bet-section">
        <div class="bet-row">
            <input type="number" id="bet-1-amt" value="100" style="background:#000; color:white; border:1px solid #444; text-align:center; border-radius:4px;">
            <button id="btn-1" class="bet-btn" onclick="toggleBet(1)" style="background:var(--success-green);">BET</button>
        </div>
        <div class="bet-row">
            <input type="number" id="bet-2-amt" value="100" style="background:#000; color:white; border:1px solid #444; text-align:center; border-radius:4px;">
            <button id="btn-2" class="bet-btn" onclick="toggleBet(2)" style="background:var(--success-green);">BET</button>
        </div>
    </div>

    <div class="players-box" id="player-feed"></div>
</div>

<div id="mpesa-modal" class="modal">
    <div style="background:green; padding:20px; border-radius:12px; width:220px; text-align:center; color:#333;">
        <h4 style="color:var(--mpesa-green); margin:0;">M-PESA DEPOSIT</h4>
        <p style="font-size:0.7rem;">Enter PIN to deposit KES 500</p>
        <input type="password" id="mpesa-pin" maxlength="4" style="width:80%; margin:15px 0; font-size:20px; text-align:center; letter-spacing:10px;">
        <button onclick="confirmDeposit()" style="width:100%; padding:10px; background:var(--mpesa-green); border:none; color:white; border-radius:5px; font-weight:bold;">CONFIRM</button>
    </div>
</div>

<script>
    /* --- INJECTED PWA APP LOGIC --- */
    const manifest = {
        "name": "Aviator Pro Flight",
        "short_name": "Aviator",
        "start_url": ".",
        "display": "standalone",
        "background_color": "#0b0b0b",
        "theme_color": "#0b0b0b",
        "icons": [{
            "src": "https://img.icons8.com/ios-filled/512/ff0000/fighter-jet.png",
            "sizes": "512x512",
            "type": "image/png"
        }]
    };
    const blob = new Blob([JSON.stringify(manifest)], {type: 'application/json'});
    const manifestURL = URL.createObjectURL(blob);
    document.querySelector('#pwa-manifest').setAttribute('href', manifestURL);

    if ('serviceWorker' in navigator) {
        window.addEventListener('load', () => {
            const swCode = `self.addEventListener('fetch', (e) => e.respondWith(fetch(e.request)));`;
            const swBlob = new Blob([swCode], { type: 'text/javascript' });
            navigator.serviceWorker.register(URL.createObjectURL(swBlob));
        });
    }

    /* --- ORIGINAL GAME LOGIC --- */
    let balance = 1000.0;
    let multiplier = 1.00;
    let isFlying = false;
    let bets = { 1: { state: 'none', amount: 0 }, 2: { state: 'none', amount: 0 } };
    let history = ["1.52", "2.10", "1.05", "4.20", "1.18","80","34","11"];

    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    let engineOsc = null;
    let engineGain = null;

    function showToast(msg, isSuccess = true) {
        const t = document.getElementById('toast');
        t.innerText = msg;
        t.style.borderLeftColor = isSuccess ? 'var(--mpesa-green)' : 'var(--accent-red)';
        t.classList.add('show');
        setTimeout(() => t.classList.remove('show'), 3000);
    }

    function startEngineSound() {
        engineOsc = audioCtx.createOscillator();
        engineGain = audioCtx.createGain();
        engineOsc.type = 'sawtooth';
        engineOsc.frequency.setValueAtTime(50, audioCtx.currentTime);
        engineGain.gain.setValueAtTime(0.01, audioCtx.currentTime);
        engineOsc.connect(engineGain);
        engineGain.connect(audioCtx.destination);
        engineOsc.start();
    }

    function updateEngineSound(mult) {
        if (engineOsc) {
            engineOsc.frequency.setTargetAtTime(50 + (mult * 20), audioCtx.currentTime, 0.1);
            engineGain.gain.setTargetAtTime(Math.min(0.05, 0.01 * mult), audioCtx.currentTime, 0.1);
        }
    }

    function stopEngineSound() {
        if (engineOsc) {
            engineGain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 0.1);
            engineOsc.stop(audioCtx.currentTime + 0.2);
            engineOsc = null;
        }
    }

    function playCrashSound() {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = 'triangle';
        osc.frequency.setValueAtTime(150, audioCtx.currentTime);
        osc.frequency.exponentialRampToValueAtTime(40, audioCtx.currentTime + 0.4);
        gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
        gain.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 0.4);
        osc.connect(gain); gain.connect(audioCtx.destination);
        osc.start(); osc.stop(audioCtx.currentTime + 0.4);
    }

    function updateHistory() {
        const hList = document.getElementById('history-list');
        hList.innerHTML = history.map(v => `<span class="hist-val" style="color:${v > 2 ? 'var(--accent-gold)' : '#fff'}">${v}x</span>`).join('');
    }
    updateHistory();

    function toggleBet(n) {
        const btn = document.getElementById(`btn-${n}`);
        const amt = parseFloat(document.getElementById(`bet-${n}-amt`).value);
        if (bets[n].state === 'none') {
            if (amt > balance || amt <= 0) return;
            balance -= amt; bets[n] = { state: 'waiting', amount: amt };
            btn.innerText = "CANCEL"; btn.style.background = "#e91e63";
        } else if (bets[n].state === 'waiting') {
            balance += bets[n].amount; bets[n].state = 'none';
            btn.innerText = "BET"; btn.style.background = "#28a745";
        } else if (bets[n].state === 'active') {
            let win = bets[n].amount * multiplier;
            balance += win; bets[n].state = 'none';
            btn.innerText = "WON!"; btn.style.background = "#444";
            showToast(`Cashout Successful: KES ${win.toFixed(2)}`);
        }
        document.getElementById('balance-amount').innerText = "KES " + balance.toFixed(2);
    }

    function startCooldown() {
        isFlying = false;
        document.getElementById('multiplier').style.display = 'none';
        document.getElementById('plane-wrapper').style.display = 'none';
        document.getElementById('next-round-timer').style.display = 'block';
        let timeLeft = 3000;
        const cd = setInterval(() => {
            timeLeft -= 50;
            document.getElementById('progress-fill').style.width = (timeLeft/3000)*100 + "%";
            if (timeLeft <= 0) { clearInterval(cd); document.getElementById('next-round-timer').style.display = 'none'; startFlight(); }
        }, 50);
    }

    function startFlight() {
        isFlying = true; multiplier = 1.00;
        const crashPoint = (Math.random() * 5 + 1.1).toFixed(2);
        const plane = document.getElementById('plane-wrapper');
        const curve = document.getElementById('curve-path');
        const multDisplay = document.getElementById('multiplier');
        
        plane.style.display = 'block';
        multDisplay.style.display = 'block';
        multDisplay.style.color = 'white';
        startEngineSound();

        [1, 2].forEach(n => {
            if (bets[n].state === 'waiting') {
                bets[n].state = 'active';
                document.getElementById(`btn-${n}`).style.background = "var(--accent-gold)";
                document.getElementById(`btn-${n}`).style.color = "black";
            }
        });

        const flightLoop = setInterval(() => {
            multiplier += 0.01;
            multDisplay.innerText = multiplier.toFixed(2) + "x";
            updateEngineSound(multiplier);

            let progress = (multiplier - 1);
            let x = Math.min(progress * 75, 365); 
            let y = Math.min(Math.pow(progress * 30, 1.25), 265); 

            plane.style.left = x + "px";
            plane.style.bottom = y + "px";
            curve.setAttribute('d', `M 0 300 Q ${x * 0.3} 300 ${x} ${300 - y}`);

            [1, 2].forEach(n => {
                if(bets[n].state === 'active') {
                    document.getElementById(`btn-${n}`).innerText = "CASH " + (bets[n].amount * multiplier).toFixed(0);
                }
            });

            if (multiplier >= crashPoint) {
                clearInterval(flightLoop);
                stopEngineSound();
                playCrashSound();
                multDisplay.style.color = "var(--accent-red)";
                multDisplay.innerText = "FLEW AWAY!";
                history.unshift(multiplier.toFixed(2));
                if(history.length > 10) history.pop();
                updateHistory();
                
                [1,2].forEach(n => {
                    if(bets[n].state === 'active') {
                        bets[n].state = 'none';
                        document.getElementById(`btn-${n}`).innerText = "BET";
                        document.getElementById(`btn-${n}`).style.background = "var(--success-green)";
                        document.getElementById(`btn-${n}`).style.color = "white";
                    }
                });
                setTimeout(startCooldown, 2000);
            }
        }, 60);
        updatePlayers();
    }

    function updatePlayers() {
        const names = ["Mpesa legend", "Shan clare", "Pilot_KE", "Ogeto Isaac", "Jakin Denser","Branton wanjala"];
        const feed = document.getElementById('player-feed');
        feed.innerHTML = '<div style="color:var(--text-dim); font-size:0.5rem; margin-bottom:4px;">LIVE BETS</div>';
        names.forEach(name => {
            feed.innerHTML += `<div class="player-row"><span>${name}</span><span style="color:var(--success-green)">${(Math.random()*2 + 1).toFixed(2)}x</span></div>`;
        });
    }

    function triggerMpesaPush() { document.getElementById('mpesa-modal').style.display = 'flex'; }
    
    function confirmDeposit() {
        balance += 500;
        document.getElementById('balance-amount').innerText = "KES " + balance.toFixed(2);
        document.getElementById('mpesa-modal').style.display = 'none';
        showToast("Deposit Successful: KES 500.00 deposited succefully");
    }

    function triggerWithdraw() {
        if(balance < 50) {
            showToast("Minimum withdrawal is KES 50", false);
            return;
        }
        let amt = balance;
        balance = 0;
        document.getElementById('balance-amount').innerText = "KES 0.00";
        showToast(`Withdrawal of KES ${amt.toFixed(2)} processed to M-Pesa`);
    }

    startCooldown();
</script>
</body>
</html>
# Aviator-x2
