<script>
  import { slide } from "svelte/transition";

  let input = $state("");

  let inputNotAllowed = $state(false);
  let dictionary = $state([]);
  let steps = $state([]);

  function compress() {
    console.log("effect", input);

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
      console.log(element);
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

    //if (outputCode === "") {
    steps.push({
      1: lastEntry,
      2: "Wortende",
      3: "–",
      4: dictionary.find((entry) => entry.char === lastEntry)?.code || "–",
    });
    //}
  }
</script>

<div class="row mb-4">
  <div class="col-md-6 text-start">
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
  <div class="col-md-6 text-end resultBox">
    <div class="fw-semibold fs-5">
      Ausgabe <i style="font-size: 0.8rem; font-weight: normal">(hex)</i>
    </div>
    <code
      ><b>
        {steps.length > 0
          ? steps
              .map((s) => s[4])
              .filter((code) => code !== "–")
              .map((code) => code.padStart(3, "0"))
              .join(" ")
          : ""}
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
        {#each steps as s}
          <tr>
            <td>{s[1]}</td>
            <td>{s[2]}</td>
            <td>{s[3]}</td>
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

  .dictionary {
    padding: 1rem;
    border-radius: 15px;
    width: 30%;
    margin-right: 2rem;
    background-color: #e9e9e9;
  }

  .steps {
    padding: 1rem;
    border-radius: 15px;
    background-color: #e9e9e9;
    width: 70%;
  }
</style>
