
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
button,select{background:#223348;color:var(--text);border:1px solid #3a5068;border-radius:8px;padding:9px 11px;font:inherit;cursor:pointer}
button:hover{border-color:var(--green)}button.active{background:#145c48;border-color:var(--green)}button:disabled{opacity:.45;cursor:not-allowed}
main{max-width:1450px;margin:auto;padding:14px;display:grid;gap:14px}
.grid{display:grid;grid-template-columns:minmax(0,1.7fr) minmax(300px,1fr);gap:14px}
.panel{background:var(--panel);border:1px solid var(--line);border-radius:13px;padding:14px;min-width:0}
.hud{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:9px;margin-bottom:12px}
.metric{background:#0b1520;border:1px solid #24374a;border-radius:9px;padding:10px}
.metric .value{font-size:23px;font-weight:750;font-variant-numeric:tabular-nums}
.metric small{display:block}
canvas{display:block;width:100%;border-radius:8px;background:#101b26}
#road{aspect-ratio:16/7}#map{height:180px}
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
#routeList{max-height:250px;overflow:auto;padding-left:22px}
#routeList li{padding:3px 0}
#routeList li.current{color:var(--green);font-weight:700}
#routeList li.done{color:#8195a9}
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
<small>경부고속도로 · 서울반포 → 부산센텀 · 27석 수소전기버스</small>
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
<div class="between"><h2>경부고속도로 노선도</h2><small>파란색: 버스전용차로 표시</small></div>
<canvas id="map" width="960" height="180"></canvas>
<div class="between"><small>서울반포</small><small id="mapCurrent">현재 위치: 서울반포</small><small>부산센텀</small></div>
</div>
</section>

<section class="panel" style="margin-top:14px">
<h2>운전 조작</h2>
<div class="controls">
<button id="accelerate">▲ 가속</button>
<button id="brake">▼ 브레이크</button>
<button id="horn">📢 경적 B</button>
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
<p><small>방향키를 누르고 있으면 2초마다 한 차로 변경</small></p>
</div>
</section>

<section class="panel" style="margin-top:14px">
<h2>운행 노선</h2>
<p class="muted">필수 정차 휴게소는 반드시 정차합니다. 필요 시 정차 휴게소는 도착 후 정차 또는 통과를 선택할 수 있습니다.</p>
<ol id="routeList"></ol>
<div class="status">다음 지점: <b id="nextStop">서울반포</b></div>
<div class="row">
<button id="stopNowBtn">현재 지점 정차</button>
<button id="passBtn">필요 시 정차 지점 통과</button>
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
<div class="row" style="margin-top:8px"><button id="clutchBtn">클러치 Shift</button><span id="gearHelp" class="muted">수동변속 모드</span></div>
<small>변속하려면 클러치를 밟으세요. 전진·후진 변경은 완전 정차 후 가능합니다.</small>
</div>

<div class="section">
<h2>출입문</h2>
<div class="row"><button id="openL">좌측 문 열기 (W+O)</button><button id="closeL">좌측 문 닫기 (W+P)</button></div>
<div class="row" style="margin-top:7px"><button id="openR">우측 문 열기 (E+O)</button><button id="closeR">우측 문 닫기 (E+P)</button></div>
<small>지점에 완전히 정차해야 문을 열 수 있습니다.</small>
</div>

<div class="section">
<h2>출입문 부저 주파수</h2>
<div class="between"><label for="doorFreq1">첫 번째 부저</label><b id="doorFreq1Label">440 Hz</b></div>
<input id="doorFreq1" type="range" min="150" max="1200" step="10" value="440">
<div class="between"><label for="doorFreq2">두 번째 부저</label><b id="doorFreq2Label">370 Hz</b></div>
<input id="doorFreq2" type="range" min="150" max="1200" step="10" value="370">
<div class="between"><label for="doorVolume">부저 음량</label><b id="doorVolumeLabel">60%</b></div>
<input id="doorVolume" type="range" min="5" max="100" step="5" value="60">
<div class="row"><button id="testDoorBuzzer">🔊 부저 테스트</button></div>
<small>문 열기·닫기 조작 시 설정한 부저가 재생됩니다.</small>
</div>

<div class="section">
<h2>정류장 업무</h2>
<div class="row"><button id="boardBtn">승객 승하차</button><button id="departBtn">출발 준비</button></div>
<div class="row" style="margin-top:7px"><button id="fuelBtn">수소 충전</button><button id="chargeBtn">배터리 충전</button></div>
<small id="stopInfo">서울반포: 승차 24명 예정</small>
</div>
</section>

<section class="panel" style="margin-top:14px">
<h2>냉난방</h2>
<div class="row"><label><input type="checkbox" id="ac"> 에어컨</label><label><input type="checkbox" id="heater"> 난방</label></div>
<div class="between"><label for="fan">송풍 단계</label><b id="fanText">3단</b></div>
<input id="fan" type="range" min="1" max="5" value="3">
</section>

<section class="panel" style="margin-top:14px">
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

<section class="panel" style="margin-top:14px">
<h2>운행 정산</h2>
<div class="between"><span>운임 수입</span><b id="revenue">₩0</b></div>
<div class="between"><span>수소 충전 비용</span><b id="fuelCost">₩0</b></div>
<div class="between"><span>전기 충전 비용</span><b id="electricCost">₩0</b></div>
<div class="between"><b>순이익</b><b id="net" class="good">₩0</b></div>
<small>운임 2,000원/km · 수소 12,000원/kg · 전기 190원/kWh</small>
</section>

<section class="panel" style="margin-top:14px">
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
<span><kbd>5</kbd> 1단</span><span><kbd>6</kbd> 2단</span><span><kbd>7</kbd> 3단</span>
<span><kbd>8</kbd> 4단</span><span><kbd>9</kbd> 5단</span><span><kbd>4</kbd> N</span>
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

<footer>초록도시 II FCEV · 경부고속도로 노선 · 27석 · 수동변속 기본</footer>

<script>
"use strict";
const $=id=>document.getElementById(id);
const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
const fmt=n=>Math.round(n).toLocaleString("ko-KR");
const CAPACITY=27;
const pressed=new Set();
const GEAR_MAP={
 Digit5:"1",Numpad5:"1",Digit6:"2",Numpad6:"2",
 Digit7:"3",Numpad7:"3",Digit8:"4",Numpad8:"4",
 Digit9:"5",Numpad9:"5",Digit4:"N",Numpad4:"N",
 Digit3:"R1",Numpad3:"R1",Digit2:"R2",Numpad2:"R2",
 Digit1:"R3",Numpad1:"R3"
};
const FCAP={"1":25,"2":45,"3":70,"4":105,"5":197};
const RCAP={R1:10,R2:20,R3:30};

/*
 * 경부고속도로를 기준으로 한 시뮬레이터상의 경유 지점.
 * 거리는 실제 도로 측량 데이터가 아닌 게임용 진행 거리다.
 * 필수 정차: 판교·천안호두·김천·경주 휴게소.
 * 선택 정차: 신탄진·옥천·영동·영천 휴게소.
 */
const ROUTE=[
 {name:"서울반포",km:0,type:"stop",board:24,alight:0},
 {name:"판교",km:24,type:"stop",board:0,alight:0},
 {name:"판교휴게소",km:31,type:"mandatory",board:0,alight:0},
 {name:"죽전",km:40,type:"stop",board:0,alight:0},
 {name:"동탄",km:52,type:"stop",board:0,alight:0},
 {name:"평택지제",km:66,type:"stop",board:0,alight:0},
 {name:"천안호두휴게소",km:86,type:"mandatory",board:0,alight:0},
 {name:"천안아산",km:102,type:"stop",board:0,alight:0},
 {name:"오송",km:119,type:"stop",board:0,alight:0},
 {name:"신탄진휴게소",km:143,type:"optional",board:0,alight:0},
 {name:"북대전분기점",km:153,type:"pass",board:0,alight:0},
 {name:"대전",km:160,type:"stop",board:12,alight:14},
 {name:"옥천",km:177,type:"stop",board:0,alight:0},
 {name:"옥천휴게소",km:186,type:"optional",board:0,alight:0},
 {name:"영동",km:203,type:"stop",board:0,alight:0},
 {name:"영동휴게소",km:216,type:"optional",board:0,alight:0},
 {name:"김천",km:247,type:"stop",board:0,alight:0},
 {name:"김천휴게소",km:258,type:"mandatory",board:0,alight:0},
 {name:"동김천",km:274,type:"stop",board:0,alight:0},
 {name:"구미",km:291,type:"stop",board:0,alight:0},
 {name:"왜관",km:310,type:"stop",board:0,alight:0},
 {name:"대구",km:328,type:"stop",board:0,alight:0},
 {name:"동대구",km:337,type:"stop",board:0,alight:0},
 {name:"경산",km:349,type:"stop",board:0,alight:0},
 {name:"영천휴게소",km:365,type:"optional",board:0,alight:0},
 {name:"경주",km:385,type:"stop",board:0,alight:0},
 {name:"경주휴게소",km:395,type:"mandatory",board:0,alight:0},
 {name:"북울산",km:403,type:"stop",board:0,alight:0},
 {name:"울산시내",km:410,type:"stop",board:0,alight:0},
 {name:"부산센텀",km:416.2,type:"terminal",board:0,alight:27}
];
const ROUTE_LENGTH=ROUTE[ROUTE.length-1].km;
let s,steerCooldown=0,last=performance.now();
let hornAudio=null;

function freshState(){
 return {
  speed:0,distance:0,lane:3,steer:0,mode:"manual",
  gear:"D1",manualGear:"1",clutch:false,
  leftDoor:false,rightDoor:false,doorBusy:false,signal:"off",hazard:false,
  passengers:0,h2:33,battery:47,revenue:0,fuelCost:0,electricCost:0,
  ac:false,heater:false,fan:3,cruise:null,paused:false,finished:false,
  routeIndex:0,warningSeen:new Set(),log:[],doorTimer:null,
  boardDone:false,lastDirection:"forward",optionalChoice:null
 };
}
s=freshState();

function log(m){
 s.log.unshift("["+new Date().toLocaleTimeString("ko-KR",{hour12:false})+"] "+m);
 s.log=s.log.slice(0,120);
 $("log").innerHTML=s.log.map(v=>"<div>"+v.replace(/[&<>]/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;"}[c]))+"</div>").join("");
}
function notify(m){$("roadStatus").textContent=m;log(m)}
function currentStop(){return ROUTE[s.routeIndex]}
function nextStop(){return ROUTE[Math.min(ROUTE.length-1,s.routeIndex+1)]}
function atStop(){return Math.abs(s.distance-currentStop().km)<.09&&s.speed<.1}
function doorsClosed(){return !s.leftDoor&&!s.rightDoor&&!s.doorBusy}
function gear(){return s.mode==="auto"?s.gear:s.manualGear}
function capSpeed(){
 const g=gear();
 if(g==="N")return 0;
 if(g.startsWith("R"))return RCAP[g]||10;
 return s.mode==="auto"?197:(FCAP[g]||25);
}
function canDrive(){
 return !s.paused&&!s.finished&&doorsClosed()&&s.h2>.001&&
 !(s.mode==="manual"&&s.clutch)&&gear()!=="N";
}
function isStopPoint(p){return p.type==="stop"||p.type==="mandatory"||p.type==="optional"||p.type==="terminal"}
function stopLabel(p){
 if(p.type==="mandatory")return "필수 정차";
 if(p.type==="optional")return "필요 시 정차";
 if(p.type==="terminal")return "종점";
 if(p.type==="pass")return "분기점 통과";
 return "일반 정류장";
}
function renderRoute(){
 const ol=$("routeList");
 ol.innerHTML="";
 ROUTE.forEach((p,i)=>{
  const li=document.createElement("li");
  li.textContent=`${p.name} · ${p.km.toFixed(1)} km · ${stopLabel(p)}`;
  if(i===s.routeIndex)li.classList.add("current");
  else if(i<s.routeIndex)li.classList.add("done");
  ol.appendChild(li);
 });
 $("nextStop").textContent=currentStop().name;
 $("mapCurrent").textContent="현재 위치: "+currentStop().name;
}
function shiftGear(g){
 if(s.mode!=="manual"){notify("자동변속 모드입니다. 수동변속으로 전환하세요.");return}
 if(!s.clutch){notify("변속하려면 Shift 클러치를 밟으세요.");return}
 const old=s.manualGear;
 const oldDir=old==="N"?s.lastDirection:old.startsWith("R")?"reverse":"forward";
 const newDir=g==="N"?null:g.startsWith("R")?"reverse":"forward";
 if(s.speed>.1&&newDir!==null&&newDir!==oldDir){
  notify("전진과 후진 사이의 변속은 완전히 정차한 뒤 가능합니다.");return;
 }
 s.manualGear=g;
 if(newDir)s.lastDirection=newDir;
 if(g==="N"||g.startsWith("R"))s.cruise=null;
 notify(old+" → "+g+" 변속 완료");
 render();
}
function toggleMode(){
 if(s.speed>.1||!doorsClosed()){
  notify("모드 변경은 완전 정차 및 문 닫힘 상태에서 가능합니다.");return;
 }
 s.mode=s.mode==="auto"?"manual":"auto";
 s.clutch=false;s.manualGear="1";s.gear="D1";s.cruise=null;
 notify(s.mode==="auto"?"자동변속 모드":"수동변속 모드");render();
}
function setClutch(v){
 if(s.mode!=="manual"){if(v)notify("클러치는 수동 모드에서만 작동합니다.");return}
 s.clutch=v;render();
}
function setSignal(v){s.hazard=false;s.signal=s.signal===v?"off":v;render()}

function playDoorBuzzer(){
 try{
  const AudioContextClass=window.AudioContext||window.webkitAudioContext;
  if(!AudioContextClass)return;
  if(!hornAudio)hornAudio=new AudioContextClass();
  const ctx=hornAudio;
  const f1=Number($("doorFreq1").value);
  const f2=Number($("doorFreq2").value);
  const volume=Number($("doorVolume").value)/100;
  ctx.resume().then(()=>{
   const start=ctx.currentTime;
   const notes=[
    {time:0,duration:.28,frequency:f1},
    {time:.43,duration:.65,frequency:f2}
   ];
   for(const note of notes){
    const osc=ctx.createOscillator(),gain=ctx.createGain();
    osc.type="sine";
    osc.frequency.setValueAtTime(note.frequency,start+note.time);
    const begin=start+note.time,end=begin+note.duration;
    gain.gain.setValueAtTime(.0001,begin);
    gain.gain.linearRampToValueAtTime(volume*.32,begin+.035);
    gain.gain.setValueAtTime(volume*.32,end-.04);
    gain.gain.linearRampToValueAtTime(.0001,end);
    osc.connect(gain);gain.connect(ctx.destination);
    osc.start(begin);osc.stop(end+.02);
    osc.onended=()=>{osc.disconnect();gain.disconnect()};
   }
  });
 }catch(err){console.warn("출입문 부저 재생 오류:",err)}
}
function updateBuzzerLabels(){
 $("doorFreq1Label").textContent=$("doorFreq1").value+" Hz";
 $("doorFreq2Label").textContent=$("doorFreq2").value+" Hz";
 $("doorVolumeLabel").textContent=$("doorVolume").value+"%";
}

function setDoor(side,open){
 if(s.doorBusy)return;
 if(open){
  if(!atStop()){
   notify("현재 노선 지점에 완전히 정차해야 문을 열 수 있습니다.");return;
  }
  if((side==="left"&&s.rightDoor)||(side==="right"&&s.leftDoor)){
   notify("반대편 문을 먼저 닫으세요.");return;
  }
  playDoorBuzzer();
  if(side==="left")s.leftDoor=true;else s.rightDoor=true;
  s.cruise=null;
  notify((side==="left"?"좌측":"우측")+" 문 열림");
  render();return;
 }
 const opened=side==="left"?s.leftDoor:s.rightDoor;
 if(!opened){notify("해당 문은 이미 닫혀 있습니다.");return}
 playDoorBuzzer();
 s.doorBusy=true;notify("문 닫는 중…");render();
 clearTimeout(s.doorTimer);
 s.doorTimer=setTimeout(()=>{
  s.doorBusy=false;
  if(side==="left")s.leftDoor=false;else s.rightDoor=false;
  notify((side==="left"?"좌측":"우측")+" 문 닫힘");render();
 },1000);
}

function serviceStop(){
 if(!atStop()||(!s.leftDoor&&!s.rightDoor)){
  notify("정차 후 문을 열어 승객 승하차를 진행하세요.");return;
 }
 const p=currentStop();
 if(p.type==="optional"||p.type==="mandatory"||p.type==="pass"){
  notify("휴게소·분기점에서는 일반 정류장 승객 승하차를 처리하지 않습니다.");return;
 }
 if(s.boardDone){notify("이 지점의 승객 업무는 이미 완료했습니다.");return}
 const alight=Math.min(s.passengers,p.alight||0);
 s.passengers-=alight;
 const board=Math.min(p.board||0,CAPACITY-s.passengers);
 s.passengers+=board;
 s.boardDone=true;
 notify(`${alight}명 하차, ${board}명 승차 · 현재 ${s.passengers}/${CAPACITY}명`);
 render();
}
function depart(){
 if(!atStop()||!doorsClosed()){
  notify("완전 정차 및 문 닫힘 상태가 필요합니다.");return;
 }
 const p=currentStop();
 if(p.type==="terminal"){
  s.finished=true;notify("부산센텀 종점. 운행 종료!");render();return;
 }
 if(p.type==="mandatory"&&!s.boardDone){
  // 필수 휴게소는 휴식·점검을 완료해야 출발 가능
  notify("필수 정차 휴게소입니다. '출발 준비'를 다시 눌러 정차 확인을 완료하세요.");
  s.boardDone=true;render();return;
 }
 if(p.type==="optional"&&!s.boardDone){
  notify("필요 시 정차 지점입니다. 출발하려면 '출발 준비'를 다시 눌러 통과를 확정하세요.");
  s.boardDone=true;render();return;
 }
 if(p.type==="stop"&&!s.boardDone){
  notify("승객 승하차를 먼저 진행하세요.");return;
 }
 if(gear().startsWith("R")){notify("전진 기어를 선택하세요.");return}
 s.signal="left";
 s.optionalChoice=null;
 if(s.routeIndex===ROUTE.length-1){s.finished=true;notify("운행 종료");render();return}
 notify("출발 준비 완료. ↑ 키로 가속하세요.");
 render();
}
function passOptional(){
 const p=currentStop();
 if(p.type!=="optional"){
  notify("현재 지점은 필요 시 정차 휴게소가 아닙니다.");return;
 }
 if(!atStop()||!doorsClosed()){
  notify("휴게소 통과 확정은 완전 정차 및 문 닫힘 상태에서 가능합니다.");return;
 }
 s.boardDone=true;
 s.optionalChoice="pass";
 notify(p.name+" 통과 선택. '출발 준비'를 눌러 운행을 재개하세요.");
 render();
}
function stopNow(){
 const p=currentStop();
 if(!isStopPoint(p)){notify("현재 지점은 정차 지점이 아닙니다.");return}
 if(s.speed>.1){notify("정차 지점에 도착한 뒤 정차할 수 있습니다.");return}
 if(p.type==="optional"){
  s.boardDone=true;s.optionalChoice="stop";
  notify(p.name+" 선택 정차. 휴식 후 출발 준비를 진행하세요.");
 }else if(p.type==="mandatory"){
  s.boardDone=true;
  notify(p.name+" 필수 정차 완료. 출발 준비를 진행하세요.");
 }else{
  notify(p.name+" 정차 중.");
 }
 render();
}
function refuel(){
 if(!atStop()||!doorsClosed()){notify("정차 및 문 닫힘 상태에서 충전하세요.");return}
 const n=33-s.h2;
 if(n<.001){notify("수소가 가득 찼습니다.");return}
 s.h2=33;s.fuelCost+=n*12000;
 notify(`수소 ${n.toFixed(2)} kg 충전 · ${fmt(n*12000)}원`);render();
}
function recharge(){
 if(!atStop()||!doorsClosed()){notify("정차 및 문 닫힘 상태에서 충전하세요.");return}
 const n=47-s.battery;
 if(n<.001){notify("배터리가 가득 찼습니다.");return}
 s.battery=47;s.electricCost+=n*190;
 notify(`배터리 ${n.toFixed(1)} kWh 충전 · ${fmt(n*190)}원`);render();
}
function setCruise(v){
 if(v!==null&&(s.speed<1||!canDrive())){notify("주행 중에 정속 주행을 설정하세요.");return}
 s.cruise=v;notify(v===null?"정속 주행 해제":`정속 주행 ${v} km/h`);render();
}
function laneChange(d){
 if(s.speed<.1)return;
 const maxLane=4,n=clamp(s.lane+d,1,maxLane);
 if(n===s.lane)return;
 s.lane=n;notify(n+"차로로 변경");
}
function autoShift(){
 if(s.mode==="auto"){
  s.gear=s.speed<18?"D1":s.speed<38?"D2":s.speed<62?"D3":s.speed<95?"D4":"D5";
 }
}
function arriveAtNext(){
 const next=s.routeIndex+1;
 if(next>=ROUTE.length){s.finished=true;s.speed=0;notify("부산센텀 종점. 운행 종료!");return}
 s.routeIndex=next;
 s.distance=ROUTE[next].km;s.speed=0;s.cruise=null;
 s.boardDone=false;s.optionalChoice=null;
 if(s.mode==="auto")s.gear="D1";else s.manualGear="1";
 s.signal="off";
 const p=currentStop();
 if(p.type==="pass"){
  notify(p.name+" 통과 구간 도착. 계속 운행합니다.");
  s.boardDone=true;
  return;
 }
 notify(p.name+" 도착 · "+stopLabel(p));
 if(p.type==="mandatory"){
  s.boardDone=false;
  notify(p.name+" 필수 정차. 휴식 후 출발 준비를 진행하세요.");
 }
 if(p.type==="optional"){
  s.boardDone=false;
  notify(p.name+" 도착. 필요 시 정차하거나 통과를 선택하세요.");
 }
 if(p.type==="terminal"){
  s.finished=true;notify("부산센텀 종점 도착. 운행 종료!");
 }
}
function update(dt){
 if(s.paused||s.finished)return;
 if(pressed.has("ArrowLeft")){
  s.steer=clamp(s.steer-20*dt,-60,60);
  if(steerCooldown<=0){laneChange(-1);steerCooldown=2}
 }else if(pressed.has("ArrowRight")){
  s.steer=clamp(s.steer+20*dt,-60,60);
  if(steerCooldown<=0){laneChange(1);steerCooldown=2}
 }else{s.steer+=(0-s.steer)*Math.min(1,dt*5);steerCooldown=0}
 steerCooldown=Math.max(0,steerCooldown-dt);

 const accel=pressed.has("ArrowUp"),brake=pressed.has("ArrowDown");
 if(s.cruise!==null&&canDrive()){
  if(s.speed<s.cruise-1)s.speed+=3*dt;
  else if(s.speed>s.cruise+1)s.speed-=3*dt;
 }else if(accel&&canDrive())s.speed+=3*dt;
 else if(!accel&&!brake)s.speed=Math.max(0,s.speed-.25*dt);
 if(brake){s.speed=Math.max(0,s.speed-6*dt);s.cruise=null}
 if(!canDrive()&&accel)s.speed=Math.max(0,s.speed-1.5*dt);
 s.speed=clamp(s.speed,0,capSpeed());

 const rev=gear().startsWith("R"),km=s.speed*dt/3600;
 if(km>0&&s.speed>0){
  if(rev){
   const prev=ROUTE[Math.max(0,s.routeIndex-1)].km;
   if(s.distance-km<=prev){s.distance=prev;s.speed=0;notify("이전 지점까지만 후진할 수 있습니다.")}
   else{s.distance-=km;s.h2=Math.max(0,s.h2-km*.036)}
  }else{
   const target=nextStop().km;
   if(target-s.distance<=km){
    const moved=Math.max(0,target-s.distance);
    s.revenue+=moved*2000;
    s.h2=Math.max(0,s.h2-moved*.036);
    s.distance=target;
    arriveAtNext();
   }else{
    s.distance+=km;s.revenue+=km*2000;
    s.h2=Math.max(0,s.h2-km*.036);
   }
  }
 }
 if(s.ac||s.heater)s.battery=Math.max(0,s.battery-.5*(s.fan/5)*dt/3600);
 autoShift();render();
}

function drawRoad(){
 const c=$("road"),x=c.getContext("2d"),w=c.width,h=c.height;
 x.clearRect(0,0,w,h);
 const g=x.createLinearGradient(0,0,0,h*.55);
 g.addColorStop(0,"#17344b");g.addColorStop(1,"#8bb3c3");
 x.fillStyle=g;x.fillRect(0,0,w,h);
 x.fillStyle="#263d36";x.fillRect(0,h*.4,w,h*.6);
 x.fillStyle="#344f4d";x.beginPath();x.moveTo(0,h*.48);
 for(let a=0;a<=w;a+=60)x.lineTo(a,h*(.34+Math.abs(Math.sin(a*.012))*.12));
 x.lineTo(w,h*.6);x.lineTo(0,h*.6);x.fill();

 const hy=h*.4,ny=h*.96,cx=w/2,rn=w*.94,rf=w*.22,n=4;
 x.fillStyle="#343b43";x.beginPath();
 x.moveTo(cx-rf/2,hy);x.lineTo(cx+rf/2,hy);
 x.lineTo(cx+rn/2,ny);x.lineTo(cx-rn/2,ny);x.closePath();x.fill();
 for(let i=0;i<n;i++){
  const xn=cx-rn/2+rn*i/n,xf=cx-rf/2+rf*i/n;
  if(i===0){
   x.fillStyle="#0876df";x.beginPath();
   x.moveTo(xf,hy);x.lineTo(xf+rf/n,hy);
   x.lineTo(xn+rn/n,ny);x.lineTo(xn,ny);x.closePath();x.fill();
   x.fillStyle="white";x.font="bold 13px sans-serif";x.fillText("BUS",xn+15,ny-20);
  }
  if(i>0){
   x.strokeStyle="#f4f5f5";x.lineWidth=2;x.setLineDash([12,13]);
   x.beginPath();x.moveTo(xf,hy);x.lineTo(xn,ny);x.stroke();x.setLineDash([]);
  }
 }
 x.strokeStyle="#f7d86d";x.lineWidth=4;x.beginPath();
 x.moveTo(cx-rn/2,ny);x.lineTo(cx-rf/2,hy);
 x.moveTo(cx+rn/2,ny);x.lineTo(cx+rf/2,hy);x.stroke();
 const lw=rn/n,bx=cx-rn/2+lw*(s.lane-.5),by=h*.83;
 x.save();x.translate(bx,by);x.rotate(s.steer*Math.PI/180*.08);
 x.fillStyle="#0a1a26";x.fillRect(-36,-54,72,78);
 x.fillStyle="#3bc9b0";x.fillRect(-32,-52,64,69);
 x.fillStyle="#b9edee";x.fillRect(-27,-47,54,24);
 x.fillStyle="#102c3b";x.fillRect(-27,-18,54,28);
 x.fillStyle="#ffdf85";x.fillRect(-31,12,10,7);x.fillRect(21,12,10,7);
 x.fillStyle="#102c3b";x.font="bold 11px sans-serif";x.textAlign="center";x.fillText("FCEV",0,-30);x.restore();
 x.fillStyle="rgba(0,0,0,.55)";x.fillRect(12,12,300,52);
 x.fillStyle="#fff";x.font="bold 17px sans-serif";
 x.fillText(`${s.distance.toFixed(1)} km · ${Math.round(s.speed)} km/h`,22,34);
 x.font="12px sans-serif";x.fillText(`${s.lane}차로 · 경부고속도로`,22,52);
 if((s.signal!=="off"||s.hazard)&&Math.floor(performance.now()/400)%2===0){
  x.fillStyle="#ffbf45";x.font="bold 26px sans-serif";
  if(s.hazard||s.signal==="left")x.fillText("◀",w*.27,h*.78);
  if(s.hazard||s.signal==="right")x.fillText("▶",w*.68,h*.78);
 }
}
function drawMirror(id){
 const c=$(id),x=c.getContext("2d"),w=c.width,h=c.height;
 x.clearRect(0,0,w,h);x.fillStyle="#172b38";x.fillRect(0,0,w,h);
 x.fillStyle="#303c44";x.beginPath();x.moveTo(w*.35,0);x.lineTo(w*.65,0);
 x.lineTo(w,h);x.lineTo(0,h);x.closePath();x.fill();
 x.strokeStyle="#e2e7e9";x.setLineDash([10,10]);
 x.beginPath();x.moveTo(w*.5,0);x.lineTo(w*.5,h);x.stroke();x.setLineDash([]);
}
function drawMap(){
 const c=$("map"),x=c.getContext("2d"),w=c.width,h=c.height;
 x.fillStyle="#101923";x.fillRect(0,0,w,h);
 const left=38,right=w-38,y=80;
 x.strokeStyle="#4c5b69";x.lineWidth=13;x.beginPath();x.moveTo(left,y);x.lineTo(right,y);x.stroke();
 x.strokeStyle="#238dff";x.lineWidth=5;x.beginPath();x.moveTo(left,y-7);x.lineTo(right,y-7);x.stroke();
 x.strokeStyle="#a6b4c2";x.lineWidth=2;x.setLineDash([8,7]);
 x.beginPath();x.moveTo(left,y+3);x.lineTo(right,y+3);x.stroke();x.setLineDash([]);
 ROUTE.forEach((p,i)=>{
  const px=left+(right-left)*p.km/ROUTE_LENGTH;
  x.beginPath();x.arc(px,y,p.type==="mandatory"?5:3,0,Math.PI*2);
  x.fillStyle=p.type==="mandatory"?"#49e0a0":p.type==="optional"?"#ffbf45":"#d6e5f5";x.fill();
  if(i%2===0||p.type==="mandatory"||p.type==="optional"){
   x.save();x.translate(px,y+18);x.rotate(-Math.PI/4);
   x.fillStyle="#d6e5f5";x.font="10px sans-serif";x.fillText(p.name,0,0);x.restore();
  }
 });
 const p=left+(right-left)*s.distance/ROUTE_LENGTH;
 x.fillStyle="#fff";x.beginPath();x.arc(p,y,7,0,Math.PI*2);x.fill();
 x.strokeStyle="#49e0a0";x.lineWidth=3;x.stroke();
 x.fillStyle="#8fbaff";x.font="12px sans-serif";x.fillText("경부고속도로",left,18);
}

function render(){
 const p=currentStop();
 $("speed").textContent=Math.round(s.speed);
 $("distance").textContent=s.distance.toFixed(1);
 $("passengers").textContent=s.passengers;
 $("gearDisplay").textContent=gear();
 $("modeBadge").textContent=s.mode==="auto"?"자동변속":"수동변속";
 $("modeBtn").textContent=s.mode==="auto"?"수동변속으로 전환":"자동변속으로 전환";
 $("clutchState").textContent=s.clutch?"밟음":"해제";
 $("signalState").textContent=s.hazard?"비상등":s.signal==="off"?"꺼짐":s.signal==="left"?"좌측":"우측";
 $("doorState").textContent=s.doorBusy?"문 닫는 중":s.leftDoor?"좌측 문 열림":s.rightDoor?"우측 문 열림":"모든 문 닫힘";
 $("state").textContent=s.finished?"운행 종료":s.paused?"일시정지":s.speed>0?"주행 중":atStop()?p.name+" 정차":"정차";
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
 $("laneDescription").textContent=s.lane===1?"버스전용차로":s.lane===2?"추월 차로":s.lane===3?"일반 주행 차로":"바깥 차로";
 $("cruiseText").textContent=s.cruise===null?"해제":s.cruise+" km/h";
 $("fanText").textContent=s.fan+"단";
 $("gearHelp").textContent=s.mode==="auto"?"자동변속 모드":"클러치를 밟고 변속";
 $("stopInfo").textContent=p.name+": "+(p.type==="mandatory"?"필수 정차":p.type==="optional"?"필요 시 정차":p.type==="terminal"?"종점":p.type==="pass"?"분기점 통과":`승차 ${p.board||0}명 · 하차 ${p.alight||0}명 예정`);
 $("boardBtn").disabled=!atStop()||(!s.leftDoor&&!s.rightDoor);
 $("departBtn").disabled=!atStop()||!doorsClosed()||s.finished;
 $("fuelBtn").disabled=!atStop()||!doorsClosed()||s.h2>=32.999;
 $("chargeBtn").disabled=!atStop()||!doorsClosed()||s.battery>=46.999;
 $("passBtn").disabled=!atStop()||p.type!=="optional"||!doorsClosed();
 $("stopNowBtn").disabled=!atStop()||!isStopPoint(p);
 $("clutchBtn").classList.toggle("active",s.clutch);
 $("sigL").classList.toggle("active",s.signal==="left");
 $("sigR").classList.toggle("active",s.signal==="right");
 $("hazard").classList.toggle("active",s.hazard);
 document.querySelectorAll(".cruise").forEach(b=>b.classList.toggle("active",Number(b.dataset.speed)===s.cruise));
 document.querySelectorAll("#gearButtons button").forEach(b=>b.classList.toggle("active",b.textContent===gear()));
 renderRoute();drawRoad();drawMirror("mirrorL");drawMirror("mirrorR");drawMap();
}

function bindHold(id,key){
 const el=$(id);
 el.addEventListener("pointerdown",e=>{
  e.preventDefault();pressed.add(key);
  if(el.setPointerCapture)el.setPointerCapture(e.pointerId);
 });
 const release=()=>pressed.delete(key);
 el.addEventListener("pointerup",release);
 el.addEventListener("pointercancel",release);
 el.addEventListener("lostpointercapture",release);
 el.addEventListener("contextmenu",e=>e.preventDefault());
}
bindHold("accelerate","ArrowUp");
bindHold("brake","ArrowDown");
bindHold("left","ArrowLeft");
bindHold("right","ArrowRight");

$("modeBtn").onclick=toggleMode;
$("pauseBtn").onclick=()=>{
 s.paused=!s.paused;
 $("pauseBtn").textContent=s.paused?"운행 재개":"일시정지";
 notify(s.paused?"일시정지":"운행 재개");render();
};
$("resetBtn").onclick=()=>{
 clearTimeout(s.doorTimer);pressed.clear();s=freshState();steerCooldown=0;
 $("pauseBtn").textContent="일시정지";$("log").innerHTML="";
 notify("초기화 완료. 경부고속도로 서울반포 출발 · 정원 27석.");
 render();
};
$("clutchBtn").onclick=()=>setClutch(!s.clutch);
$("sigL").onclick=()=>setSignal("left");
$("sigR").onclick=()=>setSignal("right");
$("hazard").onclick=()=>{s.hazard=!s.hazard;s.signal="off";render()};
$("horn").onpointerdown=()=>notify("경적 작동");
$("center").onclick=()=>{s.steer=0;render()};
$("openL").onclick=()=>setDoor("left",true);
$("closeL").onclick=()=>setDoor("left",false);
$("openR").onclick=()=>setDoor("right",true);
$("closeR").onclick=()=>setDoor("right",false);
$("boardBtn").onclick=serviceStop;
$("departBtn").onclick=depart;
$("stopNowBtn").onclick=stopNow;
$("passBtn").onclick=passOptional;
$("fuelBtn").onclick=refuel;
$("chargeBtn").onclick=recharge;
$("cruiseOff").onclick=()=>setCruise(null);
$("testDoorBuzzer").onclick=playDoorBuzzer;
["doorFreq1","doorFreq2","doorVolume"].forEach(id=>$(id).addEventListener("input",updateBuzzerLabels));
document.querySelectorAll(".cruise").forEach(b=>b.onclick=()=>setCruise(Number(b.dataset.speed)));
$("ac").onchange=e=>{s.ac=e.target.checked;if(s.ac){s.heater=false;$("heater").checked=false}render()};
$("heater").onchange=e=>{s.heater=e.target.checked;if(s.heater){s.ac=false;$("ac").checked=false}render()};
$("fan").oninput=e=>{s.fan=Number(e.target.value);render()};
$("clearLog").onclick=()=>{s.log=[];$("log").innerHTML=""};

["1","2","3","4","5","N","R1","R2","R3"].forEach(g=>{
 const b=document.createElement("button");b.textContent=g;b.onclick=()=>shiftGear(g);$("gearButtons").appendChild(b);
});
function keyDown(e){
 if(e.target&&["INPUT","TEXTAREA","SELECT"].includes(e.target.tagName))return;
 if(["ArrowUp","ArrowDown","ArrowLeft","ArrowRight","Space"].includes(e.code))e.preventDefault();
 if(e.repeat)return;
 pressed.add(e.code);
 if(e.code==="ShiftLeft"||e.code==="ShiftRight")setClutch(true);
 if(e.code==="KeyA")setSignal("left");
 if(e.code==="KeyF")setSignal("right");
 if(e.code==="KeyB")notify("경적 작동");
 if(e.code==="KeyO"){
  if(pressed.has("KeyW"))setDoor("left",true);
  if(pressed.has("KeyE"))setDoor("right",true);
 }
 if(e.code==="KeyP"){
  if(pressed.has("KeyW"))setDoor("left",false);
  if(pressed.has("KeyE"))setDoor("right",false);
 }
 if(GEAR_MAP[e.code])shiftGear(GEAR_MAP[e.code]);
}
function keyUp(e){
 pressed.delete(e.code);
 if((e.code==="ShiftLeft"||e.code==="ShiftRight")&&!pressed.has("ShiftLeft")&&!pressed.has("ShiftRight"))setClutch(false);
}
window.addEventListener("keydown",keyDown);
window.addEventListener("keyup",keyUp);
window.addEventListener("blur",()=>{
 pressed.clear();s.steer=0;steerCooldown=0;
 if(s.mode==="manual")s.clutch=false;
});
document.addEventListener("visibilitychange",()=>{
 if(document.hidden){pressed.clear();steerCooldown=0}
});

function frame(now){
 const dt=Math.min((now-last)/1000,.05);last=now;
 update(dt);requestAnimationFrame(frame);
}
updateBuzzerLabels();
log("준비 완료. 경부고속도로 운행 노선이 설정되었습니다.");
log("필수 정차: 판교·천안호두·김천·경주 휴게소.");
log("필요 시 정차: 신탄진·옥천·영동·영천 휴게소.");
log("서울반포 출발: 문 열기 → 승객 승하차 → 문 닫기 → 출발 준비.");
render();requestAnimationFrame(frame);
</script>
</body>
</html>
