<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Feeding Frenzy - Adit si Ikan Nemo</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: 'Segoe UI', Arial, sans-serif;
    }
    #ui {
      position: absolute;
      top: 20px;
      left: 20px;
      color: white;
      text-shadow: 2px 2px 8px rgba(0,0,0,0.8);
      font-size: 28px;
      background: rgba(0, 20, 50, 0.6);
      padding: 10px 20px;
      border-radius: 15px;
      backdrop-filter: blur(5px);
      pointer-events: none;
    }
    #nameLabel {
      position: absolute;
      pointer-events: none;
      color: white;
      background: rgba(0, 0, 0, 0.65);
      padding: 4px 14px;
      border-radius: 20px;
      font-size: 20px;
      font-weight: bold;
      transform: translate(-50%, -50%);
      text-shadow: 0 0 10px #ffaa00;
      border: 1px solid rgba(255, 255, 255, 0.3);
    }
    #winMessage {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: gold;
      font-size: 60px;
      font-weight: bold;
      text-shadow: 0 0 30px black, 0 0 15px #ff8800;
      background: rgba(0, 0, 0, 0.7);
      padding: 30px 50px;
      border-radius: 30px;
      display: none;
      text-align: center;
      pointer-events: none;
      backdrop-filter: blur(10px);
    }
    .instruction {
      position: absolute;
      bottom: 30px;
      left: 50%;
      transform: translateX(-50%);
      color: rgba(255,255,255,0.8);
      font-size: 16px;
      background: rgba(0,0,0,0.5);
      padding: 8px 18px;
      border-radius: 20px;
      pointer-events: none;
    }
  </style>
</head>
<body>
  <div id="ui">
    🐟 Skor: <span id="score">0</span> &nbsp; | &nbsp; 📏 Ukuran: <span id="size">0.80</span>
  </div>
  <div id="nameLabel">Adit</div>
  <div id="winMessage">🏆 KAMU MENANG! 🏆<br><span style="font-size:24px;">Adit sudah besar</span></div>
  <div class="instruction">🖱️ Gerakkan mouse untuk mengarahkan Adit & memakan ikan kecil</div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

  <script>
    // ==================== SCENE SETUP ====================
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x001133);
    scene.fog = new THREE.FogExp2(0x001133, 0.0006);

    const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 200);
    camera.position.set(0, 8, 18);
    camera.lookAt(0, 0, 0);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);

    // ==================== LIGHTING ====================
    const ambientLight = new THREE.AmbientLight(0x334466);
    scene.add(ambientLight);
    
    const sunLight = new THREE.DirectionalLight(0xffffff, 0.9);
    sunLight.position.set(15, 25, 10);
    sunLight.castShadow = true;
    sunLight.receiveShadow = true;
    sunLight.shadow.mapSize.width = 512;
    sunLight.shadow.mapSize.height = 512;
    sunLight.shadow.camera.near = 1;
    sunLight.shadow.camera.far = 60;
    sunLight.shadow.camera.left = -35;
    sunLight.shadow.camera.right = 35;
    sunLight.shadow.camera.top = 35;
    sunLight.shadow.camera.bottom = -35;
    scene.add(sunLight);

    const underwaterLight = new THREE.PointLight(0x2255aa, 0.6, 30);
    underwaterLight.position.set(0, 2, 0);
    scene.add(underwaterLight);

    // ==================== LANDSCAPE ====================
    // Dasar laut
    const floorGeo = new THREE.PlaneGeometry(100, 100);
    const floorMat = new THREE.MeshPhongMaterial({ color: 0x2a3f4a, side: THREE.DoubleSide });
    const floor = new THREE.Mesh(floorGeo, floorMat);
    floor.rotation.x = -Math.PI / 2;
    floor.position.y = -5;
    floor.receiveShadow = true;
    scene.add(floor);

    // Rumput laut
    for (let i = 0; i < 25; i++) {
      const seaweed = new THREE.Group();
      const stemGeo = new THREE.CylinderGeometry(0.15, 0.25, 1.8 + Math.random() * 3);
      const stem = new THREE.Mesh(stemGeo, new THREE.MeshPhongMaterial({ color: 0x2d5a27 }));
      stem.castShadow = true;
      stem.receiveShadow = true;
      seaweed.add(stem);
      
      for (let j = 0; j < 4; j++) {
        const leafGeo = new THREE.SphereGeometry(0.25, 4);
        const leaf = new THREE.Mesh(leafGeo, new THREE.MeshPhongMaterial({ color: 0x3a7d32 }));
        leaf.position.set(0, 0.6 + j * 0.7, 0.35);
        leaf.scale.set(0.6, 0.15, 0.15);
        seaweed.add(leaf);
      }
      seaweed.position.set((Math.random() - 0.5) * 45, -4.8, (Math.random() - 0.5) * 45);
      scene.add(seaweed);
    }

    // Batu karang
    for (let i = 0; i < 12; i++) {
      const rockGeo = new THREE.DodecahedronGeometry(0.5 + Math.random() * 0.8);
      const rock = new THREE.Mesh(rockGeo, new THREE.MeshPhongMaterial({ color: 0x5a4a3a }));
      rock.position.set((Math.random() - 0.5) * 40, -4.6, (Math.random() - 0.5) * 40);
      rock.scale.set(1, 0.6 + Math.random() * 0.5, 1);
      rock.castShadow = true;
      rock.receiveShadow = true;
      scene.add(rock);
    }

    // ==================== PLAYER FISH (ADIT) ====================
    function createClownfish() {
      const fish = new THREE.Group();
      
      // Badan
      const bodyGeo = new THREE.SphereGeometry(1, 32, 16);
      bodyGeo.scale(1.5, 0.75, 0.85);
      const body = new THREE.Mesh(bodyGeo, new THREE.MeshPhongMaterial({ color: 0xff6600, shininess: 25 }));
      body.castShadow = true;
      body.receiveShadow = true;
      fish.add(body);
      
      // Garis putih (tiga garis)
      const whiteMat = new THREE.MeshPhongMaterial({ color: 0xffffff });
      const stripePositions = [-0.6, 0, 0.6];
      stripePositions.forEach(z => {
        const stripeGeo = new THREE.TorusGeometry(0.8, 0.18, 8, 20);
        const stripe = new THREE.Mesh(stripeGeo, whiteMat);
        stripe.position.z = z;
        stripe.rotation.x = Math.PI / 2;
        fish.add(stripe);
      });
      
      // Sirip ekor
      const tailGeo = new THREE.ConeGeometry(0.75, 0.9, 8);
      const tail = new THREE.Mesh(tailGeo, new THREE.MeshPhongMaterial({ color: 0xff8800 }));
      tail.position.z = -1.5;
      tail.rotation.x = Math.PI / 2;
      tail.castShadow = true;
      fish.add(tail);
      
      // Sirip punggung
      const dorsalGeo = new THREE.ConeGeometry(0.55, 0.8, 8);
      const dorsal = new THREE.Mesh(dorsalGeo, new THREE.MeshPhongMaterial({ color: 0xff8800 }));
      dorsal.position.set(0, 0.75, 0);
      dorsal.rotation.x = 0.3;
      dorsal.castShadow = true;
      fish.add(dorsal);
      
      // Sirip samping
      for (let side = -1; side <= 1; side += 2) {
        const finGeo = new THREE.ConeGeometry(0.4, 0.65, 6);
        const fin = new THREE.Mesh(finGeo, new THREE.MeshPhongMaterial({ color: 0xffaa00 }));
        fin.position.set(side * 0.85, -0.1, 0.5);
        fin.rotation.z = side * 0.6;
        fin.rotation.x = 0.4;
        fin.castShadow = true;
        fish.add(fin);
      }
      
      // Mata
      for (let side = -1; side <= 1; side += 2) {
        const eyeWhite = new THREE.Mesh(new THREE.SphereGeometry(0.22, 16, 16), new THREE.MeshPhongMaterial({ color: 0xffffff }));
        eyeWhite.position.set(side * 0.55, 0.15, 0.95);
        fish.add(eyeWhite);
        const eyePupil = new THREE.Mesh(new THREE.SphereGeometry(0.12, 16, 16), new THREE.MeshPhongMaterial({ color: 0x000000 }));
        eyePupil.position.set(side * 0.65, 0.15, 1.05);
        fish.add(eyePupil);
      }
      
      return fish;
    }

    const playerFish = createClownfish();
    playerFish.castShadow = true;
    playerFish.receiveShadow = true;
    scene.add(playerFish);
    playerFish.position.set(0, 0, 0);
    playerFish.scale.set(0.8, 0.8, 0.8);

    // Label nama "Adit" di atas ikan (sprite 3D)
    function createNameSprite(text) {
      const canvas = document.createElement('canvas');
      canvas.width = 256;
      canvas.height = 64;
      const ctx = canvas.getContext('2d');
      ctx.fillStyle = 'rgba(0,0,0,0)';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.font = 'Bold 34px "Segoe UI", Arial';
      ctx.fillStyle = 'white';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.shadowColor = 'rgba(0,0,0,0.8)';
      ctx.shadowBlur = 10;
      ctx.fillText(text, 128, 32);
      const texture = new THREE.CanvasTexture(canvas);
      const material = new THREE.SpriteMaterial({ map: texture, depthTest: false, depthWrite: false });
      const sprite = new THREE.Sprite(material);
      sprite.scale.set(2.2, 0.55, 1);
      sprite.position.y = 1.9;
      return sprite;
    }
    
    const nameSprite = createNameSprite('Adit');
    playerFish.add(nameSprite);

    // ==================== MAKANAN (IKAN KECIL) ====================
    const foodFishes = [];
    const FOOD_COUNT = 18;

    function createFoodFish() {
      const fish = new THREE.Group();
      const bodyColor = new THREE.Color(Math.random() * 0xffffff);
      
      const bodyGeo = new THREE.SphereGeometry(0.32, 8, 8);
      const body = new THREE.Mesh(bodyGeo, new THREE.MeshPhongMaterial({ color: bodyColor, shininess: 20 }));
      body.castShadow = true;
      body.receiveShadow = true;
      fish.add(body);
      
      const tailGeo = new THREE.ConeGeometry(0.2, 0.35, 6);
      const tail = new THREE.Mesh(tailGeo, new THREE.MeshPhongMaterial({ color: bodyColor }));
      tail.position.z = -0.45;
      tail.rotation.x = Math.PI / 2;
      fish.add(tail);
      
      for (let s = -1; s <= 1; s += 2) {
        const eye = new THREE.Mesh(new THREE.SphereGeometry(0.1, 6), new THREE.MeshPhongMaterial({ color: 0xffffff }));
        eye.position.set(s * 0.16, 0.1, 0.28);
        fish.add(eye);
      }
      
      return fish;
    }

    function spawnFoodFish() {
      const fish = createFoodFish();
      const angle = Math.random() * Math.PI * 2;
      const distance = 9 + Math.random() * 16;
      fish.position.set(
        Math.cos(angle) * distance,
        (Math.random() * 3) - 1.2,
        Math.sin(angle) * distance
      );
      fish.rotation.y = Math.random() * Math.PI * 2;
      fish.userData = {
        speed: 0.025 + Math.random() * 0.04,
        direction: new THREE.Vector3(Math.random() - 0.5, 0, Math.random() - 0.5).normalize(),
        turnTimer: 40 + Math.random() * 120
      };
      scene.add(fish);
      foodFishes.push(fish);
    }

    for (let i = 0; i < FOOD_COUNT; i++) {
      spawnFoodFish();
    }

    // ==================== GELEMBUNG ====================
    const bubbles = [];
    function createBubble() {
      const geo = new THREE.SphereGeometry(0.1 + Math.random() * 0.25, 8, 8);
      const mat = new THREE.MeshPhongMaterial({ 
        color: 0xaaccff, 
        transparent: true, 
        opacity: 0.45,
        shininess: 50
      });
      const bubble = new THREE.Mesh(geo, mat);
      bubble.position.set(
        (Math.random() - 0.5) * 25,
        -4 + Math.random() * 2,
        (Math.random() - 0.5) * 25
      );
      bubble.userData = { speed: 0.015 + Math.random() * 0.05 };
      scene.add(bubble);
      bubbles.push(bubble);
    }
    for (let i = 0; i < 35; i++) createBubble();

    // ==================== KONTROL MOUSE ====================
    const mouse = new THREE.Vector2();
    const raycaster = new THREE.Raycaster();
    const groundPlane = new THREE.Plane(new THREE.Vector3(0, 1, 0), 0);
    const targetPosition = new THREE.Vector3(0, 0, 0);

    window.addEventListener('mousemove', (event) => {
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    });

    function updateTargetFromMouse() {
      raycaster.setFromCamera(mouse, camera);
      const intersection = new THREE.Vector3();
      if (raycaster.ray.intersectPlane(groundPlane, intersection)) {
        intersection.x = THREE.MathUtils.clamp(intersection.x, -22, 22);
        intersection.z = THREE.MathUtils.clamp(intersection.z, -22, 22);
        intersection.y = 0;
        targetPosition.copy(intersection);
      }
    }

    // ==================== GAME STATE ====================
    let score = 0;
    let playerScale = 0.8;
    const MAX_SCALE = 2.6;
    const GROWTH_FACTOR = 0.06;
    const scoreElement = document.getElementById('score');
    const sizeElement = document.getElementById('size');
    const winMessage = document.getElementById('winMessage');
    let gameActive = true;

    // ==================== ANIMASI UTAMA ====================
    function animate() {
      requestAnimationFrame(animate);
      if (!gameActive) {
        // Tetap render walau game selesai, biar pemandangan masih terlihat
        renderer.render(scene, camera);
        return;
      }
      
      updateTargetFromMouse();
      
      // Gerakan halus ikan Adit menuju target
      const fishPos = playerFish.position;
      const direction = new THREE.Vector3().subVectors(targetPosition, fishPos);
      const distance = direction.length();
      
      if (distance > 0.15) {
        direction.normalize();
        const speed = 0.18;
        playerFish.position.add(direction.clone().multiplyScalar(Math.min(speed, distance)));
        
        // Rotasi ikan menghadap arah gerak (depan = sumbu Z positif)
        const angle = Math.atan2(direction.x, direction.z);
        playerFish.rotation.y = angle;
      }
      
      // Kamera mengikuti ikan dari belakang dan atas
      const quat = new THREE.Quaternion().setFromEuler(new THREE.Euler(0, playerFish.rotation.y, 0));
      const offset = new THREE.Vector3(0, 5.5, -9);
      offset.applyQuaternion(quat);
      const camTarget = fishPos.clone().add(offset);
      camera.position.lerp(camTarget, 0.08);
      camera.lookAt(fishPos.x, fishPos.y + 1.2, fishPos.z);
      
      // Gerakan ikan makanan
      foodFishes.forEach(food => {
        const data = food.userData;
        food.position.add(data.direction.clone().multiplyScalar(data.speed));
        
        // Batas area
        if (Math.abs(food.position.x) > 28 || Math.abs(food.position.z) > 28 || food.position.y > 6 || food.position.y < -4) {
          const angle = Math.random() * Math.PI * 2;
          const dist = 10 + Math.random() * 14;
          food.position.set(Math.cos(angle) * dist, Math.random() * 3 - 1, Math.sin(angle) * dist);
        }
        
        data.turnTimer--;
        if (data.turnTimer <= 0) {
          data.direction = new THREE.Vector3((Math.random() - 0.5) * 2, 0, (Math.random() - 0.5) * 2).normalize();
          data.turnTimer = 50 + Math.random() * 110;
        }
        food.rotation.y = Math.atan2(data.direction.x, data.direction.z);
      });
      
      // Deteksi makan (berdasarkan jarak)
      const eatRadius = playerScale * 1.1;
      for (let i = foodFishes.length - 1; i >= 0; i--) {
        const food = foodFishes[i];
        const distToFood = playerFish.position.distanceTo(food.position);
        if (distToFood < eatRadius + 0.4) {
          // Makan!
          scene.remove(food);
          foodFishes.splice(i, 1);
          
          score += 10;
          playerScale = Math.min(MAX_SCALE, playerScale + GROWTH_FACTOR);
          playerFish.scale.set(playerScale, playerScale, playerScale);
          
          scoreElement.textContent = score;
          sizeElement.textContent = playerScale.toFixed(2);
          
          // Spawn ikan baru sebagai pengganti
          spawnFoodFish();
          
          // Cek kemenangan
          if (playerScale >= MAX_SCALE) {
            gameActive = false;
            winMessage.style.display = 'block';
          }
        }
      }
      
      // Animasi gelembung
      bubbles.forEach(bubble => {
        bubble.position.y += bubble.userData.speed;
        if (bubble.position.y > 7) {
          bubble.position.y = -5;
          bubble.position.x = (Math.random() - 0.5) * 25;
          bubble.position.z = (Math.random() - 0.5) * 25;
        }
      });
      
      // Update posisi label nama di layar (HTML overlay)
      const worldPos = playerFish.position.clone();
      worldPos.y += 2.0;
      const screenPos = worldPos.clone().project(camera);
      const nameLabel = document.getElementById('nameLabel');
      const x = (screenPos.x * 0.5 + 0.5) * window.innerWidth;
      const y = (-screenPos.y * 0.5 + 0.5) * window.innerHeight;
      
      if (screenPos.z > 1) {
        nameLabel.style.display = 'none';
      } else {
        nameLabel.style.display = 'block';
        nameLabel.style.left = x + 'px';
        nameLabel.style.top = y + 'px';
      }
      
      renderer.render(scene, camera);
    }

    animate();

    // ==================== RESPONSIVE ====================
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
