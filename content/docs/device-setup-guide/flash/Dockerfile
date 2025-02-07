# Copyright (c) 2025 Qualcomm Innovation Center, Inc. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause-Clear

# Pull ubuntu base image
FROM public.ecr.aws/ubuntu/ubuntu:latest

# Install dependencies
RUN apt-get -y update && apt-get install -y \
  build-essential \
  git \
  libxml2-dev \
  libusb-1.0-0-dev \
  pkg-config \
  && rm -rf /var/lib/apt/lists/*

RUN git clone --depth 1 --branch v2.0 https://github.com/linux-msm/qdl \
  && cd qdl \
  && make \
  && make install

WORKDIR /build
CMD ["/bin/sh", "-c", "qdl ./prog_firehose_ddr.elf rawprogram*.xml patch*.xml"]
