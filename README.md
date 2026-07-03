<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Sinh Tồn Chiến Trường</title>
    <script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { overflow: hidden; background: #87CEEB; font-family: Arial, sans-serif; }
        #ui { position: fixed; top: 15px; left: 15px; z-index: 10; color: white; text-shadow: 0 0 5px #000; font-size: 18px; font-weight: bold; }
        #joystick { position: fixed; bottom: 30px; left: 30px; width: 120px; height: 120px; background: rgba(255,255,255,0.2); border-radius: 50%; z-index: 10; touch-action: none; }
        #stick { position: absolute; top: 50%; left: 50%; width: 50px; height: 50px; background: rgba(255,255,255,0.6); border-radius: 50%; transform: translate(-50%, -50%); }
        #btnShoot { position: fixed; bottom: 40px; right: 40px; width: 80px; height: 80px; background: #e53e3e; color: white; border: none; border-radius: 50%; font-size: 18px; font-weight: bold; box-shadow: 0 0 15px rgba(0,0,0,0.5); z-index: 10; }
        #btnRun { position: fixed; bottom: 140px; right: 40px; width: 60px; height: 60px; background: #38a169; color: white; border: none; border-radius: 50%; font-size: 14px; z-index: 10; }
    </style>
</head>
<body>
    <div id="ui">
        <p>Đạn: <span id="ammo">30</span> | Máu: <span id="hp">100</span></p>
    </div>
    <div id="joystick">
        <div id="stick"></div>
    </div>
    <button id="btnShoot">BẮN</button>
    <button id="btnRun">CHẠY</button>

    <script>
        // Khởi tạo không gian 3D
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB);
        scene.fog = new THREE.Fog(0x87CEEB, 50, 200);

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
