# Maintainer: Zion Nimchuk <zionnimchuk@gmail.com>
# Submitter: Maxime Gauduin <alucryd@archlinux.org>
# Contributor: Michele Beccalossi <michele.beccalossi@protonmail.com>

pkgname=rpcs3-git
pkgver=0.0.21.13393.3002e592c
pkgrel=1
pkgdesc="A Sony PlayStation 3 emulator"
arch=('x86_64')
url="https://rpcs3.net"
license=('GPL2')
depends=(
  'alsa-lib'
  'glew'
  'glu'
  'libavcodec.so'
  'libavutil.so'
  'libevdev'
  'libgl'
  'libice'
  'libncursesw.so'
  'libpng'
  'libpulse'
  'libsm'
  'libswscale.so'
  'libx11'
  'libxext'
  'openal'
  'qt5-base'
  'qt5-declarative'
  'qt5-multimedia'
  'qt5-svg'
  'sdl2'
  'vulkan-icd-loader'
  'zlib'
  'curl'
  'wolfssl'
# newer system flatbuffers is incompatible with the version RPCS3 depends on
# 'flatbuffers'
  'pugixml'
)
makedepends=(
  'cmake'
  'git'
  'libglvnd'
  'python'
  'vulkan-validation-layers'
  'clang'
)
provides=('rpcs3')
conflicts=('rpcs3')
options=('!emptydirs')
source=(
  'git+https://github.com/RPCS3/rpcs3.git'
  'rpcs3-llvm::git+https://github.com/RPCS3/llvm-mirror.git'
  'git+https://github.com/KhronosGroup/glslang.git'
)
sha256sums=(
  'SKIP'
  'SKIP'
  'SKIP'
)

pkgver() {
  cd rpcs3

  COMM_TAG="$(grep 'version{.*}' rpcs3/rpcs3_version.cpp | awk -F[{,] '{printf "%d.%d.%d", $2, $3, $4}')"
  COMM_COUNT="$(git rev-list --count HEAD)"
  COMM_HASH="$(git rev-parse --short HEAD)"
  echo "${COMM_TAG}.${COMM_COUNT}.${COMM_HASH}"
}

prepare() {
  cd rpcs3

  git submodule init 3rdparty/glslang/glslang llvm
  git config submodule.3rdparty/glslang.url ../glslang
  git config submodule.llvm.url ../rpcs3-llvm

  SUBMODULES=($(git config --file .gitmodules --get-regexp path | \
    awk '!/libpng/ && !/zlib/ && !/curl/ && !/llvm/ && !/glslang/ && !/wolfssl/ && !/pugixml/'))

  # We need to convert from a relative folder path to a https://github.com path
  for ((i=0;i<${#SUBMODULES[@]};i+=2))
  do
    pathid=${SUBMODULES[$i]}
    path=${SUBMODULES[$i+1]}

    git submodule init $path
    urlid=${pathid/%.path/.url}

    # This gets the last two paths in the url, ie RPCS3/rpcs3.git
    url=$(git config $urlid | awk -F/ '{print $(NF-1)"/"$(NF-0)}')

    git config $urlid https://github.com/$url
    git submodule update --init --depth=1 $path
  done

  git submodule update 3rdparty/glslang/glslang llvm
}

build() {
  # GLIBCXX_ASSERTIONS is know to cause unwanted assertions and crash rpcs3
  BAD_FLAG="-Wp,-D_GLIBCXX_ASSERTIONS"
  CXXFLAGS="${CXXFLAGS//$BAD_FLAG/}"

  cmake -S rpcs3 -B build \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_SKIP_RPATH=ON \
    -DCMAKE_CXX_COMPILER=clang++ \
    -DCMAKE_C_COMPILER=clang \
    -DUSE_NATIVE_INSTRUCTIONS=ON \
    -DUSE_SYSTEM_FFMPEG=OFF \
    -DUSE_SYSTEM_LIBPNG=ON \
    -DUSE_SYSTEM_ZLIB=ON \
    -DUSE_SYSTEM_CURL=ON \
    -DUSE_SYSTEM_WOLFSSL=ON \
    -DUSE_SYSTEM_FLATBUFFERS=OFF \
    -DUSE_SYSTEM_PUGIXML=ON \
    -DCMAKE_CXX_FLAGS="$CXXFLAGS -fpermissive"

  make -j$(expr $(nproc) / 2) -C build
}

package() {
  make DESTDIR="${pkgdir}" -C build install
}

# vim: ts=2 sw=2 et:
