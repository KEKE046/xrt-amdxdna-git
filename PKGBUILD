pkgname='xrt-amdxdna-git'
pkgver=1.0
pkgrel=1
pkgdesc="XDNA driver and XRT"
arch=('x86_64')
conflict=('xrt')
source=(
    "99-amdxdna.rules"
    "aie-rt.patch"
    "xrt.patch"
    "xdna-driver::git+https://github.com/amd/xdna-driver.git"
    "aie-rt::git+https://github.com/Xilinx/aie-rt"
    "ELFIO::git+https://github.com/serge1/ELFIO"
    "cxxopts::git+https://github.com/jarro2783/cxxopts"
    "XRT::git+https://github.com/Xilinx/XRT"
    "dma_ip_drivers::git+https://github.com/Xilinx/dma_ip_drivers"
    "GSL::git+https://github.com/microsoft/GSL"
    "aiebu::git+https://github.com/Xilinx/aiebu"
    "aie-rt::git+https://github.com/Xilinx/aie-rt"
)
sha256sums=('5ba1d1d1fb8c3d8b898d03c30e1a94afc069dd8f70e87821e2146c5a821095b4'
            '84dca17e271b310ed445d9e4f65a4978598d3d6101da99ed16d8e266532ee9be'
            'f775d9544175c5991601954552452f401c12a47f9176dee7af40ec907f37f6e4'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')

pkgver() {
  cd "$srcdir/xdna-driver"
  git describe --long --abbrev=7 | sed 's/\([^-]*-g\)/r\1/;s/-/./g'
}

prepare_submodule() {
    git -C "$srcdir/xdna-driver" -c protocol.file.allow=always config submodule.xrt.url "$srcdir/XRT"
    git -C "$srcdir/xdna-driver" -c protocol.file.allow=always submodule update --init
    git -C "$srcdir/xdna-driver/xrt" -c protocol.file.allow=always config submodule.src/runtime_src/core/pcie/driver/linux/xocl/lib/libqdma.url "$srcdir/dma_ip_drivers"
    git -C "$srcdir/xdna-driver/xrt" -c protocol.file.allow=always config submodule.src/runtime_src/core/common/elf.url "$srcdir/ELFIO"
    git -C "$srcdir/xdna-driver/xrt" -c protocol.file.allow=always config submodule.src/runtime_src/core/common/gsl.url "$srcdir/GSL"
    git -C "$srcdir/xdna-driver/xrt" -c protocol.file.allow=always config submodule.src/runtime_src/core/common/aiebu.url "$srcdir/aiebu"
    git -C "$srcdir/xdna-driver/xrt" -c protocol.file.allow=always config submodule.src/runtime_src/aie-rt.url "$srcdir/aie-rt"
    git -C "$srcdir/xdna-driver/xrt" -c protocol.file.allow=always submodule update --init
    git -C "$srcdir/xdna-driver/xrt/src/runtime_src/core/common/aiebu" -c protocol.file.allow=always config submodule.lib/aie-rt.url "$srcdir/aie-rt"
    git -C "$srcdir/xdna-driver/xrt/src/runtime_src/core/common/aiebu" -c protocol.file.allow=always config submodule.src/cpp/ELFIO.url "$srcdir/ELFIO"
    git -C "$srcdir/xdna-driver/xrt/src/runtime_src/core/common/aiebu" -c protocol.file.allow=always config submodule.src/cpp/cxxopts.url "$srcdir/cxxopts"
    git -C "$srcdir/xdna-driver/xrt/src/runtime_src/core/common/aiebu" -c protocol.file.allow=always submodule update --init
    git -C "$srcdir/xdna-driver/xrt/src/runtime_src/core/common/aiebu/lib/aie-rt" apply "$srcdir/aie-rt.patch"
    git -C "$srcdir/xdna-driver/xrt" apply "$srcdir/xrt.patch"
}

prepare() {
    prepare_submodule
}

build() {
    cd "$srcdir/xdna-driver/xrt/build"
    ./build.sh -noinit -npu -opt -j `nproc`
    cd "$srcdir/xdna-driver/build"
    ./build.sh -release -j `nproc` 
    ./build.sh -package
}

package() {
    cd "$srcdir/xdna-driver/build"
    bsdtar xvf Release/xrt_plugin.2.19.0_arch-x86_64-amdxdna.deb
    tar xvf data.tar.gz -C "$pkgdir"
    mkdir -p "$pkgdir/usr"
    mv "$pkgdir/lib" "$pkgdir/usr/lib"
    . "$pkgdir/opt/xilinx/xrt/amdxdna/dkms.conf"
    mkdir -p "$pkgdir/usr/src/$PACKAGE_NAME-$PACKAGE_VERSION"
    tar xvf "$pkgdir/opt/xilinx/xrt/amdxdna/amdxdna.tar.gz" -C "$pkgdir/usr/src/$PACKAGE_NAME-$PACKAGE_VERSION"
    cp "$pkgdir/opt/xilinx/xrt/amdxdna/dkms.conf" "$pkgdir/usr/src/$PACKAGE_NAME-$PACKAGE_VERSION/"
    cd "$srcdir/xdna-driver/xrt/build"
    tar xvf Release/xrt_202510.2.19.0_--base-.tar.gz -C "$pkgdir"
    tar xvf Release/xrt_202510.2.19.0_--runtime.tar.gz -C "$pkgdir"
    tar xvf Release/xrt_202510.2.19.0_--npu.tar.gz -C "$pkgdir"
    tar xvf Release/xrt_202510.2.19.0_--Runtime.tar.gz -C "$pkgdir"
    install -Dm644 "$srcdir/99-amdxdna.rules" "$pkgdir/etc/udev/rules.d/99-amdxdna.rules"
}

post_install() {
    echo "rmmod amdxdna; modprobe amdxdna" 
}
