ARG BASE_IMAGE=registry.cn-hangzhou.aliyuncs.com/troila-klcloud/ubuntu-ai-development:latest

FROM ${BASE_IMAGE}

ARG CUDA_VERSION=12.8.1
ARG CUDNN_VERSION=9.10.0

RUN conda install -y -q \
    cuda=${CUDA_VERSION} \
    cudnn=${CUDNN_VERSION} \
    && conda clean -a -f -y

