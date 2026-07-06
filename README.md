<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Giang Đô - Game</title>
    <script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: Arial, sans-serif; overflow: hidden; }

        /* === MÀN HÌNH SẢNH CHỜ === */
        #lobby-screen {
            position: fixed;
            top: 0;
            left: 0;
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
            font-size: 18px;
            color: #cccccc;
            margin-bottom: 8px;
            text-shadow: 0 0 8px rgba(0,0,0,0.8);
        }

        .title-text {
            font-size: 48px;
            font-weight: bold;
            color: #ff3333;
            margin-bottom: 50px;
            text-shadow: 0 0 12px rgba(0,0,0,0.9);
            letter-spacing: 3px;
        }

        .play-button {
            width: 220px;
            height: 70px;
            background-color: #2ecc71;
            border: none;
            border-radius: 8px;
            font-size: 28px;
            font-weight: bold;
            color: white;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
            cursor: pointer;
            transition: transform 0.2s ease;
        }

        .play-button:active {
            transform: scale(0.96);
            background-color: #27ae60;
        }

        /* === KHU VỰC GAME (ẨN BAN ĐẦU) === */
        #game-container {
            display: none;
            width: 100vw;
            height: 100vh;
        }

        /* Giữ nguyên giao diện game cũ */
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
        <h1 class="title-text">Giang đô</h1>
        <button class="play-button">Play</button>
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
        // === NÚT PLAY - TẠM THỜI CHƯA LÀM GÌ ===
        document.querySelector('.play-button').addEventListener('click', () => {
            // Bạn có thể thêm chức năng sau này, bây giờ để trống
            console.log('Nút Play đã được nhấn');
            // Khi muốn vào game thì bỏ dấu // ở 2 dòng bên dưới:
            // document.getElementById('lobby-screen').style.display = 'none';
            // document.getElementById('game-container').style.display = 'block';
        });

        // === CODE GAME GIỐNG NHƯ TRƯỚC ===
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB);
        scene.fog = new THREE.Fog(0x87CEEB, 30, 70);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100);
        camera.position.set(0, 2.2, 0);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        document.getElementById('game-container').appendChild(renderer.domElement);

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

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 200);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        document.body.appendChild(renderer.domElement);

        // Ánh sáng
        const sun = new THREE.DirectionalLight(0xffffff, 1);
        sun.position.set(50, 80, 40);
        sun.castShadow = true;
        scene.add(sun);
        scene.add(new THREE.AmbientLight(0xf0f0f0, 0.5));

        // Tạo mặt đất rộng
        const groundGeo = new THREE.PlaneGeometry(300, 300);
        const groundMat = new THREE.MeshStandardMaterial({ color: 0x228B22 });
        const ground = new THREE.Mesh(groundGeo, groundMat);
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        scene.add(ground);

        // Thêm một số cây/cột làm vật cản
        function createTree(x, z) {
            const trunk = new THREE.Mesh(
                new THREE.CylinderGeometry(0.5, 0.7, 5),
                new THREE.MeshStandardMaterial({ color: 0x8B4513 })
            );
            trunk.position.set(x, 2.5, z);
            trunk.castShadow = true;
            const leaves = new THREE.Mesh(
                new THREE.SphereGeometry(3),
                new THREE.MeshStandardMaterial({ color: 0x006400 })
            );
            leaves.position.set(x, 7, z);
            leaves.castShadow = true;
            scene.add(trunk, leaves);
        }
        for (let i = 0; i < 15; i++) {
            const x = (Math.random() - 0.5) * 200;
            const z = (Math.random() - 0.5) * 200;
            if (Math.abs(x) > 10 || Math.abs(z) > 10) createTree(x, z);
        }

        // Tạo nhân vật người chơi
        const playerGeo = new THREE.CapsuleGeometry(0.8, 1.5, 4, 8);
        const playerMat = new THREE.MeshStandardMaterial({ color: 0x4169E1 });
        const player = new THREE.Mesh(playerGeo, playerMat);
        player.position.set(0, 1.2, 0);
        player.castShadow = true;
        scene.add(player);

        // Súng
        const gunGeo = new THREE.BoxGeometry(0.3, 0.2, 1);
        const gunMat = new THREE.MeshStandardMaterial({ color: 0x222222 });
        const gun = new THREE.Mesh(gunGeo, gunMat);
        gun.position.set(0.6, 0.3, -1);
        player.add(gun);

        // Camera theo sau nhân vật
        camera.position.set(0, 4, 8);
        let cameraAngle = 0;

        // Thông số trò chơi
        let speed = 0.15;
        let runSpeed = 0.3;
        let currentSpeed = speed;
        let ammo = 30;
        let hp = 100;
        let bullets = [];
        const maxAmmo = 30;

        // --- Điều khiển cần điều khiển ---
        const joystick = document.getElementById('joystick');
        const stick = document.getElementById('stick');
        let isDragging = false;
        let joyX = 0, joyY = 0;

        joystick.addEventListener('touchstart', (e) => {
            isDragging = true;
            updateStick(e.touches[0].clientX, e.touches[0].clientY);
        });
        document.addEventListener('touchmove', (e) => {
            if (!isDragging) return;
            updateStick(e.touches[0].clientX, e.touches[0].clientY);
        });
        document.addEventListener('touchend', () => {
            isDragging = false;
            joyX = 0; joyY = 0;
            stick.style.transform = `translate(-50%, -50%)`;
        });

        function updateStick(clientX, clientY) {
            const rect = joystick.getBoundingClientRect();
            let dx = clientX - rect.left - rect.width / 2;
            let dy = clientY - rect.top - rect.height / 2;
            const maxDist = rect.width / 2 - 25;
            const dist = Math.sqrt(dx * dx + dy * dy);
            if (dist > maxDist) {
                dx *= maxDist / dist;
                dy *= maxDist / dist;
            }
            joyX = dx / maxDist;
            joyY = dy / maxDist;
            stick.style.transform = `translate(calc(-50% + ${dx}px), calc(-50% + ${dy}px))`;
        }

        // --- Nút bắn & chạy ---
        document.getElementById('btnShoot').addEventListener('touchstart', shoot);
        document.getElementById('btnRun').addEventListener('touchstart', () => currentSpeed = runSpeed);
        document.getElementById('btnRun').addEventListener('touchend', () => currentSpeed = speed);

        function shoot() {
            if (ammo <= 0) return;
            ammo--;
            document.getElementById('ammo').textContent = ammo;

            const bullet = new THREE.Mesh(
                new THREE.SphereGeometry(0.15),
                new THREE.MeshStandardMaterial({ color: 0xffff00, emissive: 0xffff00 })
            );
            bullet.position.copy(player.position);
            bullet.position.y += 1;
            const dir = new THREE.Vector3(0, 0, -1).applyQuaternion(player.quaternion);
            bullet.velocity = dir.multiplyScalar(1.2);
            scene.add(bullet);
            bullets.push(bullet);
        }

        // --- Vòng lặp cập nhật ---
        function animate() {
            requestAnimationFrame(animate);

            // Di chuyển nhân vật
            if (joyX !== 0 || joyY !== 0) {
                cameraAngle += joyX * 0.03;
                const moveX = Math.sin(cameraAngle) * currentSpeed * joyY;
                const moveZ = Math.cos(cameraAngle) * currentSpeed * joyY;
                player.position.x += moveX;
                player.position.z += moveZ;
                player.rotation.y = cameraAngle;
            }

            // Cập nhật vị trí camera theo sau
            camera.position.x = player.position.x - Math.sin(cameraAngle) * 8;
            camera.position.z = player.position.z - Math.cos(cameraAngle) * 8;
            camera.position.y = player.position.y + 4;
            camera.lookAt(player.position.x, player.position.y + 1, player.position.z);

            // Di chuyển đạn
            bullets.forEach((b, i) => {
                b.position.add(b.velocity);
                if (b.position.distanceTo(player.position) > 100) {
                    scene.remove(b);
                    bullets.splice(i, 1);
                }
            });

            renderer.render(scene, camera);
        }

        // Điều chỉnh kích thước màn hình
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>
document.getElementById('moveRight').addEventListener('touchstart', () => player.position.x = Math.min(8, player.position.x + 0.8));
        document.getElementById('shoot').addEventListener('touchstart', shootBullet);

        // Kiểm tra va chạm
        function checkCollision(obj1, obj2, distance = 1.5) {
            return obj1.position.distanceTo(obj2.position) < distance;
        }

        // Vòng lặp trò chơi
        function animate() {
            requestAnimationFrame(animate);

            // Hiệu ứng chuyển động đường hầm 4D
            tunnelOffset += 0.05;
            tunnel.material.mapOffset = new THREE.Vector2(0, tunnelOffset);
            tunnel.rotation.z += 0.003;

            // Di chuyển kẻ thù
            enemies.forEach((enemy, index) => {
                enemy.position.z += gameSpeed;
                enemy.rotation.x += enemy.rotationSpeed;
                enemy.rotation.y += enemy.rotationSpeed;

                if (enemy.position.z > 10) {
                    scene.remove(enemy);
                    enemies.splice(index, 1);
                    lives--;
                    document.getElementById('lives').textContent = lives;
                    if (lives <= 0) alert("Trò chơi kết thúc! Điểm của bạn: " + score);
                }
            });

            // Di chuyển đạn
            bullets.forEach((bullet, bIndex) => {
                bullet.position.z -= 0.4;
                if (bullet.position.z < -50) {
                    scene.remove(bullet);
                    bullets.splice(bIndex, 1);
                }

                // Kiểm tra bắn trúng
                enemies.forEach((enemy, eIndex) => {
                    if (checkCollision(bullet, enemy)) {
                        scene.remove(bullet);
                        scene.remove(enemy);
                        bullets.splice(bIndex, 1);
                        enemies.splice(eIndex, 1);
                        score += 10;
                        document.getElementById('score').textContent = score;
                    }
                });
            });

            // Tạo kẻ thù định kỳ
            if (Math.random() < 0.02) createEnemy();

            renderer.render(scene, camera);
        }

        // Điều chỉnh kích thước màn hình
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>
# GAME
