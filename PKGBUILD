# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-besrv
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=4.14
_patchver=45
pkgrel=1
arch=('x86_64')
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
    'config-server.x86_64'
    # standard config files for mkinitcpio ramdisk
    "linux$_kernelname.preset"
    # pacman hooks
    'linux-besrv-01-depmod.hook'
    'linux-besrv-02-initcpio.hook'
    'linux-besrv-remove.hook'
    # sysctl config
    'sysctl-linux-besrv.conf'
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
    '0001-objtool_Fix_noreturn_detection_for_recursive_sibling_calls.patch'
    '0002-objtool_support_GCC8_cold_subfunctions.patch'
    '0003-objtool_support_GCC8_switch_tables.patch'
)
if [ ${#_extrapatches[@]} -ne 0 ]; then
    source=( "${source[@]}"
        "${_extrapatches[@]}"
    )
fi

sha512sums=('77e43a02d766c3d73b7e25c4aafb2e931d6b16e870510c22cef0cdb05c3acb7952b8908ebad12b10ef982c6efbe286364b1544586e715cf38390e483927904d8'
            'SKIP'
            'c4125d029f653b06d00903ae35545610ce44b1c2748cc72fc036b442302dd297c759a7cf5ccca693050ef5ac0039e0e02a5e7dc1b59d095bf37bb40432ce5af2'
            '75f580633a48a15efa83e44a2e091ba33e1d615107eb192349b7ff3ea6aec3230f4206795747e238fe015d511125ab78b58571904577dd4eb687bba937ad95a6'
            '4bd79cd8b10c30a80c6b4c8b4ff173803a69e5af20b4d56cad8e5275547e7d4c5918522fb8e4a71c05a1247c68a2201af389526086b6d77965ad0bd18c95da83'
            'f03250e32620071f27d33dbda859958ecbb206f2723a3c14f4f41734435011c87b4809bda558d687393d9fd2665531904f8963f1038f0bf8fb5598adc1d0518e'
            'e7ba6fcf986022ec56614b1acedf1e6ad723ffea12f8bf73741eef317da59f57b9df83e1800ea3e9b2d9e25207e6ac7fe4286927602d82435e1aa6525ceed0dc'
            'c898a80f38c03853bccee184b978ddaaf3c16de0ad1907adc32ac308e7d6b4d82615f743450fc3e7cecdc226e9f5017f4f94aea88da4f47c4de211abbcdbd0af'
            '64987f0efa64f483c21cbfd51b6615238e0152eef760de809a868e1d7c26720793d936277e4f1a1bb1a8bbc7131c34ed12b3cb914058bc3edd4d3b1b33ecf868'
            'SKIP'
            '5d2ac655af70769167a7627d40ba328148d191551b650f555db19f91f9a962769e5946f655a390947fcee9262410340380cb95786c7325871ed4fdf4cf3e4902'
            'f09a78a325b8f9f2edafbfea3101a2b4aa22922485c9977583ad5ee044fc821b09c11dda4546cbb9e4c4cf3f0d0d10ca8bf5dc9f1814e1e2c11d8b6caea7c6eb'
            '2e6bb4b717b5794cbb92711eb3c2b4c0fbb0e055f8ecc5f2129987d027976fc7062a1f1629cafd48d7e1e884f284f41da16f3808b9609aa9ded33109b78ece8b')

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
    cat "$srcdir/config-server.x86_64" >./.config
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

    # sync-check does not have executable flag
    chmod +x tools/objtool/sync-check.sh

}

build() {
    cd "$srcdir/linux-$_basekernel"

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

    # install bzImage
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

    # move module tree /lib -> /usr/lib
    mv "$pkgdir/lib" "$pkgdir/usr/"

    # install pacman hooks
    install -Dm644 "$srcdir/linux-besrv-01-depmod.hook" "$pkgdir/usr/share/libalpm/hooks/linux-besrv-01-depmod.hook"
    install -Dm644 "$srcdir/linux-besrv-02-initcpio.hook" "$pkgdir/usr/share/libalpm/hooks/linux-besrv-02-initcpio.hook"
    install -Dm644 "$srcdir/linux-besrv-remove.hook" "$pkgdir/usr/share/libalpm/hooks/linux-besrv-remove.hook"

    # install sysctl tweaks
    install -Dm644 "$srcdir/sysctl-linux-besrv.conf" "$pkgdir/usr/lib/sysctl.d/60-linux-besrv.conf"
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

    # add objtool for external module building and enabled VALIDATION_STACK option
    install -Dm755 tools/objtool/objtool \
        "$pkgdir/usr/src/linux-$_kernver/tools/objtool/objtool"

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
