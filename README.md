<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>초록도시 II FCEV</title>
<style>
:root{color-scheme:dark;--bg:#0b1119;--panel:#141f2d;--line:#2b3d50;--text:#eef6ff;--muted:#94a9bf;--green:#49e0a0;--blue:#258dff;--red:#ff6576}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font:14px/1.5 system-ui,sans-serif}
header{padding:18px;background:#111b28;border-bottom:1px solid var(--line);display:flex;justify-content:space-between;align-items:center;gap:12px;flex-wrap:wrap}
h1{font-size:21px;margin:0}h2{font-size:15px;margin:0 0 12px}
small,.muted{color:var(--muted)}
button{background:#223348;color:var(--text);border:1px solid #3a5068;border-radius:8px;padding:9px 11px;font:inherit;cursor:pointer}
button:hover{border-color:var(--green)}button.active{background:#145c48;border-color:var(--green)}button:disabled{opacity:.45;cursor:not-allowed}
main{max-width:1450px;margin:auto;padding:14px;display:grid;gap:14px}
.grid{display:grid;grid-template-columns:minmax(0,1.7fr) minmax(300px,1fr);gap:14px}
.panel{background:var(--panel);border:1px solid var(--line);border-radius:13px;padding:14px;min-width:0}
.hud{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:9px;margin-bottom:12px}
.metric{background:#0b1520;border:1px solid #24374a;border-radius:9px;padding:10px}
.metric .value{font-size:23px;font-weight:750;font-variant-numeric:tabular-nums}
.metric small{display:block}
canvas{display:block;width:100%;border-radius:8px;background:#101b26}
#road{aspect-ratio:16/7}#map{height:140px}
.mirrors{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:10px}
.mirrors canvas{height:78px}
.section{margin-top:14px;padding-top:13px;border-top:1px solid var(--line)}
.row{display:flex;flex-wrap:wrap;gap:7px;align-items:center}
.between{display:flex;justify-content:space-between;align-items:center;gap:8px}
.status{padding:7px 10px;border-radius:7px;background:#0b1520;margin:8px 0}
.good{color:var(--green)}.bad{color:var(--red)}
.bar{height:9px;background:#263548;border-radius:20px;overflow:hidden;margin:7px 0 12px}
.fill{height:100%;width:100%;background:var(--green)}.fill.blue{background:var(--blue)}
.gears{display:grid;grid-template-columns:repeat(5,1fr);gap:5px}.gears button{padding:8px 3px}
.controls{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:7px}.controls button{min-height:44px}
#log{height:160px;overflow:auto;font-size:12px;color:#bdcddd}
#log div{padding:4px 0;border-bottom:1px solid #233243}
kbd{border:1px solid #51647a;border-radius:4px;padding:1px 5px;background:#101923}
input[type=range]{width:100%;accent-color:var(--green)}input[type=checkbox]{accent-color:var(--green)}
footer{text-align:center;color:var(--muted);padding:15px}
@media(max-width:850px){.grid{grid-template-columns:1fr}.hud{grid-template-columns:repeat(2,minmax(0,1fr))}}
</style>
</head>
<body>
<header>
  <div>
    <h1>🚌 초록도시 II FCEV</h1>
    <small>경부고속도로 기반 · 서울반포 → 부산센텀 · 27석 수소전기버스</small>
  </div>
  <div class="row">
    <span id="modeBadge" class="good">수동변속</span>
    <button id="modeBtn">자동변속으로 전환</button>
    <button id="pauseBtn">일시정지</button>
    <button id="resetBtn">초기화</button>
  </div>
</header>

<main>
<section class="hud">
  <div class="metric"><small>속도</small><div class="value"><span id="speed">0</span> <small>km/h</small></div></div>
  <div class="metric"><small>주행 거리</small><div class="value"><span id="distance">0.0</span> <small>km</small></div></div>
  <div class="metric"><small>기어</small><div class="value" id="gearDisplay">1</div></div>
  <div class="metric"><small>승객</small><div class="value"><span id="passengers">0</span><small> / 27명</small></div></div>
</section>

<div class="grid">
<div>
<section class="panel">
  <div class="between"><h2>전방 도로 · 경부고속도로</h2><span id="roadStatus" class="good">서울반포</span></div>
  <canvas id="road" width="960" height="420"></canvas>
  <div class="mirrors">
    <div><small>좌측 사이드미러</small><canvas id="mirrorL" width="400" height="90"></canvas></div>
    <div><small>우측 사이드미러</small><canvas id="mirrorR" width="400" height="90"></canvas></div>
  </div>
  <div class="section">
    <div class="between"><h2>노선 배선도</h2><small>경부고속도로 노선</small></div>
    <canvas id="map" width="960" height="140"></canvas>
    <small id="nextStopText">다음 주요 지점: 판교</small>
  </div>
</section>

<section class="panel">
  <h2>운전 조작</h2>
  <div class="controls">
    <button id="accelerate">▲ 가속</button>
    <button id="brake">▼ 브레이크</button>
    <button id="horn">경적 B</button>
    <button id="left">◀ 좌측 변경</button>
    <button id="center">핸들 중앙</button>
    <button id="right">우측 변경 ▶</button>
    <button id="sigL">좌측 방향지시등 A</button>
    <button id="hazard">비상등</button>
    <button id="sigR">우측 방향지시등 F</button>
  </div>
  <div class="section">
    <div class="between"><b>핸들</b><span id="steerDisplay">0°</span></div>
    <div class="bar"><div id="steerBar" class="fill blue"></div></div>
    <div class="between"><b>현재 차로</b><span id="laneDisplay">3차로</span></div>
    <small id="laneDescription">일반 주행 차로</small>
    <p><small>방향키를 계속 누르면 차로를 변경합니다.</small></p>
  </div>
</section>
</div>

<div>
<section class="panel">
  <h2>운전 계기판</h2>
  <div class="status">운행 상태: <b id="state">정차</b></div>
  <div class="status">문 상태: <b id="doorState">모든 문 닫힘</b></div>
  <div class="status">클러치: <b id="clutchState">해제</b> · 방향지시등: <b id="signalState">꺼짐</b></div>
  <div class="status">
    수소: <b id="h2Text">33.00 kg</b>
    <div class="bar"><div id="h2Bar" class="fill"></div></div>
    배터리: <b id="batteryText">47.0 kWh</b>
    <div class="bar"><div id="batteryBar" class="fill blue"></div></div>
  </div>

  <div class="section">
    <h2>변속기 · 수동 기본</h2>
    <div class="gears" id="gearButtons"></div>
    <div class="row" style="margin-top:8px">
      <button id="clutchBtn">클러치 Shift</button>
      <span id="gearHelp" class="muted">수동변속 모드</span>
    </div>
    <small>수동변속 시 클러치를 밟고 변속하세요.</small>
  </div>

  <div class="section">
    <h2>출입문 및 도어 부저</h2>
    <div class="row">
      <button id="openL">좌측 문 열기 (W+O)</button>
      <button id="closeL">좌측 문 닫기 (W+P)</button>
    </div>
    <div class="row" style="margin-top:7px">
      <button id="openR">우측 문 열기 (E+O)</button>
      <button id="closeR">우측 문 닫기 (E+P)</button>
    </div>

    <div class="status">도어 부저: <b id="buzzerStatus">대기</b></div>

    <div class="section">
      <div class="between">
        <label for="openFreq">문 열림 부저 주파수</label>
        <b id="openFreqText">500 Hz</b>
      </div>
      <input id="openFreq" type="range" min="500" max="580" step="10" value="500">

      <div class="between">
        <label for="closeFreq">문 닫힘 부저 주파수</label>
        <b id="closeFreqText">580 Hz</b>
      </div>
      <input id="closeFreq" type="range" min="500" max="580" step="10" value="580">

      <div class="between">
        <label for="buzzerVolume">부저 음량</label>
        <b id="buzzerVolumeText">50%</b>
      </div>
      <input id="buzzerVolume" type="range" min="0" max="100" step="1" value="50">

      <div class="row">
        <button id="testOpen">열림 부저 시험</button>
        <button id="testClose">닫힘 부저 시험</button>
      </div>
      <small>문 열림과 문 닫힘의 주파수 및 음량을 각각 조절할 수 있습니다. 브라우저 오디오 기능을 사용합니다.</small>
    </div>
    <small>정차 상태에서만 문을 열 수 있습니다. 문을 닫을 때에도 부저가 울립니다.</small>
  </div>

  <div class="section">
    <h2>정류장 · 휴게소 업무</h2>
    <div class="row">
      <button id="boardBtn">승객 승하차</button>
      <button id="departBtn">출발 준비</button>
    </div>
    <div class="row" style="margin-top:7px">
      <button id="fuelBtn">수소 충전</button>
      <button id="chargeBtn">배터리 충전</button>
    </div>
    <div class="status">현재 지점: <b id="currentStop">서울반포</b></div>
    <div class="status">정차 여부: <b id="stopStatus">정차 지점 아님</b></div>
    <small id="stopInfo">서울반포: 승차 24명 예정</small>

    <div class="section">
      <h2>필요 시 정차</h2>
      <div class="row">
        <button class="restStop" data-stop="판교휴게소">판교휴게소</button>
        <button class="restStop" data-stop="천안호두휴게소">천안호두휴게소</button>
        <button class="restStop" data-stop="김천휴게소">김천휴게소</button>
        <button class="restStop" data-stop="경주휴게소">경주휴게소</button>
      </div>
      <small>휴게소 버튼을 누르면 해당 지점이 정차 목표가 됩니다. 운전자가 정차를 선택할 수 있습니다.</small>
    </div>
  </div>
</section>

<section class="panel">
  <h2>냉난방</h2>
  <div class="row">
    <label><input type="checkbox" id="ac"> 에어컨</label>
    <label><input type="checkbox" id="heater"> 난방</label>
  </div>
  <div class="between"><label for="fan">송풍 단계</label><b id="fanText">3단</b></div>
  <input id="fan" type="range" min="1" max="5" value="3">
</section>

<section class="panel">
  <h2>정속 주행</h2>
  <div class="row">
    <button class="cruise" data-speed="85">85</button>
    <button class="cruise" data-speed="90">90</button>
    <button class="cruise" data-speed="95">95</button>
    <button class="cruise" data-speed="100">100</button>
    <button class="cruise" data-speed="105">105</button>
    <button class="cruise" data-speed="110">110</button>
    <button id="cruiseOff">해제</button>
  </div>
  <div class="status">설정 속도: <b id="cruiseText">해제</b></div>
</section>

<section class="panel">
  <h2>운행 정산</h2>
  <div class="between"><span>운임 수입</span><b id="revenue">₩0</b></div>
  <div class="between"><span>수소 충전 비용</span><b id="fuelCost">₩0</b></div>
  <div class="between"><span>전기 충전 비용</span><b id="electricCost">₩0</b></div>
  <div class="between"><b>순이익</b><b id="net" class="good">₩0</b></div>
  <small>운임 2,000원/km · 수소 12,000원/kg · 전기 190원/kWh</small>
</section>

<section class="panel">
  <div class="between"><h2>운행 기록</h2><button id="clearLog">기록 지우기</button></div>
  <div id="log"></div>
</section>
</div>
</div>

<section class="panel">
  <h2>키보드 안내</h2>
  <div class="row">
    <span><kbd>↑</kbd> 가속</span><span><kbd>↓</kbd> 제동</span>
    <span><kbd>←</kbd><kbd>→</kbd> 차로 변경</span>
    <span><kbd>A</kbd> 좌측 깜빡이</span><span><kbd>F</kbd> 우측 깜빡이</span>
    <span><kbd>B</kbd> 경적</span><span><kbd>Shift</kbd> 클러치</span>
  </div>
  <div class="row" style="margin-top:8px">
    <span><kbd>5</kbd> 1단</span><span><kbd>6</kbd> 2단</span>
    <span><kbd>7</kbd> 3단</span><span><kbd>8</kbd> 4단</span>
    <span><kbd>9</kbd> 5단</span><span><kbd>4</kbd> N</span>
    <span><kbd>3</kbd> R1</span><span><kbd>2</kbd> R2</span><span><kbd>1</kbd> R3</span>
  </div>
  <div class="row" style="margin-top:8px">
    <span><kbd>W</kbd>+<kbd>O</kbd> 좌측 문 열기</span>
    <span><kbd>W</kbd>+<kbd>P</kbd> 좌측 문 닫기</span>
    <span><kbd>E</kbd>+<kbd>O</kbd> 우측 문 열기</span>
    <span><kbd>E</kbd>+<kbd>P</kbd> 우측 문 닫기</span>
  </div>
</section>
</main>

<footer>초록도시 II FCEV · 경부고속도로 · 27석 · 도어 부저 주파수 조절</footer>

<script>
"use strict";

const $=id=>document.getElementById(id);
const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
const fmt=n=>Math.round(n).toLocaleString("ko-KR");
const CAPACITY=27;
const pressed=new Set();

const GEAR_MAP={
  Digit5:"1",Numpad5:"1",
  Digit6:"2",Numpad6:"2",
  Digit7:"3",Numpad7:"3",
  Digit8:"4",Numpad8:"4",
  Digit9:"5",Numpad9:"5",
  Digit4:"N",Numpad4:"N",
  Digit3:"R1",Numpad3:"R1",
  Digit2:"R2",Numpad2:"R2",
  Digit1:"R3",Numpad1:"R3"
};

const FCAP={"1":25,"2":45,"3":70,"4":105,"5":197};
const RCAP={R1:10,R2:20,R3:30};

/*
  경부고속도로의 실제 주행 순서를 바탕으로 만든
  시뮬레이션용 거리 좌표입니다.
  실제 도로의 실측 거리와 정확히 일치하지 않을 수 있습니다.
*/
const ROUTE=[
  {name:"서울반포",km:0,type:"terminal",board:24,alight:0},
  {name:"판교",km:25,type:"station",board:0,alight:0},
  {name:"판교휴게소",km:35,type:"rest",board:0,alight:0},
  {name:"죽전",km:42,type:"station",board:0,alight:0},
  {name:"동탄",km:57,type:"station",board:0,alight:0},
  {name:"평택지제",km:75,type:"station",board:0,alight:0},
  {name:"천안호두휴게소",km:105,type:"rest",board:0,alight:0},
  {name:"천안아산",km:120,type:"station",board:0,alight:0},
  {name:"오송",km:140,type:"station",board:0,alight:0},
  {name:"신탄진휴게소",km:155,type:"rest",board:0,alight:0},
  {name:"북대전분기점",km:165,type:"waypoint",board:0,alight:0},
  {name:"대전",km:175,type:"terminal",board:12,alight:14},
  {name:"옥천",km:195,type:"station",board:0,alight:0},
  {name:"옥천휴게소",km:205,type:"rest",board:0,alight:0},
  {name:"영동",km:225,type:"station",board:0,alight:0},
  {name:"영동휴게소",km:240,type:"rest",board:0,alight:0},
  {name:"김천",km:270,type:"station",board:0,alight:0},
  {name:"김천휴게소",km:285,type:"rest",board:0,alight:0},
  {name:"동김천",km:300,type:"station",board:0,alight:0},
  {name:"구미",km:320,type:"station",board:0,alight:0},
  {name:"왜관",km:340,type:"station",board:0,alight:0},
  {name:"대구",km:360,type:"station",board:0,alight:0},
  {name:"동대구",km:370,type:"station",board:0,alight:0},
  {name:"경산",km:382,type:"station",board:0,alight:0},
  {name:"영천휴게소",km:395,type:"rest",board:0,alight:0},
  {name:"경주",km:420,type:"station",board:0,alight:0},
  {name:"경주휴게소",km:432,type:"rest",board:0,alight:0},
  {name:"북울산",km:450,type:"station",board:0,alight:0},
  {name:"울산시내",km:470,type:"station",board:0,alight:0},
  {name:"부산센텀",km:500,type:"terminal",board:0,alight:27}
];

const OPTIONAL_REST=new Set([
  "판교휴게소","천안호두휴게소","김천휴게소","경주휴게소"
]);

let s;
let steerCooldown=0;
let last=performance.now();
let audioCtx=null;
let buzzerSequence=0;

function freshState(){
  return {
    speed:0,
    distance:0,
    lane:3,
    steer:0,
    mode:"manual",
    gear:"D1",
    manualGear:"1",
    clutch:false,
    leftDoor:false,
    rightDoor:false,
    doorBusy:false,
    signal:"off",
    hazard:false,
    passengers:0,
    h2:33,
    battery:47,
    revenue:0,
    fuelCost:0,
    electricCost:0,
    ac:false,
    heater:false,
    fan:3,
    cruise:null,
    paused:false,
    finished:false,
    routeIndex:0,
    stopTarget:1,
    log:[],
    doorTimer:null,
    boardDone:false,
    lastDirection:"forward",
    stopRequested:false
  };
}

s=freshState();

function log(message){
  s.log.unshift(
    "["+new Date().toLocaleTimeString("ko-KR",{hour12:false})+"] "+message
  );
  s.log=s.log.slice(0,100);
  $("log").innerHTML=s.log.map(v=>
    "<div>"+v.replace(/[&<>]/g,c=>({
      "&":"&amp;","<":"&lt;",">":"&gt;"
    }[c]))+"</div>"
  ).join("");
}

function notify(message){
  $("roadStatus").textContent=message;
  log(message);
}

function currentRoute(){
  return ROUTE[s.routeIndex];
}

function targetRoute(){
  return ROUTE[Math.min(ROUTE.length-1,s.stopTarget)];
}

function atStop(){
  return Math.abs(s.distance-currentRoute().km)<.12&&s.speed<.1;
}

function doorsClosed(){
  return !s.leftDoor&&!s.rightDoor&&!s.doorBusy;
}

function gear(){
  return s.mode==="auto"?s.gear:s.manualGear;
}

function capSpeed(){
  const g=gear();
  if(g==="N")return 0;
  if(g.startsWith("R"))return RCAP[g]||10;
  return s.mode==="auto"?197:(FCAP[g]||25);
}

function canDrive(){
  return !s.paused&&
    !s.finished&&
    doorsClosed()&&
    s.h2>.001&&
    s.battery>0&&
    !(s.mode==="manual"&&s.clutch)&&
    gear()!=="N"&&
    !gear().startsWith("R");
}

function getAudio(){
  const AudioContextClass=window.AudioContext||window.webkitAudioContext;

  if(!AudioContextClass){
    $("buzzerStatus").textContent="오디오 미지원 브라우저";
    return null;
  }

  try{
    if(!audioCtx)audioCtx=new AudioContextClass();

    if(audioCtx.state==="suspended"){
      audioCtx.resume().catch(()=>{});
    }

    return audioCtx;
  }catch(error){
    $("buzzerStatus").textContent="오디오 초기화 실패";
    return null;
  }
}

/*
  도어 부저:
  - 문 열림과 문 닫힘에 서로 다른 주파수 사용
  - 음량 슬라이더로 음량 조절
  - 각 문 조작 시작 시 부저 재생
*/
function playDoorBuzzer(kind){
  const ctx=getAudio();
  if(!ctx)return;

  const freq=Number($(kind==="open"?"openFreq":"closeFreq").value);
  const volume=Number($("buzzerVolume").value)/100;
  const thisSequence=++buzzerSequence;

  $("buzzerStatus").textContent=
    kind==="open"?"열림 부저 재생 ("+freq+" Hz)":
    "닫힘 부저 재생 ("+freq+" Hz)";

  if(volume<=0){
    $("buzzerStatus").textContent="음량 0% — 무음";
    return;
  }

  const osc=ctx.createOscillator();
  const gain=ctx.createGain();
  const now=ctx.currentTime;
  const duration=kind==="open"?.32:.42;

  osc.type="sine";
  osc.frequency.setValueAtTime(freq,now);

  gain.gain.setValueAtTime(.0001,now);
  gain.gain.linearRampToValueAtTime(volume*.32,now+.025);
  gain.gain.setValueAtTime(volume*.32,now+duration-.06);
  gain.gain.linearRampToValueAtTime(.0001,now+duration);

  osc.connect(gain);
  gain.connect(ctx.destination);

  osc.onended=()=>{
    osc.disconnect();
    gain.disconnect();

    if(thisSequence===buzzerSequence){
      $("buzzerStatus").textContent="대기";
    }
  };

  osc.start(now);
  osc.stop(now+duration+.02);
}

function playHorn(){
  const ctx=getAudio();
  if(!ctx)return;

  const now=ctx.currentTime;

  [220,277].forEach(freq=>{
    const osc=ctx.createOscillator();
    const gain=ctx.createGain();

    osc.type="sawtooth";
    osc.frequency.value=freq;

    gain.gain.setValueAtTime(.0001,now);
    gain.gain.linearRampToValueAtTime(.12,now+.04);
    gain.gain.setValueAtTime(.12,now+.32);
    gain.gain.linearRampToValueAtTime(.0001,now+.4);

    osc.connect(gain);
    gain.connect(ctx.destination);

    osc.onended=()=>{
      osc.disconnect();
      gain.disconnect();
    };

    osc.start(now);
    osc.stop(now+.42);
  });
}

function shiftGear(g){
  if(s.mode!=="manual"){
    notify("자동변속 모드입니다. 수동변속으로 전환하세요.");
    return;
  }

  if(!s.clutch){
    notify("변속하려면 Shift 클러치를 밟으세요.");
    return;
  }

  const old=s.manualGear;
  const oldDir=old==="N"?s.lastDirection:
    old.startsWith("R")?"reverse":"forward";
  const newDir=g==="N"?null:
    g.startsWith("R")?"reverse":"forward";

  if(s.speed>.1&&newDir!==null&&newDir!==oldDir){
    notify("전진과 후진 변속은 완전히 정차한 뒤 가능합니다.");
    return;
  }

  s.manualGear=g;
  if(newDir)s.lastDirection=newDir;
  if(g==="N"||g.startsWith("R"))s.cruise=null;

  notify(old+" → "+g+" 변속 완료");
  render();
}

function toggleMode(){
  if(s.speed>.1||!doorsClosed()){
    notify("모드 변경은 완전 정차 및 문 닫힘 상태에서 가능합니다.");
    return;
  }

  s.mode=s.mode==="auto"?"manual":"auto";
  s.clutch=false;
  s.manualGear="1";
  s.gear="D1";
  s.cruise=null;

  notify(s.mode==="auto"?"자동변속 모드":"수동변속 모드");
  render();
}

function setClutch(value){
  if(s.mode!=="manual"){
    if(value)notify("클러치는 수동 모드에서만 작동합니다.");
    return;
  }

  s.clutch=value;
  render();
}

function setSignal(value){
  s.hazard=false;
  s.signal=s.signal===value?"off":value;
  render();
}

function setDoor(side,open){
  if(s.doorBusy)return;

  const isLeft=side==="left";
  const isOpen=isLeft?s.leftDoor:s.rightDoor;
  const sideName=isLeft?"좌측":"우측";

  if(open){
    if(!atStop()){
      notify("지정 지점에 완전히 정차해야 문을 열 수 있습니다.");
      return;
    }

    if((isLeft&&s.rightDoor)||(!isLeft&&s.leftDoor)){
      notify("반대편 문을 먼저 닫으세요.");
      return;
    }

    if(isOpen){
      notify("해당 문은 이미 열려 있습니다.");
      return;
    }

    s.cruise=null;

    // 문 열림 동작 시작과 동시에 열림 부저
    playDoorBuzzer("open");

    if(isLeft)s.leftDoor=true;
    else s.rightDoor=true;

    notify(sideName+" 문 열림 · 열림 부저");
    render();
    return;
  }

  if(!isOpen){
    notify("해당 문은 이미 닫혀 있습니다.");
    return;
  }

  s.doorBusy=true;
  s.cruise=null;

  // 문 닫힘 동작 시작과 동시에 닫힘 부저
  playDoorBuzzer("close");

  notify(sideName+" 문 닫는 중 · 닫힘 부저");
  render();

  clearTimeout(s.doorTimer);

  s.doorTimer=setTimeout(()=>{
    s.doorBusy=false;

    if(isLeft)s.leftDoor=false;
    else s.rightDoor=false;

    notify(sideName+" 문 닫힘");
    render();
  },1000);
}

function serviceStop(){
  if(!atStop()||(!s.leftDoor&&!s.rightDoor)){
    notify("정차 후 문을 열어 승객 승하차를 진행하세요.");
    return;
  }

  const stop=currentRoute();
  const alight=Math.min(s.passengers,stop.alight||0);

  s.passengers-=alight;

  const board=Math.min(
    stop.board||0,
    CAPACITY-s.passengers
  );

  s.passengers+=board;
  s.boardDone=true;

  notify(
    `${stop.name}: ${alight}명 하차, ${board}명 승차. `+
    `현재 ${s.passengers}/${CAPACITY}명`
  );

  render();
}

function depart(){
  if(!atStop()||!doorsClosed()){
    notify("완전 정차 및 문 닫힘 상태가 필요합니다.");
    return;
  }

  if(s.routeIndex===ROUTE.length-1){
    s.finished=true;
    notify("부산센텀 종점. 운행 종료!");
    render();
    return;
  }

  const stop=currentRoute();

  if((stop.board||stop.alight)&&!s.boardDone){
    notify("승객 승하차를 먼저 진행하세요.");
    return;
  }

  if(gear().startsWith("R")){
    notify("출발하려면 전진 기어를 선택하세요.");
    return;
  }

  if(gear()==="N"){
    notify("출발하려면 전진 기어를 선택하세요.");
    return;
  }

  s.stopRequested=false;
  s.stopTarget=Math.min(ROUTE.length-1,s.routeIndex+1);
  s.signal="left";

  notify("출발 준비 완료. 다음 지점: "+targetRoute().name);
  render();
}

function requestRestStop(name){
  const idx=ROUTE.findIndex(r=>r.name===name);

  if(idx<0)return;

  if(idx<=s.routeIndex){
    notify("이미 지나갔거나 현재 정차 지점인 휴게소입니다.");
    return;
  }

  s.stopTarget=idx;
  s.stopRequested=true;

  notify(name+" 정차 요청 설정. 해당 지점에서 제동해 정차하세요.");
  render();
}

function refuel(){
  if(!atStop()||!doorsClosed()){
    notify("지정 지점 정차 및 문 닫힘 상태에서 충전하세요.");
    return;
  }

  const amount=33-s.h2;

  if(amount<.001){
    notify("수소가 가득 찼습니다.");
    return;
  }

  s.h2=33;
  s.fuelCost+=amount*12000;

  notify(`수소 ${amount.toFixed(2)} kg 충전 · ${fmt(amount*12000)}원`);
  render();
}

function recharge(){
  if(!atStop()||!doorsClosed()){
    notify("지정 지점 정차 및 문 닫힘 상태에서 충전하세요.");
    return;
  }

  const amount=47-s.battery;

  if(amount<.001){
    notify("배터리가 가득 찼습니다.");
    return;
  }

  s.battery=47;
  s.electricCost+=amount*190;

  notify(`배터리 ${amount.toFixed(1)} kWh 충전 · ${fmt(amount*190)}원`);
  render();
}

function setCruise(value){
  if(value!==null&&(s.speed<1||!canDrive())){
    notify("주행 중에 정속 주행을 설정하세요.");
    return;
  }

  s.cruise=value;
  notify(value===null?"정속 주행 해제":`정속 주행 ${value} km/h`);
  render();
}

function laneChange(direction){
  if(s.speed<.1)return;

  const maxLane=s.distance<160?4:3;
  const nextLane=clamp(s.lane+direction,1,maxLane);

  if(nextLane===s.lane)return;

  s.lane=nextLane;
  notify(nextLane+"차로로 변경");
}

function stopAt(index){
  s.speed=0;
  s.cruise=null;
  s.routeIndex=index;
  s.stopTarget=Math.min(ROUTE.length-1,index+1);
  s.boardDone=false;
  s.stopRequested=false;
  s.signal="off";

  if(s.mode==="auto")s.gear="D1";
  else s.manualGear="1";

  notify(ROUTE[index].name+" 도착");

  if(index===ROUTE.length-1){
    s.finished=true;
    notify("부산센텀 종점 도착. 운행 종료!");
  }
}

function autoShift(){
  if(s.mode!=="auto")return;

  s.gear=
    s.speed<18?"D1":
    s.speed<38?"D2":
    s.speed<62?"D3":
    s.speed<95?"D4":"D5";
}

function update(dt){
  if(s.paused||s.finished)return;

  if(pressed.has("ArrowLeft")){
    s.steer=clamp(s.steer-20*dt,-60,60);

    if(steerCooldown<=0){
      laneChange(-1);
      steerCooldown=.7;
    }
  }else if(pressed.has("ArrowRight")){
    s.steer=clamp(s.steer+20*dt,-60,60);

    if(steerCooldown<=0){
      laneChange(1);
      steerCooldown=.7;
    }
  }else{
    s.steer+=(0-s.steer)*Math.min(1,dt*5);
    steerCooldown=0;
  }

  steerCooldown=Math.max(0,steerCooldown-dt);

  const accelerate=pressed.has("ArrowUp");
  const brake=pressed.has("ArrowDown");

  if(s.cruise!==null&&canDrive()){
    if(s.speed<s.cruise-1)s.speed+=3*dt;
    else if(s.speed>s.cruise+1)s.speed-=3*dt;
  }else if(accelerate&&canDrive()){
    s.speed+=3*dt;
  }else if(!accelerate&&!brake){
    s.speed=Math.max(0,s.speed-.25*dt);
  }

  if(brake){
    s.speed=Math.max(0,s.speed-6*dt);
    s.cruise=null;
  }

  if(!canDrive()&&accelerate){
    s.speed=Math.max(0,s.speed-1.5*dt);
  }

  s.speed=clamp(s.speed,0,capSpeed());

  const reverse=gear().startsWith("R");
  const km=s.speed*dt/3600;

  if(km>0&&s.speed>0){
    if(reverse){
      const previousIndex=Math.max(0,s.routeIndex-1);
      const previousKm=ROUTE[previousIndex].km;

      if(s.distance-km<=previousKm){
        s.distance=previousKm;
        s.speed=0;
        notify("이전 지점까지만 후진할 수 있습니다.");
      }else{
        s.distance-=km;
        s.h2=Math.max(0,s.h2-km*.036);
      }
    }else{
      const target=targetRoute();

      if(target.km-s.distance<=km){
        const travel=Math.max(0,target.km-s.distance);

        s.distance=target.km;
        s.revenue+=travel*2000;
        s.h2=Math.max(0,s.h2-travel*.036);

        stopAt(s.stopTarget);
      }else{
        s.distance+=km;
        s.revenue+=km*2000;
        s.h2=Math.max(0,s.h2-km*.036);
      }
    }
  }

  if(s.ac||s.heater){
    s.battery=Math.max(
      0,
      s.battery-.5*(s.fan/5)*dt/3600
    );
  }

  autoShift();
  render();
}

function drawRoad(){
  const canvas=$("road");
  const ctx=canvas.getContext("2d");
  const w=canvas.width,h=canvas.height;

  ctx.clearRect(0,0,w,h);

  const sky=ctx.createLinearGradient(0,0,0,h*.55);
  sky.addColorStop(0,"#17344b");
  sky.addColorStop(1,"#8bb3c3");

  ctx.fillStyle=sky;
  ctx.fillRect(0,0,w,h);

  ctx.fillStyle="#263d36";
  ctx.fillRect(0,h*.4,w,h*.6);

  ctx.fillStyle="#344f4d";
  ctx.beginPath();
  ctx.moveTo(0,h*.48);

  for(let a=0;a<=w;a+=60){
    ctx.lineTo(a,h*(.34+Math.abs(Math.sin(a*.012))*.12));
  }

  ctx.lineTo(w,h*.6);
  ctx.lineTo(0,h*.6);
  ctx.fill();

  const horizon=h*.4;
  const nearY=h*.96;
  const centerX=w/2;
  const nearWidth=w*.94;
  const farWidth=w*.22;
  const lanes=s.distance<160?4:3;

  ctx.fillStyle="#343b43";
  ctx.beginPath();
  ctx.moveTo(centerX-farWidth/2,horizon);
  ctx.lineTo(centerX+farWidth/2,horizon);
  ctx.lineTo(centerX+nearWidth/2,nearY);
  ctx.lineTo(centerX-nearWidth/2,nearY);
  ctx.closePath();
  ctx.fill();

  for(let i=0;i<lanes;i++){
    const nearX=centerX-nearWidth/2+nearWidth*i/lanes;
    const farX=centerX-farWidth/2+farWidth*i/lanes;

    if(s.distance<160&&i===0){
      ctx.fillStyle="#0876df";
      ctx.beginPath();
      ctx.moveTo(farX,horizon);
      ctx.lineTo(farX+farWidth/lanes,horizon);
      ctx.lineTo(nearX+nearWidth/lanes,nearY);
      ctx.lineTo(nearX,nearY);
      ctx.closePath();
      ctx.fill();

      ctx.fillStyle="white";
      ctx.font="bold 13px sans-serif";
      ctx.fillText("BUS",nearX+15,nearY-20);
    }

    if(i>0){
      ctx.strokeStyle="#f4f5f5";
      ctx.lineWidth=2;
      ctx.setLineDash([12,13]);
      ctx.beginPath();
      ctx.moveTo(farX,horizon);
      ctx.lineTo(nearX,nearY);
      ctx.stroke();
      ctx.setLineDash([]);
    }
  }

  ctx.strokeStyle="#f7d86d";
  ctx.lineWidth=4;
  ctx.beginPath();
  ctx.moveTo(centerX-nearWidth/2,nearY);
  ctx.lineTo(centerX-farWidth/2,horizon);
  ctx.moveTo(centerX+nearWidth/2,nearY);
  ctx.lineTo(centerX+farWidth/2,horizon);
  ctx.stroke();

  const laneWidth=nearWidth/lanes;
  const busX=centerX-nearWidth/2+laneWidth*(s.lane-.5);
  const busY=h*.83;

  ctx.save();
  ctx.translate(busX,busY);
  ctx.rotate(s.steer*Math.PI/180*.08);

  ctx.fillStyle="#0a1a26";
  ctx.fillRect(-36,-54,72,78);

  ctx.fillStyle="#3bc9b0";
  ctx.fillRect(-32,-52,64,69);

  ctx.fillStyle="#b9edee";
  ctx.fillRect(-27,-47,54,24);

  ctx.fillStyle="#102c3b";
  ctx.fillRect(-27,-18,54,28);

  ctx.fillStyle="#ffdf85";
  ctx.fillRect(-31,12,10,7);
  ctx.fillRect(21,12,10,7);

  ctx.fillStyle="#102c3b";
  ctx.font="bold 11px sans-serif";
  ctx.textAlign="center";
  ctx.fillText("FCEV",0,-30);

  ctx.restore();

  ctx.fillStyle="rgba(0,0,0,.55)";
  ctx.fillRect(12,12,300,52);

  ctx.fillStyle="#fff";
  ctx.font="bold 17px sans-serif";
  ctx.fillText(
    `${s.distance.toFixed(1)} km · ${Math.round(s.speed)} km/h`,
    22,34
  );

  ctx.font="12px sans-serif";
  ctx.fillText(`${s.lane}차로 · 경부고속도로`,22,52);

  if((s.signal!=="off"||s.hazard)&&Math.floor(performance.now()/400)%2===0){
    ctx.fillStyle="#ffbf45";
    ctx.font="bold 26px sans-serif";

    if(s.hazard||s.signal==="left")ctx.fillText("◀",w*.27,h*.78);
    if(s.hazard||s.signal==="right")ctx.fillText("▶",w*.68,h*.78);
  }
}

function drawMirror(id){
  const canvas=$(id);
  const ctx=canvas.getContext("2d");
  const w=canvas.width,h=canvas.height;

  ctx.clearRect(0,0,w,h);
  ctx.fillStyle="#172b38";
  ctx.fillRect(0,0,w,h);

  ctx.fillStyle="#303c44";
  ctx.beginPath();
  ctx.moveTo(w*.35,0);
  ctx.lineTo(w*.65,0);
  ctx.lineTo(w,h);
  ctx.lineTo(0,h);
  ctx.closePath();
  ctx.fill();

  ctx.strokeStyle="#e2e7e9";
  ctx.setLineDash([10,10]);
  ctx.beginPath();
  ctx.moveTo(w*.5,0);
  ctx.lineTo(w*.5,h);
  ctx.stroke();
  ctx.setLineDash([]);
}

function drawMap(){
  const canvas=$("map");
  const ctx=canvas.getContext("2d");
  const w=canvas.width,h=canvas.height;

  ctx.fillStyle="#101923";
  ctx.fillRect(0,0,w,h);

  const left=28;
  const right=w-28;
  const y=58;
  const maxKm=ROUTE[ROUTE.length-1].km;

  ctx.strokeStyle="#4c5b69";
  ctx.lineWidth=12;
  ctx.beginPath();
  ctx.moveTo(left,y);
  ctx.lineTo(right,y);
  ctx.stroke();

  ROUTE.forEach((route,index)=>{
    const px=left+(right-left)*route.km/maxKm;

    ctx.fillStyle=index===s.routeIndex?"#49e0a0":
      OPTIONAL_REST.has(route.name)?"#ffbf45":"#9eb1c4";

    ctx.beginPath();
    ctx.arc(px,y,OPTIONAL_REST.has(route.name)?5:3,0,Math.PI*2);
    ctx.fill();
  });

  const pos=left+(right-left)*s.distance/maxKm;

  ctx.fillStyle="#fff";
  ctx.beginPath();
  ctx.arc(pos,y,8,0,Math.PI*2);
  ctx.fill();

  ctx.strokeStyle="#49e0a0";
  ctx.lineWidth=3;
  ctx.stroke();

  ctx.fillStyle="#fff";
  ctx.font="bold 15px sans-serif";
  ctx.fillText("서울반포",left,y+28);
  ctx.fillText("대전",left+(right-left)*175/maxKm-12,y+28);
  ctx.fillText("부산센텀",right-65,y+28);

  ctx.fillStyle="#ffbf45";
  ctx.font="12px sans-serif";
  ctx.fillText("주황색: 선택 정차 휴게소",left,18);
}

function render(){
  const stop=currentRoute();

  $("speed").textContent=Math.round(s.speed);
  $("distance").textContent=s.distance.toFixed(1);
  $("passengers").textContent=s.passengers;
  $("gearDisplay").textContent=gear();

  $("modeBadge").textContent=s.mode==="auto"?"자동변속":"수동변속";
  $("modeBtn").textContent=s.mode==="auto"?"수동변속으로 전환":"자동변속으로 전환";

  $("clutchState").textContent=s.clutch?"밟음":"해제";
  $("signalState").textContent=s.hazard?"비상등":
    s.signal==="off"?"꺼짐":
    s.signal==="left"?"좌측":"우측";

  $("doorState").textContent=s.doorBusy?"문 닫는 중":
    s.leftDoor?"좌측 문 열림":
    s.rightDoor?"우측 문 열림":"모든 문 닫힘";

  $("state").textContent=s.finished?"운행 종료":
    s.paused?"일시정지":
    s.speed>0?"주행 중":
    atStop()?"지점 정차":"정차";

  $("h2Text").textContent=s.h2.toFixed(2)+" kg";
  $("batteryText").textContent=s.battery.toFixed(1)+" kWh";

  $("h2Bar").style.width=s.h2/33*100+"%";
  $("batteryBar").style.width=s.battery/47*100+"%";
  $("h2Bar").style.background=s.h2<5?"var(--red)":"var(--green)";

  $("revenue").textContent="₩"+fmt(s.revenue);
  $("fuelCost").textContent="₩"+fmt(s.fuelCost);
  $("electricCost").textContent="₩"+fmt(s.electricCost);

  const net=s.revenue-s.fuelCost-s.electricCost;
  $("net").textContent=(net<0?"−":"")+"₩"+fmt(Math.abs(net));
  $("net").className=net<0?"bad":"good";

  $("steerDisplay").textContent=Math.round(s.steer)+"°";
  $("steerBar").style.width=Math.abs(s.steer)/60*100+"%";
  $("laneDisplay").textContent=s.lane+"차로";

  $("laneDescription").textContent=s.distance<160?
    ["버스전용차로","추월 차로","일반 주행 차로","시설 진출입 차로"][s.lane-1]:
    ["추월 차로","주행 차로","시설 진출입 차로"][s.lane-1];

  $("cruiseText").textContent=s.cruise===null?"해제":s.cruise+" km/h";
  $("fanText").textContent=s.fan+"단";
  $("gearHelp").textContent=s.mode==="auto"?"자동변속 모드":"클러치를 밟고 변속";

  $("currentStop").textContent=stop.name;
  $("stopStatus").textContent=atStop()?"완전 정차":"주행 중";

  $("stopInfo").textContent=stop.board||stop.alight?
    `${stop.name}: 승차 ${stop.board||0}명 · 하차 ${stop.alight||0}명 예정`:
    `${stop.name}: 필요 시 정차`;

  $("nextStopText").textContent="다음 목표 지점: "+targetRoute().name;

  $("boardBtn").disabled=!atStop()||(!s.leftDoor&&!s.rightDoor);
  $("departBtn").disabled=!atStop()||!doorsClosed()||s.finished;
  $("fuelBtn").disabled=!atStop()||!doorsClosed()||s.h2>=32.999;
  $("chargeBtn").disabled=!atStop()||!doorsClosed()||s.battery>=46.999;

  $("clutchBtn").classList.toggle("active",s.clutch);
  $("sigL").classList.toggle("active",s.signal==="left");
  $("sigR").classList.toggle("active",s.signal==="right");
  $("hazard").classList.toggle("active",s.hazard);

  $("openFreqText").textContent=$("openFreq").value+" Hz";
  $("closeFreqText").textContent=$("closeFreq").value+" Hz";
  $("buzzerVolumeText").textContent=$("buzzerVolume").value+"%";

  document.querySelectorAll(".cruise").forEach(button=>{
    button.classList.toggle(
      "active",
      Number(button.dataset.speed)===s.cruise
    );
  });

  document.querySelectorAll("#gearButtons button").forEach(button=>{
    button.classList.toggle("active",button.textContent===gear());
  });

  drawRoad();
  drawMirror("mirrorL");
  drawMirror("mirrorR");
  drawMap();
}

function bindHold(id,key){
  const element=$(id);

  element.addEventListener("pointerdown",event=>{
    event.preventDefault();
    pressed.add(key);

    if(element.setPointerCapture){
      element.setPointerCapture(event.pointerId);
    }
  });

  const release=()=>pressed.delete(key);

  element.addEventListener("pointerup",release);
  element.addEventListener("pointercancel",release);
  element.addEventListener("lostpointercapture",release);
  element.addEventListener("contextmenu",event=>event.preventDefault());
}

bindHold("accelerate","ArrowUp");
bindHold("brake","ArrowDown");
bindHold("left","ArrowLeft");
bindHold("right","ArrowRight");

$("modeBtn").onclick=toggleMode;

$("pauseBtn").onclick=()=>{
  s.paused=!s.paused;
  $("pauseBtn").textContent=s.paused?"운행 재개":"일시정지";
  notify(s.paused?"일시정지":"운행 재개");
  render();
};

$("resetBtn").onclick=()=>{
  clearTimeout(s.doorTimer);
  pressed.clear();
  s=freshState();
  steerCooldown=0;
  $("pauseBtn").textContent="일시정지";
  $("ac").checked=false;
  $("heater").checked=false;
  $("fan").value="3";
  $("openFreq").value="500";
  $("closeFreq").value="580";
  $("buzzerVolume").value="50";
  $("buzzerStatus").textContent="대기";
  $("log").innerHTML="";

  notify("초기화 완료. 경부고속도로 노선 · 수동변속 · 정원 27석.");
  render();
};

$("clutchBtn").onclick=()=>setClutch(!s.clutch);
$("sigL").onclick=()=>setSignal("left");
$("sigR").onclick=()=>setSignal("right");

$("hazard").onclick=()=>{
  s.hazard=!s.hazard;
  s.signal="off";
  render();
};

$("horn").onclick=()=>{
  playHorn();
  notify("경적 작동");
};

$("center").onclick=()=>{
  s.steer=0;
  render();
};

$("openL").onclick=()=>setDoor("left",true);
$("closeL").onclick=()=>setDoor("left",false);
$("openR").onclick=()=>setDoor("right",true);
$("closeR").onclick=()=>setDoor("right",false);

$("testOpen").onclick=()=>playDoorBuzzer("open");
$("testClose").onclick=()=>playDoorBuzzer("close");

$("openFreq").oninput=render;
$("closeFreq").oninput=render;
$("buzzerVolume").oninput=render;

$("boardBtn").onclick=serviceStop;
$("departBtn").onclick=depart;
$("fuelBtn").onclick=refuel;
$("chargeBtn").onclick=recharge;
$("cruiseOff").onclick=()=>setCruise(null);

document.querySelectorAll(".restStop").forEach(button=>{
  button.onclick=()=>requestRestStop(button.dataset.stop);
});

document.querySelectorAll(".cruise").forEach(button=>{
  button.onclick=()=>setCruise(Number(button.dataset.speed));
});

$("ac").onchange=event=>{
  s.ac=event.target.checked;

  if(s.ac){
    s.heater=false;
    $("heater").checked=false;
  }

  render();
};

$("heater").onchange=event=>{
  s.heater=event.target.checked;

  if(s.heater){
    s.ac=false;
    $("ac").checked=false;
  }

  render();
};

$("fan").oninput=event=>{
  s.fan=Number(event.target.value);
  render();
};

$("clearLog").onclick=()=>{
  s.log=[];
  $("log").innerHTML="";
};

["1","2","3","4","5","N","R1","R2","R3"].forEach(g=>{
  const button=document.createElement("button");
  button.textContent=g;
  button.onclick=()=>shiftGear(g);
  $("gearButtons").appendChild(button);
});

function keyDown(event){
  if(event.target&&["INPUT","TEXTAREA","SELECT"].includes(event.target.tagName)){
    return;
  }

  if(["ArrowUp","ArrowDown","ArrowLeft","ArrowRight","Space"].includes(event.code)){
    event.preventDefault();
  }

  if(event.repeat)return;

  pressed.add(event.code);

  if(event.code==="ShiftLeft"||event.code==="ShiftRight"){
    setClutch(true);
  }

  if(event.code==="KeyA")setSignal("left");
  if(event.code==="KeyF")setSignal("right");

  if(event.code==="KeyB"){
    playHorn();
    notify("경적 작동");
  }

  if(event.code==="KeyO"){
    if(pressed.has("KeyW"))setDoor("left",true);
    if(pressed.has("KeyE"))setDoor("right",true);
  }

  if(event.code==="KeyP"){
    if(pressed.has("KeyW"))setDoor("left",false);
    if(pressed.has("KeyE"))setDoor("right",false);
  }

  if(GEAR_MAP[event.code]){
    shiftGear(GEAR_MAP[event.code]);
  }
}

function keyUp(event){
  pressed.delete(event.code);

  if(
    (event.code==="ShiftLeft"||event.code==="ShiftRight")&&
    !pressed.has("ShiftLeft")&&
    !pressed.has("ShiftRight")
  ){
    setClutch(false);
  }
}

window.addEventListener("keydown",keyDown);
window.addEventListener("keyup",keyUp);

window.addEventListener("blur",()=>{
  pressed.clear();
  s.steer=0;
  steerCooldown=0;

  if(s.mode==="manual")s.clutch=false;
});

document.addEventListener("visibilitychange",()=>{
  if(document.hidden){
    pressed.clear();
    steerCooldown=0;
  }
});

function frame(now){
  const dt=Math.min((now-last)/1000,.05);
  last=now;
  update(dt);
  requestAnimationFrame(frame);
}

log("준비 완료. 경부고속도로 노선을 따라 운행합니다.");
log("출입문 작동 시 열림·닫힘 부저가 울립니다. 주파수와 음량을 조절하세요.");
log("서울반포 출발: 문 열기 → 승객 승하차 → 문 닫기 → 출발 준비.");

render();
requestAnimationFrame(frame);
</script>
</body>
</html>
