<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>astrocorebyamina| 3D Solar Explorer</title>
    <style>
        body { margin: 0; background: #000; color: white; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; overflow: hidden; }
        
        /* UI Layers */
        #gui { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 10; display: flex; flex-direction: column; justify-content: space-between; }
        
        header { padding: 20px; background: linear-gradient(to bottom, rgba(0,0,0,0.8), transparent); pointer-events: auto; }
        
        /* Dashboard - Sorted by Size */
        .dashboard { padding: 15px; background: rgba(0,0,0,0.6); backdrop-filter: blur(5px); display: flex; flex-wrap: wrap; gap: 8px; pointer-events: auto; }
        
        button { background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.3); color: white; padding: 10px 15px; cursor: pointer; border-radius: 4px; transition: 0.3s; font-size: 12px; }
        button:hover { background: #6366f1; border-color: #fff; }

        /* Planet Info Panel */
        .info-panel { position: absolute; right: 20px; top: 80px; width: 300px; background: rgba(10,10,10,0.85); border-left: 4px solid #6366f1; padding: 20px; border-radius: 0 10px 10px 0; pointer-events: auto; display: none; }
        
        /* Study Section */
        #study-section { position: absolute; bottom: 100px; right: 20px; width: 300px; background: rgba(20,20,30,0.9); padding: 15px; border-radius: 10px; font-size: 13px; pointer-events: auto; border: 1px solid #444; }
        
        .highlight { color: #00f2ff; font-weight: bold; }
        h2 { margin: 0 0 10px 0; font-size: 22px; text-transform: uppercase; letter-spacing: 2px; }
    </style>
</head>
<body>

<div id="gui">
    <header>
        <h1>ASTRO-DYNAMICS 360</h1>
    </header>

    <div class="info-panel" id="planetInfo">
        <h2 id="name">Earth</h2>
        <p id="description" style="font-style: italic; color: #ccc;"></p>
        <hr style="opacity:0.2">
        <p>📡 <b>Gravity:</b> <span class="highlight" id="gravity"></span> m/s²</p>
        <p>☀️ <b>Light from Sun:</b> <span class="highlight" id="lightTime"></span></p>
        <p>🧪 <b>Composition:</b> <span id="composition"></span></p>
    </div>

    <div id="study-section">
        <h3 style="color:#facc15; margin-top:0;">Earth-Sun-Moon Study</h3>
        <p>The <b style="color:#ffcc00">Sun</b> is a star providing 99.8% of the system's mass. The <b>Earth</b> orbits it at 30km/s. The <b>Moon</b> is Earth's only natural satellite, rotating synchronously so we always see the same side.</p>
    </div>

    <div class="dashboard" id="dash">
        </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<script>
    // 1. DATA - Sorted by Diameter (Biggest to Smallest)
    const planets = [
        { name: "Jupiter", color: 0xd39c7e, size: 2.4, gravity: 24.79, light: "43.2 min", comp: "Hydrogen, Helium", desc: "The King of Planets, a massive gas giant with a Great Red Spot." },
        { name: "Saturn", color: 0xc5ab6e, size: 2.0, gravity: 10.44, light: "1.3 hours", comp: "Hydrogen, Helium, Ice", desc: "Famous for its spectacular ring system made of ice and rock." },
        { name: "Uranus", color: 0xb5e3e3, size: 1.2, gravity: 8.69, light: "2.7 hours", comp: "Methane, Ice, Ammonia", desc: "An ice giant that rotates on its side (98-degree tilt)." },
        { name: "Neptune", color: 0x4b70dd, size: 1.1, gravity: 11.15, light: "4.1 hours", comp: "Ice, Hydrogen, Helium", desc: "The most distant planet, known for supersonic winds." },
        { name: "Earth", color: 0x2271b3, size: 0.8, gravity: 9.81, light: "8.3 min", comp: "Nitrogen, Oxygen, Silicate", desc: "Our home. The only planet known to harbor life and liquid water." },
        { name: "Venus", color: 0xe3bb76, size: 0.75, gravity: 8.87, light: "6 min", comp: "Carbon Dioxide, Sulfuric Acid", desc: "Earth's 'Evil Twin' with a runaway greenhouse effect." },
        { name: "Mars", color: 0xe27b58, size: 0.5, gravity: 3.71, light: "12.6 min", comp: "Iron Oxide, Basalt", desc: "The Red Planet, home to the largest volcano in the solar system." },
        { name: "Mercury", color: 0x8c8c8c, size: 0.3, gravity: 3.7, light: "3.2 min", comp: "Iron, Silicates", desc: "Smallest planet, closest to the Sun, with extreme temperatures." }
    ];

    // 2. 3D ENGINE
    let scene, camera, renderer, currentPlanet;

    function init() {
        scene = new THREE.Scene();
        camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.1, 2000);
        
        renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // Lighting (Cinematic)
        const sunLight = new THREE.PointLight(0xffffff, 1.5, 100);
        sunLight.position.set(10, 10, 10);
        scene.add(sunLight);
        scene.add(new THREE.AmbientLight(0x333333));

        // 3D Starfield
        const starGeo = new THREE.BufferGeometry();
        const starPos = [];
        for(let i=0; i<6000; i++) {
            starPos.push((Math.random()-0.5)*1000, (Math.random()-0.5)*1000, (Math.random()-0.5)*1000);
        }
        starGeo.setAttribute('position', new THREE.Float32BufferAttribute(starPos, 3));
        const starMat = new THREE.PointsMaterial({ color: 0xffffff, size: 0.7 });
        scene.add(new THREE.Points(starGeo, starMat));

        camera.position.z = 6;

        // Create Buttons
        const dash = document.getElementById('dash');
        planets.forEach(p => {
            const btn = document.createElement('button');
            btn.innerHTML = `<b>${p.name}</b>`;
            btn.onclick = () => selectPlanet(p);
            dash.appendChild(btn);
        });

        selectPlanet(planets[4]); // Start with Earth
    }

    function selectPlanet(p) {
        if(currentPlanet) scene.remove(currentPlanet);
        
        const geo = new THREE.SphereGeometry(p.size, 64, 64);
        const mat = new THREE.MeshStandardMaterial({ 
            color: p.color, 
            roughness: 0.8,
            metalness: 0.2
        });
        currentPlanet = new THREE.Mesh(geo, mat);
        scene.add(currentPlanet);

        // Update Panel
        const panel = document.getElementById('planetInfo');
        panel.style.display = 'block';
        document.getElementById('name').innerText = p.name;
        document.getElementById('description').innerText = p.desc;
        document.getElementById('gravity').innerText = p.gravity;
        document.getElementById('lightTime').innerText = p.light;
        document.getElementById('composition').innerText = p.comp;
    }

    function animate() {
        requestAnimationFrame(animate);
        if(currentPlanet) {
            currentPlanet.rotation.y += 0.005; // Auto-rotation
        }
        renderer.render(scene, camera);
    }

    window.onresize = () => {
        camera.aspect = window.innerWidth/window
