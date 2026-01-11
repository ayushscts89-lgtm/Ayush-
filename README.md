# Ayush-
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Classic Grandmaster Chess</title>
<style>
body{margin:0;font-family:"Georgia","Times New Roman",serif;height:100vh;display:flex;align-items:center;justify-content:center;background:#2b1d10;color:#f5f5f5;user-select:none}
body.wood{background:radial-gradient(circle at top,#4b2e14,#1b1208)}
body.dark{background:#0b1220}
body.light{background:#e5e7eb;color:#000}
#home,#game{width:100%;height:100vh;display:flex;align-items:center;justify-content:center}
#home{flex-direction:column;gap:20px;text-align:center}
#home h1{font-size:56px;color:#f7e7c6}
#home button{padding:16px 38px;font-size:20px;border-radius:14px;border:none;cursor:pointer}
#game{display:none;gap:24px;flex-wrap:wrap}
#board{display:grid;grid-template-columns:repeat(8,70px);grid-template-rows:repeat(8,70px);border-radius:18px;overflow:hidden;border:14px solid #6b4226}
.square{display:flex;align-items:center;justify-content:center;font-size:44px;cursor:pointer}
.light{background:#deb887}
.dark{background:#8b5a2b}
.square.highlight{box-shadow:inset 0 0 0 4px gold}
.square.move{box-shadow:inset 0 0 0 4px rgba(0,0,0,.45)}
.white-piece{color:#fff}
.black-piece{color:#000}
#panel{width:300px;background:rgba(0,0,0,.65);padding:18px;border-radius:18px}
button,select{width:100%;margin-top:10px;padding:12px;border-radius:10px;border:none;font-size:15px}
#status{text-align:center;margin-top:10px;font-weight:bold;cursor:pointer}
#moves{margin-top:12px;max-height:180px;overflow-y:auto;font-size:14px}
#popup{position:fixed;inset:0;background:rgba(0,0,0,.75);display:none;align-items:center;justify-content:center}
#popup div{background:#1e293b;padding:34px;border-radius:20px;text-align:center}
</style>
<script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
</head>
<body class="wood">

<div id="home">
  <h1>♚ Grandmaster Chess ♛</h1>
  <button id="startBtn">Enter the Board</button>
</div>

<div id="game">
  <div id="board"></div>
  <div id="panel">
    <select id="themeSel"><option value="wood">Wood</option><option value="dark">Dark</option><option value="light">Light</option></select>
    <button id="aiBtn">Toggle AI</button>
    <button id="undoBtn">Undo</button>
    <button id="resetBtn">Restart</button>
    <button id="homeBtn">⬅ Home</button>
    <div id="status">White to move</div>
    <div id="moves"></div>
  </div>
</div>

<div id="popup"><div><h2 id="popupText"></h2><button id="newBtn">New Game</button></div></div>

<script>
// ===== SAFE, FIXED & STABLE CHESS ENGINE =====

// DOM
const home=document.getElementById('home');
const game=document.getElementById('game');
const boardEl=document.getElementById('board');
const statusEl=document.getElementById('status');
const movesEl=document.getElementById('moves');
const popup=document.getElementById('popup');
const popupText=document.getElementById('popupText');
const startBtn=document.getElementById('startBtn');
const resetBtn=document.getElementById('resetBtn');
const homeBtn=document.getElementById('homeBtn');
const undoBtn=document.getElementById('undoBtn');
const aiBtn=document.getElementById('aiBtn');
const themeSel=document.getElementById('themeSel');
const newBtn=document.getElementById('newBtn');

// STATE
let board,turn,selected,history,vsAI=false;

const winSound=new Audio('https://assets.mixkit.co/active_storage/sfx/2019/2019-preview.mp3');
const pieces={r:'♜',n:'♞',b:'♝',q:'♛',k:'♚',p:'♟',R:'♖',N:'♘',B:'♗',Q:'♕',K:'♔',P:'♙'};

function initialBoard(){return[['r','n','b','q','k','b','n','r'],['p','p','p','p','p','p','p','p'],['','','','','','','',''],['','','','','','','',''],['','','','','','','',''],['','','','','','','',''],['P','P','P','P','P','P','P','P'],['R','N','B','Q','K','B','N','R']]}

// ===== GAME FLOW =====
function startGame(){resetGame();home.style.display='none';game.style.display='flex'}
function resetGame(){board=initialBoard();turn='white';selected=null;history=[];movesEl.innerHTML='';popup.style.display='none';statusEl.style.display='block';statusEl.textContent='White to move';drawBoard()}
function goHome(){game.style.display='none';home.style.display='flex'}

// ===== BOARD RENDER =====
function drawBoard(){boardEl.innerHTML='';for(let r=0;r<8;r++)for(let c=0;c<8;c++){const s=document.createElement('div');s.className='square '+((r+c)%2?'dark':'light');const p=board[r][c];if(p){s.textContent=pieces[p];s.classList.add(p===p.toUpperCase()?'white-piece':'black-piece')}s.onclick=()=>clickSquare(r,c,s);boardEl.appendChild(s)}}

function isTurn(p){return turn==='white'?p===p.toUpperCase():p===p.toLowerCase()}

// ===== INPUT =====
function clickSquare(r,c,el){if(selected){if(isLegal(selected.r,selected.c,r,c))makeMove(selected.r,selected.c,r,c);clearMarks();selected=null}else if(board[r][c]&&isTurn(board[r][c])){selected={r,c};el.classList.add('highlight');showMoves(r,c)}}

// ===== MOVE RULES =====
function isLegal(sr,sc,dr,dc){if(sr===dr&&sc===dc)return false;if(board[dr][dc]&&isTurn(board[dr][dc]))return false;const piece=board[sr][sc];if(!piece)return false;const p=piece.toLowerCase();const dir=piece==='P'?-1:1;let ok=false;if(p==='p'){if(sc===dc&&!board[dr][dc]&&dr===sr+dir)ok=true;if(Math.abs(sc-dc)===1&&board[dr][dc]&&dr===sr+dir)ok=true}if(p==='r'&&(sr===dr||sc===dc))ok=clear(sr,sc,dr,dc);if(p==='b'&&Math.abs(sr-dr)===Math.abs(sc-dc))ok=clear(sr,sc,dr,dc);if(p==='q'&&(sr===dr||sc===dc||Math.abs(sr-dr)===Math.abs(sc-dc)))ok=clear(sr,sc,dr,dc);if(p==='n'&&Math.abs(sr-dr)*Math.abs(sc-dc)===2)ok=true;if(p==='k'&&Math.max(Math.abs(sr-dr),Math.abs(sc-dc))===1)ok=true;if(!ok)return false;
// king safety
const snap=JSON.parse(JSON.stringify(board));snap[dr][dc]=snap[sr][sc];snap[sr][sc]='';return !isCheck(turn,snap)}

function clear(sr,sc,dr,dc){const rs=Math.sign(dr-sr),cs=Math.sign(dc-sc);let r=sr+rs,c=sc+cs;while(r!==dr||c!==dc){if(board[r][c])return false;r+=rs;c+=cs}return true}

// ===== CHECK / CHECKMATE =====
function isCheck(color,b){const king=color==='white'?'K':'k';let kr,kc;for(let r=0;r<8;r++)for(let c=0;c<8;c++)if(b[r][c]===king){kr=r;kc=c}for(let r=0;r<8;r++)for(let c=0;c<8;c++){const p=b[r][c];if(!p)continue;if(color==='white'?p===p.toLowerCase():p===p.toUpperCase()){if(isAttack(r,c,kr,kc,b))return true}}return false}

function isAttack(sr,sc,dr,dc,b){const p=b[sr][sc].toLowerCase();const dir=b[sr][sc]==='P'?-1:1;if(p==='p')return Math.abs(sc-dc)===1&&dr===sr+dir;if(p==='r'&&(sr===dr||sc===dc))return clearPath(sr,sc,dr,dc,b);if(p==='b'&&Math.abs(sr-dr)===Math.abs(sc-dc))return clearPath(sr,sc,dr,dc,b);if(p==='q'&&(sr===dr||sc===dc||Math.abs(sr-dr)===Math.abs(sc-dc)))return clearPath(sr,sc,dr,dc,b);if(p==='n'&&Math.abs(sr-dr)*Math.abs(sc-dc)===2)return true;if(p==='k'&&Math.max(Math.abs(sr-dr),Math.abs(sc-dc))===1)return true;return false}

function clearPath(sr,sc,dr,dc,b){const rs=Math.sign(dr-sr),cs=Math.sign(dc-sc);let r=sr+rs,c=sc+cs;while(r!==dr||c!==dc){if(b[r][c])return false;r+=rs;c+=cs}return true}

function hasMove(color){for(let r=0;r<8;r++)for(let c=0;c<8;c++){const p=board[r][c];if(!p)continue;if(color==='white'?p===p.toUpperCase():p===p.toLowerCase())for(let rr=0;rr<8;rr++)for(let cc=0;cc<8;cc++)if(isLegal(r,c,rr,cc))return true}return false}

function evaluate(){if(isCheck(turn,board)){if(!hasMove(turn))endGame(turn==='white'?'Black':'White');else statusEl.textContent=turn+' in Check'}else statusEl.textContent=turn+' to move'}

function endGame(winner){popupText.textContent=winner+' wins by Checkmate!';popup.style.display='flex';try{winSound.play()}catch(e){}confetti({particleCount:200,spread:120,origin:{y:0.6}})}

// ===== MOVE =====
function makeMove(sr,sc,dr,dc){history.push(JSON.stringify({board,turn}));board=JSON.parse(JSON.stringify(board));board[dr][dc]=board[sr][sc];board[sr][sc]='';if(board[dr][dc]==='P'&&dr===0)board[dr][dc]='Q';if(board[dr][dc]==='p'&&dr===7)board[dr][dc]='q';turn=turn==='white'?'black':'white';movesEl.innerHTML+=`<div>${sr},${sc} → ${dr},${dc}</div>`;drawBoard();evaluate();if(vsAI&&turn==='black')setTimeout(aiMove,300)}

function showMoves(r,c){document.querySelectorAll('.square').forEach((s,i)=>{const rr=Math.floor(i/8),cc=i%8;if(isLegal(r,c,rr,cc))s.classList.add('move')})}
function clearMarks(){document.querySelectorAll('.square').forEach(s=>s.classList.remove('highlight','move'))}

// ===== AI =====
function aiMove(){let m=[];for(let r=0;r<8;r++)for(let c=0;c<8;c++)if(board[r][c]&&board[r][c]===board[r][c].toLowerCase())for(let rr=0;rr<8;rr++)for(let cc=0;cc<8;cc++)if(isLegal(r,c,rr,cc))m.push([r,c,rr,cc]);if(m.length)makeMove(...m[Math.floor(Math.random()*m.length)])}

// ===== EVENTS =====
startBtn.onclick=startGame;resetBtn.onclick=resetGame;homeBtn.onclick=goHome;newBtn.onclick=resetGame;undoBtn.onclick=()=>{if(!history.length)return;const h=JSON.parse(history.pop());board=h.board;turn=h.turn;drawBoard();evaluate()};aiBtn.onclick=()=>vsAI=!vsAI;themeSel.onchange=e=>document.body.className=e.target.value;statusEl.ondblclick=()=>{statusEl.style.display='none';setTimeout(()=>statusEl.style.display='block',1200)};
</script>
</body>
</html>
