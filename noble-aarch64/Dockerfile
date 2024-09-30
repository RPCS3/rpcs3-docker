FROM arm64v8/ubuntu:24.04
LABEL maintainer="kd-11"

ENV LLVMVER=18

ENV CLANG_BINARY=clang-${LLVMVER}
ENV CLANGXX_BINARY=clang++-${LLVMVER}
ENV LLD_BINARY=lld-${LLVMVER}

ENV GCCVER=13
ENV GCC_BINARY=gcc-${GCCVER}
ENV GXX_BINARY=g++-${GCCVER}

# Base packages
RUN \
        apt-get update -y && \
        apt -y update && \
        apt -y install \
                        git \
                        cmake \
                        curl \
                        libfreetype6 \
                        ninja-build \
                        shellcheck \
                        xz-utils \
                        openssl \
                        \
                        gcc=4:${GCCVER}.\* \
                        g++=4:${GCCVER}.\* \
                        clang=1:${LLVMVER}.\* \
                        llvm=1:${LLVMVER}.\* \
                        libclang-dev=1:${LLVMVER}.\* \
                        \
                        libavcodec-dev=7:6.1\* \
                        libavdevice-dev=7:6.1\* \
                        libavformat-dev=7:6.1\* \
                        libavfilter-dev=7:6.1\* \
                        libavutil-dev=7:6.1\* \
                        libpostproc-dev=7:6.1\* \
                        libswscale-dev=7:6.1\* \
                        libswresample-dev=7:6.1\* \
                        \
                        libasound2-dev \
                        libpulse-dev \
                        libopenal-dev \
                        libfreetype6-dev \
                        libfontconfig-dev \
                        libglew-dev \
                        zlib1g-dev \
                        libedit-dev \
                        libudev-dev \
                        libevdev-dev \
                        libsdl2-dev=2.30\* \
                        libssl-dev \
                        libjack-dev \
                        libsndio-dev \
                        libcurl4-gnutls-dev \
                        libusb-dev \
                        libvulkan-dev \
                        libzstd-dev

# For QT build we need some unknown X11/XCB deps. Just install everything by prefix
RUN \
        apt -y install \
                libx11-* \
                libxcb-* \
                libxkb*

ENV FFMPEG_NV_CODEC_VER=12.1.14.0
RUN \
        curl -sSfLO https://github.com/FFmpeg/nv-codec-headers/releases/download/n${FFMPEG_NV_CODEC_VER}/nv-codec-headers-${FFMPEG_NV_CODEC_VER}.tar.gz && \
        tar -xf ./nv-codec-headers-*.tar.gz && \
        cd nv-codec-headers-* && \
        make && \
        PREFIX=/usr make install && \
        cd ../.. && \
        rm ./*.tar.gz && \
        rm -r ./nv-codec-headers-*

ENV QTVER=6.7.3
ENV QTVERMIN=66
RUN \
        curl -sSfL -o qt.tar.xz https://download.qt.io/official_releases/qt/${QTVER%.*}/$QTVER/single/qt-everywhere-src-${QTVER}.tar.xz && \
        tar -xf qt.tar.xz && \
        cd  qt-everywhere-src-* && \
        mkdir build && cd build && \
        cmake ../ -G Ninja \
                        -DCMAKE_BUILD_TYPE=MinSizeRel \
                        -DCMAKE_C_COMPILER=$CLANG_BINARY \
                        -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
                        -DFEATURE_ffmpeg=ON \
                        -DFEATURE_gstreamer=OFF \
                        -DFEATURE_fontconfig=ON \
                        -DFEATURE_system_freetype=ON \
                        -DFEATURE_system_xcb_xinput=ON \
                        -DFEATURE_openssl_linked=ON \
                        -DCMAKE_INSTALL_PREFIX=/usr && \
        echo "Need to cleanup for latest GCC" && \
        sed -i 's/std::uint32_t/uint32_t/' ../qt3d/src/3rdparty/assimp/src/code/AssetLib/FBX/FBXBinaryTokenizer.cpp && \
        ninja && \
        ninja install && \
        cd ../../ && \
        rm -r ./qt-everywhere-src-* && \
        rm qt.tar.xz

# Extra packages required for deployment. These are only needed by the deploy script.
RUN \
        apt -y install \
                file
