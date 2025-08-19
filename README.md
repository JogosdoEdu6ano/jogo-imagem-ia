
<!DOCTYPE html> 
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Jogo - Imagem Real ou IA?</title>
<style>
body { font-family: Arial; text-align: center; background: #f5f5f5; margin:0; padding:20px; }
#game,#result,#setup { display:none; opacity:0; transition: opacity 0.8s ease; }
#game.show,#result.show,#setup.show { display:block; opacity:1; }
.images { display:flex; justify-content:center; gap:20px; margin:20px 0; }
.images img { width:250px; height:200px; object-fit:cover; border:3px solid #333; cursor:pointer; border-radius:10px; transition:transform 0.3s, border 0.3s, box-shadow 0.3s; }
.images img:hover { transform:scale(1.05); }
.correct { border:5px solid #4CAF50 !important; box-shadow:0 0 20px #4CAF50; animation: glow 0.8s ease; }
@keyframes glow { 0% { box-shadow:0 0 0 #4CAF50; } 50% { box-shadow:0 0 25px #4CAF50; } 100% { box-shadow:0 0 20px #4CAF50; } }
button { padding:10px 20px; font-size:16px; margin-top:20px; cursor:pointer; }
fieldset { margin:10px auto; width:80%; text-align:left; }
#ranking { margin-top:20px; text-align:center; }
#ranking h3 { margin-bottom:5px; }
#ranking ol { list-style:none; padding:0; }
#ranking li { background:#fff; margin:3px auto; padding:6px; border-radius:8px; width:220px; box-shadow:0 2px 4px rgba(0,0,0,0.1); }
#timer { font-size:20px; font-weight:bold; margin-top:10px; color:#d9534f; }
</style>
</head>
<body>
<h1>Jogo: Ache a Imagem Real</h1>

<div id="setup">
<h2>Carregue as imagens para as 10 rodadas</h2>
<form id="uploadForm">
<div id="roundUploads"></div>
<button type="button" onclick="finishUpload()">Confirmar imagens</button>
</form>
</div>

<div id="start">
<p>Digite seu nome para começar:</p>
<input type="text" id="playerName" placeholder="Seu nome"><br><br>
<button onclick="prepareUpload()">Carregar Imagens</button>
<div id="ranking"></div>
</div>

<div id="game">
<h2 id="roundTitle"></h2>
<div class="images">
<img id="img1" onclick="chooseImage(0)">
<img id="img2" onclick="chooseImage(1)">
</div>
<p id="feedback"></p>
<p id="timer"></p>
<button onclick="nextRound()" id="nextBtn" style="display:none;">Próxima</button>
</div>

<div id="result">
<h2 id="finalMessage"></h2>
<p id="finalScore"></p>
<button onclick="restartGame()">Jogar Novamente</button>
<div id="ranking"></div>
</div>

<!-- Sons -->
<audio id="soundCorrect" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
<audio id="soundFast" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
<audio id="soundWrong" src="https://actions.google.com/sounds/v1/cartoon/slide_whistle_down.ogg"></audio>
<audio id="soundRecord" src="https://actions.google.com/sounds/v1/alarms/digital_watch_alarm_long.ogg"></audio>

<script>
let playerName="", currentRound=0, score=0, rounds=[], correctIndex=0;
let timer, timeLeft=10, answered=false;

function playSound(id){ let s=document.getElementById(id); s.currentTime=0; s.play(); }

function showElement(id){ document.getElementById(id).classList.add("show"); }
function hideElement(id){ document.getElementById(id).classList.remove("show"); }

function prepareUpload() {
  playerName=document.getElementById("playerName").value;
  if(playerName.trim()===""){ alert("Digite seu nome!"); return; }
  hideElement("start"); setTimeout(()=>showElement("setup"),500);
  let container=document.getElementById("roundUploads"); container.innerHTML="";
  for(let i=1;i<=10;i++){
    container.innerHTML+=`<fieldset><legend>Rodada ${i}</legend>
    Imagem Real: <input type="file" accept="image/*" id="real${i}" required><br><br>
    Imagem IA: <input type="file" accept="image/*" id="ai${i}" required></fieldset>`;
  }
}

function finishUpload() {
  rounds=[];
  for(let i=1;i<=10;i++){
    let realFile=document.getElementById("real"+i).files[0];
    let aiFile=document.getElementById("ai"+i).files[0];
    if(!realFile || !aiFile){ alert(`Selecione as duas imagens da rodada ${i}`); return; }
    let realURL=URL.createObjectURL(realFile);
    let aiURL=URL.createObjectURL(aiFile);
    rounds.push({real:realURL, ai:aiURL});
  }
  hideElement("setup"); setTimeout(()=>{ showElement("game"); loadRound(); },500);
}

function loadRound() {
  answered=false; clearInterval(timer); timeLeft=10;
  document.getElementById("timer").textContent=`⏳ Tempo: ${timeLeft}s`;
  document.getElementById("feedback").textContent="";
  document.getElementById("nextBtn").style.display="none";
  let round=rounds[currentRound];
  document.getElementById("roundTitle").textContent=`Rodada ${currentRound+1} de ${rounds.length}`;
  let order=Math.random()<0.5?[round.real,round.ai]:[round.ai,round.real];
  correctIndex=order.indexOf(round.real);
  document.getElementById("img1").src=order[0]; document.getElementById("img2").src=order[1];
  document.getElementById("img1").classList.remove("correct"); 
  document.getElementById("img2").classList.remove("correct");

  timer=setInterval(()=>{
    timeLeft--;
    document.getElementById("timer").textContent=`⏳ Tempo: ${timeLeft}s`;
    if(timeLeft<=0){
      clearInterval(timer);
      if(!answered){
        document.getElementById("feedback").textContent="⏰ Tempo esgotado! ❌";
        showCorrect(); playSound("soundWrong");
        document.getElementById("nextBtn").style.display="inline-block";
        answered=true;
      }
    }
  },1000);
}

function chooseImage(choice){
  if(answered) return;
  answered=true; clearInterval(timer);
  if(choice===correctIndex){
    let points=(10-timeLeft<=3)?2:1;
    score+=points;
    if(points===2){ document.getElementById("feedback").textContent="⚡ Acertou rápido! +2 pontos!"; playSound("soundFast"); }
    else{ document.getElementById("feedback").textContent="✅ Acertou! +1 ponto!"; playSound("soundCorrect"); }
  } else {
    document.getElementById("feedback").textContent="❌ Errou!"; playSound("soundWrong");
  }
  showCorrect();
  document.getElementById("nextBtn").style.display="inline-block";
}

function showCorrect(){
  if(correctIndex===0) document.getElementById("img1").classList.add("correct");
  else document.getElementById("img2").classList.add("correct");
}

function nextRound(){
  hideElement("game");
  setTimeout(()=>{
    currentRound++; 
    if(currentRound<rounds.length){ showElement("game"); loadRound(); }
    else{ showElement("result"); endGame(); }
  },600);
}

function endGame(){
  clearInterval(timer);
  document.getElementById("finalMessage").textContent=`Fim do jogo, ${playerName}!`;
  document.getElementById("finalScore").textContent=`Sua pontuação: ${score} de ${rounds.length}`;
  saveRanking(playerName,score); showRanking();
}

function restartGame(){ currentRound=0; score=0; hideElement("result"); setTimeout(()=>{ showElement("game"); loadRound(); },500); }

function saveRanking(name,points){
  let ranking=JSON.parse(localStorage.getItem("ranking"))||[];
  let topScore=ranking.length>0?ranking[0].points:0;
  ranking.push({name,points}); ranking.sort((a,b)=>b.points-a.points); ranking=ranking.slice(0,5);
  localStorage.setItem("ranking",JSON.stringify(ranking));
  if(points>topScore){ alert("🏆 Novo recorde! Parabéns!"); playSound("soundRecord"); }
}

function showRanking(){
  let ranking=JSON.parse(localStorage.getItem("ranking"))||[];
  let html="<h3>🏆 Ranking</h3><ol>";
  if(ranking.length===0){ html+="<li>Ainda não há jogadores</li>"; }
  else{ ranking.forEach((r,i)=>{ html+=`<li>${i+1}º - ${r.name}: ${r.points} pontos</li>`; }); }
  html+="</ol>";
  document.querySelectorAll("#ranking").forEach(el=>el.innerHTML=html);
}

showRanking();
</script>
</body>
</html>

