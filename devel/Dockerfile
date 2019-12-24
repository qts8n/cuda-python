FROM nvidia/cuda:10.2-cudnn7-devel-ubuntu18.04
LABEL maintainer="qts8n <efisher9680@gmail.com>"

# nvidia cuda 10.2 paths
ENV LD_LIBRARY_PATH=/usr/local/cuda-10.2/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
ENV LIBRARY_PATH=${LIBRARY_PATH}:/usr/local/cuda-10.2/lib64

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

ENV LANG C.UTF-8

# ensure annoying packages like `tzdata` won't ask a lot of questions
ARG DEBIAN_FRONTEND=noninteractive

# runtime dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates curl netbase && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get install -y build-essential

