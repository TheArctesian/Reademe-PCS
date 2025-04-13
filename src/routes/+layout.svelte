<script lang="ts">
  import "../app.css";
  import { dev } from "$app/environment";
  import { inject } from "@vercel/analytics";
  // inject({ mode: dev ? "development" : "production" });
  // import { injectSpeedInsights } from "@vercel/speed-insights/sveltekit";
  // injectSpeedInsights();
</script>

<div class="">
  <h1 class="font-bold text-3xl m-4 ">Reademe.md</h1>
  <hr class="rounded-sm border-2 border-[var(--bg)]">
</div>
<!-- <span class="font-normal italic float-right text-xl m-auto">By PCS Berkeley</span> -->
<slot />

<!-- <div class="wrapper">
  <div class="text">
    <h1 class="text-center">
      Email me at <span class="underline">me@danielokita.com</span>
    </h1>
    <h1 class="text-center">
      <h1>
        <div class="content">
          <div class="element">
            <a href="https://www.linkedin.com/in/stephen-okita/">
              <img
                src="https://cdn-icons-png.flaticon.com/512/3536/3536569.png"
              />
            </a>
          </div>
          <div class="element">
            <a href="https://github.com/TheArctesian">
              <img src="https://simpleicons.org/icons/github.svg" />
            </a>
          </div>
          <div class="element">
            <a href="https://www.instagram.com/stephen.okita/">
              <img src="https://simpleicons.org/icons/instagram.svg" />
            </a>
          </div>
        </div>
        as well as the source code for this website&nbsp<a
          class="s"
          href="https://github.com/TheArctesian/danielokita.com"
        >
          here</a
        >
      </h1>
    </h1>
  </div>
</div> -->

<style>
  :global(body) {
    background-color: var(--fg);
    font-family: "Fira Code", monospace;
    color: var(--bg)
  }

  
</style>
