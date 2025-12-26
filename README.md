<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>اختبار الصداقة</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
@import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;800&display=swap');

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: 'Cairo', sans-serif;
  min-height: 100vh;
  background: radial-gradient(circle at top, #7f7fd5, #86a8e7, #91eae4);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 15px;
}

.app {
  width: 100%;
  max-width: 420px;
  backdrop-filter: blur(20px);
  background: rgba(255,255,255,0.25);
  border-radius: 24px;
  padding: 25px;
  box-shadow: 0 30px 60px rgba(0,0,0,.25);
  animation: fade 0.6s ease;
}

@keyframes fade {
  from { opacity:0; transform:translateY(20px); }
  to { opacity:1; transform:none; }
}

h1,h2 {
  margin: 0 0 15px;
  font-weight: 800;
}

p {
  opacity: .8;
  margin-bottom: 15px;
}

input {
  width: 100%;
  padding: 14px;
  border-radius: 14px;
  border: none;
  font-size: 16px;
  outline: none;
  margin-bottom: 15px;
}

button {
  width: 100%;
  padding: 14px;
  border-radius: 14px;
  border: none;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
  background: linear-gradient(135deg,#667eea,#764ba2);
  color: #fff;
  transition: 0.2s;
}

button:hover {
  transform: scale(1.03);
}

.answers button {
  background: rgba(255,255,255,0.85);
  color: #333;
  margin-top: 10px;
}

.progress {
  height: 6px;
  background: rgba(255,255,255,0.3);
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 15px;
}

.bar {
  height: 100%;
  width: 0%;
  background: linear-gradient(90deg,#ff512f,#dd2476);
  transition: width 0.3s;
}

.rank {
  text-align: right;
  margin-top: 10px;
  font-weight: 600;
}

.share {
  background: linear-gradient(135deg,#25D366,#1da851);
  margin-top: 10px;
}
</style>
</head>

<body>

<div class="app" id="app"></div>

<script>
const questions = [
 {q:"🎨 لونك المفضل؟",a:["أسود","أزرق","أحمر","أخضر"]},
 {q:"☕ مشروبك المفضل؟",a:["قهوة","شاي"]},
 {q:"🌙 تفضل؟",a:["السهر","النوم مبكرًا"]},
 {q:"✈️ تحب؟",a:["السفر","الجلوس في البيت"]},
 {q:"📱 هاتفك؟",a:["أندرويد","آيفون"]}
];

const app = document.getElementById("app");
const params = new URLSearchParams(location.search);
const host = params.get("host");

let i = 0, answers = [], score = 0, player = "";

if(!host){
  home();
}else{
  guest();
}

function home(){
  app.innerHTML = `
    <h1>🤝 اختبار الصداقة</h1>
    <p>أنشئ اختبارك وشارك الرابط مع أصدقائك</p>
    <input id="name" placeholder="اسمك">
    <button onclick="startHost()">ابدأ</button>
  `;
}

function startHost(){
  const name = document.getElementById("name").value.trim();
  if(!name) return alert("اكتب اسمك");
  localStorage.setItem("answers_"+name,JSON.stringify([]));
  play(name,true);
}

function guest(){
  app.innerHTML = `
    <h1>🎯 اختبار ${host}</h1>
    <input id="player" placeholder="اسمك">
    <button onclick="join()">ابدأ التحدي</button>
  `;
}

function join(){
  player = document.getElementById("player").value.trim();
  if(!player) return alert("اكتب اسمك");
  play(host,false);
}

function play(owner,isHost){
  i=0; answers=[]; score=0;
  showQ(owner,isHost);
}

function showQ(owner,isHost){
  if(i>=questions.length) return finish(owner,isHost);
  const q = questions[i];
  app.innerHTML = `
    <div class="progress"><div class="bar" style="width:${(i/questions.length)*100}%"></div></div>
    <h2>${q.q}</h2>
    <div class="answers">
      ${q.a.map((x,n)=>`<button onclick="pick(${n},'${owner}',${isHost})">${x}</button>`).join("")}
    </div>
  `;
}

function pick(n,owner,isHost){
  answers.push(n);
  i++;
  showQ(owner,isHost);
}

function finish(owner,isHost){
  if(isHost){
    localStorage.setItem("answers_"+owner,JSON.stringify(answers));
    const link = location.href + "?host=" + owner;
    app.innerHTML = `
      <h2>✅ تم إنشاء اختبارك</h2>
      <input value="${link}" readonly>
      <button class="share" onclick="navigator.share({url:'${link}'})">مشاركة الرابط</button>
    `;
  }else{
    const correct = JSON.parse(localStorage.getItem("answers_"+owner)||"[]");
    correct.forEach((x,n)=>{ if(x===answers[n]) score++; });

    const list = JSON.parse(localStorage.getItem("rank_"+owner)||"[]");
    list.push({name:player,score});
    list.sort((a,b)=>b.score-a.score);
    localStorage.setItem("rank_"+owner,JSON.stringify(list));

    app.innerHTML = `
      <h2>⭐ نتيجتك ${score}/${questions.length}</h2>
      <p>🏆 ترتيب الأصدقاء</p>
      <div class="rank">
        ${list.map((x,n)=>`${n+1}. ${x.name} (${x.score})`).join("<br>")}
      </div>
    `;
  }
}
</script>

</body>
</html>quiz-pro.html
 
