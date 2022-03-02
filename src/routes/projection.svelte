<script lang="ts">
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	// WARNING: .js file extension is necessary here for ssr to work
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
	import WireframeObject from '../scripts/wireframeobject';

	import cell8Data from '../assets/wireframes/cell8.json';

	let canvas: HTMLCanvasElement;
	// let height: number, width: number;
	let canvasParent: HTMLElement;
	let animationFrame: number;

	let scene: THREE.Scene, camera: THREE.PerspectiveCamera, renderer: THREE.WebGLRenderer;
	let cube: WireframeObject;

	onMount(() => {
		init();
		// Add resize event listener to the window
		window.addEventListener('resize', resize, false);
		startAnimating(90);

		return destroy;
	});

	function init() {
		// Set canvas width and height
		const sceneWidth = canvasParent.clientWidth;
		const sceneHeight = canvasParent.clientHeight;

		////////// Inital Setup //////////
		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(75, sceneWidth / sceneHeight, 0.1, 1000);
		renderer = new THREE.WebGLRenderer({
			canvas: canvas,
			antialias: true,
			alpha: true
		});
		renderer.setSize(sceneWidth, sceneHeight);
		renderer.sortObjects = false;

		// scene.background = new THREE.Color(0xf8f8ff);
		// scene.background = new THREE.Color( 0xffdf06 );
		scene.background = new THREE.Color(0xf9fafb);
		renderer.shadowMap.type = THREE.PCFSoftShadowMap;
		renderer.shadowMap.enabled = true;
		renderer.setSize(sceneWidth, sceneHeight);

		const controls = new OrbitControls(camera, renderer.domElement);

		////////// Scene Setup //////////
		// Camera
		camera.position.set(0, 2, -5);
		camera.lookAt(new THREE.Vector3(0, 0, 0));
		controls.update(); // OrbitControls must be updated after changes to camera position/rotation

		// Objects
		const floorGeometry = new THREE.PlaneGeometry(22, 22);
		const floorMaterial = new THREE.ShadowMaterial({ opacity: 0.3 });
		const floor = new THREE.Mesh(floorGeometry, floorMaterial);
		floor.name = 'floor';
		floor.position.y = -4;
		floor.rotateX(-Math.PI / 2);
		floor.receiveShadow = true;

		scene.add(floor);

		// Lighting
		const ambientLight = new THREE.AmbientLight(0xc4c4c4, 0.7);
		scene.add(ambientLight);

		const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
		directionalLight.castShadow = true;
		directionalLight.shadow.mapSize.width = 2048;
		directionalLight.shadow.mapSize.height = 2048;
		directionalLight.position.y = 10;
		scene.add(directionalLight);

		cube = new WireframeObject(scene, cell8Data);
		cube.updateMeshes();
	}

	let fpsInterval: number, now: number, then: number, elapsed: number;

	// initialize the timer variables and start the animation

	function startAnimating(fps: number) {
		fpsInterval = 1000 / fps;
		then = Date.now();
		animate();
	}

	function animate() {
		// request another frame
		animationFrame = requestAnimationFrame(animate);

		// calc elapsed time since last loop
		now = Date.now();
		elapsed = now - then;

		// if enough time has elapsed, draw the next frame
		if (elapsed > fpsInterval) {
			// Get ready for next frame by setting then=now, but also adjust for your
			// specified fpsInterval not being a multiple of RAF's interval (16.7ms)
			then = now - (elapsed % fpsInterval);

			// Put your drawing code here
			cube.rotate({ xw: 0.01 });
			cube.projectTo3D();
			cube.updateMeshes();
			renderer.render(scene, camera);
		}
	}

	function resize() {
		const sceneWidth = canvasParent.clientWidth;
		const sceneHeight = canvasParent.clientHeight;
		renderer.setSize(sceneWidth, sceneHeight);
		camera.aspect = sceneWidth / sceneHeight;
		camera.updateProjectionMatrix();
	}

	function destroy() {
		cancelAnimationFrame(animationFrame);
		window.removeEventListener('resize', resize, false);

		renderer.dispose();
	}
</script>

<div class="flex h-screen bg-gray-100 p-2">
	<div class="h-full w-8/12 p-2">
		<div class="h-full w-full bg-gray-50 shadow" bind:this={canvasParent}>
			<canvas width="100" height="100" bind:this={canvas} />
		</div>
	</div>
	<div class="h-full w-4/12 p-2">
		<div class="h-full w-full bg-gray-50 shadow">
			<a href="/">Home</a>
		</div>
	</div>
</div>
