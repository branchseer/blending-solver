<script lang="ts" context="module">
  import type { Sample } from "./lib/SampleInput.svelte";
  import type { ColorFormats, Instance } from "tinycolor2";

  interface Solution {
    color: Instance,
    std: ColorFormats.RGBA
  }

  const rgbaZero: ColorFormats.RGBA = { r: 0, g: 0, b: 0, a: 0}

  const colorComponentNames: Array<keyof ColorFormats.RGB> = ['r', 'g', 'b'];

  function solve(sample1: Sample, sample2: Sample): Solution {
    function normalize(val: number): number {
      if (val < 0 && val > -1e-3) {
        return 0
      }
      if (val > 1 && val - 1 < 1e-3) {
        return 1
      }
      return val
    }
    const alphaSolutions = colorComponentNames.flatMap((componentName) => {
      const cb1 = sample1.backgroundColor[componentName] / 255
      const cb2 = sample2.backgroundColor[componentName] / 255
      const co1 = sample1.compositedColor[componentName] / 255
      const co2 = sample2.compositedColor[componentName] / 255

      if (Math.abs(cb1 - cb2) < 1e-6 || Math.abs(co1 - co2) < 1e-6) {
        return []
      }
      let alpha = normalize(1 - (co1 - co2) / (cb1 - cb2))
      if (alpha > 1 || alpha < 0) {
        throw new Error(`Alpha(${alpha.toPrecision(3)}) calculated from "${componentName}" is not between 0 and 1`)
      }
      return [alpha]
    })
    if (alphaSolutions.length === 0) {
      throw new Error('Backgroud colors cannot be the same')
    }
    const avgAlpha = alphaSolutions.reduce((prev, cur) => prev + cur, 0) / alphaSolutions.length
    const alphaStd = alphaSolutions.length === 1 ? 0 : Math.sqrt(
      alphaSolutions.reduce((prev, cur) => prev + Math.pow(cur - avgAlpha, 2), 0) /
      (alphaSolutions.length - 1)
    )
    const color = { ...rgbaZero, a: avgAlpha}
    const std= { ...rgbaZero, a: alphaStd}
    for (const componentName of colorComponentNames) {
      const cb1 = sample1.backgroundColor[componentName] / 255
      const cb2 = sample2.backgroundColor[componentName] / 255
      const co1 = sample1.compositedColor[componentName] / 255
      const co2 = sample2.compositedColor[componentName] / 255

      const ca1 = (co1 - cb1 * (1 - avgAlpha)) / avgAlpha
      const ca2 = (co2 - cb2 * (1 - avgAlpha)) / avgAlpha
      const ca = normalize((ca1 + ca2) / 2)
      if (ca > 1 || ca < 0) {
        throw new Error(`"${componentName}" value (${ca}) is not between 0 and 1`)
      }

      color[componentName] = ca * 255
      std[componentName] = Math.abs(ca1 - ca2)
    }
    return {
      color: tinycolor(color),
      std
    }
  }
</script>
<script lang="ts">
  import SampleInput from "./lib/SampleInput.svelte";
  import RGBAViewer from "./lib/RGBAViewer.svelte";
  import tinycolor from 'tinycolor2';

  let sample1: Sample = {
    backgroundColor: tinycolor('#ffffff').toRgb(),
    compositedColor: tinycolor('#aaccee').toRgb()
  }
  let sample2: Sample = {
    backgroundColor: tinycolor('#cce8ff').toRgb(),
    compositedColor: tinycolor('#88bdee').toRgb()
  }

  let solution: Solution
  let error: any
  $: {
    try {
      solution = solve(sample1, sample2)
      error = null
    }
    catch (err) {
      solution = {
        color: tinycolor(rgbaZero),
        std: rgbaZero
      }
      error = err
    }
  }
  
</script>

<main>
  <h1>Blending Solver</h1>
  <section>
    <h2>Sample 1</h2>
    <SampleInput bind:value={sample1} />
  </section>
  <section>
    <h2>Sample 2</h2>
    <SampleInput bind:value={sample2} />
  </section>
  <section>
    <h2>Solution</h2>
    
    <p>
      {#if error == null}
        { solution.color.toHex8String() }<br />
        { solution.color.toRgbString() } <br />
        Standard deviation:
          rgba({ solution.std.r.toPrecision(3) },
          { solution.std.g.toPrecision(3) },
          { solution.std.b.toPrecision(3) },
          { solution.std.a.toPrecision(3) }
          )
      {:else}
        {error}
      {/if}
    </p>
    <RGBAViewer style="width: 50%; height: 168px; margin: 0 auto;" rgba={solution.color.toRgb()} />
  </section>
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
      Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  }
  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }
</style>
