# FRDM-i.MX93 Reproducible BSP Manifest

Revision-pinned Yocto BSP manifest for the NXP FRDM-i.MX93
Rev B2 board with 2 GB LPDDR4X.

## BSP configuration

- NXP BSP: 6.18.2-1.0.0
- Yocto series: whinlatter
- Machine: imx93-11x11-lpddr4x-frdm
- Distribution: fsl-imx-xwayland
- Image: imx-image-core
- Host used for the initial successful build: Ubuntu 22.04

## Repository checkout

Install Google's repo tool and the host dependencies required by
the NXP BSP.

Create a new BSP directory:

```bash
mkdir -p ~/linxx/frdm-imx93-repro
cd ~/linxx/frdm-imx93-repro
```

Initialize the manifest:

```bash
repo init \
  -u https://github.com/roosbeh-ALMASI/imx-manifest.git \
  -b frdm-imx93-lf-6.18.2 \
  -m frdm-imx93/frdm-imx93-6.18.2.xml
```

Synchronize the repositories:

```bash
repo sync -j4
```

The manifest pins the NXP/Yocto repositories and the custom
meta-imx93-frdm layer to specific Git commits.

## Build configuration

Initialize the NXP build environment using the supplied
imx-setup-release.sh script, selecting:

- Machine: imx93-11x11-lpddr4x-frdm
- Distribution: fsl-imx-xwayland
- Build directory: build-frdm

Then run the custom configuration helper:

```bash
./sources/meta-imx93-frdm/tools/setup-frdm.sh "$PWD/build-frdm"
```

Enter the build environment:

```bash
source setup-environment build-frdm
```

Build the image:

```bash
bitbake imx-image-core
```

## Source snapshots

Linux:
https://github.com/roosbeh-ALMASI/linux-imx

U-Boot:
https://github.com/roosbeh-ALMASI/uboot-imx

Custom Yocto layer:
https://github.com/roosbeh-ALMASI/meta-imx93-frdm

The custom layer redirects the Linux kernel, Linux headers,
and U-Boot recipes to the corresponding pinned source snapshots.

## Validation status

The original NXP BSP with the custom build fixes has been
successfully built and booted from microSD.

The revision-pinned manifest has been checked for valid XML
and verified against the original NXP manifest.

A fresh repo sync and full image build using this custom
manifest have not yet been completed.
