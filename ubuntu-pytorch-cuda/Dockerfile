ARG BASE_IMAGE=registry.cn-hangzhou.aliyuncs.com/troila-klcloud/ubuntu-ai-development:latest
FROM ${BASE_IMAGE}

# args - software versions
# https://github.com/pytorch/pytorch/releases
# https://github.com/pytorch/audio/releases
# https://github.com/pytorch/vision/releases
# NOTE: correct version alignments are found at https://pytorch.org/get-started/previous-versions/
#       they sometimes forget to publish a github release of `torchaudio` or `torchvision` when the cut a
#       patch release of `torch`, but they still publish a patch of `torchaudio` and `torchvision` on pypi
# TODO: it seems like torch 2.6.0+ with CUDA 12.6+ will have experimental aarch64 support
#       when updating past that version, remember to test building these images on arm
ARG PYTORCH_VERSION=2.7.0
ARG TORCHAUDIO_VERSION=2.7.0
ARG TORCHVISION_VERSION=0.22.0

# nvidia container toolkit
# https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/docker-specialized.html
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
ENV NVIDIA_REQUIRE_CUDA "cuda>=12.4"

# install - pytorch (cuda)
RUN pip install --quiet --no-cache-dir --index-url https://download.pytorch.org/whl/cu126 --extra-index-url https://pypi.org/simple \
    torch==${PYTORCH_VERSION} \
    torchaudio==${TORCHAUDIO_VERSION} \
    torchvision==${TORCHVISION_VERSION}
