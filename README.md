# tear
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AR Simple - Sin Marcadores</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body { 
            margin: 0; 
            overflow: hidden; 
            background: #000; 
            font-family: 'Arial', sans-serif;
            touch-action: none;
        }

        #video {
            position: absolute;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            object-fit: cover;
            z-index: 1;
        }

        #canvas-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 2;
            pointer-events: none;
        }

        canvas {
            display: block;
            pointer-events: all;
        }

        /* Panel de controles lateral */
        .controls-panel {
            position: fixed;
            right: 20px;
            top: 50%;
            transform: translateY(-50%);
            display: flex;
            flex-direction: column;
            gap: 15px;
            z-index: 100;
        }

        .control-btn {
            width: 60px;
            height: 60px;
            border-radius: 50%;
            border: 3px solid white;
            background: rgba(0, 150, 255, 0.9);
            color: white;
            font-size: 24px;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.4);
            transition: all 0.2s;
            user-select: none;
        }

        .control-btn:active {
            transform: scale(0.9);
            background: rgba(0, 180, 255, 1);
        }

        .control-btn.active {
            background: rgba(255, 100, 100, 0.9);
            border-color: #ffaa00;
        }

        .control-btn.small {
            width: 50px;
            height: 50px;
            font-size: 20px;
        }

        /* Info panel superior */
        #info-panel {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(10, 20, 30, 0.85);
            backdrop-filter: blur(5px);
            border-left: 6px solid #00ccff;
            border-radius: 12px;
            padding: 15px 20px;
            color: white;
            font-size: 14px;
            max-width: 250px;
            z-index: 100;
        }

        #info-panel h3 {
            margin: 0 0 8px 0;
            color: #88ddff;
            font-size: 16px;
        }

        #info-panel p {
            margin: 5px 0;
            line-height: 1.3;
            font-size: 12px;
        }

        .highlight {
            color: #ffaa66;
            font-weight: bold;
        }

        /* Instrucciones iniciales */
        #welcome-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            z-index: 1000;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            text-align: center;
            padding: 20px;
        }

        #welcome-screen h1 {
            font-size: 32px;
            margin-bottom: 20px;
        }

        #welcome-screen p {
            font-size: 16px;
            margin-bottom: 30px;
            max-width: 350px;
            line-height: 1.5;
        }

        #start-btn {
            padding: 15px 40px;
            font-size: 20px;
            background: white;
            color: #667eea;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 8px 20px rgba(0,0,0,0.3);
            font-weight: bold;
        }

        #start-btn:active {
            transform: scale(0.95);
        }

        .hidden {
            display: none !important;
        }

        /* Indicador de modo */
        #mode-indicator {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(100, 50, 200, 0.9);
            color: white;
            padding: 15px 30px;
            border-radius: 30px;
            font-size: 18px;
            border: 2px solid white;
            z-index: 150;
            display: none;
            pointer-events: none;
            box-shadow: 0 8px 20px rgba(0,0,0,0.5);
        }

        /* Contador de objetos */
        #object-counter {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 10px 20px;
            border-radius: 20px;
            font-size: 14px;
            z-index: 100;
        }
    </style>
</head>
<body>
    <!-- Pantalla de bienvenida -->
    <div id="welcome-screen">
        <h1>🎨 AR Simple</h1>
        <p>
            Coloca modelos 3D en tu espacio usando la cámara.<br><br>
            <strong>Controles:</strong><br>
            • Toca para colocar objetos<br>
            • 1 dedo: rotar<br>
            • 2 dedos: escalar<br>
            • Botones laterales: cambiar objeto
        </p>
        <button id="start-btn">🚀 INICIAR</button>
    </div>

    <!-- Video de cámara -->
    <video id="video" autoplay playsinline muted></video>

    <!-- Canvas de Three.js -->
    <div id="canvas-container"></div>

    <!-- Panel de información -->
    <div id="info-panel">
        <h3>🎮 Modo Actual</h3>
        <p><span class="highlight">Objeto:</span> <span id="current-object">Oxígeno</span></p>
        <p><span class="highlight">Acción:</span> <span id="current-action">Toca para colocar</span></p>
    </div>

    <!-- Controles laterales -->
    <div class="controls-panel">
        <button class="control-btn active" onclick="selectObject('oxygen')" title="Oxígeno">
            🔴
        </button>
        <button class="control-btn" onclick="selectObject('hydrogen')" title="Hidrógeno">
            ⚪
        </button>
        <button class="control-btn" onclick="selectObject('water')" title="Agua">
            💧
        </button>
        <button class="control-btn small" onclick="selectObject('box')" title="Cubo">
            📦
        </button>
        <button class="control-btn small" onclick="selectObject('sphere')" title="Esfera">
            ⚫
        </button>
        <div style="height: 20px;"></div>
        <button class="control-btn small" onclick="clearAll()" title="Limpiar todo">
            🗑️
        </button>
        <button class="control-btn small" onclick="toggleInfo()" title="Info">
            ℹ️
        </button>
    </div>

    <!-- Indicador de modo -->
    <div id="mode-indicator">🔄 Rotación</div>

    <!-- Contador -->
    <div id="object-counter">Objetos: 0</div>

    <script>
        // ==========================================
        // CONFIGURACIÓN INICIAL
        // ==========================================
        
        let scene, camera, renderer;
        let objects = [];
        let selectedObject = null;
        let currentObjectType = 'oxygen';
        let objectCount = 0;
        
        // Touch controls
        let touches = [];
        let touchStartPos = { x: 0, y: 0 };
        let initialRotation = { x: 0, y: 0, z: 0 };
        let initialScale = 1;
        let initialDistance = 0;
        let manipulationMode = 'none';
        
        const modeIndicator = document.getElementById('mode-indicator');
        const infoPanel = document.getElementById('info-panel');
        const welcomeScreen = document.getElementById('welcome-screen');
        const startBtn = document.getElementById('start-btn');
        const video = document.getElementById('video');
        const canvasContainer = document.getElementById('canvas-container');

        // ==========================================
        // INICIAR APLICACIÓN
        // ==========================================
        
        startBtn.addEventListener('click', async () => {
            welcomeScreen.classList.add('hidden');
            await initCamera();
            initThreeJS();
            animate();
        });

        // ==========================================
        // CÁMARA
        // ==========================================
        
        async function initCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({
                    video: { 
                        facingMode: 'environment',
                        width: { ideal: 1280 },
                        height: { ideal: 720 }
                    }
                });
                video.srcObject = stream;
                video.play();
            } catch (error) {
                alert('Error al acceder a la cámara: ' + error.message);
            }
        }

        // ==========================================
        // THREE.JS SETUP
        // ==========================================
        
        function initThreeJS() {
            scene = new THREE.Scene();
            
            camera = new THREE.PerspectiveCamera(
                60,
                window.innerWidth / window.innerHeight,
                0.01,
                1000
            );
            camera.position.set(0, 0, 0);
            
            // Iluminación
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.9);
            scene.add(ambientLight);
            
            const directionalLight = new THREE.DirectionalLight(0xffffff, 1.2);
            directionalLight.position.set(1, 2, 1);
            scene.add(directionalLight);
            
            const backLight = new THREE.DirectionalLight(0x99ccff, 0.6);
            backLight.position.set(-1, 1, -1);
            scene.add(backLight);
            
            // Renderer
            renderer = new THREE.WebGLRenderer({ 
                antialias: true, 
                alpha: true 
            });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(window.devicePixelRatio);
            canvasContainer.appendChild(renderer.domElement);
            
            window.addEventListener('resize', onWindowResize);
        }

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // ==========================================
        // CREAR OBJETOS
        // ==========================================
        
        function createOxygen() {
            const group = new THREE.Group();
            
            const sphereGeo = new THREE.SphereGeometry(0.08, 32, 16);
            const sphereMat = new THREE.MeshStandardMaterial({ 
                color: 0xff5555, 
                roughness: 0.25, 
                metalness: 0.3,
                emissive: 0xff5555,
                emissiveIntensity: 0.2
            });
            const sphere = new THREE.Mesh(sphereGeo, sphereMat);
            group.add(sphere);
            
            const torusGeo = new THREE.TorusGeometry(0.14, 0.012, 16, 32);
            const torusMat = new THREE.MeshStandardMaterial({ 
                color: 0x88ccff, 
                emissive: 0x88ccff, 
                emissiveIntensity: 0.2,
                transparent: true, 
                opacity: 0.6
            });
            
            const torus1 = new THREE.Mesh(torusGeo, torusMat);
            torus1.rotation.x = Math.PI / 2;
            group.add(torus1);
            
            const torus2 = new THREE.Mesh(torusGeo, torusMat.clone());
            torus2.rotation.y = Math.PI / 2;
            torus2.scale.set(1.1, 1.1, 1.1);
            group.add(torus2);
            
            group.userData.type = 'oxygen';
            return group;
        }

        function createHydrogen() {
            const group = new THREE.Group();
            
            const hGeo = new THREE.SphereGeometry(0.05, 24, 12);
            const hMat = new THREE.MeshStandardMaterial({ 
                color: 0xffffff, 
                roughness: 0.3, 
                metalness: 0.1,
                emissive: 0x446688,
                emissiveIntensity: 0.1
            });
            
            const h1 = new THREE.Mesh(hGeo, hMat);
            h1.position.x = -0.075;
            group.add(h1);
            
            const h2 = new THREE.Mesh(hGeo, hMat.clone());
            h2.position.x = 0.075;
            group.add(h2);
            
            const bondGeo = new THREE.CylinderGeometry(0.015, 0.015, 0.15, 6);
            const bondMat = new THREE.MeshStandardMaterial({ 
                color: 0xaaaaaa, 
                roughness: 0.6, 
                metalness: 0.8 
            });
            const bond = new THREE.Mesh(bondGeo, bondMat);
            bond.rotation.z = Math.PI / 2;
            group.add(bond);
            
            group.userData.type = 'hydrogen';
            return group;
        }

        function createWater() {
            const group = new THREE.Group();
            
            const oGeo = new THREE.SphereGeometry(0.08, 32, 16);
            const oMat = new THREE.MeshStandardMaterial({ 
                color: 0xff3333, 
                roughness: 0.25, 
                metalness: 0.2,
                emissive: 0x331111,
                emissiveIntensity: 0.3
            });
            const oxygen = new THREE.Mesh(oGeo, oMat);
            group.add(oxygen);
            
            const hGeo = new THREE.SphereGeometry(0.05, 24, 12);
            const hMat = new THREE.MeshStandardMaterial({ 
                color: 0xffffff, 
                roughness: 0.3, 
                metalness: 0.1
            });
            
            const h1 = new THREE.Mesh(hGeo, hMat);
            h1.position.set(0.1, 0.075, 0);
            group.add(h1);
            
            const h2 = new THREE.Mesh(hGeo, hMat.clone());
            h2.position.set(0.1, -0.075, 0);
            group.add(h2);
            
            const bondGeo = new THREE.CylinderGeometry(0.018, 0.018, 0.125, 6);
            const bondMat = new THREE.MeshStandardMaterial({ 
                color: 0xcccccc, 
                roughness: 0.5, 
                metalness: 0.8 
            });
            
            const bond1 = new THREE.Mesh(bondGeo, bondMat);
            bond1.position.set(0.05, 0.0375, 0);
            bond1.rotation.z = -0.7;
            group.add(bond1);
            
            const bond2 = new THREE.Mesh(bondGeo, bondMat);
            bond2.position.set(0.05, -0.0375, 0);
            bond2.rotation.z = 0.7;
            group.add(bond2);
            
            group.userData.type = 'water';
            return group;
        }

        function createBox() {
            const geometry = new THREE.BoxGeometry(0.1, 0.1, 0.1);
            const material = new THREE.MeshStandardMaterial({ 
                color: 0xff6600,
                roughness: 0.3,
                metalness: 0.5
            });
            const box = new THREE.Mesh(geometry, material);
            box.userData.type = 'box';
            return box;
        }

        function createSphere() {
            const geometry = new THREE.SphereGeometry(0.06, 32, 16);
            const material = new THREE.MeshStandardMaterial({ 
                color: 0x9900ff,
                roughness: 0.2,
                metalness: 0.7,
                emissive: 0x440088,
                emissiveIntensity: 0.2
            });
            const sphere = new THREE.Mesh(geometry, material);
            sphere.userData.type = 'sphere';
            return sphere;
        }

        function createObject(type) {
            switch(type) {
                case 'oxygen': return createOxygen();
                case 'hydrogen': return createHydrogen();
                case 'water': return createWater();
                case 'box': return createBox();
                case 'sphere': return createSphere();
                default: return createOxygen();
            }
        }

        // ==========================================
        // CONTROLES TÁCTILES
        // ==========================================
        
        let tapCount = 0;
        let tapTimer = null;
        
        renderer.domElement.addEventListener('touchstart', (e) => {
            e.preventDefault();
            touches = Array.from(e.touches);
            
            if (touches.length === 1) {
                const touch = touches[0];
                
                // Detectar doble tap para colocar objeto
                tapCount++;
                if (tapTimer) clearTimeout(tapTimer);
                
                if (tapCount === 2) {
                    tapCount = 0;
                    placeObject(touch.clientX, touch.clientY);
                } else {
                    tapTimer = setTimeout(() => { tapCount = 0; }, 300);
                }
                
                // Si hay objeto seleccionado, preparar rotación
                if (selectedObject) {
                    touchStartPos = { x: touch.clientX, y: touch.clientY };
                    initialRotation = {
                        x: selectedObject.rotation.x,
                        y: selectedObject.rotation.y,
                        z: selectedObject.rotation.z
                    };
                    manipulationMode = 'rotate';
                    showMode('🔄 Rotación');
                }
            } else if (touches.length === 2 && selectedObject) {
                const touch1 = touches[0];
                const touch2 = touches[1];
                
                initialDistance = Math.sqrt(
                    Math.pow(touch2.clientX - touch1.clientX, 2) +
                    Math.pow(touch2.clientY - touch1.clientY, 2)
                );
                
                initialScale = selectedObject.scale.x;
                manipulationMode = 'scale';
                showMode('📏 Escala');
            }
        }, { passive: false });

        renderer.domElement.addEventListener('touchmove', (e) => {
            e.preventDefault();
            touches = Array.from(e.touches);
            
            if (!selectedObject) return;
            
            if (touches.length === 1 && manipulationMode === 'rotate') {
                const touch = touches[0];
                const deltaX = touch.clientX - touchStartPos.x;
                const deltaY = touch.clientY - touchStartPos.y;
                
                const rotationSpeed = 0.008;
                selectedObject.rotation.y = initialRotation.y + deltaX * rotationSpeed;
                selectedObject.rotation.x = initialRotation.x - deltaY * rotationSpeed;
                
            } else if (touches.length === 2 && manipulationMode === 'scale') {
                const touch1 = touches[0];
                const touch2 = touches[1];
                
                const currentDistance = Math.sqrt(
                    Math.pow(touch2.clientX - touch1.clientX, 2) +
                    Math.pow(touch2.clientY - touch1.clientY, 2)
                );
                
                const scaleFactor = currentDistance / initialDistance;
                const newScale = Math.max(0.2, Math.min(4, initialScale * scaleFactor));
                
                selectedObject.scale.set(newScale, newScale, newScale);
                showMode(`📏 ${newScale.toFixed(1)}x`);
            }
        }, { passive: false });

        renderer.domElement.addEventListener('touchend', (e) => {
            e.preventDefault();
            touches = Array.from(e.touches);
            
            if (touches.length === 0) {
                if (manipulationMode !== 'none') {
                    manipulationMode = 'none';
                    hideMode();
                }
            }
        }, { passive: false });

        // ==========================================
        // COLOCAR OBJETOS
        // ==========================================
        
        function placeObject(screenX, screenY) {
            const obj = createObject(currentObjectType);
            
            // Convertir coordenadas de pantalla a 3D
            const x = ((screenX / window.innerWidth) * 2 - 1) * 0.5;
            const y = -((screenY / window.innerHeight) * 2 - 1) * 0.5;
            const z = -1.5;
            
            obj.position.set(x, y, z);
            scene.add(obj);
            objects.push(obj);
            
            // Seleccionar automáticamente
            selectedObject = obj;
            
            // Efecto de aparición
            obj.scale.set(0.1, 0.1, 0.1);
            animateScale(obj, 1.0, 300);
            
            objectCount++;
            updateCounter();
            updateInfo();
            
            showMode(`✓ ${getObjectName(currentObjectType)} colocado`);
            setTimeout(hideMode, 1500);
        }

        function animateScale(object, targetScale, duration) {
            const startScale = object.scale.x;
            const startTime = performance.now();
            
            function animate() {
                const elapsed = performance.now() - startTime;
                const progress = Math.min(elapsed / duration, 1);
                const eased = 1 - Math.pow(1 - progress, 3);
                
                const scale = startScale + (targetScale - startScale) * eased;
                object.scale.set(scale, scale, scale);
                
                if (progress < 1) {
                    requestAnimationFrame(animate);
                }
            }
            animate();
        }

        // ==========================================
        // UI FUNCTIONS
        // ==========================================
        
        window.selectObject = function(type) {
            currentObjectType = type;
            
            // Actualizar botones
            document.querySelectorAll('.control-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            event.target.classList.add('active');
            
            document.getElementById('current-object').textContent = getObjectName(type);
            document.getElementById('current-action').textContent = 'Doble tap para colocar';
        };

        function getObjectName(type) {
            const names = {
                'oxygen': 'Oxígeno (O)',
                'hydrogen': 'Hidrógeno (H₂)',
                'water': 'Agua (H₂O)',
                'box': 'Cubo',
                'sphere': 'Esfera'
            };
            return names[type] || type;
        }

        window.clearAll = function() {
            if (confirm('¿Eliminar todos los objetos?')) {
                objects.forEach(obj => scene.remove(obj));
                objects = [];
                selectedObject = null;
                objectCount = 0;
                updateCounter();
                showMode('🗑️ Todo eliminado');
                setTimeout(hideMode, 1500);
            }
        };

        window.toggleInfo = function() {
            if (infoPanel.style.display === 'none') {
                infoPanel.style.display = 'block';
            } else {
                infoPanel.style.display = 'none';
            }
        };

        function showMode(text) {
            modeIndicator.textContent = text;
            modeIndicator.style.display = 'block';
        }

        function hideMode() {
            modeIndicator.style.display = 'none';
        }

        function updateCounter() {
            document.getElementById('object-counter').textContent = `Objetos: ${objectCount}`;
        }

        function updateInfo() {
            document.getElementById('current-action').textContent = 
                selectedObject ? '1 dedo: rotar | 2 dedos: escalar' : 'Doble tap para colocar';
        }

        // ==========================================
        // ANIMACIÓN
        // ==========================================
        
        function animate() {
            requestAnimationFrame(animate);
            
            // Rotar objetos automáticamente
            objects.forEach(obj => {
                if (obj.userData.type !== 'box' && obj !== selectedObject) {
                    obj.rotation.y += 0.005;
                }
            });
            
            renderer.render(scene, camera);
        }

        // Prevenir gestos del navegador
        document.addEventListener('gesturestart', (e) => e.preventDefault());
        document.addEventListener('gesturechange', (e) => e.preventDefault());
        document.addEventListener('gestureend', (e) => e.preventDefault());
    </script>
</body>
</html>
