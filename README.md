<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8" />
  <title>วงล้อมหาภัยMS 🍻</title>
  <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@400;700&display=swap" rel="stylesheet" />
  <style>
    body {
      margin: 0;
      background-color: #1e3a8a;
      font-family: 'Kanit', sans-serif;
      color: white;
      text-align: center;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      height: 100vh;
      padding-top: 40px;
      overflow: hidden;
    }
    h1 {
      font-size: 500%;
      margin-bottom: 30px;
      text-shadow: 4px 4px #000;
    }
    #container {
      position: relative;
      width: 900px;
      height: 900px;
      margin-top: 30px;
      filter: drop-shadow(0 0 20px rgba(255,255,255,0.3));
    }
    canvas {
      width: 100%;
      height: 100%;
      background-color: white;
      border-radius: 50%;
      box-shadow: 0 0 30px rgba(255, 255, 255, 0.3);
    }
    .arrow {
      position: absolute;
      top: calc(50% - 450px + 80px);
      left: 50%;
      transform: translate(-50%, 0);
      width: 0;
      height: 0;
      border-left: 30px solid transparent;
      border-right: 30px solid transparent;
      border-bottom: 60px solid red;
      z-index: 10;
    }
    button {
      margin-top: 50px;
      padding: 30px 60px;
      font-size: 3rem;
      border: none;
      border-radius: 20px;
      background-color: #ef4444;
      color: white;
      cursor: pointer;
      transition: background-color 0.3s ease, transform 0.1s;
    }
    button:active {
      transform: scale(0.95);
    }
    #result {
      margin-top: 30px;
      font-size: 3.5rem;
      color: #fde047;
      font-weight: bold;
      text-shadow: 2px 2px black;
    }
    .shake {
      animation: shake 0.4s ease;
    }
    @keyframes shake {
      0% { transform: rotate(0deg); }
      25% { transform: rotate(2deg); }
      50% { transform: rotate(-2deg); }
      75% { transform: rotate(1.5deg); }
      100% { transform: rotate(0deg); }
    }
  </style>
</head>
<body>
  <h1>วงล้อมหาภัยMS 🍻</h1>
  <div id="container">
    <div class="arrow"></div>
    <canvas id="wheel" width="900" height="900"></canvas>
  </div>
  <button id="spinBtn" onclick="spin()">หมุนวงล้อ</button>
  <div id="result"></div>

  <!-- 🔊 เสียง -->
  <audio id="spin-sound" src="wow.mp3" preload="auto"></audio>
  <audio id="tick-sound" src="tick.mp3" preload="auto"></audio>

  <script>
    const canvas = document.getElementById("wheel");
    const ctx = canvas.getContext("2d");
    const radius = canvas.width / 2;
    const spinBtn = document.getElementById("spinBtn");

    const options = [
      "ดื่ม 1 แก้ว", "ให้ใครก็ได้ดื่ม", "คนถัดไปดื่ม", "คนละครึ่งกับเพื่อน",
      "คนก่อนหน้าดื่ม", "ถอด 1 ชิ้น", "ไม่ต้องดื่ม", "หมุนใหม่อีกที"
    ];

    const colors = ["#facc15", "#fb7185", "#34d399", "#60a5fa", "#c084fc", "#f97316", "#10b981", "#e879f9"];
    const slice = 2 * Math.PI / options.length;

    let rotation = 0;
    let isSpinning = false;

    function drawWheel(angleOffset = 0) {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      for (let i = 0; i < options.length; i++) {
        const start = i * slice + angleOffset;
        const end = start + slice;

        ctx.beginPath();
        ctx.moveTo(radius, radius);
        ctx.arc(radius, radius, radius, start, end);
        ctx.fillStyle = colors[i];
        ctx.fill();

        ctx.save();
        ctx.translate(radius, radius);
        ctx.rotate(start + slice / 2);
        ctx.textAlign = "right";
        ctx.fillStyle = "black";
        ctx.font = "bold 36px Kanit";
        ctx.fillText(options[i], radius - 20, 10);
        ctx.restore();
      }
    }

    // easing แบบนุ่มมากขึ้น (quart)
    function easeOutQuart(t) {
      return 1 - Math.pow(1 - t, 4);
    }

    function spin() {
      if (isSpinning) return;
      isSpinning = true;
      spinBtn.disabled = true;
      spinBtn.style.opacity = "0.7";
      document.getElementById("result").textContent = "";

      const duration = Math.floor(Math.random() * 180) + 360; // 360–540 เฟรม = 6–9 วิ
      const extraSpins = Math.floor(Math.random() * 3) + 5; // 5–7 รอบเต็ม
      const totalRotation = rotation + extraSpins * 2 * Math.PI + Math.random() * 2 * Math.PI;

      let frame = 0;
      const startRotation = rotation;
      let lastTickSlice = -1;

      function animate() {
        frame++;
        const t = frame / duration;
        const eased = easeOutQuart(t);
        rotation = startRotation + (totalRotation - startRotation) * eased;
        drawWheel(rotation);

        // เล่นเสียง tick
        const currentAngle = rotation % (2 * Math.PI);
        const currentSlice = Math.floor(currentAngle / slice);
        if (currentSlice !== lastTickSlice) {
          document.getElementById("tick-sound").currentTime = 0;
          document.getElementById("tick-sound").play();
          lastTickSlice = currentSlice;
        }

        if (frame < duration) {
          requestAnimationFrame(animate);
        } else {
          const finalAngle = (rotation % (2 * Math.PI) + 2 * Math.PI) % (2 * Math.PI);
          const index = Math.floor(((1.5 * Math.PI - finalAngle + 2 * Math.PI) % (2 * Math.PI)) / slice);

          document.getElementById("result").textContent = `ผลลัพธ์: ${options[index]}`;
          document.getElementById("spin-sound").play();

          const container = document.getElementById("container");
          container.classList.add("shake");
          setTimeout(() => container.classList.remove("shake"), 500);

          isSpinning = false;
          spinBtn.disabled = false;
          spinBtn.style.opacity = "1";
        }
      }

      animate();
    }

    drawWheel();
  </script>
</body>
</html>