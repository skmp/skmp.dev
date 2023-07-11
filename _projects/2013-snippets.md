---
layout: post
title: snippets.nihil.cc
img: snippets.png
slug: snippets
year: 2013
---

Distilled Moments.

If no audio, press the play button in the video embed.

Best enjoyed fullscreen.

<button onclick="openFullscreen();">Makez Fullscreenerz</button>

<iframe id="snippets" width="720" height="640" src="https://skmp.github.io/snippets.skmp.dev/" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<script>
var elem = document.getElementById("snippets");
function openFullscreen() {
  if (elem.requestFullscreen) {
    elem.requestFullscreen();
  } else if (elem.webkitRequestFullscreen) { /* Safari */
    elem.webkitRequestFullscreen();
  } else if (elem.msRequestFullscreen) { /* IE11 */
    elem.msRequestFullscreen();
  }
}
</script>


<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/jBaikK9vR7M?autoplay=1&loop=1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
