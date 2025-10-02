<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Album Kenangan</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: radial-gradient(circle at top, #0f2027, #203a43, #2c5364);
      margin: 0;
      padding: 0;
      text-align: center;
      overflow-x: hidden;
      color: white;
    }
    header {
      padding: 20px;
      font-size: 32px;
      font-weight: bold;
      text-shadow: 2px 2px 6px rgba(0,0,0,0.6);
    }
    #controls {
      margin: 20px;
    }
    #controls input {
      display: none;
    }
    #controls label, #musicControls button {
      background: #6a11cb;
      color: #fff;
      border: none;
      padding: 10px 20px;
      margin: 5px;
      border-radius: 25px;
      cursor: pointer;
      font-size: 14px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.2);
      transition: 0.3s;
    }
    #controls label:hover, #musicControls button:hover {
      background: #2575fc;
    }
    .gallery {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
      gap: 20px;
      padding: 20px;
    }
    .photo-frame {
      position: relative;
      padding: 12px;
      background: #fff;
      border-radius: 15px;
      box-shadow: 0 8px 20px rgba(0,0,0,0.2);
      border: 6px solid #6a11cb;
      animation: fadeInUp 1s ease both;
    }
    .photo-frame img {
      width: 100%;
      height: auto;
      border-radius: 10px;
      display: block;
    }
    .delete-btn {
      position: absolute;
      top: 5px;
      right: 5px;
      background: rgba(255,0,0,0.8);
      color: #fff;
      border: none;
      border-radius: 50%;
      width: 28px;
      height: 28px;
      cursor: pointer;
      font-size: 16px;
      line-height: 28px;
      text-align: center;
      transition: 0.3s;
    }
    .delete-btn:hover {
      background: red;
    }
    @keyframes fadeInUp {
      from { opacity: 0; transform: translateY(40px); }
      to { opacity: 1; transform: translateY(0); }
    }
    /* Popup overlay */
    #startOverlay {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.85);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 9999;
      flex-direction: column;
      color: white;
      text-align: center;
      padding: 20px;
    }
    #startOverlay button {
      margin-top: 20px;
      background: #6a11cb;
      padding: 12px 25px;
      font-size: 16px;
      border: none;
      border-radius: 25px;
      cursor: pointer;
      color: white;
      box-shadow: 0 4px 10px rgba(0,0,0,0.3);
    }
    #startOverlay button:hover {
      background: #2575fc;
    }
    /* Pesan Ucapan */
    .message {
      max-width: 800px;
      margin: 30px auto;
      padding: 20px;
      background: #fff;
      border-radius: 15px;
      box-shadow: 0 6px 20px rgba(0,0,0,0.15);
      font-size: 18px;
      line-height: 1.6;
      color: black;
      animation: slideUp 2s ease;
    }
    @keyframes slideUp {
      from { opacity: 0; transform: translateY(50px); }
      to { opacity: 1; transform: translateY(0); }
    }
    /* Canvas Effects */
    #confettiCanvas, #snowCanvas, #starsCanvas {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      pointer-events: none;
    }
    #starsCanvas { z-index: 100; }
    #snowCanvas { z-index: 200; }
    #confettiCanvas { z-index: 300; }
  </style>
</head>
<body>
  <!-- Overlay Mulai -->
  <div id="startOverlay">
    <h1>💖 Selamat Datang di Album Kenangan 💖</h1>
    <p>Klik tombol di bawah untuk memulai & mendengarkan musik</p>
    <button onclick="startApp()">Mulai</button>
  </div>

  <canvas id="starsCanvas"></canvas>
  <canvas id="snowCanvas"></canvas>
  <canvas id="confettiCanvas"></canvas>

  <header>📸 Album Kenangan 📸</header>

  <!-- Pesan ucapan muncul sebelum foto -->
  <div class="message">
    <p>
      Kami kelas F mengucapkan <b>selamat</b> kepada kamu atas pencapaiannya 💐<br><br>
      Terima kasih atas waktunya dari kemarin telah bersama kami. Kami juga mengucapkan maaf apabila ada kesalahan yang disengaja maupun tidak 🙏<br><br>
      Semoga kita semua dapat meraih mimpi kita masing-masing 🌟<br><br>
      Harapan kami agar tetap berkomunikasi dengan baik ❤️
    </p>
  </div>

  <div id="controls">
    <input type="file" id="photoUpload" accept="image/*" multiple>
    <label for="photoUpload">➕ Tambah Foto</label>
  </div>

  <div id="musicControls" style="display:none;">
    <button onclick="toggleMusic()">⏯️ Play/Pause Musik</button>
  </div>

  <div class="gallery" id="gallery"></div>

  <!-- Musik -->
  <audio id="bgMusic" loop>
    <source src="kenangan-terindah.mp3" type="audio/mpeg">
    Browser kamu tidak mendukung audio.
  </audio>

  <script>
    const uploadInput = document.getElementById('photoUpload');
    const gallery = document.getElementById('gallery');
    const bgMusic = document.getElementById('bgMusic');

    function startApp() {
      document.getElementById('startOverlay').style.display = 'none';
      document.getElementById('musicControls').style.display = 'block';
      bgMusic.play().catch(e => console.log("Autoplay dicegah:", e));
      startConfetti();
      setTimeout(stopConfetti, 5000);
    }

    function toggleMusic() {
      if (bgMusic.paused) {
        bgMusic.play();
      } else {
        bgMusic.pause();
      }
    }

    window.onload = function() {
      const savedPhotos = JSON.parse(localStorage.getItem("photos")) || [];
      savedPhotos.forEach(src => addPhotoToGallery(src));
      startSnowfall();
      startStars();
    }

    uploadInput.addEventListener('change', function(event) {
      const files = event.target.files;
      for (const file of files) {
        const reader = new FileReader();
        reader.onload = function(e) {
          addPhotoToGallery(e.target.result);
          savePhoto(e.target.result);
        }
        reader.readAsDataURL(file);
      }
    });

    function addPhotoToGallery(src) {
      const frame = document.createElement('div');
      frame.className = 'photo-frame';
      frame.innerHTML = `
        <img src="${src}" alt="Foto">
        <button class="delete-btn" onclick="deletePhoto(this, '${src}')">×</button>
      `;
      gallery.appendChild(frame);
    }

    function savePhoto(src) {
      let savedPhotos = JSON.parse(localStorage.getItem("photos")) || [];
      savedPhotos.push(src);
      localStorage.setItem("photos", JSON.stringify(savedPhotos));
    }

    function deletePhoto(btn, src) {
      btn.parentElement.remove();
      let savedPhotos = JSON.parse(localStorage.getItem("photos")) || [];
      savedPhotos = savedPhotos.filter(photo => photo !== src);
      localStorage.setItem("photos", JSON.stringify(savedPhotos));
    }

    /* Confetti */
    const confettiCanvas = document.getElementById("confettiCanvas");
    const ctx = confettiCanvas.getContext("2d");
    confettiCanvas.width = window.innerWidth;
    confettiCanvas.height = window.innerHeight;
    let confetti = [];
    function Confetto() {
      this.x = Math.random() * confettiCanvas.width;
      this.y = Math.random() * confettiCanvas.height - confettiCanvas.height;
      this.size = Math.random() * 8 + 4;
      this.speed = Math.random() * 3 + 2;
      this.color = `hsl(${Math.random() * 360}, 70%, 60%)`;
    }
    function startConfetti() {
      confetti = [];
      for (let i = 0; i < 200; i++) confetti.push(new Confetto());
      animateConfetti();
    }
    function animateConfetti() {
      ctx.clearRect(0, 0, confettiCanvas.width, confettiCanvas.height);
      confetti.forEach(c => {
        c.y += c.speed;
        if (c.y > confettiCanvas.height) c.y = -10;
        ctx.fillStyle = c.color;
        ctx.fillRect(c.x, c.y, c.size, c.size);
      });
      confettiAnimation = requestAnimationFrame(animateConfetti);
    }
    function stopConfetti() {
      cancelAnimationFrame(confettiAnimation);
      ctx.clearRect(0, 0, confettiCanvas.width, confettiCanvas.height);
    }

    /* Snowfall */
    const snowCanvas = document.getElementById("snowCanvas");
    const snowCtx = snowCanvas.getContext("2d");
    snowCanvas.width = window.innerWidth;
    snowCanvas.height = window.innerHeight;
    let snowflakes = [];
    function Snowflake() {
      this.x = Math.random() * snowCanvas.width;
      this.y = Math.random() * snowCanvas.height;
      this.radius = Math.random() * 3 + 2;
      this.speedY = Math.random() * 2 + 1;
      this.speedX = Math.random() * 1 - 0.5;
    }
    function drawSnowflake(s) {
      snowCtx.beginPath();
      snowCtx.arc(s.x, s.y, s.radius, 0, Math.PI * 2);
      snowCtx.fillStyle = "white";
      snowCtx.fill();
    }
    function updateSnowflake(s) {
      s.y += s.speedY;
      s.x += s.speedX;
      if (s.y > snowCanvas.height) {
        s.y = -5;
        s.x = Math.random() * snowCanvas.width;
      }
    }
    function animateSnowfall() {
      snowCtx.clearRect(0, 0, snowCanvas.width, snowCanvas.height);
      snowflakes.forEach(s => {
        drawSnowflake(s);
        updateSnowflake(s);
      });
      requestAnimationFrame(animateSnowfall);
    }
    function startSnowfall() {
      for (let i = 0; i < 100; i++) snowflakes.push(new Snowflake());
      animateSnowfall();
    }

    /* Stars Twinkling */
    const starsCanvas = document.getElementById("starsCanvas");
    const starsCtx = starsCanvas.getContext("2d");
    starsCanvas.width = window.innerWidth;
    starsCanvas.height = window.innerHeight;
    let stars = [];
    function Star() {
      this.x = Math.random() * starsCanvas.width;
      this.y = Math.random() * starsCanvas.height;
      this.radius = Math.random() * 2;
      this.alpha = Math.random();
      this.fade = Math.random() * 0.02 + 0.005;
    }
    function drawStar(s) {
      starsCtx.beginPath();
      starsCtx.arc(s.x, s.y, s.radius, 0, Math.PI * 2);
      starsCtx.fillStyle = `rgba(255,255,255,${s.alpha})`;
      starsCtx.fill();
    }
    function updateStar(s) {
      s.alpha += s.fade;
      if (s.alpha <= 0 || s.alpha >= 1) s.fade = -s.fade;
    }
    function animateStars() {
      starsCtx.clearRect(0, 0, starsCanvas.width, starsCanvas.height);
      stars.forEach(s => {
        drawStar(s);
        updateStar(s);
      });
      requestAnimationFrame(animateStars);
    }
    function startStars() {
      for (let i = 0; i < 150; i++) stars.push(new Star());
      animateStars();
    }

    window.addEventListener("resize", () => {
      confettiCanvas.width = window.innerWidth;
      confettiCanvas.height = window.innerHeight;
      snowCanvas.width = window.innerWidth;
      snowCanvas.height = window.innerHeight;
      starsCanvas.width = window.innerWidth;
      starsCanvas.height = window.innerHeight;
    });
  </script>
</body>
</html>
