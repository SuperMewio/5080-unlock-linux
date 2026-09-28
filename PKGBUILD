# Maintainer: SuperMewio
pkgname=omen-wmi-boost-dkms
pkgver=1.1
pkgrel=1
pkgdesc="HP Omen WMI GPU boost driver (DKMS) for RTX 5080 laptops"
arch=(x86_64)
url="https://github.com/SuperMewio/5080-unlock-linux"
license=(MIT)
depends=(dkms)
makedepends=(git)
source=("git+https://github.com/SuperMewio/5080-unlock-linux.git#branch=original-gpu-unlock-only")
sha256sums=('SKIP')

package() {
  local srcdirmod="$srcdir/5080-Unlock_Linux/omen_wmi_boost"
  local installdir="$pkgdir/usr/src/omen-wmi-boost-$pkgver"

  # ---- DKMS source tree ----
  install -dm755 "$installdir"
  install -Dm644 "$srcdirmod/omen_wmi_boost.c" -t "$installdir/"
  install -Dm644 "$srcdirmod/Makefile"        -t "$installdir/"

  # ---- dkms.conf (generated so the version always matches pkgver) ----
  install -Dm644 /dev/stdin "$installdir/dkms.conf" <<EOF
PACKAGE_NAME="omen-wmi-boost"
PACKAGE_VERSION="$pkgver"
BUILT_MODULE_NAME[0]="omen_wmi_boost"
DEST_MODULE_LOCATION[0]="/updates/dkms"
AUTOINSTALL="yes"
EOF

  # ---- Module options: max performance at load ----
  install -Dm644 /dev/stdin "$pkgdir/etc/modprobe.d/omen_wmi_boost.conf" <<EOF
options omen_wmi_boost persist=1 auto_boost=1 thermal_profile=1
EOF

  # ---- Auto-load at boot ----
  install -Dm644 /dev/stdin "$pkgdir/etc/modules-load.d/omen_wmi_boost.conf" <<EOF
omen_wmi_boost
EOF

  # ---- License file (MIT requires preserving the notice) ----
  # If LICENSE sits in the repo root:
  install -Dm644 "$srcdir/5080-Unlock_Linux/LICENSE" -t "$pkgdir/usr/share/licenses/$pkgname/"
  # If it's inside omen_wmi_boost/ instead, use this line and delete the one above:
  # install -Dm644 "$srcdirmod/LICENSE" -t "$pkgdir/usr/share/licenses/$pkgname/"
}
