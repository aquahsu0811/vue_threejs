<template>
  <canvas ref="experience" />
</template>

<script setup lang="ts">
import {
  PerspectiveCamera,
  Scene,
  WebGLRenderer,
  Mesh,
  MeshBasicMaterial,
  SphereGeometry,
} from "three";

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { onMounted, ref, computed, watch, render } from "vue";
import { useWindowSize } from '@vueuse/core';
import {useTweakPane} from '../utils/useTweakpane';

let renderer: WebGLRenderer;
let camera: PerspectiveCamera;
let controls: OrbitControls;

const experience = ref<HTMLCanvasElement | null>(null);
const scene = new Scene();

const { width, height } = useWindowSize();
const aspectRation = computed(()=> width.value / height.value);
function updateRenderer(){
	renderer.setSize(width.value, height.value);
	renderer.setPixelRatio(window.devicePixelRatio);
};

function updateCamera(){
	camera.aspect = aspectRation.value;
	camera.updateProjectionMatrix();
};

 watch(aspectRation, updateRenderer);
 watch(aspectRation, updateCamera);

camera = new PerspectiveCamera(
  45,
  aspectRation.value,
  0.1,
  1000
);

camera.position.z = 5;

scene.add(camera);

const sphere = new Mesh(
  new SphereGeometry(1, 20, 20),
  new MeshBasicMaterial({ color: 0x008080 })
);

scene.add(sphere);

const {pane, fpsGraph} = useTweakPane();

const loop = () =>{
	fpsGraph.begin();
	renderer.render(scene, camera);
	fpsGraph.end();
	controls.update();
	//sphere.position.y += 0.01;
	requestAnimationFrame(loop);
}
onMounted(() => {
	renderer = new WebGLRenderer({
		canvas: experience.value as unknown as HTMLCanvasElement,
    antialias: true,
  });
  updateRenderer();
	updateCamera();

	controls = new OrbitControls(camera, renderer.domElement);

	loop();
});
</script>