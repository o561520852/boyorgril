# boyorgril
boyorgril
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>猜寶寶性別闖關遊戲</title>
<style>
  body {
    font-family: "Noto Sans TC", sans-serif;
    background: linear-gradient(135deg, #ffd6e8, #d7ecff);
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    margin: 0;
    text-align: center;
    color: #444;
  }

  .card {
    background: #fff;
    padding: 30px;
    border-radius: 20px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    max-width: 360px;
    width: 90%;
    transition: all 0.5s ease;
    animation: fadeIn 1s;
    z-index: 10;
  }

  h1 {
    color: #ff8fab;
  }

  button {
    background: #89cff0;
    border: none;
    padding: 12px 24px;
    border-radius: 12px;
    margin: 8px;
    font-size: 16px;
    color: #fff;
    cursor: pointer;
    transition: 0.3s;
  }

  button:hover {
    background: #6ab7e1;
  }

  .hidden {
    display: none;
  }

  @keyframes fadeIn {
    from {opacity: 0; transform: translateY(10px);}
    to {opacity: 1; transform: translateY(0);}
  }

  /* 翻牌動畫 */
  .flip-box {
    background-color: transparent;
    width: 200px;
    height: 120px;
    perspective: 1000px;
    margin: 20px auto;
  }

  .flip-inner {
    position: relative;
    width: 100%;
    height: 100%;
    transform-style: preserve-3d;
    transition: transform 2s ease-in-out;
  }

  .flip-box.flip .flip-inner {
    transform: rotateY(720deg);
  }

  .flip-front, .flip-back {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    background: #fce4ec;
    border-radius: 15px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 28px;
    color: #ff69b4;
  }

  .flip-back {
    background: #ffe4b5;
    transform: rotateY(180deg);
  }

  /* 🎈氣球動畫 */
  .balloon {
    position: absolute;
    bottom: -150px;
    width: 40px;
    height: 55px;
    background-color: pink;
    border-radius: 50%;
    opacity: 0.7;
    animation: float 10s linear infinite;
  }

  @keyframes float {
    0% {transform: translateY(0) scale(1);}
    100% {transform: translateY(-120vh) scale(1.2);}
  }

  /* 🌸花瓣動畫 */
  .petal {
    position: fixed;
    top: -20px;
    width: 20px;
    height: 20px;
    background-color: #ffb6c1;
    border-radius: 50%;
    opacity: 0.8;
    pointer-events: none;
    animation: fall 8s linear infinite;
  }

  @keyframes fall {
    0% {transform: translateY(0) rotate(0deg);}
    100% {transform: translateY(100vh) rotate(360deg);}
  }
</style>
</head>
<body>

<!-- 背景氣球 -->
<div id="balloons"></div>

<!-- 第一關 -->
<div class="card" id="stage1">
  <h1>第一關：味覺挑戰 🍋🍰</h1>
  <p>孕媽最近最愛吃什麼？</p>
  <button onclick="startGame(); nextStage(2)">酸酸的水果</button>
  <button onclick="startGame(); nextStage(2)">甜甜的蛋糕</button>
</div>

<!-- 第二關 -->
<div class="card hidden" id="stage2">
  <h1>第二關：小動作觀察 👶</h1>
  <p>寶寶在肚子裡最常怎麼動？</p>
  <button onclick="nextStage(3)">踢得像在踢足球 ⚽️</button>
  <button onclick="nextStage(3)">滾來滾去像在跳舞 💃</button>
</div>

<!-- 第三關 -->
<div class="card hidden" id="stage3">
  <h1>第三關：直覺選擇 💭</h1>
  <p>你覺得寶寶的性別是？</p>
  <button onclick="nextStage(4)">男孩 👦</button>
  <button onclick="nextStage(4)">女孩 👧</button>
</div>

<!-- 第四關 -->
<div class="card hidden" id="stage4">
  <h1>第四關：真相揭曉 🎠</h1>
  <p>翻轉看看寶寶是弟弟還是妹妹！</p>

  <div class="flip-box" id="flipBox">
    <div class="flip-inner">
      <div class="flip-front">👶 弟弟</div>
      <div class="flip-back">🎀 妹妹</div>
    </div>
  </div>

  <button onclick="revealAnswer()">翻轉開始！</button>
  <p id="finalMsg" style="display:none; font-size:20px; color:#ff69b4; font-weight:bold;">💖 是可愛的小妹妹！💖</p>
</div>

<!-- 背景音樂（活潑版） -->
<audio id="bgm" loop>
  <source src="https://cdn.pixabay.com/download/audio/2023/03/01/audio_6f6b2292a3.mp3?filename=happy-day-140738.mp3" type="audio/mpeg">
</audio>

<script>
let musicStarted = false;

function startGame() {
  if (!musicStarted) {
    const bgm = document.getElementById('bgm');
    bgm.play().catch(()=>{}); // 嘗試播放（若被阻擋不報錯）
    musicStarted = true;
  }
}

function nextStage(n) {
  document.querySelectorAll('.card').forEach(c => c.classList.add('hidden'));
  document.getElementById('stage' + n).classList.remove('hidden');

  // 第四關時生成花瓣效果
  if (n === 4) createPetals();
}

// 🎈動態生成氣球
function createBalloons() {
  const container = document.getElementById('balloons');
  for (let i = 0; i < 15; i++) {
    const b = document.createElement('div');
    b.classList.add('balloon');
    b.style.left = Math.random() * 100 + 'vw';
    b.style.backgroundColor = ['#ffd6e8', '#aee3ff', '#ffe4b5', '#fcbad3'][Math.floor(Math.random() * 4)];
    b.style.animationDuration = (8 + Math.random() * 6) + 's';
    container.appendChild(b);
  }
}
createBalloons();

// 🌸 產生花瓣
function createPetals() {
  for (let i = 0; i < 25; i++) {
    const petal = document.createElement('div');
    petal.classList.add('petal');
    petal.style.left = Math.random() * 100 + 'vw';
    petal.style.animationDuration = (5 + Math.random() * 5) + 's';
    petal.style.backgroundColor = ['#ffb6c1','#ffc0cb','#ffe4e1'][Math.floor(Math.random()*3)];
    document.body.appendChild(petal);
    setTimeout(()=>petal.remove(),10000);
  }
}

// 🎀 翻轉效果
function revealAnswer() {
  const box = document.getElementById('flipBox');
  const msg = document.getElementById('finalMsg');
  box.classList.add('flip');
  setTimeout(() => {
    msg.style.display = 'block';
  }, 2200);
}
</script>

</body>
</html
