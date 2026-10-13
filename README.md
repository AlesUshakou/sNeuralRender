<h1 align="center">sNeuralRender</h1>

<p align="center"><b>NVIDIA DLSS 5 Neural Rendering as a native Nuke node.</b><br>
Neural relighting and detail on your footage, right in the node graph — with a mask input,
highlight recovery and temporal history.</p>

<p align="center">
  <a href="https://github.com/AlesUshakou/sNeuralRender/releases/latest"><img alt="Release" src="https://img.shields.io/github/v/release/AlesUshakou/sNeuralRender?style=flat-square&color=76b900"></a>
  <img alt="Nuke 15.1 | 16.1 | 17.0 | 17.1" src="https://img.shields.io/badge/Nuke-15.1%20%7C%2016.1%20%7C%2017.0%20%7C%2017.1-f26a21?style=flat-square">
  <img alt="Windows x64" src="https://img.shields.io/badge/Windows-x64-0078d4?style=flat-square">
  <img alt="NVIDIA RTX" src="https://img.shields.io/badge/GPU-NVIDIA%20RTX-76b900?style=flat-square">
  <a href="LICENSE"><img alt="MIT" src="https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square"></a>
</p>

<p align="center"><img src="docs/panel.png" alt="sNeuralRender panel in Nuke" width="720"></p>

<p align="center">
  <a href="https://github.com/AlesUshakou/sNeuralRender/releases/latest"><b>⬇ Download the latest release</b></a>
  &nbsp;·&nbsp; <a href="#install">Install</a>
  &nbsp;·&nbsp; <a href="#using-the-node">Using the node</a>
  &nbsp;·&nbsp; <a href="#troubleshooting">Troubleshooting</a>
</p>

A native Nuke plugin that runs NVIDIA's **DLSS 5 Neural Rendering** (NGX feature 18) on your
footage: structure/tone controls, the runtime's automask, a Nuke-style mask input, highlight
recovery for scene-linear footage, and temporal history driven by NVIDIA Optical Flow or your own
motion vectors. `sNeuralRender-vX.Y-win64.zip` covers Nuke 15.1, 16.1, 17.0 and 17.1 on Windows x64.

<p align="center"><img src="docs/compare/wipe_A_01.webp" alt="Curtain: original on the right, DLSS 2x then the neural render on the left" width="800"></p>
<p align="center"><sub>Original vs. DLSS pre-pass ×2 followed by the neural render (model A). &nbsp;
<a href="https://alesushakou.github.io/sNeuralRender/"><b>Open the interactive before / after slider</b></a> — five pairs, models A / B / C, drag the handle.
Frames from <a href="https://peach.blender.org/">Big Buck Bunny</a>, © 2008 Blender Foundation, CC BY 3.0.</sub></p>

> Not affiliated with or endorsed by NVIDIA. The plugin only drives NVIDIA's runtime; the
> runtime itself is NVIDIA software and is **not included** in this download (see below).

## Requirements

| | |
|---|---|
| OS | Windows 10 / 11, x64 |
| Nuke | 15.1, 16.1, 17.0 or 17.1 (Windows builds included; NukeX / Studio work the same) |
| GPU | NVIDIA GPU supported by DLSS 5 Neural Rendering (GeForce RTX 40 / 50 series). **Tested on GeForce RTX 5090 and RTX PRO 6000 Blackwell.** |
| Driver | A recent NVIDIA display driver. Tested with **610.88** (R610). The Neural Rendering runtime itself declares a minimum of 470; on 596.86 it created and evaluated feature 18 in our logs, but the plugin was not tested there. If `runtime info` shows an NGX error, update the driver first. |
| Runtime | **`nvngx_dlssnr.dll`, not included** — NVIDIA's Neural Rendering runtime, tested build 310.8.0. `get_runtime.bat` in the download fetches it (see [Install](#install)); by hand: [docs/RUNTIME.md](docs/RUNTIME.md). |

## Install

> ⚠️ **One file is not in this download: `nvngx_dlssnr.dll`**, NVIDIA's Neural Rendering
> runtime (the neural network itself). It is NVIDIA software, so it is **not redistributed**
> with the plugin, and **the node does nothing without it**. Step 2 fetches it.

1. Download `sNeuralRender-vX.Y-win64.zip` from the
   [latest release](https://github.com/AlesUshakou/sNeuralRender/releases/latest) (not GitHub's
   "Source code" archive, it holds only this README) and unzip it. You get one folder,
   `sNeuralRender`, with this layout:

   ```
   sNeuralRender\
     get_runtime.bat    downloads NVIDIA's runtime into runtime\ (step 2)
     init.py, menu.py   pick the build for your Nuke version, add the Nodes menu entry
     15.1\ 16.1\ 17.0\ 17.1\   sNeuralRender.dll per Nuke version
     core\              dlss5nr_core.dll, nvngx.dll_snr.dll (plugin code, Nuke-independent)
     runtime\           NVIDIA's nvngx_dlssnr.dll lands here; nvngx_dlss.dll optional (DLAA modes)
   ```

2. Double-click **`sNeuralRender\get_runtime.bat`**. It downloads `nvngx_dlssnr.dll` (about
   110 MB; tested build 310.8.0, NVIDIA-signed) and the optional `nvngx_dlss.dll` (30 MB, enables
   the `DLAA` modes) from public GitHub mirrors, checks their SHA-256 and puts them into
   `sNeuralRender\runtime\`. By downloading the files you accept NVIDIA's terms.

   **If the script fails** (no internet on the workstation, GitHub blocked, antivirus): get the
   files by hand, see **[docs/RUNTIME.md](docs/RUNTIME.md)** — the sources, the checksums and
   where to put them.

3. Put the `sNeuralRender` folder somewhere on your Nuke plugin path, usually
   `C:\Users\<you>\.nuke\`, and add to `C:\Users\<you>\.nuke\init.py` (create the file if
   it does not exist):

   ```python
   nuke.pluginAddPath('./sNeuralRender')
   ```

   The folder's own `menu.py` puts the node into the Nodes menu as **Filter > sNeuralRender**;
   that is also what makes the Tab search find it.

4. Restart Nuke. Press Tab in the node graph and type `sNeuralRender`, or pick it from
   Filter > sNeuralRender.

5. Open the node's **System** tab: `runtime info` shows the GPU, the NGX core that was found,
   the runtime path (`...\sNeuralRender\runtime`) and whether NVIDIA Optical Flow is available.
   `runtime not found` means step 2 did not complete. Everything the plugin and the runtime log goes
   to `sNeuralRender\<version>\sNeuralRender.log`.

Studio / pipeline install: any directory on `NUKE_PATH` works the same way; only the
`nuke.pluginAddPath('./sNeuralRender')` line is needed, the folder's own `init.py` picks the
Nuke version. If a build for your Nuke version is missing, the Nuke console says
`sNeuralRender: no build for Nuke X.Y`.

## Using the node

Inputs: `src` (the image; `rgb` is processed, alpha and other channels pass through),
`motion` (optional Nuke `backward` motion vectors) and `mask` (optional). Output: `rgba` plus
`nr_motion.u/v`, the vectors the runtime actually used (pixels).

The panel follows NVIDIA's own "DLSS 5 developer controls":

- **model** A / B / C — the runtime's three styles. **DLSS on** bypasses the node.
- **DLAA** — optional DLSS stage from `nvngx_dlss.dll` (see Install): `pre-pass` runs DLAA
  before the neural render, which is what the DLSS 5 player does and makes the result steadier
  on compressed or noisy footage; `only` gives you DLSS without the neural render. The `scale`
  slider (1.5×, 2×, 3×) turns DLSS into an upscaler, the output format grows; with `pre-pass` the
  neural render then runs on the upscaled frame, so the detail is added at the output resolution.
  Footage has no camera jitter, so DLSS itself resamples rather than invents detail.
- **input colorspace** — the runtime is display-referred SDR (sRGB), so with `linear` (default)
  the node encodes to sRGB before it and decodes after. Use `display` when your footage is
  already video-encoded.
- **GLOBAL CONTROLS** — `structure intensity` / `tone intensity` of the neural pass.
- **HIGHLIGHTS** — `highlight recovery` (default 1) decides what happens to scene values above
  `recovery threshold` (default 1), which the runtime can only see as white. The neural result is
  applied to those pixels as a gain on the original value, so speculars and practicals keep their
  level and colour instead of coming back flat white. Pull recovery down to blend back towards the
  clipped result, or raise the threshold to leave the first stops above white to the runtime.
  Pixels at or below the threshold are untouched.
- **MODEL AUTOMASK** — the runtime's automatic character/skin mask with its own
  `skin structure intensity` (it only has an effect while automask is on).
- **MASK** — the usual Nuke mask: connect anything to the `mask` input, pick the channel
  (alpha by default), `invert` if needed. The neural result is applied only where the mask is;
  outside it the frame is untouched. With nothing connected the mask controls are greyed out and
  the whole frame is processed.
- **mix** — final blend with the source (multiplied by the mask when one is connected).

**Temporal** tab: `temporal` (on by default) keeps the runtime's history across frames, which is
what removes frame-to-frame flicker. Playing or rendering forward continues the history;
jumping to a frame replays up to `history` frames before it (from the last detected cut).
`reset on cut` / `cut threshold` detect cuts automatically. Motion comes from **NVIDIA Optical
Flow** (default), from the `motion` input (SmartVector / VectorGenerator `backward` layer), or is
zero. Finished frames are cached (`cache (MB)`).

## Known limitations

- The runtime is SDR: it expects display-referred sRGB and sees nothing above 1. Scene values
  above `recovery threshold` keep their level and colour through `highlight recovery`, but the
  network itself gets them as flat white, so it adds no new texture inside such an area; what
  comes back there is the original detail, not reconstructed one.
- Runtime 310.8.0 ignores its depth input and has one weight set; the node therefore exposes
  neither depth nor presets (they will come back when a runtime uses them).
- Windows only, Nuke 15.1, 16.1, 17.0 and 17.1 (other versions need a rebuild from source).
- Temporal results depend on the frame order: a Write that starts mid-shot replays `history`
  frames first, and Nuke's own caching does not know about frames before the current one.

## Troubleshooting

| Message / symptom | Meaning |
|---|---|
| `Unknown command sNeuralRender` | The folder is not on the plugin path, or no build for this Nuke version (see the console). |
| `runtime not found` in `runtime info` | `nvngx_dlssnr.dll` is not in `sNeuralRender\runtime\`: run `get_runtime.bat` or fetch it by hand ([docs/RUNTIME.md](docs/RUNTIME.md)); or `runtime dir` on the System tab points elsewhere. |
| NGX errors in `runtime info` (`0xBAD00001`, `feature ... not supported`) | The NGX core of your driver or the runtime build rejects the GPU / driver combination: update the driver (610.88 tested), then check that the GPU is one the runtime supports (RTX 40 / 50, RTX PRO Blackwell). |
| Flicker between frames | Turn `temporal` on (Temporal tab). |
| Slow scrubbing with heavy upstream nodes | Each random access replays `history` frames; lower `history`, or precomp the input. |

## Credits

- NVIDIA DLSS 5 Neural Rendering and NVIDIA Optical Flow — NVIDIA Corporation.
- Runtime integration research: [lisitskyaa/ComfyUI-DLSS5-NR](https://github.com/lisitskyaa/ComfyUI-DLSS5-NR)
  and [2600th/dlss5-video-player](https://github.com/2600th/dlss5-video-player).
- Before / after frames: [Big Buck Bunny](https://peach.blender.org/), © 2008 Blender Foundation /
  www.bigbuckbunny.org, [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/).
- Plugin: Aleš Ushakou, 2026 — [LinkedIn](https://www.linkedin.com/in/ale%C5%A1-ushakou-84250814) · [GitHub](https://github.com/AlesUshakou)

## License

The plugin is MIT licensed (see `LICENSE`). NVIDIA components are NVIDIA software under
NVIDIA's terms and are not part of this download. DLSS is a trademark of NVIDIA Corporation.
