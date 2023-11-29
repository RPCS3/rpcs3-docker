FROM ubuntu:20.04
LABEL maintainer="hcorion"

ENV LLVMVER=17

ENV CLANG_BINARY=clang-${LLVMVER}
ENV CLANGXX_BINARY=clang++-${LLVMVER}
ENV LLD_BINARY=lld-${LLVMVER}

ENV GCCVER=13
ENV GCC_BINARY=gcc-${GCCVER}
ENV GXX_BINARY=g++-${GCCVER}

RUN \
	apt-get update -y && \
	apt-get install -y curl libcurl4-openssl-dev software-properties-common apt-transport-https && \
	curl -Lvs https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add - && \
	add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
	add-apt-repository -y "deb http://apt.llvm.org/focal/ llvm-toolchain-focal-${LLVMVER} main" && \
	apt-get update -y --fix-missing && \
	\
	apt-get install -y \
	$GCC_BINARY $GXX_BINARY \
	build-essential binutils git ccache bc $CLANG_BINARY $CLANGXX_BINARY $LLD_BINARY \
	libasound2-dev libopenal-dev freeglut3-dev libudev-dev libevdev-dev libpulse-dev libxrandr-dev libglvnd-dev \
	'^libxcb.*-dev' libxcb-cursor0 libx11-xcb-dev libxcb-util-dev \
	libssl-dev libfontconfig1-dev libv4l-dev yasm libgmp-dev libmp3lame-dev libopenjp2-7-dev libjack-dev \
	libtheora-dev libwebp-dev libopus-dev libvorbis-dev libvpx-dev libx264-dev libxml2-dev ocl-icd-opencl-dev \
	libwayland-dev libwayland-cursor0 libxkbcommon-x11-dev libwayland-client0 libwayland-server0 \
	desktop-file-utils dput gnupg pbuilder ubuntu-dev-tools apt-file patchelf && \
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
ENV CMAKEVER=3.26.3
RUN \
	curl -sSfLO https://github.com/Kitware/CMake/releases/download/v${CMAKEVER}/cmake-${CMAKEVER}-linux-x86_64.sh && \
	sh cmake-${CMAKEVER}-linux-x86_64.sh --prefix=/usr --skip-license && \
	rm ./cmake*.sh && \
	cmake --version

# We gotta build our own version of Ninja as well
ENV NINJAVER=1.11.1
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

ENV STATIC_LLVM_VER=16.0.6
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
ENV VULKAN_HEADER_VER=1.3.224.0
ENV VULKAN_LOADER_VER=1.3.224.1
RUN \
	curl -sSfL -o Vulkan-Headers.tar.gz https://github.com/KhronosGroup/Vulkan-Headers/archive/sdk-${VULKAN_HEADER_VER}.tar.gz && \
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
	curl -sSfL -o Vulkan-Loader.tar.gz https://github.com/KhronosGroup/Vulkan-Loader/archive/sdk-${VULKAN_LOADER_VER}.tar.gz && \
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

ENV FFMPEG_NV_CODEC_VER=12.0.16.0
RUN \
	curl -sSfLO https://github.com/FFmpeg/nv-codec-headers/releases/download/n${FFMPEG_NV_CODEC_VER}/nv-codec-headers-${FFMPEG_NV_CODEC_VER}.tar.gz && \
	tar -xf ./nv-codec-headers-*.tar.gz && \
	cd nv-codec-headers-* && \
	make && \
	PREFIX=/usr make install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./nv-codec-headers-*

ENV FFMPEGVER=6.0
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
	--enable-nvdec --enable-nvenc --enable-ffnvcodec --enable-opencl --enable-opengl --enable-shared --enable-version3 --enable-vulkan && \
	make && \
	PREFIX=/usr make install && \
	cd ../.. && \
	rm ./*.tar.xz && \
	rm -r ./ffmpeg-*

ENV QTVER=6.6.1
ENV QTVERMIN=66
RUN \
	curl -sSfL -o qt.tar.xz https://download.qt.io/official_releases/qt/${QTVER%.*}/$QTVER/single/qt-everywhere-src-${QTVER}.tar.xz && \
	tar -xf qt.tar.xz && \
	cd  qt-everywhere-src-* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=MinSizeRel -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY -DFEATURE_ffmpeg=ON -DFEATURE_gstreamer=OFF -DFEATURE_fontconfig=ON -DFEATURE_system_freetype=ON -DFEATURE_system_xcb_xinput=ON -DFEATURE_openssl_linked=ON \
	-DCMAKE_INSTALL_PREFIX=/usr && \
	echo "Need to cleanup for latest GCC" && \
	sed -i 's/std::uint32_t/uint32_t/' ../qt3d/src/3rdparty/assimp/src/code/AssetLib/FBX/FBXBinaryTokenizer.cpp && \
	ninja && \
	ninja install && \
	cd ../../ && \
	rm -r ./qt-everywhere-src-* && \
	rm qt.tar.xz

# We need to build a custom version of SDL2 as well
ENV SDL2VER=2.28.4
RUN \
	curl -sSfL -o SDL2.tar.gz https://www.libsdl.org/release/SDL2-${SDL2VER}.tar.gz && \
	tar -xf SDL2.tar.gz && \
	cd SDL2* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr -DSDL_DLOPEN=ON -DARTS=OFF -DESD=OFF -DNAS=OFF -DALSA=ON -DPULSEAUDIO_SHARED=ON -DRPATH=OFF -DCLOCK_GETTIME=ON && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./SDL2*

ENV SHELLCHECKVER=0.9.0
RUN \
	curl -sSfL -o shellcheck.tar.gz https://github.com/koalaman/shellcheck/releases/download/v${SHELLCHECKVER}/shellcheck-v${SHELLCHECKVER}.linux.x86_64.tar.xz && \
	tar -xf shellcheck.tar.gz && \
	cd shellcheck* && \
	cp ./shellcheck /usr/bin/ && \
	cd .. && \
	rm -r ./shellcheck* && \
	shellcheck --version