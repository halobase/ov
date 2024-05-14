<script>
  export let loading = false;
  export let error = "";
</script>

<div
  class="flex items-center justify-center w-full h-full absolute top-0 left-0 bg-black bg-opacity-80"
>
  {#if loading}
    <div class="text-2xl">
      加载中...
    </div>
  {:else if error}
    <strong>ERROR: </strong>
    {error}
  {/if}
</div>
