# 
  // Initialize the app
  document.addEventListener('DOMContentLoaded', () => {
    new App();
  }); const CACHE_NAME = 'tj-deck-designer-v1';
  const ASSETS = [
    '/',
    '/index.html',
    '/css/main.css',
    '/css/components.css',
    '/css/utilities.css',
    '/js/app.js',
    '/js/dashboard.js',
    '/js/project-manager.js',
    '/js/service-templates-manager.js',
    '/js/service-template-ui.js',
    '/js/estimator-manager.js',
    '/assets/icons/icon-72x72.png',
    '/assets/icons/icon-96x96.png',
    '/assets/icons/icon-128x128.png',
    '/assets/icons/icon-144x144.png',
    '/assets/icons/icon-152x152.png',
    '/assets/icons/icon-192x192.png',
    '/assets/icons/icon-384x384.png',
    '/assets/icons/icon-512x512.png'
  ];

  self.addEventListener('install', (event) => {
    event.waitUntil(
      caches.open(CACHE_NAME)
        .then((cache) => cache.addAll(ASSETS))
        .then(() => self.skipWaiting())
    );
  });

  self.addEventListener('activate', (event) => {
    event.waitUntil(
      caches.keys().then((cacheNames) => {
        return Promise.all(
          cacheNames.map((cacheName) => {
            if (cacheName !== CACHE_NAME) {
              return caches.delete(cacheName);
            }
          })
        );
      })
    );
  });

  self.addEventListener('fetch', (event) => {
    event.respondWith(
      caches.match(event.request)
        .then((response) => {
          return response || fetch(event.request);
        })
    );
  });  {
    "name": "TJ Deck Designer Pro",
    "short_name": "Deck Pro",
    "description": "Professional deck design and estimation tool",
    "start_url": "/",
    "display": "standalone",
    "background_color": "#F5F6FA",
    "theme_color": "#4A90E2",
    "icons": [
      {
        "src": "/assets/icons/icon-72x72.png",
        "sizes": "72x72",
        "type": "image/png"
      },
      {
        "src": "/assets/icons/icon-96x96.png",
        "sizes": "96x96",
        "type": "image/png"
      },
      {
        "src": "/assets/icons/icon-128x128.png",
        "sizes": "128x128",
        "type": "image/png"
      },
      {
        "src": "/assets/icons/icon-144x144.png",
        "sizes": "144x144",
        "type": "image/png"
      },
      {
        "src": "/assets/icons/icon-152x152.png",
        "sizes": "152x152",
        "type": "image/png"
      },
      {
        "src": "/assets/icons/icon-192x192.png",
        "sizes": "192x192",
        "type": "image/png"
      },
      {
        "src": "/assets/icons/icon-384x384.png",
        "sizes": "384x384",
        "type": "image/png"
      },
      {
        "src": "/assets/icons/icon-512x512.png",
        "sizes": "512x512",
        "type": "image/png"
      }
    ]
  }   #  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta name="theme-color" content="#4A90E2">
      <title>TJ Deck Designer Pro - 3D Edition</title>
      <link rel="manifest" href="manifest.json">
      <!-- Three.js for 3D rendering -->
      <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
      <!-- OrbitControls for 3D navigation -->
      <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
      <style>
          /* Previous CSS styles remain the same */
          
          #3dViewer {
              width: 100%;
              height: 500px;
              border-radius: 8px;
              overflow: hidden;
              position: relative;
          }
          
          .controls-overlay {
              position: absolute;
              top: 10px;
              right: 10px;
              z-index: 100;
              background: rgba(0,0,0,0.7);
              padding: 10px;
              border-radius: 4px;
              color: white;
          }
          
          .ar-view {
              position: fixed;
              top: 0;
              left: 0;
              width: 100%;
              height: 100%;
              background: rgba(0,0,0,0.9);
              z-index: 1000;
              display: none;
          }
          
          .ar-controls {
              position: fixed;
              bottom: 20px;
              left: 50%;
              transform: translateX(-50%);
              z-index: 1001;
              display: flex;
              gap: 10px;
          }
          
          .measurement-overlay {
              position: absolute;
              background: rgba(255,255,255,0.9);
              padding: 10px;
              border-radius: 4px;
              pointer-events: none;
          }
      </style>
  </head>
  <body>
      <!-- Previous HTML structure remains, adding new 3D and LiDAR sections -->
      
      <div class="card">
          <h2>3D Visualization</h2>
          <div id="3dViewer">
              <div class="controls-overlay">
                  <button onclick="toggleMeasurementMode()" class="btn">Toggle Measurement</button>
                  <button onclick="startARView()" class="btn">View in AR</button>
                  <button onclick="captureLiDAR()" class="btn">Scan Area</button>
              </div>
          </div>
      </div>
      
      <div class="ar-view" id="arView">
          <div class="ar-controls">
              <button onclick="closeARView()" class="btn">Exit AR</button>
              <button onclick="takeMeasurement()" class="btn">Measure</button>
              <button onclick="saveARData()" class="btn">Save Scan</button>
          </div>
      </div>

      <script>
          // Three.js setup
          let scene, camera, renderer, controls;
          let deckModel, measurementPoints = [];
          let isLiDARScanning = false;
          
          function initializeViewer() {
              scene = new THREE.Scene();
              camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
              
              renderer = new THREE.WebGLRenderer({ antialias: true });
              renderer.setSize(document.getElementById('3dViewer').clientWidth, 
                             document.getElementById('3dViewer').clientHeight);
              document.getElementById('3dViewer').appendChild(renderer.domElement);
              
              // Add OrbitControls
              controls = new THREE.OrbitControls(camera, renderer.domElement);
              controls.enableDamping = true;
              controls.dampingFactor = 0.05;
              
              // Initial camera position
              camera.position.set(5, 5, 5);
              controls.update();
              
              // Add lighting
              const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
              scene.add(ambientLight);
              
              const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
              directionalLight.position.set(10, 10, 10);
              scene.add(directionalLight);
              
              animate();
          }
          
          function animate() {
              requestAnimationFrame(animate);
              controls.update();
              renderer.render(scene, camera);
          }
          
          function updateDeckModel() {
              // Remove existing deck model if any
              if (deckModel) scene.remove(deckModel);
              
              const length = parseFloat(document.getElementById('length').value) || 10;
              const width = parseFloat(document.getElementById('width').value) || 10;
              const height = parseFloat(document.getElementById('height').value) || 3;
              
              // Create deck geometry
              const deckGeometry = new THREE.BoxGeometry(width, 0.2, length);
              const deckMaterial = new THREE.MeshPhongMaterial({ 
                  color: getDeckColor(document.getElementById('deckType').value)
              });
              
              deckModel = new THREE.Mesh(deckGeometry, deckMaterial);
              deckModel.position.y = height;
              scene.add(deckModel);
              
              // Add support posts
              addSupportPosts(length, width, height);
              
              // Add railings if selected
              if (document.getElementById('railingType').value !== 'none') {
                  addRailings(length, width, height);
              }
              
              // Add stairs if selected
              if (document.getElementById('stairs').value !== 'no') {
                  addStairs(height);
              }
          }
          
          function getDeckColor(deckType) {
              const colors = {
                  'pressure-treated': 0x8B7355,
                  'cedar': 0xD2691E,
                  'composite': 0x8B4513,
                  'pvc': 0xA0522D
              };
              return colors[deckType] || 0x8B7355;
          }
          
          function addSupportPosts(length, width, height) {
              const postGeometry = new THREE.BoxGeometry(0.4, height, 0.4);
              const postMaterial = new THREE.MeshPhongMaterial({ color: 0x4A4A4A });
              
              // Add posts at corners
              const corners = [
                  [-width/2, 0, -length/2],
                  [-width/2, 0, length/2],
                  [width/2, 0, -length/2],
                  [width/2, 0, length/2]
              ];
              
              corners.forEach(pos => {
                  const post = new THREE.Mesh(postGeometry, postMaterial);
                  post.position.set(pos[0], height/2, pos[2]);
                  scene.add(post);
              });
          }
          
          function addRailings(length, width, height) {
              const railingType = document.getElementById('railingType').value;
              const railingHeight = 3;
              const railingMaterial = new THREE.MeshPhongMaterial({
                  color: railingType === 'wood' ? 0x8B7355 : 0xC0C0C0,
                  opacity: railingType === 'glass' ? 0.5 : 1,
                  transparent: railingType === 'glass'
              });
              
              // Create railing geometry based on type
              if (railingType === 'glass') {
                  addGlassRailings(length, width, height, railingHeight);
              } else {
                  addTraditionalRailings(length, width, height, railingHeight, railingMaterial);
              }
          }
          
          function addStairs(deckHeight) {
              const stairType = document.getElementById('stairs').value;
              const stepHeight = 0.2;
              const stepDepth = 0.3;
              const numSteps = Math.ceil(deckHeight / stepHeight);
              
              const stepGeometry = new THREE.BoxGeometry(2, stepHeight, stepDepth);
              const stepMaterial = new THREE.MeshPhongMaterial({ 
                  color: getDeckColor(document.getElementById('deckType').value)
              });
              
              for (let i = 0; i < numSteps; i++) {
                  const step = new THREE.Mesh(stepGeometry, stepMaterial);
                  if (stairType === 'straight') {
                      step.position.set(0, i * stepHeight, -length/2 - (i * stepDepth));
                  } else {
                      // Curved stairs - calculate position on arc
                      const angle = (i / numSteps) * Math.PI / 2;
                      const radius = 2;
                      step.position.set(
                          radius * Math.cos(angle),
                          i * stepHeight,
                          -length/2 - radius * Math.sin(angle)
                      );
                      step.rotation.y = angle;
                  }
                  scene.add(step);
              }
          }
          
          // LiDAR and AR functionality
          async function captureLiDAR() {
              if (!window.XRSystem) {
                  showToast('LiDAR scanning requires compatible hardware', 'error');
                  return;
              }
              
              try {
                  isLiDARScanning = true;
                  const session = await navigator.xr.requestSession('immersive-ar', {
                      requiredFeatures: ['depth-sensing']
                  });
                  
                  session.addEventListener('select', handleLiDARData);
                  showToast('LiDAR scanning started. Move device to capture area.', 'success');
              } catch (error) {
                  showToast('Error starting LiDAR scan: ' + error.message, 'error');
              }
          }
          
          function handleLiDARData(event) {
              if (!isLiDARScanning) return;
              
              const frame = event.frame;
              const depthData = frame.getDepthInformation();
              
              if (depthData) {
                  // Process depth data and update 3D model
                  updateModelFromScan(depthData);
              }
          }
          
          function updateModelFromScan(depthData) {
              // Convert depth data to point cloud
              const pointCloud = new THREE.Points(
                  new THREE.BufferGeometry(),
                  new THREE.PointsMaterial({ size: 0.01, color: 0x00ff00 })
              );
              
              // Update geometry with scan data
              const positions = new Float32Array(depthData.width * depthData.height * 3);
              let index = 0;
              
              for (let y = 0; y < depthData.height; y++) {
                  for (let x = 0; x < depthData.width; x++) {
                      const depth = depthData.getDepth(x, y);
                      positions[index++] = x * 0.01;
                      positions[index++] = y * 0.01;
                      positions[index++] = depth;
                  }
              }
              
              pointCloud.geometry.setAttribute('position', 
                  new THREE.BufferAttribute(positions, 3));
              scene.add(pointCloud);
          }
          
          // Initialize everything when document loads
          document.addEventListener('DOMContentLoaded', () => {
              initializeViewer();
              updateDeckModel();
              
              // Add event listeners for dimension changes
              document.querySelectorAll('input, select').forEach(input => {
                  input.addEventListener('change', () => {
                      updateDeckModel();
                      calculateEstimate();
                  });
              });
          });
          
          // Handle window resize
          window.addEventListener('resize', () => {
              camera.aspect = window.innerWidth / window.innerHeight;
              camera.updateProjectionMatrix();
              renderer.setSize(document.getElementById('3dViewer').clientWidth,
                             document.getElementById('3dViewer').clientHeight);
          });
      </script>
  </body>
  </html>
          width: 100%;
          height: 500px;
          border-radius: 8px;
          overflow: hidden;
          position: relative;
      }
      
      .controls-overlay {
          position: absolute;
          top: 10px;
          right: 10px;
          z-index: 100;
          background: rgba(0,0,0,0.7);
          padding: 10px;
          border-radius: 4px;
          color: white;
      }
      
      .ar-view {
          position: fixed;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          background: rgba(0,0,0,0.9);
          z-index: 1000;
          display: none;
      }
      
      .ar-controls {
          position: fixed;
          bottom: 20px;
          left: 50%;
          transform: translateX(-50%);
          z-index: 1001;
          display: flex;
          gap: 10px;
      }
      
      .measurement-overlay {
          position: absolute;
          background: rgba(255,255,255,0.9);
          padding: 10px;
          border-radius: 4px;
          pointer-events: none;
      }
  </style>
  <div class="card">
      <h2>3D Visualization</h2>
      <div id="3dViewer">
          <div class="controls-overlay">
              <button onclick="toggleMeasurementMode()" class="btn">Toggle Measurement</button>
              <button onclick="startARView()" class="btn">View in AR</button>
              <button onclick="captureLiDAR()" class="btn">Scan Area</button>
          </div>
      </div>
  </div>
  
  <div class="ar-view" id="arView">
      <div class="ar-controls">
          <button onclick="closeARView()" class="btn">Exit AR</button>
          <button onclick="takeMeasurement()" class="btn">Measure</button>
          <button onclick="saveARData()" class="btn">Save Scan</button>
      </div>
  </div>

  <script>
      // Three.js setup
      let scene, camera, renderer, controls;
      let deckModel, measurementPoints = [];
      let isLiDARScanning = false;
      
      function initializeViewer() {
          scene = new THREE.Scene();
          camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
          
          renderer = new THREE.WebGLRenderer({ antialias: true });
          renderer.setSize(document.getElementById('3dViewer').clientWidth, 
                         document.getElementById('3dViewer').clientHeight);
          document.getElementById('3dViewer').appendChild(renderer.domElement);
          
          // Add OrbitControls
          controls = new THREE.OrbitControls(camera, renderer.domElement);
          controls.enableDamping = true;
          controls.dampingFactor = 0.05;
          
          // Initial camera position
          camera.position.set(5, 5, 5);
          controls.update();
          
          // Add lighting
          const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
          scene.add(ambientLight);
          
          const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
          directionalLight.position.set(10, 10, 10);
          scene.add(directionalLight);
          
          animate();
      }
      
      function animate() {
          requestAnimationFrame(animate);
          controls.update();
          renderer.render(scene, camera);
      }
      
      function updateDeckModel() {
          // Remove existing deck model if any
          if (deckModel) scene.remove(deckModel);
          
          const length = parseFloat(document.getElementById('length').value) || 10;
          const width = parseFloat(document.getElementById('width').value) || 10;
          const height = parseFloat(document.getElementById('height').value) || 3;
          
          // Create deck geometry
          const deckGeometry = new THREE.BoxGeometry(width, 0.2, length);
          const deckMaterial = new THREE.MeshPhongMaterial({ 
              color: getDeckColor(document.getElementById('deckType').value)
          });
          
          deckModel = new THREE.Mesh(deckGeometry, deckMaterial);
          deckModel.position.y = height;
          scene.add(deckModel);
          
          // Add support posts
          addSupportPosts(length, width, height);
          
          // Add railings if selected
          if (document.getElementById('railingType').value !== 'none') {
              addRailings(length, width, height);
          }
          
          // Add stairs if selected
          if (document.getElementById('stairs').value !== 'no') {
              addStairs(height);
          }
      }
      
      function getDeckColor(deckType) {
          const colors = {
              'pressure-treated': 0x8B7355,
              'cedar': 0xD2691E,
              'composite': 0x8B4513,
              'pvc': 0xA0522D
          };
          return colors[deckType] || 0x8B7355;
      }
      
      function addSupportPosts(length, width, height) {
          const postGeometry = new THREE.BoxGeometry(0.4, height, 0.4);
          const postMaterial = new THREE.MeshPhongMaterial({ color: 0x4A4A4A });
          
          // Add posts at corners
          const corners = [
              [-width/2, 0, -length/2],
              [-width/2, 0, length/2],
              [width/2, 0, -length/2],
              [width/2, 0, length/2]
          ];
          
          corners.forEach(pos => {
              const post = new THREE.Mesh(postGeometry, postMaterial);
              post.position.set(pos[0], height/2, pos[2]);
              scene.add(post);
          });
      }
      
      function addRailings(length, width, height) {
          const railingType = document.getElementById('railingType').value;
          const railingHeight = 3;
          const railingMaterial = new THREE.MeshPhongMaterial({
              color: railingType === 'wood' ? 0x8B7355 : 0xC0C0C0,
              opacity: railingType === 'glass' ? 0.5 : 1,
              transparent: railingType === 'glass'
          });
          
          // Create railing geometry based on type
          if (railingType === 'glass') {
              addGlassRailings(length, width, height, railingHeight);
          } else {
              addTraditionalRailings(length, width, height, railingHeight, railingMaterial);
          }
      }
      
      function addStairs(deckHeight) {
          const stairType = document.getElementById('stairs').value;
          const stepHeight = 0.2;
          const stepDepth = 0.3;
          const numSteps = Math.ceil(deckHeight / stepHeight);
          
          const stepGeometry = new THREE.BoxGeometry(2, stepHeight, stepDepth);
          const stepMaterial = new THREE.MeshPhongMaterial({ 
              color: getDeckColor(document.getElementById('deckType').value)
          });
          
          for (let i = 0; i < numSteps; i++) {
              const step = new THREE.Mesh(stepGeometry, stepMaterial);
              if (stairType === 'straight') {
                  step.position.set(0, i * stepHeight, -length/2 - (i * stepDepth));
              } else {
                  // Curved stairs - calculate position on arc
                  const angle = (i / numSteps) * Math.PI / 2;
                  const radius = 2;
                  step.position.set(
                      radius * Math.cos(angle),
                      i * stepHeight,
                      -length/2 - radius * Math.sin(angle)
                  );
                  step.rotation.y = angle;
              }
              scene.add(step);
          }
      }
      
      // LiDAR and AR functionality
      async function captureLiDAR() {
          if (!window.XRSystem) {
              showToast('LiDAR scanning requires compatible hardware', 'error');
              return;
          }
          
          try {
              isLiDARScanning = true;
              const session = await navigator.xr.requestSession('immersive-ar', {
                  requiredFeatures: ['depth-sensing']
              });
              
              session.addEventListener('select', handleLiDARData);
              showToast('LiDAR scanning started. Move device to capture area.', 'success');
          } catch (error) {
              showToast('Error starting LiDAR scan: ' + error.message, 'error');
          }
      }
      
      function handleLiDARData(event) {
          if (!isLiDARScanning) return;
          
          const frame = event.frame;
          const depthData = frame.getDepthInformation();
          
          if (depthData) {
              // Process depth data and update 3D model
              updateModelFromScan(depthData);
          }
      }
      
      function updateModelFromScan(depthData) {
          // Convert depth data to point cloud
          const pointCloud = new THREE.Points(
              new THREE.BufferGeometry(),
              new THREE.PointsMaterial({ size: 0.01, color: 0x00ff00 })
          );
          
          // Update geometry with scan data
          const positions = new Float32Array(depthData.width * depthData.height * 3);
          let index = 0;
          
          for (let y = 0; y < depthData.height; y++) {
              for (let x = 0; x < depthData.width; x++) {
                  const depth = depthData.getDepth(x, y);
                  positions[index++] = x * 0.01;
                  positions[index++] = y * 0.01;
                  positions[index++] = depth;
              }
          }
          
          pointCloud.geometry.setAttribute('position', 
              new THREE.BufferAttribute(positions, 3));
          scene.add(pointCloud);
      }
      
      // Initialize everything when document loads
      document.addEventListener('DOMContentLoaded', () => {
          initializeViewer();
          updateDeckModel();
          
          // Add event listeners for dimension changes
          document.querySelectorAll('input, select').forEach(input => {
              input.addEventListener('change', () => {
                  updateDeckModel();
                  calculateEstimate();
              });
          });
      });
      
      // Handle window resize
      window.addEventListener('resize', () => {
          camera.aspect = window.innerWidth / window.innerHeight;
          camera.updateProjectionMatrix();
          renderer.setSize(document.getElementById('3dViewer').clientWidth,
                         document.getElementById('3dViewer').clientHeight);
      });
  </script>
      * {
          box-sizing: border-box;
          margin: 0;
          padding: 0;
      }
      
      body {
          font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
          background-color: var(--background-color);
          color: var(--secondary-color);
          line-height: 1.6;
      }
      
      .container {
          max-width: 100%;
          padding: 1rem;
          margin: 0 auto;
      }
      
      .header {
          background-color: var(--primary-color);
          color: white;
          padding: 1rem;
          text-align: center;
          position: sticky;
          top: 0;
          z-index: 100;
      }
      
      .input-group {
          margin-bottom: 1.5rem;
          background: white;
          padding: 1rem;
          border-radius: 8px;
          box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      }
      
      label {
          display: block;
          margin-bottom: 0.5rem;
          font-weight: 600;
      }
      
      input, select {
          width: 100%;
          padding: 0.75rem;
          border: 1px solid #ddd;
          border-radius: 4px;
          font-size: 1rem;
          margin-bottom: 0.5rem;
      }
      
      .calculate-btn {
          background-color: var(--primary-color);
          color: white;
          padding: 1rem;
          border: none;
          border-radius: 4px;
          width: 100%;
          font-size: 1.1rem;
          font-weight: 600;
          cursor: pointer;
          margin-top: 1rem;
      }
      
      .results {
          margin-top: 2rem;
          background: white;
          padding: 1rem;
          border-radius: 8px;
          box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      }
      
      .result-item {
          display: flex;
          justify-content: space-between;
          padding: 0.5rem 0;
          border-bottom: 1px solid #eee;
      }
      
      .result-item:last-child {
          border-bottom: none;
          font-weight: 600;
      }
      
      @media (min-width: 768px) {
          .container {
              max-width: 768px;
          }
      }
  </style>
TJ Deck Designer  <div class="container">
      <div class="input-group">
          <label for="deckType">Deck Type</label>
          <select id="deckType">
              <option value="pressure-treated">Pressure Treated</option>
              <option value="cedar">Cedar</option>
              <option value="composite">Composite</option>
              <option value="pvc">PVC</option>
          </select>
          
          <label for="length">Length (ft)</label>
          <input type="number" id="length" min="0" step="0.1">
          
          <label for="width">Width (ft)</label>
          <input type="number" id="width" min="0" step="0.1">
          
          <label for="height">Height (ft)</label>
          <input type="number" id="height" min="0" step="0.1">
          
          <label for="railingType">Railing Type</label>
          <select id="railingType">
              <option value="wood">Wood</option>
              <option value="aluminum">Aluminum</option>
              <option value="cable">Cable</option>
              <option value="glass">Glass</option>
          </select>
          
          <button class="calculate-btn" onclick="calculateEstimate()">Calculate Estimate</button>
      </div>
      
      <div class="results" id="results">
          <div class="result-item">
              <span>Square Footage:</span>
              <span id="squareFootage">-</span>
          </div>
          <div class="result-item">
              <span>Material Cost:</span>
              <span id="materialCost">-</span>
          </div>
          <div class="result-item">
              <span>Labor Cost:</span>
              <span id="laborCost">-</span>
          </div>
          <div class="result-item">
              <span>Total Estimate:</span>
              <span id="totalEstimate">-</span>
          </div>
      </div>
  </div>

  <script>
      // Material costs per square foot
      const materialCosts = {
          'pressure-treated': 15,
          'cedar': 25,
          'composite': 35,
          'pvc': 40
      };
      
      // Railing costs per linear foot
      const railingCosts = {
          'wood': 30,
          'aluminum': 45,
          'cable': 60,
          'glass': 80
      };
      
      // Labor cost per square foot
      const laborCostPerSqFt = 20;
      
      function calculateEstimate() {
          const length = parseFloat(document.getElementById('length').value);
          const width = parseFloat(document.getElementById('width').value);
          const height = parseFloat(document.getElementById('height').value);
          const deckType = document.getElementById('deckType').value;
          const railingType = document.getElementById('railingType').value;
          
          if (!length || !width) {
              alert('Please enter valid dimensions');
              return;
          }
          
          const squareFootage = length * width;
          const perimeterLength = 2 * (length + width);
          
          const materialCost = squareFootage * materialCosts[deckType];
          const railingCost = perimeterLength * railingCosts[railingType];
          const laborCost = squareFootage * laborCostPerSqFt;
          
          const totalCost = materialCost + railingCost + laborCost;
          
          document.getElementById('squareFootage').textContent = 
              `${squareFootage.toFixed(1)} sq ft`;
          document.getElementById('materialCost').textContent = 
              `$${(materialCost + railingCost).toFixed(2)}`;
          document.getElementById('laborCost').textContent = 
              `$${laborCost.toFixed(2)}`;
          document.getElementById('totalEstimate').textContent = 
              `$${totalCost.toFixed(2)}`;
      }
      
      // Service Worker Registration
      if ('serviceWorker' in navigator) {
          window.addEventListener('load', () => {
              navigator.serviceWorker.register('service-worker.js')
                  .then(registration => {
                      console.log('ServiceWorker registration successful');
                  })
                  .catch(err => {
                      console.log('ServiceWorker registration failed: ', err);
                  });
          });
      }
  </script>
self.addEventListener('install', event => {
    event.waitUntil(
      caches.open(CACHE_NAME)
        .then(cache => cache.addAll(urlsToCache))
    );
  });self.addEventListener('fetch', event => {
    event.respondWith(
      caches.match(event.request)
        .then(response => response || fetch(event.request))
    );
  });   const CACHE_NAME = 'deck-designer-v1';
  const urlsToCache = [
    '/',
    '/index.html',
    '/manifest.json',
    '/icon-192.png',
    '/icon-512.png'
  ];self.addEventListener('install', event => {
    event.waitUntil(
      caches.open(CACHE_NAME)
        .then(cache => cache.addAll(urlsToCache))
    );
  });self.addEventListener('fetch', event => {
    event.respondWith(
      caches.match(event.request)
        .then(response => response || fetch(event.request))
    );
  });   import React from 'react'
  import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table"
  import { Card, CardContent } from "@/components/ui/card"const QuoteGeneratorDesign = () => {
    return (
      <div className="space-y-8">
        <Card>
          <CardContent className="p-6">
            <h2 className="text-xl font-bold mb-4">Quote Generator Sheet Layout</h2>
            <Table>
              <TableHeader>
                <TableRow>
                  <TableHead>Section</TableHead>
                  <TableHead>Location</TableHead>
                  <TableHead>Formulas/Links</TableHead>
                  <TableHead>Features</TableHead>
                </TableRow>
              </TableHeader>
              <TableBody>
                <TableRow>
                  <TableCell>Client Info (A1:D10)</TableCell>
                  <TableCell>Top Left</TableCell>
                  <TableCell>
                    <code>VLOOKUP(ClientID, ClientDB)</code>
                    <br />
                    Data validation for existing clients
                  </TableCell>
                  <TableCell>
                    <ul className="list-disc list-inside">
                      <li>Name</li>
                      <li>Address</li>
                      <li>Contact</li>
                      <li>Project Date</li>
                    </ul>
                  </TableCell>
                </TableRow>
                <TableRow>
                  <TableCell>Project Details (E1:H10)</TableCell>
                  <TableCell>Top Right</TableCell>
                  <TableCell>
                    <code>TODAY()</code>
                    <br />
                    <code>QUOTENUMBER()</code>
                    <br />
                    Auto-numbering system
                  </TableCell>
                  <TableCell>
                    <ul className="list-disc list-inside">
                      <li>Quote #</li>
                      <li>Date</li>
                      <li>Valid Until</li>
                      <li>Terms</li>
                    </ul>
                  </TableCell>
                </TableRow>
                <TableRow>
                  <TableCell>Item Selection (A11:H30)</TableCell>
                  <TableCell>Main Body</TableCell>
                  <TableCell>
                    <code>INDEX(Catalogs!...)</code>
                    <br />
                    <code>VLOOKUP(ItemCode, Catalogs)</code>
                  </TableCell>
                  <TableCell>
                    <ul className="list-disc list-inside">
                      <li>Dropdown menus</li>
                      <li>Auto-pricing</li>
                      <li>Quantity inputs</li>
                      <li>Line totals</li>
                    </ul>
                  </TableCell>
                </TableRow>
                <TableRow>
                  <TableCell>Calculations (I11:K30)</TableCell>
                  <TableCell>Right Side</TableCell>
                  <TableCell>
                    <code>SUM(LineItems)</code>
                    <br />
                    <code>SUMPRODUCT(Qty, Price)</code>
                  </TableCell>
                  <TableCell>
                    <ul className="list-disc list-inside">
                      <li>Subtotals</li>
                      <li>Tax</li>
                      <li>Total</li>
                      <li>Discounts</li>
                    </ul>
                  </TableCell>
                </TableRow>
              </TableBody>
            </Table>
          </CardContent>
        </Card>
      </div>
    )
  }export default QuoteGeneratorDesignimport React from 'react'
  import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table"
  import { Card, CardContent } from "@/components/ui/card"
  import { Input } from "@/components/ui/input"
  import { Label } from "@/components/ui/label"const QuoteGenerator = () => {
    const today = new Date().toLocaleDateString()
    const expiryDate = new Date()
    expiryDate.setDate(expiryDate.getDate() + 30)return (
  <div className="space-y-6 p-4">
    {/* Header */}
    <div className="text-2xl font-bold text-center">QUOTE</div>

    {/* Client and Quote Details Section */}
    <div className="grid md:grid-cols-2 gap-6">
      {/* Client Details */}
      <Card>
        <CardContent className="p-4">
          <h3 className="font-semibold mb-4">Client Details</h3>
          <div className="space-y-4">
            <div>
              <Label htmlFor="name">Name:</Label>
              <Input id="name" />
            </div>
            <div>
              <Label htmlFor="address">Address:</Label>
              <Input id="address" />
            </div>
            <div>
              <Label htmlFor="phone">Phone:</Label>
              <Input id="phone" />
            </div>
            <div>
              <Label htmlFor="email">Email:</Label>
              <Input id="email" type="email" />
            </div>
          </div>
        </CardContent>
      </Card>

      {/* Quote Details */}
      <Card>
        <CardContent className="p-4">
          <h3 className="font-semibold mb-4">Quote Details</h3>
          <div className="space-y-4">
            <div>
              <Label htmlFor="quoteNumber">Quote #:</Label>
              <Input id="quoteNumber" />
            </div>
            <div>
              <Label>Date:</Label>
              <Input value={today} readOnly />
            </div>
            <div>
              <Label>Valid Until:</Label>
              <Input value={expiryDate.toLocaleDateString()} readOnly />
            </div>
          </div>
        </CardContent>
      </Card>
    </div>

    {/* Line Items Table */}
    <Card>
      <CardContent className="p-4">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead className="w-[40%]">Item Description</TableHead>
              <TableHead>Quantity</TableHead>
              <TableHead>Unit Price</TableHead>
              <TableHead>Total</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {[...Array(5)].map((_, index) => (
              <TableRow key={index}>
                <TableCell>
                  <Input placeholder="Description" />
                </TableCell>
                <TableCell>
                  <Input type="number" min="0" placeholder="0" />
                </TableCell>
                <TableCell>
                  <Input type="number" min="0" placeholder="0.00" />
                </TableCell>
                <TableCell className="font-mono">$0.00</TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </CardContent>
    </Card>

    {/* Totals Section */}
    <Card>
      <CardContent className="p-4">
        <div className="space-y-2 ml-auto w-[200px]">
          <div className="flex justify-between">
            <span>Subtotal:</span>
            <span className="font-mono">$0.00</span>
          </div>
          <div className="flex justify-between">
            <span>Tax (10%):</span>
            <span className="font-mono">$0.00</span>
          </div>
          <div className="flex justify-between font-bold">
            <span>TOTAL:</span>
            <span className="font-mono">$0.00</span>
          </div>
        </div>
      </CardContent>
    </Card>
  </div>
)
}export default QuoteGenerator
  export default WorkbookStructureCatalogs Sheet Structure
  ' Column Layout:
  A: Item Code     ' Unique identifier for each product/service
  B: Description   ' Detailed item description
  C: Base Price    ' Standard price per unit
  D: Unit Type     ' Each, Square Foot, Linear Foot, etc.
  E: Category      ' Product/Service category
  F: Labor Hours   ' Estimated labor hours
  G: Material Cost ' Base material cost
  H: Markup %      ' Standard markup percentage' Named Ranges for Easy Reference:
  ' CatalogData = Catalogs!A2:H1000
  ' ItemCodes = Catalogs!A2:A1000
  ' PriceLookup = Catalogs!A2:C1000' Sample Data Format:
  ' A        B                C      D    E        F     G      H
  ' DK-001   Deck Base       45.00  SF   Decking  0.5   25.00  35%
  ' DK-002   Premium Deck    65.00  SF   Decking  0.75  40.00  40%
  ' FN-001   Basic Fence     35.00  LF   Fencing  0.3   20.00  30%  ' Quote Generator Formulas and Validations' Item Code Column (A11:A24)
  ' Data Validation:
  =INDIRECT("ItemCodes")  ' Drop-down list from Catalogs' Description (B11:B24)
  ' Formula:
  =IFERROR(VLOOKUP(A11,PriceLookup,2,FALSE),"Select an item")' Unit Price (C11:C24)
  ' Formula:
  =IFERROR(VLOOKUP(A11,PriceLookup,3,FALSE),0)' Quantity (B11:B24)
  ' Data Validation:
  Minimum: 0
  Maximum: 1000
  Error Message: "Please enter a valid quantity"' Line Total (D11:D24)
  ' Formula:
  =IF(AND(B11<>"",C11>0),B11*C11,0)' Subtotal (E25)
  ' Formula:
  =SUM(D11:D24)' Tax (E26)
  ' Formula:
  =E25*0.1' Total (E27)
  ' Formula:
  =E25+E26' Additional Validation Rules:
  ' 1. Prevent duplicate item codes in column A
  ' 2. Ensure quantity is entered if item code is selected
  ' 3. Format currency cells appropriately  ' Project Calculator Integration' Labor Cost Calculation
  ' Formula (hidden helper column):
  =IFERROR(VLOOKUP(A11,CatalogData,6,FALSE)*LaborRate,0)' Material Cost Calculation
  ' Formula (hidden helper column):
  =IFERROR(VLOOKUP(A11,CatalogData,7,FALSE)*B11,0)' Markup Calculation
  ' Formula (hidden helper column):
  =IFERROR(VLOOKUP(A11,CatalogData,8,FALSE),0)' Total Project Cost
  ' Formula:
  =SUM(MaterialCost) + SUM(LaborCost) * (1 + ProjectMarkup)  '   ' Master_Catalog Structure
  ' Columns A-C: Service codes and categories
  A: Service_Code    ' Format: CAT-###  (PER-001, FEN-001, etc.)
  B: Category        ' Main category (Pergolas, Fencing, etc.)
  C: Description     ' Detailed service description' Columns D-F: Base pricing and units
  D: Base_Price      ' Standard price point
  E: Unit_Type       ' SF, LF, EA, etc.
  F: Min_Quantity    ' Minimum order quantity' Columns G-I: Modification factors
  G: Complexity_Factor  ' 1 = Standard, 1.5 = Custom, 2 = Premium
  H: Season_Modifier    ' Seasonal pricing adjustments
  I: Volume_Discount    ' Quantity discount thresholds' Columns J-L: Related materials
  J: Material_Codes     ' Reference to Material_Library
  K: Labor_Category     ' Reference to Labor_Rates
  L: Notes             ' Special considerations' Named Ranges:
  ' MasterCatalogData = Master_Catalog!A2:L1000
  ' ServiceCodes = Master_Catalog!A2:A1000
  ' CategoryList = Master_Catalog!B2:B1000' Key Formulas for Category Links:
  ' In Category sheets (e.g., Pergolas_Catalog):
  =FILTER(MasterCatalogData, MasterCatalogData[Category]="Pergolas")' Price Calculation with all factors:
  =[@Base_Price] *
   [@Complexity_Factor] *
   [@Season_Modifier] *
   IF([@Quantity]>[@Volume_Discount_Threshold],
      (1-[@Volume_Discount_Rate]),
      1)  ' Service Code Validation
  ' Format: CAT-### (Where CAT is 3 letters, ### is 3 numbers)
  Custom Formula: =AND(
    LEN(A2)=7,
    LEFT(A2,3)=UPPER(LEFT(A2,3)),
    MID(A2,4,1)="-",
    ISNUMBER(--RIGHT(A2,3))
  )' Category Validation
  List Source: =CategoryList
  Error Message: "Select a valid category from the list"' Base Price Validation
  Decimal > 0
  Error Message: "Price must be greater than zero"' Unit Type Validation
  List: "SF,LF,EA,HR,SET"
  Error Message: "Select a valid unit type"' Complexity Factor Validation
  List: "1,1.5,2"
  Error Message: "Select standard (1), custom (1.5), or premium (2)"' Season Modifier Validation
  Decimal between 0.8 and 1.2
  Error Message: "Seasonal adjustment must be between -20% and +20%"' Material Codes Validation
  List Source: =Material_Library!A2:A1000
  Error Message: "Select valid material code from library"Material Cost Lookup (in Category sheets)
  =XLOOKUP(
    [@Material_Codes],
    Material_Library[Code],
    Material_Library[Current_Cost],
    0,
    0
  )' Labor Rate Calculation
  =XLOOKUP(
    [@Labor_Category],
    Labor_Rates[Category],
    Labor_Rates[Hourly_Rate],
    0,
    0
  ) * [@Complexity_Factor]' Total Cost Formula
  =[@Material_Cost] +
   ([@Labor_Hours] * [@Labor_Rate]) *
   (1 + [@Overhead_Rate])' Project Calculator Reference
  =SUMIFS(
    MasterCatalogData[Base_Price],
    MasterCatalogData[Service_Code], [@Selected_Services],
    MasterCatalogData[Category], [@Selected_Category]
  )' Beam Properties Table (Name: BeamProperties)
  ' Format: Size, Weight/ft, Moment of Inertia, Max Span, Cost/ft
  W8x10,    10,   30.8,   20,   45.00
  W8x13,    13,   39.6,   22,   52.00
  W8x15,    15,   48.0,   24,   58.00
  W10x12,   12,   53.8,   25,   54.00
  W10x15,   15,   68.9,   27,   62.00
  W12x14,   14,   88.6,   30,   60.00
  W12x16,   16,   103.0,  32,   65.00' Joist Properties Table (Name: JoistProperties)
  ' Format: Size, Max Span(ft), Load Capacity(psf), Cost/ft
  2x6,      9,    40,    4.50
  2x8,      12,   50,    6.25
  2x10,     14,   60,    8.75
  2x12,     16,   70,    12.00
  2x14,     18,   80,    14.50
  2x16,     20,   90,    16.75' Deck Material Properties (Name: DeckProperties)
  ' Format: Material, Cost/sqft, Load Rating, Installation Factor
  Plywood 1/2,     2.50,  40,  1.0
  Plywood 5/8,     3.25,  50,  1.0
  Plywood 3/4,     3.75,  60,  1.1
  OSB 7/16,        2.00,  35,  0.9
  OSB 23/32,       2.75,  45,  0.9
  Tongue & Groove 3/4, 4.25, 65, 1.2' Calculation Formulas for Estimates Sheet' Material Cost Calculation
  =SUMPRODUCT(
    VLOOKUP(B11, BeamProperties, 5, FALSE) * BeamLength,
    VLOOKUP(B12, JoistProperties, 4, FALSE) * NumberOfJoists,
    VLOOKUP(B13, DeckProperties, 2, FALSE) * FloorArea
  )' Load Capacity Verification
  =IF(AND(
    VLOOKUP(B11, BeamProperties, 4, FALSE) >= SpanLength,
    VLOOKUP(B12, JoistProperties, 2, FALSE) >= JoistSpan,
    VLOOKUP(B13, DeckProperties, 3, FALSE) >= RequiredLoadRating
  ), "Design Valid", "Review Required")' Installation Time Estimate
  =SUM(
    BaseInstallHours,
    FloorArea * VLOOKUP(B13, DeckProperties, 4, FALSE),
    NumberOfJoists * JoistInstallFactor,
    BeamLength * BeamInstallFactor
  )' Total Project Cost Formula
  =SUM(
    MaterialCost,
    InstallationHours * LaborRate,
    OverheadFactor,
    ContingencyAmount
  ) <!DOCTYPE html>      body {
          background: #f0f2f5;
          padding: 20px;
      }

      .container {
          max-width: 800px;
          margin: 0 auto;
          background: white;
          padding: 20px;
          border-radius: 10px;
          box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      }

      h1 {
          color: #1a73e8;
          margin-bottom: 20px;
          text-align: center;
      }

      .input-group {
          display: flex;
          gap: 10px;
          margin-bottom: 20px;
      }

      input[type="text"] {
          flex: 1;
          padding: 10px;
          border: 1px solid #ddd;
          border-radius: 5px;
          font-size: 16px;
      }

      button {
          padding: 10px 20px;
          background: #1a73e8;
          color: white;
          border: none;
          border-radius: 5px;
          cursor: pointer;
          transition: background 0.3s;
      }

      button:hover {
          background: #1557b0;
      }

      .task {
          display: flex;
          align-items: center;
          padding: 15px;
          background: #f8f9fa;
          margin-bottom: 10px;
          border-radius: 5px;
          gap: 10px;
      }

      .task.completed {
          background: #e8f5e9;
          text-decoration: line-through;
          color: #666;
      }

      .task input[type="checkbox"] {
          width: 20px;
          height: 20px;
      }

      .task-text {
          flex: 1;
      }

      .delete-btn {
          background: #dc3545;
          padding: 5px 10px;
          font-size: 14px;
      }

      .delete-btn:hover {
          background: #c82333;
      }

      .filters {
          display: flex;
          gap: 10px;
          margin-bottom: 20px;
          justify-content: center;
      }

      .filter-btn {
          background: #f8f9fa;
          color: #1a73e8;
          border: 1px solid #1a73e8;
      }

      .filter-btn.active {
          background: #1a73e8;
          color: white;
      }

      @media (max-width: 600px) {
          .container {
              padding: 10px;
          }

          .input-group {
              flex-direction: column;
          }

          .filters {
              flex-wrap: wrap;
          }
      }
  </style>
Task Manager      <div class="input-group">
          <input type="text" id="taskInput" placeholder="Enter a new task...">
          <button onclick="addTask()">Add Task</button>
      </div>

      <div class="filters">
          <button class="filter-btn active" onclick="filterTasks('all')">All</button>
          <button class="filter-btn" onclick="filterTasks('active')">Active</button>
          <button class="filter-btn" onclick="filterTasks('completed')">Completed</button>
      </div>

      <div id="taskList"></div>
  </div>

  <script>
      let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
      let currentFilter = 'all';

      function addTask() {
          const input = document.getElementById('taskInput');
          const text = input.value.trim();
          
          if (text) {
              const task = {
                  id: Date.now(),
                  text: text,
                  completed: false
              };
              
              tasks.push(task);
              saveTasks();
              input.value = '';
              renderTasks();
          }
      }

      function toggleTask(id) {
          tasks = tasks.map(task => 
              task.id === id ? {...task, completed: !task.completed} : task
          );
          saveTasks();
          renderTasks();
      }

      function deleteTask(id) {
          tasks = tasks.filter(task => task.id !== id);
          saveTasks();
          renderTasks();
      }

      function filterTasks(filter) {
          currentFilter = filter;
          document.querySelectorAll('.filter-btn').forEach(btn => {
              btn.classList.remove('active');
          });
          event.target.classList.add('active');
          renderTasks();
      }

      function renderTasks() {
          const taskList = document.getElementById('taskList');
          let filteredTasks = tasks;
          
          if (currentFilter === 'active') {
              filteredTasks = tasks.filter(task => !task.completed);
          } else if (currentFilter === 'completed') {
              filteredTasks = tasks.filter(task => task.completed);
          }

          taskList.innerHTML = filteredTasks.map(task => `
              <div class="task ${task.completed ? 'completed' : ''}">
                  <input type="checkbox" 
                      ${task.completed ? 'checked' : ''} 
                      onchange="toggleTask(${task.id})">
                  <span class="task-text">${task.text}</span>
                  <button class="delete-btn" onclick="deleteTask(${task.id})">Delete</button>
              </div>
          `).join('');
      }

      function saveTasks() {
          localStorage.setItem('tasks', JSON.stringify(tasks));
      }

      // Add task when Enter key is pressed
      document.getElementById('taskInput').addEventListener('keypress', function(e) {
          if (e.key === 'Enter') {
              addTask();
          }
      });

      // Initial render
      renderTasks();
  </script>
  // Labor rates
  laborRates: {
      skilled: 45,    // per hour
      unskilled: 25,  // per hour
      supervisor: 65  // per hour
  },

  // Calculate material cost with wastage
  calculateMaterialCost: function(material, quantity) {
      const materialData = this.materials[material];
      if (!materialData) return 0;

      const baseQuantity = quantity;
      const wastageQuantity = quantity * (materialData.wastagePercent / 100);
      const totalQuantity = baseQuantity + wastageQuantity;

      return {
          materialCost: totalQuantity * materialData.basePrice,
          laborCost: totalQuantity * materialData.laborCost,
          wastageAmount: wastageQuantity,
          total: totalQuantity * (materialData.basePrice + materialData.laborCost)
      };
  },

  // Calculate labor hours based on project complexity
  calculateLaborHours: function(complexity, area) {
      const laborFactors = {
          low: 0.1,
          medium: 0.2,
          high: 0.3
      };
      return area * laborFactors[complexity];
  },

  // Generate detailed estimate
  generateDetailedEstimate: function(projectData) {
      const estimate = {
          materials: {},
          labor: {},
          total: 0
      };

      // Calculate material costs
      Object.keys(projectData.materials).forEach(material => {
          estimate.materials[material] = this.calculateMaterialCost(
              material, 
              projectData.materials[material]
          );
      });

      // Calculate labor costs
      const laborHours = this.calculateLaborHours(
          projectData.complexity, 
          projectData.area
      );

      estimate.labor = {
          skilled: laborHours * this.laborRates.skilled,
          unskilled: laborHours * this.laborRates.unskilled,
          supervisor: (laborHours * 0.2) * this.laborRates.supervisor
      };

      // Calculate total
      estimate.total = Object.values(estimate.materials)
          .reduce((sum, mat) => sum + mat.total, 0) +
          Object.values(estimate.labor)
          .reduce((sum, labor) => sum + labor, 0);

      return estimate;
  }
};  <!DOCTYPE html>          <!-- Estimation Panel -->
          <div class="col-md-4 estimation-panel">
              <h3>Project Estimation</h3>
              <div id="materialsList">
                  <!-- Materials will be populated here -->
              </div>
              <div class="mt-4">
                  <h4>Total Estimate</h4>
                  <div id="totalEstimate" class="h3">$0.00</div>
              </div>
          </div>
      </div>
  </div>

  <script>
      let viewer;
      let materials = [];
      
      // Initialize SketchUp Viewer
      window.onload = async function() {
          try {
              viewer = await sketchup.Viewer.create({
                  container: 'viewer',
                  api_token: 'YOUR_SKETCHUP_API_TOKEN'
              });

              // Load default model
              await viewer.loadModel('MODEL_URL');
              
              // Initialize materials tracking
              initializeMaterialsTracking();
          } catch (error) {
              console.error('Error initializing viewer:', error);
          }
      };

      function initializeMaterialsTracking() {
          // Sample materials data
          materials = [
              { id: 1, name: 'Concrete', price: 120, unit: 'cubic yard', quantity: 0 },
              { id: 2, name: 'Steel Beams', price: 800, unit: 'ton', quantity: 0 },
              { id: 3, name: 'Glass Panels', price: 40, unit: 'sqft', quantity: 0 },
              { id: 4, name: 'Wood Framing', price: 3.50, unit: 'linear ft', quantity: 0 }
          ];
          updateMaterialsList();
      }

      function updateMaterialsList() {
          const materialsContainer = document.getElementById('materialsList');
          materialsContainer.innerHTML = materials.map(material => `
              <div class="material-item">
                  <div class="d-flex justify-content-between">
                      <strong>${material.name}</strong>
                      <span>$${material.price}/${material.unit}</span>
                  </div>
                  <div class="input-group mt-2">
                      <input type="number" class="form-control" 
                             value="${material.quantity}"
                             onchange="updateQuantity(${material.id}, this.value)">
                      <span class="input-group-text">${material.unit}</span>
                  </div>
              </div>
          `).join('');
          calculateTotal();
      }

      function updateQuantity(materialId, quantity) {
          const material = materials.find(m => m.id === materialId);
          if (material) {
              material.quantity = parseFloat(quantity) || 0;
              calculateTotal();
          }
      }

      function calculateTotal() {
          const total = materials.reduce((sum, material) => 
              sum + (material.price * material.quantity), 0);
          document.getElementById('totalEstimate').innerText = 
              `$${total.toFixed(2)}`;
      }

      function generateEstimate() {
          // Get model measurements
          viewer.getMeasurements().then(measurements => {
              // Update quantities based on measurements
              // This is a simplified example
              materials.forEach(material => {
                  if (material.name === 'Concrete') {
                      material.quantity = calculateConcreteVolume(measurements);
                  } else if (material.name === 'Glass Panels') {
                      material.quantity = calculateGlassArea(measurements);
                  }
                  // Add more material calculations as needed
              });
              updateMaterialsList();
          });
      }

      function calculateConcreteVolume(measurements) {
          // Simplified calculation - replace with actual logic
          return Math.round(measurements.volume * 0.037); // Convert to cubic yards
      }

      function calculateGlassArea(measurements) {
          // Simplified calculation - replace with actual logic
          return Math.round(measurements.area * 10.764); // Convert to square feet
      }

      function takeScreenshot() {
          viewer.getScreenshot().then(imageData => {
              // Handle screenshot data
              const link = document.createElement('a');
              link.download = 'design-screenshot.png';
              link.href = imageData;
              link.click();
          });
      }

      function measureDistance() {
          viewer.startMeasurement();
      }
  </script>
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
Create BIM component extractordef extract_deck_components(image_data):
    components = {
        'structural': [],
        'finishing': [],
        'hardware': [],
        'dimensions': {}
    }
    return components
  import numpy as np
  import cv2
  from dataclasses import dataclass@dataclass
  class DeckComponent:
      type: str
      dimensions: dict
      material: str
      connections: list
      texture: dictclass DeckComponentExtractor:
      def init(self):
          self.components = []
          self.textures = {}  def extract_from_image(self, image):
      # Image processing and component detection
      components = []
      return components
      
  def analyze_texture(self, component_image):
      # Texture analysis
      texture_data = {}
      return texture_data
      
  def identify_connections(self, component):
      # Connection point detection
      connections = []
      return connections
      
  def get_dimensions(self, component):
      # Dimension extraction
      dimensions = {}
      return dimensions import React from 'react';
import { Card } from "@/components/ui/card";
  import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";const PartsLibrary = () => {
    return (
      <Tabs defaultValue="structural" className="w-full">
        <TabsList>
          <TabsTrigger value="structural">Structural</TabsTrigger>
          <TabsTrigger value="hardware">Hardware</TabsTrigger>
          <TabsTrigger value="finishing">Finishing</TabsTrigger>
        </TabsList>    <TabsContent value="structural">
      <Card className="grid grid-cols-3 gap-4 p-4">
        {/* Structural Components */}
        <div className="component-card">
          <h4>Posts</h4>
          {/* Component details */}
        </div>
        <div className="component-card">
          <h4>Beams</h4>
          {/* Component details */}
        </div>
        <div className="component-card">
          <h4>Joists</h4>
          {/* Component details */}
        </div>
      </Card>
    </TabsContent>

    <TabsContent value="hardware">
      <Card className="grid grid-cols-3 gap-4 p-4">
        {/* Hardware Components */}
      </Card>
    </TabsContent>

    <TabsContent value="finishing">
      <Card className="grid grid-cols-3 gap-4 p-4">
        {/* Finishing Components */}
      </Card>
    </TabsContent>
  </Tabs>
);
};export default PartsLibrary;    // Excel BIM Integration System
  const ExcelBIMSystem = {
    // Component Registry
    components: {
      structural: {
        posts: [],
        beams: [],
        joists: [],
        blocking: []
      },
      hardware: {
        connectors: [],
        fasteners: [],
        brackets: []
      },
      finishing: {
        decking: [],
        railings: [],
        stairs: []
      }
    },// Texture Management
textureLibrary: {
  wood: {
    pressure_treated: {},
    cedar: {},
    composite: {}
  },
  metal: {
    galvanized: {},
    powder_coated: {}
  }
},

// Component Extraction
extractComponents: function(imageData) {
  return {
    type: '',
    dimensions: {},
    material: '',
    connections: []
  };
},

// Real-time Rendering
renderComponent: function(component) {
  // Component rendering logic
  return {
    geometry: {},
    texture: {},
    position: {}
  };
},

// BIM Cell Updates
updateBIMCell: function(cell, componentData) {
  return {
    value: componentData,
    formatting: {},
    calculations: []
  };
}
};  import React from 'react';
  import { Card } from "@/components/ui/card";
  import { Select } from "@/components/ui/select";const DeckDesignSystem = () => {
    return (
      <div className="grid gap-4">
        <Card className="p-4">
          <h3 className="text-lg font-bold mb-2">Component Library</h3>
          <div className="grid grid-cols-3 gap-2">
            <Select>
              <option value="structural">Structural Components</option>
              <option value="hardware">Hardware & Fasteners</option>
              <option value="finishing">Finishing Materials</option>
            </Select>
          </div>
        </Card>    <Card className="p-4">
      <h3 className="text-lg font-bold mb-2">Design Parameters</h3>
      <div className="grid grid-cols-2 gap-4">
        <div>
          <label>Deck Dimensions</label>
          <input type="text" className="w-full border rounded p-2" />
        </div>
        <div>
          <label>Height</label>
          <input type="text" className="w-full border rounded p-2" />
        </div>
      </div>
    </Card>
  </div>
);
};export default DeckDesignSystem;{
    "name": "TJ Deck Designer",
    "short_name": "Deck Designer",
    "start_url": "index.html",
    "display": "standalone",
    "background_color": "#F5F6FA",
    "theme_color": "#4A90E2",
    "icons": [
      {
        "src": "icon-192.png",
        "sizes": "192x192",
        "type": "image/png"
      },
      {
        "src": "icon-512.png",
        "sizes": "512x512",
        "type": "image/png"
      }
    ]
  }const EstimateResults = () => {
  return (
    <div className="min-h-screen bg-gray-50 p-4">
      <header className="mb-6">
        <div className="flex justify-between items-start">
          <div>
            <h1 className="text-2xl font-bold text-gray-900">Project Estimate</h1>
            <p className="text-gray-500">Custom Deck Construction</p>
          </div>
          <div className="text-right">
            <p className="text-2xl font-bold text-blue-600">$15,750</p>
            <p className="text-sm text-gray-500">Total Estimate</p>
          </div>
        </div>
      </header>  {/* Quick Summary */}
  <div className="grid grid-cols-3 gap-4 mb-6">
    <Card className="p-4">
      <Clock className="w-5 h-5 text-blue-500 mb-2" />
      <p className="text-sm text-gray-500">Timeline</p>
      <p className="font-semibold">2-3 weeks</p>
    </Card>
    <Card className="p-4">
      <Tool className="w-5 h-5 text-blue-500 mb-2" />
      <p className="text-sm text-gray-500">Labor Hours</p>
      <p className="font-semibold">80-100 hrs</p>
    </Card>
    <Card className="p-4">
      <Calendar className="w-5 h-5 text-blue-500 mb-2" />
      <p className="text-sm text-gray-500">Start Date</p>
      <p className="font-semibold">Apr 1, 2024</p>
    </Card>
  </div>

  {/* Detailed Breakdown */}
  <Card className="mb-6">
    <div className="p-4 border-b">
      <h2 className="text-lg font-semibold">Cost Breakdown</h2>
    </div>
    <div className="divide-y">
      <div className="p-4">
        <h3 className="font-medium mb-4">Materials</h3>
        <div className="space-y-3">
          <div className="flex justify-between text-sm">
            <span>Composite Decking</span>
            <span className="font-medium">$6,000</span>
          </div>
          <div className="flex justify-between text-sm">
            <span>Framing Lumber</span>
            <span className="font-medium">$2,200</span>
          </div>
          <div className="flex justify-between text-sm">
            <span>Hardware & Fasteners</span>
            <span className="font-medium">$800</span>
          </div>
          <div className="flex justify-between text-sm">
            <span>Railings</span>
            <span className="font-medium">$1,500</span>
          </div>
        </div>
      </div>

      <div className="p-4">
        <h3 className="font-medium mb-4">Labor</h3>
        <div className="space-y-3">
          <div className="flex justify-between text-sm">
            <span>Construction Labor</span>
            <span className="font-medium">$4,000</span>
          </div>
          <div className="flex justify-between text-sm">
            <span>Design & Planning</span>
            <span className="font-medium">$750</span>
          </div>
          <div className="flex justify-between text-sm">
            <span>Permit Filing</span>
            <span className="font-medium">$500</span>
          </div>
        </div>
      </div>

      <div className="p-4">
        <h3 className="font-medium mb-4">Project Specifications</h3>
        <div className="grid grid-cols-2 gap-4 text-sm">
          <div>
            <p className="text-gray-500">Dimensions</p>
            <p className="font-medium">20' x 16'</p>
          </div>
          <div>
            <p className="text-gray-500">Height</p>
            <p className="font-medium">3 feet</p>
          </div>
          <div>
            <p className="text-gray-500">Material</p>
            <p className="font-medium">Composite Decking</p>
          </div>
          <div>
            <p className="text-gray-500">Features</p>
            <p className="font-medium">Railings, Stairs</p>
          </div>
        </div>
      </div>
    </div>
  </Card>

  {/* Notes */}
  <Card className="mb-20 p-4">
    <h2 className="font-semibold mb-2">Additional Notes</h2>
    <ul className="text-sm space-y-2 text-gray-600">
      <li>• Estimate includes all necessary permits and inspections</li>
      <li>• Timeline assumes normal weather conditions</li>
      <li>• 5-year warranty on workmanship</li>
      <li>• Manufacturer warranty on materials</li>
    </ul>
  </Card>

  {
<!<!<!DOCTYPE html>Welcome to TJ SuiteYour Personal Management Solution    <div class="feature-card">
        <h2>Quick Actions</h2>
        <button class="button" onclick="alert('Feature coming soon!')">Start Task</button>
        <button class="button" onclick="alert('Feature coming soon!')">View Schedule</button>
    </div>

    <div class="feature-card">
        <h2>Recent Activities</h2>
        <div id="activities">
            <p>No recent activities</p>
        </div>
    </div>

    <div class="feature-card">
        <h2>Tools</h2>
        <button class="button" onclick="alert('Calendar feature coming soon!')">Calendar</button>
        <button class="button" onclick="alert('Tasks feature coming soon!')">Tasks</button>
        <button class="button" onclick="alert('Notes feature coming soon!')">Notes</button>
    </div>
</div>

<script>
    // Basic interaction
    document.querySelectorAll('.feature-card').forEach(card => {
        card.addEventListener('click', () => {
            card.style.backgroundColor = '#f8f9fa';
            setTimeout(() => {
                card.style.backgroundColor = 'white';
            }, 200);
        });
    });
</script>
T&J LLC Business Suite    <main id="main-content">
        <!-- Content will be dynamically loaded -->
    </main>

    <div id="loading" style="display: none;">
        <div class="spinner"></div>
    </div>
</div>
<script src="app.js"></script>
      * {
          box-sizing: border-box;
          margin: 0;
          padding: 0;
          font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      }
      
      body {
          background-color: var(--background-color);
          color: var(--secondary-color);
      }
      
      .app-container {
          display: grid;
          grid-template-columns: 300px 1fr;
          min-height: 100vh;
      }
      
      .sidebar {
          background: white;
          padding: 1.5rem;
          border-right: 1px solid #ddd;
          overflow-y: auto;
      }
      
      .main-content {
          padding: 1.5rem;
          overflow-y: auto;
      }
      
      .card {
          background: white;
          border-radius: 8px;
          box-shadow: 0 2px 4px rgba(0,0,0,0.1);
          padding: 1.5rem;
          margin-bottom: 1.5rem;
      }
      
      .viewer-container {
          width: 100%;
          height: 600px;
          border-radius: 8px;
          overflow: hidden;
          position: relative;
      }
      
      .controls-overlay {
          position: absolute;
          top: 1rem;
          right: 1rem;
          z-index: 100;
          display: flex;
          gap: 0.5rem;
      }
      
      .btn {
          background: var(--primary-color);
          color: white;
          border: none;
          padding: 0.75rem 1.5rem;
          border-radius: 4px;
          cursor: pointer;
          font-weight: 600;
          transition: background 0.3s;
      }
      
      .btn:hover {
          background: #357ABD;
      }
      
      .btn.secondary {
          background: var(--secondary-color);
      }
      
      .input-group {
          margin-bottom: 1rem;
      }
      
      label {
          display: block;
          margin-bottom: 0.5rem;
          font-weight: 600;
      }
      
      input, select {
          width: 100%;
          padding: 0.75rem;
          border: 1px solid #ddd;
          border-radius: 4px;
          font-size: 1rem;
      }
      
      .results {
          display: grid;
          grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
          gap: 1rem;
      }
      
      .result-card {
          background: white;
          padding: 1rem;
          border-radius: 4px;
          box-shadow: 0 1px 3px rgba(0,0,0,0.1);
      }
      
      .toast {
          position: fixed;
          bottom: 2rem;
          right: 2rem;
          padding: 1rem 2rem;
          border-radius: 4px;
          color: white;
          z-index: 1000;
          display: none;
      }
      
      .toast.success {
          background: var(--success-color);
      }
      
      .toast.error {
          background: var(--danger-color);
      }
      
      .measurement-overlay {
          position: absolute;
          background: rgba(255,255,255,0.9);
          padding: 0.5rem;
          border-radius: 4px;
          pointer-events: none;
      }
      
      @media (max-width: 768px) {
          .app-container {
              grid-template-columns: 1fr;
          }
          
          .sidebar {
              display: none;
          }
      }
  </style>
Project Settings          <div class="input-group">
              <label for="length">Length (ft)</label>
              <input type="number" id="length" min="0" step="0.1">
          </div>
          
          <div class="input-group">
              <label for="width">Width (ft)</label>
              <input type="number" id="width" min="0" step="0.1">
          </div>
          
          <div class="input-group">
              <label for="height">Height (ft)</label>
              <input type="number" id="height" min="0" step="0.1">
          </div>
          
          <div class="input-group">
              <label for="railingType">Railing Type</label>
              <select id="railingType">
                  <option value="wood">Wood</option>
                  <option value="aluminum">Aluminum</option>
                  <option value="cable">Cable</option>
                  <option value="glass">Glass</option>
              </select>
          </div>
          
          <div class="input-group">
              <label for="stairs">Stairs</label>
              <select id="stairs">
                  <option value="no">No Stairs</option>
                  <option value="straight">Straight Stairs</option>
                  <option value="curved">Curved Stairs</option>
              </select>
          </div>
          
          <button class="btn" onclick="generateEstimate()">Generate Estimate</button>
      </aside>
      
      <main class="main-content">
          <div class="card">
              <h2>3D Visualization</h2>
              <div class="viewer-container">
                  <div id="3dViewer"></div>
                  <div class="controls-overlay">
                      <button class="btn" onclick="toggleMeasurementMode()">Measure</button>
                      <button class="btn" onclick="startARView()">AR View</button>
                      <button class="btn" onclick="captureLiDAR()">Scan Area</button>
                  </div>
              </div>
          </div>
          
          <div class="card">
              <h2>Estimate Summary</h2>
              <div class="results">
                  <div class="result-card">
                      <h3>Materials</h3>
                      <p id="materialCost">$0.00</p>
                  </div>
                  <div class="result-card">
                      <h3>Labor</h3>
                      <p id="laborCost">$0.00</p>
                  </div>
                  <div class="result-card">
                      <h3>Total</h3>
                      <p id="totalEstimate">$0.00</p>
                  </div>
              </div>
          </div>
      </main>
  </div>

  <div class="toast" id="toast"></div>

  <script>
      // Initialize Three.js scene
      let scene, camera, renderer, controls;
      let deckModel;
      let isLiDARScanning = false;
      
      function initializeViewer() {
          scene = new THREE.Scene();
          camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
          
          renderer = new THREE.WebGLRenderer({ antialias: true });
          renderer.setSize(document.getElementById('3dViewer').clientWidth, 
                         document.getElementById('3dViewer').clientHeight);
          document.getElementById('3dViewer').appendChild(renderer.domElement);
          
          controls = new THREE.OrbitControls(camera, renderer.domElement);
          controls.enableDamping = true;
          
          camera.position.set(5, 5, 5);
          controls.update();
          
          const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
          scene.add(ambientLight);
          
          const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
          directionalLight.position.set(10, 10, 10);
          scene.add(directionalLight);
          
          animate();
          updateDeckModel();
      }
      
      function animate() {
          requestAnimationFrame(animate);
          controls.update();
          renderer.render(scene, camera);
      }
      
      function updateDeckModel() {
          if (deckModel) scene.remove(deckModel);
          
          const length = parseFloat(document.getElementById('length').value) || 10;
          const width = parseFloat(document.getElementById('width').value) || 10;
          const height = parseFloat(document.getElementById('height').value) || 3;
          
          // Create deck platform
          const deckGeometry = new THREE.BoxGeometry(width, 0.2, length);
          const deckMaterial = new THREE.MeshPhongMaterial({ 
              color: getDeckColor(document.getElementById('deckType').value)
          });
          
          deckModel = new THREE.Mesh(deckGeometry, deckMaterial);
          deckModel.position.y = height;
          scene.add(deckModel);
          
          addSupportPosts(length, width, height);
          
          if (document.getElementById('railingType').value !== 'none') {
              addRailings(length, width, height);
          }
          
          if (document.getElementById('stairs').value !== 'no') {
              addStairs(height);
          }
          
          calculateEstimate();
      }
      
      // Material costs and calculations
      const MATERIAL_COSTS = {
          'pressure-treated': { base: 15, labor: 20 },
          'cedar': { base: 25, labor: 25 },
          'composite': { base: 35, labor: 30 },
          'pvc': { base: 40, labor: 30 }
      };
      
      const RAILING_COSTS = {
          'wood': { base: 30, labor: 15 },
          'aluminum': { base: 45, labor: 20 },
          'cable': { base: 60, labor: 25 },
          'glass': { base: 80, labor: 30 }
      };
      
      function calculateEstimate() {
          const length = parseFloat(document.getElementById('length').value) || 0;
          const width = parseFloat(document.getElementById('width').value) || 0;
          const deckType = document.getElementById('deckType').value;
          const railingType = document.getElementById('railingType').value;
          
          const squareFootage = length * width;
          const perimeterLength = 2 * (length + width);
          
          const materialCost = squareFootage * MATERIAL_COSTS[deckType].base +
                             perimeterLength * RAILING_COSTS[railingType].base;
          
          const laborCost = squareFootage * MATERIAL_COSTS[deckType].labor +
                           perimeterLength * RAILING_COSTS[railingType].labor;
          
          document.getElementById('materialCost').textContent = `$${materialCost.toFixed(2)}`;
          document.getElementById('laborCost').textContent = `$${laborCost.toFixed(2)}`;
          document.getElementById('totalEstimate').textContent = 
              `$${(materialCost + laborCost).toFixed(2)}`;
      }
      
      // Event listeners
      document.addEventListener('DOMContentLoaded', initializeViewer);
      
      window.addEventListener('resize', () => {
          camera.aspect = window.innerWidth / window.innerHeight;
          camera.updateProjectionMatrix();
          renderer.setSize(document.getElementById('3dViewer').clientWidth,
                         document.getElementById('3dViewer').clientHeight);
      });
      
      // Add input change listeners
      document.querySelectorAll('input, select').forEach(input => {
          input.addEventListener('change', updateDeckModel);
      });
      
      // Helper functions
      function showToast(message, type = 'success') {
          const toast = document.getElementById('toast');
          toast.textContent = message;
          toast.className = `toast ${type}`;
          toast.style.display = 'block';
          
          setTimeout(() => {
              toast.style.display = 'none';
          }, 3000);
      }
      
      // AR and LiDAR functionality
      async function startARView() {
          try {
              const session = await navigator.xr.requestSession('immersive-ar');
              showToast('AR view started');
          } catch (error) {
              showToast('AR not supported on this device', 'error');
          }
      }
      
      async function captureLiDAR() {
          if (!window.XRSystem) {
              showToast('LiDAR scanning requires compatible hardware', 'error');
              return;
          }
          
          try {
              isLiDARScanning = true;
              const session = await navigator.xr.requestSession('immersive-ar', {
                  requiredFeatures: ['depth-sensing']
              });
              showToast('LiDAR scanning started');
          } catch (error) {
              showToast('Error starting LiDAR scan: ' + error.message, 'error');
          }
      }
  </script>
This is heading 1This is heading 2This is heading 3
# TJ Deck Designer Pro - Complete PWA Project Structure

```plaintext
deck-designer-pro/
├── index.html                 # Main application entry
├── manifest.json              # PWA manifest
├── service-worker.js          # Service worker for offline functionality
├── assets/
│   ├── icons/                 # App icons for various sizes
│   ├── images/               
│   └── models/                # 3D model assets
├── css/
│   ├── main.css              # Main styles
│   ├── components.css        # Component styles
│   └── utilities.css         # Utility classes
├── js/
│   ├── app.js                # Main application logic
│   ├── three-viewer.js       # 3D viewer component
│   ├── lidar-scanner.js      # LiDAR scanning functionality
│   ├── estimate-engine.js    # Estimation calculations
│   ├── data-manager.js       # Data management
│   └── utils/
│       ├── materials.js      # Material definitions
│       ├── measurements.js    # Measurement utilities
│       └── export.js         # Export functionality
└── components/
  ├── project-manager/      # Project management components
  ├── design-tools/         # Design tool components
  └── estimate-tools/       # Estimation components
Key Files to Create:

Main Application Files
Core Functionality Modules
Component Modules
Service Worker
Manifest
Style Sheets
Utility Scripts :root {
      --primary-color: #4A90E2;
      --secondary-color: #2C3E50;
      --success-color: #2ECC71;
      --warning-color: #F1C40F;
      --danger-color: #E74C3C;
      --background-color: #F5F6FA;
      --border-color: #E1E4E8;
      --text-primary: #2C3E50;
      --text-secondary: #6B7C93;
      --shadow-sm: 0 1px 3px rgba(0,0,0,0.1);
      --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
      --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);
  }

  * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
  }

  body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      background-color: var(--background-color);
      color: var(--text-primary);
      line-height: 1.5;
  }

  .app-container {
      display: flex;
      flex-direction: column;
      min-height: 100vh;
  }

  .app-header {
      background: white;
      padding: 1rem;
      box-shadow: var(--shadow-sm);
      display: flex;
      justify-content: space-between;
      align-items: center;
  }

  .main-content {
      flex: 1;
      padding: 1rem;
  }

  .workspace {
      display: grid;
      grid-template-columns: 300px 1fr 300px;
      gap: 1rem;
      height: calc(100vh - 120px);
  }

  .project-panel,
  .estimate-panel {
      background: white;
      border-radius: 8px;
      box-shadow: var(--shadow-md);
      padding: 1rem;
      overflow-y: auto;
  }

  .viewer-section {
      background: white;
      border-radius: 8px;
      box-shadow: var(--shadow-md);
      overflow: hidden;
  }

  .viewer-container {
      width: 100%;
      height: 100%;
      position: relative;
  }

  .panel-section {
      margin-bottom: 1.5rem;
  }

  .panel-section h3 {
      margin-bottom: 1rem;
      color: var(--secondary-color);
      font-size: 1.1rem;
  }

  .input-group {
      margin-bottom: 1rem;
  }

  .input-group label {
      display: block;
      margin-bottom: 0.5rem;
      color: var(--text-secondary);
      font-size: 0.9rem;
  }

  input,
  select {
      width: 100%;
      padding: 0.75rem;
      border: 1px solid var(--border-color);
      border-radius: 4px;
      font-size: 1rem;
      transition: border-color 0.2s;
  }

  input:focus,
  select:focus {
      outline: none;
      border-color: var(--primary-color);
  }

  .btn {
      background: var(--primary-color);
      color: white;
      border: none;
      padding: 0.75rem 1.5rem;
      border-radius: 4px;
      cursor: pointer;
      font-weight: 500;
      transition: background 0.2s;
  }

  .btn:hover {
      background: #357ABD;
  }

  .toast {
      position: fixed;
      bottom: 2rem;
      right: 2rem;
      padding: 1rem 2rem;
      border-radius: 4px;
      color: white;
      z-index: 1000;
      display: none;
  }

  .toast.success {
      background: var(--success-color);
  }

  .toast.error {
      background: var(--danger-color);
  }

  @media (max-width: 1200px) {
      .workspace {
          grid-template-columns: 250px 1fr 250px;
      }
  }

  @media (max-width: 992px) {
      .workspace {
          grid-template-columns: 1fr;
          grid-template-rows: auto 1fr auto;
      }
  } # TJ Deck Designer Pro - Complete PWA Project Structure

  ```plaintext
  deck-designer-pro/
  ├── index.html                 # Main application entry
  ├── manifest.json              # PWA manifest
  ├── service-worker.js          # Service worker for offline functionality
  ├── assets/
  │   ├── icons/                 # App icons for various sizes
  │   ├── images/               
  │   └── models/                # 3D model assets
  ├── css/
  │   ├── main.css              # Main styles
  │   ├── components.css        # Component styles
  │   └── utilities.css         # Utility classes
  ├── js/
  │   ├── app.js                # Main application logic
  │   ├── three-viewer.js       # 3D viewer component
  │   ├── lidar-scanner.js      # LiDAR scanning functionality
  │   ├── estimate-engine.js    # Estimation calculations
  │   ├── data-manager.js       # Data management
  │   └── utils/
  │       ├── materials.js      # Material definitions
  │       ├── measurements.js    # Measurement utilities
  │       └── export.js         # Export functionality
  └── components/
      ├── project-manager/      # Project management components
      ├── design-tools/         # Design tool components
      └── estimate-tools/       # Estimation components
  ```

  ## Key Files to Create:

  1. Main Application Files
  2. Core Functionality Modules
  3. Component Modules
  4. Service Worker
  5. Manifest
  6. Style Sheets
  7. Utility Scripts/* Dashboard Styles */
  .dashboard-container {
    padding: 1.5rem;
    background-color: var(--background-color);
  }

  .dashboard-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1.5rem;
    margin-bottom: 2rem;
  }

  .dashboard-card {
    background: white;
    border-radius: 8px;
    box-shadow: var(--shadow-md);
    padding: 1.5rem;
  }

  /* Weather Widget */
  .weather-widget {
    background: linear-gradient(135deg, #4A90E2, #357ABD);
    color: white;
  }

  .weather-main {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin: 1rem 0;
  }

  .weather-main #weather-temp {
    font-size: 3rem;
    font-weight: bold;
  }

  .weather-details {
    margin: 1rem 0;
    font-size: 0.9rem;
  }

  .weather-forecast {
    display: flex;
    justify-content: space-between;
    margin-top: 1rem;
    padding-top: 1rem;
    border-top: 1px solid rgba(255,255,255,0.2);
  }

  /* Analytics Overview */
  .analytics-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1rem;
    margin: 1rem 0;
  }

  .analytics-item {
    padding: 1rem;
    background: var(--background-color);
    border-radius: 6px;
    text-align: center;
  }

  .analytics-label {
    display: block;
    font-size: 0.9rem;
    color: var(--text-secondary);
    margin-bottom: 0.5rem;
  }

  .analytics-value {
    font-size: 1.5rem;
    font-weight: bold;
    color: var(--text-primary);
  }

  .analytics-chart {
    margin-top: 1.5rem;
    height: 200px;
  }

  /* Project Management */
  .project-filters {
    display: flex;
    gap: 1rem;
    margin-bottom: 1rem;
  }

  .project-list {
    max-height: 400px;
    overflow-y: auto;
  }

  .project-item {
    padding: 1rem;
    border-bottom: 1px solid var(--border-color);
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .project-item:last-child {
    border-bottom: none;
  }

  .project-actions {
    display: flex;
    gap: 1rem;
    margin-top: 1rem;
    padding-top: 1rem;
    border-top: 1px solid var(--border-color);
  }

  /* Recent Activity */
  .activity-list {
    max-height: 300px;
    overflow-y: auto;
  }

  .activity-item {
    padding: 0.75rem 0;
    border-bottom: 1px solid var(--border-color);
    font-size: 0.9rem;
  }

  .activity-item:last-child {
    border-bottom: none;
  }

  .activity-time {
    color: var(--text-secondary);
    font-size: 0.8rem;
  }

  /* Responsive Adjustments */
  @media (max-width: 768px) {
    .dashboard-grid {
      grid-template-columns: 1fr;
    }
    
    .analytics-grid {
      grid-template-columns: 1fr;
    }
  }!-- Dashboard Section -->
<section class="dashboard-container">
<div class="dashboard-grid">
  <!-- Weather Widget -->
  <div class="dashboard-card weather-widget">
    <h3>Local Weather</h3>
    <div class="weather-content">
      <div class="weather-main">
        <span id="weather-temp">--°F</span>
        <img id="weather-icon" src="" alt="Weather condition">
      </div>
      <div class="weather-details">
        <p>Condition: <span id="weather-condition">--</span></p>
        <p>Humidity: <span id="weather-humidity">--%</span></p>
        <p>Wind: <span id="weather-wind">-- mph</span></p>
      </div>
      <div class="weather-forecast" id="weather-forecast">
        <!-- Forecast items will be inserted here -->
      </div>
    </div>
  </div>

  <!-- Analytics Overview -->
  <div class="dashboard-card analytics-overview">
    <h3>Project Analytics</h3>
    <div class="analytics-grid">
      <div class="analytics-item">
        <span class="analytics-label">Active Projects</span>
        <span class="analytics-value" id="active-projects">0</span>
      </div>
      <div class="analytics-item">
        <span class="analytics-label">Completed</span>
        <span class="analytics-value" id="completed-projects">0</span>
      </div>
      <div class="analytics-item">
        <span class="analytics-label">Total Revenue</span>
        <span class="analytics-value" id="total-revenue">$0</span>
      </div>
      <div class="analytics-item">
        <span class="analytics-label">Avg. Project Value</span>
        <span class="analytics-value" id="avg-project-value">$0</span>
      </div>
    </div>
    <div class="analytics-chart">
      <canvas id="projectsChart"></canvas>
    </div>
  </div>

  <!-- Project Management -->
  <div class="dashboard-card project-management">
    <h3>Project Management</h3>
    <div class="project-filters">
      <select id="project-status-filter">
        <option value="all">All Projects</option>
        <option value="active">Active</option>
        <option value="pending">Pending</option>
        <option value="completed">Completed</option>
      </select>
      <input type="text" id="project-search" placeholder="Search projects...">
    </div>
    <div class="project-list" id="project-list">
      <!-- Project items will be inserted here -->
    </div>
    <div class="project-actions">
      <button class="btn btn-primary" id="new-project">New Project</button>
      <button class="btn" id="import-project">Import</button>
    </div>
  </div>

  <!-- Recent Activity -->
  <div class="dashboard-card recent-activity">
    <h3>Recent Activity</h3>
    <div class="activity-list" id="activity-list">
      <!-- Activity items will be inserted here -->
    </div>
  </div>
</div>
</section>
  // service-templates-manager.js
  export class ServiceTemplatesManager {
    constructor() {
      this.templates = {
        interior: {
          'kitchen-remodel': {
            name: 'Kitchen Remodel',
            basePrice: 15000,
            pricePerSqFt: 150,
            categories: [
              {
                name: 'Cabinetry',
                items: [
                  { name: 'Cabinet Replacement', unit: 'linear-ft', rate: 350 },
                  { name: 'Cabinet Refinishing', unit: 'linear-ft', rate: 120 },
                  { name: 'Custom Cabinets', unit: 'linear-ft', rate: 850 }
                ]
              },
              {
                name: 'Countertops',
                items: [
                  { name: 'Granite', unit: 'sq-ft', rate: 65 },
                  { name: 'Quartz', unit: 'sq-ft', rate: 75 },
                  { name: 'Butcher Block', unit: 'sq-ft', rate: 40 }
                ]
              },
              {
                name: 'Appliances',
                items: [
                  { name: 'Appliance Installation', unit: 'piece', rate: 250 },
                  { name: 'Gas Line Installation', unit: 'linear-ft', rate: 45 }
                ]
              }
            ]
          },
          'bathroom-remodel': {
            name: 'Bathroom Remodel',
            basePrice: 8500,
            pricePerSqFt: 250,
            categories: [
              {
                name: 'Fixtures',
                items: [
                  { name: 'Toilet Installation', unit: 'piece', rate: 350 },
                  { name: 'Vanity Installation', unit: 'piece', rate: 450 },
                  { name: 'Shower/Tub Installation', unit: 'piece', rate: 1200 }
                ]
              },
              {
                name: 'Tile Work',
                items: [
                  { name: 'Floor Tile', unit: 'sq-ft', rate: 18 },
                  { name: 'Wall Tile', unit: 'sq-ft', rate: 20 },
                  { name: 'Custom Tile Design', unit: 'sq-ft', rate: 35 }
                ]
              }
            ]
          },
          'built-in-shelving': {
            name: 'Built-in Shelving',
            basePrice: 2500,
            pricePerLinearFt: 200,
            categories: [
              {
                name: 'Materials',
                items: [
                  { name: 'Basic Shelving', unit: 'linear-ft', rate: 150 },
                  { name: 'Premium Shelving', unit: 'linear-ft', rate: 250 },
                  { name: 'Custom Design', unit: 'linear-ft', rate: 350 }
                ]
              }
            ]
          },
          'interior-trim': {
            name: 'Interior Trimming Package',
            basePrice: 1500,
            pricePerLinearFt: 12,
            categories: [
              {
                name: 'Trim Types',
                items: [
                  { name: 'Crown Molding', unit: 'linear-ft', rate: 15 },
                  { name: 'Baseboards', unit: 'linear-ft', rate: 8 },
                  { name: 'Door Casings', unit: 'piece', rate: 120 }
                ]
              }
            ]
          },
          'flooring': {
            name: 'Flooring Installation',
            categories: [
              {
                name: 'Vinyl Planks',
                items: [
                  { name: 'Standard Vinyl', unit: 'sq-ft', rate: 7 },
                  { name: 'Premium Vinyl', unit: 'sq-ft', rate: 9 },
                  { name: 'Luxury Vinyl', unit: 'sq-ft', rate: 12 }
                ]
              },
              {
                name: 'Hardwood',
                items: [
                  { name: 'Engineered Hardwood', unit: 'sq-ft', rate: 12 },
                  { name: 'Solid Hardwood', unit: 'sq-ft', rate: 15 }
                ]
              }
            ]
          },
          'painting': {
            name: 'Interior Painting',
            basePrice: 500,
            pricePerSqFt: 3.5,
            categories: [
              {
                name: 'Paint Types',
                items: [
                  { name: 'Standard Paint', unit: 'sq-ft', rate: 3 },
                  { name: 'Premium Paint', unit: 'sq-ft', rate: 4.5 },
                  { name: 'Custom Color Mix', unit: 'sq-ft', rate: 5 }
                ]
              }
            ]
          }
        },
        exterior: {
          'roof-repair': {
            name: 'Roof Repair',
            basePrice: 750,
            pricePerSqFt: 8,
            categories: [
              {
                name: 'Repair Types',
                items: [
                  { name: 'Shingle Replacement', unit: 'sq-ft', rate: 7 },
                  { name: 'Leak Repair', unit: 'piece', rate: 350 },
                  { name: 'Flashing Repair', unit: 'linear-ft', rate: 25 }
                ]
              }
            ]
          },
          'roof-replacement': {
            name: 'Roof Replacement',
            basePrice: 5000,
            pricePerSqFt: 7,
            categories: [
              {
                name: 'Roofing Materials',
                items: [
                  { name: 'Asphalt Shingles', unit: 'sq-ft', rate: 6.5 },
                  { name: 'Metal Roofing', unit: 'sq-ft', rate: 12 },
                  { name: 'Tile Roofing', unit: 'sq-ft', rate: 15 }
                ]
              }
            ]
          }
        }
      };
    }

    calculateEstimate(templateId, measurements, selections = {}) {
      const [category, template] = templateId.split('.');
      const serviceTemplate = this.templates[category][template];
      
      let totalCost = serviceTemplate.basePrice || 0;
      let lineItems = [];

      // Calculate based on square footage if applicable
      if (serviceTemplate.pricePerSqFt && measurements.squareFootage) {
        totalCost += serviceTemplate.pricePerSqFt * measurements.squareFootage;
      }

      // Calculate based on linear footage if applicable
      if (serviceTemplate.pricePerLinearFt && measurements.linearFootage) {
        totalCost += serviceTemplate.pricePerLinearFt * measurements.linearFootage;
      }

      // Add selected items
      serviceTemplate.categories.forEach(category => {
        category.items.forEach(item => {
          if (selections[item.name]) {
            const quantity = selections[item.name];
            const itemCost = item.rate * quantity;
            totalCost += itemCost;
            lineItems.push({
              name: item.name,
              quantity,
              unit: item.unit,
              rate: item.rate,
              total: itemCost
            });
          }
        });
      });

      return {
        templateName: serviceTemplate.name,
        basePrice: serviceTemplate.basePrice || 0,
        lineItems,
        totalCost,
        measurements
      };
    }

    getFinancingOptions(totalCost) {
      return [
        {
          term: 12,
          monthlyPayment: totalCost / 12,
          interestRate: 0,
          type: 'Same as Cash'
        },
        {
          term: 36,
          monthlyPayment: (totalCost * 1.0499) / 36,
          interestRate: 4.99,
          type: 'Fixed Rate'
        },
        {
          term: 60,
          monthlyPayment: (totalCost * 1.0699) / 60,
          interestRate: 6.99,
          type: 'Fixed Rate'
        }
      ];
    }
  } // service-templates-manager.js
  export class ServiceTemplatesManager {
    constructor() {
      this.templates = {
        interior: {
          'kitchen-remodel': {
            name: 'Kitchen Remodel',
            basePrice: 15000,
            pricePerSqFt: 150,
            categories: [
              {
                name: 'Cabinetry',
                items: [
                  { name: 'Cabinet Replacement', unit: 'linear-ft', rate: 350 },
                  { name: 'Cabinet Refinishing', unit: 'linear-ft', rate: 120 },
                  { name: 'Custom Cabinets', unit: 'linear-ft', rate: 850 }
                ]
              },
              {
                name: 'Countertops',
                items: [
                  { name: 'Granite', unit: 'sq-ft', rate: 65 },
                  { name: 'Quartz', unit: 'sq-ft', rate: 75 },
                  { name: 'Butcher Block', unit: 'sq-ft', rate: 40 }
                ]
              },
              {
                name: 'Appliances',
                items: [
                  { name: 'Appliance Installation', unit: 'piece', rate: 250 },
                  { name: 'Gas Line Installation', unit: 'linear-ft', rate: 45 }
                ]
              }
            ]
          },
          'bathroom-remodel': {
            name: 'Bathroom Remodel',
            basePrice: 8500,
            pricePerSqFt: 250,
            categories: [
              {
                name: 'Fixtures',
                items: [
                  { name: 'Toilet Installation', unit: 'piece', rate: 350 },
                  { name: 'Vanity Installation', unit: 'piece', rate: 450 },
                  { name: 'Shower/Tub Installation', unit: 'piece', rate: 1200 }
                ]
              },
              {
                name: 'Tile Work',
                items: [
                  { name: 'Floor Tile', unit: 'sq-ft', rate: 18 },
                  { name: 'Wall Tile', unit: 'sq-ft', rate: 20 },
                  { name: 'Custom Tile Design', unit: 'sq-ft', rate: 35 }
                ]
              }
            ]
          },
          'built-in-shelving': {
            name: 'Built-in Shelving',
            basePrice: 2500,
            pricePerLinearFt: 200,
            categories: [
              {
                name: 'Materials',
                items: [
                  { name: 'Basic Shelving', unit: 'linear-ft', rate: 150 },
                  { name: 'Premium Shelving', unit: 'linear-ft', rate: 250 },
                  { name: 'Custom Design', unit: 'linear-ft', rate: 350 }
                ]
              }
            ]
          },
          'interior-trim': {
            name: 'Interior Trimming Package',
            basePrice: 1500,
            pricePerLinearFt: 12,
            categories: [
              {
                name: 'Trim Types',
                items: [
                  { name: 'Crown Molding', unit: 'linear-ft', rate: 15 },
                  { name: 'Baseboards', unit: 'linear-ft', rate: 8 },
                  { name: 'Door Casings', unit: 'piece', rate: 120 }
                ]
              }
            ]
          },
          'flooring': {
            name: 'Flooring Installation',
            categories: [
              {
                name: 'Vinyl Planks',
                items: [
                  { name: 'Standard Vinyl', unit: 'sq-ft', rate: 7 },
                  { name: 'Premium Vinyl', unit: 'sq-ft', rate: 9 },
                  { name: 'Luxury Vinyl', unit: 'sq-ft', rate: 12 }
                ]
              },
              {
                name: 'Hardwood',
                items: [
                  { name: 'Engineered Hardwood', unit: 'sq-ft', rate: 12 },
                  { name: 'Solid Hardwood', unit: 'sq-ft', rate: 15 }
                ]
              }
            ]
          },
          'painting': {
            name: 'Interior Painting',
            basePrice: 500,
            pricePerSqFt: 3.5,
            categories: [
              {
                name: 'Paint Types',
                items: [
                  { name: 'Standard Paint', unit: 'sq-ft', rate: 3 },
                  { name: 'Premium Paint', unit: 'sq-ft', rate: 4.5 },
                  { name: 'Custom Color Mix', unit: 'sq-ft', rate: 5 }
                ]
              }
            ]
          }
        },
        exterior: {
          'roof-repair': {
            name: 'Roof Repair',
            basePrice: 750,
            pricePerSqFt: 8,
            categories: [
              {
                name: 'Repair Types',
                items: [
                  { name: 'Shingle Replacement', unit: 'sq-ft', rate: 7 },
                  { name: 'Leak Repair', unit: 'piece', rate: 350 },
                  { name: 'Flashing Repair', unit: 'linear-ft', rate: 25 }
                ]
              }
            ]
          },
          'roof-replacement': {
            name: 'Roof Replacement',
            basePrice: 5000,
            pricePerSqFt: 7,
            categories: [
              {
                name: 'Roofing Materials',
                items: [
                  { name: 'Asphalt Shingles', unit: 'sq-ft', rate: 6.5 },
                  { name: 'Metal Roofing', unit: 'sq-ft', rate: 12 },
                  { name: 'Tile Roofing', unit: 'sq-ft', rate: 15 }
                ]
              }
            ]
          }
        }
      };
    }

    calculateEstimate(templateId, measurements, selections = {}) {
      const [category, template] = templateId.split('.');
      const serviceTemplate = this.templates[category][template];
      
      let totalCost = serviceTemplate.basePrice || 0;
      let lineItems = [];

      // Calculate based on square footage if applicable
      if (serviceTemplate.pricePerSqFt && measurements.squareFootage) {
        totalCost += serviceTemplate.pricePerSqFt * measurements.squareFootage;
      }

      // Calculate based on linear footage if applicable
      if (serviceTemplate.pricePerLinearFt && measurements.linearFootage) {
        totalCost += serviceTemplate.pricePerLinearFt * measurements.linearFootage;
      }

      // Add selected items
      serviceTemplate.categories.forEach(category => {
        category.items.forEach(item => {
          if (selections[item.name]) {
            const quantity = selections[item.name];
            const itemCost = item.rate * quantity;
            totalCost += itemCost;
            lineItems.push({
              name: item.name,
              quantity,
              unit: item.unit,
              rate: item.rate,
              total: itemCost
            });
          }
        });
      });

      return {
        templateName: serviceTemplate.name,
        basePrice: serviceTemplate.basePrice || 0,
        lineItems,
        totalCost,
        measurements
      };
    }

    getFinancingOptions(totalCost) {
      return [
        {
          term: 12,
          monthlyPayment: totalCost / 12,
          interestRate: 0,
          type: 'Same as Cash'
        },
        {
          term: 36,
          monthlyPayment: (totalCost * 1.0499) / 36,
          interestRate: 4.99,
          type: 'Fixed Rate'
        },
        {
          term: 60,
          monthlyPayment: (totalCost * 1.0699) / 60,
          interestRate: 6.99,
          type: 'Fixed Rate'
        }
      ];
    }
  }DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta name="theme-color" content="#4A90E2">
      <title>TJ Deck Designer Pro - 3D Edition</title>
      <link rel="manifest" href="manifest.json">
      <!-- Three.js for 3D rendering -->
      <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
      <!-- OrbitControls for 3D navigation -->
      <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
      <style>
          /* Previous CSS styles remain the same */
          
          #3dViewer {
              width: 100%;
              height: 500px;
              border-radius: 8px;
              overflow: hidden;
              position: relative;
          }
          
          .controls-overlay {
              position: absolute;
              top: 10px;
              right: 10px;
              z-index: 100;
              background: rgba(0,0,0,0.7);
              padding: 10px;
              border-radius: 4px;
              color: white;
          }
          
          .ar-view {
              position: fixed;
              top: 0;
              left: 0;
              width: 100%;
              height: 100%;
              background: rgba(0,0,0,0.9);
              z-index: 1000;
              display: none;
          }
          
          .ar-controls {
              position: fixed;
              bottom: 20px;
              left: 50%;
              transform: translateX(-50%);
              z-index: 1001;
              display: flex;
              gap: 10px;
          }
          
          .measurement-overlay {
              position: absolute;
              background: rgba(255,255,255,0.9);
              padding: 10px;
              border-radius: 4px;
              pointer-events: none;
          }
      </style>
  </head>
  <body>
      <!-- Previous HTML structure remains, adding new 3D and LiDAR sections -->
      
      <div class="card">
          <h2>3D Visualization</h2>
          <div id="3dViewer">
              <div class="controls-overlay">
                  <button onclick="toggleMeasurementMode()" class="btn">Toggle Measurement</button>
                  <button onclick="startARView()" class="btn">View in AR</button>
                  <button onclick="captureLiDAR()" class="btn">Scan Area</button>
              </div>
          </div>
      </div>
      
      <div class="ar-view" id="arView">
          <div class="ar-controls">
              <button onclick="closeARView()" class="btn">Exit AR</button>
              <button onclick="takeMeasurement()" class="btn">Measure</button>
              <button onclick="saveARData()" class="btn">Save Scan</button>
          </div>
      </div>

      <script>
          // Three.js setup
          let scene, camera, renderer, controls;
          let deckModel, measurementPoints = [];
          let isLiDARScanning = false;
          
          function initializeViewer() {
              scene = new THREE.Scene();
              camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
              
              renderer = new THREE.WebGLRenderer({ antialias: true });
              renderer.setSize(document.getElementById('3dViewer').clientWidth, 
                             document.getElementById('3dViewer').clientHeight);
              document.getElementById('3dViewer').appendChild(renderer.domElement);
              
              // Add OrbitControls
              controls = new THREE.OrbitControls(camera, renderer.domElement);
              controls.enableDamping = true;
              controls.dampingFactor = 0.05;
              
              // Initial camera position
              camera.position.set(5, 5, 5);
              controls.update();
              
              // Add lighting
              const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
              scene.add(ambientLight);
              
              const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
              directionalLight.position.set(10, 10, 10);
              scene.add(directionalLight);
              
              animate();
          }
          
          function animate() {
              requestAnimationFrame(animate);
              controls.update();
              renderer.render(scene, camera);
          }
          
          function updateDeckModel() {
              // Remove existing deck model if any
              if (deckModel) scene.remove(deckModel);
              
              const length = parseFloat(document.getElementById('length').value) || 10;
              const width = parseFloat(document.getElementById('width').value) || 10;
              const height = parseFloat(document.getElementById('height').value) || 3;
              
              // Create deck geometry
              const deckGeometry = new THREE.BoxGeometry(width, 0.2, length);
              const deckMaterial = new THREE.MeshPhongMaterial({ 
                  color: getDeckColor(document.getElementById('deckType').value)
              });
              
              deckModel = new THREE.Mesh(deckGeometry, deckMaterial);
              deckModel.position.y = height;
              scene.add(deckModel);
              
              // Add support posts
              addSupportPosts(length, width, height);
              
              // Add railings if selected
              if (document.getElementById('railingType').value !== 'none') {
                  addRailings(length, width, height);
              }
              
              // Add stairs if selected
              if (document.getElementById('stairs').value !== 'no') {
                  addStairs(height);
              }
          }
          
          function getDeckColor(deckType) {
              const colors = {
                  'pressure-treated': 0x8B7355,
                  'cedar': 0xD2691E,
                  'composite': 0x8B4513,
                  'pvc': 0xA0522D
              };
              return colors[deckType] || 0x8B7355;
          }
          
          function addSupportPosts(length, width, height) {
              const postGeometry = new THREE.BoxGeometry(0.4, height, 0.4);
              const postMaterial = new THREE.MeshPhongMaterial({ color: 0x4A4A4A });
              
              // Add posts at corners
              const corners = [
                  [-width/2, 0, -length/2],
                  [-width/2, 0, length/2],
                  [width/2, 0, -length/2],
                  [width/2, 0, length/2]
              ];
              
              corners.forEach(pos => {
                  const post = new THREE.Mesh(postGeometry, postMaterial);
                  post.position.set(pos[0], height/2, pos[2]);
                  scene.add(post);
              });
          }
          
          function addRailings(length, width, height) {
              const railingType = document.getElementById('railingType').value;
              const railingHeight = 3;
              const railingMaterial = new THREE.MeshPhongMaterial({
                  color: railingType === 'wood' ? 0x8B7355 : 0xC0C0C0,
                  opacity: railingType === 'glass' ? 0.5 : 1,
                  transparent: railingType === 'glass'
              });
              
              // Create railing geometry based on type
              if (railingType === 'glass') {
                  addGlassRailings(length, width, height, railingHeight);
              } else {
                  addTraditionalRailings(length, width, height, railingHeight, railingMaterial);
              }
          }
          
          function addStairs(deckHeight) {
              const stairType = document.getElementById('stairs').value;
              const stepHeight = 0.2;
              const stepDepth = 0.3;
              const numSteps = Math.ceil(deckHeight / stepHeight);
              
              const stepGeometry = new THREE.BoxGeometry(2, stepHeight, stepDepth);
              const stepMaterial = new THREE.MeshPhongMaterial({ 
                  color: getDeckColor(document.getElementById('deckType').value)
              });
              
              for (let i = 0; i < numSteps; i++) {
                  const step = new THREE.Mesh(stepGeometry, stepMaterial);
                  if (stairType === 'straight') {
                      step.position.set(0, i * stepHeight, -length/2 - (i * stepDepth));
                  } else {
                      // Curved stairs - calculate position on arc
                      const angle = (i / numSteps) * Math.PI / 2;
                      const radius = 2;
                      step.position.set(
                          radius * Math.cos(angle),
                          i * stepHeight,
                          -length/2 - radius * Math.sin(angle)
                      );
                      step.rotation.y = angle;
                  }
                  scene.add(step);
              }
          }
          
          // LiDAR and AR functionality
          async function captureLiDAR() {
              if (!window.XRSystem) {
                  showToast('LiDAR scanning requires compatible hardware', 'error');
                  return;
              }
              
              try {
                  isLiDARScanning = true;
                  const session = await navigator.xr.requestSession('immersive-ar', {
                      requiredFeatures: ['depth-sensing']
                  });
                  
                  session.addEventListener('select', handleLiDARData);
                  showToast('LiDAR scanning started. Move device to capture area.', 'success');
              } catch (error) {
                  showToast('Error starting LiDAR scan: ' + error.message, 'error');
              }
          }
          
          function handleLiDARData(event) {
              if (!isLiDARScanning) return;
              
              const frame = event.frame;
              const depthData = frame.getDepthInformation();
              
              if (depthData) {
                  // Process depth data and update 3D model
                  updateModelFromScan(depthData);
              }
          }
          
          function updateModelFromScan(depthData) {
              // Convert depth data to point cloud
              const pointCloud = new THREE.Points(
                  new THREE.BufferGeometry(),
                  new THREE.PointsMaterial({ size: 0.01, color: 0x00ff00 })
              );
              
              // Update geometry with scan data
              const positions = new Float32Array(depthData.width * depthData.height * 3);
              let index = 0;
              
              for (let y = 0; y < depthData.height; y++) {
                  for (let x = 0; x < depthData.width; x++) {
                      const depth = depthData.getDepth(x, y);
                      positions[index++] = x * 0.01;
                      positions[index++] = y * 0.01;
                      positions[index++] = depth;
                  }
              }
              
              pointCloud.geometry.setAttribute('position', 
                  new THREE.BufferAttribute(positions, 3));
              scene.add(pointCloud);
          }
          
          // Initialize everything when document loads
          document.addEventListener('DOMContentLoaded', () => {
              initializeViewer();
              updateDeckModel();
              
              // Add event listeners for dimension changes
              document.querySelectorAll('input, select').forEach(input => {
                  input.addEventListener('change', () => {
                      updateDeckModel();
                      calculateEstimate();
                  });
              });
          });
          
          // Handle window resize
          window.addEventListener('resize', () => {
              camera.aspect = window.innerWidth / window.innerHeight;
              camera.updateProjectionMatrix();
              renderer.setSize(document.getElementById('3dViewer').clientWidth,
                             document.getElementById('3dViewer').clientHeight);
          });
      </script>
  </body>
  </html> <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta name="theme-color" content="#4A90E2">
      <title>TJ Deck Designer</title>
      <link rel="manifest" href="manifest.json">
      <style>
          :root {
              --primary-color: #4A90E2;
              --secondary-color: #2C3E50;
              --background-color: #F5F6FA;
          }
          
          * {
              box-sizing: border-box;
              margin: 0;
              padding: 0;
          }
          
          body {
              font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
              background-color: var(--background-color);
              color: var(--secondary-color);
              line-height: 1.6;
          }
          
          .container {
              max-width: 100%;
              padding: 1rem;
              margin: 0 auto;
          }
          
          .header {
              background-color: var(--primary-color);
              color: white;
              padding: 1rem;
              text-align: center;
              position: sticky;
              top: 0;
              z-index: 100;
          }
          
          .input-group {
              margin-bottom: 1.5rem;
              background: white;
              padding: 1rem;
              border-radius: 8px;
              box-shadow: 0 2px 4px rgba(0,0,0,0.1);
          }
          
          label {
              display: block;
              margin-bottom: 0.5rem;
              font-weight: 600;
          }
          
          input, select {
              width: 100%;
              padding: 0.75rem;
              border: 1px solid #ddd;
              border-radius: 4px;
              font-size: 1rem;
              margin-bottom: 0.5rem;
          }
          
          .calculate-btn {
              background-color: var(--primary-color);
              color: white;
              padding: 1rem;
              border: none;
              border-radius: 4px;
              width: 100%;
              font-size: 1.1rem;
              font-weight: 600;
              cursor: pointer;
              margin-top: 1rem;
          }
          
          .results {
              margin-top: 2rem;
              background: white;
              padding: 1rem;
              border-radius: 8px;
              box-shadow: 0 2px 4px rgba(0,0,0,0.1);
          }
          
          .result-item {
              display: flex;
              justify-content: space-between;
              padding: 0.5rem 0;
              border-bottom: 1px solid #eee;
          }
          
          .result-item:last-child {
              border-bottom: none;
              font-weight: 600;
          }
          
          @media (min-width: 768px) {
              .container {
                  max-width: 768px;
              }
          }
      </style>
  </head>
  <body>
      <div class="header">
          <h1>TJ Deck Designer</h1>
      </div>
      
      <div class="container">
          <div class="input-group">
              <label for="deckType">Deck Type</label>
              <select id="deckType">
                  <option value="pressure-treated">Pressure Treated</option>
                  <option value="cedar">Cedar</option>
                  <option value="composite">Composite</option>
                  <option value="pvc">PVC</option>
              </select>
              
              <label for="length">Length (ft)</label>
              <input type="number" id="length" min="0" step="0.1">
              
              <label for="width">Width (ft)</label>
              <input type="number" id="width" min="0" step="0.1">
              
              <label for="height">Height (ft)</label>
              <input type="number" id="height" min="0" step="0.1">
              
              <label for="railingType">Railing Type</label>
              <select id="railingType">
                  <option value="wood">Wood</option>
                  <option value="aluminum">Aluminum</option>
                  <option value="cable">Cable</option>
                  <option value="glass">Glass</option>
              </select>
              
              <button class="calculate-btn" onclick="calculateEstimate()">Calculate Estimate</button>
          </div>
          
          <div class="results" id="results">
              <div class="result-item">
                  <span>Square Footage:</span>
                  <span id="squareFootage">-</span>
              </div>
              <div class="result-item">
                  <span>Material Cost:</span>
                  <span id="materialCost">-</span>
              </div>
              <div class="result-item">
                  <span>Labor Cost:</span>
                  <span id="laborCost">-</span>
              </div>
              <div class="result-item">
                  <span>Total Estimate:</span>
                  <span id="totalEstimate">-</span>
              </div>
          </div>
      </div>

      <script>
          // Material costs per square foot
          const materialCosts = {
              'pressure-treated': 15,
              'cedar': 25,
              'composite': 35,
              'pvc': 40
          };
          
          // Railing costs per linear foot
          const railingCosts = {
              'wood': 30,
              'aluminum': 45,
              'cable': 60,
              'glass': 80
          };
          
          // Labor cost per square foot
          const laborCostPerSqFt = 20;
          
          function calculateEstimate() {
              const length = parseFloat(document.getElementById('length').value);
              const width = parseFloat(document.getElementById('width').value);
              const height = parseFloat(document.getElementById('height').value);
              const deckType = document.getElementById('deckType').value;
              const railingType = document.getElementById('railingType').value;
              
              if (!length || !width) {
                  alert('Please enter valid dimensions');
                  return;
              }
              
              const squareFootage = length * width;
              const perimeterLength = 2 * (length + width);
              
              const materialCost = squareFootage * materialCosts[deckType];
              const railingCost = perimeterLength * railingCosts[railingType];
              const laborCost = squareFootage * laborCostPerSqFt;
              
              const totalCost = materialCost + railingCost + laborCost;
              
              document.getElementById('squareFootage').textContent = 
                  `${squareFootage.toFixed(1)} sq ft`;
              document.getElementById('materialCost').textContent = 
                  `$${(materialCost + railingCost).toFixed(2)}`;
              document.getElementById('laborCost').textContent = 
                  `$${laborCost.toFixed(2)}`;
              document.getElementById('totalEstimate').textContent = 
                  `$${totalCost.toFixed(2)}`;
          }
          
          // Service Worker Registration
          if ('serviceWorker' in navigator) {
              window.addEventListener('load', () => {
                  navigator.serviceWorker.register('service-worker.js')
                      .then(registration => {
                          console.log('ServiceWorker registration successful');
                      })
                      .catch(err => {
                          console.log('ServiceWorker registration failed: ', err);
                      });
              });
          }
      </script>
  </body>
  </html> {
    "name": "TJ Deck Designer",
    "short_name": "Deck Designer",
    "start_url": "index.html",
    "display": "standalone",
    "background_color": "#F5F6FA",
    "theme_color": "#4A90E2",
    "icons": [
      {
        "src": "icon-192.png",
        "sizes": "192x192",
        "type": "image/png"
      },
      {
        "src": "icon-512.png",
        "sizes": "512x512",
        "type": "image/png"
      }
    ]
  }  const CACHE_NAME = 'deck-designer-v1';
  const urlsToCache = [
    '/',
    '/index.html',
    '/manifest.json',
    '/icon-192.png',
    '/icon-512.png'
  ];

  self.addEventListener('install', event => {
    event.waitUntil(
      caches.open(CACHE_NAME)
        .then(cache => cache.addAll(urlsToCache))
    );
  });

  self.addEventListener('fetch', event => {
    event.respondWith(
      caches.match(event.request)
        .then(response => response || fetch(event.request))
    );
  });   const CACHE_NAME = 'deck-designer-v1';
  const urlsToCache = [
    '/',
    '/index.html',
    '/manifest.json',
    '/icon-192.png',
    '/icon-512.png'
  ];

  self.addEventListener('install', event => {
    event.waitUntil(
      caches.open(CACHE_NAME)
        .then(cache => cache.addAll(urlsToCache))
    );
  });

  self.addEventListener('fetch', event => {
    event.respondWith(
      caches.match(event.request)
        .then(response => response || fetch(event.request))
    );
  });   import React from 'react'
  import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table"
  import { Card, CardContent } from "@/components/ui/card"

  const QuoteGeneratorDesign = () => {
    return (
      <div className="space-y-8">
        <Card>
          <CardContent className="p-6">
            <h2 className="text-xl font-bold mb-4">Quote Generator Sheet Layout</h2>
            <Table>
              <TableHeader>
                <TableRow>
                  <TableHead>Section</TableHead>
                  <TableHead>Location</TableHead>
                  <TableHead>Formulas/Links</TableHead>
                  <TableHead>Features</TableHead>
                </TableRow>
              </TableHeader>
              <TableBody>
                <TableRow>
                  <TableCell>Client Info (A1:D10)</TableCell>
                  <TableCell>Top Left</TableCell>
                  <TableCell>
                    <code>VLOOKUP(ClientID, ClientDB)</code>
                    <br />
                    Data validation for existing clients
                  </TableCell>
                  <TableCell>
                    <ul className="list-disc list-inside">
                      <li>Name</li>
                      <li>Address</li>
                      <li>Contact</li>
                      <li>Project Date</li>
                    </ul>
                  </TableCell>
                </TableRow>
                <TableRow>
                  <TableCell>Project Details (E1:H10)</TableCell>
                  <TableCell>Top Right</TableCell>
                  <TableCell>
                    <code>TODAY()</code>
                    <br />
                    <code>QUOTENUMBER()</code>
                    <br />
                    Auto-numbering system
                  </TableCell>
                  <TableCell>
                    <ul className="list-disc list-inside">
                      <li>Quote #</li>
                      <li>Date</li>
                      <li>Valid Until</li>
                      <li>Terms</li>
                    </ul>
                  </TableCell>
                </TableRow>
                <TableRow>
                  <TableCell>Item Selection (A11:H30)</TableCell>
                  <TableCell>Main Body</TableCell>
                  <TableCell>
                    <code>INDEX(Catalogs!...)</code>
                    <br />
                    <code>VLOOKUP(ItemCode, Catalogs)</code>
                  </TableCell>
                  <TableCell>
                    <ul className="list-disc list-inside">
                      <li>Dropdown menus</li>
                      <li>Auto-pricing</li>
                      <li>Quantity inputs</li>
                      <li>Line totals</li>
                    </ul>
                  </TableCell>
                </TableRow>
                <TableRow>
                  <TableCell>Calculations (I11:K30)</TableCell>
                  <TableCell>Right Side</TableCell>
                  <TableCell>
                    <code>SUM(LineItems)</code>
                    <br />
                    <code>SUMPRODUCT(Qty, Price)</code>
                  </TableCell>
                  <TableCell>
                    <ul className="list-disc list-inside">
                      <li>Subtotals</li>
                      <li>Tax</li>
                      <li>Total</li>
                      <li>Discounts</li>
                    </ul>
                  </TableCell>
                </TableRow>
              </TableBody>
            </Table>
          </CardContent>
        </Card>
      </div>
    )
  }

  export default QuoteGeneratorDesignimport React from 'react'
  import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table"
  import { Card, CardContent } from "@/components/ui/card"
  import { Input } from "@/components/ui/input"
  import { Label } from "@/components/ui/label"

  const QuoteGenerator = () => {
    const today = new Date().toLocaleDateString()
    const expiryDate = new Date()
    expiryDate.setDate(expiryDate.getDate() + 30)

    return (
      <div className="space-y-6 p-4">
        {/* Header */}
        <div className="text-2xl font-bold text-center">QUOTE</div>

        {/* Client and Quote Details Section */}
        <div className="grid md:grid-cols-2 gap-6">
          {/* Client Details */}
          <Card>
            <CardContent className="p-4">
              <h3 className="font-semibold mb-4">Client Details</h3>
              <div className="space-y-4">
                <div>
                  <Label htmlFor="name">Name:</Label>
                  <Input id="name" />
                </div>
                <div>
                  <Label htmlFor="address">Address:</Label>
                  <Input id="address" />
                </div>
                <div>
                  <Label htmlFor="phone">Phone:</Label>
                  <Input id="phone" />
                </div>
                <div>
                  <Label htmlFor="email">Email:</Label>
                  <Input id="email" type="email" />
                </div>
              </div>
            </CardContent>
          </Card>

          {/* Quote Details */}
          <Card>
            <CardContent className="p-4">
              <h3 className="font-semibold mb-4">Quote Details</h3>
              <div className="space-y-4">
                <div>
                  <Label htmlFor="quoteNumber">Quote #:</Label>
                  <Input id="quoteNumber" />
                </div>
                <div>
                  <Label>Date:</Label>
                  <Input value={today} readOnly />
                </div>
                <div>
                  <Label>Valid Until:</Label>
                  <Input value={expiryDate.toLocaleDateString()} readOnly />
                </div>
              </div>
            </CardContent>
          </Card>
        </div>

        {/* Line Items Table */}
        <Card>
          <CardContent className="p-4">
            <Table>
              <TableHeader>
                <TableRow>
                  <TableHead className="w-[40%]">Item Description</TableHead>
                  <TableHead>Quantity</TableHead>
                  <TableHead>Unit Price</TableHead>
                  <TableHead>Total</TableHead>
                </TableRow>
              </TableHeader>
              <TableBody>
                {[...Array(5)].map((_, index) => (
                  <TableRow key={index}>
                    <TableCell>
                      <Input placeholder="Description" />
                    </TableCell>
                    <TableCell>
                      <Input type="number" min="0" placeholder="0" />
                    </TableCell>
                    <TableCell>
                      <Input type="number" min="0" placeholder="0.00" />
                    </TableCell>
                    <TableCell className="font-mono">$0.00</TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </CardContent>
        </Card>

        {/* Totals Section */}
        <Card>
          <CardContent className="p-4">
            <div className="space-y-2 ml-auto w-[200px]">
              <div className="flex justify-between">
                <span>Subtotal:</span>
                <span className="font-mono">$0.00</span>
              </div>
              <div className="flex justify-between">
                <span>Tax (10%):</span>
                <span className="font-mono">$0.00</span>
              </div>
              <div className="flex justify-between font-bold">
                <span>TOTAL:</span>
                <span className="font-mono">$0.00</span>
              </div>
            </div>
          </CardContent>
        </Card>
      </div>
    )
  }

  export default QuoteGenerator
  export default WorkbookStructureCatalogs Sheet Structure
  ' Column Layout:
  A: Item Code     ' Unique identifier for each product/service
  B: Description   ' Detailed item description
  C: Base Price    ' Standard price per unit
  D: Unit Type     ' Each, Square Foot, Linear Foot, etc.
  E: Category      ' Product/Service category
  F: Labor Hours   ' Estimated labor hours
  G: Material Cost ' Base material cost
  H: Markup %      ' Standard markup percentage

  ' Named Ranges for Easy Reference:
  ' CatalogData = Catalogs!$A$2:$H$1000
  ' ItemCodes = Catalogs!$A$2:$A$1000
  ' PriceLookup = Catalogs!$A$2:$C$1000

  ' Sample Data Format:
  ' A        B                C      D    E        F     G      H
  ' DK-001   Deck Base       45.00  SF   Decking  0.5   25.00  35%
  ' DK-002   Premium Deck    65.00  SF   Decking  0.75  40.00  40%
  ' FN-001   Basic Fence     35.00  LF   Fencing  0.3   20.00  30%  ' Quote Generator Formulas and Validations

  ' Item Code Column (A11:A24)
  ' Data Validation:
  =INDIRECT("ItemCodes")  ' Drop-down list from Catalogs

  ' Description (B11:B24)
  ' Formula:
  =IFERROR(VLOOKUP(A11,PriceLookup,2,FALSE),"Select an item")

  ' Unit Price (C11:C24)
  ' Formula:
  =IFERROR(VLOOKUP(A11,PriceLookup,3,FALSE),0)

  ' Quantity (B11:B24)
  ' Data Validation:
  Minimum: 0
  Maximum: 1000
  Error Message: "Please enter a valid quantity"

  ' Line Total (D11:D24)
  ' Formula:
  =IF(AND(B11<>"",C11>0),B11*C11,0)

  ' Subtotal (E25)
  ' Formula:
  =SUM(D11:D24)

  ' Tax (E26)
  ' Formula:
  =E25*0.1

  ' Total (E27)
  ' Formula:
  =E25+E26

  ' Additional Validation Rules:
  ' 1. Prevent duplicate item codes in column A
  ' 2. Ensure quantity is entered if item code is selected
  ' 3. Format currency cells appropriately  ' Project Calculator Integration
  
  ' Labor Cost Calculation
  ' Formula (hidden helper column):
  =IFERROR(VLOOKUP(A11,CatalogData,6,FALSE)*LaborRate,0)

  ' Material Cost Calculation
  ' Formula (hidden helper column):
  =IFERROR(VLOOKUP(A11,CatalogData,7,FALSE)*B11,0)

  ' Markup Calculation
  ' Formula (hidden helper column):
  =IFERROR(VLOOKUP(A11,CatalogData,8,FALSE),0)

  ' Total Project Cost
  ' Formula:
  =SUM(MaterialCost) + SUM(LaborCost) * (1 + ProjectMarkup)  '   ' Master_Catalog Structure
  ' Columns A-C: Service codes and categories
  A: Service_Code    ' Format: CAT-###  (PER-001, FEN-001, etc.)
  B: Category        ' Main category (Pergolas, Fencing, etc.)
  C: Description     ' Detailed service description

  ' Columns D-F: Base pricing and units
  D: Base_Price      ' Standard price point
  E: Unit_Type       ' SF, LF, EA, etc.
  F: Min_Quantity    ' Minimum order quantity

  ' Columns G-I: Modification factors
  G: Complexity_Factor  ' 1 = Standard, 1.5 = Custom, 2 = Premium
  H: Season_Modifier    ' Seasonal pricing adjustments
  I: Volume_Discount    ' Quantity discount thresholds

  ' Columns J-L: Related materials
  J: Material_Codes     ' Reference to Material_Library
  K: Labor_Category     ' Reference to Labor_Rates
  L: Notes             ' Special considerations

  ' Named Ranges:
  ' MasterCatalogData = Master_Catalog!$A$2:$L$1000
  ' ServiceCodes = Master_Catalog!$A$2:$A$1000
  ' CategoryList = Master_Catalog!$B$2:$B$1000

  ' Key Formulas for Category Links:
  ' In Category sheets (e.g., Pergolas_Catalog):
  =FILTER(MasterCatalogData, MasterCatalogData[Category]="Pergolas")

  ' Price Calculation with all factors:
  =[@Base_Price] * 
   [@Complexity_Factor] * 
   [@Season_Modifier] * 
   IF([@Quantity]>[@Volume_Discount_Threshold],
      (1-[@Volume_Discount_Rate]),
      1)  ' Service Code Validation
  ' Format: CAT-### (Where CAT is 3 letters, ### is 3 numbers)
  Custom Formula: =AND(
    LEN(A2)=7,
    LEFT(A2,3)=UPPER(LEFT(A2,3)),
    MID(A2,4,1)="-",
    ISNUMBER(--RIGHT(A2,3))
  )

  ' Category Validation
  List Source: =CategoryList
  Error Message: "Select a valid category from the list"

  ' Base Price Validation
  Decimal > 0
  Error Message: "Price must be greater than zero"

  ' Unit Type Validation
  List: "SF,LF,EA,HR,SET"
  Error Message: "Select a valid unit type"

  ' Complexity Factor Validation
  List: "1,1.5,2"
  Error Message: "Select standard (1), custom (1.5), or premium (2)"

  ' Season Modifier Validation
  Decimal between 0.8 and 1.2
  Error Message: "Seasonal adjustment must be between -20% and +20%"

  ' Material Codes Validation
  List Source: =Material_Library!$A$2:$A$1000
  Error Message: "Select valid material code from library"Material Cost Lookup (in Category sheets)
  =XLOOKUP(
    [@Material_Codes],
    Material_Library[Code],
    Material_Library[Current_Cost],
    0,
    0
  )

  ' Labor Rate Calculation
  =XLOOKUP(
    [@Labor_Category],
    Labor_Rates[Category],
    Labor_Rates[Hourly_Rate],
    0,
    0
  ) * [@Complexity_Factor]

  ' Total Cost Formula
  =[@Material_Cost] + 
   ([@Labor_Hours] * [@Labor_Rate]) * 
   (1 + [@Overhead_Rate])

  ' Project Calculator Reference
  =SUMIFS(
    MasterCatalogData[Base_Price],
    MasterCatalogData[Service_Code], [@Selected_Services],
    MasterCatalogData[Category], [@Selected_Category]
  )' Beam Properties Table (Name: BeamProperties)
  ' Format: Size, Weight/ft, Moment of Inertia, Max Span, Cost/ft
  W8x10,    10,   30.8,   20,   45.00
  W8x13,    13,   39.6,   22,   52.00
  W8x15,    15,   48.0,   24,   58.00
  W10x12,   12,   53.8,   25,   54.00
  W10x15,   15,   68.9,   27,   62.00
  W12x14,   14,   88.6,   30,   60.00
  W12x16,   16,   103.0,  32,   65.00

  ' Joist Properties Table (Name: JoistProperties)
  ' Format: Size, Max Span(ft), Load Capacity(psf), Cost/ft
  2x6,      9,    40,    4.50
  2x8,      12,   50,    6.25
  2x10,     14,   60,    8.75
  2x12,     16,   70,    12.00
  2x14,     18,   80,    14.50
  2x16,     20,   90,    16.75

  ' Deck Material Properties (Name: DeckProperties)
  ' Format: Material, Cost/sqft, Load Rating, Installation Factor
  Plywood 1/2,     2.50,  40,  1.0
  Plywood 5/8,     3.25,  50,  1.0
  Plywood 3/4,     3.75,  60,  1.1
  OSB 7/16,        2.00,  35,  0.9
  OSB 23/32,       2.75,  45,  0.9
  Tongue & Groove 3/4, 4.25, 65, 1.2

  ' Calculation Formulas for Estimates Sheet

  ' Material Cost Calculation
  =SUMPRODUCT(
    VLOOKUP(B11, BeamProperties, 5, FALSE) * BeamLength,
    VLOOKUP(B12, JoistProperties, 4, FALSE) * NumberOfJoists,
    VLOOKUP(B13, DeckProperties, 2, FALSE) * FloorArea
  )

  ' Load Capacity Verification
  =IF(AND(
    VLOOKUP(B11, BeamProperties, 4, FALSE) >= SpanLength,
    VLOOKUP(B12, JoistProperties, 2, FALSE) >= JoistSpan,
    VLOOKUP(B13, DeckProperties, 3, FALSE) >= RequiredLoadRating
  ), "Design Valid", "Review Required")

  ' Installation Time Estimate
  =SUM(
    BaseInstallHours,
    FloorArea * VLOOKUP(B13, DeckProperties, 4, FALSE),
    NumberOfJoists * JoistInstallFactor,
    BeamLength * BeamInstallFactor
  )

  ' Total Project Cost Formula
  =SUM(
    MaterialCost,
    InstallationHours * LaborRate,
    OverheadFactor,
    ContingencyAmount
  ) <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Task Manager</title>
      <style>
          * {
              margin: 0;
              padding: 0;
              box-sizing: border-box;
              font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
          }

          body {
              background: #f0f2f5;
              padding: 20px;
          }

          .container {
              max-width: 800px;
              margin: 0 auto;
              background: white;
              padding: 20px;
              border-radius: 10px;
              box-shadow: 0 2px 5px rgba(0,0,0,0.1);
          }

          h1 {
              color: #1a73e8;
              margin-bottom: 20px;
              text-align: center;
          }

          .input-group {
              display: flex;
              gap: 10px;
              margin-bottom: 20px;
          }

          input[type="text"] {
              flex: 1;
              padding: 10px;
              border: 1px solid #ddd;
              border-radius: 5px;
              font-size: 16px;
          }

          button {
              padding: 10px 20px;
              background: #1a73e8;
              color: white;
              border: none;
              border-radius: 5px;
              cursor: pointer;
              transition: background 0.3s;
          }

          button:hover {
              background: #1557b0;
          }

          .task {
              display: flex;
              align-items: center;
              padding: 15px;
              background: #f8f9fa;
              margin-bottom: 10px;
              border-radius: 5px;
              gap: 10px;
          }

          .task.completed {
              background: #e8f5e9;
              text-decoration: line-through;
              color: #666;
          }

          .task input[type="checkbox"] {
              width: 20px;
              height: 20px;
          }

          .task-text {
              flex: 1;
          }

          .delete-btn {
              background: #dc3545;
              padding: 5px 10px;
              font-size: 14px;
          }

          .delete-btn:hover {
              background: #c82333;
          }

          .filters {
              display: flex;
              gap: 10px;
              margin-bottom: 20px;
              justify-content: center;
          }

          .filter-btn {
              background: #f8f9fa;
              color: #1a73e8;
              border: 1px solid #1a73e8;
          }

          .filter-btn.active {
              background: #1a73e8;
              color: white;
          }

          @media (max-width: 600px) {
              .container {
                  padding: 10px;
              }

              .input-group {
                  flex-direction: column;
              }

              .filters {
                  flex-wrap: wrap;
              }
          }
      </style>
  </head>
  <body>
      <div class="container">
          <h1>Task Manager</h1>
          
          <div class="input-group">
              <input type="text" id="taskInput" placeholder="Enter a new task...">
              <button onclick="addTask()">Add Task</button>
          </div>

          <div class="filters">
              <button class="filter-btn active" onclick="filterTasks('all')">All</button>
              <button class="filter-btn" onclick="filterTasks('active')">Active</button>
              <button class="filter-btn" onclick="filterTasks('completed')">Completed</button>
          </div>

          <div id="taskList"></div>
      </div>

      <script>
          let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
          let currentFilter = 'all';

          function addTask() {
              const input = document.getElementById('taskInput');
              const text = input.value.trim();
              
              if (text) {
                  const task = {
                      id: Date.now(),
                      text: text,
                      completed: false
                  };
                  
                  tasks.push(task);
                  saveTasks();
                  input.value = '';
                  renderTasks();
              }
          }

          function toggleTask(id) {
              tasks = tasks.map(task => 
                  task.id === id ? {...task, completed: !task.completed} : task
              );
              saveTasks();
              renderTasks();
          }

          function deleteTask(id) {
              tasks = tasks.filter(task => task.id !== id);
              saveTasks();
              renderTasks();
          }

          function filterTasks(filter) {
              currentFilter = filter;
              document.querySelectorAll('.filter-btn').forEach(btn => {
                  btn.classList.remove('active');
              });
              event.target.classList.add('active');
              renderTasks();
          }

          function renderTasks() {
              const taskList = document.getElementById('taskList');
              let filteredTasks = tasks;
              
              if (currentFilter === 'active') {
                  filteredTasks = tasks.filter(task => !task.completed);
              } else if (currentFilter === 'completed') {
                  filteredTasks = tasks.filter(task => task.completed);
              }

              taskList.innerHTML = filteredTasks.map(task => `
                  <div class="task ${task.completed ? 'completed' : ''}">
                      <input type="checkbox" 
                          ${task.completed ? 'checked' : ''} 
                          onchange="toggleTask(${task.id})">
                      <span class="task-text">${task.text}</span>
                      <button class="delete-btn" onclick="deleteTask(${task.id})">Delete</button>
                  </div>
              `).join('');
          }

          function saveTasks() {
              localStorage.setItem('tasks', JSON.stringify(tasks));
          }

          // Add task when Enter key is pressed
          document.getElementById('taskInput').addEventListener('keypress', function(e) {
              if (e.key === 'Enter') {
                  addTask();
              }
          });

          // Initial render
          renderTasks();
      </script>
  </body>
  </html>  // Estimation System Configuration
  const EstimationSystem = {
      // Material cost database
      materials: {
          concrete: {
              basePrice: 120,
              unit: 'cubic yard',
              laborCost: 80,
              wastagePercent: 10
          },
          steel: {
              basePrice: 800,
              unit: 'ton',
              laborCost: 200,
              wastagePercent: 5
          },
          glass: {
              basePrice: 40,
              unit: 'sqft',
              laborCost: 15,
              wastagePercent: 15
          }
      },

      // Labor rates
      laborRates: {
          skilled: 45,    // per hour
          unskilled: 25,  // per hour
          supervisor: 65  // per hour
      },

      // Calculate material cost with wastage
      calculateMaterialCost: function(material, quantity) {
          const materialData = this.materials[material];
          if (!materialData) return 0;

          const baseQuantity = quantity;
          const wastageQuantity = quantity * (materialData.wastagePercent / 100);
          const totalQuantity = baseQuantity + wastageQuantity;

          return {
              materialCost: totalQuantity * materialData.basePrice,
              laborCost: totalQuantity * materialData.laborCost,
              wastageAmount: wastageQuantity,
              total: totalQuantity * (materialData.basePrice + materialData.laborCost)
          };
      },

      // Calculate labor hours based on project complexity
      calculateLaborHours: function(complexity, area) {
          const laborFactors = {
              low: 0.1,
              medium: 0.2,
              high: 0.3
          };
          return area * laborFactors[complexity];
      },

      // Generate detailed estimate
      generateDetailedEstimate: function(projectData) {
          const estimate = {
              materials: {},
              labor: {},
              total: 0
          };

          // Calculate material costs
          Object.keys(projectData.materials).forEach(material => {
              estimate.materials[material] = this.calculateMaterialCost(
                  material, 
                  projectData.materials[material]
              );
          });

          // Calculate labor costs
          const laborHours = this.calculateLaborHours(
              projectData.complexity, 
              projectData.area
          );

          estimate.labor = {
              skilled: laborHours * this.laborRates.skilled,
              unskilled: laborHours * this.laborRates.unskilled,
              supervisor: (laborHours * 0.2) * this.laborRates.supervisor
          };

          // Calculate total
          estimate.total = Object.values(estimate.materials)
              .reduce((sum, mat) => sum + mat.total, 0) +
              Object.values(estimate.labor)
              .reduce((sum, labor) => sum + labor, 0);

          return estimate;
      }
  };  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>3D Design & Estimation Tool</title>
      <script src="https://app.sketchup.com/api/sketchup.js"></script>
      <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
      <style>
          .viewer-container {
              height: 600px;
              border: 1px solid #ddd;
              position: relative;
          }
          .estimation-panel {
              padding: 20px;
              background: #f8f9fa;
              border-left: 1px solid #ddd;
          }
          .material-item {
              padding: 10px;
              margin: 5px 0;
              background: white;
              border: 1px solid #eee;
              border-radius: 4px;
          }
      </style>
  </head>
  <body>
      <div class="container-fluid">
          <div class="row">
              <!-- 3D Viewer Section -->
              <div class="col-md-8">
                  <div class="viewer-container" id="viewer"></div>
                  <div class="toolbar mt-3">
                      <button class="btn btn-primary" onclick="takeScreenshot()">Screenshot</button>
                      <button class="btn btn-secondary" onclick="measureDistance()">Measure</button>
                      <button class="btn btn-info" onclick="generateEstimate()">Generate Estimate</button>
                  </div>
              </div>
              
              <!-- Estimation Panel -->
              <div class="col-md-4 estimation-panel">
                  <h3>Project Estimation</h3>
                  <div id="materialsList">
                      <!-- Materials will be populated here -->
                  </div>
                  <div class="mt-4">
                      <h4>Total Estimate</h4>
                      <div id="totalEstimate" class="h3">$0.00</div>
                  </div>
              </div>
          </div>
      </div>

      <script>
          let viewer;
          let materials = [];
          
          // Initialize SketchUp Viewer
          window.onload = async function() {
              try {
                  viewer = await sketchup.Viewer.create({
                      container: 'viewer',
                      api_token: 'YOUR_SKETCHUP_API_TOKEN'
                  });

                  // Load default model
                  await viewer.loadModel('MODEL_URL');
                  
                  // Initialize materials tracking
                  initializeMaterialsTracking();
              } catch (error) {
                  console.error('Error initializing viewer:', error);
              }
          };

          function initializeMaterialsTracking() {
              // Sample materials data
              materials = [
                  { id: 1, name: 'Concrete', price: 120, unit: 'cubic yard', quantity: 0 },
                  { id: 2, name: 'Steel Beams', price: 800, unit: 'ton', quantity: 0 },
                  { id: 3, name: 'Glass Panels', price: 40, unit: 'sqft', quantity: 0 },
                  { id: 4, name: 'Wood Framing', price: 3.50, unit: 'linear ft', quantity: 0 }
              ];
              updateMaterialsList();
          }

          function updateMaterialsList() {
              const materialsContainer = document.getElementById('materialsList');
              materialsContainer.innerHTML = materials.map(material => `
                  <div class="material-item">
                      <div class="d-flex justify-content-between">
                          <strong>${material.name}</strong>
                          <span>$${material.price}/${material.unit}</span>
                      </div>
                      <div class="input-group mt-2">
                          <input type="number" class="form-control" 
                                 value="${material.quantity}"
                                 onchange="updateQuantity(${material.id}, this.value)">
                          <span class="input-group-text">${material.unit}</span>
                      </div>
                  </div>
              `).join('');
              calculateTotal();
          }

          function updateQuantity(materialId, quantity) {
              const material = materials.find(m => m.id === materialId);
              if (material) {
                  material.quantity = parseFloat(quantity) || 0;
                  calculateTotal();
              }
          }

          function calculateTotal() {
              const total = materials.reduce((sum, material) => 
                  sum + (material.price * material.quantity), 0);
              document.getElementById('totalEstimate').innerText = 
                  `$${total.toFixed(2)}`;
          }

          function generateEstimate() {
              // Get model measurements
              viewer.getMeasurements().then(measurements => {
                  // Update quantities based on measurements
                  // This is a simplified example
                  materials.forEach(material => {
                      if (material.name === 'Concrete') {
                          material.quantity = calculateConcreteVolume(measurements);
                      } else if (material.name === 'Glass Panels') {
                          material.quantity = calculateGlassArea(measurements);
                      }
                      // Add more material calculations as needed
                  });
                  updateMaterialsList();
              });
          }

          function calculateConcreteVolume(measurements) {
              // Simplified calculation - replace with actual logic
              return Math.round(measurements.volume * 0.037); // Convert to cubic yards
          }

          function calculateGlassArea(measurements) {
              // Simplified calculation - replace with actual logic
              return Math.round(measurements.area * 10.764); // Convert to square feet
          }

          function takeScreenshot() {
              viewer.getScreenshot().then(imageData => {
                  // Handle screenshot data
                  const link = document.createElement('a');
                  link.download = 'design-screenshot.png';
                  link.href = imageData;
                  link.click();
              });
          }

          function measureDistance() {
              viewer.startMeasurement();
          }
      </script>
      <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  </body>
  </html>import pandas as pd
import numpy as np
from PIL import Image
import cv2

# Create BIM component extractor
def extract_deck_components(image_data):
    components = {
        'structural': [],
        'finishing': [],
        'hardware': [],
        'dimensions': {}
    }
    return components
  import numpy as np
  import cv2
  from dataclasses import dataclass
  
  @dataclass
  class DeckComponent:
      type: str
      dimensions: dict
      material: str
      connections: list
      texture: dict
      
  class DeckComponentExtractor:
      def __init__(self):
          self.components = []
          self.textures = {}
          
      def extract_from_image(self, image):
          # Image processing and component detection
          components = []
          return components
          
      def analyze_texture(self, component_image):
          # Texture analysis
          texture_data = {}
          return texture_data
          
      def identify_connections(self, component):
          # Connection point detection
          connections = []
          return connections
          
      def get_dimensions(self, component):
          # Dimension extraction
          dimensions = {}
          return dimensions import React from 'react';
  import { Card } from "@/components/ui/card";
  import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";

  const PartsLibrary = () => {
    return (
      <Tabs defaultValue="structural" className="w-full">
        <TabsList>
          <TabsTrigger value="structural">Structural</TabsTrigger>
          <TabsTrigger value="hardware">Hardware</TabsTrigger>
          <TabsTrigger value="finishing">Finishing</TabsTrigger>
        </TabsList>
        
        <TabsContent value="structural">
          <Card className="grid grid-cols-3 gap-4 p-4">
            {/* Structural Components */}
            <div className="component-card">
              <h4>Posts</h4>
              {/* Component details */}
            </div>
            <div className="component-card">
              <h4>Beams</h4>
              {/* Component details */}
            </div>
            <div className="component-card">
              <h4>Joists</h4>
              {/* Component details */}
            </div>
          </Card>
        </TabsContent>

        <TabsContent value="hardware">
          <Card className="grid grid-cols-3 gap-4 p-4">
            {/* Hardware Components */}
          </Card>
        </TabsContent>

        <TabsContent value="finishing">
          <Card className="grid grid-cols-3 gap-4 p-4">
            {/* Finishing Components */}
          </Card>
        </TabsContent>
      </Tabs>
    );
  };

  export default PartsLibrary;    // Excel BIM Integration System
  const ExcelBIMSystem = {
    // Component Registry
    components: {
      structural: {
        posts: [],
        beams: [],
        joists: [],
        blocking: []
      },
      hardware: {
        connectors: [],
        fasteners: [],
        brackets: []
      },
      finishing: {
        decking: [],
        railings: [],
        stairs: []
      }
    },

    // Texture Management
    textureLibrary: {
      wood: {
        pressure_treated: {},
        cedar: {},
        composite: {}
      },
      metal: {
        galvanized: {},
        powder_coated: {}
      }
    },

    // Component Extraction
    extractComponents: function(imageData) {
      return {
        type: '',
        dimensions: {},
        material: '',
        connections: []
      };
    },

    // Real-time Rendering
    renderComponent: function(component) {
      // Component rendering logic
      return {
        geometry: {},
        texture: {},
        position: {}
      };
    },

    // BIM Cell Updates
    updateBIMCell: function(cell, componentData) {
      return {
        value: componentData,
        formatting: {},
        calculations: []
      };
    }
  };  import React from 'react';
  import { Card } from "@/components/ui/card";
  import { Select } from "@/components/ui/select";
  
  const DeckDesignSystem = () => {
    return (
      <div className="grid gap-4">
        <Card className="p-4">
          <h3 className="text-lg font-bold mb-2">Component Library</h3>
          <div className="grid grid-cols-3 gap-2">
            <Select>
              <option value="structural">Structural Components</option>
              <option value="hardware">Hardware & Fasteners</option>
              <option value="finishing">Finishing Materials</option>
            </Select>
          </div>
        </Card>
        
        <Card className="p-4">
          <h3 className="text-lg font-bold mb-2">Design Parameters</h3>
          <div className="grid grid-cols-2 gap-4">
            <div>
              <label>Deck Dimensions</label>
              <input type="text" className="w-full border rounded p-2" />
            </div>
            <div>
              <label>Height</label>
              <input type="text" className="w-full border rounded p-2" />
            </div>
          </div>
        </Card>
      </div>
    );
  };

  export default DeckDesignSystem;{
    "name": "TJ Deck Designer",
    "short_name": "Deck Designer",
    "start_url": "index.html",
    "display": "standalone",
    "background_color": "#F5F6FA",
    "theme_color": "#4A90E2",
    "icons": [
      {
        "src": "icon-192.png",
        "sizes": "192x192",
        "type": "image/png"
      },
      {
        "src": "icon-512.png",
        "sizes": "512x512",
        "type": "image/png"
      }
    ]
  }

const EstimateResults = () => {
  return (
    <div className="min-h-screen bg-gray-50 p-4">
      <header className="mb-6">
        <div className="flex justify-between items-start">
          <div>
            <h1 className="text-2xl font-bold text-gray-900">Project Estimate</h1>
            <p className="text-gray-500">Custom Deck Construction</p>
          </div>
          <div className="text-right">
            <p className="text-2xl font-bold text-blue-600">$15,750</p>
            <p className="text-sm text-gray-500">Total Estimate</p>
          </div>
        </div>
      </header>

      {/* Quick Summary */}
      <div className="grid grid-cols-3 gap-4 mb-6">
        <Card className="p-4">
          <Clock className="w-5 h-5 text-blue-500 mb-2" />
          <p className="text-sm text-gray-500">Timeline</p>
          <p className="font-semibold">2-3 weeks</p>
        </Card>
        <Card className="p-4">
          <Tool className="w-5 h-5 text-blue-500 mb-2" />
          <p className="text-sm text-gray-500">Labor Hours</p>
          <p className="font-semibold">80-100 hrs</p>
        </Card>
        <Card className="p-4">
          <Calendar className="w-5 h-5 text-blue-500 mb-2" />
          <p className="text-sm text-gray-500">Start Date</p>
          <p className="font-semibold">Apr 1, 2024</p>
        </Card>
      </div>

      {/* Detailed Breakdown */}
      <Card className="mb-6">
        <div className="p-4 border-b">
          <h2 className="text-lg font-semibold">Cost Breakdown</h2>
        </div>
        <div className="divide-y">
          <div className="p-4">
            <h3 className="font-medium mb-4">Materials</h3>
            <div className="space-y-3">
              <div className="flex justify-between text-sm">
                <span>Composite Decking</span>
                <span className="font-medium">$6,000</span>
              </div>
              <div className="flex justify-between text-sm">
                <span>Framing Lumber</span>
                <span className="font-medium">$2,200</span>
              </div>
              <div className="flex justify-between text-sm">
                <span>Hardware & Fasteners</span>
                <span className="font-medium">$800</span>
              </div>
              <div className="flex justify-between text-sm">
                <span>Railings</span>
                <span className="font-medium">$1,500</span>
              </div>
            </div>
          </div>

          <div className="p-4">
            <h3 className="font-medium mb-4">Labor</h3>
            <div className="space-y-3">
              <div className="flex justify-between text-sm">
                <span>Construction Labor</span>
                <span className="font-medium">$4,000</span>
              </div>
              <div className="flex justify-between text-sm">
                <span>Design & Planning</span>
                <span className="font-medium">$750</span>
              </div>
              <div className="flex justify-between text-sm">
                <span>Permit Filing</span>
                <span className="font-medium">$500</span>
              </div>
            </div>
          </div>

          <div className="p-4">
            <h3 className="font-medium mb-4">Project Specifications</h3>
            <div className="grid grid-cols-2 gap-4 text-sm">
              <div>
                <p className="text-gray-500">Dimensions</p>
                <p className="font-medium">20' x 16'</p>
              </div>
              <div>
                <p className="text-gray-500">Height</p>
                <p className="font-medium">3 feet</p>
              </div>
              <div>
                <p className="text-gray-500">Material</p>
                <p className="font-medium">Composite Decking</p>
              </div>
              <div>
                <p className="text-gray-500">Features</p>
                <p className="font-medium">Railings, Stairs</p>
              </div>
            </div>
          </div>
        </div>
      </Card>

      {/* Notes */}
      <Card className="mb-20 p-4">
        <h2 className="font-semibold mb-2">Additional Notes</h2>
        <ul className="text-sm space-y-2 text-gray-600">
          <li>• Estimate includes all necessary permits and inspections</li>
          <li>• Timeline assumes normal weather conditions</li>
          <li>• 5-year warranty on workmanship</li>
          <li>• Manufacturer warranty on materials</li>
        </ul>
      </Card>

      {

<!<!<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TJ Suite</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            margin: 0;
            padding: 0;
            background: #f0f2f5;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        .header {
            background: #1a73e8;
            color: white;
            padding: 20px;
            text-align: center;
            border-radius: 10px;
            margin-bottom: 20px;
        }
        .feature-card {
            background: white;
            padding: 20px;
            margin: 10px 0;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            cursor: pointer;
            transition: transform 0.2s;
        }
        .feature-card:hover {
            transform: translateY(-2px);
        }
        .button {
            background: #1a73e8;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            margin: 5px;
        }
        .button:hover {
            background: #1557b0;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Welcome to TJ Suite</h1>
            <p>Your Personal Management Solution</p>
        </div>
        
        <div class="feature-card">
            <h2>Quick Actions</h2>
            <button class="button" onclick="alert('Feature coming soon!')">Start Task</button>
            <button class="button" onclick="alert('Feature coming soon!')">View Schedule</button>
        </div>

        <div class="feature-card">
            <h2>Recent Activities</h2>
            <div id="activities">
                <p>No recent activities</p>
            </div>
        </div>

        <div class="feature-card">
            <h2>Tools</h2>
            <button class="button" onclick="alert('Calendar feature coming soon!')">Calendar</button>
            <button class="button" onclick="alert('Tasks feature coming soon!')">Tasks</button>
            <button class="button" onclick="alert('Notes feature coming soon!')">Notes</button>
        </div>
    </div>

    <script>
        // Basic interaction
        document.querySelectorAll('.feature-card').forEach(card => {
            card.addEventListener('click', () => {
                card.style.backgroundColor = '#f8f9fa';
                setTimeout(() => {
                    card.style.backgroundColor = 'white';
                }, 200);
            });
        });
    </script>
</body>
</html> html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>T&J LLC Business Suite</title>
    <link rel="stylesheet" href="styles.css">
    <link rel="manifest" href="manifest.json">
    <link rel="icon" type="image/png" sizes="192x192" href="icons/icon-192x192.png">
    <link rel="apple-touch-icon" href="icons/icon-192x192.png">
    <meta name="theme-color" content="#2563eb">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black">
    <meta name="apple-mobile-web-app-title" content="T&J LLC Business Suite">
</head>
<body>
    <div id="app" class="app-container">
        <header class="card">
            <h1>T&J LLC Business Suite</h1>
            <nav id="main-nav">
                <!-- Navigation will be populated by JavaScript -->
            </nav>
        </header>

        <main id="main-content">
            <!-- Content will be dynamically loaded -->
        </main>

        <div id="loading" style="display: none;">
            <div class="spinner"></div>
        </div>
    </div>
    <script src="app.js"></script>
</body>
</html> html>
<html>
  <head>
    <title>Title of the document</title>
  </  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta name="theme-color" content="#4A90E2">
      <title>TJ Deck Designer Pro</title>
      <link rel="manifest" href="manifest.json">
      <!-- Three.js and dependencies -->
      <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
      <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
      <style>
          :root {
              --primary-color: #4A90E2;
              --secondary-color: #2C3E50;
              --success-color: #2ECC71;
              --warning-color: #F1C40F;
              --danger-color: #E74C3C;
              --background-color: #F5F6FA;
          }
          
          * {
              box-sizing: border-box;
              margin: 0;
              padding: 0;
              font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
          }
          
          body {
              background-color: var(--background-color);
              color: var(--secondary-color);
          }
          
          .app-container {
              display: grid;
              grid-template-columns: 300px 1fr;
              min-height: 100vh;
          }
          
          .sidebar {
              background: white;
              padding: 1.5rem;
              border-right: 1px solid #ddd;
              overflow-y: auto;
          }
          
          .main-content {
              padding: 1.5rem;
              overflow-y: auto;
          }
          
          .card {
              background: white;
              border-radius: 8px;
              box-shadow: 0 2px 4px rgba(0,0,0,0.1);
              padding: 1.5rem;
              margin-bottom: 1.5rem;
          }
          
          .viewer-container {
              width: 100%;
              height: 600px;
              border-radius: 8px;
              overflow: hidden;
              position: relative;
          }
          
          .controls-overlay {
              position: absolute;
              top: 1rem;
              right: 1rem;
              z-index: 100;
              display: flex;
              gap: 0.5rem;
          }
          
          .btn {
              background: var(--primary-color);
              color: white;
              border: none;
              padding: 0.75rem 1.5rem;
              border-radius: 4px;
              cursor: pointer;
              font-weight: 600;
              transition: background 0.3s;
          }
          
          .btn:hover {
              background: #357ABD;
          }
          
          .btn.secondary {
              background: var(--secondary-color);
          }
          
          .input-group {
              margin-bottom: 1rem;
          }
          
          label {
              display: block;
              margin-bottom: 0.5rem;
              font-weight: 600;
          }
          
          input, select {
              width: 100%;
              padding: 0.75rem;
              border: 1px solid #ddd;
              border-radius: 4px;
              font-size: 1rem;
          }
          
          .results {
              display: grid;
              grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
              gap: 1rem;
          }
          
          .result-card {
              background: white;
              padding: 1rem;
              border-radius: 4px;
              box-shadow: 0 1px 3px rgba(0,0,0,0.1);
          }
          
          .toast {
              position: fixed;
              bottom: 2rem;
              right: 2rem;
              padding: 1rem 2rem;
              border-radius: 4px;
              color: white;
              z-index: 1000;
              display: none;
          }
          
          .toast.success {
              background: var(--success-color);
          }
          
          .toast.error {
              background: var(--danger-color);
          }
          
          .measurement-overlay {
              position: absolute;
              background: rgba(255,255,255,0.9);
              padding: 0.5rem;
              border-radius: 4px;
              pointer-events: none;
          }
          
          @media (max-width: 768px) {
              .app-container {
                  grid-template-columns: 1fr;
              }
              
              .sidebar {
                  display: none;
              }
          }
      </style>
  </head>
  <body>
      <div class="app-container">
          <aside class="sidebar">
              <h2>Project Settings</h2>
              <div class="input-group">
                  <label for="deckType">Deck Type</label>
                  <select id="deckType">
                      <option value="pressure-treated">Pressure Treated</option>
                      <option value="cedar">Cedar</option>
                      <option value="composite">Composite</option>
                      <option value="pvc">PVC</option>
                  </select>
              </div>
              
              <div class="input-group">
                  <label for="length">Length (ft)</label>
                  <input type="number" id="length" min="0" step="0.1">
              </div>
              
              <div class="input-group">
                  <label for="width">Width (ft)</label>
                  <input type="number" id="width" min="0" step="0.1">
              </div>
              
              <div class="input-group">
                  <label for="height">Height (ft)</label>
                  <input type="number" id="height" min="0" step="0.1">
              </div>
              
              <div class="input-group">
                  <label for="railingType">Railing Type</label>
                  <select id="railingType">
                      <option value="wood">Wood</option>
                      <option value="aluminum">Aluminum</option>
                      <option value="cable">Cable</option>
                      <option value="glass">Glass</option>
                  </select>
              </div>
              
              <div class="input-group">
                  <label for="stairs">Stairs</label>
                  <select id="stairs">
                      <option value="no">No Stairs</option>
                      <option value="straight">Straight Stairs</option>
                      <option value="curved">Curved Stairs</option>
                  </select>
              </div>
              
              <button class="btn" onclick="generateEstimate()">Generate Estimate</button>
          </aside>
          
          <main class="main-content">
              <div class="card">
                  <h2>3D Visualization</h2>
                  <div class="viewer-container">
                      <div id="3dViewer"></div>
                      <div class="controls-overlay">
                          <button class="btn" onclick="toggleMeasurementMode()">Measure</button>
                          <button class="btn" onclick="startARView()">AR View</button>
                          <button class="btn" onclick="captureLiDAR()">Scan Area</button>
                      </div>
                  </div>
              </div>
              
              <div class="card">
                  <h2>Estimate Summary</h2>
                  <div class="results">
                      <div class="result-card">
                          <h3>Materials</h3>
                          <p id="materialCost">$0.00</p>
                      </div>
                      <div class="result-card">
                          <h3>Labor</h3>
                          <p id="laborCost">$0.00</p>
                      </div>
                      <div class="result-card">
                          <h3>Total</h3>
                          <p id="totalEstimate">$0.00</p>
                      </div>
                  </div>
              </div>
          </main>
      </div>

      <div class="toast" id="toast"></div>

      <script>
          // Initialize Three.js scene
          let scene, camera, renderer, controls;
          let deckModel;
          let isLiDARScanning = false;
          
          function initializeViewer() {
              scene = new THREE.Scene();
              camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
              
              renderer = new THREE.WebGLRenderer({ antialias: true });
              renderer.setSize(document.getElementById('3dViewer').clientWidth, 
                             document.getElementById('3dViewer').clientHeight);
              document.getElementById('3dViewer').appendChild(renderer.domElement);
              
              controls = new THREE.OrbitControls(camera, renderer.domElement);
              controls.enableDamping = true;
              
              camera.position.set(5, 5, 5);
              controls.update();
              
              const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
              scene.add(ambientLight);
              
              const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
              directionalLight.position.set(10, 10, 10);
              scene.add(directionalLight);
              
              animate();
              updateDeckModel();
          }
          
          function animate() {
              requestAnimationFrame(animate);
              controls.update();
              renderer.render(scene, camera);
          }
          
          function updateDeckModel() {
              if (deckModel) scene.remove(deckModel);
              
              const length = parseFloat(document.getElementById('length').value) || 10;
              const width = parseFloat(document.getElementById('width').value) || 10;
              const height = parseFloat(document.getElementById('height').value) || 3;
              
              // Create deck platform
              const deckGeometry = new THREE.BoxGeometry(width, 0.2, length);
              const deckMaterial = new THREE.MeshPhongMaterial({ 
                  color: getDeckColor(document.getElementById('deckType').value)
              });
              
              deckModel = new THREE.Mesh(deckGeometry, deckMaterial);
              deckModel.position.y = height;
              scene.add(deckModel);
              
              addSupportPosts(length, width, height);
              
              if (document.getElementById('railingType').value !== 'none') {
                  addRailings(length, width, height);
              }
              
              if (document.getElementById('stairs').value !== 'no') {
                  addStairs(height);
              }
              
              calculateEstimate();
          }
          
          // Material costs and calculations
          const MATERIAL_COSTS = {
              'pressure-treated': { base: 15, labor: 20 },
              'cedar': { base: 25, labor: 25 },
              'composite': { base: 35, labor: 30 },
              'pvc': { base: 40, labor: 30 }
          };
          
          const RAILING_COSTS = {
              'wood': { base: 30, labor: 15 },
              'aluminum': { base: 45, labor: 20 },
              'cable': { base: 60, labor: 25 },
              'glass': { base: 80, labor: 30 }
          };
          
          function calculateEstimate() {
              const length = parseFloat(document.getElementById('length').value) || 0;
              const width = parseFloat(document.getElementById('width').value) || 0;
              const deckType = document.getElementById('deckType').value;
              const railingType = document.getElementById('railingType').value;
              
              const squareFootage = length * width;
              const perimeterLength = 2 * (length + width);
              
              const materialCost = squareFootage * MATERIAL_COSTS[deckType].base +
                                 perimeterLength * RAILING_COSTS[railingType].base;
              
              const laborCost = squareFootage * MATERIAL_COSTS[deckType].labor +
                               perimeterLength * RAILING_COSTS[railingType].labor;
              
              document.getElementById('materialCost').textContent = `$${materialCost.toFixed(2)}`;
              document.getElementById('laborCost').textContent = `$${laborCost.toFixed(2)}`;
              document.getElementById('totalEstimate').textContent = 
                  `$${(materialCost + laborCost).toFixed(2)}`;
          }
          
          // Event listeners
          document.addEventListener('DOMContentLoaded', initializeViewer);
          
          window.addEventListener('resize', () => {
              camera.aspect = window.innerWidth / window.innerHeight;
              camera.updateProjectionMatrix();
              renderer.setSize(document.getElementById('3dViewer').clientWidth,
                             document.getElementById('3dViewer').clientHeight);
          });
          
          // Add input change listeners
          document.querySelectorAll('input, select').forEach(input => {
              input.addEventListener('change', updateDeckModel);
          });
          
          // Helper functions
          function showToast(message, type = 'success') {
              const toast = document.getElementById('toast');
              toast.textContent = message;
              toast.className = `toast ${type}`;
              toast.style.display = 'block';
              
              setTimeout(() => {
                  toast.style.display = 'none';
              }, 3000);
          }
          
          // AR and LiDAR functionality
          async function startARView() {
              try {
                  const session = await navigator.xr.requestSession('immersive-ar');
                  showToast('AR view started');
              } catch (error) {
                  showToast('AR not supported on this device', 'error');
              }
          }
          
          async function captureLiDAR() {
              if (!window.XRSystem) {
                  showToast('LiDAR scanning requires compatible hardware', 'error');
                  return;
              }
              
              try {
                  isLiDARScanning = true;
                  const session = await navigator.xr.requestSession('immersive-ar', {
                      requiredFeatures: ['depth-sensing']
                  });
                  showToast('LiDAR scanning started');
              } catch (error) {
                  showToast('Error starting LiDAR scan: ' + error.message, 'error');
              }
          }
      </script>
  </body>
  </html>
    <h1>This is heading 1</h1>
    <h2>This is heading 2</h2>
    <h3>This is heading 3</h3>
    <h4>This is heading 4</h4>
    <h5>This is heading 5</h5>
    <h6>This is heading 6</h6>
  </body>
</html> + TypeScript + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

While this project uses React, Vite supports many popular JS frameworks. [See all the supported frameworks](https://vitejs.dev/guide/#scaffolding-your-first-vite-project).

## Deploy Your Own

Deploy your own Vite project with Vercel.

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/vercel/vercel/tree/main/examples/vite-react&template=vite-react)

_Live Example: https://vite-react-example.vercel.app_

### Deploying From Your Terminal

You can deploy your new Vite project with a single command from your terminal using [Vercel CLI](https://vercel.com/download):

```shell
$ vercel
```
