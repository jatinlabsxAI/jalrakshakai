# jalrakshakai
JalRakshak AI is an AI-powered flood intelligence platform for the Yamuna floodplain. It combines satellite data, terrain, rainfall, and historical river levels to assess flood risk, predict potential inundation, generate location-specific risk maps, and provide actionable alerts for citizens and authorities.
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>JalRakshak — Flood Early Warning System</title>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.js"></script>
<style>
  :root{
    --navy: #0B2A4A;
    --navy-deep: #071B31;
    --steel: #1D5B79;
    --paper: #F4F6F5;
    --panel: #FFFFFF;
    --ink: #142B36;
    --muted: #5A6E77;
    --line: #DDE3E1;
    --green: #2E8B4F;
    --yellow: #B7913A;
    --orange: #C57B3B;
    --red: #B23F30;
    --sans: -apple-system, "Segoe UI", Helvetica, Arial, sans-serif;
    --mono: ui-monospace, "SF Mono", "Cascadia Mono", Consolas, monospace;
  }
  *{box-sizing:border-box; margin:0; padding:0;}
  body{ background: var(--paper); color: var(--ink); font-family: var(--sans); }

  /* ---- gov header ---- */
  header{
    background: var(--navy);
    color: #fff;
    padding: 12px 22px;
    display:flex;
    align-items:center;
    justify-content:space-between;
    flex-wrap:wrap;
    gap:10px;
    border-bottom: 3px solid var(--steel);
  }
  .brand{ display:flex; align-items:center; gap:12px; }
  .brand svg{ flex-shrink:0; }
  .brand h1{ font-size: 19px; font-weight:700; letter-spacing:0.2px; }
  .brand p{ font-size: 11.5px; color: #B9C7D2; margin-top:1px; }
  .header-right{ display:flex; align-items:center; gap:10px; flex-wrap:wrap; }
  .official-tag{
    font-size: 10.5px;
    background: rgba(255,255,255,0.1);
    border: 1px solid rgba(255,255,255,0.25);
    padding: 5px 10px;
    border-radius: 3px;
    color: #DCE6EC;
  }

  .disclaimer-strip{
    background: #FFF4E0;
    color: #7A5A1E;
    font-size: 12px;
    text-align:center;
    padding: 6px 12px;
    border-bottom: 1px solid #E9D8AE;
  }

  /* ---- map section ---- */
  .map-section{ position:relative; }
  #map{ height: 62vh; min-height: 420px; width: 100%; }

  .map-float{
    position:absolute;
    z-index: 1000;
    background: rgba(255,255,255,0.96);
    border-radius: 5px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.18);
    padding: 12px 14px;
  }
  .float-top-right{ top:14px; right:14px; width: 220px; }
  .float-top-left{ top:14px; left:14px; }
  .float-bottom{ left:14px; right:14px; bottom:14px; }

  .locate-btn{
    background: var(--steel);
    color:#fff;
    border:none;
    padding: 9px 14px;
    border-radius: 4px;
    font-size: 13px;
    font-weight:600;
    cursor:pointer;
    display:flex;
    align-items:center;
    gap:7px;
  }
  .locate-btn:hover{ background: var(--navy); }
  .locate-status{
    font-size: 11.5px;
    color: var(--muted);
    margin-top: 8px;
    max-width: 220px;
    line-height:1.4;
  }
  .locate-alert{
    margin-top: 8px;
    font-size: 12px;
    font-weight: 600;
    padding: 7px 9px;
    border-radius: 4px;
    display:none;
  }

  .gauge-mini-label{ font-size: 11px; color: var(--muted); margin-bottom: 4px; }
  .gauge-mini-value{ font-family: var(--mono); font-size: 22px; font-weight:700; color: var(--navy); }
  .gauge-mini-band{ font-size: 12px; font-weight:700; margin-top: 4px; }

  .timeline-inner{ display:flex; align-items:center; gap: 14px; }
  .timeline-track{ position:relative; flex:1; height:4px; background: var(--line); border-radius:2px; margin: 0 6px; }
  .timeline-fill{ position:absolute; left:0; top:0; height:100%; background: var(--steel); border-radius:2px; width:0%; transition: width .25s ease; }
  .timeline-stops{ position:absolute; top:-9px; left:0; right:0; display:flex; justify-content:space-between; }
  .stop{ display:flex; flex-direction:column; align-items:center; gap:5px; cursor:pointer; background:none; border:none; }
  .stop-dot{ width:11px; height:11px; border-radius:50%; background:#fff; border:2px solid var(--line); }
  .stop.active .stop-dot{ background: var(--steel); border-color: var(--steel); }
  .stop-label{ font-family: var(--mono); font-size: 11px; color: var(--muted); margin-top:12px; }
  .stop.active .stop-label{ color: var(--navy); font-weight:700; }
  .timeline-caption{ font-size: 11.5px; color: var(--muted); white-space:nowrap; }

  /* ---- lower sections ---- */
  .wrap{ max-width: 1180px; margin: 0 auto; padding: 20px; }
  .readout-strip{ display:grid; grid-template-columns: repeat(5,1fr); gap:12px; margin-bottom:18px; }
  @media (max-width: 800px){ .readout-strip{ grid-template-columns: repeat(2,1fr); } }
  .readout{ background:var(--panel); border:1px solid var(--line); border-radius:5px; padding:12px 14px; }
  .readout-label{ font-size:11px; color:var(--muted); margin-bottom:5px; }
  .readout-value{ font-family:var(--mono); font-size:19px; font-weight:700; color:var(--navy); }
  .readout-sub{ font-size:10.5px; color:var(--muted); margin-top:3px; }

  .bottom-grid{ display:grid; grid-template-columns: 1.3fr 1fr; gap:16px; }
  @media (max-width: 800px){ .bottom-grid{ grid-template-columns:1fr; } }
  .panel{ background:var(--panel); border:1px solid var(--line); border-radius:5px; padding:16px 18px; }
  .panel h3{ font-size:14.5px; color:var(--navy); margin-bottom:12px; font-weight:700; }
  .road-row{ display:flex; justify-content:space-between; align-items:center; padding:9px 0; border-bottom:1px solid var(--line); font-size:13px; gap:10px; }
  .road-row:last-child{ border-bottom:none; }
  .road-status{ font-size:11px; font-weight:700; padding:3px 9px; border-radius:999px; white-space:nowrap; }
  .status-clear{ background:rgba(46,139,79,0.13); color:var(--green); }
  .status-caution{ background:rgba(183,145,58,0.16); color:var(--yellow); }
  .status-avoid{ background:rgba(178,63,48,0.14); color:var(--red); }
  .legend-row{ display:flex; justify-content:space-between; font-family:var(--mono); font-size:12px; padding:6px 0; border-bottom:1px solid var(--line); }
  .legend-row:last-child{ border-bottom:none; }
  .legend-dot{ display:inline-block; width:8px; height:8px; border-radius:50%; margin-right:8px; }

  footer{ text-align:center; font-size:11px; color:var(--muted); padding: 16px 20px 26px 20px; line-height:1.6; }
</style>
</head>
<body>

<div class="disclaimer-strip">Hackathon prototype for government evaluation — not a live operational alert system. Data reflects the validated July 2023 Yamuna flood event.</div>

<header>
  <div class="brand">
    <svg width="34" height="34" viewBox="0 0 34 34">
      <path d="M17 2 L30 8 V17 C30 25 24 30 17 32 C10 30 4 25 4 17 V8 Z" fill="var(--steel)" stroke="#fff" stroke-width="1.2"/>
      <path d="M11 18 Q17 12 23 18 Q17 15 11 18Z" fill="#fff" opacity="0.9"/>
      <path d="M11 22 Q17 16 23 22 Q17 19 11 22Z" fill="#fff" opacity="0.7"/>
    </svg>
    <div>
      <h1>JalRakshak — Flood Early Warning System</h1>
      <p>Yamuna river corridor, Delhi NCR · Disaster Management Pilot</p>
    </div>
  </div>
  <div class="header-right">
    <span class="official-tag">Sentinel-1 SAR · SRTM DEM · GPM IMERG</span>
    <span class="official-tag">Validated event: July 2023</span>
  </div>
</header>

<div class="map-section">
  <div id="map"></div>

  <div class="map-float float-top-left">
    <button class="locate-btn" onclick="locateUser()">
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2.5"><circle cx="12" cy="12" r="3"/><path d="M12 2v3M12 19v3M2 12h3M19 12h3"/></svg>
      Show my location
    </button>
    <div class="locate-status" id="locateStatus">Tap to check your distance from active flood-risk zones.</div>
    <div class="locate-alert" id="locateAlert"></div>
  </div>

  <div class="map-float float-top-right">
    <div class="gauge-mini-label" id="dateLabel">July 9, 2023 — Old Railway Bridge gauge</div>
    <div class="gauge-mini-value" id="gaugeReading">203.18 m</div>
    <div class="gauge-mini-band" id="gaugeBand">Normal</div>
  </div>

  <div class="map-float float-bottom">
    <div class="timeline-inner">
      <div class="timeline-caption">Jul 9</div>
      <div class="timeline-track">
        <div class="timeline-fill" id="timelineFill"></div>
        <div class="timeline-stops">
          <button class="stop active" data-i="0" onclick="setStop(0)"><span class="stop-dot"></span><span class="stop-label">9</span></button>
          <button class="stop" data-i="1" onclick="setStop(1)"><span class="stop-dot"></span><span class="stop-label">10</span></button>
          <button class="stop" data-i="2" onclick="setStop(2)"><span class="stop-dot"></span><span class="stop-label">11</span></button>
          <button class="stop" data-i="3" onclick="setStop(3)"><span class="stop-dot"></span><span class="stop-label">12</span></button>
          <button class="stop" data-i="4" onclick="setStop(4)"><span class="stop-dot"></span><span class="stop-label">13</span></button>
        </div>
      </div>
      <div class="timeline-caption">Jul 13</div>
    </div>
  </div>
</div>

<div class="wrap">
  <div class="readout-strip">
    <div class="readout"><div class="readout-label">Risk score</div><div class="readout-value" id="riskValue">32/100</div><div class="readout-sub">(level−200)/10 × 100</div></div>
    <div class="readout"><div class="readout-label">3-day rainfall</div><div class="readout-value" id="rainValue">131 mm</div><div class="readout-sub">NASA GPM IMERG</div></div>
    <div class="readout"><div class="readout-label">SAR images used</div><div class="readout-value">5 / 2</div><div class="readout-sub">baseline / flood-window</div></div>
    <div class="readout"><div class="readout-label">Peak flood depth</div><div class="readout-value">8.66 m</div><div class="readout-sub">above 200m reference</div></div>
    <div class="readout"><div class="readout-label">Rainfall → level R²</div><div class="readout-value">0.271</div><div class="readout-sub">n=5, demo-scale</div></div>
  </div>

  <div class="bottom-grid">
    <div class="panel">
      <h3>Route advisory for emergency services</h3>
      <div id="roadsList"></div>
    </div>
    <div class="panel">
      <h3>Alert classification (CWC official thresholds)</h3>
      <div class="legend-row"><span><span class="legend-dot" style="background:var(--green)"></span>Normal</span><span>&lt; 204.50 m</span></div>
      <div class="legend-row"><span><span class="legend-dot" style="background:var(--yellow)"></span>Watch</span><span>204.50–205.32 m</span></div>
      <div class="legend-row"><span><span class="legend-dot" style="background:var(--orange)"></span>Danger mark crossed</span><span>205.33–205.99 m</span></div>
      <div class="legend-row"><span><span class="legend-dot" style="background:var(--red)"></span>Evacuation, critical</span><span>≥ 206.00 m</span></div>
    </div>
  </div>
</div>

<footer>
  JalRakshak AI — DEM susceptibility + Sentinel-1 SAR validation + GPM IMERG rainfall fused into one spatial risk model.<br>
  Prototype built for hackathon evaluation. Location data is used only in this browser session and is never stored or transmitted.
</footer>

<script>
  const landmarks = [
    { name: "Wazirabad Barrage", lat: 28.7167, lng: 77.2283 },
    { name: "Kashmere Gate / Yamuna Bazar", lat: 28.6670, lng: 77.2420 },
    { name: "ITO", lat: 28.6289, lng: 77.2493 },
    { name: "Shastri Park / Usmanpur", lat: 28.6780, lng: 77.2620 },
    { name: "Mayur Vihar", lat: 28.6096, lng: 77.2934 },
    { name: "Okhla Barrage", lat: 28.5535, lng: 77.2965 }
  ];

  const stopsData = [
    { date:"July 9, 2023",  level: 203.18, risk: 32, band: "Normal",                    color: "#2E8B4F", rain: 131 },
    { date:"July 10, 2023", level: 205.36, risk: 54, band: "Danger mark crossed",       color: "#C57B3B", rain: 188 },
    { date:"July 11, 2023", level: 206.60, risk: 66, band: "Evacuation level, critical",color: "#B23F30", rain: 189 },
    { date:"July 12, 2023", level: 207.25, risk: 72, band: "Evacuation level, critical",color: "#B23F30", rain: 127 },
    { date:"July 13, 2023", level: 208.66, risk: 87, band: "Evacuation level, critical",color: "#B23F30", rain: 28 }
  ];

  const roadsData = [
    { name: "ITO Underpass", floodsAtStop: 2 },
    { name: "Kashmere Gate / Ring Road", floodsAtStop: 3 },
    { name: "Mayur Vihar Link Road", floodsAtStop: 4 }
  ];

  // ---- map setup ----
  const map = L.map('map', { zoomControl: true }).setView([28.63, 77.26], 11);

  // Esri World Imagery avoids the OSM 403 tile issue you hit in Colab
  L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
    attribution: 'Esri, Maxar, Earthstar Geographics',
    maxZoom: 18
  }).addTo(map);

  L.control.scale({ imperial: false }).addTo(map);

  // study region boundary
  L.rectangle([[28.53, 77.15],[28.73, 77.32]], {
    color: "#DCE6EC", weight: 1.5, dashArray: "5 5", fill: false
  }).addTo(map);

  const riverMarkers = landmarks.map(lm =>
    L.marker([lm.lat, lm.lng]).addTo(map).bindPopup(`<b>${lm.name}</b>`)
  );

  const riskCircles = landmarks.map(lm =>
    L.circle([lm.lat, lm.lng], { radius: 500, color: "#2E8B4F", fillColor: "#2E8B4F", fillOpacity: 0.28, weight: 1 })
      .addTo(map)
      .bindPopup('')
  );

  let userMarker = null;

  function haversine(lat1, lon1, lat2, lon2){
    const R = 6371;
    const dLat = (lat2-lat1) * Math.PI/180;
    const dLon = (lon2-lon1) * Math.PI/180;
    const a = Math.sin(dLat/2)**2 + Math.cos(lat1*Math.PI/180)*Math.cos(lat2*Math.PI/180)*Math.sin(dLon/2)**2;
    return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
  }

  function checkUserRisk(userLat, userLng, stopIndex){
    const d = stopsData[stopIndex];
    const atRisk = (d.band !== "Normal");
    const alertBox = document.getElementById('locateAlert');
    if(!atRisk){
      alertBox.style.display = 'none';
      return;
    }
    let nearest = null, nearestDist = Infinity;
    landmarks.forEach(lm => {
      const dist = haversine(userLat, userLng, lm.lat, lm.lng);
      if(dist < nearestDist){ nearestDist = dist; nearest = lm; }
    });
    alertBox.style.display = 'block';
    alertBox.style.background = d.color + '22';
    alertBox.style.color = d.color;
    alertBox.innerHTML = `⚠ You are ${nearestDist.toFixed(1)} km from ${nearest.name}<br>Current status: ${d.band}`;
  }

  function locateUser(){
    const statusEl = document.getElementById('locateStatus');
    statusEl.textContent = "Requesting location permission...";
    if(!navigator.geolocation){
      statusEl.textContent = "Geolocation is not available in this browser.";
      return;
    }
    navigator.geolocation.getCurrentPosition(
      pos => {
        const { latitude, longitude } = pos.coords;
        if(userMarker) map.removeLayer(userMarker);
        userMarker = L.circleMarker([latitude, longitude], {
          radius: 8, color: "#1D5B79", fillColor: "#1D5B79", fillOpacity: 0.9, weight: 2
        }).addTo(map).bindPopup("Your location").openPopup();
        map.setView([latitude, longitude], 12);
        statusEl.textContent = "Location found. Checking nearby risk zones...";
        checkUserRisk(latitude, longitude, currentStop);
      },
      err => {
        statusEl.innerHTML = "Location access was not available (" + err.message + "). " +
          "<br>Note: if you opened this file directly (file://), browsers block location for security. " +
          "Run it through a local server instead — see the setup note.";
      }
    );
  }

  let currentStop = 0;

  function updateRoads(currentStop){
    const list = document.getElementById('roadsList');
    list.innerHTML = '';
    roadsData.forEach(road => {
      let statusText, statusClass;
      if(currentStop >= road.floodsAtStop){ statusText = "Avoid — flooded"; statusClass = "status-avoid"; }
      else if(currentStop >= road.floodsAtStop - 1){ statusText = "Avoid — flooding likely"; statusClass = "status-avoid"; }
      else if(currentStop >= road.floodsAtStop - 2){ statusText = "Caution — risk rising"; statusClass = "status-caution"; }
      else { statusText = "Clear"; statusClass = "status-clear"; }
      list.innerHTML += `<div class="road-row"><span>${road.name}</span><span class="road-status ${statusClass}">${statusText}</span></div>`;
    });
  }

  function setStop(i){
    currentStop = i;
    document.querySelectorAll('.stop').forEach(s => s.classList.remove('active'));
    document.querySelector('.stop[data-i="'+i+'"]').classList.add('active');
    document.getElementById('timelineFill').style.width = (i/4*100) + '%';

    const d = stopsData[i];
    document.getElementById('dateLabel').textContent = d.date + " — Old Railway Bridge gauge";
    document.getElementById('gaugeReading').textContent = d.level.toFixed(2) + ' m';
    document.getElementById('gaugeReading').style.color = d.color;
    document.getElementById('gaugeBand').textContent = d.band;
    document.getElementById('gaugeBand').style.color = d.color;
    document.getElementById('riskValue').textContent = d.risk + '/100';
    document.getElementById('rainValue').textContent = d.rain + ' mm';

    // scale risk circles: only the landmarks nearest the river bend (ITO, Shastri Park, Mayur Vihar) go critical first
    const criticalOrder = [2, 3, 4, 1, 0, 5]; // indices into landmarks, ITO/Shastri/Mayur flood earliest
    riskCircles.forEach((circle, idx) => {
      const rank = criticalOrder.indexOf(idx);
      const localRisk = Math.max(10, d.risk - rank * 8);
      const color = localRisk > 70 ? "#B23F30" : localRisk > 50 ? "#C57B3B" : localRisk > 30 ? "#B7913A" : "#2E8B4F";
      circle.setRadius(300 + localRisk * 15);
      circle.setStyle({ color: color, fillColor: color });
      circle.setPopupContent(`<b>${landmarks[idx].name}</b><br>Local risk: ${localRisk}/100`);
    });

    updateRoads(i);
    if(userMarker){
      const ll = userMarker.getLatLng();
      checkUserRisk(ll.lat, ll.lng, i);
    }
  }

  setStop(0);
</script>

</body>
</html>
