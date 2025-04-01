pkgbase="linux-888"
_major=6.14
_minor=0
pkgver=${_major}.${_minor}
_stable=${_major}
_srcname=linux-${_stable}
pkgdesc='A kernel based on linux-cachyos-bore-lto'
pkgrel=1
arch=('x86_64')
url='https://github.com/archlinux/linux'
license=('GPL-2.0-only')
options=('!strip' '!debug' '!lto')
makedepends=(
  bc
  cpio
  gettext
  libelf
  pahole
  perl
  python
  tar
  xz
  zstd
  clang
  llvm
  lld
)

_cfgsource="https://raw.githubusercontent.com/CachyOS/linux-cachyos/master"
_patchsource="https://raw.githubusercontent.com/cachyos/kernel-patches/master/${_major}"
source=(
    "https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.xz"
    "${_cfgsource}/linux-cachyos-bore/auto-cpu-optimization.sh"
    "${_patchsource}/all/0001-cachyos-base-all.patch"
    "${_patchsource}/sched/0001-bore-cachy.patch"
    "${_patchsource}/misc/dkms-clang.patch"
    "Add-14-USB-device-IDs-for-Qualcomm-WCN785x.patch")

BUILD_FLAGS=(
    CC=clang
    LD=ld.lld
    LLVM=1
    LLVM_IAS=1
)

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER="$pkgbase"
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  git clone --depth 1 --branch ath-next https://git.kernel.org/pub/scm/linux/kernel/git/ath/ath.git
  cd "$_srcname"

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  echo "Applying ath driver from ath-next..."
  rm -rf "${srcdir}/${_srcname}/Documentation/devicetree/bindings/net/wireless"
  rm -rf "${srcdir}/${_srcname}/drivers/net/wireless/ath"
  cp -r "${srcdir}/ath/Documentation/devicetree/bindings/net/wireless" "${srcdir}/${_srcname}/Documentation/devicetree/bindings/net/"
  cp -r "${srcdir}/ath/drivers/net/wireless/ath" "${srcdir}/${_srcname}/drivers/net/wireless/"

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    src="${src%.zst}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  echo "Setting config..."
  yes "" | make "${BUILD_FLAGS[@]}" localmodconfig >/dev/null

  chmod +x "${srcdir}"/auto-cpu-optimization.sh
  "${srcdir}"/auto-cpu-optimization.sh

  scripts/config -e CACHY

  scripts/config -e SCHED_BORE

  scripts/config -e LTO_CLANG_THIN

  scripts/config -d HZ_300 \
            -e "HZ_1000" \
            --set-val HZ "1000"

  scripts/config -d CPU_FREQ_DEFAULT_GOV_SCHEDUTIL \
            -e CPU_FREQ_DEFAULT_GOV_PERFORMANCE

  scripts/config -d HZ_PERIODIC \
            -d NO_HZ_IDLE \
            -e NO_HZ_FULL_NODEF \
            -e NO_HZ_FULL \
            -e NO_HZ \
            -e NO_HZ_COMMON \
            -e CONTEXT_TRACKING

  scripts/config -e PREEMPT_DYNAMIC \
            -e PREEMPT \
            -d PREEMPT_VOLUNTARY \
            -d PREEMPT_LAZY \
            -d PREEMPT_NONE

  scripts/config -d CC_OPTIMIZE_FOR_PERFORMANCE \
            -e CC_OPTIMIZE_FOR_PERFORMANCE_O3

  scripts/config -m TCP_CONG_CUBIC \
            -e TCP_CONG_BBR \
            -e DEFAULT_BBR \
            --set-str DEFAULT_TCP_CONG bbr \
            -m NET_SCH_FQ_CODEL \
            -e NET_SCH_FQ \
            -d DEFAULT_FQ_CODEL \
            -e DEFAULT_FQ \
            -d TCP_CONG_BIC \
            -d TCP_CONG_WESTWOOD \
            -d TCP_CONG_HTCP \
            -d TCP_CONG_HSTCP \
            -d TCP_CONG_HYBLA \
            -d TCP_CONG_VEGAS \
            -d TCP_CONG_NV \
            -d TCP_CONG_SCALABLE \
            -d TCP_CONG_LP \
            -d TCP_CONG_VENO \
            -d TCP_CONG_YEAH \
            -d TCP_CONG_ILLINOIS \
            -d TCP_CONG_DCTCP \
            -d TCP_CONG_CDG \
            -d NET_SCH_HFSC \
            -d NET_SCH_PRIO \
            -d NET_SCH_MULTIQ \
            -d NET_SCH_RED \
            -d NET_SCH_SFB \
            -d NET_SCH_SFQ \
            -d NET_SCH_TEQL \
            -d NET_SCH_TBF \
            -d NET_SCH_CBS \
            -d NET_SCH_ETF \
            -d NET_SCH_TAPRIO \
            -d NET_SCH_GRED \
            -d NET_SCH_NETEM \
            -d NET_SCH_DRR \
            -d NET_SCH_MQPRIO \
            -d NET_SCH_SKBPRIO \
            -d NET_SCH_CHOKE \
            -d NET_SCH_QFQ \
            -d NET_SCH_CODEL \
            -d NET_SCH_HHF \
            -d NET_SCH_PIE \
            -d NET_SCH_INGRESS \
            -d NET_SCH_PLUG \
            -d NET_SCH_ETS

  scripts/config -d TRANSPARENT_HUGEPAGE_MADVISE \
            -e TRANSPARENT_HUGEPAGE_ALWAYS

  scripts/config -e USER_NS

  # Additions
  # Base
  scripts/config -e XFRM \
              -m XFRM_ALGO \
              -m XFRM_USER \
              -e XFRM_SUB_POLICY \
              -e XFRM_MIGRATE \
              -e XFRM_STATISTICS \
              -m NET_IP_TUNNEL \
              -m NET_UDP_TUNNEL \
              -m NF_CONNTRACK \
              -m NF_LOG_SYSLOG \
              -e NF_CONNTRACK_MARK \
              -e NF_CONNTRACK_SECMARK \
              -e NF_CONNTRACK_ZONES \
              -e NF_CONNTRACK_PROCFS \
              -e NF_CONNTRACK_EVENTS \
              -e NF_CONNTRACK_TIMEOUT \
              -e NF_CONNTRACK_TIMESTAMP \
              -e NF_CONNTRACK_LABELS \
              -e NF_CT_PROTO_DCCP \
              -e NF_CT_PROTO_SCTP \
              -e NF_CT_PROTO_UDPLITE \
              -m NF_NAT \
              -e NF_NAT_MASQUERADE \
              -m NFT_CT \
              -m NFT_MASQ \
              -m NFT_NAT \
              -m NFT_REJECT \
              -m NFT_REJECT_INET \
              -m NFT_COMPAT \
              -m NETFILTER_XT_SET \
              -m NETFILTER_XT_TARGET_LOG \
              -m NETFILTER_XT_NAT \
              -m NETFILTER_XT_TARGET_MASQUERADE \
              -m NETFILTER_XT_MATCH_ADDRTYPE \
              -m NETFILTER_XT_MATCH_CONNTRACK \
              -m NETFILTER_XT_MATCH_HL \
              -m NETFILTER_XT_MATCH_LIMIT \
              -m NETFILTER_XT_MATCH_RECENT \
              -m IP_SET \
              -m NF_DEFRAG_IPV4 \
              -m NFT_REJECT_IPV4 \
              -m NF_REJECT_IPV4 \
              -m IP_NF_IPTABLES \
              -m IP_NF_FILTER \
              -m IP_NF_TARGET_REJECT \
              -m IP_NF_NAT \
              -m NFT_COMPAT_ARP \
              -m IP6_NF_IPTABLES_LEGACY \
              -m NFT_REJECT_IPV6 \
              -m NF_REJECT_IPV6 \
              -m NF_LOG_IPV6 \
              -m IP6_NF_IPTABLES \
              -m IP6_NF_MATCH_RT \
              -m IP6_NF_FILTER \
              -m IP6_NF_NAT \
              -m NF_DEFRAG_IPV6 \
              -m STP \
              -m BRIDGE \
              -e BRIDGE_IGMP_SNOOPING \
              -e BRIDGE_MRP \
              -e BRIDGE_CFM \
              -m LLC \
              -m NET_CLS_U32 \
              -e CLS_U32_PERF \
              -e CLS_U32_MARK \
              -m NET_ACT_CSUM \
              -e GRO_CELLS \
              -m NET_SELFTESTS \
              -m ENCLOSURE_SERVICES \
              -m SCSI_ENCLOSURE \
              -m SCSI_SAS_ATTRS \
              -m MII \
              -m WIREGUARD \
              -m TUN \
              -m PHYLIB \
              -e SWPHY \
              -e LED_TRIGGER_PHY \
              -m FIXED_PHY \
              -m MDIO_DEVICE \
              -m MDIO_BUS \
              -m FWNODE_MDIO \
              -m ACPI_MDIO \
              -m MDIO_DEVRES \
              -m USB_NET_DRIVERS \
              -m USB_RTL8152 \
              -m USB_USBNET \
              -m USB_NET_CDCETHER \
              -m USB_RTL8153_ECM \
              -m USB_STORAGE \
              -m USB_UAS \
              -m APPLE_MFI_FASTCHARGE \
              -m XFS_FS \
              -e XFS_SUPPORT_V4 \
              -e XFS_SUPPORT_ASCII_CI \
              -e XFS_POSIX_ACL \
              -e XFS_RT \
              -e XFS_DRAIN_INTENTS \
              -e XFS_LIVE_HOOKS \
              -e XFS_MEMORY_BUFS \
              -e XFS_BTREE_IN_MEM \
              -e XFS_ONLINE_SCRUB \
              -e XFS_ONLINE_REPAIR \
              -m OVERLAY_FS \
              -e OVERLAY_FS_REDIRECT_DIR \
              -e OVERLAY_FS_INDEX \
              -e OVERLAY_FS_XINO_AUTO \
              -e OVERLAY_FS_METACOPY \
              -m NTFS3_FS \
              -e NTFS3_LZX_XPRESS \
              -e NTFS3_FS_POSIX_ACL \
              -e SECURITY_NETWORK_XFRM \
              -m CRYPTO_CURVE25519_X86 \
              -m CRYPTO_CHACHA20_X86_64 \
              -m CRYPTO_POLY1305_X86_64 \
              -m CRYPTO_ARCH_HAVE_LIB_CHACHA \
              -m CRYPTO_LIB_CHACHA_GENERIC \
              -m CRYPTO_LIB_CHACHA \
              -m CRYPTO_ARCH_HAVE_LIB_CURVE25519 \
              -m CRYPTO_LIB_CURVE25519_GENERIC \
              -m CRYPTO_LIB_CURVE25519 \
              -m CRYPTO_ARCH_HAVE_LIB_POLY1305 \
              -m CRYPTO_LIB_POLY1305_GENERIC \
              -m CRYPTO_LIB_POLY1305 \
              -m CRYPTO_LIB_CHACHA20POLY1305 \
              -e NETFILTER_FAMILY_ARP \
              -m NF_TABLES \
              -e NF_TABLES_INET \
              -e NF_TABLES_NETDEV \
              -m NFT_CT \
              -m NFT_MASQ \
              -m NFT_NAT \
              -m NFT_REJECT \
              -m NFT_REJECT_INET \
              -m NFT_COMPAT \
              -e NF_TABLES_IPV4 \
              -m NFT_REJECT_IPV4 \
              -e NF_TABLES_ARP \
              -m NFT_COMPAT_ARP \
              -e NF_TABLES_IPV6 \
              -m NFT_REJECT_IPV6 \
              -m SND_RAWMIDI \
              -m SND_UMP \
              -e SND_UMP_LEGACY_RAWMIDI \
              -m SND_SEQ_MIDI_EVENT \
              -m SND_SEQ_MIDI \
              -m SND_SEQ_UMP_CLIENT \
              -m SND_USB_AUDIO \
              -e SND_USB_AUDIO_MIDI_V2 \
              -e SND_USB_AUDIO_USE_MEDIA_CONTROLLER \
              -m SND_USB_UA101 \
              -m SND_USB_USX2Y \
              -m SND_USB_US122L \
              -m CRYPTO_DEFLATE \
              -m CRYPTO_842 \
              -m CRYPTO_LZ4 \
              -m CRYPTO_LZ4HC \
              -m THINKPAD_ACPI \
              -m DM_INTEGRITY \
              -m HW_RANDOM_INTEL \
              -m HW_RANDOM_AMD \
              -m HID_APPLE \
              -m HID_LOGITECH_DJ \
              -e HID_BPF \
              -e LOGITECH_FF \
              -e LOGIRUMBLEPAD2_FF \
              -e LOGIG940_FF \
              -e LOGIWHEELS_FF

  # x86_64 cryptos
  scripts/config -m CRYPTO_BLOWFISH_X86_64 \
              -m CRYPTO_CAMELLIA_X86_64 \
              -m CRYPTO_CAMELLIA_AESNI_AVX_X86_64 \
              -m CRYPTO_CAMELLIA_AESNI_AVX2_X86_64 \
              -m CRYPTO_CAST5_AVX_X86_64 \
              -m CRYPTO_CAST6_AVX_X86_64 \
              -m CRYPTO_DES3_EDE_X86_64 \
              -m CRYPTO_SERPENT_SSE2_X86_64 \
              -m CRYPTO_SERPENT_AVX_X86_64 \
              -m CRYPTO_SERPENT_AVX2_X86_64 \
              -m CRYPTO_SM4_AESNI_AVX_X86_64 \
              -m CRYPTO_SM4_AESNI_AVX2_X86_64 \
              -m CRYPTO_TWOFISH_X86_64 \
              -m CRYPTO_TWOFISH_X86_64_3WAY \
              -m CRYPTO_TWOFISH_AVX_X86_64 \
              -m CRYPTO_ARIA_AESNI_AVX_X86_64 \
              -m CRYPTO_ARIA_AESNI_AVX2_X86_64 \
              -m CRYPTO_ARIA_GFNI_AVX512_X86_64 \
              -m CRYPTO_AEGIS128_AESNI_SSE2 \
              -m CRYPTO_NHPOLY1305_SSE2 \
              -m CRYPTO_NHPOLY1305_AVX2 \
              -m CRYPTO_SM3_AVX_X86_64

  # Reduction
  # Unused features
  scripts/config -d ACCESSIBILITY \
            -d HIBERNATION \
            -d NETWORK_FILESYSTEMS \
            -d MEMORY_FAILURE \
            -d ANDROID_BINDER_IPC
            
  # Unused devices
  scripts/config -d USB_NET_AX8817X \
              -d USB_NET_AX88179_178A \
              -d USB_NET_NET1080 \
              -d USB_NET_CDC_SUBSET \
              -d USB_NET_ZAURUS \
              -d SENSORS_SPD5118 \
              -d USB_NET_CDC_NCM \
              -d FUSION \
              -d WLAN_VENDOR_ATMEL \
              -d WLAN_VENDOR_BROADCOM \
              -d WLAN_VENDOR_INTERSIL \
              -d WLAN_VENDOR_MARVELL \
              -d WLAN_VENDOR_MEDIATEK \
              -d WLAN_VENDOR_MICROCHIP \
              -d WLAN_VENDOR_PURELIFI \
              -d WLAN_VENDOR_RALINK \
              -d WLAN_VENDOR_REALTEK \
              -d WLAN_VENDOR_RSI \
              -d WLAN_VENDOR_SILABS \
              -d WLAN_VENDOR_ST \
              -d WLAN_VENDOR_TI \
              -d WLAN_VENDOR_ZYDAS \
              -d WLAN_VENDOR_QUANTENNA \
              -d MEGARAID_NEWGEN

  # Performance
  scripts/config -d AUDIT \
            -d FTRACE \
            -d KEXEC \
            -d KEXEC_FILE \
            -d KPROBES \
            -d PROC_KCORE \
            -d PROFILING \
            -d SECURITY_APPARMOR \
            -d SECURITY_LOADPIN \
            -d SECURITY_LOCKDOWN_LSM \
            -d SECURITY_SAFESETID \
            -d SECURITY_LANDLOCK \
            -d SECURITY_SMACK \
            -d SECURITY_TOMOYO \
            -d SECURITY_YAMA \
            --set-str LSM "" \
            -d WATCHDOG \
            -d QUOTA \
            -d XFS_QUOTA \
            -d TMPFS_QUOTA \
            -d ZSWAP \
            -d BTRFS_FS \
            -d HAMRADIO

  # Platform-specific
  scripts/config -d CHROME_PLATFORMS \
            -d GOOGLE_FIRMWARE \
            -d MACINTOSH_DRIVERS \
            -d MELLANOX_PLATFORM \
            -d SURFACE_PLATFORMS \
            -d X86_PLATFORM_DRIVERS_DELL \
            -d X86_PLATFORM_DRIVERS_HP

  # Virtualisation
  scripts/config -d XEN

  # Legacy Hardware
  scripts/config -d AGP \
            -d ATA_SFF \
            -d ISDN \
            -d SERIAL_NONSTANDARD

  # Bluetooth
  scripts/config -d BT_HCIBTUSB_BCM \
            -d BT_HCIBTUSB_RTL \
            -d BT_HCIBTUSB_MTK

  # DRM
  scripts/config -d DRM_AMDGPU_CIK \
            -d DRM_AMDGPU_SI

  # Debugging
  scripts/config -d PM_DEBUG \
            -d ACPI_DEBUG \
            -d PNP_DEBUG_MESSAGES \
            -d DM_DEBUG \
            -d ATH12K_DEBUG \
            -d SND_DEBUG \
            -d VIRTIO_DEBUG \
            -d DYNAMIC_DEBUG \
            -d DYNAMIC_DEBUG_CORE \
            -d DEBUG_RODATA_TEST \
            -d DEBUG_WX \
            -d SHRINKER_DEBUG \
            -d DEBUG_SHIRQ \
            -d SCHED_DEBUG \
            -d RUNTIME_TESTING_MENU \
            -d MEMTEST

  ### Rewrite configuration
  echo "Rewrite configuration..."
  yes "" | make "${BUILD_FLAGS[@]}" prepare >/dev/null
  yes "" | make "${BUILD_FLAGS[@]}" config >/dev/null

  ### Prepared version
  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd "$_srcname"
  make "${BUILD_FLAGS[@]}" -j"$(nproc)" all
  make -C tools/bpf/bpftool vmlinux.h feature-clang-bpf-co-re=1
}

_package() {
  pkgdesc="The $pkgdesc kernel and modules"
  depends=('coreutils' 'kmod' 'initramfs')
  optdepends=('wireless-regdb: to set the correct wireless channels of your country'
              'linux-firmware: firmware images needed for some devices'
              'modprobed-db: Keeps track of EVERY kernel module that has ever been probed - useful for those of us who make localmodconfig'
              'scx-scheds: to use sched-ext schedulers')
  provides=(VIRTUALBOX-GUEST-MODULES WIREGUARD-MODULE KSMBD-MODULE UKSMD-BUILTIN NTSYNC-MODULE VHBA-MODULE ADIOS-MODULE)

  cd "$_srcname"

  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

  echo "Installing boot image..."
  # systemd expects to find the kernel here to allow hibernation
  # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

  # Used by mkinitcpio to name the kernel
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  echo "Installing modules..."
  ZSTD_CLEVEL=19 make "${BUILD_FLAGS[@]}" INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
      DEPMOD=/doesnt/exist  modules_install  # Suppress depmod

  # remove build links
  rm "$modulesdir"/build
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
  depends=('pahole' "${pkgbase}")
  depends+=(clang llvm lld)

  cd "${_srcname}"
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux tools/bpf/bpftool/vmlinux.h
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
  cp -t "$builddir" -a scripts
  ln -srt "$builddir" "$builddir/scripts/gdb/vmlinux-gdb.py"

  # required when STACK_VALIDATION is enabled
  install -Dt "$builddir/tools/objtool" tools/objtool/objtool

  # required when DEBUG_INFO_BTF_MODULES is enabled
  if [ -f tools/bpf/resolve_btfids/resolve_btfids ]; then
    install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids
  fi

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/x86" -a arch/x86/include
  install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */x86/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -Sib "$file")" in
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

  echo "Stripping vmlinux..."
  strip -v $STRIP_STATIC "$builddir/vmlinux"

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=("$pkgbase")
pkgname+=("$pkgbase-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

b2sums=('SKIP'
        'SKIP'
        'SKIP'
        'SKIP'
        'SKIP'
        'SKIP')
