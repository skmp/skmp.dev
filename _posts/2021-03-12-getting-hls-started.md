---
layout: post
title: Getting started with FPGAdc and HLS, part 1
img: fpagdc-hls-clx2core.jpg
---

I've been meaning to work on an fpga implementation of the Dreamcast for almost 2 years now, and last week finally I have an ultra96 I can play with.

Current idea is to move CLX2's CORE into a memory mapped device, then once that is working, to add the TA component, and offload all of the rendering to the fpga, while the rest of the Dreamcast runs emulated.


What's what?
---
- HLS - High Level Synthesis, essentially a C/C++ to hdl (verilog/vhdl) compiler.
- CLX2 - The PowerVR part used in the Dreamcast
- CORE - The pixel rendering unit of CLX2. Reads command lists and renders to an on chip tile buffer.
- TA - The "Tile Accelerator" unit of CLX2. Gets polygons and prepares commands lists for CORE.
- ISP - The "Image Synthesis Processor", the part of CORE that determines which pixels are visible, and to which polygon they belong. This process is fairly fast (> 1 pixel/clock, i'm told)
- TSP - The "Texture and Shading Processor", the part of CORE that calculates the color of a pixel by interpolating vertex colors, mixing them with texture color, and blending to the tile buffer.


First steps
---
Getting started with fpga dev is always frustrating. I have Vivado 2019.2 installed, and i've downloaded the ultra96-v2 board files. I tried generating a linux image, but failed, as the steps involved into getting a fully working kernel + userland are non trivial.

So I decided to instead "cheat" and try HLS using Vivado HLS. While the HLS path is no substitute for hand-written hdl code, I already have a clean C-based implementation in [reicast](https://github.com/reicast/reicast-emulator/blob/alpha/libswirl/rend/soft/refsw_pixel.cpp) that I can quickly port.

The main rationale is to see how much of the FPGA is used up with a crude, unoptimized version of ISP and TSP.

Getting the code to compile for HLS
---

The reicast refsw code is quite abstracted, uses unions and pointers liberally, and in general was not meant to be compiled for hardware. I had to refactor the code to be more "hardware friendly", which mostly consisted of:
- Getting rid of unions and replacing them with bitmath.
- Getting rid of pointers, and instead replace them with global indexes to global arrays.
- Making template parameters function arguments. RefSW uses templates to generate optimized code paths, but the hardware can just generate MUXes for that sort of thing.
- Implementing an interface that can be mapped into memory, that takes 32-bit address, and 32-bit data parameters

The resulting source can be found in this [gist](https://gist.github.com/skmp/fc3ae9b26f88b37706642cc23b49adca).

I also wrote a small [test bench](https://gist.github.com/skmp/92ed37e53a94978cdc5c8dfd7e8591ef) that renders some polygons


Show me the pixels
---
Initial ISP results looked like this:

![Hey it's a triangle](/images/fpgadc-triangle-isp.png)

After tinkering some more, implementing multiple triangle support, and adding the TSP module everything seemed to be working:

![Hey it's two shaded triangles](/images/fpgadc-triangle-tsp.png)

At that point I went back to reicast and I hooked the HLS code to the emulator, and a few iterations later I had:

![Hey it's the Dreamcast Bios](/images/fpagdc-hls-clx2core.jpg)

Keep in mind, this is still running as C code, and not in the FPGA fabric. The glue logic is included in the [gist](https://gist.github.com/skmp/fc3ae9b26f88b37706642cc23b49adca).


But will it fit?
---
TL;DR answer: Maybe, Vivado HLS thinks so. I was told not to trust this too much before I see it running.

![Hey Vivado thinks it'll fit](/images/fpgadc-hls-result.png)

There's quite a bit more work left to do here, which I'll hopefully cover in a follow up post.
- The resulting verilog needs to be synthesized, connected over AXI and exposed to linux.
- I need to get a linux kernel working
- I need to get an ubuntu userland working
- I need to get reicast working on the board. I'll likely have to scrap the UI.
- I need to change the glue logic to use the memory mapped device
- Then it we'll have real numbers, and it'll be party time.

Anyways, that's all for now ~
