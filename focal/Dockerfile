FROM ubuntu:20.04
LABEL maintainer="hcorion"

ENV LLVMVER=16

ENV CLANG_BINARY=clang-${LLVMVER}
ENV CLANGXX_BINARY=clang++-${LLVMVER}
ENV LLD_BINARY=lld-${LLVMVER}

ENV GCCVER=11
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
	libwayland-dev libwayland-cursor0 libwayland-client0 libwayland-server0 \
	desktop-file-utils dput gnupg pbuilder ubuntu-dev-tools apt-file && \
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

# Need to manually build vulkan headers and libvulkan
ENV VULKAN_HEADER_VER=1.2.198.0
ENV VULKAN_LOADER_VER=1.2.198.1
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

# We need to build a custom version of SDL2 as well
ENV SDL2VER=2.0.22
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

ENV SHELLCHECKVER=0.8.0
RUN \
	curl -sSfL -o shellcheck.tar.gz https://github.com/koalaman/shellcheck/releases/download/v${SHELLCHECKVER}/shellcheck-v${SHELLCHECKVER}.linux.x86_64.tar.xz && \
	tar -xf shellcheck.tar.gz && \
	cd shellcheck* && \
	cp ./shellcheck /usr/bin/ && \
	cd .. && \
	rm -r ./shellcheck* && \
	shellcheck --version

ENV QTVER=6.5.0
ENV QTVERMIN=65
RUN \
	curl -sSfL -o qt.tar.xz https://download.qt.io/official_releases/qt/${QTVER%.*}/$QTVER/single/qt-everywhere-src-${QTVER}.tar.xz && \
	tar -xf qt.tar.xz && \
	cd qt && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr