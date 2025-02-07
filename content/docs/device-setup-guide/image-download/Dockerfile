# Copyright (c) 2025 Qualcomm Innovation Center, Inc. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause-Clear

# Pull ubuntu base image
FROM public.ecr.aws/ubuntu/ubuntu:20.04

# Install dependencies
RUN apt-get -y update && apt-get install -y \
  build-essential \
  coreutils \
  checkinstall \
  fakechroot \
  fakeroot \
  kmod \
  libc6-arm64-cross \
  qemu-user-static \
  udev \
  usbutils \
  adb \
  wget \
  unzip \
  ldap-utils \
  python-is-python2 \
  sudo \
  && rm -rf /var/lib/apt/lists/*

# Install qemu-user-static
RUN wget http://archive.ubuntu.com/ubuntu/pool/universe/q/qemu/qemu-user-static_6.2+dfsg-2ubuntu6_amd64.deb \
  && dpkg -i qemu-user-static_6.2+dfsg-2ubuntu6_amd64.deb \
  && rm qemu-user-static_6.2+dfsg-2ubuntu6_amd64.deb

# Download and install sdkmanager
ARG SDKMANAGER_BASE_URL=https://thundercomm.s3.ap-northeast-1.amazonaws.com/uploads/web/common
ARG SDKMANAGER_VERSION=4.2.0
RUN wget ${SDKMANAGER_BASE_URL}/TC-sdkmanager-${SDKMANAGER_VERSION}.zip -O sdkmanager.zip \
  && mkdir sdkmanager \
  && unzip sdkmanager.zip -d sdkmanager \
  && dpkg -i sdkmanager/TC-sdkmanager-${SDKMANAGER_VERSION}/tc-sdkmanager-v${SDKMANAGER_VERSION}_amd64.deb \
  && rm -r sdkmanager sdkmanager.zip \
  && rm -rf /lib/ld-linux-aarch64.so.1 \
  && ln -sf /usr/aarch64-linux-gnu/lib/ld-2.31.so /lib/ld-linux-aarch64.so.1 \
  && ln -sf /bin/bash /bin/sh

# User should run sdkmanager from within the container context
CMD ["/bin/bash"]
