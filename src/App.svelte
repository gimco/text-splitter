<script>
import { onMount } from 'svelte';

const pasteable = !!navigator.clipboard.readText

function paste() {
  navigator.clipboard.readText()
  .then(cliptext => text = cliptext)
  .then(trocear)
}

let text = ""
/**
 * @type {string[]}
 */
let parts = []
let curpart = 0
let paging = false
$: paging = parts.length > 0

function trocear() {
  if (text.trim().length == 0) return
  localStorage.setItem("text", text)
  parts = splitText(text) || []
  if (parts.length > 0) {
    showPart(curpart)
  }
}

/**
 * @param {string} text
 */
function splitText(text) {
  const limit = 4900
  let chunks = []
  while (text.length > limit) {
    let nlpos = text.substr(0, limit).lastIndexOf("\n")
    if (nlpos <= 0) nlpos = limit
    chunks.push(text.substring(0, nlpos))
    text = text.substring(nlpos)
  }
  chunks.push(text)
  return chunks;
}

function reset() {
  localStorage.clear()
  text = ""
  parts = []
  curpart = 0
  copy()
}

function showPart(part) {
  curpart = part
  text = parts[curpart]
  localStorage.setItem("curpart", curpart.toString())
  copy()
}

/**
 * @type {HTMLTextAreaElement}
 */
let textarea
function copy () {
  textarea.value = text
  textarea.select()
  navigator.clipboard.writeText(text)
}

onMount(async () => {
  text = localStorage.getItem("text") || ""
  curpart = +localStorage.getItem("curpart")
  if (!!text) {
    trocear()
  }
});
</script>

{#if !paging}
<div>
  {#if pasteable}
  <button on:click={paste}>Paste!</button>
  {:else}
  <button on:click={trocear} disabled={text.length == 0}>Split!</button>
  {/if}
</div>
{:else}
<div>
  <button on:click={reset}>Reset</button>
  <button disabled={curpart == 0} on:click={() => showPart(curpart - 1)}>Prev</button>
  <button disabled={curpart + 1 == parts.length} on:click={() => showPart(curpart + 1)}>Next</button>
  <button on:click={copy}>Copy</button>
</div>
{/if}
<textarea on:paste={trocear} readonly={parts.length > 0} bind:this={textarea} bind:value={text}></textarea>
{#if paging}
<small>Part {curpart+1} of {parts.length} ({parts[curpart].length} currently, {text.length} total characters)</small>
{/if}

<style>
  textarea {
    height: 200px;
  }
</style>