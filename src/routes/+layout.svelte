<script lang="ts">
  import "../app.css";
  import { dev } from "$app/environment";
  import { inject } from "@vercel/analytics";
  // inject({ mode: dev ? "development" : "production" });
  // import { injectSpeedInsights } from "@vercel/speed-insights/sveltekit";
  // injectSpeedInsights();
</script>

<div class="">
  <h1 class="font-bold text-3xl m-4">Readme.md</h1>
  <hr class="rounded-sm border-2 border-[var(--bg)]" />
</div>
<!-- <span class="font-normal italic float-right text-xl m-auto">By PCS Berkeley</span> -->
<slot />

<style>
  :global(body) {
    background-color: var(--fg);
    font-family: "Fira Code", monospace;
    color: var(--bg);
  }
</style>
