<script lang="ts" context="module">
  import type { Sample } from "./lib/SampleInput.svelte";
  import type { ColorFormats } from "tinycolor2";

  interface Solution {
    color: tinycolor.Instance,
    std: number
  }

  function solve(sample1: Sample, sample2: Sample): Solution {
    console.log(sample1, sample2)
    const solutions = new Array<keyof ColorFormats.RGB>('r', 'g', 'b').map((componentName) => {
      const cb1 = sample1.backgroundColor[componentName] / 255
      const cb2 = sample2.backgroundColor[componentName] / 255
      const co1 = sample1.compositedColor[componentName] / 255
      const co2 = sample2.compositedColor[componentName] / 255

      if (cb1 === cb2) {
        throw new Error('Backgroud colors cannot have the same coresponding component values')
      }
      const aa = 1 - (co1 - co2) / (cb1 - cb2)
      const ca = (co1 - (cb1 * (1 - aa))) / aa
      if (ca < 0 || ca > 1 || aa < 0 || aa > 1) {
        throw new Error('Composited color is not possible to be blended from backgroud color')
      }
      console.log(componentName, aa);
      return { aa, ca }
    })
    const aaAverage = solutions.reduce((prev, cur) => prev + cur.aa, 0) / solutions.length
    const std = Math.sqrt(
      solutions.reduce((prev, cur) => prev + Math.pow(cur.aa - aaAverage, 2), 0) /
      (solutions.length - 1)
    )
    console.log(solutions)
    return {
      color: tinycolor({
        r: solutions[0].ca * 255,
        g: solutions[1].ca * 255,
        b: solutions[2].ca * 255,
        a: aaAverage
      }),
      std
    }
  }
</script>
<script lang="ts">
  import SampleInput from "./lib/SampleInput.svelte";
  import RGBAViewer from "./lib/RGBAViewer.svelte";
  import tinycolor from 'tinycolor2';
  import type { Instance } from 'tinycolor2';
import { prevent_default } from "svelte/internal";

  let sample1: Sample = {
    backgroundColor: tinycolor('#ffffff').toRgb(),
    compositedColor: tinycolor('#aaccef').toRgb()
  }
  let sample2: Sample = {
    backgroundColor: tinycolor('#e6e3e4').toRgb(),
    compositedColor: tinycolor('#9abbde').toRgb()
  }

  let solution: Solution
  let error: string | null;
  $: {
    try {
      solution = solve(sample1, sample2)
      error = null
    }
    catch (err) {
      error = err
      solution = {
        color: tinycolor('#00000000'),
        std: 0
      }
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
        Standard deviation: { solution.std.toPrecision(3) }
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
