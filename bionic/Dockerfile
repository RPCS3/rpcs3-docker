FROM ubuntu:18.04
LABEL maintainer="hcorion"

ENV LLVMVER=14

ENV CLANG_BINARY=clang-${LLVMVER}
ENV CLANGXX_BINARY=clang++-${LLVMVER}
ENV LLD_BINARY=lld-${LLVMVER}

RUN \
	apt-get update -y && \
	apt-get install -y curl libcurl4-openssl-dev software-properties-common apt-transport-https && \
	curl -Lvs https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add - && \
	add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
	add-apt-repository -y "deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-${LLVMVER} main" && \
	apt-get update -y --fix-missing && \
	\
	apt-get install -y \
	build-essential binutils git ccache bc $CLANG_BINARY $CLANGXX_BINARY $LLD_BINARY \
	libasound2-dev libopenal-dev freeglut3-dev libudev-dev libevdev-dev libpulse-dev libxrandr-dev libglvnd-dev \
	libwayland-dev libwayland-cursor0 libwayland-client0 libwayland-server0 \
	desktop-file-utils dput gnupg pbuilder ubuntu-dev-tools apt-file && \
	apt-get install --only-upgrade dpkg && \
	apt-get clean autoclean && \
	apt-get autoremove --yes && \
	rm -rf /var/lib/apt
	
RUN \
	curl -sSfLO http://http.us.debian.org/debian/pool/main/g/glew/libglew2.2_2.2.0-4_amd64.deb && \
	curl -sSfLO http://http.us.debian.org/debian/pool/main/g/glew/libglew-dev_2.2.0-4_amd64.deb && \
	dpkg -i ./*.deb && \
	rm ./*.deb

ENV GCCVERFULL=12.2.0
ENV GCCVER=12
ENV GCC_BINARY=gcc-${GCCVER}
ENV GXX_BINARY=g++-${GCCVER}

RUN \
	curl -sSfL -o gcc.tar.gz https://ftp.gnu.org/gnu/gcc/gcc-12.2.0/gcc-12.2.0.tar.gz && \
	tar -xf gcc.tar.gz && \
	cd gcc && \
	./configure --prefix=/usr --libdir=/usr/lib --libexecdir=/usr/lib --mandir=/usr/share/man --infodir=/usr/share/info \
      --with-bugurl=https://bugs.archlinux.org/ --with-build-config=bootstrap-lto --with-linker-hash-style=gnu --with-system-zlib \
      --enable-__cxa_atexit --enable-cet=auto --enable-checking=release --enable-clocale=gnu --enable-gnu-indirect-function \
      --enable-gnu-unique-object --enable-libstdcxx-backtrace --enable-link-serialization=1 --enable-linker-build-id \
      --enable-lto --enable-multilib --enable-plugin --enable-shared --enable-threads=posix --disable-libssp --disable-libstdcxx-pch \
      --disable-werror \
	  --enable-languages=c,c++,lto --enable-bootstrap && \
	  make -O bootstrap && \
	  make && \
	  make install

# Setup newer CMake
ENV CMAKEVER=3.16.9
RUN \
	curl -sSfLO https://cmake.org/files/v${CMAKEVER%.*}/cmake-${CMAKEVER}-Linux-x86_64.sh && \
	sh cmake-${CMAKEVER}-Linux-x86_64.sh --skip-license && \
	rm ./cmake*.sh && \
	cmake --version

# We gotta build our own version of Ninja as well
ENV NINJAVER=1.11.0
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
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
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
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
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

ENV QTVER=6.4.0
ENV QTVERMIN=64
RUN \
	curl -sSfL -o qt.tar.xz https://download.qt.io/official_releases/qt/${QTVER%.*}/$QTVER/single/qt-everywhere-src-${QTVER}.tar.xz && \
	tar -xf qt.tar.xz && \
	cd qt && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr
# qt${QTVERMIN}base qt${QTVERMIN}declarative qt${QTVERMIN}xmlpatterns qt${QTVERMIN}svg qt${QTVERMIN}multimedia

# Need to patch Qt for compatibility with GCC 11
# See https://codereview.qt-project.org/c/qt/qtbase/+/339417 and https://github.com/RPCS3/rpcs3/issues/10291
RUN \
	sed -i 's/ThreadEngineStarter<void>(ThreadEngine<void> \*_threadEngine)/ThreadEngineStarter(ThreadEngine<void> \*_threadEngine)/' \
	/opt/qt515/include/QtConcurrent/qtconcurrentthreadengine.h
