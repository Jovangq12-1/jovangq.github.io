<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Browser Map PvE Game</title>

  <!-- Leaflet CSS -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />

  <style>
    html, body, #map { height: 100%; margin: 0; padding: 0; }
    #combat-ui {
      position: absolute;
      bottom: 10%;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0,0,0,0.8);
      color: #fff;
      padding: 15px;
      border-radius: 10px;
      display: none;
      text-align: center;
      width: 250px;
    }
    #combat-ui button { margin-top: 10px; padding: 5px 10px; width: 100px; }
    #xp-display {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(0,0,0,0.7);
      color: #fff;
      padding: 8px 12px;
      border-radius: 8px;
      font-weight: bold;
    }
  </style>
</head>
<body>

<div id="map"></div>
<div id="xp-display">XP: 0</div>

<div id="combat-ui">
  <div id="mob-name">Mob: </div>
  <div id="mob-hp">HP: </div>
  <button id="attack-btn">Attack</button>
</div>

<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<script>
/* =========================
   Map Initialization
========================= */
var map = L.map('map').setView([37.7749, -122.4194], 16); // default coords
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
  attribution: '© OpenStreetMap contributors'
}).addTo(map);

/* =========================
   Player Setup
========================= */
var playerMarker = L.circleMarker([37.7749, -122.4194], {
  radius: 10, color: 'blue', fillColor: '#00f', fillOpacity: 0.8
}).addTo(map);

var xp = 0;
var xpDisplay = document.getElementById('xp-display');

function updatePlayerLocation(lat, lng){
  playerMarker.setLatLng([lat, lng]);
  map.setView([lat, lng]); // center map on player
}

/* =========================
   GPS Tracking
========================= */
if (navigator.geolocation) {
  navigator.geolocation.watchPosition(function(position){
    updatePlayerLocation(position.coords.latitude, position.coords.longitude);
    spawnNearbyMobs(position.coords.latitude, position.coords.longitude);
  }, function(error){
    console.error("GPS error: " + error.message);
  }, { enableHighAccuracy: true });
} else {
  alert("Geolocation not supported by this browser.");
}

/* =========================
   Mob System
========================= */
var mobs = [];
var spawnRadius = 0.001; // ~100m radius

function spawnNearbyMobs(playerLat, playerLng){
  if (mobs.length >= 5) return; // max 5 mobs nearby

  var mobLat = playerLat + (Math.random() - 0.5) * spawnRadius;
  var mobLng = playerLng + (Math.random() - 0.5) * spawnRadius;

  var mob = {
    name: "Goblin",
    hp: 10,
    marker: L.marker([mobLat, mobLng]).addTo(map)
  };

  mob.marker.on('click', function(){
    startCombat(mob);
  });

  mobs.push(mob);
}

/* =========================
   Combat System
========================= */
var combatUI = document.getElementById('combat-ui');
var mobNameEl = document.getElementById('mob-name');
var mobHpEl = document.getElementById('mob-hp');
var attackBtn = document.getElementById('attack-btn');
var currentMob = null;

function startCombat(mob){
  currentMob = mob;
  mobNameEl.innerText = "Mob: " + mob.name;
  mobHpEl.innerText = "HP: " + mob.hp;
  combatUI.style.display = "block";
}

attackBtn.onclick = function(){
  if (!currentMob) return;

  currentMob.hp -= 5; // simple attack damage
  mobHpEl.innerText = "HP: " + currentMob.hp;

  if(currentMob.hp <= 0){
    xp += 10; // reward XP
    xpDisplay.innerText = "XP: " + xp;
    map.removeLayer(currentMob.marker);
    mobs = mobs.filter(m => m !== currentMob);
    currentMob = null;
    combatUI.style.display = "none";
  }
};
</script>

</body>
</html>
