# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-besrv
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=5.4
_patchver=53
if [[ $_patchver -ne 0 ]]; then
    _tag=v${_basekernel}.${_patchver}
    pkgver=${_basekernel}.${_patchver}
else
    _tag=v${_basekernel}
    pkgver=${_basekernel}
fi
pkgrel=1
arch=('x86_64')
license=('GPL2')
makedepends=('git' 'bc' 'kmod')
url="http://www.kernel.org"
options=(!strip)

validpgpkeys=(
    'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # "Linus Torvalds"
    '647F28654894E3BD457199BE38DBBDC86092693E'  # "Greg Kroah-Hartman"
    'E27E5D8A3403A2EF66873BBCDEA66FF797772CDC'  # "Sasha Levin"
)

source=(
    "linux-stable::git+https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git#tag=${_tag}?signed"
    # the main kernel config files
    'config-server.x86_64'
    # sysctl config
    'sysctl-linux-besrv.conf'
)

## extra patches
_extrapatches=(
)
if [[ ${#_extrapatches[@]} -ne 0 ]]; then
    source=( "${source[@]}"
        "${_extrapatches[@]}"
    )
fi

sha512sums=('SKIP'
            '363f4547b6333a12c3caf8d467e5684f93cc039a435abf6de4564348e0b2f9d9a0a7dcb4daef91c655a217ade96081aa96d20016300caf3104eccdf000211834'
            '651e94c285cef48e600cf42a66651c71f3a9c7774b16c54936c35e1485a35382777ef84b3f01bf036c95fa141c941eae33867fd605c91e5f8b47617e0c0ef0c3')

prepare() {
    cd "$srcdir/linux-stable"

    # extra patches
    for patch in ${_extrapatches[@]}; do
        patch="$(basename "$patch" | sed -e 's/\.\(gz\|bz2\|xz\)//')"
        pext=${patch##*.}
        if [[ "$pext" == 'patch' ]] || [[ "$pext" == 'diff' ]]; then
            msg2 "apply $patch"
            patch -Np1 -i "$srcdir/$patch"
        fi
    done

    # set configuration
    msg2 "copy configuration"
    cp "$srcdir/config-server.x86_64" ./.config
    if [[ "$_kernelname" != "" ]]; then
        sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"\U$_kernelname\"|g" ./.config
    fi

    # set extraversion to pkgrel
    msg2 "set extraversion to $pkgrel"
    sed -ri "s|^(EXTRAVERSION =).*|\1 -$pkgrel|" Makefile

    # hack to prevent output kernel from being marked as dirty or git
    msg2 "apply hack to prevent kernel tree being marked dirty"
    echo "" > "$srcdir/linux-stable/.scmversion"
}

build() {
    cd "$srcdir/linux-stable"

    msg2 "prepare"
    make prepare
    # load configuration
    # Configure the kernel. Replace the line below with one of your choice.
    #make nconfig # preferred CLI menu for configuration
    #make menuconfig # CLI menu for configuration
    # ... or manually edit .config
    ####################
    # stop here
    # this is useful to configure the kernel
    #msg "Stopping build"
    #return 1
    ####################
    # yes "" | make config
    # build!
    make -s kernelrelease > version
    msg2 "build"
    make $MAKEFLAGS bzImage modules
}

package_linux-besrv() {
    pkgdesc="The Linux Kernel and modules, BlackEagle Server Edition"
    provides=('linux')
    depends=('coreutils' 'kmod>=26' 'mkinitcpio>=27')
    optdepends=(
        'crda: to set the correct wireless channels of your country'
        'linux-firmware: when having some hardware needing special firmware'
    )
    install=$pkgname.install

    cd "$srcdir/linux-stable"
    local kernver="$(<version)"
    local modulesdir="$pkgdir/usr/lib/modules/$kernver"

    msg2 "Installing boot image..."
    # systemd expects to find the kernel here to allow hibernation
    # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
    install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

    # Used by mkinitcpio to name the kernel
    echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

    msg2 "Installing modules..."
    make INSTALL_MOD_STRIP=1 INSTALL_MOD_PATH="$pkgdir/usr" modules_install

    # remove build and source links
    rm "$modulesdir"/{source,build}

    # install sysctl tweaks
    install -Dm644 "$srcdir/sysctl-linux-besrv.conf" \
        "$pkgdir/usr/lib/sysctl.d/60-linux-besrv.conf"

    msg2 "Fixing permissions..."
    chmod -Rc u=rwX,go=rX "$pkgdir"

}

package_linux-besrv-headers() {
    pkgdesc="Header files and scripts for building modules for linux$_kernelname"
    provides=('linux-headers')

    KARCH=x86

    cd "$srcdir/linux-stable"
    local kernver="$(<version)"
    local builddir="$pkgdir/usr/lib/modules/$kernver/build"

    msg2 "Installing build files..."
    install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
        version
        #version vmlinux
    install -Dt "$builddir/kernel" -m644 kernel/Makefile
    install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile

    # add objtool for external module building and enabled VALIDATION_STACK option
    install -Dt "$builddir/tools/objtool" tools/objtool/objtool

    msg2 "Installing headers..."
    find . -path './include/*' -prune -o -path './scripts/*' -prune \
        -o -type f \( -name 'Makefile*' -o -name 'Kconfig*' \
        -o -name 'Kbuild*' -o -name '*.sh' -o -name '*.pl' \
        -o -name '*.lds' \) | bsdcpio -pdm "$builddir"
    cp -t "$builddir" -a scripts include
    find $(find arch/$KARCH -name include -type d -print) -type f \
        | bsdcpio -pdm "$builddir"
    find $(find arch/$KARCH -name kernel -type d -print) -iname '*.s' -type f \
        | bsdcpio -pdm "$builddir"


    # add objtool for external module building and enabled VALIDATION_STACK option
    install -Dt "$builddir/tools/objtool" tools/objtool/objtool

    msg2 "Removing unneeded architectures..."
    local arch
    for arch in "$builddir"/arch/*/; do
        [[ $arch = */$KARCH/ ]] && continue
        echo "Removing $(basename "$arch")"
        rm -r "$arch"
    done

    msg2 "Removing documentation..."
    rm -r "$builddir/Documentation"

    msg2 "Removing broken symlinks..."
    find -L "$builddir" -type l -printf 'Removing %P\n' -delete

    msg2 "Removing loose objects..."
    find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

    msg2 "Stripping build tools..."
    local file
    while read -rd '' file; do
        case "$(file -bi "$file")" in
            application/x-sharedlib\;*)      # Libraries (.so)
                strip -v $STRIP_SHARED "$file" ;;
            application/x-archive\;*)        # Libraries (.a)
                strip -v $STRIP_STATIC "$file" ;;
            application/x-executable\;*)     # Binaries
                strip -v $STRIP_BINARIES "$file" ;;
            application/x-pie-executable\;*) # Relocatable binaries
                strip -v $STRIP_SHARED "$file" ;;
        esac
    done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

    msg2 "Adding symlink..."
    mkdir -p "$pkgdir/usr/src"
    ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"

    msg2 "Fixing permissions..."
    chmod -Rc u=rwX,go=rX "$pkgdir"
}
