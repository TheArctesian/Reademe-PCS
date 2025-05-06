<script lang="ts">
  import { formatDate } from "$lib/utils.js";
  export let data;
  import { fade, slide } from "svelte/transition";
  import { onMount, afterUpdate } from "svelte";

  // Use Svelte's reactive statements to track DOM updates
  let mounted = false;
  let pdfPreviewVisible = false;

  function togglePdfPreview() {
    pdfPreviewVisible = !pdfPreviewVisible;
  }

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
    <h1>Readme.md</h1>
    <div class="pdf-section">
      <div class="pdf-container">
        <div class="pdf-header">
          <div class="pdf-icon">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="20"
              height="20"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <path
                d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"
              ></path>
              <polyline points="14 2 14 8 20 8"></polyline>
              <line x1="16" y1="13" x2="8" y2="13"></line>
              <line x1="16" y1="17" x2="8" y2="17"></line>
              <polyline points="10 9 9 9 8 9"></polyline>
            </svg>
          </div>
          <span class="pdf-name">Reading_demo_pdf.pdf</span>
          <div class="pdf-actions">
            <button class="preview-button" on:click={togglePdfPreview}>
              {pdfPreviewVisible ? "Hide Preview" : "Show Preview"}
            </button>
            <a
              href="/other/Reading_demo_pdf.pdf"
              target="_blank"
              class="download-button"
            >
              Open PDF
            </a>
          </div>
        </div>

        {#if pdfPreviewVisible}
          <div class="pdf-preview" transition:fade={{ duration: 200 }}>
            <div class="pdf-preview-inner">
              <!-- PDF Preview - Using a mock image to represent the preview -->
              <div class="pdf-page">
                <div class="pdf-mockup">
                  <div class="pdf-title">The Reading Room</div>
                  <div class="pdf-subtitle">
                    A Collection of Essays & Articles
                  </div>
                  <div class="pdf-sample-text">
                    <div class="pdf-line"></div>
                    <div class="pdf-line pdf-line-short"></div>
                    <div class="pdf-line"></div>
                    <div class="pdf-line pdf-line-medium"></div>
                    <div class="pdf-line"></div>
                  </div>
                  <div class="pdf-sample-section">
                    <div class="pdf-section-title"></div>
                    <div class="pdf-line"></div>
                    <div class="pdf-line"></div>
                    <div class="pdf-line pdf-line-short"></div>
                  </div>
                </div>
              </div>
              <div class="pdf-preview-caption">
                This is a preview of the first page. Click "Open PDF" to view
                the full document.
              </div>
            </div>
          </div>
        {/if}
      </div>
    </div>
    <p class="intro-text">
      Welcome to our digital magazine collection. Browse through our curated
      articles below. Each card represents a unique perspective on technology,
      society, and culture.
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
    margin: 3rem 3rem 2rem;
  }

  h1 {
    font-size: 2.4rem;
    margin-bottom: 1.5rem;
    font-weight: 700;
    color: #333;
  }

  /* PDF Section */
  .pdf-section {
    margin: 1.5rem 0;
  }

  .pdf-container {
    background-color: #f9f9f9;
    border-radius: 8px;
    border: 1px solid #e0e0e0;
    overflow: hidden;
  }

  .pdf-header {
    display: flex;
    align-items: center;
    padding: 1rem;
    background-color: #f5f5f5;
    border-bottom: 1px solid #e0e0e0;
  }

  .pdf-icon {
    margin-right: 10px;
    color: #e53935;
    flex-shrink: 0;
  }

  .pdf-name {
    font-weight: 500;
    color: #333;
    flex-grow: 1;
  }

  .pdf-actions {
    display: flex;
    gap: 10px;
  }

  .preview-button,
  .download-button {
    padding: 6px 12px;
    border-radius: 4px;
    font-size: 0.85rem;
    cursor: pointer;
    text-decoration: none;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s ease;
  }

  .preview-button {
    background-color: #f0f0f0;
    border: 1px solid #ddd;
    color: #333;
  }

  .preview-button:hover {
    background-color: #e0e0e0;
  }

  .download-button {
    background-color: #0366d6;
    border: 1px solid #0366d6;
    color: white;
  }

  .download-button:hover {
    background-color: #0256b9;
  }

  /* PDF Preview */
  .pdf-preview {
    padding: 1rem;
    background-color: #f9f9f9;
  }

  .pdf-preview-inner {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .pdf-page {
    width: 100%;
    max-width: 500px;
    aspect-ratio: 0.7;
    background-color: white;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    margin-bottom: 1rem;
    padding: 2rem;
    box-sizing: border-box;
    overflow: hidden;
  }

  .pdf-mockup {
    height: 100%;
    display: flex;
    flex-direction: column;
  }

  .pdf-title {
    font-size: 1.5rem;
    font-weight: 700;
    text-align: center;
    margin-bottom: 0.5rem;
    color: #333;
  }

  .pdf-subtitle {
    font-size: 1rem;
    text-align: center;
    margin-bottom: 2rem;
    color: #666;
  }

  .pdf-sample-text {
    margin-bottom: 2rem;
  }

  .pdf-line {
    height: 10px;
    background-color: #f0f0f0;
    margin-bottom: 10px;
    border-radius: 2px;
  }

  .pdf-line-short {
    width: 70%;
  }

  .pdf-line-medium {
    width: 85%;
  }

  .pdf-sample-section {
    margin-top: auto;
  }

  .pdf-section-title {
    height: 15px;
    width: 40%;
    background-color: #e0e0e0;
    margin-bottom: 15px;
    border-radius: 2px;
  }

  .pdf-preview-caption {
    font-size: 0.85rem;
    color: #666;
    text-align: center;
  }

  .intro-text {
    font-size: 1.1rem;
    margin: 1.5rem 0;
    color: #444;
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

    .pdf-actions {
      flex-direction: column;
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

    .pdf-header {
      flex-direction: column;
      align-items: flex-start;
    }

    .pdf-name {
      margin-bottom: 0.5rem;
    }

    .pdf-actions {
      width: 100%;
    }
  }
</style>
