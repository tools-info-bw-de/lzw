<script>
  import { onMount } from "svelte";
  import { slide } from "svelte/transition";

  let input = $state("");

  let inputNotAllowed = $state(false);
  let dictionary = $state([]);
  let steps = $state([]);
  let hoveredStepIndex = $state(null);
  let hoveredDictCode = $state(null);

  let outputSteps = $derived(
    steps
      .map((s, index) => ({
        code: s[4],
        index,
      }))
      .filter((item) => item.code !== "–"),
  );

  function getStepPopupText(step) {
    return step[2] === "Wortende"
      ? "Ausgabe, weil das Wortende erreicht ist."
      : "Ausgabe, weil ein neuer Wörterbuch-Eintrag erstellt wurde.";
  }

  function shouldShowLastCharNotice(step, index) {
    const previousStep = steps[index - 1];
    return Boolean(previousStep && previousStep[3] !== "–");
  }

  function getPreviousCurrentChar(index) {
    const previousStep = steps[index - 1];
    return previousStep ? previousStep[2] : "–";
  }

  onMount(() => {
    // @ts-ignore
    document.querySelector("#inputEncode").focus();
  });

  function compress() {
    // remove all whitespace characters from input
    input = input.replace(/\s/g, "");

    // check, that input only contains iso-8859-1 characters
    if (input.split("").some((char) => char.charCodeAt(0) > 255)) {
      inputNotAllowed = true;
      return;
    } else {
      inputNotAllowed = false;
    }

    // input all used iso-8859-1 characters from input into the dictionary
    dictionary = [];
    input.split("").forEach((element) => {
      if (!dictionary.some((entry) => entry.char === element)) {
        dictionary.push({
          char: element,
          code: element
            .charCodeAt(0)
            .toString(16)
            .toUpperCase()
            .padStart(3, "0"),
        });
      }
    });
    // sort dictionary by code
    dictionary.sort((a, b) => a.code.localeCompare(b.code));

    let nextDictCode = 256;

    // traverse input and create steps
    steps = [];
    let lastEntry = "";
    let outputCode = "";
    for (let i = 0; i < input.length; i++) {
      const currentChar = input[i];
      let newEntry = lastEntry + currentChar;

      // wenn neuer eintrag im wörterbuch gemacht wird --> codeausgabe != 0
      outputCode = "";
      if (!dictionary.some((entry) => entry.char === newEntry)) {
        dictionary.push({
          char: newEntry,
          code: nextDictCode.toString(16).toUpperCase().padStart(3, "0"),
        });
        nextDictCode++;
        outputCode =
          dictionary.find((entry) => entry.char === lastEntry)?.code || "";
      }

      steps.push({
        1: lastEntry === "" ? "–" : lastEntry,
        2: currentChar,
        3: outputCode === "" ? "–" : newEntry,
        4: outputCode === "" ? "–" : outputCode,
      });

      // reset NewEntry if it was added to the dictionary
      if (outputCode !== "") {
        lastEntry = currentChar;
      } else {
        lastEntry = newEntry;
      }
    }

    steps.push({
      1: lastEntry,
      2: "Wortende",
      3: "–",
      4: dictionary.find((entry) => entry.char === lastEntry)?.code || "–",
    });
  }
</script>

<div class="row mb-4">
  <div class="col-md-5 text-start">
    <label for="inputEncode" class="form-label fw-semibold fs-5"
      >Eingabe <i class="text-muted ms-2 fs-6 fw-normal"
        >Groß-/Kleinschreibung beachten!</i
      >
    </label>
    <div class="text-secondary"></div>
    <input
      bind:value={input}
      oninput={compress}
      type="text"
      class="form-control {inputNotAllowed ? 'is-invalid' : ''}"
      id="inputEncode"
      placeholder="Gib hier deinen Text ein..."
    />
    {#if inputNotAllowed}
      <div transition:slide>
        <div class="pt-2 pb-3">
          <div class="alert alert-danger mb-0" role="alert">
            Es sind nur ISO-8859-1 Zeichen erlaubt!
          </div>
        </div>
      </div>
    {/if}
    Länge: {input.length} Zeichen<br />
    <b>Speicherplatz: {input.length} Bytes</b>
  </div>
  <div
    class="col-md-2 d-flex align-items-center justify-content-center io-arrow"
  >
    <span aria-hidden="true">→</span>
  </div>
  <div class="col-md-5 text-start resultBox">
    <div class="fw-semibold fs-5">
      Ausgabe <i style="font-size: 0.8rem; font-weight: normal">(hex)</i>
    </div>
    <code class="output-code fs-4">
      <b>
        {#if outputSteps.length > 0}
          {#each outputSteps as item, idx}
            {#if idx > 0}
              {" "}
            {/if}
            <span
              role="button"
              tabindex="0"
              class:hovered={hoveredStepIndex === item.index}
              onmouseenter={() => {
                hoveredStepIndex = item.index;
                hoveredDictCode = item.code;
              }}
              onmouseleave={() => {
                hoveredStepIndex = null;
                hoveredDictCode = null;
              }}>{item.code.padStart(3, "0")}</span
            >
          {/each}
        {/if}
      </b>
    </code>
    <br />
    {#if steps.length > 0}
      <b
        >Speicherplatz: {steps
          .map((s) => s[4])
          .filter((code) => code !== "–")
          .map((code) => parseInt(code, 16)).length * 1.5} Bytes
      </b>
    {/if}
  </div>
</div>

<div class="d-flex flex-row">
  <div class="d-flex flex-column dictionary">
    <h3>
      Wörterbuch
      <span class="help-popup" aria-label="Erklärung zum Wörterbuch">
        ?
        <span class="popup-bubble" aria-hidden="true">
          {@html "Zeigt alle bisher bekannten Zeichen und <em>Zeichenfolgen</em> mit ihren Hex-Codes.<br/>Bis zu <code>0FF</code> (Zeichen ÿ) entsprechen die Codes den ISO-8859-1 Zeichen. Alle weiteren Einträge entstehen durch die Kompression und erhalten fortlaufende Codes ab <code>100</code>."}
        </span>
        <span class="popup-arrow" aria-hidden="true"></span>
      </span>
    </h3>
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
          <tr class:highlight-row={entry.code === hoveredDictCode}>
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
          <th scope="col">Letztes Zeichen / Kombination</th>
          <th scope="col">Aktuelles Zeichen</th>
          <th scope="col">Neuer Eintrag Wörterbuch</th>
          <th scope="col"
            >Ausgabe Code des letzten Zeichens / Kombination <i
              style="font-size: 0.8rem; font-weight: normal">(hex)</i
            ></th
          >
        </tr>
      </thead>
      <tbody>
        {#each steps as s, i}
          <tr>
            <td class:highlight-cell={i === hoveredStepIndex}>
              {s[1]}
            </td>
            <td>{s[2]}</td>
            <td>{s[3]}</td>
            <td
              class:highlight-cell={i === hoveredStepIndex}
              class:popup-active={s[4] !== "–"}
              onmouseenter={() => {
                if (s[4] === "–") return;
                hoveredStepIndex = i;
                hoveredDictCode = s[4];
              }}
              onmouseleave={() => {
                hoveredStepIndex = null;
                hoveredDictCode = null;
              }}
            >
              {s[4]}
              {#if s[4] !== "–"}
                <span class="popup-bubble" aria-hidden="true">
                  {@html getStepPopupText(s)}
                </span>
                <span class="popup-arrow" aria-hidden="true"></span>
              {/if}
            </td>
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

  .output-code span {
    cursor: pointer;
    border-radius: 4px;
    padding: 0 2px;
  }

  .output-code span.hovered {
    background-color: #ffe7a3;
  }

  .highlight-cell {
    background-color: #ffe7a3 !important;
    color: #000 !important;
    font-weight: 600;
  }

  .dictionary .highlight-row td {
    background-color: #ffe7a3 !important;
    color: #000 !important;
    font-weight: 600;
  }

  .help-popup {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 1.1rem;
    height: 1.1rem;
    margin-left: 0.4rem;
    border-radius: 50%;
    background-color: #1f1f1f;
    color: #ffffff;
    font-size: 0.85rem;
    line-height: 1.5;
    cursor: help;
    position: relative;
    vertical-align: middle;
  }

  td.popup-active,
  .help-popup {
    position: relative;
  }

  .popup-bubble {
    position: absolute;
    left: 50%;
    bottom: 120%;
    transform: translateX(-50%);
    width: max-content;
    max-width: 30rem;
    white-space: normal;
    overflow-wrap: anywhere;
    box-sizing: border-box;
    background-color: #1f1f1f;
    color: #ffffff;
    padding: 0.35rem 0.5rem;
    border-radius: 6px;
    font-size: 0.9rem;
    opacity: 0;
    pointer-events: none;
    z-index: 6;
  }

  .popup-arrow {
    position: absolute;
    left: 50%;
    bottom: 105%;
    transform: translateX(-50%);
    border-width: 6px;
    border-style: solid;
    border-color: #1f1f1f transparent transparent transparent;
    opacity: 0;
    pointer-events: none;
    z-index: 6;
  }

  td.popup-active:hover .popup-bubble,
  td.popup-active:hover .popup-arrow,
  .help-popup:hover .popup-bubble,
  .help-popup:hover .popup-arrow {
    opacity: 1;
  }

  @media (max-width: 640px) {
    .popup-bubble {
      position: fixed;
      left: 50%;
      bottom: 1rem;
      top: auto;
      transform: translateX(-50%);
      width: calc(100vw - 2rem);
      max-width: 36rem;
      z-index: 1000;
    }

    .popup-arrow {
      display: none;
    }
  }
</style>
