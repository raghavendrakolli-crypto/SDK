# Build the Cooper SDK 2.5.4 (CV75 PTPD EVK) Using Docker

This repository contains a Docker image with everything needed to build the
Ambarella Cooper Linux SDK 2.5.4 (cv7x) firmware for the CV75 PTPD EVK. You do not
need to install the Ambarella toolchain on your machine.

**Image tag:** `cooper-sdk-build:2.5.4` (Ubuntu 22.04, Cortex-A76 GCC 14.3 toolchain,
gcc-arm-none-eabi 10.3, makedevs, and all required packages)

> The image contains Ambarella vendor software. Use it only if you are covered by
> the Ambarella license / NDA, and do not share it outside the team.

## What you need

- A Linux PC with Docker (`sudo apt install docker.io`) and `sudo` access
- Your own copy of the SDK folder `cooper_linux_sdk_2.5.4_cv7x`
- About 60 GB free disk space and internet access for the first build

## Files in this repository

| File | Purpose |
|---|---|
| `cooper-sdk-build_2.5.4.tar.gz` | Docker image (about 2 GB) |
| `cooper-sdk-build_2.5.4.tar.gz.sha256` | Checksum to verify the image file |
| `README.md` | These instructions |

## Step 1: Verify and load the image

```bash
sha256sum -c cooper-sdk-build_2.5.4.tar.gz.sha256
gunzip -c cooper-sdk-build_2.5.4.tar.gz | sudo docker load
sudo docker images | grep cooper-sdk-build
```

The last command should list `cooper-sdk-build 2.5.4`.

## Step 2: Start the container

Set the path of your SDK and make sure you own the files:

```bash
export SDK_DIR="$HOME/cctv/cooper_linux_sdk_2.5.4_cv7x"    # change to your path
sudo chown -R $(id -u):$(id -g) "$SDK_DIR"
```

Start the container. Your SDK folder is mounted at the same path inside it:

```bash
sudo docker run --rm -it \
  --user $(id -u):$(id -g) \
  -v "$SDK_DIR:$SDK_DIR" \
  -w "$SDK_DIR/ambarella" \
  cooper-sdk-build:2.5.4
```

The prompt changes to something like `youruser@5955238df7bd:...$`. That means you
are inside the container.

## Step 3: Build the firmware (inside the container)

```bash
source build/env/amba-build.env      # choose cv75_ptpd from the menu
make ipcam_emmc_lp5_config
make -j$(nproc)
```

## Step 4: Check the output

```bash
find . -name "bst_bld_env_dtb_kernel_rootfs.bin"
```

The firmware is at
`cooper_linux_sdk_2.5.4_cv7x/ambarella/out/amba_out/cv75_ptpd/firmware`.
Because the SDK folder is mounted, the files are also on your PC after you leave
the container (`exit`). Flash the CV75 PTPD EVK with the standard Ambarella
flashing procedure.

## Tips

- Run `docker` commands on your PC, not inside the container. Inside the container
  there is no `docker`, and `sudo` asks for a password that does not exist.
- The container is temporary (`--rm`). Your build output stays in the SDK folder.
- If a build step needs loop devices or qemu, add `--privileged` to `docker run`.
- If an old `out/` folder was built outside Docker and the build misbehaves, run
  `mv out out_backup` and build clean.

## Troubleshooting

| Problem | Fix |
|---|---|
| `sudo: Sorry, try again` | You are inside the container. Type `exit`, then run the command on your PC. |
| `docker: command not found` | Install Docker on your PC. |
| Permission denied in the SDK folder | Run the `chown` command from Step 2 again. |
| Checksum does not match | Download the image file again. |
| `make` stops with an error | Send the last 30 lines of the output to the maintainer. |
