<!doctype html>
<html lang="vi">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Zoom tự động Việt Nam → Kon Tum → Quảng Ngãi → Hoàng Sa</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.Default.css">

<style>
html, body { height:100%; margin:0; overflow:hidden; }
#map { height:100%; }
.topbar {
  position:absolute; top:10px; left:10px;
  z-index:1000;
  background:rgba(255,255,255,0.9);
  padding:10px; border-radius:8px;
  box-shadow:0 2px 8px rgba(0,0,0,0.2);
}
.btn {
  padding:8px 12px; margin-right:6px;
  border:none; background:#1976d2; color:white;
  border-radius:6px; cursor:pointer;
}
.btn:disabled { background:#9bbde6; cursor:default; }

#videoModal {
  display:none;
  position:fixed; top:0; left:0;
  width:100%; height:100%;
  background:rgba(0,0,0,0.85);
  z-index:999999;
  justify-content:center;
  align-items:center;
}
#videoModal iframe {
  width:80vw;
  height:45vw;
  max-width:900px;
  max-height:500px;
  border:none; 
  border-radius:12px;
  box-shadow:0 0 20px rgba(255,255,255,0.3);
}
</style>
</head>
<body>

<div id="map"></div>

<div class="topbar">
  <div>Zoom sequence tự động</div>
  <button class="btn" id="start">Bắt đầu</button>
  <button class="btn" id="reset">Reset</button>
</div>

<div id="videoModal">
  <iframe id="modalFrame" allow="autoplay"></iframe>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet.markercluster/dist/leaflet.markercluster.js"></script>

<script>

// =====================================
// GOOGLE DRIVE URL AUTO-CONVERTER
// =====================================
function convertDriveUrl(url) {
  if (!url) return "";

  // Trường hợp: https://drive.google.com/file/d/ID/view?usp=...
  let m1 = url.match(/\/d\/([^/]+)\//);
  if (m1) return `https://drive.google.com/file/d/${m1[1]}/preview`;

  // Trường hợp chỉ đưa ID
  if (/^[A-Za-z0-9_-]{10,}$/.test(url))
    return `https://drive.google.com/file/d/${url}/preview`;

  return url;
}

// =============================
// VIDEO SYSTEM
// =============================
const videoList = {
  "Cồng Chiêng Tây Nguyên": "https://drive.google.com/file/d/1C_Ht3F5C3khAG977r2OSP_7sGL03aqfj/view?usp=drive_link",
  "Thổ Cẩm Kon Tum": "https://drive.google.com/file/d/1SRh8NTWVfDrW0iIz8tYbVcFgqQqFbS90/view?usp=drive_link",
  "Nhà Rông Kon Tum": "https://drive.google.com/file/d/15GHEVH16EkSZSg8SurC8ngQMWHV--KQB/view?usp=drive_link",
  "Nhà Thờ gỗ Kon Tum": "https://drive.google.com/file/d/17bOVsnfDJJsieO0f72_NfiDhnBjLQNzu/view?usp=drive_link",

  "Thổ Cẩm Quảng Ngãi": "https://drive.google.com/file/d/1A6XEbjURBkj9hhVVK_Dhc6ytvoqEA7HM/view?usp=drive_link",
  "Văn hóa Sa Huỳnh": "https://drive.google.com/file/d/1MKX5_tNxgb8FK8pspOHwyKEvOOkKj0Y1/view?usp=drive_link",
  "Quần đảo Hoàng Sa": "https://drive.google.com/file/d/1fo1R4VnJkdKtos2IKdbWUYfh0IyKMrSp/view?usp=drive_link",
  "Đảo Lý Sơn": "https://drive.google.com/file/d/1c3kjjsoZY3hKAFeRFhBzJUCfq0N2xfY7/view?usp=drive_link"
};

function openVideo(src) {
  const finalUrl = convertDriveUrl(src);

  if (!finalUrl) {
    alert("Chưa có video cho địa điểm này!");
    return;
  }

  document.getElementById("modalFrame").src = finalUrl;
  document.getElementById("videoModal").style.display = "flex";
}

document.getElementById("videoModal").onclick = () => {
  document.getElementById("modalFrame").src = "";
  document.getElementById("videoModal").style.display = "none";
};

// =============================
// MAP
// =============================
const map = L.map("map").setView([20, 0], 2);

L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
  maxZoom: 20
}).addTo(map);

// =============================
// MARKERS
// =============================
const markers = L.markerClusterGroup();
const markerMap = {};

const data = [
  {lat:14.3537, lng:107.9772, text:"Kon Tum"},
  {lat:14.78306, lng:107.73278, text:"Cồng Chiêng Tây Nguyên"},
  {lat:14.32932, lng:108.06449, text:"Thổ Cẩm Kon Tum"},
  {lat:14.58082, lng:108.27663, text:"Nhà Rông Kon Tum"},
  {lat:14.585, lng:108.28, text:"Nhà Thờ gỗ Kon Tum"},

  {lat:15.1250, lng:108.7970, text:"Quảng Ngãi"},
  {lat:14.98427, lng:108.53643, text:"Thổ Cẩm Quảng Ngãi"},
  {lat:14.81, lng:108.96, text:"Văn hóa Sa Huỳnh"},
  {lat:15.4445, lng:113.4820, text:"Quần đảo Hoàng Sa"},
  {lat:15.065, lng:109.125, text:"Đảo Lý Sơn"}
];

data.forEach(d => {
  const key = `${d.lat},${d.lng}`;

  const popup = `
    <b>${d.text}</b><br>
    <button onclick="openVideo('${videoList[d.text]}')"
      style="margin-top:6px;padding:6px 10px;border:none;background:#d32f2f;color:white;border-radius:6px;cursor:pointer;">
      Xem video
    </button>
  `;

  const m = L.marker([d.lat, d.lng]).bindPopup(popup, {autoClose:false});
  markers.addLayer(m);
  markerMap[key] = m;
});

map.addLayer(markers);

// =============================
// ZOOM CONTROL
// =============================
function delay(ms){ return new Promise(r=>setTimeout(r,ms)); }

function safeFly(latlng, zoom, duration=1.5){
  return new Promise(resolve=>{
    let done=false;
    const finish=()=>{ if(!done){done=true; resolve();} };
    map.once("moveend", finish);
    setTimeout(finish, duration*1300);
    map.flyTo(latlng, zoom, {duration});
  });
}

async function showMarker(key){
  const m = markerMap[key];
  const latlng = m.getLatLng();

  await safeFly(latlng, 10, 1.2);
  await safeFly(latlng, 16, 1.2);

  await new Promise(resolve=>{
    let done=false;
    const finish=()=>{ if(!done){done=true; resolve();} };
    setTimeout(finish,1500);

    markers.zoomToShowLayer(m, ()=>{
      m.openPopup();
      finish();
    });
  });

  await delay(1200);
}

// =============================
// ZOOM SEQUENCE
// =============================
async function run(){
  document.getElementById("start").disabled = true;

  await safeFly([20,0],2,2);
  await safeFly([16,108],6,2);

  await showMarker("14.3537,107.9772");    
  await showMarker("14.78306,107.73278");  
  await showMarker("14.32932,108.06449");  
  await showMarker("14.58082,108.27663");  
  await showMarker("14.585,108.28");       

  await showMarker("15.125,108.797");      
  await showMarker("14.98427,108.53643");  
  await showMarker("14.81,108.96");        
  await showMarker("15.4445,113.482");     
  await showMarker("15.065,109.125");      

  document.getElementById("start").disabled = false;
}

document.getElementById("start").onclick = run;

document.getElementById("reset").onclick = ()=>{
  map.stop();
  map.setView([20,0],2);
  Object.values(markerMap).forEach(m=>m.closePopup());
  document.getElementById("start").disabled = false;
};

</script>
</body>
</html>
