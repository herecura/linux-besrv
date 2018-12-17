# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-besrv
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=4.14
_patchver=89
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
    'ABAF11C65A2970B130ABE3C479BE3E4300411886'
    '647F28654894E3BD457199BE38DBBDC86092693E'
)

source=(
    "linux-stable::git+https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git#tag=${_tag}?signed"
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

## extra patches
_extrapatches=(
)
if [[ ${#_extrapatches[@]} -ne 0 ]]; then
    source=( "${source[@]}"
        "${_extrapatches[@]}"
    )
fi

sha512sums=('SKIP'
            'a6ecf9e6fb2860a955229adb9d554e7fc0b65d2d07f40fcbc970c0c8f6d0dea5bd8e61e4405a13ceaab84ae0eff50eff5d4662ca215e4f5c2bb7a5bb01d18469'
            '75f580633a48a15efa83e44a2e091ba33e1d615107eb192349b7ff3ea6aec3230f4206795747e238fe015d511125ab78b58571904577dd4eb687bba937ad95a6'
            '7762ad6385306dc16c5bd15ffe24caf7c73a1defd594aaf9620039554ce8fb9cf284a6919a8909cbc01a2b9d7b8f88b8fc35f1913a74aca2323c78b804370326'
            'f03250e32620071f27d33dbda859958ecbb206f2723a3c14f4f41734435011c87b4809bda558d687393d9fd2665531904f8963f1038f0bf8fb5598adc1d0518e'
            'e7ba6fcf986022ec56614b1acedf1e6ad723ffea12f8bf73741eef317da59f57b9df83e1800ea3e9b2d9e25207e6ac7fe4286927602d82435e1aa6525ceed0dc'
            '85c73eb30f4cb15b8eeadff19dbe08cb16d1ff0cdb0c0352f8647f4b6eb493fbb6d83b2cb327119ce7e779c7cbdcddab4631dc2f946879a00b66c99afa021bf6')

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
    cat "$srcdir/config-server.x86_64" >./.config
    if [[ "$_kernelname" != "" ]]; then
        sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"\U$_kernelname\"|g" ./.config
    fi

    # set extraversion to pkgrel
    msg2 "set extraversion to $pkgrel"
    sed -ri "s|^(EXTRAVERSION =).*|\1 -$pkgrel|" Makefile

    # don't run depmod on 'make install'. We'll do this ourselves in packaging
    sed -i '2iexit 0' scripts/depmod.sh

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

    cd "$srcdir/linux-stable"

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

    KARCH=x86

    install -dm755 "$pkgdir/usr/lib/modules/$_kernver"
    cd "$pkgdir/usr/lib/modules/$_kernver"
    ln -sf ../../../src/linux-$_kernver build
    cd "$srcdir/linux-stable"
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

    # cleanup other architectures
    for arch in "$pkgdir/usr/src/linux-$_kernver"/arch/*/; do
        [[ $arch = */$KARCH/ ]] && continue
        echo "Removing ./arch/$(basename "$arch")"
        rm -r "$arch"
    done
    for arch in "$pkgdir/usr/src/linux-$_kernver"/tools/perf/arch/*/; do
        [[ $arch = */$KARCH/ ]] && continue
        echo "Removing ./tools/perf/arch/$(basename "$arch")"
        rm -r "$arch"
    done

    # remove object files
    find "$pkgdir/usr/src/linux-$_kernver" -type f -name '*.o' -printf 'Removing %P\n' -delete

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
            *application/x-pie-executable*) # Relocatable binaries
                /usr/bin/strip $STRIP_SHARED "$binary"
                ;;
        esac
    done

    # "fix" owner
    chown -R root:root "$pkgdir/usr/src/linux-$_kernver"
    # "fix" permissions
    chmod -Rc u=rwX,go=rX "$pkgdir"
}
