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
  --line: #304155;
  --green: #48e09a;
  --text: #edf4fc;
  --muted: #9cafc4;
}
* { box-sizing: border-box; }
body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
}
header {
  padding: 16px;
  background: #101a27;
  border-bottom: 1px solid var(--line);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
}
h1 { margin: 0; font-size: 21px; }
h2 { font-size: 17px; margin: 0 0 14px; }
#clock { font-size: 27px; font-weight: 800; color: var(--green); }
main { max-width: 1450px; margin: auto; padding: 14px; }
.grid {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  gap: 14px;
}
.panel {
  min-width: 0;
  background: var(--panel);
  border: 1px solid var(--line);
  border-radius: 12px;
  padding: 15px;
  margin-bottom: 14px;
}
.metrics {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 8px;
  margin-bottom: 14px;
}
.metric {
  min-width: 0;
  background: #0c1622;
  border: 1px solid var(--line);
  padding: 11px;
  border-radius: 9px;
}
.metric span { color: var(--muted); font-size: 12px; }
.metric strong {
  display: block;
  margin-top: 5px;
  font-size: 23px;
  overflow-wrap: anywhere;
}
canvas {
  display: block;
  width: 100%;
  height: 210px;
  background: #09121d;
  border-radius: 8px;
  margin-bottom: 14px;
}
.controls {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 8px;
}
button, select {
  width: 100%;
  padding: 12px 6px;
  color: white;
  background: #25374b;
  border: 1px solid #40556d;
  border-radius: 8px;
  font-size: 14px;
  cursor: pointer;
}
button:active { background: #3c5874; }
.green { background: #176b4b; }
.red { background: #7b3038; }
.range { width: 100%; accent-color: var(--green); }
.status {
  margin-top: 12px;
  padding: 12px;
  border-radius: 8px;
  background: #0b1721;
  color: var(--green);
}
.small { color: var(--muted); font-size: 12px; }
.table-wrap { overflow: auto; max-height: 390px; }
table { width: 100%; border-collapse: collapse; font-size: 13px; }
th, td {
  padding: 9px 7px;
  text-align: left;
  border-bottom: 1px solid var(--line);
  white-space: nowrap;
}
th { color: var(--muted); background: #151f2c; }
tr.active { background: #194034; }
#log {
  max-height: 130px;
  overflow: auto;
  background: #0b131d;
  padding: 10px;
  border-radius: 8px;
  font-size: 12px;
  line-height: 1.7;
}
#error {
  display: none;
  color: #fff;
  background: #7b3038;
  padding: 12px;
  border-radius: 8px;
  white-space: pre-wrap;
  overflow-wrap: anywhere;
}
@media(max-width: 800px) {
  .grid { grid-template-columns: minmax(0, 1fr); }
}
@media(max-width: 380px) {
  main { padding: 8px; }
  .panel { padding: 10px; }
  .metric { padding: 7px; }
  .metric strong { font-size: 18px; }
  .controls { grid-template-columns: repeat(2, minmax(0, 1fr)); }
}
</style>
</head>
<body>
<header>
  <div>
    <h1>🚄 ERR Eco Rapid Rail</h1>
    <div class="small">고속철도 시뮬레이터</div>
  </div>
  <div>
    <div id="clock">05:00:00</div>
    <div class="small">시뮬레이션 시계</div>
  </div>
</header>

<main>
  <div id="error" role="alert"></div>

  <section class="metrics">
    <div class="metric">
      <span>현재 속도</span>
      <strong><b id="speed">0</b> <span>km/h</span></strong>
    </div>
    <div class="metric">
      <span>주행 거리</span>
      <strong><b id="distance">0.0</b> <span>km</span></strong>
    </div>
    <div class="metric">
      <span>신호</span>
      <strong id="signal" style="color:#48e09a">진행</strong>
    </div>
  </section>

  <div class="grid">
    <section class="panel">
      <h2>운전실</h2>
      <canvas id="routeCanvas" width="900" height="210">
        노선 표시 영역
      </canvas>

      <div class="metrics">
        <div class="metric">
          <span>다음 역</span>
          <strong id="nextStation">서울</strong>
        </div>
        <div class="metric">
          <span>역까지 거리</span>
          <strong id="remaining">17.0 km</strong>
        </div>
        <div class="metric">
          <span>제한 속도</span>
          <strong>300 <span>km/h</span></strong>
        </div>
      </div>

      <h2>운전 조작</h2>
      <div class="controls">
        <button class="green" id="accelerate">가속 +</button>
        <button id="reducePower">가속 −</button>
        <button class="red" id="brake">제동</button>
        <button id="releaseBrake">제동 해제</button>
        <button id="hold">정차 유지</button>
        <button id="resume">운행 재개</button>
      </div>

      <p class="small">가속 설정: <b id="powerLabel">50%</b></p>
      <input id="power" class="range" type="range"
        min="0" max="100" value="50">

      <p class="small">시뮬레이션 배속</p>
      <select id="timeScale">
        <option value="1">1배속</option>
        <option value="5">5배속</option>
        <option value="10">10배속</option>
        <option value="30">30배속</option>
      </select>

      <div id="status" class="status">초기화 중...</div>
    </section>

    <section class="panel">
      <h2>열차 시간표</h2>
      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>역</th><th>도착</th><th>출발</th>
              <th>거리</th><th>상태</th>
            </tr>
          </thead>
          <tbody id="timetable">
            <tr><td>서울</td><td>-</td><td>05:00</td><td>0.0 km</td><td>현재 위치</td></tr>
            <tr><td>광명</td><td>05:20</td><td>05:23</td><td>17.0 km</td><td>예정</td></tr>
            <tr><td>천안아산</td><td>05:50</td><td>05:53</td><td>96.5 km</td><td>예정</td></tr>
            <tr><td>오송</td><td>06:10</td><td>06:13</td><td>131.5 km</td><td>예정</td></tr>
            <tr><td>대전</td><td>06:30</td><td>06:35</td><td>166.3 km</td><td>예정</td></tr>
            <tr><td>김천(구미)</td><td>07:00</td><td>07:03</td><td>227.3 km</td><td>예정</td></tr>
            <tr><td>동대구</td><td>07:20</td><td>07:25</td><td>293.1 km</td><td>예정</td></tr>
            <tr><td>경주</td><td>07:40</td><td>07:45</td><td>346.8 km</td><td>예정</td></tr>
            <tr><td>태화강</td><td>08:10</td><td>08:15</td><td>380.0 km</td><td>예정</td></tr>
            <tr><td>부산센텀</td><td>08:30</td><td>-</td><td>417.4 km</td><td>예정</td></tr>
          </tbody>
        </table>
      </div>

      <h2 style="margin-top:20px">운행 열차</h2>
      <div class="table-wrap">
        <table>
          <thead>
            <tr><th>열차</th><th>위치</th><th>속도</th><th>상태</th></tr>
          </thead>
          <tbody id="trainTable">
            <tr><td>PLAYER</td><td>서울</td><td>0 km/h</td><td>대기</td></tr>
            <tr><td>ERR001</td><td>광명</td><td>180 km/h</td><td>운행 중</td></tr>
            <tr><td>ERR002</td><td>천안아산</td><td>200 km/h</td><td>운행 중</td></tr>
          </tbody>
        </table>
      </div>
    </section>
  </div>

  <section class="panel">
    <h2>운행 기록</h2>
    <div id="log">05:00:00 · 시뮬레이터 준비</div>
  </section>
</main>

<script>
(function () {
  "use strict";

  var stations = [
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

  function el(id) {
    return document.getElementById(id);
  }

  var simSeconds = 5 * 3600;
  var km = 0;
  var speed = 0;
  var power = 50;
  var brakeOn = false;
  var held = false;
  var completed = false;
  var lastFrame = 0;
  var aiTrains = [
    {id:"ERR001", km:42, speed:180},
    {id:"ERR002", km:100, speed:200},
    {id:"ERR003", km:165, speed:210},
    {id:"ERR004", km:230, speed:190},
    {id:"ERR005", km:295, speed:220},
    {id:"ERR006", km:350, speed:180}
  ];

  function clockString(seconds) {
    var n = Math.floor(seconds);
    var h = Math.floor(n / 3600) % 24;
    var m = Math.floor(n / 60) % 60;
    var s = n % 60;
    function pad(v) { return v < 10 ? "0" + v : String(v); }
    return pad(h) + ":" + pad(m) + ":" + pad(s);
  }

  function log(message) {
    var box = el("log");
    if (!box) return;
    var line = document.createElement("div");
    line.textContent = clockString(simSeconds) + " · " + message;
    box.insertBefore(line, box.firstChild);
    while (box.children.length > 50) {
      box.removeChild(box.lastChild);
    }
  }

  function nextIndex() {
    var i;
    for (i = 0; i < stations.length; i++) {
      if (stations[i].km > km + 0.01) return i;
    }
    return stations.length - 1;
  }

  function currentIndex() {
    var i, found = 0;
    for (i = 0; i < stations.length; i++) {
      if (stations[i].km <= km + 0.01) found = i;
    }
    return found;
  }

  function drawRoute() {
    var canvas = el("routeCanvas");
    if (!canvas) return;
    var rect = canvas.getBoundingClientRect();
    var w = Math.max(300, Math.floor(rect.width || 600));
    var h = 210;
    var ratio = window.devicePixelRatio || 1;

    if (canvas.width !== Math.floor(w * ratio) ||
        canvas.height !== Math.floor(h * ratio)) {
      canvas.width = Math.floor(w * ratio);
      canvas.height = Math.floor(h * ratio);
    }

    var ctx = canvas.getContext("2d");
    if (!ctx) return;
    ctx.setTransform(ratio, 0, 0, ratio, 0, 0);
    ctx.clearRect(0, 0, w, h);
    ctx.fillStyle = "#09121d";
    ctx.fillRect(0, 0, w, h);

    var left = 30, right = w - 30, y = 100;
    ctx.strokeStyle = "#354b60";
    ctx.lineWidth = 5;
    ctx.beginPath();
    ctx.moveTo(left, y);
    ctx.lineTo(right, y);
    ctx.stroke();

    stations.forEach(function (s) {
      var x = left + s.km / 417.4 * (right - left);
      ctx.fillStyle = "#9cafc4";
      ctx.beginPath();
      ctx.arc(x, y, 4, 0, Math.PI * 2);
      ctx.fill();
      ctx.fillStyle = "#eaf2fc";
      ctx.font = "11px sans-serif";
      ctx.textAlign = "center";
      ctx.fillText(s.name, x, y + 20);
    });

    aiTrains.forEach(function (t) {
      var x = left + t.km / 417.4 * (right - left);
      ctx.fillStyle = "#ffbd4a";
      ctx.beginPath();
      ctx.arc(x, y, 5, 0, Math.PI * 2);
      ctx.fill();
    });

    var px = left + km / 417.4 * (right - left);
    ctx.fillStyle = "#48e09a";
    ctx.beginPath();
    ctx.moveTo(px, y - 8);
    ctx.lineTo(px - 7, y - 21);
    ctx.lineTo(px + 7, y - 21);
    ctx.closePath();
    ctx.fill();

    ctx.textAlign = "left";
    ctx.fillStyle = "#48e09a";
    ctx.font = "bold 12px sans-serif";
    ctx.fillText("PLAYER", left, 25);
    ctx.fillStyle = "#9cafc4";
    ctx.fillText("진행률 " + (km / 417.4 * 100).toFixed(1) + "%", left, 185);
    ctx.fillStyle = "#24384c";
    ctx.fillRect(left, 165, right - left, 7);
    ctx.fillStyle = "#48e09a";
    ctx.fillRect(left, 165, (right - left) * km / 417.4, 7);
  }

  function renderTimetable() {
    var body = el("timetable");
    if (!body) return;
    var ci = currentIndex();

    body.innerHTML = stations.map(function (s, i) {
      var state = "예정";
      if (completed) state = "운행 완료";
      else if (i < ci) state = "통과";
      else if (i === ci) state = "현재 위치";

      return '<tr class="' + (i === ci ? "active" : "") + '">' +
        "<td>" + s.name + "</td>" +
        "<td>" + s.arr + "</td>" +
        "<td>" + s.dep + "</td>" +
        "<td>" + s.km.toFixed(1) + " km</td>" +
        "<td>" + state + "</td></tr>";
    }).join("");
  }

  function renderTrains() {
    var body = el("trainTable");
    if (!body) return;
    var trains = [{
      id:"PLAYER", km:km, speed:speed,
      state:completed ? "도착" : held ? "정차" : brakeOn ? "제동" : "운행 중"
    }];

    aiTrains.forEach(function (t) { trains.push(t); });

    body.innerHTML = trains.map(function (t) {
      var nearest = stations[0];
      stations.forEach(function (s) {
        if (Math.abs(s.km - t.km) < Math.abs(nearest.km - t.km)) {
          nearest = s;
        }
      });
      return "<tr><td>" + t.id + "</td><td>" + nearest.name +
        "</td><td>" + Math.round(t.speed) + " km/h</td><td>" +
        (t.state || "운행 중") + "</td></tr>";
    }).join("");
  }

  function updateUI() {
    el("clock").textContent = clockString(simSeconds);
    el("speed").textContent = Math.round(speed);
    el("distance").textContent = km.toFixed(1);
    el("powerLabel").textContent = power + "%";

    var ni = nextIndex();
    var next = stations[ni];
    el("nextStation").textContent = completed ? "도착" : next.name;
    el("remaining").textContent =
      Math.max(0, next.km - km).toFixed(1) + " km";

    var signal = "진행";
    var color = "#48e09a";
    var nearestAhead = null;

    aiTrains.forEach(function (t) {
      if (t.km > km && (!nearestAhead || t.km < nearestAhead.km)) {
        nearestAhead = t;
      }
    });

    if (nearestAhead && nearestAhead.km - km < 2) {
      signal = "주의";
      color = "#ffbd4a";
    }
    if (nearestAhead && nearestAhead.km - km < 0.6) {
      signal = "정지";
      color = "#ff646e";
    }
    el("signal").textContent = signal;
    el("signal").style.color = color;

    if (completed) el("status").textContent = "부산센텀 도착 · 운행 완료";
    else if (held) el("status").textContent = "정차 유지 중";
    else if (brakeOn) el("status").textContent = "제동 작동 중";
    else el("status").textContent = "운행 중 · " + signal + " 신호";

    renderTimetable();
    renderTrains();
    drawRoute();
  }

  function updateAI(dt) {
    aiTrains.forEach(function (t) {
      var gap = t.km - km;
      var target = 200;

      if (gap > 0 && gap < 1.2) target = 0;

      var change = Math.max(-50 * dt, Math.min(10 * dt, target - t.speed));
      t.speed = Math.max(0, t.speed + change);
      t.km += t.speed * dt / 3600;

      if (t.km > 417.4) t.km = 0;
    });
  }

  function frame(now) {
    try {
      if (!lastFrame) lastFrame = now;
      var realDt = Math.min((now - lastFrame) / 1000, 0.1);
      lastFrame = now;
      var scale = Number(el("timeScale").value) || 1;
      var dt = realDt * scale;

      if (!held && !completed) {
        simSeconds += dt;
        var ni = nextIndex();
        var next = stations[ni];
        var remaining = Math.max(0, next.km - km);
        var target = power / 100 * 300;

        var brakingDistance = speed * speed / (2 * 45 * 3.6) + 0.15;
        if (remaining < brakingDistance + 0.5) {
          target = Math.min(target,
            Math.sqrt(Math.max(0, 2 * 45 * remaining)) * 3.6);
        }
        if (brakeOn) target = 0;

        var ahead = null;
        aiTrains.forEach(function (t) {
          if (t.km > km && (!ahead || t.km < ahead.km)) ahead = t;
        });
        if (ahead && ahead.km - km < 1.2) target = 0;

        var accel = brakeOn ? 90 : 18;
        speed += Math.max(-accel * dt, Math.min(accel * dt, target - speed));
        speed = Math.max(0, Math.min(300, speed));
        km += speed * dt / 3600;

        if (km >= next.km) {
          km = next.km;
          speed = 0;
          if (ni === stations.length - 1) {
            completed = true;
            log("부산센텀 도착 · 운행 완료");
          } else {
            held = true;
            log(next.name + " 역 정차");
            window.setTimeout(function () {
              if (!completed) {
                held = false;
                log(next.name + " 역 출발");
              }
            }, 1800);
          }
        }
        updateAI(dt);
      }

      updateUI();
    } catch (err) {
      var box = el("error");
      if (box) {
        box.style.display = "block";
        box.textContent = "오류: " + err.message;
      }
    }

    window.requestAnimationFrame(frame);
  }

  function bind(id, eventName, handler) {
    var node = el(id);
    if (node) node.addEventListener(eventName, handler);
  }

  bind("accelerate", "click", function () {
    power = Math.min(100, power + 10);
    el("power").value = power;
    updateUI();
  });
  bind("reducePower", "click", function () {
    power = Math.max(0, power - 10);
    el("power").value = power;
    updateUI();
  });
  bind("brake", "click", function () {
    brakeOn = true;
    log("제동 작동");
    updateUI();
  });
  bind("releaseBrake", "click", function () {
    brakeOn = false;
    log("제동 해제");
    updateUI();
  });
  bind("hold", "click", function () {
    held = true;
    speed = 0;
    log("정차 유지");
    updateUI();
  });
  bind("resume", "click", function () {
    held = false;
    log("운행 재개");
    updateUI();
  });
  bind("power", "input", function (event) {
    power = Number(event.target.value);
    updateUI();
  });

  window.addEventListener("resize", drawRoute);

  try {
    updateUI();
    log("시계·시간표·운전실 초기화 완료");
    window.requestAnimationFrame(frame);
  } catch (err) {
    el("error").style.display = "block";
    el("error").textContent = "초기화 오류: " + err.message;
  }
}());
</script>
</body>
</html>
