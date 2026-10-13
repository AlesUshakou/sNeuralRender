# Getting the NVIDIA runtime by hand

`sNeuralRender\get_runtime.bat` normally does all of this for you. This page is for the case
where the script cannot run (no internet on the workstation, GitHub blocked, antivirus, a
locked-down Windows without `curl.exe` / `certutil.exe`): it lists the files, where they come
from, their checksums and where to put them.

The files are NVIDIA software under NVIDIA's terms. They are not redistributed with the plugin.

## The files

| File | Needed | What it is | Size |
|---|---|---|---|
| `nvngx_dlssnr.dll` | **yes** | NVIDIA DLSS 5 Neural Rendering runtime: the neural network itself. The node does nothing without it. | ~165 MB |
| `nvngx_dlss.dll` | optional | NVIDIA DLSS Super Resolution. Enables the node's `DLAA` modes (pre-pass, upscale). | ~59 MB |

Put them here, file names unchanged:

```
sNeuralRender\runtime\nvngx_dlssnr.dll
sNeuralRender\runtime\nvngx_dlss.dll        (optional)
```

or point `runtime dir` on the node's System tab at any folder that contains them.

The two other NVIDIA parts, the NGX core `_nvngx.dll` and NVIDIA Optical Flow `nvofapi64.dll`,
come with your display driver. There is nothing to copy for them.

## Known builds

The plugin was developed and tested with Neural Rendering build **310.8.0**. All builds below
produced bit-identical output on our RTX PRO 6000; `get_runtime.bat` accepts any of them.

### `nvngx_dlssnr.dll`

| Build | Signature | SHA-256 | Where |
|---|---|---|---|
| 310.8.0 | NVIDIA, valid | `e16bcf15e16e13f527491cdf7845b2fe6521a738d8f7c9c721866a8496e1fc8e` | [RankFTW/rhi-repo `dlssnr-310.8.0`](https://github.com/RankFTW/rhi-repo/releases/tag/dlssnr-310.8.0) — `nvngx_dlssnr_310.8.0.zip`, the DLL alone (109 MB zip). **Preferred.** |
| 310.8.SF.0 (community build) | none | `6eb209e764f39872625debd6abaf45e2bb6322f6f270f781f70c059ae30b3927` | [DLSS 5 AIO](https://github.com/SenjuWoo/dlss5-aio) release archive, `02-DLSS5-Neural-Rendering\nvngx_dlssnr.dll` (the archive is a split `.7z.001/.002/.003`, 623 MB, needs [7-Zip](https://www.7-zip.org/)); [DLSS 5 video player](https://github.com/2600th/dlss5-video-player) release zip (395 MB), `neural-runtime\nvngx_dlssnr.dll`; [RankFTW/rhi-repo `dlssnr-310.8.SF-v2`](https://github.com/RankFTW/rhi-repo/releases/tag/dlssnr-310.8.SF-v2). |
| 310.8.0 (player v0.14 copy) | NVIDIA, hash mismatch (patched) | `4b8d19bc3eff58a084f5eca7489c921501c203450169fb82ff4f649a4482ba05` | Older DLSS 5 video player packages. This is the file the plugin was developed on. |

The `rhi-repo` zips are the download backend of [RHI](https://github.com/RankFTW/RHI) (ReShade
HDR Installer); its `dlss_manifest.json` lists them.

### `nvngx_dlss.dll` (optional)

| Build | SHA-256 | Where |
|---|---|---|
| 310.8.0 | `c85f971ce023c9f3492fc7455f0b01a24ba18ea39636407a846902c4360b0b7e` | [RankFTW/rhi-repo `dlss-310.8.0`](https://github.com/RankFTW/rhi-repo/releases/tag/dlss-310.8.0) — `nvngx_dlss_310.8.0.zip`. Tested build. |
| 310.9.1 | `3975567b8943c53acce397f2b72380092f84f162d00b0d2c7d08a1025c563983` | [NVIDIA DLSS SDK on GitHub](https://github.com/NVIDIA/DLSS/tree/main/lib/Windows_x86_64/rel), `lib/Windows_x86_64/rel/nvngx_dlss.dll`; DLSS 5 AIO `01-Official-NVIDIA-DLLs`; DLSS 5 video player package. |

Both are signed by NVIDIA. Any 310.x build should work for the DLAA modes.

## Steps

1. Download one of the sources above and take the DLL out of it (the `rhi-repo` zips hold the
   single file; unzip with Explorer, 7-Zip or `tar -xf <zip>` in a command prompt).
2. Check the file, in a command prompt:

   ```
   certutil -hashfile nvngx_dlssnr.dll SHA256
   ```

   The hash must match one of the rows above. A different hash means a different build: it may
   work, but it is not one the plugin was tested with.
3. Copy the file into `sNeuralRender\runtime\`.
4. Start Nuke, create an `sNeuralRender` node, open the **System** tab: `runtime info` shows the
   runtime path and `NR snippet: ...\runtime\nvngx_dlssnr.dll`; `DLSS (DLAA): nvngx_dlss.dll
   available` when the optional file is there. `runtime not found` means the file is missing or
   misnamed.

## What `get_runtime.bat` does

For the record, so that you can repeat it by hand or audit it: it tries the sources in this
order, stops at the first file whose SHA-256 is in the tables above, and deletes anything else.

```
nvngx_dlssnr.dll
  1. https://github.com/RankFTW/rhi-repo/releases/download/dlssnr-310.8.0/nvngx_dlssnr_310.8.0.zip
  2. https://github.com/RankFTW/rhi-repo/releases/download/dlssnr-310.8.SF-v2/nvngx_dlssnr_310.8.SF-v2.zip
  3. https://github.com/2600th/dlss5-video-player/releases/download/dlss5-video-player-v0.28.2/dlss5-video-player-v0.28.2-win64.zip
     (entry DLSSVideoPlayer-v0.28.2-win64/neural-runtime/nvngx_dlssnr.dll)
nvngx_dlss.dll
  1. https://github.com/RankFTW/rhi-repo/releases/download/dlss-310.8.0/nvngx_dlss_310.8.0.zip
  2. https://raw.githubusercontent.com/NVIDIA/DLSS/main/lib/Windows_x86_64/rel/nvngx_dlss.dll
```

It uses `curl.exe`, `tar.exe` and `certutil.exe`, all part of Windows 10 (1803+) and Windows 11,
and falls back to PowerShell for the download and the unzip. Options: `/nodlss` skips the
optional file, `/force` replaces a file that is already there.
