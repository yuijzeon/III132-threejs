<script setup lang="ts">
import { onMounted, ref, Ref } from "vue";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { DragControls } from "three/examples/jsm/controls/DragControls";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { RoomEnvironment } from "three/examples/jsm/environments/RoomEnvironment";
import { AnimationMixer, Clock, Color, Group, Mesh, PerspectiveCamera, PMREMGenerator, Scene, sRGBEncoding, Vector3, WebGLRenderer } from "three";

const container: Ref<HTMLCanvasElement | undefined> = ref();

onMounted(() => {
  let mixer: AnimationMixer;

  const clock = new Clock();

  const renderer = new WebGLRenderer({ canvas: container.value, antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.outputEncoding = sRGBEncoding;

  const pmremGenerator = new PMREMGenerator(renderer);

  const scene = new Scene();
  scene.background = new Color(0xbfe3dd);
  scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;

  const camera = new PerspectiveCamera(40, window.innerWidth / window.innerHeight, 1, 100);
  camera.position.set(5, 2, 8);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.target.set(0, 0.5, 0);
  controls.update();
  controls.enablePan = false;
  controls.enableDamping = true;

  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath('node_modules/three/examples/jsm/libs/draco/gltf/');

  const loader = new GLTFLoader();
  loader.setDRACOLoader(dracoLoader);
  loader.load('./public/model/scene.gltf', function (gltf) {
    const model: Group = gltf.scene;
    model.position.set(0, 0, 0);
    model.scale.set(0.5, 0.5, 0.5);
    scene.add(model);

    mixer = new AnimationMixer(model);
    mixer.clipAction(gltf.animations[0]).play();

    const dragControls = new DragControls(gltf.scene.children, camera, renderer.domElement);
    const modelBefore = new Vector3();
    // 開始拖曳
    dragControls.addEventListener('dragstart', function () {
      modelBefore.copy(model.position);
      controls.enabled = false;
    });
    // 拖曳過程
    dragControls.addEventListener('drag', function (event) {
      const object: Mesh = event.object;
      const offset = object.position.divide(new Vector3(2, 2, 2));
      model.position.copy(new Vector3().copy(modelBefore).add(offset));
      object.position.set(0, 0, 0);
    });
    // 拖曳結束
    dragControls.addEventListener('dragend', function () {
      modelBefore.set(0, 0, 0);
      controls.enabled = true;
    });

    animate();
  }, undefined, function (e) {
    console.error(e);
  });

  window.onresize = function () {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  };

  function animate() {
    requestAnimationFrame(animate);
    const delta = clock.getDelta();
    mixer.update(delta);
    controls.update();
    renderer.render(scene, camera);
  }
});
</script>

<template>
  <canvas ref="container"></canvas>
</template>

<style scoped>
canvas {
  width: 100%;
  height: 100%;
  position: fixed;
  left: 0;
  top: 0;
}
</style>
