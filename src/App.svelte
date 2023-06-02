<script>
/**
 * @type {HTMLTextAreaElement}
 */
let textarea

const pasteable = !!navigator.clipboard.readText

function paste() {
  navigator.clipboard.readText()
  .then(cliptext => textarea.value = cliptext)
  .then(trocear)
}

let text = ""
/**
 * @type {string[]}
 */
let parts = []
let curpart = 0
function trocear() {
  text = textarea.value
  parts = splitText(text) || [""]
  showPart(0)
}

/**
 * @param {string} text
 */
function splitText(text) {
  const chunkSize = 4900;
  const regex = new RegExp(`.{1,${chunkSize}}`, 'gs');
  const chunks = text.match(regex);
  return chunks;
}

function reset() {
  textarea.value = text = ""
  copy()
}

function showPart(part) {
  curpart = part
  textarea.value = parts[curpart]
  copy()
}

function prev() {
  showPart(curpart - 1)
}

function next() {
  showPart(curpart + 1)
}

function copy() {
  textarea.select()
  navigator.clipboard.writeText(textarea.value)
}

</script>

{#if text == ""}
<div>
  {#if pasteable}
  <button on:click={paste}>Paste!</button>
  {:else}
  <button on:click={trocear}>Split!</button>
  {/if}
</div>
{:else}
<div>
  <button on:click={reset}>Reset</button>
  <button disabled={curpart == 0} on:click={prev}>Prev</button>
  <button disabled={curpart + 1 == parts.length} on:click={next}>Next</button>
  <button on:click={copy}>Copy</button>
</div>
{/if}
<textarea on:paste={() => setTimeout(trocear, 0)} bind:this={textarea}></textarea>
{#if text != ""}
<small>Part {curpart+1} of {parts.length} ({parts[curpart].length} currently, {text.length} total characters)</small>
{/if}

<style>
  textarea {
    height: 200px;
  }
</style>