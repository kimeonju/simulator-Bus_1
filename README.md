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
  --panel: #141e2a;
  --panel2: #1b2938;
  --line: #304356;
  --text: #edf5ff;
  --muted: #9fb1c5;
  --green: #30d878;
  --red: #ff5353;
  --yellow: #ffd34d;
  --blue: #48a8ff;
}
* { box-sizing: border-box; }
body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
}
header {
  padding: 18px;
  background: #101a25;
  border-bottom: 1px solid var(--line);
}
h1 { font-size: 23px; margin: 0 0 8px; }
h2 { font-size: 17px; margin: 0 0 14px; }
p { color: var(--muted); }
main { padding: 14px; max-width: 1500px; margin: auto; }
.grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px;
}
.panel {
  background: var(--panel);
  border: 1px solid var(--line);
  border-radius: 12px;
  padding: 16px;
  min-width: 0;
}
.wide { grid-column: 1 / -1; }
.clock {
  font-size: clamp(30px, 6vw, 52px);
  font-weight: 800;
  font-variant-numeric: tabular-nums;
  letter-spacing: 1px;
}
.date { color: var(--muted); margin-top: 6px; }
.status {
  display: inline-block;
  padding: 5px 10px;
  border-radius: 6px;
  color: #07110b;
  font-weight: 800;
  font-size: 12px;
}
.status.running { background: var(--green); }
.status.completed { background: var(--red); }
.status.ready { background: var(--yellow); }
.status.waiting { background: var(--yellow); }
.statline {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-top: 12px;
}
.stat {
  flex: 1;
  min-width: 90px;
  background: var(--panel2);
  padding: 12px;
  border-radius: 8px;
}
.stat strong {
  display: block;
  font-size: 24px;
  margin-top: 5px;
}
.muted { color: var(--muted); font-size: 13px; }
button, select, input {
  color: var(--text);
  background: #26394d;
  border: 1px solid #48617a;
  border-radius: 7px;
  padding: 10px 12px;
  font: inherit;
}
button { cursor: pointer; }
button:hover { background: #35516c; }
button:disabled { opacity: .4; cursor: not-allowed; }
.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 12px;
}
.tablewrap { overflow-x: auto; }
table {
  width: 100%;
  border-collapse: collapse;
  min-width: 680px;
  font-size: 13px;
}
th, td {
  border-bottom: 1px solid var(--line);
  text-align: left;
  padding: 10px 8px;
  white-space: nowrap;
}
th { color: var(--muted); background: #172432; }
tr.selected { background: #21364a; }
.trainrow { cursor: pointer; }
.trainrow:hover { background: #1c3043; }
.route {
  display: flex;
  align-items: center;
  gap: 6px;
  overflow-x: auto;
  padding: 18px 4px 12px;
}
.station {
  flex: 0 0 auto;
  min-width: 74px;
  text-align: center;
  font-size: 12px;
}
.station .dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #71859a;
  margin: 0 auto 8px;
  border: 2px solid #d7e5f5;
}
.station.passed .dot { background: var(--green); }
.station.current .dot {
  background: var(--yellow);
  box-shadow: 0 0 12px var(--yellow);
}
.track {
  width: 35px;
  height: 3px;
  flex: 0 0 35px;
  background: #526a80;
}
.track.passed { background: var(--green); }
.progress {
  height: 9px;
  background: #29394a;
  border-radius: 8px;
  overflow: hidden;
  margin-top: 12px;
}
.progress > div {
  height: 100%;
  width: 0;
  background: var(--green);
  transition: width .3s;
}
canvas {
  width: 100%;
  height: 230px;
  display: block;
  background: #09131d;
  border: 1px solid var(--line);
  border-radius: 8px;
}
.notice {
  color: var(--muted);
  font-size: 12px;
  line-height: 1.6;
  margin-top: 12px;
}
footer {
  padding: 20px;
  text-align: center;
  color: var(--muted);
  font-size: 12px;
}
@media (max-width: 760px) {
  .grid { grid-template-columns: 1fr; }
  .wide { grid-column: auto; }
  header { padding: 14px; }
  main { padding: 9px; }
  .panel { padding: 12px; }
}
</style>
</head>
<body>
<header>
  <h1>🚄 ERR Eco Rapid Rail Simulator</h1>
  <div class="muted">컴퓨터 시각 연동 · 열차 운행 현황 · 운전실</div>
</header>

<main>
<div class="grid">

  <section class="panel">
    <h2>🕒 컴퓨터 실시간 시계</h2>
    <div class="clock" id="clock">--:--:--</div>
    <div class="date" id="date">날짜 확인 중</div>
    <div class="statline">
      <div class="stat">
        <span class="muted">운행준비</span>
        <strong id="readyCount" style="color:var(--yellow)">0</strong>
      </div>
      <div class="stat">
        <span class="muted">운행중</span>
        <strong id="runningCount" style="color:var(--green)">0</strong>
      </div>
      <div class="stat">
        <span class="muted">운행완료</span>
        <strong id="completedCount" style="color:var(--red)">0</strong>
      </div>
    </div>
  </section>

  <section class="panel">
    <h2>🚦 선택 열차 운전실</h2>
    <div class="muted">열차 번호</div>
    <div id="cabTrain" style="font-size:27px;font-weight:800;margin:5px 0 12px">
      ERR 0500
    </div>
    <div id="cabStatus"><span class="status ready">운행준비</span></div>
    <div class="statline">
      <div class="stat">
        <span class="muted">현재 위치</span>
        <strong id="cabLocation" style="font-size:18px">서울</strong>
      </div>
      <div class="stat">
        <span class="muted">속도</span>
        <strong id="cabSpeed">0 km/h</strong>
      </div>
      <div class="stat">
        <span class="muted">운행률</span>
        <strong id="cabProgress">0%</strong>
      </div>
    </div>
    <div class="progress"><div id="progressBar"></div></div>
    <div class="controls">
      <button id="prevTrain">◀ 이전 열차</button>
      <button id="nextTrain">다음 열차 ▶</button>
      <button id="syncButton">현재 시각 동기화</button>
    </div>
    <div class="notice">
      열차의 속도와 위치는 시간표에 기록된 구간별 시간으로 계산합니다.
      이 화면은 시뮬레이션이며 실제 철도 운행 정보를 제공하지 않습니다.
    </div>
  </section>

  <section class="panel wide">
    <h2>🗺️ 노선도 및 열차 위치</h2>
    <div class="route" id="route"></div>
    <canvas id="railCanvas" width="1200" height="230"></canvas>
    <div class="notice">
      초록색: 이미 통과한 구간 · 노란색: 현재 열차 위치 · 회색: 다음 구간
    </div>
  </section>

  <section class="panel wide">
    <h2>📋 전체 열차 운행 현황</h2>
    <div class="controls">
      <label>
        운행 상태
        <select id="statusFilter">
          <option value="all">전체 열차</option>
          <option value="ready">운행준비</option>
          <option value="running">운행중</option>
          <option value="completed">운행완료</option>
        </select>
      </label>
      <label>
        열차 검색
        <input id="trainSearch" placeholder="열차 번호 검색">
      </label>
    </div>
    <div class="tablewrap" style="margin-top:12px">
      <table>
        <thead>
          <tr>
            <th>열차 번호</th>
            <th>출발 시각</th>
            <th>도착 예정</th>
            <th>현재 위치</th>
            <th>속도</th>
            <th>운행 상태</th>
            <th>운행률</th>
          </tr>
        </thead>
        <tbody id="trainTable">
          <tr><td colspan="7">열차 정보를 불러오는 중...</td></tr>
        </tbody>
      </table>
    </div>
  </section>

  <section class="panel wide">
    <h2>🕓 주요 역 시간표</h2>
    <div class="tablewrap">
      <table>
        <thead>
          <tr>
            <th>역명</th>
            <th>거리</th>
            <th>기준 도착</th>
            <th>기준 출발</th>
            <th>선택 열차 도착</th>
            <th>선택 열차 출발</th>
            <th>상태</th>
          </tr>
        </thead>
        <tbody id="timetable">
          <tr><td colspan="7">시간표를 불러오는 중...</td></tr>
        </tbody>
      </table>
    </div>
    <div class="notice">
      기준 시간표는 05:00 서울 출발, 08:30 부산 센텀 도착입니다.
      후속 열차는 20분 간격으로 편성됩니다. 날짜가 바뀌어도 현재 컴퓨터
      시각으로 운행 상태를 계산합니다.
    </div>
  </section>

</div>
</main>

<footer>ERR Eco Rapid Rail Simulator · Local Computer Clock</footer>

<script>
(function () {
"use strict";

const stations = [
  { name: "서울", km: 0,       arr: null,  dep: "05:00" },
  { name: "광명", km: 17,      arr: "05:20", dep: "05:23" },
  { name: "천안아산", km: 96.5, arr: "05:50", dep: "05:53" },
  { name: "오송", km: 131.5,   arr: "06:10", dep: "06:13" },
  { name: "대전", km: 166.3,   arr: "06:30", dep: "06:35" },
  { name: "김천(구미)", km: 227.3, arr: "07:00", dep: "07:03" },
  { name: "동대구", km: 293.1, arr: "07:20", dep: "07:25" },
  { name: "경주", km: 346.8,  arr: "07:40", dep: "07:45" },
  { name: "태화강", km: 380,  arr: "08:10", dep: "08:15" },
  { name: "부산 센텀", km: 417.4, arr: "08:30", dep: null }
];

const baseDeparture = 5 * 60;
const baseArrival = 8 * 60 + 30;
const routeMinutes = baseArrival - baseDeparture;
const intervalMinutes = 20;
const firstDeparture = 5 * 60;
const lastDeparture = 23 * 60;

let trains = [];
let selectedTrainId = 0;
let lastMinute = -1;

function timeToMinutes(time) {
  if (!time) return null;
  const parts = time.split(":").map(Number);
  return parts[0] * 60 + parts[1];
}

function formatTime(totalMinutes) {
  if (totalMinutes == null) return "—";
  const normalized = ((totalMinutes % 1440) + 1440) % 1440;
  const hours = Math.floor(normalized / 60);
  const minutes = normalized % 60;
  return String(hours).padStart(2, "0") + ":" +
         String(minutes).padStart(2, "0");
}

function formatClock(date) {
  return [date.getHours(), date.getMinutes(), date.getSeconds()]
    .map(n => String(n).padStart(2, "0")).join(":");
}

function makeTrains() {
  trains = [];
  let id = 0;

  for (let dep = firstDeparture; dep <= lastDeparture; dep += intervalMinutes) {
    trains.push({
      id: id++,
      number: "ERR " + formatTime(dep).replace(":", ""),
      departure: dep,
      arrival: dep + routeMinutes
    });
  }
  selectedTrainId = 0;
}

function getNowMinutes(date) {
  return date.getHours() * 60 + date.getMinutes() +
         date.getSeconds() / 60;
}

function stationArrival(train, index) {
  if (index === 0) return null;
  return train.departure +
    (timeToMinutes(stations[index].arr) - baseDeparture);
}

function stationDeparture(train, index) {
  if (index === stations.length - 1) return null;
  return train.departure +
    (timeToMinutes(stations[index].dep) - baseDeparture);
}

function getTrainState(train, now) {
  const t = getNowMinutes(now);
  const elapsed = t - train.departure;
  const duration = train.arrival - train.departure;

  if (elapsed < 0) {
    return {
      state: "ready",
      stateText: "운행준비",
      progress: 0,
      km: 0,
      speed: 0,
      location: "서울",
      segment: 0
    };
  }

  if (elapsed >= duration) {
    return {
      state: "completed",
      stateText: "운행완료",
      progress: 100,
      km: stations[stations.length - 1].km,
      speed: 0,
      location: "부산 센텀",
      segment: stations.length - 1
    };
  }

  const elapsedWhole = elapsed;
  const progress = Math.max(0, Math.min(100, elapsedWhole / duration * 100));

  // 시간표의 역 도착·출발 시각 사이에서 현재 위치를 계산합니다.
  for (let i = 0; i < stations.length - 1; i++) {
    const depart = stationDeparture(train, i);
    const arriveNext = stationArrival(train, i + 1);
    const arriveHere = stationArrival(train, i);
    const stopEnd = depart == null ? train.departure : depart;
    const segmentStart = i === 0 ? train.departure : stopEnd;

    if (i > 0 && arriveHere != null &&
        elapsedWhole >= arriveHere - train.departure &&
        elapsedWhole < stopEnd - train.departure) {
      return {
        state: "running",
        stateText: "운행중",
        progress,
        km: stations[i].km,
        speed: 0,
        location: stations[i].name + " 정차",
        segment: i
      };
    }

    const from = segmentStart - train.departure;
    const to = arriveNext - train.departure;

    if (elapsedWhole >= from && elapsedWhole < to) {
      const fraction = Math.max(0, Math.min(1, (elapsedWhole - from) / (to - from)));
      const km = stations[i].km +
        (stations[i + 1].km - stations[i].km) * fraction;
      const speed = (stations[i + 1].km - stations[i].km) /
        ((to - from) / 60);

      return {
        state: "running",
        stateText: "운행중",
        progress,
        km,
        speed: Math.max(0, Math.round(speed)),
        location: stations[i].name + "–" + stations[i + 1].name,
        segment: i
      };
    }
  }

  return {
    state: "running",
    stateText: "운행중",
    progress,
    km: 0,
    speed: 0,
    location: "운행 중",
    segment: 0
  };
}

function getSelectedTrain() {
  return trains.find(t => t.id === selectedTrainId) || trains[0];
}

function statusBadge(state, text) {
  return '<span class="status ' + state + '">' + text + '</span>';
}

function renderClock(now) {
  document.getElementById("clock").textContent = formatClock(now);
  document.getElementById("date").textContent =
    now.getFullYear() + "년 " +
    (now.getMonth() + 1) + "월 " +
    now.getDate() + "일 · 컴퓨터 현지 시각";
}

function renderStats(now) {
  let ready = 0, running = 0, completed = 0;
  trains.forEach(train => {
    const state = getTrainState(train, now).state;
    if (state === "ready") ready++;
    else if (state === "running") running++;
    else completed++;
  });
  document.getElementById("readyCount").textContent = ready;
  document.getElementById("runningCount").textContent = running;
  document.getElementById("completedCount").textContent = completed;
}

function renderCab(now) {
  const train = getSelectedTrain();
  const info = getTrainState(train, now);

  document.getElementById("cabTrain").textContent = train.number;
  document.getElementById("cabStatus").innerHTML =
    statusBadge(info.state, info.stateText);
  document.getElementById("cabLocation").textContent =
    info.location + (info.state === "running" ? " · " +
      info.km.toFixed(1) + " km" : "");
  document.getElementById("cabSpeed").textContent =
    info.speed + " km/h";
  document.getElementById("cabProgress").textContent =
    Math.round(info.progress) + "%";
  document.getElementById("progressBar").style.width =
    info.progress + "%";
}

function renderRoute(now) {
  const train = getSelectedTrain();
  const info = getTrainState(train, now);
  const route = document.getElementById("route");
  route.innerHTML = "";

  stations.forEach((station, i) => {
    const node = document.createElement("div");
    node.className = "station";
    if (station.km < info.km) node.classList.add("passed");
    if (i === info.segment && info.state === "running")
      node.classList.add("current");
    if (info.state === "completed" && i === stations.length - 1)
      node.classList.add("passed");

    node.innerHTML =
      '<div class="dot"></div><div>' + station.name +
      '</div><div class="muted">' + station.km + ' km</div>';
    route.appendChild(node);

    if (i < stations.length - 1) {
      const track = document.createElement("div");
      track.className = "track";
      if (station.km < info.km) track.classList.add("passed");
      route.appendChild(track);
    }
  });

  drawCanvas(now);
}

function drawCanvas(now) {
  const canvas = document.getElementById("railCanvas");
  const ctx = canvas.getContext("2d");
  const w = canvas.width, h = canvas.height;
  ctx.clearRect(0, 0, w, h);

  ctx.fillStyle = "#09131d";
  ctx.fillRect(0, 0, w, h);

  const left = 45, right = w - 45, y = 105;
  const width = right - left;
  const xForKm = km => left + km / 417.4 * width;

  ctx.strokeStyle = "#526a80";
  ctx.lineWidth = 5;
  ctx.beginPath();
  ctx.moveTo(left, y);
  ctx.lineTo(right, y);
  ctx.stroke();

  stations.forEach((station, i) => {
    const x = xForKm(station.km);
    ctx.fillStyle = "#dce9f5";
    ctx.beginPath();
    ctx.arc(x, y, 6, 0, Math.PI * 2);
    ctx.fill();

    ctx.fillStyle = "#dce9f5";
    ctx.font = "15px sans-serif";
    ctx.textAlign = "center";
    ctx.fillText(station.name, x, y - 20);
    ctx.fillStyle = "#91a8be";
    ctx.font = "12px sans-serif";
    ctx.fillText(station.km + " km", x, y + 24);
  });

  trains.forEach(train => {
    const info = getTrainState(train, now);
    if (info.state === "ready") return;

    const x = xForKm(info.km);
    const isSelected = train.id === selectedTrainId;
    ctx.fillStyle = info.state === "completed" ? "#ff5353" : "#30d878";
    ctx.beginPath();
    ctx.arc(x, isSelected ? 63 : 151, isSelected ? 8 : 4, 0, Math.PI * 2);
    ctx.fill();

    if (isSelected) {
      ctx.fillStyle = "#ffffff";
      ctx.font = "bold 14px sans-serif";
      ctx.textAlign = "center";
      ctx.fillText(train.number, x, 43);
    }
  });

  ctx.fillStyle = "#9fb1c5";
  ctx.font = "13px sans-serif";
  ctx.textAlign = "left";
  ctx.fillText("서울", left, 205);
  ctx.textAlign = "right";
  ctx.fillText("부산 센텀", right, 205);
}

function renderTrains(now) {
  const tbody = document.getElementById("trainTable");
  const filter = document.getElementById("statusFilter").value;
  const search = document.getElementById("trainSearch").value.trim().toLowerCase();

  const rows = trains.filter(train => {
    const info = getTrainState(train, now);
    const matchStatus = filter === "all" || filter === info.state;
    const matchSearch = train.number.toLowerCase().includes(search);
    return matchStatus && matchSearch;
  });

  if (!rows.length) {
    tbody.innerHTML = '<tr><td colspan="7">조건에 맞는 열차가 없습니다.</td></tr>';
    return;
  }

  tbody.innerHTML = rows.map(train => {
    const info = getTrainState(train, now);
    return '<tr class="trainrow ' +
      (train.id === selectedTrainId ? 'selected' : '') +
      '" data-id="' + train.id + '">' +
      '<td><strong>' + train.number + '</strong></td>' +
      '<td>' + formatTime(train.departure) + '</td>' +
      '<td>' + formatTime(train.arrival) + '</td>' +
      '<td>' + info.location + '</td>' +
      '<td>' + info.speed + ' km/h</td>' +
      '<td>' + statusBadge(info.state, info.stateText) + '</td>' +
      '<td>' + Math.round(info.progress) + '%</td>' +
      '</tr>';
  }).join("");

  tbody.querySelectorAll("tr[data-id]").forEach(row => {
    row.addEventListener("click", () => {
      selectedTrainId = Number(row.dataset.id);
      renderAll(new Date());
    });
  });
}

function renderTimetable(now) {
  const train = getSelectedTrain();
  const info = getTrainState(train, now);
  const tbody = document.getElementById("timetable");

  tbody.innerHTML = stations.map((station, i) => {
    const arr = stationArrival(train, i);
    const dep = stationDeparture(train, i);
    let stationState = "운행준비";
    let css = "ready";
    const nowMinutes = getNowMinutes(now);

    if (info.state === "completed") {
      stationState = "운행완료";
      css = "completed";
    } else if (info.state === "running") {
      const stationArr = arr == null ? train.departure : arr;
      const stationDep = dep == null ? train.arrival : dep;

      if (nowMinutes >= stationDep) {
        stationState = "통과/출발";
        css = "running";
      } else if (nowMinutes >= stationArr) {
        stationState = "정차";
        css = "running";
      } else {
        stationState = "운행중";
        css = "running";
      }
    }

    return '<tr>' +
      '<td><strong>' + station.name + '</strong></td>' +
      '<td>' + station.km + ' km</td>' +
      '<td>' + (arr == null ? '—' : formatTime(arr)) + '</td>' +
      '<td>' + (dep == null ? '—' : formatTime(dep)) + '</td>' +
      '<td>' + (arr == null ? '—' : formatTime(arr)) + '</td>' +
      '<td>' + (dep == null ? '—' : formatTime(dep)) + '</td>' +
      '<td>' + statusBadge(css, stationState) + '</td>' +
      '</tr>';
  }).join("");
}

function renderAll(now) {
  renderClock(now);
  renderStats(now);
  renderCab(now);
  renderRoute(now);
  renderTrains(now);
  renderTimetable(now);
}

document.getElementById("prevTrain").addEventListener("click", () => {
  const index = trains.findIndex(t => t.id === selectedTrainId);
  selectedTrainId = trains[(index - 1 + trains.length) % trains.length].id;
  renderAll(new Date());
});

document.getElementById("nextTrain").addEventListener("click", () => {
  const index = trains.findIndex(t => t.id === selectedTrainId);
  selectedTrainId = trains[(index + 1) % trains.length].id;
  renderAll(new Date());
});

document.getElementById("syncButton").addEventListener("click", () => {
  renderAll(new Date());
});

document.getElementById("statusFilter").addEventListener("change", () => {
  renderTrains(new Date());
});

document.getElementById("trainSearch").addEventListener("input", () => {
  renderTrains(new Date());
});

makeTrains();

// 시작 시각에 가장 가까운 운행 열차를 선택합니다.
function selectCurrentTrain(now) {
  const t = getNowMinutes(now);
  let best = trains[0];
  let bestDistance = Infinity;

  trains.forEach(train => {
    const info = getTrainState(train, now);
    const distance = info.state === "running"
      ? 0
      : Math.abs(train.departure - t);
    if (distance < bestDistance) {
      bestDistance = distance;
      best = train;
    }
  });

  selectedTrainId = best.id;
}

selectCurrentTrain(new Date());
renderAll(new Date());

// 컴퓨터의 현재 시각을 매초 다시 읽어 모든 열차 상태를 갱신합니다.
setInterval(() => {
  const now = new Date();
  renderAll(now);
}, 1000);

})();
</script>
</body>
</html>
