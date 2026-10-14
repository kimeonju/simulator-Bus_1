```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>ERR Eco Rapid Rail Simulator</title>
<style>
:root {
  color-scheme: dark;
  --bg: #0b1119;
  --panel: #151f2c;
  --line: #2b3a4d;
  --green: #48e09a;
  --blue: #55baff;
  --text: #eaf2fc;
  --muted: #91a5bc;
}
* { box-sizing: border-box; }
body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
  font-family: system-ui, sans-serif;
}
header {
  padding: 16px 20px;
  background: #101a27;
  border-bottom: 1px solid var(--line);
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}
h1 { font-size: 21px; margin: 0; }
#clock { font-size: 25px; font-weight: 800; color: var(--green); }
main { padding: 16px; max-width: 1500px; margin: auto; }
.grid {
  display: grid;
  grid-template-columns: minmax(0, 1.5fr) minmax(300px, 1fr);
  gap: 14px;
}
.panel {
  background: var(--panel);
  border: 1px solid var(--line);
  border-radius: 12px;
  padding: 15px;
  min-width: 0;
}
h2 { font-size: 17px; margin: 0 0 13px; }
.metrics {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 9px;
  margin-bottom: 14px;
}
.metric {
  background: #0d1621;
  padding: 12px;
  border-radius: 9px;
  border: 1px solid var(--line);
}
.metric span { color: var(--muted); font-size: 12px; display: block; }
.metric strong { display: block; font-size: 24px; margin-top: 5px; }
canvas {
  display: block;
  width: 100%;
  height: 210px;
  background: #09121d;
  border-radius: 8px;
}
.controls {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
}
button, select {
  border: 1px solid #3a4e65;
  background: #223246;
  color: white;
  border-radius: 8px;
  padding: 12px 8px;
  font-size: 14px;
  cursor: pointer;
}
button:hover { background: #30465f; }
button:disabled { opacity: .4; cursor: not-allowed; }
button.green { background: #176c4b; }
button.red { background: #7b3038; }
button.yellow { background: #6e551b; }
.range { width: 100%; accent-color: var(--green); }
.status {
  margin-top: 12px;
  padding: 11px;
  border-radius: 8px;
  background: #0d1621;
  color: var(--green);
  min-height: 43px;
}
.table-wrap { overflow-x: auto; max-height: 430px; }
table { border-collapse: collapse; width: 100%; font-size: 13px; }
th, td {
  padding: 9px 8px;
  border-bottom: 1px solid var(--line);
  text-align: left;
  white-space: nowrap;
}
th { color: var(--muted); position: sticky; top: 0; background: #151f2c; }
tr.active { background: #193b35; }
tr.active td:first-child { color: var(--green); font-weight: 800; }
.small { color: var(--muted); font-size: 12px; }
#log {
  height: 115px;
  overflow: auto;
  background: #0b131d;
  border-radius: 8px;
  padding: 10px;
  font-size: 12px;
  line-height: 1.7;
}
.traffic { font-weight: 800; color: var(--green); }
@media(max-width: 850px) {
  .grid { grid-template-columns: 1fr; }
  .metrics { grid-template-columns: repeat(3, 1fr); }
}
</style>
</head>
<body>
<header>
  <div>
    <h1>🚄 ERR Eco Rapid Rail</h1>
    <div class="small">고속철도 운전 시뮬레이터</div>
  </div>
  <div>
    <div id="clock">05:00:00</div>
    <div id="dateText" class="small">시뮬레이션 시각</div>
  </div>
</header>

<main>
  <div class="metrics">
    <div class="metric"><span>현재 속도</span><strong><span id="speed">0</span> <small>km/h</small></strong></div>
    <div class="metric"><span>주행 거리</span><strong><span id="distance">0.0</span> <small>km</small></strong></div>
    <div class="metric"><span>신호</span><strong id="signal" class="traffic">진행</strong></div>
  </div>

  <div class="grid">
    <section class="panel">
      <h2>운전실 · 주행 정보</h2>
      <canvas id="routeCanvas" width="900" height="250"></canvas>
      <div style="height:12px"></div>
      <div class="metrics">
        <div class="metric"><span>다음 역</span><strong id="nextStation" style="font-size:18px">서울</strong></div>
        <div class="metric"><span>역까지 거리</span><strong id="remaining" style="font-size:18px">0.0 km</strong></div>
        <div class="metric"><span>제한 속도</span><strong style="font-size:18px">300 km/h</strong></div>
      </div>

      <h2>운전 조작</h2>
      <div class="controls">
        <button class="green" id="powerUp">가속 +</button>
        <button id="powerDown">가속 −</button>
        <button class="red" id="brake">제동</button>
        <button id="release">제동 해제</button>
        <button id="hold">정차 유지</button>
        <button id="resume">운행 재개</button>
      </div>
      <p class="small">가속 설정: <b id="powerValue">50%</b></p>
      <input class="range" id="power" type="range" min="0" max="100" value="50">
      <p class="small">시뮬레이션 배속</p>
      <select id="timeScale" style="width:100%">
        <option value="1">1배속</option>
        <option value="5">5배속</option>
        <option value="10">10배속</option>
        <option value="30">30배속</option>
      </select>
      <div id="status" class="status">준비 완료 · 서울역 출발 대기</div>
      <p class="small">※ 실제 철도 운행이 아닌 가상 시뮬레이션입니다.</p>
    </section>

    <section class="panel">
      <h2>전체 운행 시간표</h2>
      <div class="table-wrap">
        <table>
          <thead>
            <tr><th>역</th><th>도착</th><th>출발</th><th>거리</th><th>상태</th></tr>
          </thead>
          <tbody id="timetable"></tbody>
        </table>
      </div>
      <div style="height:18px"></div>
      <h2>운행 열차</h2>
      <div class="table-wrap">
        <table>
          <thead><tr><th>열차</th><th>현재 위치</th><th>속도</th><th>운행 상태</th></tr></thead>
          <tbody id="trainTable"></tbody>
        </table>
      </div>
    </section>
  </div>

  <section class="panel" style="margin-top:14px">
    <h2>운행 기록</h2>
    <div id="log"></div>
  </section>
  <div id="errorBox" style="display:none;margin-top:12px;padding:12px;background:#6d202b;border-radius:8px;white-space:pre-wrap"></div>
</main>

<script>
"use strict";

const stations = [
  {name:"서울", km:0, arr:"-", dep:"05:00"},
  {name:"광명", km:17, arr:"05:20", dep:"05:23"},
  {name:"천안아산", km:96.5, arr:"05:50", dep:"05:53"},
  {name:"오송", km:131.5, arr:"06:10", dep:"06:13"},
  {name:"대전", km:166.3, arr:"06:30", dep:"06:35"},
  {name:"김천(구미)", km:227.3, arr:"07:00", dep:"07:03"},
  {name:"동대구", km:293.1, arr:"07:20", dep:"07:25"},
  {name:"경주", km:346.8, arr:"07:40", dep:"07:45"},
  {name:"태화강", km:380, arr:"08:10", dep:"08:15"},
  {name:"부산센텀", km:417.4, arr:"08:30", dep:"-"}
];

const $ = id => document.getElementById(id);
const canvas = $("routeCanvas");
const ctx = canvas.getContext("2d");

let simSeconds = 5 * 3600;
let lastFrame = performance.now();
let speed = 0;
let km = 0;
let power = 50;
let brakeOn = false;
let held = false;
let completed = false;
let lastLogMinute = -1;
let aiTrains = [];

function timeString(totalSeconds) {
  totalSeconds = Math.floor(totalSeconds);
  const h = Math.floor(totalSeconds / 3600) % 24;
  const m = Math.floor(totalSeconds / 60) % 60;
  const s = totalSeconds % 60;
  return [h,m,s].map(v => String(v).padStart(2,"0")).join(":");
}
function minutesString(value) {
  if (!value || value === "-") return "-";
  return value;
}
function log(message) {
  const line = document.createElement("div");
  line.textContent = "[" + timeString(simSeconds) + "] " + message;
  $("log").prepend(line);
}
function stationIndex() {
  let index = 0;
  for (let i=0; i<stations.length; i++) {
    if (stations[i].km <= km + 0.001) index = i;
  }
  return index;
}
function nextStationIndex() {
  for (let i=0; i<stations.length; i++) {
    if (stations[i].km > km + 0.05) return i;
  }
  return stations.length - 1;
}
function clockMinutes(str) {
  if (!str || str === "-") return null;
  const [h,m] = str.split(":").map(Number);
  return h*60+m;
}
function renderTimetable() {
  const current = simSeconds / 60;
  $("timetable").innerHTML = stations.map((s,i) => {
    const arr = clockMinutes(s.arr);
    const dep = clockMinutes(s.dep);
    let state = "예정";
    if (i < stationIndex()) state = "통과";
    if (i === stationIndex() && !completed) state = "현재 위치";
    if (completed) state = "운행 완료";
    const active = i === stationIndex() ? "active" : "";
    return `<tr class="${active}">
      <td>${s.name}</td><td>${s.arr}</td><td>${s.dep}</td>
      <td>${s.km.toFixed(1)} km</td><td>${state}</td>
    </tr>`;
  }).join("");
}
function initAI() {
  aiTrains = [];
  for (let i=1; i<=8; i++) {
    aiTrains.push({
      id:"ERR" + String(i).padStart(3,"0"),
      km: Math.max(0, Math.min(417, i*42)),
      speed: 160 + (i%3)*20,
      delay: 0
    });
  }
}
function renderTrains() {
  const trains = [{
    id:"PLAYER",
    km,
    speed,
    state:completed ? "到着" : held ? "停車保持" : brakeOn ? "制動中" : "운행 중"
  }, ...aiTrains.map(t => ({
    id:t.id, km:t.km, speed:t.speed, state:"운행 중"
  }))];
  $("trainTable").innerHTML = trains.map(t => {
    const idx = stations.reduce((best,s,i) =>
      Math.abs(s.km-t.km) < Math.abs(stations[best].km-t.km) ? i : best,0);
    return `<tr><td>${t.id}</td><td>${stations[idx].name}</td>
      <td>${Math.round(t.speed)} km/h</td><td>${t.state}</td></tr>`;
  }).join("");
}
function updateAI(dt) {
  for (const t of aiTrains) {
    const ahead = t.km > km && t.km-km < 1.5;
    let target = 230;
    if (ahead) target = 0;
    t.speed += Math.max(-80, Math.min(25, target-t.speed))*dt;
    t.km += t.speed/3600*dt;
    if (t.km > 417.4) t.km = 0;
  }
}
function drawRoute() {
  const w = canvas.width, h = canvas.height;
  ctx.clearRect(0,0,w,h);
  ctx.fillStyle = "#09121d";
  ctx.fillRect(0,0,w,h);

  const left=38, right=w-28, y=112;
  ctx.strokeStyle="#32485e";
  ctx.lineWidth=5;
  ctx.beginPath(); ctx.moveTo(left,y); ctx.lineTo(right,y); ctx.stroke();

  for (const s of stations) {
    const x=left+(s.km/417.4)*(right-left);
    ctx.fillStyle="#91a5bc";
    ctx.beginPath(); ctx.arc(x,y,5,0,Math.PI*2); ctx.fill();
    ctx.fillStyle="#dce9f8";
    ctx.font="13px system-ui";
    ctx.textAlign="center";
    ctx.fillText(s.name,x,y+25);
    ctx.fillStyle="#6e849d";
    ctx.font="11px system-ui";
    ctx.fillText(s.km.toFixed(0)+" km",x,y+41);
  }
  for (const t of aiTrains) {
    const x=left+(t.km/417.4)*(right-left);
    ctx.fillStyle="#ffbd4a";
    ctx.beginPath(); ctx.arc(x,y,7,0,Math.PI*2); ctx.fill();
  }
  const px=left+(km/417.4)*(right-left);
  ctx.fillStyle="#48e09a";
  ctx.beginPath();
  ctx.moveTo(px,y-15); ctx.lineTo(px-9,y-30); ctx.lineTo(px+9,y-30);
  ctx.closePath(); ctx.fill();
  ctx.fillStyle="#48e09a";
  ctx.textAlign="center";
  ctx.font="bold 13px system-ui";
  ctx.fillText("PLAYER",px,y-38);

  ctx.textAlign="left";
  ctx.fillStyle="#91a5bc";
  ctx.font="12px system-ui";
  ctx.fillText("서울",left,25);
  ctx.textAlign="right";
  ctx.fillText("부산센텀",right,25);
  ctx.fillStyle="#1b2a3b";
  ctx.fillRect(left,190,right-left,12);
  ctx.fillStyle="#48e09a";
  ctx.fillRect(left,190,(right-left)*(km/417.4),12);
  ctx.fillStyle="#dce9f8";
  ctx.textAlign="left";
  ctx.fillText("노선 진행률 "+(km/417.4*100).toFixed(1)+"%",left,220);
}
function updateUI() {
  $("clock").textContent = timeString(simSeconds);
  $("dateText").textContent = "시뮬레이션 시각 · 24시간제";
  $("speed").textContent = Math.round(speed);
  $("distance").textContent = km.toFixed(1);

  const ni = nextStationIndex();
  const next = stations[ni];
  $("nextStation").textContent = next.name;
  $("remaining").textContent = Math.max(0,next.km-km).toFixed(1)+" km";

  let signal = "진행";
  let signalColor = "#48e09a";
  const nearest = aiTrains.filter(t=>t.km>km).sort((a,b)=>a.km-b.km)[0];
  if (nearest && nearest.km-km < 2) {
    signal="주의";
    signalColor="#ffbd4a";
  }
  if (nearest && nearest.km-km < 0.6) {
    signal="정지";
    signalColor="#ff646e";
  }
  $("signal").textContent = signal;
  $("signal").style.color = signalColor;

  $("powerValue").textContent = power+"%";
  $("power").value = power;

  if (completed) $("status").textContent="부산센텀 도착 · 운행 완료";
  else if (held) $("status").textContent="정차 유지 중";
  else if (brakeOn) $("status").textContent="제동 작동 중";
  else $("status").textContent="운행 중 · "+signal+" 신호";

  renderTimetable();
  renderTrains();
  drawRoute();
}
function frame(now) {
  try {
    const realDt = Math.min((now-lastFrame)/1000,0.1);
    lastFrame = now;
    const scale = Number($("timeScale").value);
    const dt = realDt*scale;

    if (!held && !completed) {
      simSeconds += dt;
      const ni = nextStationIndex();
      const next = stations[ni];
      const remaining = next.km-km;

      let targetSpeed = power/100*300;
      const safeBrakeDistance = speed*speed/(2*45*3.6)+0.2;
      if (remaining < safeBrakeDistance+0.5) {
        targetSpeed = Math.min(targetSpeed,Math.sqrt(Math.max(0,2*45*remaining))*3.6);
      }
      if (brakeOn) targetSpeed=0;

      const nearest = aiTrains.filter(t=>t.km>km).sort((a,b)=>a.km-b.km)[0];
      if (nearest && nearest.km-km < 1.2) targetSpeed=0;

      const accel = brakeOn ? 90 : 18;
      speed += Math.max(-accel*dt,Math.min(accel*dt,targetSpeed-speed));
      speed = Math.max(0,Math.min(300,speed));
      km += speed/3600*dt;

      if (km >= next.km) {
        km=next.km;
        speed=0;
        if (ni===stations.length-1) {
          completed=true;
          log(next.name+" 도착. 운행 완료");
        } else {
          held=true;
          log(next.name+" 역 정차");
          setTimeout(()=>{
            if (!completed) {
              held=false;
              log(next.name+" 역 출발");
            }
          }, 1800);
        }
      }
      updateAI(dt);
      const minute = Math.floor(simSeconds/60);
      if (minute!==lastLogMinute) lastLogMinute=minute;
    }
    updateUI();
  } catch (e) {
    $("errorBox").style.display="block";
    $("errorBox").textContent="실행 오류: "+e.message+"\n"+e.stack;
  }
  requestAnimationFrame(frame);
}

$("powerUp").addEventListener("click",()=>{power=Math.min(100,power+10);});
$("powerDown").addEventListener("click",()=>{power=Math.max(0,power-10);});
$("brake").addEventListener("click",()=>{brakeOn=true;log("제동 작동");});
$("release").addEventListener("click",()=>{brakeOn=false;log("제동 해제");});
$("hold").addEventListener("click",()=>{held=true;speed=0;log("정차 유지");});
$("resume").addEventListener("click",()=>{held=false;log("운행 재개");});
$("power").addEventListener("input",e=>{power=Number(e.target.value);});
$("timeScale").addEventListener("change",()=>log("배속 변경"));

window.addEventListener("resize",()=>{
  const ratio=window.devicePixelRatio||1;
  const rect=canvas.getBoundingClientRect();
  canvas.width=Math.max(300,Math.floor(rect.width*ratio));
  canvas.height=Math.floor(210*ratio);
  ctx.setTransform(ratio,0,0,ratio,0,0);
});

initAI();
log("ERR Eco Rapid Rail 시뮬레이터 준비 완료");
updateUI();
requestAnimationFrame(frame);
</script>
</body>
</html>
```
