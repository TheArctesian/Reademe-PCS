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
    const rotateY = ((x / rect.width) - 0.5) * 10; // -5 to 5 degrees
    const rotateX = ((y / rect.height) - 0.5) * -10; // -5 to 5 degrees
    
    // Calculate position as percentage for shine effect
    const xPercent = (x / rect.width) * 100;
    const yPercent = (y / rect.height) * 100;
    
    // Apply transforms
    element.style.setProperty('--x-pos', `${xPercent}%`);
    element.style.setProperty('--y-pos', `${yPercent}%`);
    element.style.transform = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) scale(1.03)`;
  }
  
  function handleMouseLeave(element) {
    if (!element) return;
    element.style.transform = 'perspective(1000px) rotateX(0) rotateY(0) scale(1)';
  }
  
  function setupEventListeners() {
    // Use a slight delay to ensure DOM is ready
    setTimeout(() => {
      const articles = document.querySelectorAll('.post');
      
      articles.forEach(article => {
        // Remove previous event listeners if any (to prevent duplicates)
        article.removeEventListener('mousemove', article._moveHandler);
        article.removeEventListener('mouseleave', article._leaveHandler);
        
        // Create bound event handlers and store references for cleanup
        article._moveHandler = (e) => handleMouseMove(e, article);
        article._leaveHandler = () => handleMouseLeave(article);
        
        // Add new event listeners
        article.addEventListener('mousemove', article._moveHandler);
        article.addEventListener('mouseleave', article._leaveHandler);
      });
    }, 100);
  }
  
  onMount(() => {
    mounted = true;
    setupEventListeners();
    
    // Clean up on component destruction
    return () => {
      const articles = document.querySelectorAll('.post');
      articles.forEach(article => {
        if (article._moveHandler) article.removeEventListener('mousemove', article._moveHandler);
        if (article._leaveHandler) article.removeEventListener('mouseleave', article._leaveHandler);
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

  </div>
  <section>
    <div class="posts m-4">
      {#each data.posts as post, i}
        <a class="post" href={post.slug} in:fade={{ delay: i * 150, duration: 300 }}>
          <div class="shine"></div>
          <div class="content-wrapper">
            <div class="post-header">
              <div class="post-title">{post.title}</div>
              <div class="post-meta">
                {formatDate(post.date)}
                <div>{post.author}</div>
              </div>
            </div>
            <div class="post-content">
              {post.description}
            </div>
          </div>
        </a>
      {/each}
    </div>
  </section>
</div>

<style>
  .posts {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
  }
  
  .post {
    padding: 1rem;
    height: 100%;
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    position: relative;
    overflow: hidden;
    transition: transform 0.2s ease, box-shadow 0.3s ease;
    transform: perspective(1000px) rotateX(0) rotateY(0);
    transform-style: preserve-3d;
    will-change: transform, box-shadow;
    z-index: 1;
    border-radius: 8px;
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
    background-color: var(--fg);
    color: var(--bg);
    border: 1px solid var(--bg);
  }
  
  .post:nth-child(even):hover {
    box-shadow: 
      0 10px 25px rgba(0,0,0,0.2),
      0 0 15px rgba(255,255,255,0.1);
  }
  
  .post:nth-child(odd) {
    background-color: var(--bg);
    color: var(--fg);
    border: 1px solid var(--fg);
  }
  
  .post:nth-child(odd):hover {
    box-shadow: 
      0 10px 25px rgba(0,0,0,0.1),
      0 0 15px rgba(0,0,0,0.05);
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
  }
  
  .post:hover .post-title {
    transform: translateZ(40px);
  }
  
  .post-meta {
    text-align: right;
    transform: translateZ(10px);
    transition: transform 0.2s ease;
  }
  
  .post:hover .post-meta {
    transform: translateZ(25px);
  }
  
  .post-content {
    font-size: 0.9rem;
    flex: 1;
    transform: translateZ(5px);
    transition: transform 0.2s ease;
  }
  
  .post:hover .post-content {
    transform: translateZ(15px);
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
  }
  
  @media (max-width: 600px) {
    .posts {
      grid-template-columns: 1fr;
    }
  }
</style>