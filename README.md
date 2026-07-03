<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Game Bắn Súng Không Gian 3D/4D</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.158.0/examples/js/controls/OrbitControls.js"></script>
    <style>
        body { margin: 0; overflow: hidden; background: #000; }
        #ui { position: fixed; top: 0; left: 0; width: 100%; padding: 15px; z-index: 10; color: white; font-family: Arial, sans-serif; }
        .btn { background: #2563eb; padding: 12px 20px; border-radius: 8px; border: none; color: white; font-weight: bold; box-shadow: 0 0 15px #3b82f6; }
        .btn:active { transform: scale(0.95); }
        #controls { position: fixed; bottom: 30px; left: 0; width: 100%; display: flex; justify-content: space-between; padding: 0 20px; z-index: 10; }
    </style>
</head>
<body>
    <div id="ui">
        <h2 class="text-xl font-bold">🔫 Bắn Kẻ Thù Không Gian</h2>
        <p>Điểm số: <span id="score">0</span> | Mạng sống: <span id="lives">5</span></p>
    </div>

    <div id="controls">
        <button class="btn" id="moveLeft">◀</button>
        <button class="btn" id="shoot">BẮN</button>
        <button class="btn" id="moveRight">▶</button>
    </div>

    <script>
        // Khởi tạo không gian 3D
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x000511, 0.08); // Tạo hiệu ứng chiều sâu như 4D
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        document.body.appendChild(renderer.domElement);

        // Ánh sáng
        const light = new THREE.DirectionalLight(0xffffff, 1);
        light.position.set(5, 10, 7);
        scene.add(light);
        scene.add(new THREE.AmbientLight(0x404080, 0.5));

        // Tạo đường hầm không gian - tạo cảm giác 4D chuyển động
        function createTunnel() {
            const geometry = new THREE.CylinderGeometry(12, 15, 100, 32, 32, true);
            const material = new THREE.MeshStandardMaterial({
                color: 0x1a2b6a,
                wireframe: false,
                side: THREE.BackSide,
                transparent: true,
                opacity: 0.7
            });
            const tunnel = new THREE.Mesh(geometry, material);
            tunnel.rotation.x = Math.PI / 2;
            scene.add(tunnel);
            return tunnel;
        }
        const tunnel = createTunnel();

        // Tạo nhân vật người chơi
        const playerGeo = new THREE.BoxGeometry(1.2, 1.2, 2);
        const playerMat = new THREE.MeshStandardMaterial({ color: 0x00ffcc, emissive: 0x006655 });
        const player = new THREE.Mesh(playerGeo, playerMat);
        player.position.z = -5;
        scene.add(player);
        camera.position.set(0, 1.5, 0);
        camera.lookAt(player.position);

        // Biến trò chơi
        let score = 0;
        let lives = 5;
        let enemies = [];
        let bullets = [];
        let gameSpeed = 0.15;
        let tunnelOffset = 0;

        // Tạo kẻ thù
        function createEnemy() {
            const geo = new THREE.OctahedronGeometry(1);
            const mat = new THREE.MeshStandardMaterial({ color: 0xff3333, emissive: 0x660000 });
            const enemy = new THREE.Mesh(geo, mat);
            enemy.position.x = (Math.random() - 0.5) * 12;
            enemy.position.z = -40;
            enemy.rotationSpeed = Math.random() * 0.05;
            scene.add(enemy);
            enemies.push(enemy);
        }

        // Tạo đạn
        function shootBullet() {
            const geo = new THREE.SphereGeometry(0.3);
            const mat = new THREE.MeshStandardMaterial({ color: 0xffff00, emissive: 0xffff00 });
            const bullet = new THREE.Mesh(geo, mat);
            bullet.position.set(player.position.x, player.position.y, player.position.z - 2);
            scene.add(bullet);
            bullets.push(bullet);
        }

        // Điều khiển
        document.getElementById('moveLeft').addEventListener('touchstart', () => player.position.x = Math.max(-8, player.position.x - 0.8));
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
