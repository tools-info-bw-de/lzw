<script>
  import { onMount } from "svelte";
  import { slide } from "svelte/transition";

  let inputCodes = $state("");
  let inputInvalid = $state(false);
  let output = $state("");
  let dictionary = $state([]);
  let steps = $state([]);

  onMount(() => {
    document.getElementById("inputDecode").focus();
  });

  let invalidChars = $state(false);
  function normalizeInputCodes(raw) {
    invalidChars = /[^0-9a-fA-F\s]/.test(raw);
    const hexOnly = raw.replace(/[^0-9a-fA-F]/g, "").toUpperCase();
    const grouped = hexOnly.match(/.{1,3}/g)?.join(" ") || "";
    const incomplete = hexOnly.replace(/\s/g, "").length % 3 !== 0;
    return { grouped, incomplete };
  }

  function decode() {
    inputInvalid = false;

    const { grouped, incomplete } = normalizeInputCodes(inputCodes);
    const normalized = grouped.trim();
    inputCodes = normalized;

    if (incomplete || invalidChars) {
      return;
    }

    if (normalized === "") {
      output = "";
      dictionary = [];
      steps = [];
      return;
    }

    const tokens = normalized.split(" ");

    // input all used iso-8859-1 chars from input into the dictionary
    dictionary = [];
    tokens.forEach((t) => {
      if (t[0] === "0" && !dictionary.some((e) => e.code === t)) {
        dictionary.push({
          char: String.fromCharCode(parseInt(t, 16)),
          code: t,
        });
      }
    });

    // sort dictionary by code
    dictionary.sort((a, b) => a.code.localeCompare(b.code));

    let nextDictCode = 256;

    output = "";
    steps = [];

    let previousEntry = { code: "-", char: "-" };

    for (let i = 0; i < tokens.length; i++) {
      const token = tokens[i].toUpperCase();
      //const code = parseInt(token, 16);
      let currentCode = dictionary.find((e) => e.code === token);
      if (!currentCode) {
        inputInvalid = true;
        return;
      }

      // new entry for dictionary?
      let newChar = "-";
      let newCode = "-";
      if (previousEntry.code !== "-") {
        newChar = previousEntry.char + currentCode.char[0];
        newCode = nextDictCode.toString(16).toUpperCase().padStart(3, "0");
        nextDictCode++;

        if (!dictionary.some((e) => e.code === newCode)) {
          dictionary.push({
            char: newChar,
            code: newCode,
          });
        }
      }

      let outputCode = previousEntry?.char || "-";

      steps.push({
        1: previousEntry,
        2: currentCode,
        3: { char: newChar, code: newCode },
        4: outputCode,
      });

      previousEntry = currentCode || "";
    }

    steps.push({
      1: previousEntry,
      2: "-",
      3: { char: "-", code: "-" },
      4: previousEntry?.char || "-",
    });
  }
</script>

<div class="row mb-4">
  <div class="col-md-5 text-start">
    <label for="inputDecode" class="form-label fw-semibold fs-5"
      >Eingabe <i class="text-muted ms-2 fs-6 fw-normal">Hex-Codes</i>
    </label>
    <input
      bind:value={inputCodes}
      oninput={decode}
      class="form-control {inputInvalid || invalidChars ? 'is-invalid' : ''}"
      id="inputDecode"
      placeholder="Gib hier deine Hex-Codes ein, z.B. 042 041 100 ..."
    />
    {#if invalidChars}
      <div transition:slide>
        <div class="pt-2 pb-3">
          <div class="alert alert-danger mb-0" role="alert">
            Bitte nur gültige Hex-Codes angeben (3 Zeichen pro Code).
          </div>
        </div>
      </div>
    {/if}
    {#if inputInvalid}
      <div transition:slide>
        <div class="pt-2 pb-3">
          <div class="alert alert-danger mb-0" role="alert">
            Aktuelle Eingabe ist unmöglich zu dekodieren. Hast du dich vertippt?
          </div>
        </div>
      </div>
    {/if}
  </div>
  <div
    class="col-md-2 d-flex align-items-center justify-content-center io-arrow"
  >
    <span aria-hidden="true">→</span>
  </div>
  <div class="col-md-5 text-start resultBox">
    <div class="fw-semibold fs-5">Ausgabe</div>
    <code class="fs-4">
      <b>
        {#each steps as s}
          {s[4] !== "-" ? s[4] : ""}
        {/each}
      </b>
    </code>
    <br />
    {#if output.length > 0}
      <b>Speicherplatz: {output.length} Bytes</b>
    {/if}
  </div>
</div>

<div class="d-flex flex-row">
  <div class="d-flex flex-column dictionary">
    <h3>Wörterbuch</h3>
    <table class="table table-striped table-hover table-bordered">
      <thead>
        <tr>
          <th scope="col">Zeichen/Kombination</th>
          <th scope="col"
            >Code <i style="font-size: 0.8rem; font-weight: normal">(hex)</i
            ></th
          >
        </tr>
      </thead>
      <tbody>
        {#each dictionary as entry}
          <tr>
            <td>{entry.char}</td>
            <td>{entry.code}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>

  <div class="d-flex flex-column steps">
    <h3>Schritte</h3>
    <table class="table table-striped table-hover table-bordered">
      <thead>
        <tr>
          <th scope="col">Letzter Code </th>
          <th scope="col">Aktueller Code </th>
          <th scope="col">Neuer Eintrag Wörterbuch</th>
          <th scope="col">Ausgabe</th>
        </tr>
      </thead>
      <tbody>
        {#each steps as s}
          <tr>
            <td>
              {#if s[1].code === "-"}
                -
              {:else}
                {s[1].code}
                <span class="text-danger">{s[1].char}</span>
              {/if}
            </td>
            <td
              >{#if s[2] === "-"}
                -
              {:else}
                {s[2].code}
                <span class="text-danger">{s[2].char[0]}</span><span
                  class="text-primary">{s[2].char.substring(1)}</span
                >
              {/if}
            </td>
            <td>
              {#if s[3].char === "-"}
                -
              {:else}
                {s[3].code} <span class="text-danger">{s[3].char}</span>
              {/if}
            </td>
            <td>{s[4]}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>

<style>
  .resultBox {
    padding: 1rem;
    border-radius: 15px;
    background-color: var(--bs-secondary-bg);
  }

  .io-arrow {
    font-size: 4rem;
    color: #666666;
  }

  .dictionary {
    padding: 1rem;
    border-radius: 15px;
    width: 30%;
    margin-right: 2rem;
    background-color: var(--bs-secondary-bg);
  }

  .steps {
    padding: 1rem;
    border-radius: 15px;
    background-color: var(--bs-secondary-bg);
    width: 70%;
  }
</style>
