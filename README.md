<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Thế Giới Tận Thế</title>
    <script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: Arial, sans-serif; }
        body { overflow: hidden; }

        /* === MÀN HÌNH SẢNH CHỜ === */
        #lobby-screen {
            position: fixed;
            width: 100%;
            height: 100%;
            background: url('https://images.unsplash.com/photo-1462331940025-496dfbfc7564?ixlib=rb-4.0.3&auto=format&fit=crop&w=1920&q=90') center / cover no-repeat;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 100;
        }

        .welcome-text {
            font-size: 20px;
            color: #cccccc;
            margin-bottom: 10px;
            text-shadow: 0 0 10px #000;
        }

        .main-title {
            font-size: 52px;
            font-weight: bold;
            color: #ff3333;
            margin-bottom: 60px;
            text-shadow: 0 0 15px #000;
            letter-spacing: 4px;
            text-align: center;
        }

        /* Nút Play chính */
        .play-main-btn {
            width: 240px;
            height: 75px;
            background-color: #2ecc71;
            border: 3px solid #1e8449;
            border-radius: 12px;
            font-size: 30px;
            font-weight: bold;
            color: white;
            box-shadow: 0 5px 15px rgba(0,0,0,0.6);
            cursor: pointer;
            transition: 0.2s;
        }

        .play-main-btn:active {
            transform: scale(0.96);
            background-color: #27ae60;
        }

        /* === KHUNG LỰA CHỌN BẢN ĐỒ === */
        #map-selector {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(50, 50, 50, 0.85);
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            gap: 20px;
            z-index: 200;
            padding: 20px;
        }

        .map-option {
            width: 90%;
            max-width: 500px;
            height: 100px;
            background-color: #222;
            border: 2px solid #444;
            border-radius: 10px;
            display: flex;
            overflow: hidden;
            box-shadow: 0 3px 10px rgba(0,0,0,0.5);
            cursor: pointer;
        }

        /* Phần ảnh 1/3 */
        .map-img {
            width: 33.33%;
            height: 100%;
            background-size: cover;
            background-position: center;
        }

        /* Phần tên vùng 1/5 */
        .map-name {
            width: 20%;
            display: flex;
            align-items: center;
            justify-content: center;
            background-color: #333;
            color: #fff;
            font-weight: bold;
            font-size: 18px;
            border-left: 1px solid #555;
            border-right: 1px solid #555;
        }

        /* Phần nút Play 1/2 còn lại */
        .map-play {
            width: 46.67%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 26px;
            font-weight: bold;
        }

        .play-green { color: #2ecc71; }
        .play-gray { color: #888; }

        /* Nút đóng */
        .close-btn {
            margin-top: 15px;
            padding: 10px 30px;
            background: #e74c3c;
            border: none;
            border-radius: 8px;
            color: white;
            font-size: 18px;
            cursor: pointer;
        }

        /* === KHU VỰC GAME === */
        #game-container {
            display: none;
            width: 100vw;
            height: 100vh;
        }

        #crosshair {
            position: fixed; top: 50%; left: 50%;
            width: 20px; height: 20px;
            transform: translate(-50%, -50%);
            border: 2px solid #fff;
            border-radius: 50%;
            z-index: 10;
            pointer-events: none;
        }

        #hotbar {
            position: fixed; bottom: 20px; left: 50%;
            transform: translateX(-50%);
            display: flex; gap: 6px;
            background: rgba(0,0,0,0.6);
            padding: 8px;
            border-radius: 8px;
            z-index: 10;
        }

        .slot {
            width: 55px; height: 55px;
            background: rgba(255,255,255,0.2);
            border: 2px solid rgba(255,255,255,0.4);
            border-radius: 4px;
            display: flex; align-items: center; justify-content: center;
            color: white; font-weight: bold;
            font-size: 11px;
            text-shadow: 0 0 3px #000;
        }

        .slot.active {
            border-color: #fff;
            background: rgba(255,255,255,0.5);
        }

        #controls {
            position: fixed; bottom: 90px; left: 20px;
            display: flex; flex-direction: column; gap: 8px;
            z-index: 10;
        }

        .btn {
            width: 65px; height: 65px;
            background: rgba(255,255,255,0.3);
            border: none; border-radius: 50%;
            color: white; font-size: 22px;
            font-weight: bold;
            box-shadow: 0 0 5px rgba(0,0,0,0.3);
        }

        #action-btns {
            position: fixed; bottom: 90px; right: 20px;
            display: flex; flex-direction: column; gap: 8px;
            z-index: 10;
        }
    </style>
</head>
<body>

    <!-- === MÀN HÌNH SẢNH CHỜ === -->
    <div id="lobby-screen">
        <p class="welcome-text">chào mừng đến</p>
        <h1 class="main-title">THẾ GIỚI TẬN THẾ</h1>
        <button class="play-main-btn">Play</button>
    </div>

    <!-- === KHUNG LỰA CHỌN BẢN ĐỒ === -->
    <div id="map-selector">
        <!-- Khung 1: Giang Đô -->
        <div class="map-option">
            <div class="map-img" style="background-image: url('https://images.unsplash.com/photo-1503676260728-1c00da094a0b?ixlib=rb-4.0.3&auto=format&fit=crop&w=300&q=80')"></div>
            <div class="map-name">Giang Đô</div>
            <div class="map-play play-green">Play</div>
        </div>

        <!-- Khung 2: Tây Châu -->
        <div class="map-option">
            <div class="map-img" style="background-image: url('https://cdn-icons-png.flaticon.com/512/3064/3064155.png')"></div>
            <div class="map-name">Tây Châu</div>
            <div class="map-play play-gray">Play</div>
        </div>

        <!-- Khung 3: Diêm Nam -->
        <div class="map-option">
            <div class="map-img" style="background-image: url('https://cdn-icons-png.flaticon.com/512/3064/3064155.png')"></div>
            <div class="map-name">Diêm Nam</div>
            <div class="map-play play-gray">Play</div>
        </div>

        <!-- Khung 4: Kinh Thành -->
        <div class="map-option">
            <div class="map-img" style="background-image: url('https://cdn-icons-png.flaticon.com/512/3064/3064155.png')"></div>
            <div class="map-name">Kinh Thành</div>
            <div class="map-play play-gray">Play</div>
        </div>

        <button class="close-btn">Đóng</button>
    </div>

    <!-- === KHU VỰC GAME === -->
    <div id="game-container">
        <div id="crosshair"></div>

        <div id="hotbar">
            <div class="slot active" data-type="grass">Cỏ</div>
            <div class="slot" data-type="dirt">Đất</div>
            <div class="slot" data-type="stone">Đá</div>
            <div class="slot" data-type="wood">Gỗ</div>
            <div class="slot" data-type="leaves">Lá</div>
        </div>

        <div id="controls">
            <button class="btn" id="btn-up">↑</button>
            <div style="display:flex; gap:8px;">
                <button class="btn" id="btn-left">←</button>
                <button class="btn" id="btn-down">↓</button>
                <button class="btn" id="btn-right">→</button>
            </div>
            <button class="btn" id="btn-jump">NHẢY</button>
        </div>

        <div id="action-btns">
            <button class="btn" id="btn-break">PHÁ</button>
            <button class="btn" id="btn-place">ĐẶT</button>
        </div>
    </div>

    <script>
        // === CHỨC NĂNG CHUYỂN MÀN HÌNH ===
        const lobby = document.getElementById('lobby-screen');
        const mapSelector = document.getElementById('map-selector');
        const gameContainer = document.getElementById('game-container');

        // Mở khung chọn bản đồ
        document.querySelector('.play-main-btn').addEventListener('click', () => {
            lobby.style.display = 'none';
            mapSelector.style.display = 'flex';
        });

        // Đóng khung chọn bản đồ
        document.querySelector('.close-btn').addEventListener('click', () => {
            mapSelector.style.display = 'none';
            lobby.style.display = 'flex';
        });

        // Ấn vào Giang Đô (tạm thời chưa làm gì)
        document.querySelector('.map-option').addEventListener('click', () => {
            console.log('Đã chọn Giang Đô');
            // Sau này muốn vào game thì bỏ dấu // ở 2 dòng dưới:
            // mapSelector.style.display = 'none';
            // gameContainer.style.display = 'block';
        });

        // === CODE GAME ===
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB);
        scene.fog = new THREE.Fog(0x87CEEB, 30, 70);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100);
        camera.position.set(0, 2.2, 0);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        gameContainer.appendChild(renderer.domElement);

        const light = new THREE.DirectionalLight(0xffffff, 1);
        light.position.set(20, 50, 20);
        light.castShadow = true;
        scene.add(light);
        scene.add(new THREE.AmbientLight(0xcccccc, 0.5));

        const blockData = {
            grass: { color: 0x56A947 },
            dirt: { color: 0x96673E },
            stone: { color: 0x7A7A7A },
            wood: { color: 0x835A2D },
            leaves: { color: 0x3A8C30 }
        };

        let selectedBlock = 'grass';
        const blocks = new Map();
        const BLOCK_SIZE = 1;

        function createWorld() {
            for (let x = -16; x <= 16; x++) {
                for (let z = -16; z <= 16; z++) {
                    addBlock(x, 0, z, 'grass');
                    addBlock(x, -1, z, 'dirt');
                    addBlock(x, -2, z, 'stone');
                }
            }
            addTree(5, 0, 5);
            addTree(-7, 0, -3);
        }

        function addTree(x, y, z) {
            for (let i = 1; i <= 4; i++) addBlock(x, y + i, z, 'wood');
            for (let dx = -2; dx <= 2; dx++) {
                for (let dz = -2; dz <= 2; dz++) {
                    if (Math.abs(dx) + Math.abs(dz) < 4) addBlock(x + dx, y + 5, z + dz, 'leaves');
                }
            }
        }

        function addBlock(x, y, z, type) {
            const geo = new THREE.BoxGeometry(BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
            const mat = new THREE.MeshStandardMaterial({ color: blockData[type].color });
            const block = new THREE.Mesh(geo, mat);
            block.position.set(x, y, z);
            block.castShadow = true;
            block.receiveShadow = true;
            scene.add(block);
            blocks.set(`${x},${y},${z}`, { mesh: block, type });
        }

        function removeBlock(x, y, z) {
            const key = `${x},${y},${z}`;
            if (blocks.has(key)) {
                scene.remove(blocks.get(key).mesh);
                blocks.delete(key);
            }
        }

        let move = { forward: false, back: false, left: false, right: false };
        let speed = 0.18;
        let yaw = 0, pitch = 0;
        const raycaster = new THREE.Raycaster();

        let touchStart = { x: 0, y: 0 };
        renderer.domElement.addEventListener('touchstart', e => {
            touchStart.x = e.touches[0].clientX;
            touchStart.y = e.touches[0].clientY;
        });

        renderer.domElement.addEventListener('touchmove', e => {
            const dx = e.touches[0].clientX - touchStart.x;
            const dy = e.touches[0].clientY - touchStart.y;
            yaw -= dx * 0.006;
            pitch = THREE.MathUtils.clamp(pitch - dy * 0.006, -Math.PI/2, Math.PI/2);
            touchStart.x = e.touches[0].clientX;
            touchStart.y = e.touches[0].clientY;
        });

        document.querySelectorAll('.slot').forEach(slot => {
            slot.addEventListener('click', () => {
                document.querySelectorAll('.slot').forEach(s => s.classList.remove('active'));
                slot.classList.add('active');
                selectedBlock = slot.dataset.type;
            });
        });

        const btnUp = document.getElementById('btn-up');
        const btnDown = document.getElementById('btn-down');
        const btnLeft = document.getElementById('btn-left');
        const btnRight = document.getElementById('btn-right');
        const btnJump = document.getElementById('btn-jump');

        btnUp.addEventListener('touchstart', () => move.forward = true);
        btnUp.addEventListener('touchend', () => move.forward = false);
        btnDown.addEventListener('touchstart', () => move.back = true);
        btnDown.addEventListener('touchend', () => move.back = false);
        btnLeft.addEventListener('touchstart', () => move.left = true);
        btnLeft.addEventListener('touchend', () => move.left = false);
        btnRight.addEventListener('touchstart', () => move.right = true);
        btnRight.addEventListener('touchend', () => move.right = false);

        let canJump = true;
        btnJump.addEventListener('touchstart', () => {
            if (canJump) {
                camera.position.y += 0.8;
                canJump = false;
                setTimeout(() => canJump = true, 600);
            }
        });

        document.getElementById('btn-break').addEventListener('touchstart', () => {
            raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
            const hit = raycaster.intersectObjects(Array.from(blocks.values()).map(b => b.mesh))[0];
            if (hit) {
                const pos = hit.object.position;
                removeBlock(Math.round(pos.x), Math.round(pos.y), Math.round(pos.z));
            }
        });

        document.getElementById('btn-place').addEventListener('touchstart', () => {
            raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
            const hit = raycaster.intersectObjects(Array.from(blocks.values()).map(b => b.mesh))[0];
            if (hit) {
                const p = hit.point;
                const n = hit.face.normal;
                const x = Math.round(p.x + n.x * 0.5);
                const y = Math.round(p.y + n.y * 0.5);
                const z = Math.round(p.z + n.z * 0.5);
                if (!blocks.has(`${x},${y},${z}`)) addBlock(x, y, z, selectedBlock);
            }
        });

        function animate() {
            requestAnimationFrame(animate);
            camera.rotation.order = 'YXZ';
            camera.rotation.y = yaw;
            camera.rotation.x = pitch;

            const dir = new THREE.Vector3();
            camera.getWorldDirection(dir);
            dir.y = 0;
            dir.normalize();

            const right = new THREE.Vector3();
            right.crossVectors(camera.up, dir).normalize();

            if (move.forward) camera.position.addScaledVector(dir, speed);
            if (move.back) camera.position.addScaledVector(dir, -speed);
            if (move.left) camera.position.addScaledVector(right, speed);
            if (move.right) camera.position.addScaledVector(right, -speed);

            const groundY = Math.round(camera.position.y - 1.8);
            if (!blocks.has(`${Math.round(camera.position.x)},${groundY},${Math.round(camera.position.z)}`)) {
                camera.position.y -= 0.05;
            } else {
                camera.position.y = Math.max(2.2, Math.ceil(camera.position.y));
            }

            renderer.render(scene, camera);
        }

        createWorld();
        animate();

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>
