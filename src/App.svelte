<script>
  import { onMount } from "svelte";
  import "./scss/styles.scss";
  import * as bootstrap from "bootstrap";

  import Compress from "./lib/Compress.svelte";
  import Decompress from "./lib/Decompress.svelte";

  let compression = $state(true);
  let isDark = $state(false);
  let themeMode = $state("light");

  function applyTheme(mode, persist = true) {
    themeMode = mode;
    isDark = mode === "dark";
    document.documentElement.setAttribute("data-bs-theme", mode);

    if (persist) {
      localStorage.setItem("themeMode", mode);
    }
  }

  onMount(() => {
    const popoverTriggerList = document.querySelectorAll(
      '[data-bs-toggle="popover"]',
    );
    const popoverList = [...popoverTriggerList].map(
      (popoverTriggerEl) =>
        new bootstrap.Popover(popoverTriggerEl, { html: true }),
    );

    const savedMode = localStorage.getItem("themeMode");
    if (savedMode === "light" || savedMode === "dark") {
      applyTheme(savedMode, false);
      return;
    }

    const prefersDark = window.matchMedia(
      "(prefers-color-scheme: dark)",
    ).matches;
    applyTheme(prefersDark ? "dark" : "light", true);
  });

  function switchCompression(compress) {
    if (compress == compression) return;

    compression = compress;
  }
</script>

<div
  class="position-absolute top-0 end-0 d-flex flex-row align-items-end gap-3 p-3"
>
  <div
    class="theme-toggle form-check form-switch ps-0 d-flex align-items-center gap-2 me-3"
  >
    <label for="themeToggle" class="theme-icon" aria-hidden="true">
      <svg width="30" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640">
        <!--!Font Awesome Free v7.2.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2026 Fonticons, Inc.-->
        <path
          class:dark={isDark}
          d="M320 32C328.4 32 336.3 36.4 340.6 43.7L396.1 136.3L500.9 110C509.1 108 517.8 110.4 523.7 116.3C529.6 122.2 532 131 530 139.1L503.7 243.8L596.4 299.3C603.6 303.6 608.1 311.5 608.1 319.9C608.1 328.3 603.7 336.2 596.4 340.5L503.7 396.1L530 500.8C532 509 529.6 517.7 523.7 523.6C517.8 529.5 509 532 500.9 530L396.2 503.7L340.7 596.4C336.4 603.6 328.5 608.1 320.1 608.1C311.7 608.1 303.8 603.7 299.5 596.4L243.9 503.7L139.2 530C131 532 122.4 529.6 116.4 523.7C110.4 517.8 108 509 110 500.8L136.2 396.1L43.6 340.6C36.4 336.2 32 328.4 32 320C32 311.6 36.4 303.7 43.7 299.4L136.3 243.9L110 139.1C108 130.9 110.3 122.3 116.3 116.3C122.3 110.3 131 108 139.2 110L243.9 136.2L299.4 43.6L301.2 41C305.7 35.3 312.6 31.9 320 31.9zM320 176C240.5 176 176 240.5 176 320C176 399.5 240.5 464 320 464C399.5 464 464 399.5 464 320C464 240.5 399.5 176 320 176zM320 416C267 416 224 373 224 320C224 267 267 224 320 224C373 224 416 267 416 320C416 373 373 416 320 416z"
        />
      </svg>
    </label>
    <input
      class="form-check-input mt-0"
      type="checkbox"
      role="switch"
      id="themeToggle"
      bind:checked={isDark}
      onchange={() => {
        applyTheme(isDark ? "dark" : "light", true);
      }}
    />
    <label class="form-check-label" for="themeToggle">
      <svg width="30" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640">
        <!--!Font Awesome Free v7.2.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2026 Fonticons, Inc.-->
        <path
          class:dark={isDark}
          d="M320 64C178.6 64 64 178.6 64 320C64 461.4 178.6 576 320 576C388.8 576 451.3 548.8 497.3 504.6C504.6 497.6 506.7 486.7 502.6 477.5C498.5 468.3 488.9 462.6 478.8 463.4C473.9 463.8 469 464 464 464C362.4 464 280 381.6 280 280C280 207.9 321.5 145.4 382.1 115.2C391.2 110.7 396.4 100.9 395.2 90.8C394 80.7 386.6 72.5 376.7 70.3C358.4 66.2 339.4 64 320 64z"
        />
      </svg>
    </label>
  </div>
  <!-- svelte-ignore a11y_missing_attribute -->
  <!-- svelte-ignore a11y_consider_explicit_label -->
  <button
    class="btn btn-sm {isDark ? 'btn-outline-light' : 'btn-outline-dark'}"
    data-bs-container="body"
    data-bs-toggle="popover"
    data-bs-trigger="focus"
    data-bs-placement="bottom"
    data-bs-content="Quellcode auf <a target='_blank' href='https://github.com/tools-info-bw-de/lzw'>GitHub</a><br/>Lizenz: MIT<br/>Autor: Marco Kümmel"
  >
    info
  </button>
</div>

<main>
  <h1>LZW Kompression</h1>

  <ul class="text-start mt-3">
    <li>
      Hinweis: Die hier gezeigte Vorgehensweise basiert auf der auf
      <a
        target="_blank"
        href="https://info-bw.de/faecher:informatik:oberstufe:codierung:lzw:start"
        >info-bw.de</a
      >
      gezeigten Anleitung. Dies entspricht den
      <a
        target="_blank"
        href="https://lehrerfortbildung-bw.de/u_matnatech/informatik/gym/bp2016/fb2/01_duc/2_vorlagen/4_lzw/"
        >Materialien vom ZSL BW</a
      >.
    </li>
    <li>
      Dieses Tool unterstützt die <abbr
        title="Diese ist kompatibel zu ASCII und enthält zusätzlich westeuropäische Sonderzeichen"
        >ISO-8859-1</abbr
      > Zeichenkodierung.
    </li>
    <li>
      Codes werden außschließlich in <b>Hexadezimal</b> dargestellt.
    </li>
  </ul>

  <ul class="nav nav-pills nav-justified my-4">
    <li class="nav-item">
      <button
        class="nav-link {compression ? 'active' : ''}"
        onclick={() => {
          switchCompression(true);
        }}
        aria-current="page">Kompression / Codierung</button
      >
    </li>
    <li class="nav-item">
      <button
        class="nav-link {!compression ? 'active' : ''}"
        onclick={() => {
          switchCompression(false);
        }}>Dekompression / Decodierung</button
      >
    </li>
  </ul>

  {#if compression}
    <Compress />
  {:else}
    <Decompress />
  {/if}
</main>

<style>
  .form-check-input {
    height: 1.4em;
    width: 2.5em;
  }
  svg > path.dark {
    fill: #fff;
  }

  main {
    position: relative;
  }

  .theme-toggle .form-check-input {
    margin-left: 0;
  }

  .theme-toggle .form-check-label {
    margin-left: 0;
  }

  .nav {
    border: 1px solid var(--bs-primary);
    border-radius: var(--bs-nav-pills-border-radius);
  }
</style>
