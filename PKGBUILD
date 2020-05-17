# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-besrv
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=5.4
_patchver=41
if [[ $_patchver -ne 0 ]]; then
    _tag=v${_basekernel}.${_patchver}
    pkgver=${_basekernel}.${_patchver}
else
    _tag=v${_basekernel}
    pkgver=${_basekernel}
fi
pkgrel=2
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
    '0001-Makefile-disallow-data-races-on-gcc-10-as-well.patch'
    '0004-drop_monitor-work-around-gcc-10-stringop-overflow-wa.patch'
    '0005-gcc-10-warnings-fix-low-hanging-fruit.patch'
    '0007-Stop-the-ad-hoc-games-with-Wno-maybe-initialized.patch'
    '0008-gcc-10-disable-zero-length-bounds-warning-for-now.patch'
    '0009-gcc-10-disable-array-bounds-warning-for-now.patch'
    '0010-gcc-10-disable-stringop-overflow-warning-for-now.patch'
    '0011-gcc-10-disable-restrict-warning-for-now.patch'
    '0013-gcc-10-mark-more-functions-__init-to-avoid-section-m.patch'
    'x86-fix-early-boot-crash-on-gcc-10-next-try.patch'
)
if [[ ${#_extrapatches[@]} -ne 0 ]]; then
    source=( "${source[@]}"
        "${_extrapatches[@]}"
    )
fi

sha512sums=('SKIP'
            'e633608f1cd9596827eb4e52268ac49be07b6691a6c6df6ec79486d7716be35deaafe88f989eb0488d9da6f70f1817cadb993f1f376061573e8835791db86edf'
            '651e94c285cef48e600cf42a66651c71f3a9c7774b16c54936c35e1485a35382777ef84b3f01bf036c95fa141c941eae33867fd605c91e5f8b47617e0c0ef0c3'
            '19ae7c62329dae6531b60ef4a5fcb9a49afc80b03aea216c7b5caf8e05012035ff2058d22d12c82dbb6b3f2e6c1fca6f2564237f75179655cd64f01805cb4ee5'
            '5a96df55ae0e207f04640b3850026c97715e371cd9e810677dcfa7a98492694dda1fc1ec01dbbb150255f1d22a5ea62851759a0963355abb4a8fa35007057d94'
            'c4d992cb7e04883654a63d09b0d8ed6fc6dcbacc0bcca5fb55dbb8831d4d06db2ac6757c97eb095323f061885bbfedc9de529425a1db4dafdc023f24f6e2d612'
            'c1a7ab653af16db19daf1ccd2a168054dcc23aad7a9c1f1f49c6c11943fe80aa156acc69a87476cfdea3f5c7e3988c5d6777030e6b0e2d21f65cc0fc5987d88f'
            'c1153eb5d7d4f9f1ecba4827ef4897c7b6fd501f55a6c32b6ae4eb358420636640b2f53d8c34f2fbac954a36cc40aac51de73ca5c6444db89fb07c414ed6126e'
            '3582c9a91806238ff02962a5139adebbec5c6af445646e4b17ab1932bb416a6f9f77d9c4a8333a0c304e63e2b2a29ca98988b58bb5e1c3a4687f8b19adf62a6f'
            'b670d888b792e1eb0aae61431e92fecfe9b1946531ca8c710ee6081611ef7dd5ab602fa293b261b2179aac47450b6e1f78cfa31d397aae85ee4cc886dc005eaf'
            '55a36b5fcdf37d9e992691766167c44c75244ec5e413a99803e33994484a4bc36813a8d46f6cf26db7b5be6a7705149dc3ecd77f9e88fe0e7b5c7f2ee0f1de53'
            '69f59b28d815bc459f6ba1448d7b1d9455eae8afe632e7b1d19e3caba9a75e66efbe8e4b748247b5b49c9c429b4316e1462474932ddadb6c60ee97f0954edd1a'
            'ddb32996639d2d38aa71a5608b2fec32a955b8f837ec1a3dcdb3627dae2d57bf345815098417cb69d9675999ab5cba0d125e553240705a854c21043687f5b532')

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
