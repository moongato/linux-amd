# Maintainer: moongato <moongato at gmail dot com>
# Contibutor: Steven De Bondt <egnappah at gmail dot com>

pkgbase=linux-amd
_srcname=linux
gitver=v5.8.10
pkgver=5.8.10
pkgrel=1
arch=('x86_64')
url="https://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'libelf')
options=('!strip')
#_muqss_patch=0001-MultiQueue-Skiplist-Scheduler-v0.202.patch
_prjc_patch="0009-prjc_v5.8-r3.patch"
_fsgsbase_path=fsgsbase-patches-v2
_fsgsbase_patch=0001-fsgsbase-patches.patch
_uksm_patch=uksm-5.8.patch
_bfq_rev_patch="0001-bfq-reverts.patch"
_bfq_patch=5.8-bfq-dev-lucjan-v12-r2K200909.patch

source=("git+https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git#tag=$gitver"
        # the main kernel config files
        config
        # standard config files for mkinitcpio ramdisk
        "${pkgbase}.preset"
	# gcc patch
	enable_additional_cpu_optimizations_for_gcc_v10.1+_kernel_v5.8+.patch
        # muqss patch
        #http://ck.kolivas.org/patches/muqss/5.0/5.7/${_muqss_patch}
        # project-c patch
        #https://gitlab.com/alfredchen/projectc/-/raw/master/5.8/${_prjc_patch}
        https://github.com/Frogging-Family/linux-tkg/raw/master/linux58-tkg/linux58-tkg-patches/${_prjc_patch}
        # fsgs patch
        https://github.com/sirlucjan/kernel-patches/raw/master/5.8/${_fsgsbase_path}/${_fsgsbase_patch}
        # -O3 for all arches patch
        0001-init-Kconfig-enable-O3-for-all-arches.patch
        # uksm patch
        https://github.com/dolohow/uksm/raw/master/v5.x/${_uksm_patch}
        # bfq patches
        https://github.com/sirlucjan/kernel-patches/raw/master/5.8/bfq-reverts-v2-all/${_bfq_rev_patch}
        https://github.com/sirlucjan/kernel-patches/raw/master/5.8/bfq-dev-lucjan/${_bfq_patch}
        # archlinux patches
        0001-ZEN-Add-sysctl-and-CONFIG-to-disallow-unprivileged-CLONE.patch
        0002-virt-vbox-Add-support-for-the-new-VBG_IOCTL_ACQUIRE_GUEST_CAP.patch
)
sha256sums=('SKIP'
            # config
            'a6a3a10806865cfb74afdf95ddb892ddca834b659e29d2310837bcac7b9814df'
            # .preset file
            '71caf34adf69e9e2567a38cfc951d1c60b13dbe87f58a9acfeb3fe48ffdc9d08'
            # gcc patch
            '8f0ac7129abece61f11b642167ca85450fd7d57d3b60e0675c2ea8497b4c7b84'
            # muqss patch
            #'8ab5ddbbee13d271af93eb4cd00434fcb4f0444968b1b865ad30dab16c833bf7'
            # project-c patch
            'f5dbff4833a2e3ca94c202e5197894d5f1006c689ff149355353e77d2e17c943'
            # fsgs patch
            '27345951e9cd308da8f70c6d0b57f11745a67c61c3df017f3eba6242b045e63b'
            # O3 patch
            'de912c6d0de05187fd0ecb0da67326bfde5ec08f1007bea85e1de732e5a62619'
            # uksm patch
            '0389c65d8357f8b22f65aceaf9ceda5a3c76e60ca34f713ff9a09ec379f51dc7'
            # bfq patches
            '6e7785ac437243165302b889a3bc0cdcdfc25aa1090e2f876fe60b624f6cb872'
            '8ea5870b7e0f7be60e859d112f54554c8c2b84b0f9069603ec5e91c2ebe3efd7'
            # archlinux patches
            '49a2dd5231e2a492c7d31f165f679ea203e91fe12a472d3b0074f539d17caa63'
            '754a7eb440e822584bb78f4662af87b03a00565b07319630e189de0e753a485b'
)

_kernelname=${pkgbase#linux}

pkgver() {
  echo $pkgver
}

prepare() {
  cd "${_srcname}"
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
  msg2 "Preparing kernel"
  yes "" | make prepare

  # load configuration
  msg2 "Preparing config"
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
  export CFLAGS="-march=znver1 -mtune=znver1 -O3 -pipe -fstack-protector-strong"
  export CXXFLAGS="${CFLAGS}"
  make ${MAKEFLAGS} LOCALVERSION= bzImage modules
}

_package() {
  pkgdesc="Linux kernel aimed at the latest AMD CPU based hardware"
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
  pkgdesc="Header files and scripts for building modules for Linux kernel aimed at the latest AMD CPU based hardware"
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
  while read modarch; do
   rm -rf $modarch
  done <<< $(find "${pkgdir}"/usr/lib/modules/${_kernver}/build/arch/ -maxdepth 1 -mindepth 1 -type d | grep -v /x86$)

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
