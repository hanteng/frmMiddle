---
title: "Globe Experiment-— Middle Powers Selector"
format: html
---

## Interactive globe

```{=html}
<div id="globe" style="width:100%;height:auto;min-height:480px;background:#000"></div>
<div style="margin-top:10px;">
  <select id="middlePowerSelector">
    </select>
</div>

<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.155.0/build/three.module.js",
    "three/examples/jsm/": "https://cdn.jsdelivr.net/npm/three@0.155.0/examples/jsm/",
    "d3": "https://cdn.jsdelivr.net/npm/d3@7/+esm",
    "topojson-client": "https://cdn.jsdelivr.net/npm/topojson-client@3/+esm"
  }
}
</script>

<script type="module">
import * as THREE from "three";
import { Line2 } from "three/examples/jsm/lines/Line2.js";
import { LineMaterial } from "three/examples/jsm/lines/LineMaterial.js";
import { LineGeometry } from "three/examples/jsm/lines/LineGeometry.js";

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import * as d3 from "d3";
import { feature as topojsonFeature } from "topojson-client";

/* 🌍 ===== Settings ===== */
const GLOBE_CONFIG = {
    // Globe constants
    RADIUS: 200,
    // Resource URLs
    //SAT_TEXTURE_URL: "https://cdn.jsdelivr.net/npm/three-globe/example/img/earth-blue-marble.jpg",
    SAT_TEXTURE_URL: "earth-blue-marble-maritime.png",
    TOPO_URL: "https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson",
    //TOPO_URL: "https://cdn.jsdelivr.net/npm/world-atlas@2/countries-50m.json",
    // Camera settings
    CAMERA_DISTANCE: 200 * 2.6,   // ~520
    CLOSE_DISTANCE:  200 * 1.2,   // ~240
};

// Destructure values for convenience and readability
const { RADIUS, SAT_TEXTURE_URL, TOPO_URL, CAMERA_DISTANCE, CLOSE_DISTANCE } = GLOBE_CONFIG;

/* ========================================= */
/* 🌍 Middle Powers Configuration (ISO3 keys) */
/* ========================================= */
const MIDDLE_POWERS = {
  "SGP": { label: "Singapore (新加坡) 🇸🇬", name: "Singapore", lat: 1.35, lon: 103.8, color: 0x8dc1dc, closeDistF: 1.05, type: "Electoral Democracy" },
  "TWN": { label: "Taiwan (臺灣) 🇹🇼", name: "Taiwan", lat: 23.5, lon: 121.0, color: 0x2c7bb6, closeDistF: 1.125, type: "Full Democracy (Polyarchy)" },
  "NLD": { label: "Netherlands (荷蘭) 🇳🇱", name: "Netherlands", lat: 52.3, lon: 5.5, color: 0x2c7bb6, closeDistF: 1.2, type: "Full Democracy (Polyarchy)" },
  "KOR": { label: "South Korea (南韓) 🇰🇷", name: "South Korea", lat: 36.0, lon: 127.5, color: 0x2c7bb6, closeDistF: 1.28, type: "Full Democracy (Polyarchy)" },
  "ITA": { label: "Italy (義大利) 🇮🇹", name: "Italy", lat: 42.0, lon: 12.5, color: 0x2c7bb6, closeDistF: 1.30, type: "Full Democracy (Polyarchy)" },
  "DEU": { label: "Germany (德國) 🇩🇪", name: "Germany", lat: 51.0, lon: 9.0, color: 0x2c7bb6, closeDistF: 1.32, type: "Full Democracy (Polyarchy)" },
  "POL": { label: "Poland (波蘭) 🇵🇱", name: "Poland", lat: 52.0, lon: 19.0, color: 0x8dc1dc, closeDistF: 1.33, type: "Electoral Democracy" },
  "TUR": { label: "Turkey (土耳其) 🇹🇷", name: "Turkey", lat: 39.0, lon: 35.0, color: 0xE6F598, closeDistF: 1.38, type: "Multiparty Autocracy" },
  "FIN": { label: "Finland (芬蘭) 🇫🇮", name: "Finland", lat: 62.0, lon: 26.0, color: 0x2c7bb6, closeDistF: 1.35, type: "Full Democracy (Polyarchy)" },
  "UKR": { label: "Ukraine (烏克蘭) 🇺🇦", name: "Ukraine", lat: 49.0, lon: 32.0, color: 0x8dc1dc, closeDistF: 1.38, type: "Electoral Democracy" },
  "NOR": { label: "Norway (挪威) 🇳🇴", name: "Norway", lat: 61.0, lon: 8.0, color: 0x2c7bb6, closeDistF: 1.39, type: "Full Democracy (Polyarchy)" },
  "JPN": { label: "Japan (日本) 🇯🇵", name: "Japan", lat: 36.0, lon: 138.0, color: 0x2c7bb6, closeDistF: 1.45, type: "Full Democracy (Polyarchy)" },
  "ZAF": { label: "South Africa (南非) 🇿🇦", name: "South Africa", lat: -30.0, lon: 25.0, color: 0x8dc1dc, closeDistF: 1.48, type: "Electoral Democracy" },
  "MEX": { label: "Mexico (墨西哥) 🇲🇽", name: "Mexico", lat: 23.0, lon: -102.0, color: 0x8dc1dc, closeDistF: 1.55, type: "Electoral Democracy" },
  "IDN": { label: "Indonesia (印尼) 🇮🇩", name: "Indonesia", lat: -5.0, lon: 120.0, color: 0x8dc1dc, closeDistF: 1.60, type: "Electoral Democracy" },
  "SAU": { label: "Saudi Arabia (沙烏地阿拉伯) 🇸🇦", name: "Saudi Arabia", lat: 25.0, lon: 45.0, color: 0xD73027, closeDistF: 1.65, type: "Non-electoral autocracy" },
  "ARG": { label: "Argentina (阿根廷) 🇦🇷", name: "Argentina", lat: -34.0, lon: -64.0, color: 0x8dc1dc, closeDistF: 1.70, type: "Electoral Democracy" },
  "IND": { label: "India (印度) 🇮🇳", name: "India", lat: 21.0, lon: 78.0, color: 0x8dc1dc, closeDistF: 1.75, type: "Electoral Democracy" },
  "AUS": { label: "Australia (澳洲) 🇦🇺", name: "Australia", lat: -25.0, lon: 135.0, color: 0x2c7bb6, closeDistF: 2.38, type: "Full Democracy (Polyarchy)" },
  "BRA": { label: "Brazil (巴西) 🇧🇷", name: "Brazil", lat: -15.0, lon: -47.0, color: 0x8dc1dc, closeDistF: 2.43, type: "Electoral Democracy" },
  "CAN": { label: "Canada (加拿大) 🇨🇦", name: "Canada", lat: 56.0, lon: -106.0, color: 0x2c7bb6, closeDistF: 2.50, type: "Full Democracy (Polyarchy)" }
};


/* ---------------------------------------------------- */
/* 🔥 CRITICAL FIXES: Global Declarations and Handlers */
/* ---------------------------------------------------- */

// 1. Declare scene variables globally using 'let' so they can be assigned in init()
let container, camera, renderer, controls, globeGroup, scene;

// 2. Define the country select element variable here, before init() uses it
const countrySelect = document.getElementById("middlePowerSelector");

// Helper to trigger animation (Needed before init() uses it)
async function animateToSelectedCountry(countryName) {
    if (!countryName) return;

    const COUNTRY_DATA = MIDDLE_POWERS[countryName];
    const targetLoc = [COUNTRY_DATA.lat, COUNTRY_DATA.lon];
    
    // 🔥 Updated to use closeDistF
    const targetCloseDistance = RADIUS * COUNTRY_DATA.closeDistF; 

    const [lat, lon] = await findAndDrawCountryByName(countryName, COUNTRY_DATA.color);

    // Use the calculated target distance
    await animateCameraToLatLon(lat, lon, CAMERA_DISTANCE, targetCloseDistance, 1500);
}


/* ---------------------------------------------------- */
/* 🚀 CORE FUNCTIONS (latLonToVector3, easeInOutQuad, slerpUnit, etc.) */
/* ---------------------------------------------------- */
// ... (All core math and camera functions remain the same) ...
// NOTE: The function that returns the material should be defined before init() runs
async function createGlobeMaterial() {
  const texture = await new Promise((res, rej) => {
    new THREE.TextureLoader().load(
      SAT_TEXTURE_URL,
      t => {
        t.anisotropy = renderer.capabilities.getMaxAnisotropy();
        t.colorSpace = THREE.SRGBColorSpace;
        res(t);
      },
      undefined,
      rej
    );
  });
  return new THREE.MeshStandardMaterial({ map: texture });
}

/* ===== Core math utilities ===== */
/* Correct lat/lon → Cartesian (post‑Gemini working mapping) */
function latLonToVector3(lat, lon, radius = RADIUS) {
  const phi = THREE.MathUtils.degToRad(90 - lat);
  const theta = THREE.MathUtils.degToRad(lon);
  return new THREE.Vector3(
    radius * Math.sin(phi) * Math.cos(theta),  // X
    radius * Math.cos(phi),                    // Y
    -radius * Math.sin(phi) * Math.sin(theta)  // Z
  );
}

/* ========================================= */
/* 🚀 Core Animation Utilities (Math)        */
/* ========================================= */

// Easing function for smooth start/stop

function easeInOutQuad(t) {
  return t < 0.5 ? 2*t*t : 1 - Math.pow(-2*t + 2, 2) / 2;
}

// Spherical Linear Interpolation for smooth rotation along the globe surface
function slerpUnit(a, b, t) {
  const v0 = a.clone().normalize();
  const v1 = b.clone().normalize();
  let dot = THREE.MathUtils.clamp(v0.dot(v1), -1, 1);

  if (dot < -0.9995) {
    let orth = new THREE.Vector3(1,0,0).cross(v0);
    if (orth.lengthSq() < 1e-6) orth = new THREE.Vector3(0,1,0).cross(v0);
    orth.normalize();
    const angle = Math.PI * t;
    return v0.clone().multiplyScalar(Math.cos(angle))
      .add(orth.clone().multiplyScalar(Math.sin(angle))).normalize();
  }
  if (dot > 0.9995) return v0.clone().lerp(v1, t).normalize();

  const theta = Math.acos(dot);
  const sinTheta = Math.sin(theta);
  const s0 = Math.sin((1 - t) * theta) / sinTheta;
  const s1 = Math.sin(t * theta) / sinTheta;
  return v0.clone().multiplyScalar(s0).add(v1.clone().multiplyScalar(s1)).normalize();
}

/* ========================================= */
/* 🎥 High-Level Camera Controls             */
/* ========================================= */
// Immediately sets the camera to look at a point from a specific distance
function setCameraToLatLon(lat, lon, distance = CAMERA_DISTANCE) {
  const dir = latLonToVector3(lat, lon).normalize();
  camera.position.copy(dir.multiplyScalar(distance));
  camera.up.set(0, 1, 0);
  camera.lookAt(0, 0, 0);
}

// Animates camera rotation at a fixed distance (uses slerp)
function animateCameraTo(lat, lon, duration = 1500, distance = CAMERA_DISTANCE) {
  const startDir = camera.position.clone().normalize();
  const endDir = latLonToVector3(lat, lon).normalize();
  const startTime = performance.now();
  return new Promise(resolve => {
    function step(now) {
      const t = Math.min(1, (now - startTime) / duration);
      const e = easeInOutQuad(t);
      const dir = slerpUnit(startDir, endDir, e);
      camera.position.copy(dir.multiplyScalar(distance));
      camera.up.set(0, 1, 0);
      camera.lookAt(0, 0, 0);
      if (t < 1) requestAnimationFrame(step); else resolve();
    }
    requestAnimationFrame(step);
  });
}

// Animates a zoom in/out *without* changing direction (uses lerp on distance)
function animateCameraZoomTo(lat, lon, targetDistance = CLOSE_DISTANCE, duration = 1000) {
  const startDist = camera.position.length();
  const dir = latLonToVector3(lat, lon).normalize(); // correct direction
  const startTime = performance.now();

  return new Promise(resolve => {
    function step(now) {
      const t = Math.min(1, (now - startTime) / duration);
      const e = easeInOutQuad(t);
      const dist = THREE.MathUtils.lerp(startDist, targetDistance, e);
      camera.position.copy(dir.clone().multiplyScalar(dist));
      camera.up.set(0, 1, 0);
      camera.lookAt(0, 0, 0);
      if (t < 1) requestAnimationFrame(step); else resolve();
    }
    requestAnimationFrame(step);
  });
}

// Primary animation function: smooth rotation (slerp) AND zoom (lerp) simultaneously
function animateCameraToLatLon(lat, lon, fromDistance, toDistance, duration = 1500) {
  const startDir = camera.position.clone().normalize();
  const endDir = latLonToVector3(lat, lon).normalize();
  const startDist = camera.position.length();
  const startTime = performance.now();

  return new Promise(resolve => {
    function step(now) {
      const t = Math.min(1, (now - startTime) / duration);
      const e = easeInOutQuad(t);
      const dir = slerpUnit(startDir, endDir, e);
      const dist = THREE.MathUtils.lerp(fromDistance, toDistance, e);
      camera.position.copy(dir.multiplyScalar(dist));
      camera.up.set(0, 1, 0);
      camera.lookAt(0, 0, 0);
      if (t < 1) requestAnimationFrame(step); else resolve();
    }
    requestAnimationFrame(step);
  });
}


/* ===== Helpers ===== */
function drawMarker(lat, lon, color = 0xff0000, size = 2) {
  // Small sphere as marker
  const markerGeometry = new THREE.SphereGeometry(size, 16, 16);
  const markerMaterial = new THREE.MeshBasicMaterial({ color });
  const marker = new THREE.Mesh(markerGeometry, markerMaterial);

  // Convert lat/lon to 3D coordinates on the globe
  const phi = (90 - lat) * (Math.PI / 180);
  const theta = (lon + 180) * (Math.PI / 180);

  marker.position.set(
    RADIUS * Math.sin(phi) * Math.cos(theta),
    RADIUS * Math.cos(phi),
    RADIUS * Math.sin(phi) * Math.sin(theta)
  );

  globeGroup.add(marker);
  return marker;
}

function addMarker(lat, lon, color = 0xff0000) {
  const markerGeo = new THREE.SphereGeometry(2, 16, 16);
  const markerMat = new THREE.MeshBasicMaterial({ color });
  const marker = new THREE.Mesh(markerGeo, markerMat);
  marker.position.copy(latLonToVector3(lat, lon, RADIUS * 1.01));
  globeGroup.add(marker);
}

/* Draw GeoJSON feature boundaries on the globe surface */
function drawBoundary(geojson, options = {}) {
  const color = options.color ?? 0xffcc00;
  const offset = options.offset ?? 1.002;
  const linewidth = options.linewidth !== undefined ? options.linewidth : 0.002;

  const group = new THREE.Group();
  const rings = [];

  (geojson.features || []).forEach(f => {
    const g = f.geometry;
    if (!g) return;
    if (g.type === "Polygon") {
      rings.push(...g.coordinates);
    } else if (g.type === "MultiPolygon") {
      g.coordinates.forEach(polygon => rings.push(...polygon));
    }
  });

  rings.forEach(ring => {
    const pts = [];
    ring.forEach(([lon, lat]) => {
      const v = latLonToVector3(lat, lon, RADIUS * offset);
      pts.push(v.x, v.y, v.z);
    });

    const geo = new LineGeometry();
    geo.setPositions(pts);

    const mat = new LineMaterial({
      color,
      linewidth,
      dashed: false
    });
    mat.resolution.set(window.innerWidth, window.innerHeight);

    const line = new Line2(geo, mat);
    line.computeLineDistances();
    group.add(line);
  });

  globeGroup.add(group);
  return group;
}

/* Find a country by its ISO code, draw its boundary, and return its centroid [lat, lon] */
async function findAndDrawCountryByISO3(isoCode3, boundaryColor = 0x00ffff) {
  const resp = await fetch(TOPO_URL);              // world.geojson
  const countries = await resp.json();             // FeatureCollection

  // Expect MIDDLE_POWERS keys or fields to provide ISO3
  const countryData = MIDDLE_POWERS[isoCode3] || Object.values(MIDDLE_POWERS).find(c => c.iso3 === isoCode3);
  if (!countryData) return null;

  // Prefer ISO3 id match
  let feature = countries.features.find(f => f.id === (countryData.iso3 || isoCode3));

  // Fallback to name if ISO3 not present or mismatch
  if (!feature && countryData.name) {
    feature = countries.features.find(f => f.properties?.name === countryData.name);
  }

  if (feature) {
    drawBoundary(
      { type: "FeatureCollection", features: [feature] },
      { linewidth: 1.5, color: boundaryColor, offset: 1.003 } // use a visible linewidth first
    );
    const c = d3.geoCentroid(feature); // [lon, lat]
    return [c[1], c[0]];               // [lat, lon]
  }

  // Fallback to provided lat/lon if feature not found
  if (countryData.lat != null && countryData.lon != null) {
    return [countryData.lat, countryData.lon];
  }
  return null;
}



async function findAndDrawCountryByName(isoCode, boundaryColor = 0x00ffff) {
  const resp = await fetch(TOPO_URL);        // TOPO_URL points to world.geojson
  const countries = await resp.json();       // already a FeatureCollection

  const countryData = MIDDLE_POWERS[isoCode];

  console.log("isoCode:", isoCode);
  console.log("countryData:", countryData);

  if (!countryData) return null;

  const feature = countries.features.find(
    f => f.properties.name === countryData.name
  );

  if (feature) {
    drawBoundary(
      { type: "FeatureCollection", features: [feature] },
      { linewidth: 0.002, color: boundaryColor, offset: 1.003 }
    );
    const c = d3.geoCentroid(feature); // [lon, lat]
    return [c[1], c[0]];               // return [lat, lon]
  }

  // fallback if not found
  return [countryData.lat, countryData.lon];
}




/* ---------------------------------------------------- */
/* 🖱️ Widget and Event Handlers (Setup) */
/* ---------------------------------------------------- */

function setupUI() {
    const countrySelect = document.getElementById("middlePowerSelector");
    if (!countrySelect) {
        console.warn("middlePowerSelector element not found.");
        return;
    }

    // 1. Clear and Add Placeholder Option
    countrySelect.innerHTML = '';
    const placeholderOption = document.createElement('option');
    placeholderOption.value = "";
    placeholderOption.textContent = "Select a Middle Power...";
    placeholderOption.disabled = true;
    placeholderOption.selected = true;
    countrySelect.appendChild(placeholderOption);

    // 2. Get and explicitly sort the ISO codes
    const sortedIsoCodes = Object.keys(MIDDLE_POWERS).sort();

    // 3. Populate with sorted data (ISO code as value, Label as text)
    for (const isoCode of sortedIsoCodes) { 
        const countryData = MIDDLE_POWERS[isoCode];
        const option = document.createElement('option');
        
        // VALUE is the ISO code (e.g., "AR")
        option.value = isoCode;
        
        // TEXT is the user-friendly label (e.g., "Argentina (阿根廷) 🇦🇷")
        option.textContent = countryData.label; 
        countrySelect.appendChild(option);
    }

    // 4. Set Event Handler
    countrySelect.onchange = (e) => {
        // e.target.value will be the ISO code (e.g., "PL")
        animateToSelectedCountry(e.target.value); 
    };
    
    // 5. Set initial selection state in the UI for the animation sequence
    countrySelect.value = "PL"; 
}

/* ---------------------------------------------------- */
/* 🔄 SCENE & APP LIFECYCLE */
/* ---------------------------------------------------- */

// Resize & render
function onResize() {
    // These variables are globally defined and assigned in init()
    if (!container) return; 
    const w = container.clientWidth, h = container.clientHeight;
    camera.aspect = w / h;
    camera.updateProjectionMatrix();
    renderer.setSize(w, h);
}
window.addEventListener("resize", onResize);

function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
}


async function drawAllCountries(boundaryColor = 0x00ffff) {
  const resp = await fetch(TOPO_URL);
  const countries = await resp.json(); // already a FeatureCollection

  drawBoundary(countries, {
    linewidth: 0.002,
    color: boundaryColor,
    offset: 1.003
  });

  return countries;
}


/* ===== Init & sequence (The main setup logic) ===== */

async function init() {
    // 1. DOM/RENDERER SETUP
    container = document.getElementById("globe");
    if (!container) {
        console.error("Error: Could not find HTML element with ID 'globe'. Aborting initialization.");
        return;
    }
    
    // Assign scene components to the global 'let' variables
    scene = new THREE.Scene(); // 🔥 FIX 2: Create scene first
    scene.background = new THREE.Color(0x02040a);

    camera = new THREE.PerspectiveCamera(45, container.clientWidth/container.clientHeight, 0.1, 5000);

    // Create RENDERER before calling functions that use it (like createGlobeMaterial)
    renderer = new THREE.WebGLRenderer({ antialias: true }); 
    renderer.setSize(container.clientWidth, container.clientHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    container.appendChild(renderer.domElement);

    controls = new OrbitControls(camera, renderer.domElement);
    controls.enablePan = false;
    controls.enableDamping = true;
    controls.minDistance = RADIUS * 1.05;
    controls.maxDistance = RADIUS * 6;
    
    // 2. LIGHTING SETUP
    scene.add(new THREE.AmbientLight(0xffffff, 1.2)); 
    // Correctly define and add the DirectionalLight instance:
    const dirLight = new THREE.DirectionalLight(0xffffff, 0.8); 
    dirLight.position.set(5, 10, 7);
    scene.add(dirLight); // 🔥 FIX: Add the light object itself
    
    // 3. GLOBE GROUP SETUP 
    globeGroup = new THREE.Group(); 
    scene.add(globeGroup);

    // 4. ASYNC GLOBE/TOPO LOADING
    const mat = await createGlobeMaterial(); // Await ensures renderer is available
    // 🔥 FIX: Define sphereGeometry locally before use
    const sphereGeometry = new THREE.SphereGeometry(RADIUS, 64, 64); 
    globeGroup.add(new THREE.Mesh(sphereGeometry, mat));

    
    // 5. ASYNC GLOBE/BOUNDARY LOADING
    const countries = await drawAllCountries(0x888888); // draw all outlines in gray

    const centroids = {};
    for (const iso3 in MIDDLE_POWERS) {
      const country = MIDDLE_POWERS[iso3];

      if (iso3 === "SGP") {
        // Singapore: draw a marker instead of polygon
        drawMarker(country.lat, country.lon, country.color, 3);
        centroids[iso3] = { lat: country.lat, lon: country.lon };
      } else {
        const [cLat, cLon] = await findAndDrawCountryByISO3(iso3, country.color);
        centroids[iso3] = { lat: cLat, lon: cLon };
      }

    }


    // 6. INITIAL ANIMATION SEQUENCE (Refactored to use ISO3 codes)
    const twn = centroids["TWN"];
    const pol = centroids["POL"];

    // 🔥 Updated to use closeDistF
    const TWN_CLOSE_DIST = RADIUS * MIDDLE_POWERS["TWN"].closeDistF;
    const POL_CLOSE_DIST = RADIUS * MIDDLE_POWERS["POL"].closeDistF;

    // Set starting camera position at Taiwan's correct distance
    setCameraToLatLon(twn.lat, twn.lon, TWN_CLOSE_DIST);

    // Sequence: Taiwan (Zoom out to default CAMERA_DISTANCE)
    await animateCameraToLatLon(twn.lat, twn.lon, TWN_CLOSE_DIST, CAMERA_DISTANCE, 800);

    // Sequence: Poland (Fly + Zoom in to Poland's specific distance)
    await animateCameraToLatLon(pol.lat, pol.lon, CAMERA_DISTANCE, POL_CLOSE_DIST, 1500);

    // Update selector to match ISO3 key
    countrySelect.value = "TWN";
    
    // 6. START LOOP
    animate();
}

// ------------------------------------------------------------------
// 🔥 THE CRITICAL FIX: Ensure Init runs AFTER the DOM is ready
window.addEventListener('DOMContentLoaded', () => {
    // 1. Set up the UI (Populate Selector and set handlers)
    setupUI(); 
    
    // 2. Initialize the 3D scene and start the animation
    init();
});




// ------------------------------------------------------------------

</script>