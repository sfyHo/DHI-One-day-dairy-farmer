# 你的開心牧場

<html>
<head>
  <meta charset="UTF-8" />
  <title>DHI 飼養管理小遊戲</title>
  <style>
    body { font-family: Arial; padding: 20px; }
    button { padding: 10px; margin: 5px; }
    #game, #studentSetup { display:none; max-width: 600px; }
    #rankBoard { margin-top:20px; background:#f0f0f0; padding:15px; }
    #working { text-align:center; margin:20px 0; }
    #working img { width:100px; }
  </style>
</head>
<body>

<!-- 身份選擇 -->
<div id="identitySelect">
  <h2>請選擇身份開始遊戲</h2>
  <button onclick="chooseIdentity('student')">我是學生</button>
  <button onclick="chooseIdentity('tester')">我是系統測試員</button>
</div>

<!-- 學生設定畫面 -->
<div id="studentSetup">
  <h2>學生設定</h2>
  <label>選擇組別：</label>
  <select id="groupSelect">
    <option value="G1">G1</option>
    <option value="G2">G2</option>
    <option value="G3">G3</option>
    <option value="G4">G4</option>
    <option value="G5">G5</option>
    <option value="G6">G6</option>
    <option value="G7">G7</option>
    <option value="G8">G8</option>
    <option value="G9">G9</option>
    <option value="G10">G10</option>
  </select>
  <br><br>
  <label>輸入姓名（最多10字元）：</label>
  <input type="text" id="studentName" maxlength="10" />
  <button onclick="confirmStudent()">確認</button>
  <p id="nameError" style="color:red;"></p>
</div>

<!-- 遊戲主體 -->
<div id="game">
  <h2>🐄 DHI 飼養管理小遊戲</h2>
  <p id="scenario"></p>
  <div id="options"></div>
  <div id="working"></div>
  <p id="result"></p>
  <p>總收益：<span id="score">0</span></p>
  <div id="rankBoard"></div>
  <div id="wrongAnswersDiv"></div>
</div>

<audio id="bgm" loop>
  <source src="https://www.bensound.com/bensound-music/bensound-sunny.mp3" type="audio/mpeg">
</audio>

<script>
const scenarios = [
  {
    description: "乳量下降 10%，體細胞上升至 380k。",
    options: [
      { text: "改善牛床乾燥度與墊料", effect: 8, msg: "體細胞下降，乳量回升！", correct:true, reason:"改善環境可降低乳房炎風險" },
      { text: "濃料比例提高 5%", effect: -3, msg: "乳量未改善，反而有亞臨床乳房炎風險。", correct:false, reason:"濃料過多可能造成健康問題" },
      { text: "增加擠乳頻率到每日 3 次", effect: 4, msg: "乳量小幅上升。", correct:false, reason:"可提升乳量，但未根本改善乳房健康" }
    ]
  },
  {
    description: "泌乳初期（30 DIM）乳脂率僅 2.8%，疑似負能量平衡。",
    options: [
      { text: "提高乾物攝取量、改善日糧適口性", effect: 7, msg: "DMI 上升，乳脂正常化！", correct:true, reason:"增加能量攝取改善乳脂率" },
      { text: "減少飼料量以避免乳脂過高", effect: -4, msg: "問題更嚴重，能量不足！", correct:false, reason:"減少飼料會惡化負能量平衡" }
    ]
  }
];

let current = 0;
let score = 0;
let identity = "";
let studentName = "";
let studentGroup = "";
let wrongAnswers = [];

function getTodayTW() {
  const now = new Date();
  const utc = now.getTime() + (now.getTimezoneOffset() * 60000);
  const tw = new Date(utc + 8 * 60 * 60000);
  return tw.toISOString().slice(0,10);
}

function checkDailyReset() {
  const today = getTodayTW();
  const last = localStorage.getItem("score_last_reset");
  if (last !== today) {
    localStorage.setItem("score_last_reset", today);
    localStorage.setItem("student_scores", JSON.stringify([]));
  }
}

function chooseIdentity(type) {
  identity = type;
  if(identity === "student") {
    document.getElementById("identitySelect").style.display = "none";
    document.getElementById("studentSetup").style.display = "block";
  } else {
    document.getElementById("identitySelect").style.display = "none";
    startGame();
  }
}

function confirmStudent() {
  const name = document.getElementById("studentName").value.trim();
  if(!name || [...name].length > 10) {
    document.getElementById("nameError").innerText = "字元數超過10個，請縮短並重新輸入名稱";
    return;
  }
  studentName = name;
  studentGroup = document.getElementById("groupSelect").value;
  document.getElementById("studentSetup").style.display = "none";
  startGame();
}

function startGame() {
  checkDailyReset();
  document.getElementById("game").style.display = "block";
  document.getElementById("bgm").play();
  loadQuestion();
}

function loadQuestion() {
  const s = scenarios[current];
  document.getElementById("scenario").innerText = `情境：${s.description}`;
  const optionsDiv = document.getElementById("options");
  optionsDiv.innerHTML = "";
  document.getElementById("result").innerText = "";
  document.getElementById("working").innerHTML = "";

  s.options.forEach((opt, idx) => {
    const btn = document.createElement("button");
    btn.textContent = opt.text;
    btn.onclick = () => chooseWithEffect(idx);
    optionsDiv.appendChild(btn);
  });
}

function chooseWithEffect(idx) {
  const workingDiv = document.getElementById("working");
  workingDiv.innerHTML = '<p>泌乳中...</p><img src="https://i.imgur.com/7x2RJ4G.gif" />';
  document.getElementById("options").querySelectorAll("button").forEach(b=>b.disabled=true);

  setTimeout(() => {
    workingDiv.innerHTML = "";
    choose(idx);
  },3000);
}

function choose(idx) {
  const s = scenarios[current];
  const opt = s.options[idx];

  score += opt.effect;
  document.getElementById("score").innerText = score;
  document.getElementById("result").innerText =
    `結果：${opt.msg}（收益 ${opt.effect > 0 ? "+" : ""}${opt.effect}）`;

  if(!opt.correct && identity==="student") {
    wrongAnswers.push({
      question: s.description,
      wrongChoice: opt.text,
      correctChoice: s.options.find(o=>o.correct).text,
      reason: s.options.find(o=>o.correct).reason
    });
  }

  setTimeout(nextQuestion, 1000);
}

function nextQuestion() {
  current++;
  if(current >= scenarios.length) {
    endGame();
    return;
  }
  loadQuestion();
}

function endGame() {
  document.getElementById("scenario").innerText = "🎉 遊戲結束！";
  document.getElementById("options").innerHTML = "";
  document.getElementById("working").innerHTML = "";
  document.getElementById("result").innerText = "";

  if(identity==="student") {
    saveStudentScore(score);
    showRankBoard();
    showWrongAnswers();
  }
}

function saveStudentScore(s) {
  let arr = JSON.parse(localStorage.getItem("student_scores") || "[]");
  if(!arr.includes(s)) arr.push(s);
  arr.sort((a,b)=>b-a);
  localStorage.setItem("student_scores", JSON.stringify(arr));
}

function showRankBoard() {
  const arr = JSON.parse(localStorage.getItem("student_scores") || "[]");
  let html = "<h3>📊 今日學生排名</h3><ol>";
  arr.forEach(s => html += `<li>${s} 分</li>`);
  html += "</ol>";
  document.getElementById("rankBoard").innerHTML = html;
}

function showWrongAnswers() {
  if(wrongAnswers.length===0) return;
  let html = "<h3>❌ 錯題檢討表</h3><ul>";
  wrongAnswers.forEach(w=>{
    html += `<li>題目：${w.question}<br>你選擇：${w.wrongChoice}<br>建議答案：${w.correctChoice}<br>原因：${w.reason}</li><br>`;
  });
  html += "</ul>";
  document.getElementById("wrongAnswersDiv").innerHTML = html;
}
</script>
</body>
</html>"
