FROM thespaghettidetective/ml_api:base-1.1
WORKDIR /app
EXPOSE 3333

# Some images by NVIDIA omit symlinks to certain ML shared libraries that are required by darknet - this adds them in at runtime.
# Details at https://github.com/TheSpaghettiDetective/TheSpaghettiDetective/issues/552
RUN echo "ln -sf /usr/lib/aarch64-linux-gnu/tegra/libcuda.so /usr/lib/aarch64-linux-gnu/tegra/libcuda.so.1 && ln -sf /usr/lib/aarch64-linux-gnu/tegra/libnvidia-ptxjitcompiler.so.440.18 /usr/lib/aarch64-linux-gnu/tegra/libnvidia-ptxjitcompiler.so.1 && ln -sf /usr/local/cuda-10.2/targets/aarch64-linux/lib/libcudart.so.10.2 /usr/lib/aarch64-linux-gnu/tegra/libcudart.so.10.2 && ln -sf /usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublas.so.10 /usr/lib/aarch64-linux-gnu/tegra/libcublas.so.10 && ln -sf /usr/local/cuda-10.2/targets/aarch64-linux/lib/libcurand.so.10 /usr/lib/aarch64-linux-gnu/tegra/libcurand.so.10 && ln -sf /usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublasLt.so.10 /usr/lib/aarch64-linux-gnu/tegra/libcublasLt.so.10" >> ~/.bashrc

# LD_LIBRARY_PATH may not get properly set
RUN echo "LD_LIBRARY_PATH=/usr/lib/aarch64-linux-gnu/tegra/${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}" >> ~/.bashrc

ADD . /app
RUN wget --quiet -O model/model.weights $(cat model/model.weights.url | tr -d '\r')
RUN pip install --upgrade pip && pip install -r requirements.txt
SHELL ["/bin/bash", "-c"]
