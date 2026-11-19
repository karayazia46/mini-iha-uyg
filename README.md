index.html
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
  <title>6mm Motor Gaz + Gyro Servo</title>
  <style>
    * { box-sizing: border-box; }
    html, body {
      margin: 0;
      padding: 0;
      min-height: 100%;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      color: #f8fafc;
      overscroll-behavior: none;
      overflow-x: hidden;
    }

    /* Pilot Gözünden Gökyüzü ve Bulut Arka Planı */
    .sky-scene {
      position: fixed;
      inset: 0;
      z-index: -1;
      background: linear-gradient(
        180deg,
        #0b3c6d 0%,
        #1e6bb8 38%,
        #67b3ec 74%,
        #bfe3f8 100%
      );
      overflow: hidden;
    }

    .sun-glow {
      position: absolute;
      top: 12%;
      right: 16%;
      width: 170px;
      height: 170px;
      border-radius: 50%;
      background: radial-gradient(circle, rgba(255,250,220,0.95) 0%, rgba(255,235,170,0.35) 45%, rgba(255,235,170,0) 75%);
      filter: blur(2px);
      pointer-events: none;
    }

    .cloud {
      position: absolute;
      background: rgba(255, 255, 255, 0.84);
      border-radius: 999px;
      filter: blur(1px);
      box-shadow: 0 8px 24px rgba(15, 23, 42, 0.12);
      pointer-events: none;
      will-change: transform;
    }
    .cloud::before, .cloud::after {
      content: "";
      position: absolute;
      background: inherit;
      border-radius: 50%;
    }

    .c1 {
      width: 190px; height: 52px; top: 18%; left: 0;
      before-top: -38px;
    }
    .c1::before { width: 84px; height: 84px; top: -38px; left: 26px; }
    .c1::after  { width: 64px; height: 64px; top: -24px; right: 28px; }

    .c2 {
      width: 260px; height: 66px; top: 42%; left: 0;
      opacity: 0.68;
    }
    .c2::before { width: 110px; height: 110px; top: -48px; left: 38px; }
    .c2::after  { width: 85px; height: 85px; top: -32px; right: 34px; }

    .c3 {
      width: 340px; height: 82px; bottom: 18%; left: 0;
      opacity: 0.54;
      filter: blur(2.5px);
    }
    .c3::before { width: 140px; height: 140px; top: -60px; left: 55px; }
    .c3::after  { width: 105px; height: 105px; top: -40px; right: 48px; }

    /* Uçak Kokpit / Kanat / Pervane Silüeti */
    .cockpit-frame {
      position: fixed;
      inset: 0;
      pointer-events: none;
      z-index: 0;
    }
    .wing-left, .wing-right {
      position: absolute;
      bottom: -18px;
      width: 46%;
      height: 76px;
      background: linear-gradient(180deg, #1e293b 0%, #0f172a 100%);
      border-top: 2px solid rgba(255, 255, 255, 0.22);
      box-shadow: 0 -8px 24px rgba(0, 0, 0, 0.35);
    }
    .wing-left {
      left: -4%;
      transform: rotate(6deg);
      border-top-right-radius: 18px;
    }
    .wing-right {
      right: -4%;
      transform: rotate(-6deg);
      border-top-left-radius: 18px;
    }
    .nose-cone {
      position: absolute;
      bottom: -24px;
      left: 50%;
      transform: translateX(-50%);
      width: 140px;
      height: 82px;
      background: linear-gradient(180deg, #334155 0%, #0f172a 100%);
      border-top-left-radius: 46px;
      border-top-right-radius: 46px;
      border-top: 2px solid rgba(255, 255, 255, 0.25);
    }
    .prop-disc {
      position: absolute;
      bottom: 18px;
      left: 50%;
      transform: translateX(-50%) scaleX(1.35);
      width: 240px;
      height: 240px;
      border-radius: 50%;
      border: 2px dashed rgba(255, 255, 255, var(--prop-opacity, 0.12));
      background: radial-gradient(
        circle,
        rgba(255, 255, 255, calc(var(--prop-opacity, 0.12) * 0.22)) 0%,
        rgba(255, 255, 255, 0) 70%
      );
      transition: opacity 0.12s linear;
    }

    /* Kontrol Kartı */
    .wrapper {
      position: relative;
      z-index: 1;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 16px;
    }
    .card {
      width: 100%;
      max-width: 400px;
      background: rgba(15, 23, 42, 0.72);
      backdrop-filter: blur(12px);
      -webkit-backdrop-filter: blur(12px);
      border: 1px solid rgba(255, 255, 255, 0.18);
      padding: 18px;
      border-radius: 18px;
      box-shadow: 0 14px 36px rgba(2, 6, 23, 0.45);
    }
    .top-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 12px;
    }
    h2 { margin: 0; font-size: 19px; }
    .badge {
      font-size: 11px;
      padding: 4px 9px;
      border-radius: 999px;
      background: rgba(255, 255, 255, 0.14);
      color: #e2e8f0;
      font-weight: 600;
    }
    button {
      width: 100%;
      padding: 11px;
      border: 0;
      border-radius: 11px;
      font-size: 14px;
      font-weight: 600;
      cursor: pointer;
    }
    #connectBtn { background: #2563eb; color: #fff; }
    #gyroBtn { background: #059669; color: #fff; margin-top: 8px; }
    #gyroBtn.active { background: #d97706; }
    #status {
      margin-top: 9px;
      font-size: 12.5px;
      color: #cbd5e1;
      text-align: center;
    }

    .telemetry {
      display: grid;
      grid-template-columns: 1fr 1fr 1fr;
      gap: 8px;
      margin-top: 12px;
    }
    .stat {
      background: rgba(2, 6, 23, 0.48);
      border: 1px solid rgba(255, 255, 255, 0.1);
      padding: 9px 10px;
      border-radius: 11px;
      display: flex;
      flex-direction: column;
      gap: 2px;
      font-size: 11.5px;
      color: #94a3b8;
    }
    .stat strong { font-size: 16px; color: #ffffff; }

    /* Tek Eksen (Sadece Y - Gaz) Sanal Joystick */
    .pad-wrap {
      margin-top: 14px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    #throttleTrack {
      width: 114px;
      height: 230px;
      border-radius: 57px;
      background: linear-gradient(
        180deg,
        rgba(37, 99, 235, 0.3) 0%,
        rgba(15, 23, 42, 0.78) 55%,
        rgba(2, 6, 23, 0.9) 100%
      );
      border: 2px solid rgba(255, 255, 255, 0.22);
      position: relative;
      touch-action: none;
      user-select: none;
      -webkit-user-select: none;
    }
    #throttleTrack::before {
      content: "";
      position: absolute;
      left: 50%;
      top: 22px;
      bottom: 22px;
      width: 2px;
      background: rgba(255, 255, 255, 0.16);
      transform: translateX(-50%);
      pointer-events: none;
    }
    #knob {
      width: 74px;
      height: 74px;
      border-radius: 50%;
      background: linear-gradient(145deg, #38bdf8, #1d4ed8);
      border: 3px solid #e0f2fe;
      box-shadow: 0 8px 20px rgba(14, 165, 233, 0.5);
      position: absolute;
      left: 50%;
      bottom: 14px;
      transform: translate(-50%, 0);
      pointer-events: none;
    }
    .hint {
      margin-top: 9px;
      font-size: 12px;
      color: #cbd5e1;
      text-align: center;
      line-height: 1.4;
    }

    .row2 {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-top: 12px;
    }
    #calBtn { background: rgba(51, 65, 85, 0.92); color: #fff; }
    #chuteResetBtn { background: rgba(51, 65, 85, 0.92); color: #fff; }
    #chuteOpenBtn { background: #d97706; color: #fff; }
    #estopBtn { background: #dc2626; color: #fff; }
  </style>
</head>
<body>
  <div class="sky-scene">
    <div class="sun-glow"></div>
    <div class="cloud c1" id="cloud1"></div>
    <div class="cloud c2" id="cloud2"></div>
    <div class="cloud c3" id="cloud3"></div>
  </div>

  <div class="cockpit-frame">
    <div class="prop-disc" id="propDisc"></div>
    <div class="wing-left"></div>
    <div class="wing-right"></div>
    <div class="nose-cone"></div>
  </div>

  <div class="wrapper">
    <div class="card">
      <div class="top-bar">
        <h2>Gaz (Y) + Gyro Servo</h2>
        <div class="badge">BLE UART</div>
      </div>

      <button id="connectBtn" type="button">1. BLE ile Bağlan</button>
      <button id="gyroBtn" type="button">2. Gyro Sağa/Sola Yatış: KAPALI</button>
      <div id="status">Bağlı değil • Gyro kapalı</div>

      <div class="telemetry">
        <div class="stat">
          <span>Motor PWM</span>
          <strong><span id="motorOut">0</span></strong>
        </div>
        <div class="stat">
          <span>Servo (Gyro)</span>
          <strong><span id="servoOut">90</span>°</strong>
        </div>
        <div class="stat">
          <span>Yatış</span>
          <strong><span id="gammaOut">0</span>°</strong>
        </div>
      </div>

      <div class="pad-wrap">
        <div id="throttleTrack">
          <div id="knob"></div>
        </div>
        <div class="hint">
          Joystick (Sadece Y): Yukarı İt = Gaz (0–255), Bırak = 0<br />
          Gyro: Telefonu Sağa/Sola Eğ = Servo (0°–180°)
        </div>
      </div>

      <div class="row2">
        <button id="calBtn" type="button">Gyro Merkezle</button>
        <button id="chuteResetBtn" type="button">Paraşüt Kilitle</button>
      </div>
      <div class="row2">
        <button id="chuteOpenBtn" type="button">Paraşüt Aç (30°)</button>
        <button id="estopBtn" type="button">ACİL DURDUR</button>
      </div>
    </div>
  </div>

  <script>
    const SERVICE_UUID = "6e400001-b5a3-f393-e0a9-e50e24dcca9e";
    const RX_UUID      = "6e400002-b5a3-f393-e0a9-e50e24dcca9e";

    const MAX_TILT_DEG = 40;
    const INVERT_SERVO = false;

    const TRACK_HEIGHT = 230;
    const KNOB_SIZE = 74;
    const PADDING_Y = 14;
    const TRAVEL_PX = TRACK_HEIGHT - KNOB_SIZE - 2 * PADDING_Y; // 128 px

    let rxChar = null;
    let gyroEnabled = false;
    let gammaOffset = 0;
    let latestGamma = 0;

    let activePointerId = null;
    let currentPwm = 0;
    let currentServo = 90;

    let lastMotorSend = 0;
    let pendingMotor = null;
    let motorTimer = null;
    let lastSentMotor = 0;

    let lastServoSend = 0;
    let pendingServo = null;
    let servoTimer = null;
    let lastSentServo = 90;

    const connectBtn    = document.getElementById("connectBtn");
    const gyroBtn       = document.getElementById("gyroBtn");
    const calBtn        = document.getElementById("calBtn");
    const chuteOpenBtn  = document.getElementById("chuteOpenBtn");
    const chuteResetBtn = document.getElementById("chuteResetBtn");
    const estopBtn      = document.getElementById("estopBtn");
    const statusEl      = document.getElementById("status");
    const track         = document.getElementById("throttleTrack");
    const knob          = document.getElementById("knob");
    const motorOut      = document.getElementById("motorOut");
    const servoOut      = document.getElementById("servoOut");
    const gammaOut      = document.getElementById("gammaOut");
    const propDisc      = document.getElementById("propDisc");

    // Dinamik Bulut Animasyonu (Motor PWM'e göre hızlanır)
    const clouds = [
      { el: document.getElementById("cloud1"), basePxPerSec: 18, yOffset: 0,   scale: 1.0,  x: -220 },
      { el: document.getElementById("cloud2"), basePxPerSec: 12, yOffset: 0,   scale: 0.85, x: 180  },
      { el: document.getElementById("cloud3"), basePxPerSec: 28, yOffset: 0,   scale: 1.05, x: 520  }
    ];
    let lastAnimTime = performance.now();

    function animateClouds(now) {
      const dt = Math.min(0.1, (now - lastAnimTime) / 1000);
      lastAnimTime = now;

      const throttleRatio = currentPwm / 255;
      // Gaz 0 iken hafif rüzgar (1x), tam gazda ~4.5x daha hızlı geçiş
      const speedMultiplier = 1 + throttleRatio * 3.5;
      const viewWidth = window.innerWidth || 400;

      for (const c of clouds) {
        c.x += c.basePxPerSec * speedMultiplier * dt;
        if (c.x > viewWidth + 120) {
          c.x = -380;
        }
        c.el.style.transform = `translate3d(${c.x}px, 0, 0) scale(${c.scale})`;
      }
      requestAnimationFrame(animateClouds);
    }
    requestAnimationFrame(animateClouds);

    async function writeRaw(text) {
      if (!rxChar) return;
      try {
        const data = new TextEncoder().encode(text);
        if (rxChar.writeValueWithoutResponse) {
          await rxChar.writeValueWithoutResponse(data);
        } else {
          await rxChar.writeValue(data);
        }
      } catch (err) {
        statusEl.textContent = "Gönderim hatası: " + err.message;
      }
    }

    function queueMotor(pwm, immediate = false) {
      currentPwm = pwm;
      motorOut.textContent = pwm;

      const ratio = pwm / 255;
      propDisc.style.setProperty("--prop-opacity", (0.1 + ratio * 0.5).toFixed(2));

      if (immediate) {
        clearTimeout(motorTimer);
        pendingMotor = null;
        lastMotorSend = Date.now();
        if (pwm !== lastSentMotor) {
          lastSentMotor = pwm;
          writeRaw("M" + pwm);
        }
        return;
      }

      const now = Date.now();
      const elapsed = now - lastMotorSend;
      if (elapsed >= 45) {
        lastMotorSend = now;
        if (pwm !== lastSentMotor) {
          lastSentMotor = pwm;
          writeRaw("M" + pwm);
        }
      } else {
        pendingMotor = pwm;
        clearTimeout(motorTimer);
        motorTimer = setTimeout(() => {
          lastMotorSend = Date.now();
          const val = pendingMotor;
          pendingMotor = null;
          if (val !== null && val !== lastSentMotor) {
            lastSentMotor = val;
            writeRaw("M" + val);
          }
        }, 45 - elapsed);
      }
    }

    function queueServo(angle, immediate = false) {
      currentServo = Math.max(0, Math.min(180, Math.round(angle)));
      servoOut.textContent = currentServo;

      if (immediate) {
        clearTimeout(servoTimer);
        pendingServo = null;
        lastServoSend = Date.now();
        if (currentServo !== lastSentServo) {
          lastSentServo = currentServo;
          writeRaw("S" + currentServo);
        }
        return;
      }

      const now = Date.now();
      const elapsed = now - lastServoSend;
      if (elapsed >= 45) {
        lastServoSend = now;
        if (currentServo !== lastSentServo) {
          lastSentServo = currentServo;
          writeRaw("S" + currentServo);
        }
      } else {
        pendingServo = currentServo;
        clearTimeout(servoTimer);
        servoTimer = setTimeout(() => {
          lastServoSend = Date.now();
          const val = pendingServo;
          pendingServo = null;
          if (val !== null && val !== lastSentServo) {
            lastSentServo = val;
            writeRaw("S" + val);
          }
        }, 45 - elapsed);
      }
    }

    function updateStatus() {
      const bleTxt = rxChar ? "BLE Bağlı" : "BLE Bağlı değil";
      const gyroTxt = gyroEnabled ? "Gyro AÇIK" : "Gyro KAPALI";
      statusEl.textContent = `${bleTxt} • ${gyroTxt}`;

      gyroBtn.textContent = gyroEnabled
        ? "2. Gyro Sağa/Sola Yatış: AÇIK"
        : "2. Gyro Sağa/Sola Yatış: KAPALI";
      gyroBtn.classList.toggle("active", gyroEnabled);
    }

    function setKnobLift(ratio) {
      const liftPx = ratio * TRAVEL_PX;
      knob.style.transform = `translate(-50%, ${-liftPx}px)`;
    }

    function updateFromY(clientY) {
      const rect = track.getBoundingClientRect();
      const bottomEdge = rect.bottom - PADDING_Y - KNOB_SIZE / 2;
      const topEdge = rect.top + PADDING_Y + KNOB_SIZE / 2;

      const clampedY = Math.max(topEdge, Math.min(bottomEdge, clientY));
      const ratio = (bottomEdge - clampedY) / TRAVEL_PX; // 0..1
      const pwm = ratio < 0.03 ? 0 : Math.round(ratio * 255);

      setKnobLift(ratio);
      queueMotor(pwm, false);
    }

    function resetThrottle(immediate = true) {
      activePointerId = null;
      setKnobLift(0);
      queueMotor(0, immediate);
    }

    function handleOrientation(event) {
      if (!gyroEnabled || typeof event.gamma !== "number") return;

      latestGamma = event.gamma;
      let relGamma = event.gamma - gammaOffset;
      relGamma = Math.max(-MAX_TILT_DEG, Math.min(MAX_TILT_DEG, relGamma));
      if (Math.abs(relGamma) < 2) relGamma = 0;

      const norm = (relGamma + MAX_TILT_DEG) / (2 * MAX_TILT_DEG); // 0..1
      let angle = Math.round(norm * 180);
      angle = Math.max(0, Math.min(180, angle));
      if (INVERT_SERVO) angle = 180 - angle;

      gammaOut.textContent = Math.round(relGamma);
      queueServo(angle, false);
    }

    track.addEventListener("pointerdown", (e) => {
      e.preventDefault();
      track.setPointerCapture(e.pointerId);
      activePointerId = e.pointerId;
      updateFromY(e.clientY);
    });

    track.addEventListener("pointermove", (e) => {
      if (e.pointerId !== activePointerId) return;
      e.preventDefault();
      updateFromY(e.clientY);
    });

    function onPointerEnd(e) {
      if (e.pointerId !== activePointerId) return;
      e.preventDefault();
      resetThrottle(true);
    }

    track.addEventListener("pointerup", onPointerEnd);
    track.addEventListener("pointercancel", onPointerEnd);

    gyroBtn.addEventListener("click", async () => {
      if (!gyroEnabled) {
        if (typeof DeviceOrientationEvent !== "undefined" &&
            typeof DeviceOrientationEvent.requestPermission === "function") {
          try {
            const res = await DeviceOrientationEvent.requestPermission();
            if (res !== "granted") {
              statusEl.textContent = "Gyro izni verilmedi.";
              return;
            }
          } catch (err) {
            statusEl.textContent = "Gyro izni hatası: " + err.message;
            return;
          }
        }
        window.addEventListener("deviceorientation", handleOrientation);
        gyroEnabled = true;
      } else {
        window.removeEventListener("deviceorientation", handleOrientation);
        gyroEnabled = false;
        gammaOut.textContent = "0";
        queueServo(90, true);
      }
      updateStatus();
    });

    calBtn.addEventListener("click", () => {
      gammaOffset = latestGamma;
      gammaOut.textContent = "0";
      queueServo(90, true);
    });

    chuteOpenBtn.addEventListener("click", async () => {
      if (gyroEnabled) {
        window.removeEventListener("deviceorientation", handleOrientation);
        gyroEnabled = false;
        gammaOut.textContent = "0";
        updateStatus();
      }
      resetThrottle(true);
      currentServo = 30;
      servoOut.textContent = "30";
      lastSentServo = 30;
      await writeRaw("OPEN_CHUTE");
    });

    chuteResetBtn.addEventListener("click", async () => {
      currentServo = 90;
      servoOut.textContent = "90";
      lastSentServo = 90;
      await writeRaw("RESET_CHUTE");
    });

    estopBtn.addEventListener("click", async () => {
      resetThrottle(true);
      await writeRaw("STOP");
    });

    connectBtn.addEventListener("click", async () => {
      try {
        if (!navigator.bluetooth) {
          statusEl.textContent = "Bu tarayıcı Web Bluetooth desteklemiyor (Android Chrome veya iOS Bluefy kullan).";
          return;
        }
        statusEl.textContent = "ESP32C3_MicroCtrl aranıyor...";
        const device = await navigator.bluetooth.requestDevice({
          filters: [{ name: "ESP32C3_MicroCtrl" }],
          optionalServices: [SERVICE_UUID]
        });
        device.addEventListener("gattserverdisconnected", () => {
          rxChar = null;
          setKnobLift(0);
          currentPwm = 0;
          currentServo = 30;
          motorOut.textContent = "0";
          servoOut.textContent = "30";
          propDisc.style.setProperty("--prop-opacity", "0.10");
          updateStatus();
          statusEl.textContent = "Bağlantı koptu (Motor 0, Paraşüt 30°)";
        });
        const server = await device.gatt.connect();
        const service = await server.getPrimaryService(SERVICE_UUID);
        rxChar = await service.getCharacteristic(RX_UUID);
        lastSentMotor = -1;
        lastSentServo = -1;
        resetThrottle(true);
        queueServo(90, true);
        updateStatus();
      } catch (err) {
        statusEl.textContent = "Bağlantı iptal/hata: " + err.message;
      }
    });
  </script>
</body>
</html>
