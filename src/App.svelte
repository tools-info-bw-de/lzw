<script>
  import "./scss/styles.scss";
  import * as bootstrap from "bootstrap";

  import Compress from "./lib/Compress.svelte";
  import Decompress from "./lib/Decompress.svelte";

  let compression = $state(true);

  function switchCompression(compress) {
    if (compress == compression) return;

    compression = compress;
  }
</script>

<main>
  <h1>LZW Kompression</h1>

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

  <ul class="text-start mt-3">
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
  .nav {
    border: 1px solid var(--bs-primary);
    border-radius: var(--bs-nav-pills-border-radius);
  }
</style>
