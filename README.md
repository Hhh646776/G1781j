# G1781j
لعبة ضرب الكرات
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>لعبة تحدي السرعة - إصدار خاص</title>
  <style>
    body {
      background-color: #121212;
      color: #eee;
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
    }
    h1 {
      margin-bottom: 10px;
    }
    #game-area {
      position: relative;
      margin: 20px auto;
      width: 300px;
      height: 300px;
      background-color: #1f1f1f;
      border-radius: 10px;
      overflow: hidden;
    }
    .circle {
      position: absolute;
      width: 60px;
      height: 60px;
      border-radius: 50%;
      cursor: pointer;
      user-select: none;
      display: flex;
      justify-content: center;
      align-items: center;
      font-weight: bold;
      font-size: 20px;
      color: #121212;
      transition: left 0.3s ease, top 0.3s ease;
      box-shadow: 0 0 8px rgba(0,0,0,0.5);
    }
    .red { background-color: #e53935; box-shadow: 0 0 15px #e53935; }
    .white { background-color: #eee; color: #121212; box-shadow: 0 0 15px #eee; }
    .black { background-color: #212121; box-shadow: 0 0 15px #000; border: 2px solid #fff; }
    .gold { background: radial-gradient(circle at center, #ffd700, #b8860b); box-shadow: 0 0 20px #ffd700; }
    .purple { background: radial-gradient(circle at center, #9b59b6, #6a0dad); box-shadow: 0 0 25px #9b59b6; color: #fff; }
    .bomb { background-color: #444; color: red; font-size: 30px; font-weight: bold; box-shadow: 0 0 12px red; }
    #score, #time, #speed {
      font-size: 24px;
      margin: 8px;
    }
    #start-btn {
      background-color: #00bcd4;
      border: none;
      color: #121212;
      padding: 10px 20px;
      font-size: 18px;
      border-radius: 8px;
      cursor: pointer;
      margin-top: 15px;
    }
    #start-btn:hover {
      background-color: #0097a7;
    }
    #player-name {
      font-size: 18px;
      padding: 5px;
      border-radius: 6px;
      border: none;
      margin-bottom: 10px;
      width: 180px;
      text-align: center;
    }
    #leaderboard {
      margin-top: 20px;
      max-width: 320px;
      margin-left: auto;
      margin-right: auto;
      background: #222;
      padding: 15px;
      border-radius: 10px;
    }
    #leaderboard h2 {
      color: #00bcd4;
    }
  </style>
</head>
<body>

  <h1>لعبة تحدي السرعة - النسخة السرية</h1>
  <input id="player-name" type="text" placeholder="اكتب اسمك هنا" maxlength="15" />
  <div id="score">النقاط: 0</div>
  <div id="time">الوقت: 60</div>
  <div id="speed">السرعة: 1</div>
  <div id="game-area"></div>
  <button id="start-btn">ابدأ اللعبة</button>

  <div id="leaderboard">
    <h2>لوحة الصدارة</h2>
    <ol id="leaderboard-list"></ol>
  </div>

  <script>
    const gameArea = document.getElementById('game-area');
    const scoreDisplay = document.getElementById('score');
    const timeDisplay = document.getElementById('time');
    const speedDisplay = document.getElementById('speed');
    const startBtn = document.getElementById('start-btn');
    const playerNameInput = document.getElementById('player-name');
    const leaderboardList = document.getElementById('leaderboard-list');

    let score = 0;
    let time = 60;
    let speed = 1;
    let timerId;
    let timeoutId;
    let currentCircle;
    const size = 60;

    const circlesData = [
      { className: 'red', points: 3, chance: 80 },
      { className: 'white', points: 5, chance: 60 },
      { className: 'black', points: 8, chance: 40 },
      { className: 'gold', points: 150, chance: 1 }
    ];

    const purpleCircle = { className: 'purple', points: 10000, chance: 0.00000001 };
    const bombChance = 20;

    function randomPosition() {
      const maxX = gameArea.clientWidth - size;
      const maxY = gameArea.clientHeight - size;
      return {
        x: Math.floor(Math.random() * maxX),
        y: Math.floor(Math.random() * maxY)
      };
    }

    function getRandomCircleOrBomb() {
      const name = playerNameInput.value.trim();

      if (Math.random() * 100 < bombChance) {
        return { type: 'bomb' };
      }

      if (name === '10177') {
        return purpleCircle;
      }

      const possibleCircles = circlesData.filter(c => (Math.random() * 100) < c.chance);

      if (Math.random() * 100 < purpleCircle.chance) {
        possibleCircles.push(purpleCircle);
      }

      if (possibleCircles.length === 0) return circlesData[0];
      return possibleCircles[Math.floor(Math.random() * possibleCircles.length)];
    }

    function createCircle() {
      if (currentCircle) {
        gameArea.removeChild(currentCircle);
        clearTimeout(timeoutId);
      }

      const circle = document.createElement('div');
      circle.classList.add('circle');

      const data = getRandomCircleOrBomb();

      if (data.type === 'bomb') {
        circle.classList.add('bomb');
        circle.textContent = '💣';
      } else {
        circle.classList.add(data.className);
        circle.textContent = `+${data.points}`;
      }

      const pos = randomPosition();
      circle.style.left = pos.x + 'px';
      circle.style.top = pos.y + 'px';

      circle.addEventListener('click', () => {
        if (data.type === 'bomb') {
          score -= 5;
          if (score < 0) score = 0;
        } else {
          score += data.points;
          speed += 0.1;
        }

        scoreDisplay.textContent = `النقاط: ${score}`;
        speedDisplay.textContent = `السرعة: ${speed.toFixed(1)}`;

        createCircle();
      });

      gameArea.appendChild(circle);
      currentCircle = circle;

      const timeoutDuration = Math.max(400, 2000 - speed * 150);
      timeoutId = setTimeout(createCircle, timeoutDuration);
    }

    function loadLeaderboard() {
      const leaderboard = JSON.parse(localStorage.getItem('speedGameLeaderboard')) || [];
      leaderboardList.innerHTML = '';
      leaderboard.forEach(entry => {
        const li = document.createElement('li');
        li.textContent = `${entry.name} - ${entry.score} نقطة`;
        leaderboardList.appendChild(li);
      });
    }

    function saveScore(name, score) {
      if (!name) name = 'لاعب مجهول';
      const leaderboard = JSON.parse(localStorage.getItem('speedGameLeaderboard')) || [];
      leaderboard.push({ name, score });
      leaderboard.sort((a, b) => b.score - a.score);
      if (leaderboard.length > 5) leaderboard.length = 5;
      localStorage.setItem('speedGameLeaderboard', JSON.stringify(leaderboard));
    }

    function startGame() {
      const name = playerNameInput.value.trim();
      if (!name) {
        alert('يرجى كتابة اسمك أولاً!');
        return;
      }

      score = 0;
      time = 60;
      speed = 1;
      scoreDisplay.textContent = `النقاط: ${score}`;
      timeDisplay.textContent = `الوقت: ${time}`;
      speedDisplay.textContent = `السرعة: ${speed.toFixed(1)}`;
      gameArea.innerHTML = '';
      currentCircle = null;
      startBtn.disabled = true;
      playerNameInput.disabled = true;

      createCircle();

      timerId = setInterval(() => {
        time--;
        timeDisplay.textContent = `الوقت: ${time}`;
        if (time === 0) {
          clearInterval(timerId);
          clearTimeout(timeoutId);
          gameArea.innerHTML = '';
          alert(`انتهى الوقت! نقاطك: ${score}`);
          saveScore(name, score);
          loadLeaderboard();
          startBtn.disabled = false;
          playerNameInput.disabled = false;
          playerNameInput.value = '';
        }
      }, 1000);
    }

    startBtn.addEventListener('click', startGame);
    loadLeaderboard();
  </script>
</body>
</html>
