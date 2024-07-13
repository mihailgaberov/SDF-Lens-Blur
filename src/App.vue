<template>
  <main>
    
  </main>
</template>

<script setup>
import * as THREE from 'three';
import fragmentShader from '@/shaders/fragment.glsl';

// extract "variation" parameter from the url
const urlParams = new URLSearchParams(window.location.search);
const variation = urlParams.get('var') || 0;

// Scene setup
const scene = new THREE.Scene();
const vMouse = new THREE.Vector2();
const vMouseDamp = new THREE.Vector2();
const vResolution = new THREE.Vector2();

// Viewport setup (updated on resize)
let w, h = 1;

// Orthographic camera setup
const aspect = w / h;
const camera = new THREE.OrthographicCamera(-aspect, aspect, 1, -1, 0.1, 1000);

const renderer = new THREE.WebGLRenderer();
document.body.appendChild(renderer.domElement);

const onPointerMove = (e) => { vMouse.set(e.pageX, e.pageY) }
document.addEventListener('mousemove', onPointerMove);
document.addEventListener('pointermove', onPointerMove);
document.body.addEventListener('touchmove', function (e) { e.preventDefault(); }, { passive: false });

// Plane geometry covering the full viewport
const geo = new THREE.PlaneGeometry(1, 1);  // Scaled to cover full viewport

// Shader material creation
const mat = new THREE.ShaderMaterial({
  vertexShader: /* glsl */`
    varying vec2 v_texcoord;
    void main() {
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        v_texcoord = uv;
    }`,
  fragmentShader, // most of the action happening in the fragment
  uniforms: {
    u_mouse: { value: vMouseDamp },
    u_resolution: { value: vResolution },
    u_pixelRatio: { value: 2 }
  },
  defines: {
    VAR: variation
  }
});


// Mesh creation
const quad = new THREE.Mesh(geo, mat);
scene.add(quad);

// Camera position and orientation
camera.position.z = 1;  // Set appropriately for orthographic

// Animation loop to render
let time, lastTime = 0;
const update = () => {
  // calculate delta time
  time = performance.now() * 0.001;
  const dt = time - lastTime;
  lastTime = time;

  // ease mouse motion with damping
  for (const k in vMouse) {
    if (k == 'x' || k == 'y') vMouseDamp[k] = THREE.MathUtils.damp(vMouseDamp[k], vMouse[k], 8, dt);
  }

  // render scene
  requestAnimationFrame(update);
  renderer.render(scene, camera);
};
update();

const resize = () => {
  w = window.innerWidth;
  h = window.innerHeight;

  const dpr = Math.min(window.devicePixelRatio, 2);

  renderer.setSize(w, h);
  renderer.setPixelRatio(dpr);

  camera.left = -w / 2;
  camera.right = w / 2;
  camera.top = h / 2;
  camera.bottom = -h / 2;
  camera.updateProjectionMatrix();

  quad.scale.set(w, h, 1);
  vResolution.set(w, h).multiplyScalar(dpr);
  mat.uniforms.u_pixelRatio.value = dpr;
};
resize();

window.addEventListener('resize', resize)
</script>


<style scoped>
*,
*::after,
*::before {
	box-sizing: border-box;
}

:root {
	font-size: 12px;
	--color-text: #ffffff;
	--color-bg: #fff;
	--color-link: #898989;
	--color-link-hover: #fff;
	--page-padding: 1.5rem;
}

body {
	margin: 0;
	color: var(--color-text);
	background-color: var(--color-bg);
	font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
}

.js canvas {
	position: fixed;
	top: 0;
	left: 0;
}

/* Page Loader */
.js .loading::before,
.js .loading::after {
	content: '';
	position: fixed;
	z-index: 1000;
}

.js .loading::before {
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background: var(--color-bg);
}

.js .loading::after {
	top: 50%;
	left: 50%;
	width: 60px;
	height: 60px;
	margin: -30px 0 0 -30px;
	border-radius: 50%;
	opacity: 0.4;
	background: var(--color-link);
	animation: loaderAnim 0.7s linear infinite alternate forwards;

}

@keyframes loaderAnim {
	to {
		opacity: 1;
		transform: scale3d(0.5,0.5,1);
	}
}

a {
	text-decoration: none;
	color: var(--color-link);
	outline: none;
	cursor: pointer;
}

a:hover {
	text-decoration: underline;
	color: var(--color-link-hover);
	outline: none;
}

/* Better focus styles from https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible */
a:focus {
	/* Provide a fallback style for browsers
	 that don't support :focus-visible */
	outline: none;
	background: lightgrey;
}

a:focus:not(:focus-visible) {
	/* Remove the focus indicator on mouse-focus for browsers
	 that do support :focus-visible */
	background: transparent;
}

a:focus-visible {
	/* Draw a very noticeable focus style for
	 keyboard-focus on browsers that do support
	 :focus-visible */
	outline: 2px solid red;
	background: transparent;
}

.unbutton {
	background: none;
	border: 0;
	padding: 0;
	margin: 0;
	font: inherit;
	cursor: pointer;
}

.unbutton:focus {
	outline: none;
}

.frame {
	text-transform: uppercase;
	padding: var(--page-padding);
	position: relative;
	display: grid;
	z-index: 1000;
	width: 100%;
	height: 100%;
	grid-row-gap: 1rem;
	grid-column-gap: 2rem;
	pointer-events: none;
	justify-items: start;
	grid-template-columns: auto auto;
	grid-template-areas: 'title' 'back' 'archive' 'github' 'sponsor' 'demos' 'tags';
}

.frame #cdawrap {
  justify-self: start;
}

.frame a {
  pointer-events: auto;
}

.frame__title {
  grid-area: title;
  font-size: inherit;
  margin: 0;
  color: white;
}

.frame__back {
  grid-area: back;
  justify-self: start;
}

.frame__archive {
  grid-area: archive;
  justify-self: start;
}

.frame__sub {
  grid-area: sub;
}

.frame__github {
  grid-area: github;
}

.frame__tags {
  grid-area: tags;
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.frame__hire {
  grid-area: hire;
}

.frame__demos {
	grid-area: demos;
	display: flex;
	gap: 1rem;
	margin-right: 1em;
}

.frame__demos span {
	opacity: 0.8;

}

.frame__demos a.selected {
  font-weight: bold;
  color: var(--color-link-hover);
}

.content {
	padding: var(--page-padding);
	display: flex;
	flex-direction: column;
	width: 100vw;
	position: relative;
}

@media screen and (min-width: 53em) {
	body {
		--page-padding: 2rem 3rem;
	}
	.frame {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		grid-template-columns: auto auto auto auto 1fr;
		grid-template-rows: auto auto;
		align-content: space-between;
		grid-template-areas: 'title back github archive sponsor' 'tags tags demos demos demos';
	}
	.frame #cdawrap, 
	.frame__demos {
		justify-self: end;
	}
	.content {
		min-height: 100vh;
		justify-content: center;
		align-items: center;
	}
}
</style>
