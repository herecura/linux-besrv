# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-besrv
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=4.4
_patchver=41
pkgrel=1
arch=('i686' 'x86_64')
license=('GPL2')
makedepends=('bc' 'kmod')
url="http://www.kernel.org"
options=(!strip)

validpgpkeys=(
    'ABAF11C65A2970B130ABE3C479BE3E4300411886'
    '647F28654894E3BD457199BE38DBBDC86092693E'
)

source=(
    "https://www.kernel.org/pub/linux/kernel/v4.x/linux-${_basekernel}.tar.xz"
    "https://www.kernel.org/pub/linux/kernel/v4.x/linux-${_basekernel}.tar.sign"
    # the main kernel config files
    'config-server.i686'
    'config-server.x86_64'
    # standard config files for mkinitcpio ramdisk
    "linux$_kernelname.preset"
    # pacman hooks
    'linux-besrv-01-depmod.hook'
    'linux-besrv-02-initcpio.hook'
    'linux-besrv-remove.hook'
)

# revision patches
if [ ${_patchver} -ne 0 ]; then
    pkgver=$_basekernel.$_patchver
    _patchname="patch-$pkgver"
    source=( "${source[@]}"
        "https://www.kernel.org/pub/linux/kernel/v4.x/${_patchname}.xz"
        "https://www.kernel.org/pub/linux/kernel/v4.x/${_patchname}.sign"
    )
else
    pkgver=$_basekernel
fi

# extra patches
_extrapatches=(
)
_extrapatchessums=(
)
if [ ${#_extrapatches[@]} -ne 0 ]; then
    source=( "${source[@]}"
        "${_extrapatches[@]}"
    )
fi

sha256sums=('401d7c8fef594999a460d10c72c5a94e9c2e1022f16795ec51746b0d165418b2'
            'SKIP'
            'e64fd39391286177560700b26ecdd733135fdc737a9f3fc70e39c6a312dad054'
            '43169672980d57445cb139f74f315179223b1fbe8f451ce112de703700976496'
            '64b2cf77834533ae7bac0c71936087857d8787d0e2a349037795eb7e42d23dde'
            'd71ee2f7757b52dec72f8e3620d9090f33e1526b7cfce41299e1140c5e6493ad'
            'a1099b98f11f37d6e5896a869fcc50f586ea51992b0b2d142a929bcdd1bc9f89'
            '0712ed50138c1d7f70c1f4276fcdb172cc50f78b52aa86edd5e7bbbe130968ce'
            'afa6dff40bb3bd8e7ade90cbd2bab05b69e77ed317915ce0ea46863411df7b4d'
            'SKIP')

prepare() {
    cd "$srcdir/linux-$_basekernel"

    # Add revision patches
    if [ $_patchver -ne 0 ]; then
        msg2 "apply $_patchname"
        patch -Np1 -i "$srcdir/$_patchname"
    fi

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
    if [ "$CARCH" = "x86_64" ]; then
        cat "$srcdir/config-server.x86_64" >./.config
    else
        cat "$srcdir/config-server.i686" >./.config
    fi
    if [ "$_kernelname" != "" ]; then
        sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"\U$_kernelname\"|g" ./.config
    fi

    # remove sublevel, this is a server version, needs to be updateable
    # without rebooting all the time
    #msg2 "remove sublevel"
    #sed -e "s|SUBLEVEL = .*|SUBLEVEL = |g" -i Makefile

    # set extraversion to pkgrel
    msg2 "set extraversion to $pkgrel"
    sed -ri "s|^(EXTRAVERSION =).*|\1 -$pkgrel|" Makefile

    # don't run depmod on 'make install'. We'll do this ourselves in packaging
    sed -i '2iexit 0' scripts/depmod.sh

    # hack to prevent output kernel from being marked as dirty or git
    msg2 "apply hack to prevent kernel tree being marked dirty"
    echo "" > "$srcdir/linux-$_basekernel/.scmversion"

}

build() {
    cd "$srcdir/linux-$_basekernel"

    # get kernel version
    msg2 "prepare"
    make prepare
    # load configuration
    # Configure the kernel. Replace the line below with one of your choice.
    #make menuconfig # CLI menu for configuration
    #make xconfig # X-based configuration
    #make oldconfig # using old config from previous kernel version
    # ... or manually edit .config
    ####################
    # stop here
    # this is useful to configure the kernel
    #msg "Stopping build"
    #return 1
    ####################
    # yes "" | make config
    # build!
    msg2 "build"
    make $MAKEFLAGS bzImage modules
}

package_linux-besrv() {
    pkgdesc="The Linux Kernel and modules, BlackEagle Server Edition"
    provides=('linux')
    backup=(
        "etc/mkinitcpio.d/$pkgname.preset"
    )
    depends=('coreutils' 'kmod>=10' 'mkinitcpio>=0.9')
    optdepends=(
        'crda: to set the correct wireless channels of your country'
        'linux-firmware: when having some hardware needing special firmware'
    )

    install=$pkgname.install

    KARCH=x86
    cd "$srcdir/linux-$_basekernel"

    mkdir -p "$pkgdir"/{lib/modules,lib/firmware,boot,usr}

    # get kernel version
    _kernver=$(make kernelrelease)

    # install modules
    make INSTALL_MOD_STRIP=1 INSTALL_MOD_PATH="$pkgdir" modules_install

    # copy System.map and bzImage
    install -m644 System.map "$pkgdir/boot/System.map$_kernelname"
    install -m644 arch/$KARCH/boot/bzImage "$pkgdir/boot/vmlinuz$_kernelname"

    # install fallback mkinitcpio.conf file and preset file for kernel
    install -m644 -D "$srcdir/$pkgname.preset" "$pkgdir/etc/mkinitcpio.d/$pkgname.preset"

    # set correct depmod command for install
    sed \
        -e  "s/KERNEL_NAME=.*/KERNEL_NAME=$_kernelname/g" \
        -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=$_kernver/g" \
        -i "$startdir/$pkgname.install"
    sed \
        -e "s|source .*|source /etc/mkinitcpio.d/$pkgname.kver|g" \
        -e "s|default_image=.*|default_image=\"/boot/initramfs$_kernelname.img\"|g" \
        -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs$_kernelname-fallback.img\"|g" \
        -i "$pkgdir/etc/mkinitcpio.d/$pkgname.preset"

    echo -e "# DO NOT EDIT THIS FILE\nALL_kver='$_kernver'" > "$pkgdir/etc/mkinitcpio.d/$pkgname.kver"

    # remove build and source links
    rm -f "$pkgdir/lib/modules/$_kernver"/{source,build}

    # remove the firmware
    rm -rf "$pkgdir/lib/firmware"

    _fldkernelname=$(echo $_kernelname | tr "[:lower:]" "[:upper:]")
    # make room for external modules
    ln -s "../${_basekernel}$_fldkernelname-external" "$pkgdir/lib/modules/$_kernver/external"
    # add real version for building modules and running depmod from post_install/upgrade
    mkdir -p "$pkgdir/lib/modules/$_basekernel$_fldkernelname-external"
    echo "$_kernver" > "$pkgdir/lib/modules/${_basekernel}$_fldkernelname-external/version"

    # gzip all modules
    find "$pkgdir" -name '*.ko' -exec gzip -9 {}  \;

    # Now we call depmod...
    depmod -b "$pkgdir" -F System.map "$_kernver"

    # install pacman hooks
    install -Dm644 "$srcdir/linux-besrv-01-depmod.hook" "$pkgdir/usr/share/libalpm/hooks/linux-besrv-01-depmod.hook"
    install -Dm644 "$srcdir/linux-besrv-02-initcpio.hook" "$pkgdir/usr/share/libalpm/hooks/linux-besrv-02-initcpio.hook"
    install -Dm644 "$srcdir/linux-besrv-remove.hook" "$pkgdir/usr/share/libalpm/hooks/linux-besrv-remove.hook"

    # move module tree /lib -> /usr/lib
    mv "$pkgdir/lib" "$pkgdir/usr/"
}

package_linux-besrv-headers() {
    pkgdesc="Header files and scripts for building modules for linux$_kernelname"
    provides=('linux-headers')
    install -dm755 "$pkgdir/usr/lib/modules/$_kernver"
    cd "$pkgdir/usr/lib/modules/$_kernver"
    ln -sf ../../../src/linux-$_kernver build
    cd "$srcdir/linux-$_basekernel"
    install -D -m644 Makefile \
        "$pkgdir/usr/src/linux-$_kernver/Makefile"
    install -D -m644 kernel/Makefile \
        "$pkgdir/usr/src/linux-$_kernver/kernel/Makefile"
    install -D -m644 .config \
        "$pkgdir/usr/src/linux-$_kernver/.config"

    find . -path './include/*' -prune -o -path './scripts/*' -prune \
        -o -type f \( -name 'Makefile*' -o -name 'Kconfig*' \
        -o -name 'Kbuild*' -o -name '*.sh' -o -name '*.pl' \
        -o -name '*.lds' \) | bsdcpio -pdm "$pkgdir/usr/src/linux-$_kernver"
    cp -a scripts include "$pkgdir/usr/src/linux-$_kernver"
    find $(find arch/$KARCH -name include -type d -print) -type f \
        | bsdcpio -pdm "$pkgdir/usr/src/linux-$_kernver"
    install -Dm644 Module.symvers "$pkgdir/usr/src/linux-$_kernver"

    # add docbook makefile
    install -D -m644 Documentation/DocBook/Makefile \
        "$pkgdir/usr/src/linux-$_kernver/Documentation/DocBook/Makefile"

    # strip scripts directory
    find "$pkgdir/usr/src/linux-$_kernver/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
        case "$(file -bi "$binary")" in
            *application/x-sharedlib*) # Libraries (.so)
                /usr/bin/strip $STRIP_SHARED "$binary"
                ;;
            *application/x-archive*) # Libraries (.a)
                /usr/bin/strip $STRIP_STATIC "$binary"
                ;;
            *application/x-executable*) # Binaries
                /usr/bin/strip $STRIP_BINARIES "$binary"
                ;;
        esac
    done

    chown -R root:root "$pkgdir/usr/src/linux-$_kernver"
    find "$pkgdir/usr/src/linux-$_kernver" -type d -exec chmod 755 {} \;
}
