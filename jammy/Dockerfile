FROM ubuntu:22.04
LABEL maintainer="hcorion"

ENV LLVMVER=20

ENV CLANG_BINARY=clang-${LLVMVER}
ENV CLANGXX_BINARY=clang++-${LLVMVER}
ENV LLD_BINARY=lld-${LLVMVER}

ENV GCCVER=13
ENV GCC_BINARY=gcc-${GCCVER}
ENV GXX_BINARY=g++-${GCCVER}

ARG DEBIAN_FRONTEND=noninteractive

RUN \
	apt-get update -y && \
	apt-get install -y curl libcurl4-openssl-dev software-properties-common apt-transport-https && \
	curl -Lvs https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add - && \
	add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
	add-apt-repository -y "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-${LLVMVER} main" && \
	apt-get update -y --fix-missing && \
	\
	apt-get install -y \
	$GCC_BINARY $GXX_BINARY \
	build-essential binutils git ccache bc $CLANG_BINARY $CLANGXX_BINARY $LLD_BINARY \
	libasound2-dev libopenal-dev freeglut3-dev libudev-dev libevdev-dev libpulse-dev libxrandr-dev libglvnd-dev \
	'^libxcb.*-dev' libxcb-cursor0 libx11-xcb-dev libxcb-util-dev libdrm-dev libdbus-1-dev \
	libssl-dev libfontconfig1-dev libv4l-dev yasm libgmp-dev libmp3lame-dev libopenjp2-7-dev libjack-dev \
	libtheora-dev libwebp-dev libopus-dev libvorbis-dev libvpx-dev libx264-dev libxml2-dev ocl-icd-opencl-dev \
	libwayland-dev libwayland-cursor0 libxkbcommon-x11-dev libwayland-client0 libwayland-server0 \
	libpng-dev libjpeg-turbo8-dev libtiff-dev libwebp-dev \
	desktop-file-utils dput gnupg pbuilder ubuntu-dev-tools apt-file patchelf libgtest-dev python3 python3-venv python3-pip libsystemd-dev && \
	apt-get install --only-upgrade dpkg && \
	apt-get clean autoclean && \
	apt-get autoremove --yes && \
	rm -rf /var/lib/apt

RUN \
	curl -sSfLO http://http.us.debian.org/debian/pool/main/g/glew/libglew2.2_2.2.0-4+b1_amd64.deb && \
	curl -sSfLO http://http.us.debian.org/debian/pool/main/g/glew/libglew-dev_2.2.0-4+b1_amd64.deb && \
	dpkg -i ./*.deb && \
	rm ./*.deb

# Setup newer CMake
ENV CMAKEVER=3.28.6
RUN \
	curl -sSfLO https://github.com/Kitware/CMake/releases/download/v${CMAKEVER}/cmake-${CMAKEVER}-linux-x86_64.sh && \
	sh cmake-${CMAKEVER}-linux-x86_64.sh --prefix=/usr --skip-license && \
	rm ./cmake*.sh && \
	cmake --version

# We gotta build our own version of Ninja as well
ENV NINJAVER=1.12.1
RUN \
	curl -sSfL -o ninja.tar.gz https://github.com/ninja-build/ninja/archive/v${NINJAVER}.tar.gz && \
	tar -xf ninja.tar.gz && \
	cd ninja-* && \
	cmake -Bbuild-cmake -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY -H. && \
	cmake --build build-cmake && \
	cp ./build-cmake/ninja /usr/bin/ninja && \
	cd ../ && \
	rm -r ./ninja-* && \
	rm ./*.tar.gz

ENV STATIC_LLVM_VER=19.1.7
RUN \
	curl -sSfLO https://github.com/llvm/llvm-project/releases/download/llvmorg-${STATIC_LLVM_VER}/llvm-project-${STATIC_LLVM_VER}.src.tar.xz && \
	tar -xf llvm-*.tar.xz && \
	cd llvm-*/llvm && \
	mkdir build && cd build && \
	CXXFLAGS="-msse -msse2" cmake .. -G Ninja -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY -DCMAKE_INSTALL_PREFIX=/opt/llvm  \
	-DLLVM_ENABLE_CXX1Y=ON -DLLVM_TARGETS_TO_BUILD="X86" -DCMAKE_BUILD_TYPE=Release -DLLVM_USE_INTEL_JITEVENTS=ON -DLLVM_USE_PERF=ON \
	-DLLVM_BUILD_RUNTIME=OFF -DLLVM_BUILD_TOOLS=OFF -DLLVM_INCLUDE_DOCS=OFF -DLLVM_INCLUDE_EXAMPLES=OFF -DLLVM_INCLUDE_TESTS=OFF -DLLVM_INCLUDE_TOOLS=OFF \
	-DLLVM_INCLUDE_UTILS=OFF -DLLVM_INCLUDE_BENCHMARKS=OFF -DWITH_POLLY=OFF -DLLVM_ENABLE_Z3_SOLVER=OFF && \
	ninja && \
	ninja install && \
	cd ../../.. && \
	rm ./*.tar.xz && \
	rm -r llvm-*

# Need to manually build vulkan headers and libvulkan
ENV VULKAN_HEADER_VER=1.3.268.0
ENV VULKAN_LOADER_VER=1.3.268.0
RUN \
	curl -sSfL -o Vulkan-Headers.tar.gz https://github.com/KhronosGroup/Vulkan-Headers/archive/vulkan-sdk-${VULKAN_HEADER_VER}.tar.gz && \
	tar -xf Vulkan-Headers*.tar.gz && \
	cd Vulkan-Headers* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$GCC_BINARY -DCMAKE_CXX_COMPILER=$GXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./Vulkan-Headers*
RUN \
	curl -sSfL -o Vulkan-Loader.tar.gz https://github.com/KhronosGroup/Vulkan-Loader/archive/vulkan-sdk-${VULKAN_LOADER_VER}.tar.gz && \
	tar -xf Vulkan-Loader.tar.gz && \
	cd Vulkan-Loader* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$GCC_BINARY -DCMAKE_CXX_COMPILER=$GXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=/usr/lib -DCMAKE_INSTALL_SYSCONFDIR=/etc -DCMAKE_INSTALL_DATADIR=/share && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./Vulkan-Loader*

ENV FFMPEG_NV_CODEC_VER=13.0.19.0
RUN \
	curl -sSfLO https://github.com/FFmpeg/nv-codec-headers/releases/download/n${FFMPEG_NV_CODEC_VER}/nv-codec-headers-${FFMPEG_NV_CODEC_VER}.tar.gz && \
	tar -xf ./nv-codec-headers-*.tar.gz && \
	cd nv-codec-headers-* && \
	make && \
	PREFIX=/usr make install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./nv-codec-headers-*

ENV FFMPEGVER=7.1.1
RUN \
	curl -sSfLO https://ffmpeg.org/releases/ffmpeg-${FFMPEGVER}.tar.xz && \
	tar -xf ./ffmpeg-*.tar.xz && \
	cd ffmpeg-* && \
	CFLAGS=-Os CXXFLAGS=-Os ./configure \
	--prefix=/usr --disable-debug --disable-static --disable-stripping --enable-fontconfig --enable-gmp \
	--enable-gpl \
	--enable-libfreetype --enable-libjack --enable-libmp3lame \
	--enable-libopenjpeg --enable-libopus --enable-libpulse \
	--enable-libtheora --enable-libv4l2 \
	--enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx264 --enable-libxcb --enable-libxml2 \
	--enable-nvdec --enable-nvenc --enable-ffnvcodec --enable-opencl --enable-opengl --enable-shared --enable-version3 && \
	make -j$(nproc) && \
	PREFIX=/usr make install && \
	cd ../.. && \
	rm ./*.tar.xz && \
	rm -r ./ffmpeg-*

ENV QTVER=6.9.3
RUN \
	curl -sSfL -o qt.tar.xz https://mirrors.ocf.berkeley.edu/qt/official_releases/qt/${QTVER%.*}/$QTVER/single/qt-everywhere-src-${QTVER}.tar.xz && \
	tar -xf qt.tar.xz && \
	cd  qt-everywhere-src-* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=MinSizeRel -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY -DFEATURE_ffmpeg=ON -DFEATURE_gstreamer=OFF -DFEATURE_fontconfig=ON -DFEATURE_system_freetype=ON -DFEATURE_system_xcb_xinput=ON -DFEATURE_openssl_linked=ON -DFEATURE_gds=OFF -DBUILD_qtwebview=OFF -DBUILD_qtwebchannel=OFF -DBUILD_qtwebengine=OFF -DBUILD_qtquick3d=OFF -DBUILD_qtgraphs=OFF -DBUILD_qtquick3dphysics=OFF -DBUILD_qtvirtualkeyboard=OFF -DBUILD_qt3d=OFF \
	-DCMAKE_INSTALL_PREFIX=/usr && \
	echo "Need to cleanup for latest GCC/Clang" && \
	sed -i 's/<< line.v0//' ../qtquick3d/src/3rdparty/embree/kernels/geometry/pointi.h && \
	sed -i 's/<< a.u.lower//' ../qtquick3d/src/3rdparty/embree/kernels/subdiv/bezier_curve.h && \
	sed -i 's/<< a.u.upper//' ../qtquick3d/src/3rdparty/embree/kernels/subdiv/bezier_curve.h && \
	ninja && \
	ninja install && \
	cd ../../ && \
	rm -r ./qt-everywhere-src-* && \
	rm qt.tar.xz

ENV SHELLCHECKVER=0.11.0
RUN \
	curl -sSfL -o shellcheck.tar.gz https://github.com/koalaman/shellcheck/releases/download/v${SHELLCHECKVER}/shellcheck-v${SHELLCHECKVER}.linux.x86_64.tar.xz && \
	tar -xf shellcheck.tar.gz && \
	cd shellcheck* && \
	cp ./shellcheck /usr/bin/ && \
	cd .. && \
	rm -r ./shellcheck* && \
	shellcheck --version

# We need to build a custom version of SDL3 as well
ENV SDLVER=3.2.22
RUN \
	curl -sSfL -o SDL.tar.gz https://github.com/libsdl-org/SDL/releases/download/release-${SDLVER}/SDL3-${SDLVER}.tar.gz && \
	tar -xf SDL.tar.gz && \
	cd SDL3* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr -DSDL_DLOPEN=ON -DARTS=OFF -DESD=OFF -DNAS=OFF -DALSA=ON -DPULSEAUDIO_SHARED=ON -DRPATH=OFF -DCLOCK_GETTIME=ON && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./SDL*

ENV OPENCV_VER=4.11.0
RUN \
	curl -sSfL -o opencv.tar.gz https://github.com/opencv/opencv/archive/${OPENCV_VER}/opencv-${OPENCV_VER}.tar.gz && \
	tar -xf opencv.tar.gz && \
	cd opencv* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=MinSizeRel -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY -DCMAKE_CXX_STANDARD=17 \
	-DCMAKE_INSTALL_PREFIX=/usr -DBUILD_TESTS=OFF -DWITH_XINE=OFF -DOPENCV_SKIP_PYTHON_LOADER=ON -DOPENCV_ENABLE_NONFREE=ON -DWITH_VULKAN=ON -DWITH_QT=OFF \
	-DBUILD_TESTS=OFF -DBUILD_PERF_TESTS=OFF -DBUILD_EXAMPLES=OFF \
	-DCPU_BASELINE_REQUIRE=SSE2 && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./opencv*

# Gamemode package is outdated in jammy repo, we need to build own
# Gamemode uses Meson, not CMake for building
# Based on build instructions for Ubuntu 22 on repo page
ENV GAMEMODE_VER=1.8.2
RUN useradd -m rpcs3-dock && \
    echo "rpcs3-dock ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
RUN \
	python3 -m venv /home/rpcs3-dock/.venv && \
    chown -R rpcs3-dock:rpcs3-dock /home/rpcs3-dock/.venv && \
    /home/rpcs3-dock/.venv/bin/pip install meson && \
	curl -L https://github.com/FeralInteractive/gamemode/archive/refs/tags/${GAMEMODE_VER}.tar.gz -o gamemode.tar.gz && \
	tar -xzf gamemode.tar.gz && \
	rm gamemode.tar.gz && \
	mv gamemode-${GAMEMODE_VER} gamemode && \
	chown -R rpcs3-dock:rpcs3-dock /gamemode

# Script needs to be run in user mode so we do this here before going back to root for cleanup
USER rpcs3-dock
WORKDIR /gamemode
ENV CI=true
RUN sudo -k && \
	. /home/rpcs3-dock/.venv/bin/activate && \
	./bootstrap.sh && \
	deactivate

USER root
WORKDIR /
RUN \
	userdel -r rpcs3-dock 2>/dev/null || true && \
	rm -rf /home/rpcs3-dock/.venv && \
	rm -r  /gamemode
RUN ! getent passwd rpcs3-dock
