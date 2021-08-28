<script lang="ts" context="module">
  import tinycolor from 'tinycolor2'
  import type { ColorFormats } from 'tinycolor2'
  export interface Sample {
    backgroundColor: ColorFormats.RGB
    compositedColor: ColorFormats.RGB
  }
</script>
<script lang="ts">
  export let value: Sample
  let backgroundColorString = tinycolor(value.backgroundColor).toHexString()
  let compositedColorString = tinycolor(value.compositedColor).toHexString()
  $: value = { 
    backgroundColor: tinycolor(backgroundColorString).toRgb(),
    compositedColor: tinycolor(compositedColorString).toRgb()
  }
</script>

<div>
  <label>Background: <input type="color" bind:value={backgroundColorString} /></label>
  <input bind:value={backgroundColorString} />
  <br />
  <label>Composited: <input type="color" bind:value={compositedColorString} /></label>
  <input bind:value={compositedColorString} />
</div>

<style>
</style>
