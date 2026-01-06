<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thai Rhythm Piano Game</title>
    <style>
        body { margin: 0; background: #0f0f13; color: white; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; overflow: hidden; }
        #game-container { position: relative; width: 350px; height: 550px; background: linear-gradient(180deg, #222 0%, #111 100%); border: 5px solid #444; border-radius: 10px; box-shadow: 0 0 30px rgba(0,0,0,0.5); }
        .lane { position: absolute; top: 0; width: 20%; height: 100%; border-right: 1px solid rgba(255,255,255,0.1); box-sizing: border-box; }
        .note { position: absolute; width: 90%; left: 5%; height: 25px; background: #00f2ff; border-radius: 5px; box-shadow: 0 0 15px #00f2ff; transition: background 0.1s; }
        #hit-line { position: absolute; bottom: 80px; width: 100%; height: 4px; background: #ff0055; box-shadow: 0 0 10px #ff0055; z-index: 10; }
        .key-guide { position: absolute; bottom: 0; width: 100%; height: 80px; display: flex; background: #222; border-radius: 0 0 5px 5px; }
        .key { flex: 1; display: flex; align-items: center; justify-content: center; border: 1px solid #333; font-weight: bold; font-size: 20px; color: #888; transition: 0.1s; }
        .key.active { background: #444; color: #fff; transform: scale(0.95); box-shadow: inset 0 0 10px #000; }
        #ui { position: absolute; top: -60px; width: 100%; display: flex; justify-content: space-between; font-size: 20px; }
        #start-btn { padding: 15px 30px; font-size: 20px; cursor: pointer; background: #00f2ff; border: none; border-radius: 50px; font-weight: bold; color: #000; box-shadow: 0 5px 15px rgba(0,242,255,0.4); }
        #start-btn:hover { transform: scale(1.05); }
    </style>
</head>
<body>

    <div style="position: relative;">
        <div id="ui">
            <div>คะแนน: <span id="score">0</span></div>
            <div>Combo: <span id="combo">0</span></div>
        </div>
        
        <div id="game-container">
            <div class="lane" style="left: 0%;"></div>
            <div class="lane" style="left: 20%;"></div>
            <div class="lane" style="left: 40%;"></div>
            <div class="lane" style="left: 60%;"></div>
            <div class="lane" style="left: 80%;"></div>
            
            <div id="hit-line"></div>
            <div id="notes-wrapper"></div>

            <div class="key-guide">
                <div class="key" id="key-a">A</div>
                <div class="key" id="key-s">S</div>
                <div class="key" id="key-d">D</div>
                <div class="key" id="key-f">F</div>
                <div class="key" id="key-g">G</div>
            </div>
        </div>
    </div>

    <div style="margin-top: 40px;">
        <button id="start-btn" onclick="startGame()">▶ เริ่มเล่นเพลงไทย</button>
    </div>

    <script>
        const wrapper = document.getElementById('notes-wrapper');
        const scoreEl = document.getElementById('score');
        const comboEl = document.getElementById('combo');
        const keys = { 'a': 0, 's': 1, 'd': 2, 'f': 3, 'g': 4 };
        let score = 0;
        let combo = 0;
        let gameActive = false;

        // ตัวอย่างเพลงไทย (ใช้ลิงก์ตัวอย่าง - คุณเปลี่ยนเป็นไฟล์ mp3 ของคุณเองได้)
        const music = new Audio('https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3'); 

        // ตารางโน้ต (เวลาหน่วยมิลลิวินาที, เลนที่ 0-4)
        const chart = [
            { t: 1000, l: 0 }, { t: 1500, l: 2 }, { t: 2000, l: 4 },
            { t: 2500, l: 1 }, { t: 3000, l: 3 }, { t: 3500, l: 0 },
            { t: 4000, l: 2 }, { t: 4200, l: 3 }, { t: 4400, l: 4 }
        ];

        function startGame() {
            if(gameActive) return;
            gameActive = true;
            score = 0; combo = 0;
            scoreEl.innerText = score;
            comboEl.innerText = combo;
            document.getElementById('start-btn').style.display = 'none';
            
            music.currentTime = 0;
            music.play();

            chart.forEach(noteData => {
                setTimeout(() => createNote(noteData.l), noteData.t);
            });
        }

        function createNote(laneIndex) {
            const note = document.createElement('div');
            note.className = 'note';
            note.style.left = (laneIndex * 20 + 2) + '%';
            wrapper.appendChild(note);

            let pos = -20;
            const speed = 4; // ความเร็วโน้ต
            
            const fall = setInterval(() => {
                pos += speed;
                note.style.top = pos + 'px';

                if (pos > 550) {
                    clearInterval(fall);
                    if (note.parentElement) {
                        note.remove();
                        combo = 0;
                        comboEl.innerText = combo;
                    }
                }
            }, 10);

            // เก็บอ้างอิงไว้เช็คการกด
            note.dataset.lane = laneIndex;
            note.dataset.pos = pos;
        }

        window.addEventListener('keydown', (e) => {
            const key = e.key.toLowerCase();
            if (keys.hasOwnProperty(key)) {
                const laneIdx = keys[key];
                const keyBox = document.getElementById('key-' + key);
                keyBox.classList.add('active');
                setTimeout(() => keyBox.classList.remove('active'), 100);

                // เช็คว่ามีโน้ตในเลนนั้นที่อยู่ในจุดกดหรือไม่
                const allNotes = document.querySelectorAll('.note');
                allNotes.forEach(note => {
                    const notePos = parseInt(note.style.top);
                    if (parseInt(note.dataset.lane) === laneIdx && notePos > 430 && notePos < 500) {
                        score += 100;
                        combo += 1;
                        scoreEl.innerText = score;
                        comboEl.innerText = combo;
                        note.remove();
                    }
                });
            }
        });
    </script>
</body>
</html>
