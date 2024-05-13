<script lang="ts">
  import { onMount } from "svelte";
  import * as three from "three";
  import { OBJLoader } from "three/addons/loaders/OBJLoader.js";
  import { OrbitControls } from "three/addons/controls/OrbitControls.js";
  import Notify from "./Notify.svelte";

  let el: HTMLElement;
  let url: string;
  let scene: three.Scene;
  let camera: three.PerspectiveCamera;
  let renderer: three.WebGLRenderer;
  let loader: OBJLoader;
  let loading = false;
  let error = "";

  function render() {
    renderer.render(scene, camera);
  }

  onMount(() => {
    loader = new OBJLoader();
    scene = new three.Scene();
    scene.background = new three.Color(0x03adfc);
    camera = new three.PerspectiveCamera();

    renderer = new three.WebGLRenderer({ antialias: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(el.scrollWidth, el.scrollHeight);
    el.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.minDistance = 0.15;
    controls.maxDistance = 5;
    controls.addEventListener("change", render);

    window.addEventListener("resize", () => {
      camera.updateProjectionMatrix();
      renderer.setSize(el.scrollWidth, el.scrollHeight);
    });
  });

  $: if (url) {
    scene.clear();

    const ambientLight = new three.AmbientLight();
    scene.add(ambientLight);

    const pointLight = new three.PointLight();
    camera.add(pointLight);
    scene.add(camera);

    loader.load(
      url,
      function (obj) {
        loading = false;
        scene.add(obj);
        requestAnimationFrame(render);
      },
      (event) => {},
      (err) => (error = (err as Error).message),
    );
  }

  function __change(e: Event) {
    const reader = new FileReader();
    reader.addEventListener("load", () => {
      url = reader.result as string;
    });
    reader.readAsDataURL(e.target?.files[0]);
    loading = true;
  }
</script>

<svelte:head>
  <title>三维重建可视化</title>
</svelte:head>

<div class="flex flex-col sm:flex-row gap-4 p-4 h-screen">
  <header class="flex gap-4 flex-col sm:w-64">
    <div>
      <h2 class="text-2xl font-semibold mb-2">🌐 三维重建可视化</h2>
      <p>
        Lorem ipsum dolor sit, amet consectetur adipisicing elit. Sequi
        consequuntur voluptas reiciendis placeat error possimus fugiat aut
        adipisci, odit commodi iure debitis iste distinctio aliquid temporibus,
        nihil minima corporis enim.
      </p>
    </div>
    <label class="btn btn-info">
      <input
        class="hidden"
        type="file"
        accept="application/object"
        on:change={__change}
      />
      选择 .obj 文件
    </label>
  </header>
  <main class="relative grow rounded-md overflow-hidden" bind:this={el}>
    {#if loading}
      <Notify {loading} {error} />
    {/if}
  </main>
</div>
