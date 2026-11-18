\<\!DOCTYPE html\>  
\<html lang="en"\>  
\<head\>  
    \<meta charset="UTF-8"\>  
    \<meta name="viewport" content="width=device-width, initial-scale=1.0"\>  
    \<title\>Powers of Ten Scale Visualization\</title\>  
    \<script src="[https\://cdn.tailwindcss.com](https://cdn.tailwindcss.com)"\>\</script\>  
    \<script src="[https\://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js](https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js)"\>\</script\>  
    \<style\>  
        body { margin: 0; overflow: hidden; }  
        canvas { display: block; }  
    \</style\>  
\</head\>  
\<body\>  
    \<div id="info" class="absolute top-0 left-0 p-4 text-white z-10 select-none"\>  
        \<h1 id="scale-name" class="text-3xl font-bold"\>Human Scale\</h1\>  
        \<p id="scale-value" class="text-xl"\>Scale: \$10^0\$ meters (1 meter)\</p\>  
        \<p class="mt-2 text-sm"\>Use \*\*Mouse Wheel\*\* to Zoom (Change Scale)\</p\>  
    \</div\>

    \<script\>  
        // \--- SCENE SETUP \---  
        const scene \= new THREE.Scene();  
        const camera \= new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);  
        const renderer \= new THREE.WebGLRenderer({ antialias: true });  
        renderer.setSize(window.innerWidth, window.innerHeight);  
        document.body.appendChild(renderer.domElement);

        // Add ambient light  
        const ambientLight \= new THREE.AmbientLight(0x404040, 2); // soft white light  
        scene.add(ambientLight);

        // Add directional light  
        const directionalLight \= new THREE.DirectionalLight(0xffffff, 1);  
        directionalLight.position.set(5, 5, 5).normalize();  
        scene.add(directionalLight);

        // Camera position (constant in this type of visualization)  
        camera.position.z \= 5;

        // \--- STATE MANAGEMENT \---  
        // N in 10^N meters. Start at Human Scale (1 meter)  
        let currentExponent \= 0;  
        const MIN\_EXPONENT \= \-30; // Quantum  
        const MAX\_EXPONENT \= 25;  // Cosmic

        // DOM elements for display  
        const scaleNameElement \= document.getElementById('scale-name');  
        const scaleValueElement \= document.getElementById('scale-value');

        // \--- 3D OBJECTS AND SCALES \---  
        let currentObject \= null;

        /\*\*  
         \* Clears the current object from the scene and adds a new one based on the scale.  
         \* @param {number} exponent The current 10^N scale.  
         \*/  
        function updateVisualization(exponent) {  
            // Remove previous object  
            if (currentObject) {  
                scene.remove(currentObject);  
                currentObject.geometry.dispose();  
                currentObject.material.dispose();  
                currentObject \= null;  
            }

            let name \= "";  
            let geometry, material;

            // \--- SCALE DEFINITIONS (Simplified for visualization) \---  
            if (exponent \>= 20\) {  
                // Cosmic Scale (Galaxies, Clusters)  
                name \= "Cosmic Web / Galaxy Cluster";  
                geometry \= new THREE.SphereGeometry(2, 64, 64);  
                material \= new THREE.MeshPhongMaterial({ color: 0x4a008a, wireframe: true });  
            } else if (exponent \>= 10\) {  
                // Astronomical Scale (Planets, Stars)  
                name \= "Star / Solar System";  
                geometry \= new THREE.SphereGeometry(1.5, 32, 32);  
                material \= new THREE.MeshPhongMaterial({ color: 0xffa500 });  
            } else if (exponent \>= 3\) {  
                // Geographical Scale (Mountains, Cities)  
                name \= "Mountain / City";  
                geometry \= new THREE.BoxGeometry(1.5, 1.5, 1.5);  
                material \= new THREE.MeshPhongMaterial({ color: 0x228b22 });  
            } else if (exponent \>= 0\) {  
                // Human Scale  
                name \= "Human Scale";  
                geometry \= new THREE.CapsuleGeometry(0.5, 1.5, 4, 8);  
                material \= new THREE.MeshPhongMaterial({ color: 0x1e90ff });  
            } else if (exponent \>= \-6) {  
                // Microscopic Scale (Cells, Bacteria)  
                name \= "Cell / Bacteria";  
                geometry \= new THREE.OctahedronGeometry(1);  
                material \= new THREE.MeshPhongMaterial({ color: 0xff69b4, wireframe: false });  
            } else if (exponent \>= \-12) {  
                // Nanoscopic Scale (Molecules, DNA)  
                name \= "DNA Helix / Molecule";  
                geometry \= new THREE.TorusGeometry(1, 0.4, 16, 100);  
                material \= new THREE.MeshPhongMaterial({ color: 0x00ff00 });  
            } else {  
                // Quantum Scale (Atoms, Quarks)  
                name \= "Atom / Quantum Field";  
                geometry \= new THREE.IcosahedronGeometry(1);  
                material \= new THREE.MeshPhongMaterial({ color: 0xff0000, wireframe: true });  
            }

            // Create and add the new object  
            currentObject \= new THREE.Mesh(geometry, material);  
            scene.add(currentObject);

            // Update UI  
            scaleNameElement.textContent \= name;  
            // The exponent is a whole number for clarity in the visualization steps  
            scaleValueElement.innerHTML \= \`Scale: \$10^{\${exponent}}\$ meters (\${Math.pow(10, exponent).toPrecision(1)} meters)\`;  
        }

        // Initialize visualization  
        updateVisualization(currentExponent);

        // \--- INTERACTION \---

        /\*\*  
         \* Handles the mouse wheel event for "logarithmic zooming."  
         \* @param {WheelEvent} event The mouse wheel event.  
         \*/  
        function onDocumentMouseWheel(event) {  
            // Determine zoom direction: deltaY \< 0 is zoom in (smaller scale, more negative exponent)  
            const direction \= event.deltaY \< 0 ? \-1 : 1;  
             
            // Increment/decrement the exponent  
            currentExponent \+= direction;

            // Clamp the exponent to the defined range  
            currentExponent \= Math.max(MIN\_EXPONENT, Math.min(MAX\_EXPONENT, currentExponent));

            // Update the visualization for the new scale  
            updateVisualization(currentExponent);  
        }

        window.addEventListener('wheel', onDocumentMouseWheel, false);

        // \--- ANIMATION LOOP \---  
        function animate() {  
            requestAnimationFrame(animate);

            // Simple rotation for the central object to make it dynamic  
            if (currentObject) {  
                currentObject.rotation.x \+= 0.005;  
                currentObject.rotation.y \+= 0.005;  
            }

            renderer.render(scene, camera);  
        }

        // \--- RESIZE HANDLING \---  
        function onWindowResize() {  
            camera.aspect \= window.innerWidth / window.innerHeight;  
            camera.updateProjectionMatrix();  
            renderer.setSize(window.innerWidth, window.innerHeight);  
        }

        window.addEventListener('resize', onWindowResize, false);

        // Start the animation loop  
        animate();  
    \</script\>  
\</body\>  
\</html\>  
