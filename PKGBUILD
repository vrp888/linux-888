# Maintainer: Peter Jung ptr1337 <admin@ptr1337.dev>
# Maintainer: Piotr Gorski <piotrgorski@cachyos.org>
# Maintainer: Vasiliy Stelmachenok <ventureo@cachyos.org>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

### BUILD OPTIONS
# Set these variables to ANYTHING that is not null or choose proper variable to enable them

### Selecting CachyOS config
: "${_cachy_config:=yes}"

### Selecting the CPU scheduler
# ATTENTION - only one of the following values can be selected:
# 'bore' - select 'Burst-Oriented Response Enhancer'
# 'bmq' - select 'BMQ Scheduler'
# 'hardened' - select 'BORE Scheduler hardened' ## kernel with hardened config and hardening patches with the bore scheduler
# 'cachyos' - select 'CachyOS Default Scheduler (BORE)'
# 'eevdf' - select 'EEVDF Scheduler'
# 'rt' - select EEVDF, but includes a series of realtime patches
# 'rt-bore' - select Burst-Oriented Response Enhancer, but includes a series of realtime patches
: "${_cpusched:=bore}"

### Tweak kernel options prior to a build via nconfig
: "${_makenconfig:=no}"

### Tweak kernel options prior to a build via xconfig
: "${_makexconfig:=no}"

# Compile ONLY used modules to VASTLYreduce the number of modules built
# and the build time.
#
# To keep track of which modules are needed for your specific system/hardware,
# give module_db script a try: https://aur.archlinux.org/packages/modprobed-db
# This PKGBUILD read the database kept if it exists
#
# More at this wiki page ---> https://wiki.archlinux.org/index.php/Modprobed-db
: "${_localmodcfg:=no}"

# Path to the list of used modules
: "${_localmodcfg_path:="$HOME/.config/modprobed.db"}"

# Use the current kernel's .config file
# Enabling this option will use the .config of the RUNNING kernel rather than
# the ARCH defaults. Useful when the package gets updated and you already went
# through the trouble of customizing your config options.  NOT recommended when
# a new kernel is released, but again, convenient for package bumps.
: "${_use_current:=no}"

### Enable KBUILD_CFLAGS -O3
: "${_cc_harder:=yes}"

### Set performance governor as default
: "${_per_gov:=no}"

### Enable TCP_CONG_BBR3
: "${_tcp_bbr3:=yes}"

### Running with a 1000HZ, 750Hz, 600 Hz, 500Hz, 300Hz, 250Hz and 100Hz tick rate
: "${_HZ_ticks:=1000}"

## Choose between perodic, idle or full
### Full tickless can give higher performances in various cases but, depending on hardware, lower consistency.
: "${_tickrate:=full}"

## Choose between full(low-latency), lazy, voluntary or none
: "${_preempt:=full}"

### Transparent Hugepages
# ATTENTION - one of two predefined values should be selected!
# 'always' - always enable THP
# 'madvise' - madvise, prevent applications from allocating more memory resources than necessary
# More infos here:
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/performance_tuning_guide/sect-red_hat_enterprise_linux-performance_tuning_guide-configuring_transparent_huge_pages
: "${_hugepage:=always}"

# CPU compiler optimizations - Defaults to prompt at kernel config if left empty
# AMD CPUs : "k8" "k8sse3" "k10" "barcelona" "bobcat" "jaguar" "bulldozer" "piledriver" "steamroller" "excavator" "zen" "zen2" "zen3" "zen4"
# Intel CPUs : "mpsc"(P4 & older Netburst based Xeon) "atom" "core2" "nehalem" "westmere" "silvermont" "sandybridge" "ivybridge" "haswell" "broadwell" "skylake" "skylakex" "cannonlake" "icelake" "goldmont" "goldmontplus" "cascadelake" "cooperlake" "tigerlake" "sapphirerapids" "rocketlake" "alderlake"
# Other options :
# - "native_amd" (use compiler autodetection - Selecting your arch manually in the list above is recommended instead of this option)
# - "native_intel" (use compiler autodetection and will prompt for P6_NOPS - Selecting your arch manually in the list above is recommended instead of this option)
# - "generic" (kernel's default - to share the package between machines with different CPU µarch as long as they are x86-64)
#
: "${_processor_opt:=}"

# This does automatically detect your supported CPU and optimizes for it
: "${_use_auto_optimization:=yes}"

# Clang LTO mode, only available with the "llvm" compiler - options are "none", "full" or "thin".
# ATTENTION - one of three predefined values should be selected!
# "full: uses 1 thread for Linking, slow and uses more memory, theoretically with the highest performance gains."
# "thin: uses multiple threads, faster and uses less memory, may have a lower runtime performance than Full."
# "none: disable LTO
: "${_use_llvm_lto:=thin}"

# ATTENTION: Do not modify after this line
_is_lto_kernel() {
    [[ "$_use_llvm_lto" = "thin" || "$_use_llvm_lto" = "full" ]]
    return $?
}

_pkgsuffix="888"

pkgbase="linux-$_pkgsuffix"
_major=6.14
_minor=0
#_minorc=$((_minor+1))
_rcver=rc7
pkgver=${_major}.${_rcver}
#_stable=${_major}.${_minor}
#_stable=${_major}
_stable=${_major}-${_rcver}
_srcname=linux-${_stable}
#_srcname=linux-${_major}
pkgdesc='Linux BORE + LTO + Cachy Sauce Kernel by CachyOS with other patches and improvements - Release Candidate'
pkgrel=1
_kernver="$pkgver-$pkgrel"
_kernuname="${pkgver}-${_pkgsuffix}"
arch=('x86_64')
url="https://github.com/CachyOS/linux-cachyos"
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
)

_patchsource="https://raw.githubusercontent.com/cachyos/kernel-patches/master/${_major}"
source=(
    "https://github.com/torvalds/linux/archive/refs/tags/v${_major}-${_rcver}.tar.gz"
    "config"
    "auto-cpu-optimization.sh"
    "${_patchsource}/all/0001-cachyos-base-all.patch"
    "btusb-wcn785x.patch")

# LLVM makedepends
if _is_lto_kernel; then
    makedepends+=(clang llvm lld)
    source+=("${_patchsource}/misc/dkms-clang.patch")
    BUILD_FLAGS=(
        CC=clang
        LD=ld.lld
        LLVM=1
        LLVM_IAS=1
    )
fi

## List of CachyOS schedulers
case "$_cpusched" in
    cachyos|bore|rt-bore|hardened) # CachyOS Scheduler (BORE)
        source+=("${_patchsource}/sched/0001-bore-cachy.patch");;&
    bmq) ## Project C Scheduler
        source+=("${_patchsource}/sched/0001-prjc-cachy.patch");;
    hardened) ## Hardened Patches
        source+=("${_patchsource}/misc/0001-hardened.patch");;
esac

export KBUILD_BUILD_HOST=cachyos
export KBUILD_BUILD_USER="$pkgbase"
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

_die() { error "$@" ; exit 1; }

prepare() {
    git clone --depth 1 --branch ath-next https://git.kernel.org/pub/scm/linux/kernel/git/ath/ath.git
    cd "$_srcname"

    echo "Setting version..."
    echo "-$pkgrel" > localversion.10-pkgrel
    echo "${pkgbase#linux}" > localversion.20-pkgname

    echo "Applying ath patches from ath-next..."
    rm -rf "${srcdir}/${_srcname}/Documentation/devicetree/bindings/net/wireless"
    rm -rf "${srcdir}/${_srcname}/drivers/net/wireless/ath"
    cp -r "${srcdir}/ath/Documentation/devicetree/bindings/net/wireless" "${srcdir}/${_srcname}/Documentation/devicetree/bindings/net/"
    cp -r "${srcdir}/ath/drivers/net/wireless/ath" "${srcdir}/${_srcname}/drivers/net/wireless/"

    local src
    for src in "${source[@]}"; do
        src="${src%%::*}"
        # Skip nvidia patches
        [[ "$src" == "${_patchsource}"/misc/nvidia/*.patch ]] && continue
        src="${src##*/}"
        src="${src%.zst}"
        [[ $src = *.patch ]] || continue
        echo "Applying patch $src..."
        patch -Np1 < "../$src"
    done

    echo "Setting config..."
    cp ../config .config

    ### Select CPU optimization
    if [ -n "$_processor_opt" ]; then
        MARCH="${_processor_opt^^}"

        if [ "$MARCH" != "GENERIC" ]; then
            if [[ "$MARCH" =~ GENERIC_V[1-4] ]]; then
                X86_64_LEVEL="${MARCH//GENERIC_V}"
                scripts/config --set-val X86_64_VERSION "${X86_64_LEVEL}"
            else
                scripts/config -k -d CONFIG_GENERIC_CPU
                scripts/config -k -e "CONFIG_M${MARCH}"
            fi
        fi
    fi

    ### Use autooptimization
    if [ "$_use_auto_optimization" = "yes" ]; then
        "${srcdir}"/auto-cpu-optimization.sh
    fi

    ### Selecting CachyOS config
    if [ "$_cachy_config" = "yes" ]; then
        echo "Enabling CachyOS config..."
        scripts/config -e CACHY
    fi

    ### Selecting the CPU scheduler
    case "$_cpusched" in
        cachyos|bore|hardened) scripts/config -e SCHED_BORE;;
        bmq) scripts/config -e SCHED_ALT -e SCHED_BMQ;;
        eevdf) ;;
        rt) scripts/config -e PREEMPT_RT;;
        rt-bore) scripts/config -e SCHED_BORE -e PREEMPT_RT;;
        *) _die "The value $_cpusched is invalid. Choose the correct one again.";;
    esac

    echo "Selecting ${_cpusched^^} CPU scheduler..."

    ### Select LLVM level
    case "$_use_llvm_lto" in
        thin) scripts/config -e LTO_CLANG_THIN;;
        full) scripts/config -e LTO_CLANG_FULL;;
        none) scripts/config -e LTO_NONE;;
        *) _die "The value '$_use_llvm_lto' is invalid. Choose the correct one again.";;
    esac

    echo "Selecting '$_use_llvm_lto' LLVM level..."

    ### Select tick rate
    case "$_HZ_ticks" in
        100|250|500|600|750|1000)
            scripts/config -d HZ_300 -e "HZ_${_HZ_ticks}" --set-val HZ "${_HZ_ticks}";;
        300)
            scripts/config -e HZ_300 --set-val HZ 300;;
        *)
            _die "The value $_HZ_ticks is invalid. Choose the correct one again."
    esac

    echo "Setting tick rate to ${_HZ_ticks}Hz..."

    ### Select performance governor
    if [ "$_per_gov" = "yes" ]; then
        echo "Setting performance governor..."
        scripts/config -d CPU_FREQ_DEFAULT_GOV_SCHEDUTIL \
            -e CPU_FREQ_DEFAULT_GOV_PERFORMANCE
    fi

    ### Select tick type
    case "$_tickrate" in
        perodic) scripts/config -d NO_HZ_IDLE -d NO_HZ_FULL -d NO_HZ -d NO_HZ_COMMON -e HZ_PERIODIC;;
        idle) scripts/config -d HZ_PERIODIC -d NO_HZ_FULL -e NO_HZ_IDLE  -e NO_HZ -e NO_HZ_COMMON;;
        full) scripts/config -d HZ_PERIODIC -d NO_HZ_IDLE -d CONTEXT_TRACKING_FORCE -e NO_HZ_FULL_NODEF -e NO_HZ_FULL -e NO_HZ -e NO_HZ_COMMON -e CONTEXT_TRACKING;;
        *) _die "The value '$_tickrate' is invalid. Choose the correct one again.";;
    esac

    echo "Selecting '$_tickrate' tick type..."

    ### Select preempt type

    # We should not set up the PREEMPT for RT kernels
    if [[ "$_cpusched" != "rt" || "$_cpusched" != "rt-bore" ]]; then
        case "$_preempt" in
            full) scripts/config -e PREEMPT_DYNAMIC -e PREEMPT -d PREEMPT_VOLUNTARY -d PREEMPT_LAZY -d PREEMPT_NONE;;
            lazy) scripts/config -e PREEMPT_DYNAMIC -d PREEMPT -d PREEMPT_VOLUNTARY -e PREEMPT_LAZY -d PREEMPT_NONE;;
            voluntary) scripts/config -d PREEMPT_DYNAMIC -d PREEMPT -e PREEMPT_VOLUNTARY -d PREEMPT_LAZY -d PREEMPT_NONE;;
            none) scripts/config -d PREEMPT_DYNAMIC -d PREEMPT -d PREEMPT_VOLUNTARY -d PREEMPT_LAZY -e PREEMPT_NONE;;
            *) _die "The value '$_preempt' is invalid. Choose the correct one again.";;
        esac

        echo "Selecting '$_preempt' preempt type..."
    fi

    ### Enable O3
    if [ "$_cc_harder" = "yes" ]; then
        echo "Enabling KBUILD_CFLAGS -O3..."
        scripts/config -d CC_OPTIMIZE_FOR_PERFORMANCE \
            -e CC_OPTIMIZE_FOR_PERFORMANCE_O3
    fi

    ### CI-only stuff
    if [[ -n "$CI" || -n "$GITHUB_RUN_ID" ]]; then
        echo "Detected build inside CI"
        scripts/config -d CC_OPTIMIZE_FOR_PERFORMANCE \
            -d CC_OPTIMIZE_FOR_PERFORMANCE_O3 \
            -e CONFIG_CC_OPTIMIZE_FOR_SIZE \
            -d SLUB_DEBUG \
            -d PM_DEBUG \
            -d PM_ADVANCED_DEBUG \
            -d PM_SLEEP_DEBUG \
            -d ACPI_DEBUG \
            -d LATENCYTOP \
            -d SCHED_DEBUG \
            -d DEBUG_PREEMPT
    fi

    ### Enable bbr3
    if [ "$_tcp_bbr3" = "yes" ]; then
        echo "Disabling TCP_CONG_CUBIC..."
        scripts/config -m TCP_CONG_CUBIC \
            -d DEFAULT_CUBIC \
            -e TCP_CONG_BBR \
            -e DEFAULT_BBR \
            --set-str DEFAULT_TCP_CONG bbr \
            -m NET_SCH_FQ_CODEL \
            -e NET_SCH_FQ \
            -d CONFIG_DEFAULT_FQ_CODEL \
            -e CONFIG_DEFAULT_FQ
    fi

    ### Select THP
    case "$_hugepage" in
        always) scripts/config -d TRANSPARENT_HUGEPAGE_MADVISE -e TRANSPARENT_HUGEPAGE_ALWAYS;;
        madvise) scripts/config -d TRANSPARENT_HUGEPAGE_ALWAYS -e TRANSPARENT_HUGEPAGE_MADVISE;;
        *) _die "The value '$_hugepage' is invalid. Choose the correct one again.";;
    esac

    echo "Selecting '$_hugepage' TRANSPARENT_HUGEPAGE config..."

    echo "Enable USER_NS_UNPRIVILEGED"
    scripts/config -e USER_NS

    ### Optionally use running kernel's config
    # code originally by nous; http://aur.archlinux.org/packages.php?ID=40191
    if [ "$_use_current" = "yes" ]; then
        if [[ -s /proc/config.gz ]]; then
            echo "Extracting config from /proc/config.gz..."
            # modprobe configs
            zcat /proc/config.gz > ./.config
        else
            warning "Your kernel was not compiled with IKPROC!"
            warning "You cannot read the current config!"
            warning "Aborting!"
            exit
        fi
    fi

    ### Optionally load needed modules for the make localmodconfig
    # See https://aur.archlinux.org/packages/modprobed-db
    if [ "$_localmodcfg" = "yes" ]; then
        if [ -e "$_localmodcfg_path" ]; then
            echo "Running Steven Rostedt's make localmodconfig now"
            make "${BUILD_FLAGS[@]}" LSMOD="${_localmodcfg_path}" localmodconfig
        else
            _die "No modprobed.db data found"
        fi
    fi

    ### Rewrite configuration
    echo "Rewrite configuration..."
    make "${BUILD_FLAGS[@]}" prepare
    yes "" | make "${BUILD_FLAGS[@]}" config >/dev/null
    diff -u ../config .config || :

    ### Prepared version
    make -s kernelrelease > version
    echo "Prepared $pkgbase version $(<version)"

    ### Running make nconfig
    [ "$_makenconfig" = "yes" ] && make "${BUILD_FLAGS[@]}" nconfig

    ### Running make xconfig
    [ "$_makexconfig" = "yes" ] &&  make "${BUILD_FLAGS[@]}" xconfig

    ### Save configuration for later reuse
    echo "Save configuration for later reuse..."
    local basedir="$(dirname "$(readlink "${srcdir}/config")")"
    cat .config > "${basedir}/config-${pkgver}-${pkgrel}${pkgbase#linux}"
}

build() {
    cd "$_srcname"
    make "${BUILD_FLAGS[@]}" -j"$(nproc)" all
    make -C tools/bpf/bpftool vmlinux.h feature-clang-bpf-co-re=1

    local MODULE_FLAGS=(
       KERNEL_UNAME="${_kernuname}"
       IGNORE_PREEMPT_RT_PRESENCE=1
       SYSSRC="${srcdir}/${_srcname}"
       SYSOUT="${srcdir}/${_srcname}"
    )

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

    if _is_lto_kernel; then
        depends+=(clang llvm lld)
    fi

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
        'SKIP'
        'SKIP')
