# Maintainer: moongato <moongato at gmail dot com>
# Contibutor: Steven De Bondt <egnappah at gmail dot com>

pkgbase=linux-amd
_srcname=linux
gitver=v5.7.7
pkgver=5.7.7
pkgrel=1
arch=('x86_64')
url="https://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'libelf')
options=('!strip')
_muqss_patch=0001-MultiQueue-Skiplist-Scheduler-v0.202.patch
#_bmq_patch="bmq_v5.6-r4.patch"
_fsgsbase_path=fsgsbase-patches-v5
_fsgsbase_patch=0001-fsgsbase-patches.patch
_uksm_patch=uksm-5.7.patch
source=(
        'git+https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git'
        # the main kernel config files
        config
        # standard config files for mkinitcpio ramdisk
        "${pkgbase}.preset"
	# gcc patch
	enable_additional_cpu_optimizations_for_gcc_v10.1+_kernel_v5.7+.patch
        # muqss patch
        http://ck.kolivas.org/patches/muqss/5.0/5.7/${_muqss_patch}
        # bmq patch
        #https://gitlab.com/alfredchen/bmq/raw/master/5.7/${_bmq_patch}
        # fsgs patch
        https://github.com/sirlucjan/kernel-patches/raw/master/5.7/${_fsgsbase_path}/${_fsgsbase_patch}
        # -O3 for all arches patch
        0001-init-Kconfig-enable-O3-for-all-arches.patch
        # uksm patch
        https://github.com/dolohow/uksm/raw/master/v5.x/${_uksm_patch}
        # unfuck muqss
          "unfuck-ck1.patch::https://github.com/ckolivas/linux/commit/0b69e633d6b0b08ae8547dc4099c8c0985019553.patch"
        # archlinux patches
        0001-ZEN-Add-sysctl-and-CONFIG-to-disallow-unprivileged-CLONE_NEWUSER.patch
        0002-PCI-EDR-Log-only-ACPI_NOTIFY_DISCONNECT_RECOVER-events.patch
        0003-ALSA-usb-audio-Fix-packet-size-calculation.patch
        0004-drm-amd-display-Only-revalidate-bandwidth-on-medium-and-fast-updates.patch
)
sha256sums=('SKIP'
            # config
            '283f7acab9a29fe2c1c02dcb35cd572e48a0874fec157868e1bfd52d02b88755'
            # .preset file
            '71caf34adf69e9e2567a38cfc951d1c60b13dbe87f58a9acfeb3fe48ffdc9d08'
            # gcc patch
            '1f56a2466bd9b4477925682d8f944fabb38727140e246733214fe50aa326fc47'
            # muqss patch
            '8ab5ddbbee13d271af93eb4cd00434fcb4f0444968b1b865ad30dab16c833bf7'
            # bmq patch
            #'1b95d36635c7dc48ce45a33d6b1f4eb6d34f51600901395d28fd22f28daee8e9'
            # fsgs patch
            '2e0e8413302c2b6cd4e7ee6960198eb0cd9cc3e80c52b6f14054a196f0f48984'
            # O3 patch
            'de912c6d0de05187fd0ecb0da67326bfde5ec08f1007bea85e1de732e5a62619'
            # uksm patch
            'c28dc0d30bba3eedae9f5cf98a686bdfb25a0326df4e8c417d37a36597d21b37'
            # unfuck muqss
            '5a08ac04975fe784d16d6c8ec2be733c73cdcfc19795f5c7b97d7a1aa7f12328'
            # archlinux patches
            '211d7bcd02f146b28daecfeff410c66834b8736de1cad09158f8ec9ecccdcca6'
            '69dfd528a2ad7a57a5036c9250a2f99dc815eef011cdc17c323c49affdb051de'
            '863f4d199f333fbbba9d42c287b566050d3716bfbd5aed9acf1f3745f8df3a2f'
            '495d52edab5e226d24aeb3467f5f31366cf268b0cdfa6ea714e162e01067a0eb' 
)

_kernelname=${pkgbase#linux}

pkgver() {
  echo $pkgver
}

prepare() {
  cd "${_srcname}"
  #We want to base this on the release
  git checkout tags/$gitver
  if [ "${CARCH}" = "x86_64" ]; then
    cat "${srcdir}/config" > ./.config
  else
    echo "Sorry, non x86_64 arch not supported."
      exit 2
  fi

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

  # Implement all packaged patches.
  #git apply ../*.patch
  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  # get kernel version
  yes "" | make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  #make oldconfig # using old config from previous kernel version
  make olddefconfig # old config from previous kernel, defaults for new options

  # Optionally load needed modules for the make localmodconfig
  # See https://aur.archlinux.org/packages/modprobed-db
  make LSMOD=$HOME/.config/modprobed.db localmodconfig
}

build() {
  cd "${_srcname}"

  #Force zen architecture optimisation.
  export CFLAGS="-march=amdfam10 -mtune=amdfam10 -O3 -pipe -fstack-protector-strong"
  export CXXFLAGS="${CFLAGS}"
  make ${MAKEFLAGS} LOCALVERSION= bzImage modules
}

_package() {
  pkgdesc="Linux kernel for AMD CPU based hardware"
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('linux')
  backup=("etc/mkinitcpio.d/${pkgbase}.preset")
  install=linux.install

  cd "${_srcname}"

  KARCH=x86

  # get kernel version
  _kernver="$(make LOCALVERSION= kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
  make LOCALVERSION= INSTALL_MOD_PATH="${pkgdir}" modules_install
  cp arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-${pkgbase}"

  # set correct depmod command for install
  cp -f "${startdir}/${install}" "${startdir}/${install}.pkg"
  true && install=${install}.pkg
  sed \
    -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/" \
    -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/" \
    -i "${startdir}/${install}"

  # install mkinitcpio preset file for kernel
  install -D -m644 "${srcdir}/${pkgbase}.preset" "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"
  sed \
    -e "1s|'linux.*'|'${pkgbase}'|" \
    -e "s|ALL_kver=.*|ALL_kver=\"/boot/vmlinuz-${pkgbase}\"|" \
    -e "s|default_image=.*|default_image=\"/boot/initramfs-${pkgbase}.img\"|" \
    -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs-${pkgbase}-fallback.img\"|" \
    -i "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # remove build and source links
  rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
  # remove the firmware
  rm -rf "${pkgdir}/lib/firmware"
  # make room for external modules
  ln -s "../extramodules-${_basekernel}${_kernelname:--ARCH}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
  # add real version for building modules and running depmod from post_install/upgrade
  mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}"
  echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}/version"

  # Now we call depmod...
  depmod -b "${pkgdir}" -F System.map "${_kernver}"

  # move module tree /lib -> /usr/lib
  mkdir -p "${pkgdir}/usr"
  mv "${pkgdir}/lib" "${pkgdir}/usr/"

  # add vmlinux
  install -D -m644 vmlinux "${pkgdir}/usr/lib/modules/${_kernver}/build/vmlinux" 

  # add System.map
  install -D -m644 System.map "${pkgdir}/boot/System.map-${_kernver}"
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for Linux kerel for AMD CPU based hardware"
  provides=('linux-headers')

  install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"

  cd "${_srcname}"
  install -D -m644 Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/Makefile"
  install -D -m644 kernel/Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/kernel/Makefile"
  install -D -m644 .config \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/.config"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include"

  for i in acpi asm-generic config crypto drm generated keys linux math-emu \
    media net pcmcia scsi sound trace uapi video xen; do
    cp -a include/${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/include/"
  done

  # copy arch includes for external modules
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86"
  cp -a arch/x86/include "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86/"

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers "${pkgdir}/usr/lib/modules/${_kernver}/build"
  cp -a scripts "${pkgdir}/usr/lib/modules/${_kernver}/build"

  # fix permissions on scripts dir
  chmod og-w -R "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/.tmp_versions"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel"

  cp arch/${KARCH}/Makefile "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"

  if [ "${CARCH}" = "i686" ]; then
    cp arch/${KARCH}/Makefile_32.cpu "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"
  fi

  cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel/"

  # add dm headers
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"
  cp drivers/md/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"

  # add inotify.h
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux"
  cp include/linux/inotify.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux/"

  # add wireless headers
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"
  cp net/mac80211/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"

  # copy in Kconfig files
  for i in $(find . -name "Kconfig*"); do
    mkdir -p "${pkgdir}"/usr/lib/modules/${_kernver}/build/`echo ${i} | sed 's|/Kconfig.*||'`
    cp ${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/${i}"
  done

  # Fix file conflict with -doc package
  rm "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild"/Kconfig.*-*
  rm "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/Kconfig"

  # Add objtool for CONFIG_STACK_VALIDATION
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/tools"
  cp -a tools/objtool "${pkgdir}/usr/lib/modules/${_kernver}/build/tools"

  chown -R root.root "${pkgdir}/usr/lib/modules/${_kernver}/build"
  find "${pkgdir}/usr/lib/modules/${_kernver}/build" -type d -exec chmod 755 {} \;

  # strip scripts directory
  find "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
    case "$(file -bi "${binary}")" in
      *application/x-sharedlib*) # Libraries (.so)
        /usr/bin/strip ${STRIP_SHARED} "${binary}";;
      *application/x-archive*) # Libraries (.a)
        /usr/bin/strip ${STRIP_STATIC} "${binary}";;
      *application/x-executable*) # Binaries
        /usr/bin/strip ${STRIP_BINARIES} "${binary}";;
    esac
  done

  # remove unneeded architectures
  rm -rf "${pkgdir}"/usr/lib/modules/${_kernver}/build/arch/{alpha,arc,arm,arm26,arm64,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,metag,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,xtensa}

  #Fix missing vdso files after overhaul
  cp -r "include/vdso/" "${pkgdir}/usr/lib/modules/${_kernver}/build/include/"
}

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#${pkgbase}}")
    _package${_p#${pkgbase}}
  }"
done
