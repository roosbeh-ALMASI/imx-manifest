# FRDM-i.MX93 Reproducible BSP Manifest

Revision-pinned NXP Yocto BSP for the **FRDM-i.MX93 Rev B2 with 2 GB LPDDR4X**.

## BSP configuration

| Component | Version |
|---|---|
| NXP Linux BSP | 6.18.2-1.0.0 |
| Yocto series | whinlatter |
| Linux kernel | 6.18.2 |
| U-Boot | 2025.04 |
| Machine | `imx93-11x11-lpddr4x-frdm` |
| Distribution | `fsl-imx-xwayland` |
| Image | `imx-image-core` |

The BSP has been built and boot-tested using Ubuntu 22.04 as the build host.

## Repository checkout

Install the Google `repo` tool and the host dependencies required by the NXP Yocto BSP.

Create a new directory:

```bash
mkdir -p ~/linxx/frdm-imx93-repro
cd ~/linxx/frdm-imx93-repro
```

Initialize the custom manifest:

```bash
repo init \
  -u https://github.com/roosbeh-ALMASI/imx-manifest.git \
  -b frdm-imx93-lf-6.18.2 \
  -m frdm-imx93/frdm-imx93-6.18.2.xml
```

Synchronize the pinned repositories:

```bash
repo sync -j4
```

The manifest pins the upstream NXP, Yocto, and OpenEmbedded repositories and the custom `meta-imx93-frdm` layer to specific Git commits.

## Build configuration

From the BSP root:

```bash
MACHINE=imx93-11x11-lpddr4x-frdm \
DISTRO=fsl-imx-xwayland \
source imx-setup-release.sh -b build-frdm
```

Accept the NXP EULA when prompted.

Configure the custom layer:

```bash
bash sources/meta-imx93-frdm/tools/setup-frdm.sh \
  "$PWD/build-frdm"
```

The setup helper registers the custom layer and applies the required package and machine configuration.

For hosts with approximately 30 GiB RAM, the following settings were used to avoid memory exhaustion during compilation. Add them to `build-frdm/conf/local.conf`:

```bitbake
BB_NUMBER_THREADS = "4"
PARALLEL_MAKE = "-j 4"
PARALLEL_MAKEINST = "-j 4"
```

Enter the build environment if it is not already active:

```bash
source setup-environment build-frdm
```

Build the image:

```bash
bitbake imx-image-core
```

## Flashing to microSD with UUU

The generated images are located in:

```text
build-frdm/tmp/deploy/images/imx93-11x11-lpddr4x-frdm/
```

Connect the FRDM-i.MX93 through its USB ROM download interface and place it in Serial Download Mode.

Using a compatible NXP UUU utility, flash the bootloader and compressed WIC image:

```bash
sudo uuu -b sd_all \
  imx-boot-imx93-11x11-lpddr4x-frdm-sd.bin-flash_singleboot \
  imx-image-core-imx93-11x11-lpddr4x-frdm.rootfs.wic.zst
```

Run the command from the image deployment directory. It overwrites the target microSD card.

The tested UUU version was `1.5.243`.

## Custom GitHub repositories

The project uses four custom repositories:

1. `roosbeh-ALMASI/imx-manifest` — pinned Yocto BSP manifest.
2. `roosbeh-ALMASI/meta-imx93-frdm` — custom Yocto layer and build fixes.
3. `roosbeh-ALMASI/linux-imx` — pinned Linux kernel source snapshot.
4. `roosbeh-ALMASI/uboot-imx` — pinned U-Boot source snapshot.

All other BSP repositories are fetched from their original upstream repositories.

The custom layer contains BitBake append files that redirect the Linux kernel, Linux headers, and U-Boot recipes to the pinned custom source repositories.

## Validation status

**Verified:**

- Custom manifest initialization and repository synchronization.
- Custom layer registration and recipe source overrides.
- Linux kernel 6.18.2 compilation.
- U-Boot 2025.04 compilation.
- Complete `imx-image-core` build.
- SD-card flashing with NXP UUU.
- Successful boot on FRDM-i.MX93 Rev B2 with 2 GB LPDDR4X.
- Running Linux kernel revision `6.18.2-1.0.0-g725bbfc77d81`.
- SD-card root filesystem mounted from `/dev/mmcblk1p2`.

**Remaining verification:**

A fully cache-independent build on a clean host has not yet been completed. The successful reproduction build used verified download archives from an existing BSP cache to recover from upstream archive download/checksum failures.

The initial build also required reduced compilation parallelism to avoid host memory exhaustion.

The successful build and hardware boot demonstrate that the pinned source checkout produces a functional BSP, but they do not yet establish complete independence from previously downloaded source archives.
