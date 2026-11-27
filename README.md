# Slicewise

A cross-platform DICOM viewer written in Rust, built on [dicom-rs](https://github.com/Enet4/dicom-rs) and [egui](https://github.com/emilk/egui). Runs as a native app and in the browser.

Slicewise is for viewing, teaching, research and reviewing your own images. It is not a medical device and is not intended for diagnosis or treatment decisions.

The web build runs at <https://poholos.github.io/slicewise-demo/>, and the native app is under [Downloads](#downloads) below.

In the web build, press **Demo** to open a synthetic examination of an invented patient: MR, CT and radiographs, generated in the page. Or open your own with **Open files…**, or with **Open folder…**, which takes a whole folder at once. Files can be dropped on the page as well, but a folder cannot: the browser hands one over with nothing behind it. They stay in your browser; nothing is uploaded anywhere. The native app has the same Demo button, and takes a dropped folder too.

## The demo

The Demo button builds an examination and opens it like a disk: three studies, fifteen series and 93 objects with a DICOMDIR, every one made from a procedural head, so that reference lines, MPR cuts, the segmentation and the measurements agree across series. Nothing in it comes from real data: the model is a set of ellipsoids with a noise-folded cortex, and every value, date and identifier is invented.

The MR study mirrors the structure of a real examination: a survey in three planes, T2, T1 and FLAIR stacks under JPEG Lossless and RLE, a 96-frame isotropic 3D T1, an ADC map with float pixels, a two-segment segmentation, annotated key images under JPEG Baseline, a text report, a measurement report that links back to the slice it was measured on, and a presentation state that windows, flips and annotates the T2 stack. Beside it are a CT head with an overlay plane and a dose grid, and two 5-megapixel skull radiographs.

![The tree of the demo disk in the side panel, with its radiology report open](https://poholos.github.io/slicewise-demo/hero/file_set.png)

![Three-up MPR of the 3D T1 volume, the crosshair on the lesion](https://poholos.github.io/slicewise-demo/hero/mpr.png)

![The lesion measured on a T2 slice, and the measurement report that carries the same diameter and links back to the slice](https://poholos.github.io/slicewise-demo/hero/measurement.png)

![The same slice as its presentation state shows it: brain window, flipped, the lesion circled and labeled](https://poholos.github.io/slicewise-demo/hero/presentation.png)

![The lateral skull radiograph: MONOCHROME1, a VOI LUT, the overlay plane on the lesion and the scale bar](https://poholos.github.io/slicewise-demo/hero/radiograph.png)

![The key image with the segmentation painted over the T2 slice](https://poholos.github.io/slicewise-demo/hero/segmentation.png)

## What it does

- **Opening.** Files, folders and patient disks. A folder with a DICOMDIR opens through it, and so does a folder of several disks side by side: patients, studies and series show at once as a tree, and a series is read when it is clicked. Other folders are scanned and grouped into series ordered along the slice normal. File extensions play no part, so a disk's `S0000001/O0000001` layout opens as it is.
- **Viewing.** Slices by wheel, arrow keys or slider, and cine, at the frame times a multi-frame object gives. Window and level with presets per modality (the Hounsfield windows for CT), invert, flip, rotate, zoom and pan. The hover readout shows the modality value under the cursor, Hounsfield units for CT. The overlay shows patient, study and series, slice position, window, a scale bar and direction letters. Overlay planes, display shutters and supplemental palettes are drawn as the object stores them.
- **Volumes and MPR.** A series stacks into a volume, cut along the axial, coronal and sagittal planes, enhanced CT and MR objects, nuclear medicine reconstructions, dose grids, Siemens mosaics and United Imaging grids included. A series holding several volumes at the same positions, as diffusion and perfusion series do, offers each one, named by its b-value and direction, time point, echo time or segment. Slices from a tilted gantry are shifted back into line. Three-up MPR shows the three planes together, with a crosshair and reference lines.
- **Measurements.** A ruler in millimeters, rectangular and elliptical regions with mean, standard deviation, minimum, maximum and area, and angles.
- **Reports.** Structured reports render as documents, and an item made on an image links to it, with the region drawn there. Grayscale softcopy presentation states apply on request: window or VOI LUT, rotation and flip, displayed area, annotations and shutters.
- **Rendering.** WebGPU where the browser offers it and WebGL 2 otherwise; windowing runs on the GPU on both.

## Formats

Transfer syntaxes: implicit and explicit VR little endian, explicit VR big endian, deflated, RLE, JPEG Baseline, Extended (8 and 12 bit) and Lossless, JPEG-LS lossless and near-lossless, JPEG 2000 lossless and lossy, and HTJ2K. Images: MONOCHROME1 and 2 with 1, 8, 12, 16 and 32 bits allocated, RGB with 8, 16 and 32, YBR_FULL, YBR_FULL_422 (compressed), YBR_RCT and palette color; multi-frame objects; enhanced CT and MR; segmentations; RT Dose; parametric maps with float pixels; data sets with no file meta group or a broken one.

The viewer is checked against public test data, with the desktop build, which also decodes color JPEG-LS. Of the 261 entries in the [pydicom](https://github.com/pydicom/pydicom) test corpus, 164 decode, 68 carry no image (structured reports, plans, waveforms), 9 are not DICOM and 20 fail. The DICOM Working Group 4 compression samples decode bit-identically across every lossless encoding they carry, their 12-bit JPEG Extended files bit-exact against DCMTK's `dcmdjpeg`, and their near-lossless JPEG-LS files bit-exact against DCMTK's `dcmdjpls`.

## Limits

- In the browser, color JPEG-LS streams do not decode; single-component ones do.
- The page has no threads, so frames decode when shown, and a large pick blocks the page briefly.
- A phone's browser cannot pick a whole folder: there **Open folder…** picks files instead, as **Open files…** does.
- RGB and YBR series cannot be stacked into a volume.
- Presentation states other than the grayscale kind are listed but not applied.
- Not read: RLE with 1 bit allocated, uncompressed YBR_FULL_422, Deflated Image Frame Compression, truncated pixel data.
- Names in Chinese, Japanese, Korean, Arabic, Hebrew or Thai draw from a font fetched from the site the first time an object needs it, and Arabic and Hebrew run left to right.

## Downloads

A desktop build for Windows, Linux and macOS comes with each release on the [releases page](https://github.com/Poholos/slicewise-demo/releases), one archive per system. It also reads from a PACS by query and retrieve, which the browser cannot do, having no TCP socket.

| System | Archive |
|---|---|
| Windows, x86-64 | `slicewise-<version>-x86_64-pc-windows-msvc.zip` |
| Linux, x86-64 | `slicewise-<version>-x86_64-unknown-linux-gnu.tar.gz` |
| macOS, Apple silicon and Intel | `slicewise-<version>-universal-apple-darwin.tar.gz` |

Each archive holds the program, a README and the third-party notices, and beside each is its SHA-256 checksum, in a file of the same name ending in `.sha256`. The programs are not signed, which each system meets in its own way:

- **Windows** may stop the program with "Windows protected your PC", as it does any unsigned program from the internet; More info > Run anyway starts it.
- **macOS** keeps a downloaded archive's contents in quarantine and refuses an unsigned program. After extracting, `xattr -d com.apple.quarantine slicewise` lets it run, from the terminal or by a double-click.
- **Linux** needs the X11 or Wayland client libraries and xkbcommon, a Vulkan or OpenGL driver, and for the file dialogs a running xdg-desktop-portal with a backend (GTK, GNOME or KDE). A full desktop has all of them. The program is built on Ubuntu 24.04, so it needs glibc 2.39 or later.

## This repository

This repository holds only the built bundle and the pictures above, which the viewer's CI pushes here after every change to its main branch, replacing the single commit. Its releases hold the desktop builds, one release per version. Nothing here is meant to be edited. The source is not published.

## Terms

The bundle, the pictures above and the desktop builds are provided for trying and showing the viewer. Copyright 2026 Poholos; all rights reserved. They are not released under an open-source license, and no right to copy, modify or redistribute them is granted.

The bundle links open-source components, each under its own license, with their copyright notices and license texts in [third-party notices](https://poholos.github.io/slicewise-demo/third-party.html), which also names the Noto faces the site serves for names in scripts the bundled fonts do not draw. Each desktop archive carries the notices of what its own program links.
