<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>MarioIA Mobile & PC Game</title>
<style>
/* --- seu CSS existente --- */
body {margin:0; overflow:hidden; background:#000; font-family:Arial, sans-serif;}
#gameCanvas {
    background-color: #666;
    background-image:
        linear-gradient(90deg, transparent 98%, rgba(0,0,0,0.2) 98%),
        linear-gradient(rgba(0,0,0,0.2) 1px, transparent 1px);
    background-size: 50px 50px;
    width:100vw; height:100vh; position:relative;
}
#gameCanvas::before {
    content:''; position:absolute; bottom:0; left:0; right:0;
    height:70px; background:#555; border-top:3px solid #333; z-index:0;
}
.character {position:absolute; z-index:1; box-sizing:border-box; transform-origin:center;}
#luffy {
    left:100px; bottom:60px; width:70px; height:120px;
    background:rgb(255,200,150); border:2px solid black; border-radius:10px;
    position:absolute; overflow:hidden; transition:opacity 0.1s;
}
#luffy::before { content:''; position:absolute; top:0; left:0; right:0; height:35px; background:red;}
#luffy::after { content:''; position:absolute; bottom:0; left:0; right:0; height:55px; background:blue; border-top:5px solid red;}
#akainu { right:100px; bottom:60px; width:80px; height:130px; background:orange; border:2px solid black; border-radius:8px; position:relative; overflow:hidden;}
#akainu::before { content:''; position:absolute; top:0; left:0; right:0; height:130px; background:orange;}
#akainu::after { display:none;}
.lavaHand { position:absolute; width:40px; height:40px; display:flex; justify-content:center; align-items:center; font-size:35px; background:none; border:none; }
.ceilingBlock { position:absolute; width:60px; height:60px; background:#333; border:4px solid #222; top:-100px;}
.warningBlock { position:absolute; width:60px; height:10px; background:red; opacity:0.7; top:0; z-index:2;}
#controls { display:none; z-index:5;}
.controlBtn { position:absolute; border:3px solid black; border-radius:50%; cursor:pointer; display:flex; justify-content:center; align-items:center; padding:0; background:white;}
.arrowBtn { width:70px; height:70px; font-size:30px; bottom:40px;}
#leftBtn { left:30px;} #rightBtn { left:110px;} #jumpBtn { right:140px; bottom:30px; width:80px; height:80px; font-size:18px;}
#healthBars { display:none; z-index:5;}
#timer { position:absolute; top:20px; right:20px; color:white; font-size:32px; font-weight:bold; text-shadow:2px 2px 0px black; display:none; z-index:5;}
#luffyLives { position:absolute; top:20px; left:20px; color:white; font-size:22px; text-shadow:2px 2px 0px black;}
.modalScreen { position:absolute; width:100vw; height:100vh; background:rgba(0,0,0,0.8); color:white; font-size:40px; display:flex; justify-content:center; align-items:center; flex-direction:column; top:0; left:0; z-index:10; text-align:center;}
#endScreen { display:none;}
#startScreen h1 { font-size:80px; color:red; margin-bottom:20px; text-shadow:2px 2px 4px #000;}
.actionBtn { margin-top:20px; padding:15px 30px; font-size:24px; border-radius:20px; border:none; background:white; cursor:pointer;}
#startScreen p { font-size:24px; line-height:1.5; margin:5px 0;}
#exitBtn { background:red; color:white; }
</style>
</head>
<body>
<div id="gameCanvas">

<div id="startScreen" class="modalScreen">
    <h1>MarioIA</h1>
    <p><strong>PC:</strong> ← → para mover, Espaço para pular</p>
    <p><strong>Mobile:</strong> Botões na tela</p>
    <button id="playBtn" class="actionBtn">Play</button>
</div>

<div id="healthBars">
    <div id="luffyLives">Vidas: 3</div>
</div>

<div id="timer">Tempo: 30</div>
<div id="luffy" class="character"></div>
<div id="akainu" class="character"></div>

<div id="controls">
    <button id="leftBtn" class="controlBtn arrowBtn">◀</button>
    <button id="rightBtn" class="controlBtn arrowBtn">▶</button>
    <button id="jumpBtn" class="controlBtn">Jump</button>
</div>

<div id="endScreen" class="modalScreen">
    <div id="endText"></div>
    <button id="replayBtn" class="actionBtn">Jogar novamente</button>
    <button id="exitBtn" class="actionBtn">Sair</button>
</div>

<audio id="menuMusic" loop><source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-5.mp3" type="audio/mpeg"></audio>
<audio id="bgMusic" loop><source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" type="audio/mpeg"></audio>
<audio id="winMusic"><source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3" type="audio/mpeg"></audio>
<audio id="loseMusic"><source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3" type="audio/mpeg"></audio>
<audio id="buttonClick"><source src="https://www.soundjay.com/buttons/sounds/button-16.mp3" type="audio/mpeg"></audio>
<audio id="warningSound"><source src="https://www.soundjay.com/button/sounds/beep-07.mp3" type="audio/mpeg"></audio>

</div>

<script>
let gameCanvas = document.getElementById("gameCanvas");
let luffy=document.getElementById("luffy");
let akainu=document.getElementById("akainu");
let startScreen=document.getElementById("startScreen");
let endScreen=document.getElementById("endScreen");
let controls=document.getElementById("controls");
let healthBars=document.getElementById("healthBars");
let timerDisplay=document.getElementById("timer");
let replayBtn=document.getElementById("replayBtn");
let playBtn=document.getElementById("playBtn");
let jumpBtn=document.getElementById("jumpBtn");
let leftBtn=document.getElementById("leftBtn");
let rightBtn=document.getElementById("rightBtn");
let exitBtn=document.getElementById("exitBtn");

let menuMusic=document.getElementById("menuMusic");
let bgMusic=document.getElementById("bgMusic");
let winMusic=document.getElementById("winMusic");
let loseMusic=document.getElementById("loseMusic");
let buttonClick=document.getElementById("buttonClick");
let warningSound=document.getElementById("warningSound");

let luffyX=100,luffyY=60;
const groundY=60;
let velocityX=0,jumping=false,jumpSpeed=0;
let luffyLives=3,animationFrameId;
let activeTouches={},luffyDirection=1;
let gameTime=30,timerInterval,gameInterval,attackIntervalTime=3500;
let luffyInvincible=false;
let keysPressed={};

// trackers para timeouts/intervals de inimigos
let lavaTimeouts = [], lavaIntervals = [];
let ceilingTimeouts = [], ceilingIntervals = [];

// controle do loop
let running = false;

// Tocar música do menu inicial
menuMusic.play();

playBtn.onclick=()=>{buttonClick.play(); menuMusic.pause(); startGame();}
replayBtn.onclick=()=>{buttonClick.play(); resetGame(); startGame();}
exitBtn.onclick=()=>{buttonClick.play(); window.open('','_self');window.close(); setTimeout(()=>{window.location.href="about:blank";},100);};

function resetGame(){
    // limpar estados
    luffyLives=3; gameTime=30; luffyX=100; luffyY=groundY;
    document.getElementById("luffyLives").innerText="Vidas: "+luffyLives;
    attackIntervalTime=3500;

    // remover todos os objetos gerados no canvas
    document.querySelectorAll('.lavaHand,.ceilingBlock,.warningBlock').forEach(e=>e.remove());

    // parar e resetar audios
    [bgMusic,winMusic,loseMusic].forEach(m=>{ m.pause(); m.currentTime=0;});

    // limpar timers pendentes
    clearAllSpawnTimers();
    // reset movement
    velocityX = 0; jumping = false; jumpSpeed = 0; luffyDirection = 1;
    activeTouches = {}; keysPressed = {};
    luffyInvincible = false;
    luffy.style.opacity = 1;
    luffy.style.left = luffyX + "px";
    luffy.style.bottom = luffyY + "px";
}

function startGame(){
    startScreen.style.display="none"; endScreen.style.display="none";
    controls.style.display="block"; healthBars.style.display="block"; timerDisplay.style.display="block";

    // fullscreen tentativas
    if(gameCanvas.requestFullscreen) gameCanvas.requestFullscreen();
    else if(gameCanvas.webkitRequestFullscreen) gameCanvas.webkitRequestFullscreen();

    bgMusic.play();

    // ligar estado
    running = true;

    gameLoop(); startAttackInterval(); timerInterval=setInterval(updateTimer,1000);

    gameCanvas.addEventListener('touchstart',handleTouch);
    gameCanvas.addEventListener('touchmove',handleTouch);
    gameCanvas.addEventListener('touchend',handleTouchEnd);

    document.addEventListener('keydown',handleKeyDown);
    document.addEventListener('keyup',handleKeyUp);
}

function handleTouch(e){ e.preventDefault(); for(let t of e.changedTouches){ let x=t.clientX,y=t.clientY,id=t.identifier;
    if(!activeTouches[id]){ if(isInside(x,y,jumpBtn)) activeTouches[id]='jump'; else if(isInside(x,y,leftBtn)) activeTouches[id]='left'; else if(isInside(x,y,rightBtn)) activeTouches[id]='right'; } } updateVelocity(); }
function handleTouchEnd(e){ for(let t of e.changedTouches) delete activeTouches[t.identifier]; updateVelocity();}
function isInside(x,y,el){ let r=el.getBoundingClientRect(); return x>=r.left&&x<=r.right&&y>=r.top&&y<=r.bottom;}

function handleKeyDown(e){ keysPressed[e.code]=true; updateVelocity();}
function handleKeyUp(e){ keysPressed[e.code]=false; updateVelocity();}

function updateVelocity(){
    if(!running) { velocityX = 0; return; }
    velocityX=0;luffyDirection=1;
    if(Object.values(activeTouches).includes('left') || keysPressed['ArrowLeft']){ velocityX=-4;luffyDirection=-1;}
    if(Object.values(activeTouches).includes('right') || keysPressed['ArrowRight']){ velocityX=4;luffyDirection=1;}
    if(Object.values(activeTouches).includes('jump') || keysPressed['Space']) doLuffyJump();
}

function gameLoop(){
    // se não está rodando, não continua
    if(!running) return;
    luffyX+=velocityX; if(luffyX<0) luffyX=0;
    const maxX=window.innerWidth-luffy.offsetWidth-akainu.offsetWidth-20; if(luffyX>maxX) luffyX=maxX;
    luffy.style.left=luffyX+"px"; luffy.style.transform=`scaleX(${luffyDirection})`;
    if(jumping){jumpSpeed-=1;luffyY+=jumpSpeed;if(luffyY<=groundY){jumping=false;jumpSpeed=0;luffyY=groundY;}}
    luffy.style.bottom=luffyY+"px";
    animationFrameId=requestAnimationFrame(gameLoop);
}

function doLuffyJump(){if(!jumping&&luffyY===groundY){jumping=true;jumpSpeed=18;}}

function startAttackInterval(){if(gameInterval) clearInterval(gameInterval); gameInterval=setInterval(()=>{
    if(!running) return;
    spawnLavaHand(); spawnCeilingBlock();
}, attackIntervalTime);}

function updateTimer(){ 
    gameTime--; 
    timerDisplay.innerText="Tempo: "+gameTime; 
    if(gameTime<=0){ 
        endGame(true);
    }
}

function spawnLavaHand(){if(!running) return;
    for(let i=0;i<3;i++){ 
        // Guardar o timeout para poder cancelar
        let to = setTimeout(()=>{
            if(!running) return;
            let e=document.createElement("div"); e.classList.add("lavaHand"); e.textContent="💥"; e.style.bottom=(groundY-10)+"px"; gameCanvas.appendChild(e);
            let x=window.innerWidth+50+Math.random()*50;
            // guardar interval
            let interval = setInterval(()=>{
                if(!running){ e.remove(); clearInterval(interval); return; }
                x-=9; e.style.left=x+"px";
                let r1=luffy.getBoundingClientRect(), r2=e.getBoundingClientRect();
                if(r1.left<r2.right&&r1.right>r2.left&&r1.top<r2.bottom&&r1.bottom>r2.top&&!luffyInvincible){ takeDamage(); e.remove(); clearInterval(interval); }
                if(x<-100){ e.remove(); clearInterval(interval); }
            },16);
            lavaIntervals.push(interval);
        }, i*900);
        lavaTimeouts.push(to);
    }
}

function spawnCeilingBlock(){if(!running) return;
    for(let i=0;i<3;i++){ 
        let to = setTimeout(()=>{
            if(!running) return;
            let posX=Math.random()*(window.innerWidth-60);
            let warning=document.createElement("div"); warning.classList.add("warningBlock"); warning.style.left=posX+"px"; gameCanvas.appendChild(warning);
            warningSound.play();
            let innerTo = setTimeout(()=>{
                if(!running){ warning.remove(); return; }
                warning.remove();
                let b=document.createElement("div"); b.classList.add("ceilingBlock"); b.style.left=posX+"px"; gameCanvas.appendChild(b);
                let y=-100,gravity=0.5,speedY=0,floor=window.innerHeight-groundY-60;
                let interval=setInterval(()=>{
                    if(!running){ b.remove(); clearInterval(interval); return; }
                    speedY+=gravity; y+=speedY; b.style.top=y+"px";
                    let r1=luffy.getBoundingClientRect(), r2=b.getBoundingClientRect();
                    if(r1.left<r2.right&&r1.right>r2.left&&r1.top<r2.bottom&&r1.bottom>r2.top&&!luffyInvincible){ takeDamage(); b.remove(); clearInterval(interval);}
                    if(y>floor+100){ b.remove(); clearInterval(interval);}
                },16);
                ceilingIntervals.push(interval);
            }, 1000);
            ceilingTimeouts.push(innerTo);
        }, i*1300);
        ceilingTimeouts.push(to);
    }
}

function takeDamage(){
    if(luffyInvincible) return;
    luffyLives--;
    document.getElementById("luffyLives").innerText="Vidas: "+luffyLives;
    if(luffyLives<=0){ endGame(false); return; }
    luffyInvincible=true; luffy.style.opacity=0.4;
    setTimeout(()=>{ luffyInvincible=false; luffy.style.opacity=1;},2000);
}

function clearAllSpawnTimers(){
    // limpar timeouts
    lavaTimeouts.forEach(id=>clearTimeout(id));
    lavaTimeouts = [];
    ceilingTimeouts.forEach(id=>clearTimeout(id));
    ceilingTimeouts = [];
    // limpar intervals
    lavaIntervals.forEach(id=>clearInterval(id));
    lavaIntervals = [];
    ceilingIntervals.forEach(id=>clearInterval(id));
    ceilingIntervals = [];
    // limpar o intervalo principal de ataques
    if(gameInterval){ clearInterval(gameInterval); gameInterval = null; }
}

function endGame(win){
    // desligar estado de jogo
    running = false;

    // limpar timers e animações
    clearAllSpawnTimers();
    clearInterval(timerInterval); timerInterval = null;
    if(animationFrameId) cancelAnimationFrame(animationFrameId);

    // remover listeners
    gameCanvas.removeEventListener('touchstart',handleTouch);
    gameCanvas.removeEventListener('touchmove',handleTouch);
    gameCanvas.removeEventListener('touchend',handleTouchEnd);
    document.removeEventListener('keydown',handleKeyDown);
    document.removeEventListener('keyup',handleKeyUp);

    // reset input/movimento imediato para garantir que não se mova
    velocityX = 0;
    jumping = false;
    jumpSpeed = 0;
    activeTouches = {};
    keysPressed = {};
    luffyY = groundY;
    luffy.style.bottom = luffyY + "px";

    controls.style.display="none"; healthBars.style.display="none"; timerDisplay.style.display="none";
    bgMusic.pause(); bgMusic.currentTime=0;
    if(win){ winMusic.play(); document.getElementById("endText").innerText="Você salvou a princesa!";}
    else{ loseMusic.play(); loseMusic.currentTime=0; loseMusic.play(); document.getElementById("endText").innerText="O Bowser casou com a princesa!";}
    endScreen.style.display="flex";
}
</script>
</body>
</html>.controlBtn { position:absolute; border:3px solid black; border-radius:50%; cursor:pointer; display:flex; justify-content:center; align-items:center; padding:0; background:white;}
.arrowBtn { width:70px; height:70px; font-size:30px; bottom:40px;}
#leftBtn { left:30px;} #rightBtn { left:110px;} #jumpBtn { right:140px; bottom:30px; width:80px; height:80px; font-size:18px;}
#healthBars { display:none; z-index:5;}
#timer { position:absolute; top:20px; right:20px; color:white; font-size:32px; font-weight:bold; text-shadow:2px 2px 0px black; display:none; z-index:5;}
#luffyLives { position:absolute; top:20px; left:20px; color:white; font-size:22px; text-shadow:2px 2px 0px black;}
.modalScreen { position:absolute; width:100vw; height:100vh; background:rgba(0,0,0,0.8); color:white; font-size:40px; display:flex; justify-content:center; align-items:center; flex-direction:column; top:0; left:0; z-index:10; text-align:center;}
#endScreen { display:none;}
#startScreen h1 { font-size:80px; color:red; margin-bottom:20px; text-shadow:2px 2px 4px #000;}
.actionBtn { margin-top:20px; padding:15px 30px; font-size:24px; border-radius:20px; border:none; background:white; cursor:pointer;}
#startScreen p { font-size:24px; line-height:1.5; margin:5px 0;}
#exitBtn { background:red; color:white; }
</style>
</head>
<body>
<div id="gameCanvas">

<div id="startScreen" class="modalScreen">
    <h1>MarioIA</h1>
    <p><strong>PC:</strong> ← → para mover, Espaço para pular</p>
    <p><strong>Mobile:</strong> Botões na tela</p>
    <button id="playBtn" class="actionBtn">Play</button>
</div>

<div id="healthBars">
    <div id="luffyLives">Vidas: 3</div>
</div>

<div id="timer">Tempo: 30</div>
<div id="luffy" class="character"></div>
<div id="akainu" class="character"></div>

<div id="controls">
    <button id="leftBtn" class="controlBtn arrowBtn">◀</button>
    <button id="rightBtn" class="controlBtn arrowBtn">▶</button>
    <button id="jumpBtn" class="controlBtn">Jump</button>
</div>

<div id="endScreen" class="modalScreen">
    <div id="endText"></div>
    <button id="replayBtn" class="actionBtn">Jogar novamente</button>
    <button id="exitBtn" class="actionBtn">Sair</button>
</div>

<audio id="menuMusic" loop><source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-5.mp3" type="audio/mpeg"></audio>
<audio id="bgMusic" loop><source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" type="audio/mpeg"></audio>
<audio id="winMusic"><source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3" type="audio/mpeg"></audio>
<audio id="loseMusic"><source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3" type="audio/mpeg"></audio>
<audio id="buttonClick"><source src="https://www.soundjay.com/buttons/sounds/button-16.mp3" type="audio/mpeg"></audio>
<audio id="warningSound"><source src="https://www.soundjay.com/button/sounds/beep-07.mp3" type="audio/mpeg"></audio>

</div>

<script>
let luffy=document.getElementById("luffy");
let akainu=document.getElementById("akainu");
let startScreen=document.getElementById("startScreen");
let endScreen=document.getElementById("endScreen");
let controls=document.getElementById("controls");
let healthBars=document.getElementById("healthBars");
let timerDisplay=document.getElementById("timer");
let replayBtn=document.getElementById("replayBtn");
let playBtn=document.getElementById("playBtn");
let jumpBtn=document.getElementById("jumpBtn");
let leftBtn=document.getElementById("leftBtn");
let rightBtn=document.getElementById("rightBtn");
let exitBtn=document.getElementById("exitBtn");

let menuMusic=document.getElementById("menuMusic");
let bgMusic=document.getElementById("bgMusic");
let winMusic=document.getElementById("winMusic");
let loseMusic=document.getElementById("loseMusic");
let buttonClick=document.getElementById("buttonClick");
let warningSound=document.getElementById("warningSound");

let luffyX=100,luffyY=60;
const groundY=60;
let velocityX=0,jumping=false,jumpSpeed=0;
let luffyLives=3,animationFrameId;
let activeTouches={},luffyDirection=1;
let gameTime=30,timerInterval,gameInterval,attackIntervalTime=3500;
let luffyInvincible=false;
let keysPressed={};

// Tocar música do menu inicial
menuMusic.play();

playBtn.onclick=()=>{buttonClick.play(); menuMusic.pause(); startGame();}
replayBtn.onclick=()=>{buttonClick.play(); resetGame(); startGame();}
exitBtn.onclick=()=>{buttonClick.play(); window.open('','_self');window.close(); setTimeout(()=>{window.location.href="about:blank";},100);};

function resetGame(){
    luffyLives=3; gameTime=30; luffyX=100; luffyY=60;
    document.getElementById("luffyLives").innerText="Vidas: "+luffyLives;
    attackIntervalTime=3500;
    document.querySelectorAll('.lavaHand,.ceilingBlock,.warningBlock').forEach(e=>e.remove());
    [bgMusic,winMusic,loseMusic].forEach(m=>{ m.pause(); m.currentTime=0;});
}

function startGame(){
    startScreen.style.display="none"; endScreen.style.display="none";
    controls.style.display="block"; healthBars.style.display="block"; timerDisplay.style.display="block";

    if(gameCanvas.requestFullscreen) gameCanvas.requestFullscreen();
    else if(gameCanvas.webkitRequestFullscreen) gameCanvas.webkitRequestFullscreen();

    bgMusic.play();

    gameLoop(); startAttackInterval(); timerInterval=setInterval(updateTimer,1000);

    gameCanvas.addEventListener('touchstart',handleTouch);
    gameCanvas.addEventListener('touchmove',handleTouch);
    gameCanvas.addEventListener('touchend',handleTouchEnd);

    document.addEventListener('keydown',handleKeyDown);
    document.addEventListener('keyup',handleKeyUp);
}

function handleTouch(e){ e.preventDefault(); for(let t of e.changedTouches){ let x=t.clientX,y=t.clientY,id=t.identifier;
    if(!activeTouches[id]){ if(isInside(x,y,jumpBtn)) activeTouches[id]='jump'; else if(isInside(x,y,leftBtn)) activeTouches[id]='left'; else if(isInside(x,y,rightBtn)) activeTouches[id]='right'; } } updateVelocity(); }
function handleTouchEnd(e){ for(let t of e.changedTouches) delete activeTouches[t.identifier]; updateVelocity();}
function isInside(x,y,el){ let r=el.getBoundingClientRect(); return x>=r.left&&x<=r.right&&y>=r.top&&y<=r.bottom;}

function handleKeyDown(e){ keysPressed[e.code]=true; updateVelocity();}
function handleKeyUp(e){ keysPressed[e.code]=false; updateVelocity();}

function updateVelocity(){
    velocityX=0;luffyDirection=1;
    if(Object.values(activeTouches).includes('left') || keysPressed['ArrowLeft']){ velocityX=-4;luffyDirection=-1;}
    if(Object.values(activeTouches).includes('right') || keysPressed['ArrowRight']){ velocityX=4;luffyDirection=1;}
    if(Object.values(activeTouches).includes('jump') || keysPressed['Space']) doLuffyJump();
}

function gameLoop(){
    luffyX+=velocityX; if(luffyX<0) luffyX=0;
    const maxX=window.innerWidth-luffy.offsetWidth-akainu.offsetWidth-20; if(luffyX>maxX) luffyX=maxX;
    luffy.style.left=luffyX+"px"; luffy.style.transform=`scaleX(${luffyDirection})`;
    if(jumping){jumpSpeed-=1;luffyY+=jumpSpeed;if(luffyY<=groundY){jumping=false;jumpSpeed=0;luffyY=groundY;}}
    luffy.style.bottom=luffyY+"px";
    if(endScreen.style.display!=="flex") animationFrameId=requestAnimationFrame(gameLoop);
}

function doLuffyJump(){if(!jumping&&luffyY===groundY){jumping=true;jumpSpeed=18;}}

function startAttackInterval(){if(gameInterval) clearInterval(gameInterval); gameInterval=setInterval(()=>{
    spawnLavaHand(); spawnCeilingBlock();
}, attackIntervalTime);}

function updateTimer(){ 
    gameTime--; 
    timerDisplay.innerText="Tempo: "+gameTime; 
    if(gameTime<=0){ 
        endGame(true);
    }
}

function spawnLavaHand(){if(endScreen.style.display==="flex") return;
    for(let i=0;i<3;i++){ setTimeout(()=>{
        let e=document.createElement("div"); e.classList.add("lavaHand"); e.textContent="💥"; e.style.bottom=(groundY-10)+"px"; gameCanvas.appendChild(e);
        let x=window.innerWidth+50+Math.random()*50;
        let interval=setInterval(()=>{
            x-=9; e.style.left=x+"px";
            let r1=luffy.getBoundingClientRect(), r2=e.getBoundingClientRect();
            if(r1.left<r2.right&&r1.right>r2.left&&r1.top<r2.bottom&&r1.bottom>r2.top&&!luffyInvincible) takeDamage();
            if(x<-100){ e.remove(); clearInterval(interval); }
        },16);
    }, i*900);}
}

function spawnCeilingBlock(){if(endScreen.style.display==="flex") return;
    for(let i=0;i<3;i++){ 
        setTimeout(()=>{
            let posX=Math.random()*(window.innerWidth-60);
            let warning=document.createElement("div"); warning.classList.add("warningBlock"); warning.style.left=posX+"px"; gameCanvas.appendChild(warning);
            warningSound.play();
            setTimeout(()=>{
                warning.remove();
                let b=document.createElement("div"); b.classList.add("ceilingBlock"); b.style.left=posX+"px"; gameCanvas.appendChild(b);
                let y=-100,gravity=0.5,speedY=0,floor=window.innerHeight-groundY-60;
                let interval=setInterval(()=>{
                    speedY+=gravity; y+=speedY; b.style.top=y+"px";
                    let r1=luffy.getBoundingClientRect(), r2=b.getBoundingClientRect();
                    if(r1.left<r2.right&&r1.right>r2.left&&r1.top<r2.bottom&&r1.bottom>r2.top&&!luffyInvincible){ takeDamage(); b.remove(); clearInterval(interval);}
                    if(y>floor+100){ b.remove(); clearInterval(interval);}
                },16);
            }, 1000);
        }, i*1300);
    }
}

function takeDamage(){
    luffyLives--;
    document.getElementById("luffyLives").innerText="Vidas: "+luffyLives;
    if(luffyLives<=0){ endGame(false); return; }
    luffyInvincible=true; luffy.style.opacity=0.4;
    setTimeout(()=>{ luffyInvincible=false; luffy.style.opacity=1;},2000);
}

function endGame(win){
    clearInterval(gameInterval); clearInterval(timerInterval); cancelAnimationFrame(animationFrameId);
    gameCanvas.removeEventListener('touchstart',handleTouch);
    gameCanvas.removeEventListener('touchmove',handleTouch);
    gameCanvas.removeEventListener('touchend',handleTouchEnd);
    document.removeEventListener('keydown',handleKeyDown);
    document.removeEventListener('keyup',handleKeyUp);
    controls.style.display="none"; healthBars.style.display="none"; timerDisplay.style.display="none";
    bgMusic.pause(); bgMusic.currentTime=0;
    if(win){ winMusic.play(); document.getElementById("endText").innerText="Você salvou a princesa!";}
    else{ loseMusic.play(); loseMusic.currentTime=0; loseMusic.play(); document.getElementById("endText").innerText="O Bowser casou com a princesa!";}
    endScreen.style.display="flex";
}
</script>
</body>
</html>
