<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover"
  />
  <meta name="mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <title>Dead Horizon 3D - Phase 1</title>

  <!-- Three.js 3D Engine Import Map -->
  <script type="importmap">
    {
      "imports": {
        "three": "https://cdn.jsdelivr.net/npm/three@0.164.1/build/three.module.js"
      }
    }
  </script>

  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      -webkit-user-select: none;
      user-select: none;
      -webkit-touch-callout: none;
      touch-action: none;
    }

    html,
    body {
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: #0b0d0e;
      font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
      color: #e6ece8;
    }

    #game-container {
      position: fixed;
      inset: 0;
      width: 100vw;
      height: 100vh;
      z-index: 1;
    }

    #hud {
      position: fixed;
      inset: 0;
      z-index: 10;
      pointer-events: none;
    }

    /* Top-Left Telemetry */
    #telemetry {
      position: absolute;
      top: 12px;
      left: 14px;
      display: flex;
      flex-direction: column;
      gap: 4px;
      background: rgba(10, 14, 16, 0.55);
      border-left: 3px solid #d9534f;
      padding: 6px 10px;
      border-radius: 4px;
      font-size: 12px;
      font-weight: 600;
      letter-spacing: 0.5px;
    }

    /* Fullscreen Button */
    .hud-top-btn {
      position: absolute;
      top: 12px;
      right: 14px;
      pointer-events: auto;
      background: rgba(20, 26, 28, 0.75);
      color: #f0f4f2;
      border: 1px solid rgba(255, 255, 255, 0.25);
      border-radius: 6px;
      padding: 8px 12px;
      font-size: 11px;
      font-weight: 700;
      letter-spacing: 0.8px;
    }

    .hud-top-btn:active {
      background: rgba(217, 83, 79, 0.8);
    }

    /* Center Crosshair */
    #crosshair {
      position: absolute;
      top: 50%;
      left: 50%;
      width: 8px;
      height: 8px;
      transform: translate(-50%, -50%);
      border: 2px solid rgba(255, 255, 255, 0.85);
      border-radius: 50%;
      box-shadow: 0 0 4px rgba(0, 0, 0, 0.9);
    }

    /* Left Joystick Touch Zone (Left 45% of screen) */
    #joystick-zone {
      position: absolute;
      left: 0;
      bottom: 0;
      width: 45%;
      height: 75%;
      pointer-events: auto;
    }

    #joystick-base {
      position: absolute;
      left: 44px;
      bottom: 44px;
      width: 116px;
      height: 116px;
      border-radius: 50%;
      background: rgba(255, 255, 255, 0.08);
      border: 2px solid rgba(255, 255, 255, 0.25);
      display: flex;
      align-items: center;
      justify-content: center;
      transition: opacity 0.15s ease;
    }

    #joystick-stick {
      width: 48px;
      height: 48px;
      border-radius: 50%;
      background: rgba(230, 236, 232, 0.55);
      border: 2px solid rgba(255, 255, 255, 0.85);
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
      transform: translate(0px, 0px);
    }

    /* Right Screen Camera Look Zone (Right 55% of screen) */
    #look-zone {
      position: absolute;
      right: 0;
      top: 0;
      width: 55%;
      height: 100%;
      pointer-events: auto;
    }

    /* Action Buttons Cluster (Bottom Right) */
    #action-buttons {
      position: absolute;
      right: 26px;
      bottom: 28px;
      display: flex;
      flex-direction: column;
      align-items: flex-end;
      gap: 12px;
      pointer-events: none;
    }

    .action-btn {
      pointer-events: auto;
      width: 68px;
      height: 68px;
      border-radius: 50%;
      background: rgba(18, 24, 26, 0.65);
      border: 2px solid rgba(255, 255, 255, 0.3);
      color: #ffffff;
      font-size: 11px;
      font-weight: 800;
      letter-spacing: 0.6px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.45);
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .action-btn:active,
    .action-btn.active {
      background: rgba(217, 83, 79, 0.78);
      border-color: #ffffff;
      transform: scale(0.94);
    }

    #btn-jump {
      width: 76px;
      height: 76px;
      font-size: 12px;
    }

    /* Portrait Orientation Warning */
    #rotate-overlay {
      display: none;
      position: fixed;
      inset: 0;
      z-index: 100;
      background: #0b0d0e;
      align-items: center;
      justify-content: center;
      text-align: center;
      padding: 24px;
    }

    .rotate-box h2 {
      color: #d9534f;
      margin-bottom: 10px;
      font-size: 22px;
    }

    .rotate-box p {
      color: #b8c2be;
      font-size: 15px;
    }

    @media screen and (orientation: portrait) {
      #rotate-overlay {
        display: flex;
      }
    }
  </style>
</head>
<body>
  <!-- 3D WebGL Canvas Container -->
  <div id="game-container"></div>

  <!-- Landscape Mode Warning -->
  <div id="rotate-overlay">
    <div class="rotate-box">
      <h2>УТСАА ХЭВТҮҮЛНЭ ҮҮ</h2>
      <p>Хоёр эрхий хуруугаар тоглохын тулд дэлгэцээ хэвтээ (Landscape) болгоно уу.</p>
    </div>
  </div>

  <!-- Heads-Up Display (HUD) -->
  <div id="hud">
    <div id="telemetry">
      <span id="fps-counter">FPS: 60</span>
      <span id="pos-counter">POS: 0, 0, 0</span>
      <span id="stance-indicator">STANCE: STAND</span>
    </div>

    <button id="btn-fullscreen" class="hud-top-btn" type="button">FULLSCREEN</button>

    <div id="crosshair"></div>

    <!-- Left Screen: Touch Joystick -->
    <div id="joystick-zone">
      <div id="joystick-base">
        <div id="joystick-stick"></div>
      </div>
    </div>

    <!-- Right Screen: Camera Rotation -->
    <div id="look-zone"></div>

    <!-- Right Thumb Action Buttons -->
    <div id="action-buttons">
      <button id="btn-sprint" class="action-btn" type="button">SPRINT</button>
      <button id="btn-jump" class="action-btn" type="button">JUMP</button>
      <button id="btn-crouch" class="action-btn" type="button">CROUCH</button>
    </div>
  </div>

  <!-- Complete Phase 1 Game Engine, Controls & Player Physics -->
  <script type="module">
    import * as THREE from 'three';

    // ==========================================
    // 1. 3D ENGINE & COLLISION TEST ARENA
    // ==========================================
    class GameEngine {
      constructor(container) {
        this.container = container;
        this.colliders = [];

        this.scene = new THREE.Scene();
        this.scene.background = new THREE.Color(0x1c2426);
        this.scene.fog = new THREE.FogExp2(0x1c2426, 0.018);

        this.camera = new THREE.PerspectiveCamera(
          72,
          window.innerWidth / window.innerHeight,
          0.1,
          250
        );
        this.camera.rotation.order = 'YXZ';

        this.renderer = new THREE.WebGLRenderer({
          antialias: true,
          powerPreference: 'high-performance'
        });
        this.renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        this.renderer.shadowMap.enabled = true;
        this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        this.renderer.toneMapping = THREE.ACESFilmicToneMapping;
        this.renderer.toneMappingExposure = 1.05;

        this.container.appendChild(this.renderer.domElement);

        this.setupLighting();
        this.buildTestArena();

        window.addEventListener('resize', () => this.onResize());
      }

      setupLighting() {
        const hemiLight = new THREE.HemisphereLight(0x8fa8b2, 0x2b2924, 0.85);
        hemiLight.position.set(0, 50, 0);
        this.scene.add(hemiLight);

        const dirLight = new THREE.DirectionalLight(0xffe6cc, 1.4);
        dirLight.position.set(35, 60, 25);
        dirLight.castShadow = true;
        dirLight.shadow.mapSize.width = 1024;
        dirLight.shadow.mapSize.height = 1024;
        dirLight.shadow.camera.near = 1;
        dirLight.shadow.camera.far = 160;

        const shadowBound = 45;
        dirLight.shadow.camera.left = -shadowBound;
        dirLight.shadow.camera.right = shadowBound;
        dirLight.shadow.camera.top = shadowBound;
        dirLight.shadow.camera.bottom = -shadowBound;
        dirLight.shadow.bias = -0.0005;

        this.scene.add(dirLight);
      }

      createGridTexture(baseHex, lineHex, repeatCount) {
        const canvas = document.createElement('canvas');
        canvas.width = 256;
        canvas.height = 256;
        const ctx = canvas.getContext('2d');

        ctx.fillStyle = baseHex;
        ctx.fillRect(0, 0, 256, 256);

        for (let i = 0; i < 600; i++) {
          ctx.fillStyle = Math.random() > 0.5 ? 'rgba(255,255,255,0.03)' : 'rgba(0,0,0,0.05)';
          ctx.fillRect(Math.random() * 256, Math.random() * 256, 4, 4);
        }

        ctx.strokeStyle = lineHex;
        ctx.lineWidth = 4;
        ctx.strokeRect(0, 0, 256, 256);

        const texture = new THREE.CanvasTexture(canvas);
        texture.wrapS = THREE.RepeatWrapping;
        texture.wrapT = THREE.RepeatWrapping;
        texture.repeat.set(repeatCount, repeatCount);
        return texture;
      }

      addStaticBox(x, y, z, width, height, depth, colorHex, castShadow = true) {
        const geo = new THREE.BoxGeometry(width, height, depth);
        const mat = new THREE.MeshStandardMaterial({
          color: colorHex,
          roughness: 0.85,
          metalness: 0.1
        });
        const mesh = new THREE.Mesh(geo, mat);
        mesh.position.set(x, y, z);
        mesh.castShadow = castShadow;
        mesh.receiveShadow = true;
        this.scene.add(mesh);

        this.colliders.push({
          minX: x - width / 2,
          maxX: x + width / 2,
          minY: y - height / 2,
          maxY: y + height / 2,
          minZ: z - depth / 2,
          maxZ: z + depth / 2
        });

        return mesh;
      }

      buildTestArena() {
        // Ground Plane
        const groundTex = this.createGridTexture('#2a2e2d', '#3a403e', 30);
        const groundGeo = new THREE.PlaneGeometry(120, 120);
        const groundMat = new THREE.MeshStandardMaterial({
          map: groundTex,
          roughness: 0.92
        });
        const ground = new THREE.Mesh(groundGeo, groundMat);
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        this.scene.add(ground);

        // Perimeter Walls
        this.addStaticBox(0, 3, -40, 80, 6, 1.5, 0x454a48);
        this.addStaticBox(0, 3, 40, 80, 6, 1.5, 0x454a48);
        this.addStaticBox(-40, 3, 0, 1.5, 6, 80, 0x454a48);
        this.addStaticBox(40, 3, 0, 1.5, 6, 80, 0x454a48);

        // Jumpable Supply Crates
        this.addStaticBox(-4, 0.35, -8, 2.2, 0.7, 2.2, 0x6b5239);
        this.addStaticBox(-6.5, 0.7, -8, 2.2, 1.4, 2.2, 0x5c452e);
        this.addStaticBox(-9, 1.05, -8, 2.2, 2.1, 2.2, 0x4f3b26);

        // Crouch-Under Low Barrier
        this.addStaticBox(5, 2.125, -8, 4.5, 1.75, 1.5, 0x703835);
        this.addStaticBox(2.5, 1.5, -8, 0.6, 3.0, 1.5, 0x4a4f4d);
        this.addStaticBox(7.5, 1.5, -8, 0.6, 3.0, 1.5, 0x4a4f4d);

        // Cover Pillars & Ruined Structure
        this.addStaticBox(-12, 2.5, 6, 1.2, 5.0, 1.2, 0x555b59);
        this.addStaticBox(-6, 2.5, 6, 1.2, 5.0, 1.2, 0x555b59);
        this.addStaticBox(10, 1.1, 8, 5.0, 2.2, 0.8, 0x5c6361);
        this.addStaticBox(14, 3.5, -15, 7.0, 7.0, 7.0, 0x3b403e);
      }

      onResize() {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
      }

      render() {
        this.renderer.render(this.scene, this.camera);
      }
    }

    // ==========================================
    // 2. MULTI-TOUCH MOBILE CONTROLS
    // ==========================================
    class TouchControls {
      constructor() {
        this.moveX = 0;
        this.moveY = 0;
        this.lookDeltaX = 0;
        this.lookDeltaY = 0;

        this.jumpPressed = false;
        this.crouchActive = false;
        this.sprintActive = false;

        this.joystickTouchId = null;
        this.lookTouchId = null;
        this.lastLookX = 0;
        this.lastLookY = 0;

        this.lookSensitivity = 0.0045;
        this.joystickMaxRadius = 46;

        this.joystickZone = document.getElementById('joystick-zone');
        this.joystickBase = document.getElementById('joystick-base');
        this.joystickStick = document.getElementById('joystick-stick');
        this.lookZone = document.getElementById('look-zone');

        this.btnJump = document.getElementById('btn-jump');
        this.btnCrouch = document.getElementById('btn-crouch');
        this.btnSprint = document.getElementById('btn-sprint');
        this.btnFullscreen = document.getElementById('btn-fullscreen');

        this.keys = {};

        this.bindTouchEvents();
        this.bindButtonEvents();
        this.bindDesktopFallback();
      }

      bindTouchEvents() {
        // Left Thumb: Virtual Joystick
        this.joystickZone.addEventListener(
          'touchstart',
          (e) => {
            e.preventDefault();
            if (this.joystickTouchId !== null) return;

            const touch = e.changedTouches[0];
            this.joystickTouchId = touch.identifier;
            this.updateJoystick(touch.clientX, touch.clientY);
          },
          { passive: false }
        );

        this.joystickZone.addEventListener(
          'touchmove',
          (e) => {
            e.preventDefault();
            for (let i = 0; i < e.changedTouches.length; i++) {
              const touch = e.changedTouches[i];
              if (touch.identifier === this.joystickTouchId) {
                this.updateJoystick(touch.clientX, touch.clientY);
                break;
              }
            }
          },
          { passive: false }
        );

        const endJoystick = (e) => {
          for (let i = 0; i < e.changedTouches.length; i++) {
            if (e.changedTouches[i].identifier === this.joystickTouchId) {
              this.joystickTouchId = null;
              this.moveX = 0;
              this.moveY = 0;
              this.joystickStick.style.transform = 'translate(0px, 0px)';
              break;
            }
          }
        };

        this.joystickZone.addEventListener('touchend', endJoystick);
        this.joystickZone.addEventListener('touchcancel', endJoystick);

        // Right Thumb: Camera Rotation
        this.lookZone.addEventListener(
          'touchstart',
          (e) => {
            e.preventDefault();
            if (this.lookTouchId !== null) return;

            const touch = e.changedTouches[0];
            this.lookTouchId = touch.identifier;
            this.lastLookX = touch.clientX;
            this.lastLookY = touch.clientY;
          },
          { passive: false }
        );

        this.lookZone.addEventListener(
          'touchmove',
          (e) => {
            e.preventDefault();
            for (let i = 0; i < e.changedTouches.length; i++) {
              const touch = e.changedTouches[i];
              if (touch.identifier === this.lookTouchId) {
                const dx = touch.clientX - this.lastLookX;
                const dy = touch.clientY - this.lastLookY;
                this.lastLookX = touch.clientX;
                this.lastLookY = touch.clientY;

                this.lookDeltaX += dx * this.lookSensitivity;
                this.lookDeltaY += dy * this.lookSensitivity;
                break;
              }
            }
          },
          { passive: false }
        );

        const endLook = (e) => {
          for (let i = 0; i < e.changedTouches.length; i++) {
            if (e.changedTouches[i].identifier === this.lookTouchId) {
              this.lookTouchId = null;
              break;
            }
          }
        };

        this.lookZone.addEventListener('touchend', endLook);
        this.lookZone.addEventListener('touchcancel', endLook);
      }

      updateJoystick(touchX, touchY) {
        const rect = this.joystickBase.getBoundingClientRect();
        const centerX = rect.left + rect.width / 2;
        const centerY = rect.top + rect.height / 2;

        let dx = touchX - centerX;
        let dy = touchY - centerY;
        const distance = Math.hypot(dx, dy);

        if (distance > this.joystickMaxRadius) {
          dx = (dx / distance) * this.joystickMaxRadius;
          dy = (dy / distance) * this.joystickMaxRadius;
        }

        this.joystickStick.style.transform = `translate(${dx}px, ${dy}px)`;

        const normX = dx / this.joystickMaxRadius;
        const normY = dy / this.joystickMaxRadius;
        this.moveX = Math.abs(normX) < 0.12 ? 0 : normX;
        this.moveY = Math.abs(normY) < 0.12 ? 0 : normY;
      }

      bindButtonEvents() {
        this.btnJump.addEventListener(
          'touchstart',
          (e) => {
            e.preventDefault();
            e.stopPropagation();
            this.jumpPressed = true;
          },
          { passive: false }
        );
        this.btnJump.addEventListener('mousedown', () => {
          this.jumpPressed = true;
        });

        const toggleCrouch = (e) => {
          e.preventDefault();
          e.stopPropagation();
          this.crouchActive = !this.crouchActive;
          this.btnCrouch.classList.toggle('active', this.crouchActive);
        };
        this.btnCrouch.addEventListener('touchstart', toggleCrouch, { passive: false });
        this.btnCrouch.addEventListener('mousedown', toggleCrouch);

        const toggleSprint = (e) => {
          e.preventDefault();
          e.stopPropagation();
          this.sprintActive = !this.sprintActive;
          this.btnSprint.classList.toggle('active', this.sprintActive);
        };
        this.btnSprint.addEventListener('touchstart', toggleSprint, { passive: false });
        this.btnSprint.addEventListener('mousedown', toggleSprint);

        this.btnFullscreen.addEventListener('click', () => {
          if (!document.fullscreenElement) {
            document.documentElement.requestFullscreen().catch(() => {});
          } else {
            document.exitFullscreen().catch(() => {});
          }
        });
      }

      bindDesktopFallback() {
        window.addEventListener('keydown', (e) => {
          this.keys[e.code] = true;
          if (e.code === 'Space') this.jumpPressed = true;
          if (e.code === 'KeyC') {
            this.crouchActive = !this.crouchActive;
            this.btnCrouch.classList.toggle('active', this.crouchActive);
          }
          if (e.code === 'ShiftLeft') {
            this.sprintActive = true;
            this.btnSprint.classList.add('active');
          }
        });

        window.addEventListener('keyup', (e) => {
          this.keys[e.code] = false;
          if (e.code === 'ShiftLeft') {
            this.sprintActive = false;
            this.btnSprint.classList.remove('active');
          }
        });
      }

      getInput() {
        let mx = this.moveX;
        let my = this.moveY;

        if (this.keys['KeyW']) my -= 1;
        if (this.keys['KeyS']) my += 1;
        if (this.keys['KeyA']) mx -= 1;
        if (this.keys['KeyD']) mx += 1;

        const len = Math.hypot(mx, my);
        if (len > 1) {
          mx /= len;
          my /= len;
        }

        const input = {
          moveX: mx,
          moveY: my,
          lookX: this.lookDeltaX,
          lookY: this.lookDeltaY,
          jump: this.jumpPressed,
          crouch: this.crouchActive,
          sprint: this.sprintActive
        };

        this.lookDeltaX = 0;
        this.lookDeltaY = 0;
        this.jumpPressed = false;

        return input;
      }
    }

    // ==========================================
    // 3. FIRST-PERSON PLAYER & PHYSICS
    // ==========================================
    class PlayerController {
      constructor(camera, scene, colliders) {
        this.camera = camera;
        this.scene = scene;
        this.colliders = colliders;

        this.radius = 0.42;
        this.standingHeight = 1.72;
        this.crouchingHeight = 0.95;
        this.currentHeight = this.standingHeight;

        this.position = new THREE.Vector3(0, 0, 12);
        this.velocity = new THREE.Vector3(0, 0, 0);

        this.yaw = 0;
        this.pitch = 0;

        this.walkSpeed = 5.2;
        this.sprintSpeed = 8.4;
        this.crouchSpeed = 2.6;
        this.jumpForce = 7.4;
        this.gravity = 20.0;

        this.isGrounded = false;
        this.isCrouching = false;
        this.bobTimer = 0;

        this.setupFirstPersonRig();
        this.updateCameraTransform(0);
      }

      setupFirstPersonRig() {
        this.rigGroup = new THREE.Group();

        const armGeo = new THREE.BoxGeometry(0.11, 0.11, 0.45);
        const armMat = new THREE.MeshStandardMaterial({ color: 0x3b4d3b, roughness: 0.8 });
        const armMesh = new THREE.Mesh(armGeo, armMat);
        armMesh.position.set(0.28, -0.26, -0.35);
        armMesh.rotation.x = 0.15;

        const lightBodyGeo = new THREE.CylinderGeometry(0.032, 0.036, 0.22, 12);
        const lightBodyMat = new THREE.MeshStandardMaterial({
          color: 0x1a1d1e,
          metalness: 0.7,
          roughness: 0.3
        });
        const lightBody = new THREE.Mesh(lightBodyGeo, lightBodyMat);
        lightBody.rotation.x = Math.PI / 2;
        lightBody.position.set(0.28, -0.22, -0.58);

        this.flashlight = new THREE.SpotLight(0xfff5e0, 2.2, 32, Math.PI / 5, 0.45, 1.2);
        this.flashlight.position.set(0.28, -0.22, -0.68);
        this.flashlight.target.position.set(0.28, -0.22, -10);

        this.rigGroup.add(armMesh);
        this.rigGroup.add(lightBody);
        this.rigGroup.add(this.flashlight);
        this.rigGroup.add(this.flashlight.target);

        this.camera.add(this.rigGroup);
        this.scene.add(this.camera);
      }

      update(dt, input) {
        this.yaw -= input.lookX;
        this.pitch -= input.lookY;
        const maxPitch = (85 * Math.PI) / 180;
        this.pitch = Math.max(-maxPitch, Math.min(maxPitch, this.pitch));

        if (input.crouch) {
          this.isCrouching = true;
        } else if (this.isCrouching && this.canStandUp()) {
          this.isCrouching = false;
        }

        const targetHeight = this.isCrouching ? this.crouchingHeight : this.standingHeight;
        this.currentHeight += (targetHeight - this.currentHeight) * Math.min(1, dt * 12);

        let speed = this.walkSpeed;
        if (this.isCrouching) {
          speed = this.crouchSpeed;
        } else if (input.sprint && input.moveY < -0.2) {
          speed = this.sprintSpeed;
        }

        const forwardX = -Math.sin(this.yaw);
        const forwardZ = -Math.cos(this.yaw);
        const rightX = Math.cos(this.yaw);
        const rightZ = -Math.sin(this.yaw);

        const moveForward = -input.moveY;
        const moveRight = input.moveX;

        const wishX = forwardX * moveForward + rightX * moveRight;
        const wishZ = forwardZ * moveForward + rightZ * moveRight;

        const accel = this.isGrounded ? 14.0 : 4.5;
        this.velocity.x += (wishX * speed - this.velocity.x) * Math.min(1, dt * accel);
        this.velocity.z += (wishZ * speed - this.velocity.z) * Math.min(1, dt * accel);

        if (input.jump && this.isGrounded && !this.isCrouching) {
          this.velocity.y = this.jumpForce;
          this.isGrounded = false;
        }

        this.velocity.y -= this.gravity * dt;

        this.position.x += this.velocity.x * dt;
        this.resolveHorizontalCollisions('x');

        this.position.z += this.velocity.z * dt;
        this.resolveHorizontalCollisions('z');

        this.position.y += this.velocity.y * dt;
        this.resolveVerticalCollisions();

        const horizontalSpeed = Math.hypot(this.velocity.x, this.velocity.z);
        this.updateCameraTransform(dt, horizontalSpeed);
      }

      canStandUp() {
        const feetY = this.position.y;
        const headY = feetY + this.standingHeight;

        for (const box of this.colliders) {
          const overlapXZ =
            this.position.x + this.radius > box.minX &&
            this.position.x - this.radius < box.maxX &&
            this.position.z + this.radius > box.minZ &&
            this.position.z - this.radius < box.maxZ;

          if (overlapXZ && box.minY < headY && box.maxY > feetY + 0.1) {
            return false;
          }
        }
        return true;
      }

      resolveHorizontalCollisions(axis) {
        const feetY = this.position.y + 0.05;
        const headY = this.position.y + this.currentHeight;

        for (const box of this.colliders) {
          if (headY <= box.minY || feetY >= box.maxY) continue;

          const overlapX =
            this.position.x + this.radius > box.minX &&
            this.position.x - this.radius < box.maxX;
          const overlapZ =
            this.position.z + this.radius > box.minZ &&
            this.position.z - this.radius < box.maxZ;

          if (overlapX && overlapZ) {
            if (axis === 'x') {
              if (this.velocity.x > 0) {
                this.position.x = box.minX - this.radius;
              } else if (this.velocity.x < 0) {
                this.position.x = box.maxX + this.radius;
              }
              this.velocity.x = 0;
            } else if (axis === 'z') {
              if (this.velocity.z > 0) {
                this.position.z = box.minZ - this.radius;
              } else if (this.velocity.z < 0) {
                this.position.z = box.maxZ + this.radius;
              }
              this.velocity.z = 0;
            }
          }
        }
      }

      resolveVerticalCollisions() {
        this.isGrounded = false;

        if (this.position.y <= 0) {
          this.position.y = 0;
          this.velocity.y = 0;
          this.isGrounded = true;
        }

        const feetY = this.position.y;
        const headY = this.position.y + this.currentHeight;

        for (const box of this.colliders) {
          const overlapX =
            this.position.x + this.radius * 0.85 > box.minX &&
            this.position.x - this.radius * 0.85 < box.maxX;
          const overlapZ =
            this.position.z + this.radius * 0.85 > box.minZ &&
            this.position.z - this.radius * 0.85 < box.maxZ;

          if (!overlapX || !overlapZ) continue;

          if (this.velocity.y <= 0 && feetY <= box.maxY && feetY >= box.maxY - 0.35) {
            this.position.y = box.maxY;
            this.velocity.y = 0;
            this.isGrounded = true;
          } else if (this.velocity.y > 0 && headY >= box.minY && feetY < box.minY) {
            this.position.y = box.minY - this.currentHeight;
            this.velocity.y = 0;
          }
        }
      }

      updateCameraTransform(dt, horizontalSpeed = 0) {
        let bobOffsetY = 0;
        let bobOffsetX = 0;

        if (this.isGrounded && horizontalSpeed > 0.4) {
          this.bobTimer += dt * horizontalSpeed * 1.8;
          bobOffsetY = Math.sin(this.bobTimer) * 0.035;
          bobOffsetX = Math.cos(this.bobTimer * 0.5) * 0.025;
        } else {
          this.bobTimer = 0;
        }

        const eyeY = this.position.y + (this.currentHeight - 0.14) + bobOffsetY;
        this.camera.position.set(this.position.x, eyeY, this.position.z);
        this.camera.rotation.y = this.yaw;
        this.camera.rotation.x = this.pitch;

        this.rigGroup.position.set(bobOffsetX, bobOffsetY * 0.7, 0);
      }
    }

    // ==========================================
    // 4. MAIN GAME APPLICATION LOOP
    // ==========================================
    class GameApp {
      constructor() {
        this.container = document.getElementById('game-container');
        this.fpsElement = document.getElementById('fps-counter');
        this.posElement = document.getElementById('pos-counter');
        this.stanceElement = document.getElementById('stance-indicator');

        this.engine = new GameEngine(this.container);
        this.controls = new TouchControls();
        this.player = new PlayerController(
          this.engine.camera,
          this.engine.scene,
          this.engine.colliders
        );

        this.clock = new THREE.Clock();
        this.frameCount = 0;
        this.fpsTimer = 0;

        this.animate = this.animate.bind(this);
        requestAnimationFrame(this.animate);
      }

      animate() {
        requestAnimationFrame(this.animate);

        const dt = Math.min(this.clock.getDelta(), 0.05);
        const input = this.controls.getInput();

        this.player.update(dt, input);
        this.engine.render();

        this.updateTelemetry(dt);
      }

      updateTelemetry(dt) {
        this.frameCount++;
        this.fpsTimer += dt;

        if (this.fpsTimer >= 0.5) {
          const fps = Math.round(this.frameCount / this.fpsTimer);
          this.fpsElement.textContent = `FPS: ${fps}`;
          this.frameCount = 0;
          this.fpsTimer = 0;

          const p = this.player.position;
          this.posElement.textContent = `POS: ${p.x.toFixed(1)}, ${p.y.toFixed(1)}, ${p.z.toFixed(1)}`;

          let stance = 'STAND';
          if (this.player.isCrouching) stance = 'CROUCH';
          else if (this.controls.sprintActive) stance = 'SPRINT';
          this.stanceElement.textContent = `STANCE: ${stance}`;
        }
      }
    }

    if (document.readyState === 'loading') {
      window.addEventListener('DOMContentLoaded', () => new GameApp());
    } else {
      new GameApp();
    }
  </script>
</body>
</html>
