---
layout: post
title: "Tegra K1: First impression"
img: tegra-k1.jpg
---

Finally got some time to play with a shinny new toy, the nvidia jetson tk1 development board. It was ordered a couple of months ago from Seco (here to be precise) and after delays It got into my hands last week! Ohh the excitement.

The board is neatly packaged, and comes with a microusb cable - same as in nvidia shielf - and a nice 12V/5A PSU. The board had an ubuntu 14.04 image flashed, which is naturally armv7a/hardfp. Booting w/ hdmi and a wireless keyboard attached loaded the console, and after typing some commands the nvidia driver package was installed w/ full X11 working. Nice and easy.

<image>

glxinfo reports 4.3 and 4.4 compliance, and a nice list of extensions

After the initial fun, I hooked it up with Ethernet and decided to compile reicast for it. Looks like my arm makefiles are quite outdated, so I hackfixed the Beagle makefile to target hardfp. And poof, it compiled. Didn't time it, but it was not much slower than the laptop. Looks like the EGL symlinks are missing though, so I had to 

    sudo ln -s /usr/lib/arm-linux-gnueabihf/tegra-egl/libEGL.so.1 /usr/lib/libEGL.so

    sudo ln -s /usr/lib/arm-linux-gnueabihf/tegra-egl/libGLESv2.so.2 /usr/lib/libGLESv2.so

to get it ruining. (I don't think that's a proper fix - don't quote me on it). If you symlink be careful to use the /tegra-egl/ and not /mesa-egl/ folder ~

./reicast.elf opens up the dreamcast bios and ...

    newdc rel0/n - 0.78 (0.00) - 1281.08 - V: 767.95 (1.00, NTSC480i59.94) R: 738.95+28.99 VTX: 0.00

Not bad. 740 fps on the bios datetime select menu. The bios main menu is closer to 320 fps.

Moving over to ikaruga (a favorite test case), 

    ./reicast.elf -config config:image=ika.chd

    #ikaruga main menu

    newdc rel0/n - 0.73 (0.00) - 1370.66 - V: 821.65 (1.00, NTSC480i59.95) R: 763.17+58.47 VTX: 0.00

    #ikaruga ingame

    newdc rel0/n - 1.83 (0.00) - 547.52 - V: 328.21 (1.00, NTSC480i59.95) R: 278.26+49.96 VTX: 0.00

    #ikaruga slow spot after 1st phase

    vnewdc rel0/n - 3.07 (0.00) - 325.43 - V: 195.08 (1.00, NTSC480i59.95) R: 98.29+96.79 VTX: 0.00

Welp. amazing results. The gpu isn't able to keep up, but one could blame the pretty pathetic gles2 driver reicast has right now.

And for the fun of it <youtube video>

So, all in all, looks like K1 is quite capable. Let's see how it behaves over the coming weeks' stress testing :3