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
  let full = false;

  let text = `横纵径：1.9*1.4cm
总糖含量：12.2%
维 C 含量：10.1%
酸度：0.87%
可溶性固形物含量：13.8%
花青素含量：161mg/100g`

  function render() {
    renderer.render(scene, camera);
  }

  onMount(() => {
    loader = new OBJLoader();
    scene = new three.Scene();
    scene.background = new three.Color(0x2e2e2e);
    camera = new three.PerspectiveCamera();

    renderer = new three.WebGLRenderer({ antialias: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(el.scrollWidth, el.scrollHeight);
    el.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.minDistance = 0.15;
    controls.maxDistance = 30;
    controls.addEventListener("change", render);

    window.addEventListener("resize", () => {
      camera.updateProjectionMatrix();
      renderer.setSize(el.scrollWidth, el.scrollHeight);
      renderer.render(scene, camera);
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
        renderer.render(scene, camera);
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
  <header class="flex gap-4 flex-col sm:w-72" class:hidden={full}>
    <div>
      <h2 class="text-2xl font-semibold mb-2">🌐 三维重建可视化</h2>
      <textarea
        class="w-full h-64 p-4 outline-none"
        placeholder="输入描述"
        bind:value={text}
      />
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
    <button
      class="btn"
      type="button"
      on:click={() => {
        full = true;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.render(scene, camera);
      }}
    >
      全屏
    </button>
  </header>
  <main class="relative grow rounded-md overflow-hidden" bind:this={el}>
    {#if loading}
      <Notify {loading} {error} />
    {/if}
    <div class="absolute top-4 right-4" class:hidden={!full}>
      <button
        class="btn btn-circle"
        type="button"
        on:click={() => {
          full = false;
        }}
      >
        返回
      </button>
    </div>
    <div class="absolute top-4 left-4 text-white">
      <pre class="text-2xl"><code>{text}</code></pre>
    </div>
  </main>
</div>
