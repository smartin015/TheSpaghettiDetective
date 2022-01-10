# Run TSD server on NVIDIA Jetson Nano

*If you follow this guide and run into problems, please seek help at: https://discord.gg/NcZkQfj*

## Prerequisites

### Hardware requirements

TSD private server can only run on Jetson Nano 4GB model. The 2GB model doesn't have enough memory to run both the program and load the AI model in the memory.

### Software requirements

The following software is required before you start installing the server:

- [JetPack SDK](https://developer.nvidia.com/embedded/jetpack). If you already flashed a software on you sd card, you will have to replace it with this one. Slow download of the software from Nvidia is normal. **Important:** Before you flash the new software on your sd card, you will have to fully format it first, so make sure you have backed up anything important on an external device.
  - [Flashing Software](https://www.balena.io/etcher/)
  - [SD Card Formater](https://www.sdcard.org/downloads/formatter/)

*Note: the last JetPack SDK version this has been tested on is jp45.*

*If you succesfully run this on a newer version, please send a message to the official discord and mention @LyricPants66133*

### Email delivery

You will also need an email account that has SMTP access enabled. For a gmail account, this is [how you enable SMTP access](https://support.google.com/accounts/answer/6010255?hl=en). Other web mail such as Yahoo
should also work but we haven't tried them.

## Get the code and start the server.

1. Get the code:

```
git clone https://github.com/TheSpaghettiDetective/TheSpaghettiDetective.git
```

2. Run it!

```
cd TheSpaghettiDetective
./scripts/install_on_jetson.sh
```

3. Go grab a coffee. Step 2 will take 15-30 minutes.

4. There is no step 4. This is how easy it is to get The Spaghetti Detective up and running (thanks to Docker and Docker-compose).

## Continue to [server configuration the main documentation](../README.md#basic-server-configuration)


# Troubleshooting / Advanced model details

Model configuration is stored in `ml_api/model`, but uses architecture-specific shared objects (`*.so`) in `ml_api/bin` to actually run the model. 
The Jetson uses `model_gpu_aarch64.so`, which is a version of [AlexeyAB's fork of Darknet](https://github.com/AlexeyAB/darknet#how-to-use-yolo-as-dll-and-so-libraries) built
for the Jetson's architecture.

Newer versions of NVIDIA JetPack may break this setup, e.g. by [providing newer versions of CUDART than what this binary was compiled to require](https://github.com/TheSpaghettiDetective/TheSpaghettiDetective/issues/552). This can be resolved by rebuilding Darknet in libdll form and replacing the `*.so` file like so:

```shell
# Add nvcc to path (in .bashrc)
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
git clone https://github.com/AlexeyAB/darknet.git
cd darknet

# Here, set GPU=1 and LIBSO=1 in the file
vim Makefile

make -j$(nproc)

cp libdarknet.so <path_to_spaghetti_detective_repository>/ml_api/bin/model_gpu_aarch64.so

```

If you get errors relating to other missing `*.so` files, you can confirm they're missing dependencies of the new binary by `exec`ing into the running ml_api container and checking with `ldd`:

```shell
$ docker exec -it --tty thespaghettidetective_ml_api_1 /bin/bash

root@ml_api:/app# ldd bin/model_gpu_aarch64.so
	linux-vdso.so.1 (0x0000007fac03c000)
	libcuda.so.1 => /usr/lib/aarch64-linux-gnu/tegra/libcuda.so.1 (0x0000007faac5f000)
	libcudart.so.10.2 => /usr/local/cuda-10.2/targets/aarch64-linux/lib/libcudart.so.10.2 (0x0000007faabbe000)
	libcublas.so.10 => /usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublas.so.10 (0x0000007fa5e56000)
	libcurand.so.10 => /usr/local/cuda-10.2/targets/aarch64-linux/lib/libcurand.so.10 (0x0000007fa1d25000)
	libstdc++.so.6 => /usr/lib/aarch64-linux-gnu/libstdc++.so.6 (0x0000007fa1b91000)
	libm.so.6 => /lib/aarch64-linux-gnu/libm.so.6 (0x0000007fa1ad8000)
	libgcc_s.so.1 => /lib/aarch64-linux-gnu/libgcc_s.so.1 (0x0000007fa1ab4000)
	libpthread.so.0 => /lib/aarch64-linux-gnu/libpthread.so.0 (0x0000007fa1a88000)
	libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000007fa192f000)
	/lib/ld-linux-aarch64.so.1 (0x0000007fac010000)
	libdl.so.2 => /lib/aarch64-linux-gnu/libdl.so.2 (0x0000007fa191a000)
	librt.so.1 => /lib/aarch64-linux-gnu/librt.so.1 (0x0000007fa1903000)
	libnvrm_gpu.so => /usr/lib/aarch64-linux-gnu/tegra/libnvrm_gpu.so (0x0000007fa18bf000)
	libnvrm.so => /usr/lib/aarch64-linux-gnu/tegra/libnvrm.so (0x0000007fa187c000)
	libnvrm_graphics.so => /usr/lib/aarch64-linux-gnu/tegra/libnvrm_graphics.so (0x0000007fa185c000)
	libnvidia-fatbinaryloader.so.440.18 => /usr/lib/aarch64-linux-gnu/tegra/libnvidia-fatbinaryloader.so.440.18 (0x0000007fa17ed000)
	libcublasLt.so.10 => /usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublasLt.so.10 (0x0000007f9f7d7000)
	libnvos.so => /usr/lib/aarch64-linux-gnu/tegra/libnvos.so (0x0000007f9f7b9000)
```

For whatever reason, sometimes the symlinks for a `.so` file aren't populated by nvidia-docker. This has happened with libcuda.so.1 and libnvidia-ptxjitcompiler.so.1 in /usr/lib/aarch64-linux-gnu/tegra of the container, specifically with these errors:

For missing link to libnvidia-ptxjitcompiler.so.1:

```
ml_api_1  |  CUDA Error: PTX JIT compiler library not found

```

For missing link ot `libcuda.so.1`: 

```
ml_api_1  | OSError: libcuda.so.1: cannot open shared object file: No such file or directory
```

 A (hacky) workaround to this is to force create these symlinks before running the `gunicorn` container command, like so:

```
# in docker-compose.yaml
services:
  ml_api:
    ...
    command: bash -c "ln -sf /usr/lib/aarch64-linux-gnu/tegra/libcuda.so /usr/lib/aarch64-linux-gnu/tegra/libcuda.so.1 && ln -s /usr/lib/aarch64-linux-gnu/tegra/libnvidia-ptxjitcompiler.so.440.18 /usr/lib/aarch64-linux-gnu/tegra/libnvidia-ptxjitcompiler.so.1 && gunicorn --bind 0.0.0.0:3333 --workers 1 wsgi"

```

### Modifying the `ml_api` docker images

The ML API container is made up of a base docker image and an additional image that actually installs the app proper. 

To build the base image locally:
```
cd ml_api

docker build --tag thespaghettidetective/ml_api:base-1.1 -f Dockerfile.base_aarch64 .
```

To build the main image locally:
```
docker-compose build ml_api
```


*Thanks to the work of Raymond, LyricPants, and others for their contribution!*
