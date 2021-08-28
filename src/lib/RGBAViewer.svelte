<script lang="ts" context="module">
    import tinycolor from 'tinycolor2'
    import type { ColorFormats } from 'tinycolor2'
    export interface Sample {
      backgroundColor: ColorFormats.RGB
      compositedColor: ColorFormats.RGB
    }
  </script>
  <script lang="ts">
    export let style: string | undefined
    export let rgba: ColorFormats.RGBA
  </script>
  
  <div style={style} class="rgba-bg">
    <div class="rgba-fg" style={`background-color: ${tinycolor(rgba).toHex8String()}`}></div>
  </div>
  
  <style>
      .rgba-bg {
        position: relative;
        /* 
            Copyright (c) 2021 by Radovan Janjic (https://codepen.io/rjanjic/pen/dKKaXa)
            Permission is hereby granted, free of charge, to any person
            obtaining a copy of this software and associated documentation
            files (the "Software"), to deal in the Software without restriction,
            including without limitation the rights to use, copy, modify,
            merge, publish, distribute, sublicense, and/or sell copies of
            the Software, and to permit persons to whom the Software is
            furnished to do so, subject to the following conditions:

            The above copyright notice and this permission notice shall
            be included in all copies or substantial portions of the Software.

            THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
            EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
            OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
            NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
            HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
            WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
            OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
            DEALINGS IN THE SOFTWARE.
        */
        background-image:
            linear-gradient(45deg, #b0b0b0 25%, transparent 25%),
            linear-gradient(-45deg, #b0b0b0 25%, transparent 25%),
            linear-gradient(45deg, transparent 75%, #b0b0b0 75%),
            linear-gradient(-45deg, transparent 75%, #b0b0b0 75%);
        background-size: 20px 20px;
        background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
      }
      .rgba-fg {
        position: absolute;
        left: 20%;
        right: 20%;
        top: 20%;
        bottom: 20%;
      }
  </style>
  