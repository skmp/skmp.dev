---
layout: post
title: FEX-Emu
img: fex-emu.png
slug: fex-emu
year: 2020-2022
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/videoseries?list=PL_ZGlfnzU2tvlk-Ui-3D9kqwODXPwDU5O" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

[FEX-Emu](https://fex-emu.org) is a usermode x86 and x86-64 emulator, supporting arm64 and x86-64 hosts. It is similar to qemu and box86,
though it follows a different approach, using an SSA based IR.

My contributions have focused around performance and compatibility, as I love optimization problems and solving bugs.

FEX is currently in a state where it can run several 64-bit applications, including being able to build itself. At the time of writting this, 32-bit support is still an uphill battle, but things should improve soon.

For more information check the [FEX-Emu site](https://fex-emu.org)!

After a fall-out with the team, some related, private/forked work can be found at the [hex-emu gitlab](https://gitlab.com/hex-emu/hex-emu)
