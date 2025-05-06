<script lang="ts">
  import { formatDate } from "$lib/utils.js";
  export let data;
  import { fade, slide } from "svelte/transition";
  import { onMount, afterUpdate } from "svelte";

  // Use Svelte's reactive statements to track DOM updates
  let mounted = false;

  // Track mouse position for hover effects
  function handleMouseMove(event, element) {
    if (!element) return;

    const rect = element.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;

    // Calculate rotation based on mouse position
    // Limit the rotation to a subtle amount
    const rotateY = (x / rect.width - 0.5) * 10; // -5 to 5 degrees
    const rotateX = (y / rect.height - 0.5) * -10; // -5 to 5 degrees

    // Calculate position as percentage for shine effect
    const xPercent = (x / rect.width) * 100;
    const yPercent = (y / rect.height) * 100;

    // Apply transforms
    element.style.setProperty("--x-pos", `${xPercent}%`);
    element.style.setProperty("--y-pos", `${yPercent}%`);
    element.style.transform = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) scale(1.03)`;
  }

  function handleMouseLeave(element) {
    if (!element) return;
    element.style.transform =
      "perspective(1000px) rotateX(0) rotateY(0) scale(1)";
  }

  function setupEventListeners() {
    // Use a slight delay to ensure DOM is ready
    setTimeout(() => {
      const articles = document.querySelectorAll(".post");

      articles.forEach((article) => {
        // Remove previous event listeners if any (to prevent duplicates)
        article.removeEventListener("mousemove", article._moveHandler);
        article.removeEventListener("mouseleave", article._leaveHandler);

        // Create bound event handlers and store references for cleanup
        article._moveHandler = (e) => handleMouseMove(e, article);
        article._leaveHandler = () => handleMouseLeave(article);

        // Add new event listeners
        article.addEventListener("mousemove", article._moveHandler);
        article.addEventListener("mouseleave", article._leaveHandler);
      });
    }, 100);
  }

  onMount(() => {
    mounted = true;
    setupEventListeners();

    // Clean up on component destruction
    return () => {
      const articles = document.querySelectorAll(".post");
      articles.forEach((article) => {
        if (article._moveHandler)
          article.removeEventListener("mousemove", article._moveHandler);
        if (article._leaveHandler)
          article.removeEventListener("mouseleave", article._leaveHandler);
      });
    };
  });

  // Re-setup listeners when data changes
  afterUpdate(() => {
    if (mounted) {
      setupEventListeners();
    }
  });
</script>

<div out:slide>
  <section class="header-section">
    <div class="magazine-cover">
      <a href="/other/readme_v1.pdf" target="_blank" class="cover-link">
        <img
          src="/images/readmecover.png"
          alt="Readme: Volume 1: State of Information"
          class="cover-image"
        />
        <div class="cover-overlay">
          <span class="cover-text">Click to open PDF</span>
        </div>
      </a>
    </div>

    <p class="intro-text">
      <a href="https://pcs.studentorg.berkeley.edu/"
        >Political Computer Science
      </a> - Readme.md
    </p>
  </section>

  <section>
    <div class="section-header">
      <h2># Articles</h2>
    </div>
    <div class="posts">
      {#each data.posts as post, i}
        <a
          class="post"
          href={post.slug}
          in:fade={{ delay: i * 150, duration: 300 }}
        >
          <div class="shine"></div>
          <div class="content-wrapper">
            <div class="post-header">
              <div class="post-title">{post.title}</div>
              <div class="post-meta">
                <span class="date">{formatDate(post.date)}</span>
                <div class="author">
                  @{post.author.toLowerCase().replace(/\s+/g, "-")}
                </div>
              </div>
            </div>
            <div class="post-content">
              {post.description}
            </div>
            <div class="post-tags">
              {#if post.categories}
                {#each post.categories as category}
                  <span class="tag"
                    >#{category.toLowerCase().replace(/\s+/g, "-")}</span
                  >
                {/each}
              {/if}
            </div>
          </div>
        </a>
      {/each}
    </div>
  </section>
</div>

<style>
  /* Basic styling */
  :global(body) {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
    line-height: 1.6;
  }

  /* Header section */
  .header-section {
    margin: 3rem;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  h1 {
    font-size: 2.4rem;
    margin-bottom: 1.5rem;
    font-weight: 700;
    color: #333;
    align-self: flex-start;
  }

  /* Magazine Cover */
  .magazine-cover {
    position: relative;
    max-width: 300px;
    margin: 1rem 0 2rem;
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
    transition:
      transform 0.3s ease,
      box-shadow 0.3s ease;
    border-radius: 4px;
    overflow: hidden;
  }

  .magazine-cover:hover {
    transform: translateY(-5px) scale(1.02);
    box-shadow: 0 12px 30px rgba(0, 0, 0, 0.2);
  }

  .cover-link {
    display: block;
    text-decoration: none;
    color: inherit;
  }

  .cover-image {
    width: 100%;
    height: auto;
    display: block;
  }

  .cover-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.6);
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.3s ease;
  }

  .magazine-cover:hover .cover-overlay {
    opacity: 1;
  }

  .cover-text {
    color: white;
    font-weight: 500;
    padding: 0.5rem 1rem;
    border: 2px solid white;
    border-radius: 4px;
  }

  .intro-text {
    font-size: 1.1rem;
    margin: 1.5rem 0;
    color: #444;
    text-align: center;
    max-width: 700px;
  }

  /* Section styling */
  .section-header {
    margin: 0 3rem;
  }

  h2 {
    font-size: 1.8rem;
    margin-bottom: 1.5rem;
    font-weight: 600;
    border-bottom: 1px solid #ddd;
    padding-bottom: 0.5rem;
    color: #333;
  }

  /* Posts styling */
  .posts {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
    padding: 0 3rem 3rem;
  }

  .post {
    padding: 1.5rem;
    height: 100%;
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    position: relative;
    overflow: hidden;
    transition:
      transform 0.2s ease,
      box-shadow 0.3s ease;
    transform: perspective(1000px) rotateX(0) rotateY(0);
    transform-style: preserve-3d;
    will-change: transform, box-shadow;
    z-index: 1;
    border-radius: 8px;
    text-decoration: none;
  }

  .content-wrapper {
    position: relative;
    z-index: 2;
    display: flex;
    flex-direction: column;
    height: 100%;
  }

  .shine {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: radial-gradient(
      circle at var(--x-pos, 50%) var(--y-pos, 50%),
      rgba(255, 255, 255, 0.3) 0%,
      transparent 60%
    );
    opacity: 0;
    transition: opacity 0.2s ease;
    pointer-events: none;
    z-index: 1;
  }

  .post:hover .shine {
    opacity: 1;
  }

  .post:nth-child(even) {
    background-color: #f5f5f5;
    color: #333;
    border: 1px solid #ddd;
  }

  .post:nth-child(even):hover {
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  }

  .post:nth-child(odd) {
    background-color: #fff;
    color: #333;
    border: 1px solid #ddd;
  }

  .post:nth-child(odd):hover {
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  }

  .post-header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 1rem;
  }

  .post-title {
    font-weight: bold;
    transform: translateZ(20px);
    transition: transform 0.2s ease;
    color: #333;
    font-size: 1.1rem;
  }

  .post:hover .post-title {
    transform: translateZ(40px);
  }

  .post-meta {
    text-align: right;
    transform: translateZ(10px);
    transition: transform 0.2s ease;
    font-size: 0.85rem;
    color: #666;
  }

  .date {
    display: block;
    margin-bottom: 0.3rem;
  }

  .author {
    color: #666;
  }

  .post:hover .post-meta {
    transform: translateZ(25px);
  }

  .post-content {
    font-size: 0.95rem;
    flex: 1;
    transform: translateZ(5px);
    transition: transform 0.2s ease;
    line-height: 1.6;
    color: #555;
    margin-bottom: 1rem;
  }

  .post:hover .post-content {
    transform: translateZ(15px);
  }

  .post-tags {
    margin-top: auto;
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .tag {
    font-size: 0.75rem;
    padding: 0.2rem 0.5rem;
    background-color: #f0f0f0;
    border-radius: 3px;
    color: #666;
  }

  @media (max-width: 1200px) {
    .posts {
      grid-template-columns: repeat(3, 1fr);
    }
  }

  @media (max-width: 900px) {
    .posts {
      grid-template-columns: repeat(2, 1fr);
    }

    .header-section,
    .section-header,
    .posts {
      padding-left: 2rem;
      padding-right: 2rem;
      margin-left: 1rem;
      margin-right: 1rem;
    }
  }

  @media (max-width: 600px) {
    .posts {
      grid-template-columns: 1fr;
    }

    .header-section,
    .section-header,
    .posts {
      padding-left: 1rem;
      padding-right: 1rem;
      margin-left: 0.5rem;
      margin-right: 0.5rem;
    }

    .magazine-cover {
      max-width: 240px;
    }
  }
</style>
