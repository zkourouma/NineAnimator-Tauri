<script lang="ts">
  // Import required packages
  import { fade } from "svelte/transition";
  import EpisodeHolder from "$lib/components/player/EpisodeHolder.svelte";
  import { settings } from "$lib/model/settings";
  import { library } from "$lib/model/library";
  import Button from "$lib/components/public/Button.svelte";
  import type { PageData } from "./$types";
  import Loading from "$lib/components/public/Loading.svelte";
  import ExternalLink from "$lib/components/public/ExternalLink.svelte";
  import ExternalIcon from "$lib/components/assets/ExternalIcon.svelte";

  export let data: PageData;

  // Page scroll
  let y = 0;

  $: title = "";

  (async () => {
    const mediaTitle = (await data.anime.data).title;
    title = mediaTitle.english ?? mediaTitle.romaji;
  })();

  $: scale = y < 345 ? 0.005 * Math.abs(y) + 1 : 0.005 * 345 + 1;
  $: blur = y < 345 ? 0.05 * Math.abs(y) : 0.05 * 345;
  $: opacity = y < 345 ? -Math.abs(y) / 345 + 1 : 0;
</script>

<svelte:head>
  <title>
    {title}
  </title>
</svelte:head>

<svelte:window bind:scrollY={y} />

{#await data.anime.data}
  <Loading />
{:then anime}
  <section in:fade>
    {#if anime.bannerImage || anime.coverImage?.large}
      <div class="banner">
        <img
          src={anime.bannerImage ?? anime.coverImage.large}
          alt={anime.title.romaji}
          in:fade
          class:reduce-motion={$settings.reduceMotion}
          style:--banner-scale={scale}
          style:--banner-blur={`${anime.bannerImage ? blur : blur + 5}px`}
          style:--banner-opacity={opacity}
        />
      </div>
    {/if}
    <div in:fade class="text" style:--color={anime.color}>
      <div class="container">
        <div
          class="important-info"
          class:no-overlap={!(anime.bannerImage && anime.coverImage?.large)}
        >
          <div
            class="image-and-choices"
            class:overlap={anime.bannerImage || anime.coverImage?.large}
          >
            <img
              src={anime.coverImage?.large ?? "/assets/loading_failure.jpeg"}
              alt={anime.title?.english ?? anime.title?.romaji}
              in:fade
              class="thumbnail"
            />
            <Button
              size="all"
              type={$library.subscriptions.some(
                ({ media }) => media.id === data.id
              )
                ? "danger"
                : "default"}
              on:click={() => {
                $library.subscriptions.some(({ media }) => media.id === data.id)
                  ? ($library.subscriptions = [
                      ...$library.subscriptions.filter(
                        ({ media }) => media.id !== data.id
                      ),
                    ])
                  : ($library.subscriptions = [
                      ...$library.subscriptions,
                      {
                        media: anime,
                        source: data.source,
                      },
                    ]);
              }}
            >
              {$library.subscriptions.some(({ media }) => media.id === data.id)
                ? "Unsubscribe"
                : "Subscribe"}
            </Button>
          </div>
          <div>
            <h1 class="title">
              {$settings.animeLanguage === "english"
                ? anime.title.english ?? anime.title.romaji
                : anime.title.native ?? anime.title.romaji}
            </h1>
            <p in:fade class="description">
              {@html anime.description ?? ""}
            </p>
            {#if anime.siteUrl}
              <ExternalLink href={anime.siteUrl}>
                <Button>
                  Share
                  <ExternalIcon />
                </Button>
              </ExternalLink>
            {/if}
          </div>
        </div>
      </div>
    </div>
    <EpisodeHolder
      episodes={anime.streamingEpisodes}
      hoverAll={$settings.showProgress}
    />
  </section>
{:catch e}
  <p class="error">{e}</p>
{/await}

<style>
  .banner {
    width: 100%;
    z-index: 0;
    display: block;
    overflow: hidden;
    position: relative;
  }

  .banner img {
    width: 100%;
    max-height: 348px;
    object-fit: cover;
    object-position: top;
    position: relative;
    background: url("/assets/loading_failure.jpeg");
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center;
  }

  .banner img:not(.reduce-motion) {
    will-change: transform, filter, opacity;
    transform: scale(var(--banner-scale));
    -webkit-filter: blur(var(--banner-blur));
    filter: blur(var(--banner-blur));
    opacity: var(--banner-opacity);
  }

  .banner::after {
    content: "";
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background: linear-gradient(
      to bottom,
      rgba(var(--secondary-rgb), 0.2) 50%,
      rgba(var(--secondary-rgb), 1) 100%
    );
  }

  .image-and-choices {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    position: relative;
    gap: 1rem;
  }

  .text {
    z-index: 1;
    overflow: visible;
    position: relative;
    display: block;
    margin-top: 0;
    background-color: var(--secondary-color);
    padding: 1rem;
    padding-top: 0;
    border-bottom: var(--color) 2px solid;
  }

  .overlap {
    margin-top: -125px;
  }

  .no-overlap {
    margin-top: 40px;
  }

  .thumbnail {
    z-index: inherit;
    position: relative;
    border-radius: 5px;
    height: fit-content;
    max-height: 305px;
    background: url("/assets/loading_failure.jpeg");
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center;
  }

  .important-info {
    display: grid;
    grid-column-gap: 30px;
    grid-template-columns: 215px auto;
    min-width: 320px;
  }

  .container {
    display: table;
    margin: 0 auto;
  }

  .title {
    font-size: x-large;
    font-weight: 300;
  }
  .description {
    max-width: 900px;
    font-size: small;
    font-weight: 300;
    line-height: 1.5;
    height: min-content;
  }

  .error {
    margin: 5rem;
    margin-top: 50px;
    text-align: center;
  }
</style>
