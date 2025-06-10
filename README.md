
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Simulasi Gerak Parabola Interaktif</title>
<style>
  /* Reset and base */
  * {
    box-sizing: border-box;
  }
  body {
    margin: 0;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: #f9fafb;
    color: #222;
    display: flex;
    flex-direction: column;
    min-height: 100vh;
  }
  header {
    padding: 1rem;
    background-color: #222;
    color: #fff;
    text-align: center;
    font-weight: 700;
    font-size: 1.25rem;
    user-select: none;
  }
  main {
    flex: 1;
    display: flex;
    flex-direction: column;
    max-width: 960px;
    margin: 1rem auto;
    width: 90vw;
  }
  .controls {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    margin-bottom: 1rem;
    justify-content: space-around;
  }
  .control-group {
    display: flex;
    flex-direction: column;
    min-width: 120px;
  }
  label {
    font-size: 0.9rem;
    margin-bottom: 0.3rem;
    color: #444;
  }
  input[type="number"] {
    padding: 0.4rem 0.6rem;
    font-size: 1rem;
    border: 1.5px solid #ccc;
    border-radius: 4px;
    transition: border-color 0.3s ease;
  }
  input[type="number"]:focus {
    border-color: #0078d7;
    outline: none;
  }
  .buttons {
    display: flex;
    gap: 1rem;
    justify-content: center;
    margin-bottom: 1rem;
  }
  button {
    padding: 0.5rem 1.2rem;
    font-size: 1rem;
    font-weight: 600;
    color: #fff;
    background-color: #0078d7;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    user-select: none;
    transition: background-color 0.3s ease;
  }
  button:hover:not(:disabled) {
    background-color: #005fa3;
  }
  button:disabled {
    background-color: #aaa;
    cursor: not-allowed;
  }
  .info {
    display: flex;
    justify-content: space-around;
    font-weight: 600;
    font-size: 1rem;
    color: #333;
    margin-bottom: 0.3rem;
    flex-wrap: wrap;
    gap: 1.5rem;
  }
  canvas {
    border: 1px solid #ccc;
    border-radius: 6px;
    background: white;
  }
  /* Responsive adjustments */
  @media (max-width: 480px) {
    .controls {
      flex-direction: column;
      align-items: center;
    }
    .info {
      flex-direction: column;
      gap: 0.5rem;
      font-size: 0.95rem;
    }
  }
</style>
</head>
<body>
<header>
  Simulasi Gerak Parabola Interaktif
</header>
<main>
  <section class="controls" aria-label="Pengaturan simulasi">
    <div class="control-group">
      <label for="inputV0">Kecepatan Awal (v₀) m/s</label>
      <input type="number" id="inputV0" min="0" max="100" step="0.1" value="30" aria-describedby="descV0" />
      <small id="descV0">Nilai antara 0 dan 100</small>
    </div>
    <div class="control-group">
      <label for="inputTheta">Sudut Peluncuran (θ)°</label>
      <input type="number" id="inputTheta" min="0" max="90" step="0.1" value="45" aria-describedby="descTheta" />
      <small id="descTheta">Nilai antara 0° dan 90°</small>
    </div>
    <div class="control-group">
      <label for="inputH">Tinggi Awal (h) m</label>
      <input type="number" id="inputH" min="0" max="100" step="0.1" value="0" aria-describedby="descH" />
      <small id="descH">Nilai antara 0 dan 100</small>
    </div>
  </section>
  <section class="buttons" aria-label="Kontrol simulasi">
    <button id="startBtn" type="button">Start</button>
    <button id="pauseBtn" type="button" disabled>Pause</button>
    <button id="resetBtn" type="button" disabled>Reset</button>
  </section>
  <section class="info" aria-live="polite" aria-atomic="true" aria-label="Informasi hasil simulasi">
    <div>Jarak Horizontal Maksimum: <span id="maxDistance">0.00</span> m</div>
    <div>Waktu Total: <span id="totalTime">0.00</span> s</div>
    <div>Ketinggian Maksimum: <span id="maxHeight">0.00</span> m</div>
  </section>
  <section aria-label="Area simulasi dan grafik lintasan">
    <!-- p5.js canvas will be here -->
  </section>
</main>
<!-- p5.js library from CDN -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/p5.min.js"></script>
<script>
/**
 * Simulasi Interaktif Gerak Parabola menggunakan p5.js
 * 
 * Pengguna dapat mengatur:
 * - Kecepatan Awal (v₀) dalam meter per detik (m/s)
 * - Sudut Peluncuran (θ) dalam derajat (°)
 * - Tinggi Awal (h) dalam meter (m)
 * 
 * Fitur simulasi:
 * - Visualisasi lintasan parabola dalam koordinat kartesian (dengan sumbu y terbalik sesuai fisika)
 * - Animasi real-time lintasan dan vektor kecepatan
 * - Kontrol Start, Pause, Reset
 * - Tampilkan nilai numerik:
 *   * Jarak horizontal maksimum
 *   * Waktu total penerbangan
 *   * Ketinggian maksimum
 * 
 * UI dibuat responsif dan minimalis.
 */

let startBtn, pauseBtn, resetBtn;
let inputV0, inputTheta, inputH;
let maxDistanceSpan, totalTimeSpan, maxHeightSpan;

let canvasWidth, canvasHeight;

// Konstanta gravitasi (m/s^2)
const g = 9.81;

// Variabel simulasi
let isRunning = false;
let isPaused = false;
let t = 0; // waktu simulasi dalam detik
let dt = 0.02; // step waktu animasi

// Parameter awal (diset dari input)
let v0 = 30;
let theta = 45; // derajat
let h0 = 0;

// Variabel perhitungan pendukung (hasil perhitungan awal)
let v0x, v0y;
let totalFlightTime;
let maxHeight;
let maxDistance;

// Dimensi dunia fisik yang divisualisasikan (dalam meter)
// Saya akan menyesuaikan skala otomatis tergantung maxDistance dan maxHeight
let scaleX = 10; // px per meter
let scaleY = 10;

// Posisi origin di canvas (dalam pixel), di mana y=0 (permukaan tanah)
let originX, originY;

// Posisi objek saat ini (meter)
let posX, posY;

// Fungsi untuk menghitung total waktu penerbangan dan jarak maksimum untuk parabola dengan tinggi awal
function calculateTrajectoryParameters(v0, theta, h0) {
  // Konversi sudut ke radian
  let rad = radians(theta);

  // Komponen kecepatan awal
  let vx = v0 * cos(rad);
  let vy = v0 * sin(rad);

  // Waktu total penerbangan dari persamaan kuadrat (y(t) = 0)
  // y(t) = h0 + vy * t - 0.5 * g * t^2
  // Mencari akar persamaan: 0 = h0 + vy*t - 0.5*g*t^2
  // 0.5*g*t^2 - vy*t - h0 = 0
  let a = 0.5 * g;
  let b = -vy;
  let c = -h0;

  let discriminant = b*b - 4*a*c;
  let flightTime;
  if(discriminant < 0){
    // Tidak mencapai tanah (seharusnya tidak mungkin)
    flightTime = 0;
  } else {
    // Solusi positif akar kuadrat
    let t1 = (-b + sqrt(discriminant)) / (2*a);
    let t2 = (-b - sqrt(discriminant)) / (2*a);
    flightTime = max(t1, t2);
  }

  // Ketinggian maksimum
  let tMaxHeight = vy / g; // waktu saat kecepatan vertikal = 0
  let yMax = h0 + vy*tMaxHeight - 0.5*g*tMaxHeight*tMaxHeight;

  // Jarak horizontal maksimum (posX saat y=0)
  let xMax = vx * flightTime;

  return {
    flightTime: flightTime,
    maxHeight: yMax,
    maxDistance: xMax,
    vx: vx,
    vy: vy
  };
}

/**
 * Konversi derajat ke radian
 * @param {Number} deg Derajat
 * @returns {Number} Radian
 */
function radians(deg) {
  return deg * Math.PI / 180;
}

/**
 * Konversi radian ke derajat
 * @param {Number} rad Radian
 * @returns {Number} Derajat
 */
function degrees(rad) {
  return rad * 180 / Math.PI;
}

function setup() {
  // Buat canvas dan letakkan di dalam main section di akhir controls
  canvasWidth = 900;
  canvasHeight = 500;
  let cvs = createCanvas(canvasWidth, canvasHeight);
  cvs.parent(document.querySelector('main section[aria-label="Area simulasi dan grafik lintasan"]'));

  // Origin di kiri bawah (untuk koordinat kartesian)
  originX = 60;  // padding dari kiri
  originY = height - 60;  // padding dari bawah

  // Ambil elemen input dan tombol
  inputV0 = document.getElementById('inputV0');
  inputTheta = document.getElementById('inputTheta');
  inputH = document.getElementById('inputH');

  startBtn = document.getElementById('startBtn');
  pauseBtn = document.getElementById('pauseBtn');
  resetBtn = document.getElementById('resetBtn');

  maxDistanceSpan = document.getElementById('maxDistance');
  totalTimeSpan = document.getElementById('totalTime');
  maxHeightSpan = document.getElementById('maxHeight');

  // Event listeners tombol
  startBtn.addEventListener('click', startSimulation);
  pauseBtn.addEventListener('click', pauseSimulation);
  resetBtn.addEventListener('click', resetSimulation);

  // Start dengan set parameter default
  setParametersFromInput();
  calculateAndDisplay();

  frameRate(60);
}

function draw() {
  background(250);

  drawAxes();
  drawLabels();

  if (isRunning) {
    // Update posisi objek
    t += dt;
    if (t > totalFlightTime) {
      // Berhenti animasi saat sudah di tanah
      t = totalFlightTime;
      isRunning = false;
      pauseBtn.disabled = true;
      startBtn.disabled = false;
    }
    updatePosition(t);
  }

  // Gambar lintasan parabola penuh sebagai garis putus-putus (preview)
  drawParabolaPath();

  // Gambar posisi bola/objek pada waktu t
  if(t >= 0){
    drawProjectile();
    drawVelocityVector();
  }
}

// Menggambar sumbu koordinat
function drawAxes() {
  stroke(120);
  strokeWeight(1);
  fill(120);
  textSize(12);

  // Sumbu X
  line(originX, originY, width - 10, originY);
  // Panah ujung x
  line(width - 15, originY - 5, width - 10, originY);
  line(width - 15, originY + 5, width - 10, originY);
  textAlign(RIGHT, TOP);
  text('Jarak (m)', width - 15, originY + 8);

  // Sumbu Y
  line(originX, originY, originX, 10);
  // Panah ujung y
  line(originX - 5, 15, originX, 10);
  line(originX + 5, 15, originX, 10);
  textAlign(LEFT, BOTTOM);
  text('Ketinggian (m)', originX + 8, 15);

  // Grid garis bantu (garis horizontal tiap 10 m)
  stroke(200);
  strokeWeight(0.5);
  textAlign(RIGHT, CENTER);
  let gridStepY = 10;  // 10 meters per grid
  for(let y = 0; y <= maxHeight + 20; y += gridStepY){
    let py = originY - y * scaleY;
    if(py < 10) break;
    line(originX, py, width-10, py);
    noStroke();
    fill(180);
    text(y.toFixed(0), originX - 5, py);
    stroke(200);
  }
  // Garis vertikal tiap 10 m
  textAlign(CENTER, TOP);
  for(let x = 0; x <= maxDistance + 20; x += gridStepY){
    let px = originX + x * scaleX;
    if(px > width-10) break;
    line(px, originY, px, 10);
    noStroke();
    fill(180);
    text(x.toFixed(0), px, originY + 5);
    stroke(200);
  }
}

// Label tambahan (misal judul grafik, parameter)
function drawLabels() {
  fill(80);
  noStroke();
  textSize(14);
  textAlign(LEFT, TOP);
  // Parameter aktual disamping kanan bawah
  text(`v₀ = ${v0} m/s`, width - 180, 20);
  text(`θ = ${theta}°`, width - 180, 40);
  text(`h = ${h0} m`, width - 180, 60);
  text(`t = ${t.toFixed(2)} s`, width - 180, 80);
}

// Fungsi menghitung posisi x,y pada waktu t
function positionAtTime(t) {
  let x = v0x * t;
  let y = h0 + v0y * t - 0.5 * g * t * t;
  return {x: x, y: y};
}

// Fungsi update posisi objek saat waktu t
function updatePosition(t) {
  let pos = positionAtTime(t);
  posX = pos.x;
  posY = max(pos.y, 0); // Jangan tampilkan dibawah tanah
}

// Menggambar lintasan parabola penuh
function drawParabolaPath() {
  noFill();
  stroke('#0078d7');
  strokeWeight(2);
  drawingContext.setLineDash([5, 10]);

  beginShape();
  for(let time = 0; time <= totalFlightTime; time += dt*2){
    let pos = positionAtTime(time);
    let px = originX + pos.x * scaleX;
    let py = originY - pos.y * scaleY;
    vertex(px, py);
  }
  endShape();

  drawingContext.setLineDash([]); // reset dashed line
}

// Menggambar proyektil (bola)
function drawProjectile() {
  let px = originX + posX * scaleX;
  let py = originY - posY * scaleY;

  fill('#d1495b');
  stroke('#a83248');
  strokeWeight(1.5);
  ellipse(px, py, 16, 16);
}

// Menggambar vektor kecepatan di posisi saat ini
function drawVelocityVector() {
  if(t > totalFlightTime) return;

  // Kecepatan vertikal dan horizontal saat t
  let vx = v0x;
  let vy = v0y - g * t;
  let speed = sqrt(vx*vx + vy*vy);

  // Lokasi titik objek di canvas
  let px = originX + posX * scaleX;
  let py = originY - posY * scaleY;

  // Vektor kecepatan diskalakan agar terlihat proporsional
  const vectorScale = 15; // Skala panjang vektor yang digambar

  // Komponen vektor kecepatan dalam pixel
  let vxPix = (vx / speed) * vectorScale;
  let vyPix = (-vy / speed) * vectorScale; // negatif karena koordinat canvas y ke bawah

  stroke('#47b8e0');
  strokeWeight(3);
  fill('#47b8e0');
  // Garis vektor
  line(px, py, px + vxPix, py + vyPix);

  // Panah ujung vektor
  push();
  translate(px + vxPix, py + vyPix);
  let angle = atan2(vyPix, vxPix);
  rotate(angle);
  triangle(0, 0, -6, 3, -6, -3);
  pop();

  // Teks kecepatan
  noStroke();
  fill('#004a70');
  textSize(12);
  textAlign(LEFT, BOTTOM);
  text(`|v| = ${speed.toFixed(2)} m/s`, px + vxPix + 5, py + vyPix);
}

// Update parameter dari input elemen dan hitung ulang variabel fisika serta skala gambar
function setParametersFromInput() {
  let valV0 = parseFloat(inputV0.value);
  let valTheta = parseFloat(inputTheta.value);
  let valH = parseFloat(inputH.value);

  // Validasi input, beri nilai default jika tidak valid
  v0 = isNaN(valV0) || valV0 < 0 ? 30 : valV0;
  theta = isNaN(valTheta) ? 45 : constrain(valTheta, 0, 90);
  h0 = isNaN(valH) || valH < 0 ? 0 : valH;

  // Hitung variabel kecepatan awal
  let params = calculateTrajectoryParameters(v0, theta, h0);
  totalFlightTime = params.flightTime;
  maxHeight = params.maxHeight;
  maxDistance = params.maxDistance;
  v0x = params.vx;
  v0y = params.vy;

  // Estimasi skala px/m agar lintasan muat di canvas dengan margin
  const margin = 80;

  scaleX = (canvasWidth - margin - originX) / (maxDistance + 10);  // +10 meter margin
  scaleY = (originY - margin) / (maxHeight + 10);

  // Batasi scale agar tidak terlalu besar atau kecil
  scaleX = constrain(scaleX, 5, 40);
  scaleY = constrain(scaleY, 5, 40);
}

// Update tampilan nilai hasil perhitungan
function calculateAndDisplay() {
  maxDistanceSpan.textContent = maxDistance.toFixed(2);
  totalTimeSpan.textContent = totalFlightTime.toFixed(2);
  maxHeightSpan.textContent = maxHeight.toFixed(2);
}

// Fungsi untuk membatasi nilai (sebagai pengganti p5.js constrain saat JS asli)
function constrain(val, low, high) {
  return Math.min(Math.max(val, low), high);
}

/**
 * Event handler start simulasi
 */
function startSimulation() {
  // Jika simulasi belum berjalan, set parameter dari input
  if(!isRunning){
    setParametersFromInput();
    calculateAndDisplay();
    t = 0;
  }
  isRunning = true;
  isPaused = false;
  startBtn.disabled = true;
  pauseBtn.disabled = false;
  resetBtn.disabled = false;

  // Disable input saat animasi berjalan
  inputV0.disabled = true;
  inputTheta.disabled = true;
  inputH.disabled = true;
}

/**
 * Event handler pause simulasi
 */
function pauseSimulation() {
  if(isRunning){
    isRunning = false;
    isPaused = true;
    startBtn.disabled = false;
    pauseBtn.disabled = true;
  }
}

/**
 * Event handler reset simulasi
 */
function resetSimulation() {
  isRunning = false;
  isPaused = false;
  t = 0;
  posX = 0;
  posY = h0;

  // Enable input kembali
  inputV0.disabled = false;
  inputTheta.disabled = false;
  inputH.disabled = false;

  // Update parameter dan tampilkan hasil baru
  setParametersFromInput();
  calculateAndDisplay();

  maxDistanceSpan.textContent = '0.00';
  totalTimeSpan.textContent = '0.00';
  maxHeightSpan.textContent = '0.00';

  startBtn.disabled = false;
  pauseBtn.disabled = true;
  resetBtn.disabled = true;
}
</script>
</body>
</html>

