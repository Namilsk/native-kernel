pkgbase=native-kernel
pkgname=('native-kernel')
pkgver='7.1.5'
pkgrel=1
pkgdesc="Always-latest kernel compiled with -march=native"
arch=('x86_64')
url="https://kernel.org"
license=('GPL-2.0-only')

depends=(
         coreutils
         mkinitcpio 
        )
makedepends=(
    bc
    bison
    cpio
    flex
    kmod
    libelf
    pahole
    perl
    python
    tar
    clang
    git
    jq
    curl
    llvm
    lld
    gzip
)
install=linux-native.install

_menuconfig=${_menuconfig:-0}
_CLANG=${_CLANG:-1}
_verbose=${_verbose:-0}
_linker=${_linker:-ld.bfd}

prepare() {
    json=$(curl -fsSL https://www.kernel.org/releases.json)
    ver=$(jq -r '.latest_stable.version' <<<"$json")
    echo "$ver" > "$srcdir/.kernelver"

    curl -fsSL "$(jq -r '.releases[1].source' <<<"$json")" -o "linux-$ver.tar.xz"
    tar -xf "linux-$ver.tar.xz"

    cd "$srcdir/linux-$ver"

    local make_args=()

    if (( _CLANG )); then
        make_args+=(LLVM=1 LLVM_IAS=1)
    fi

    if [[ -r /proc/config.gz ]]; then
        zcat /proc/config.gz > .config
    else
        cp "/usr/lib/modules/$(uname -r)/config" .config
    fi

    if (( _CLANG )); then
        scripts/config --enable LTO_CLANG_THIN
        scripts/config --disable LTO_NONE
    fi

    make "${make_args[@]}" olddefconfig

    if (( _menuconfig )); then
        make "${make_args[@]}" menuconfig
        make "${make_args[@]}" olddefconfig
    fi
}

build() {
    ver=$(<"$srcdir/.kernelver")
    cd "$srcdir/linux-$ver"

    export KCFLAGS="-march=native -mtune=native"
    export CFLAGS="-march=native -mtune=native" 

    local make_args=(-j"$(nproc)")

    if (( _CLANG )); then
        make_args+=(LLVM=1 LLVM_IAS=1)
    fi
    if (( _verbose )); then 
        make_args+=(V=1)
    fi

    make "${make_args[@]}" LD=$_linker 
}

package() {
    ver=$(<"$srcdir/.kernelver")

    cd "$srcdir/linux-$ver"

    make INSTALL_MOD_PATH="$pkgdir/usr" modules_install

    install -Dm644 arch/x86/boot/bzImage \
        "$pkgdir/boot/vmlinuz-native"

    install -Dm644 System.map \
        "$pkgdir/usr/lib/modules/$ver/System.map"

    install -Dm644 .config \
        "$pkgdir/usr/lib/modules/$ver/config"

    install -Dm644 \
        "$startdir/linux-native.preset" \
        "$pkgdir/etc/mkinitcpio.d/linux-native.preset"
}
