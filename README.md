<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>DHI 飼養管理小遊戲</title>
  <style>
    body { font-family: Arial; padding: 20px; }
    button { padding: 10px; margin: 5px; }
    #game { max-width: 600px; display:none; }
    #identitySelect { margin-bottom:20px; }
    #rankBoard { margin-top:20px; background:#f0f0f0; padding:15px; }
  </style>
</head>
<body>

<!-- 身份選擇 -->
<div id="identitySelect">
  <h2>請選擇身份開始遊戲</h2>
  <button onclick="startGame('student')">我是學生</button>
  <button onclick="startGame('tester')">我是系統測試員</button>
</div>

<!-- 遊戲主體 -->
<div id="game">
  <h2>🐄 DHI 飼養管理小遊戲</h2>
  <p id="scenario"></p>
  <div id="options"></div>
  <p id="result"></p>
  <p>總收益：<span id="score">0</span></p>
  <button onclick="nextQuestion()">下一題</button>

  <div id="rankBoard"></div>
</div>

<script>
/* ------------------------------
   題庫
------------------------------ */
const scenarios = [
  {
    description: "乳量下降 10%，體細胞上升至 380k。",
    options: [
      { text: "改善牛床乾燥度與墊料", effect: 8, msg: "體細胞下降，乳量回升！" },
      { text: "濃料比例提高 5%", effect: -3, msg: "乳量未改善，反而有亞臨床乳房炎風險。" },
      { text: "增加擠乳頻率到每日 3 次", effect: 4, msg: "乳量小幅上升。" }
    ]
  },
  {
    description: "泌乳初期（30 DIM）乳脂率僅 2.8%，疑似負能量平衡。",
    options: [
      { text: "提高乾物攝取量、改善日糧適口性", effect: 7, msg: "DMI 上升，乳脂正常化！" },
      { text: "減少飼料量以避免乳脂過高", effect: -4, msg: "問題更嚴重，能量不足！" }
    ]
  }
];

let current = 0;
let score = 0;
let identity = "";  // student / tester

/* ------------------------------
   日期檢查（台灣時間）
------------------------------ */
function getTodayTW() {
  const now = new Date();
  const utc = now.getTime() + (now.getTimezoneOffset() * 60000);
  const tw = new Date(utc + 8 * 60 * 60000); // UTC+8
  return tw.toISOString().slice(0,10); // yyyy-mm-dd
}

function checkDailyReset() {
  const today = getTodayTW();
  const last = localStorage.getItem("score_last_reset");

  if (last !== today) {
    localStorage.setItem("score_last_reset", today);
    localStorage.setItem("student_scores", JSON.stringify([]));
  }
}

/* ------------------------------
   身份啟動遊戲
------------------------------ */
function startGame(type) {
  identity = type;

  // 檢查是否需清空紀錄
  checkDailyReset();

  // 顯示遊戲畫面
  document.getElementById("identitySelect").style.display = "none";
  document.getElementById("game").style.display = "block";

  loadQuestion();
}

/* ------------------------------
   題目載入
------------------------------ */
function loadQuestion() {
  const s = scenarios[current];
  document.getElementById("scenario").innerText = `情境：${s.description}`;
  document.getElementById("options").innerHTML = "";

  s.options.forEach((opt, idx) => {
    const btn = document.createElement("button");
    btn.textContent = opt.text;
    btn.onclick = () => choose(idx);
    document.getElementById("options").appendChild(btn);
  });
}

function choose(idx) {
  const s = scenarios[current];
  const opt = s.options[idx];

  score += opt.effect;
  document.getElementById("score").innerText = score;
  document.getElementById("result").innerText =
    `結果：${opt.msg}（收益 ${opt.effect > 0 ? "+" : ""}${opt.effect}）`;
}

function nextQuestion() {
  current++;
  document.getElementById("result").innerText = "";

  if (current >= scenarios.length) {
    endGame();
    return;
  }
  loadQuestion();
}

/* ------------------------------
   遊戲結束 + 排行榜
------------------------------ */
function endGame() {
  document.getElementById("scenario").innerText = "🎉 遊戲結束！";
  document.getElementById("options").innerHTML = "";

  if (identity === "student") {
    saveStudentScore(score);
  }

  showRankBoard();
}

function saveStudentScore(s) {
  let arr = JSON.parse(localStorage.getItem("student_scores") || "[]");
  arr.push(s);
  arr.sort((a,b)=>b-a); // 由高到低
  localStorage.setItem("student_scores", JSON.stringify(arr));
}

function showRankBoard() {
  const arr = JSON.parse(localStorage.getItem("student_scores") || "[]");

  if (arr.length === 0) {
    document.getElementById("rankBoard").innerHTML =
      "<p>📄 今日尚無學生成績紀錄。</p>";
    return;
  }

  let html = "<h3>📊 今日學生排名</h3><ol>";
  arr.forEach(s => html += `<li>${s} 分</li>`);
  html += "</ol>";

  document.getElementById("rankBoard").innerHTML = html;
}

</script>
</body>
</html>
