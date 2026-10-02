# Redmi 4A (rolex) adaptation

This fork contains the board adaptation used by Ubuntu 24.04 ARM64 and Phosh on one Redmi 4A with NT35521S EBBG display, FT5346 touch, SMB358 charger, CW2015 fuel gauge, OV13850 rear camera and OV5675 front camera. Hardware variants need separate verification.

The adaptation branch is `rolex`. Its baseline is msm89x7-mainline/linux commit `50f9719b10cef792432485b0139fbcb913316e07` (`v7.1.3-r1`). The kernel is maintained as commits on top of that upstream history. This repository contains kernel source and configuration; Ubuntu rootfs packaging, credentials, firmware dumps and binary images are kept outside it.

## Changes

- Opt-in read-only charger/fuel-gauge diagnostics, including documented DT properties. Existing non-diagnostic driver behavior is retained.
- Touch overlay support in edt-ft5x06, with proper contact/key release. Printed keys outside the LCD map to APPSELECT, HOMEPAGE and BACK.
- Standard V4L2 OV13850 sensor support adapted from Rockchip's GPL-2.0 register programming, retaining its copyright and b1/b2 initialization.
- Separate diagnostic and Ubuntu board DTs: FT5346 selection, original power-chip connections, navigation regions, camera wiring and clock rates.
- GPIO 0..3 reservation to avoid an observed early firmware access abort. This is a workaround, not a confirmed root-cause fix.

## Build

Use an ARM64-capable Clang/LLVM toolchain, lld, make, flex, bison, bc and dtc. The base config is copied unchanged from postmarketOS pmaports commit `721be33f67ba1a98f974760779fb1cb1b1b04661`, path `device/testing/linux-postmarketos-qcom-msm89x7/config-postmarketos-qcom-msm89x7.aarch64`.

```sh
git clone --branch rolex https://github.com/Railgun-wiki/linux-msm89x7.git
cd linux-msm89x7
mkdir -p ../rolex-kernel-build
export ARCH=arm64 LLVM=1 LOCALVERSION=
scripts/kconfig/merge_config.sh -m -O ../rolex-kernel-build \
  rolex/config/pmaports-aarch64.config rolex/config/ubuntu.config
make O="$PWD/../rolex-kernel-build" olddefconfig
make O="$PWD/../rolex-kernel-build" -j4 Image.gz modules dtbs
```

The generic package/repository name is `linux-msm89x7`; this config produces kernel release `7.1.3-msm89x7`. Device selection remains in the DTB. Existing installed kernels keep their original release until a new build is explicitly installed.

The board DT is `qcom/msm8917-xiaomi-rolex-ubuntu.dtb`. Merge `diagnostic.config` instead for the bounded RAM diagnostic environment. Boot packaging and the board-specific lk2nd touch selection are separate local userspace/boot tooling; plain `Image.gz` is not a complete phone boot image.

## Verification and limits

The code in the first six adaptation commits is byte-for-byte the code used for the installed `7.1.3-rolex` kernel. Its full kernel/modules/DTBs built successfully. Physical checks established display/backlight, touch and printed keys, actual Freedreno draws, Wi-Fi, Qualcomm modem discovery, and camera test-pattern/RAW capture. Configuration includes TPROXY modules and removable-card filesystems.

This does not establish complete phone functionality. Charger diagnostics do not implement battery temperature protection or active charging control. Camera autofocus, OTP calibration, auto exposure/white balance and desktop JPEG capture are unfinished. Actual SIM calls/SMS/mobile data/VoLTE, physical microSD operations and TPROXY traffic remain unverified. Audio checks did not establish all acoustic paths. Known platform warnings and the GPIO workaround remain unresolved. No experimental GPU clock divider change is included.

The upstream kernel licensing and per-file SPDX notices remain applicable. See `COPYING` and `LICENSES/`.
