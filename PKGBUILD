# Maintainer: xiota
# Contributor: nekgem2 <nekgem2@firemail.cc>
# Local rebuild for GCC 16 / CMake 4.4
#
# Sidecar files that MUST replace the AUR copies (same names):
#   lokinet.service   — CAP_NET_ADMIN + ExecStart=/etc/loki/lokinet.ini
#   lokinet.tmpfiles  — ONE-WAY link /var/lib/lokinet/lokinet.ini -> /etc/loki/lokinet.ini
#   lokinet.install   — never reverse-link /etc -> /var/lib
#
# Keep AUR: lokinet.conf (unused), lokinet-vpn@.service, lokinet-resume.service,
#           lokinet.sysusers, lokinet.rules

_pkgname=lokinet
pkgname=$_pkgname
pkgver=0.9.14
pkgrel=4
pkgdesc="Anonymous, decentralized and IP based overlay network for the internet"
url="https://github.com/oxen-io/lokinet"
license=('GPL-3.0-or-later')
arch=('x86_64' 'aarch64')
depends=(
  'curl'
  'jemalloc'
  'libsodium'
  'libuv'
  'oxen-mq'
  'systemd-libs'
  'unbound'
  'zeromq'
)
makedepends=(
  'cmake'
  'cppzmq'
  'git'
  'ninja'
  'nlohmann-json'
  'pkgconf'
  'python'
)
conflicts=('lokinet-bin')
provides=('lokinet')
install='lokinet.install'
backup=('etc/loki/lokinet.ini')
_pkgsrc="$_pkgname"
source=(
  "$_pkgsrc"::"git+$url.git#tag=v$pkgver"
  'lokinet.conf'
  'lokinet.service'
  'lokinet-vpn@.service'
  'lokinet-resume.service'
  'lokinet.sysusers'
  'lokinet.tmpfiles'
  'lokinet.rules'
)
sha256sums=('SKIP'
            'ff5e7db4e65463e50978da0185487bd4a7f213f04bdb6256e221089f833c6ab6'
            'cb594dfac267d0759a34a91e74f6d7e2e85232b0e17ef5eb7760b33295f786f0'
            '1c90e7e362bf33d824af70fcf7da509dcc166f9d1f9c90111d25c28905b81857'
            'bcf4bd7b38d2f054e25cc243353d3c9a56d1948b42ad07ee5c0260de06e8dd6c'
            '137cf7eeebc8737d62f3ccfad2398fb1c442a91cb9db7d650429b218dd949a00'
            '627518e45abbb98a1758de57853a5930e97ea2d06d65eac4c5c08e26b239dfe5'
            '6ea4d917ce2e46b2c31af31b8c8c28054c5f977bab5b050c44e2029ab3248713')
prepare() {
  cd "$_pkgsrc"

  git rm -r --ignore-unmatch --quiet \
    external/ghc-filesystem \
    external/nlohmann \
    external/pybind11 \
    test/Catch2 || true

  git submodule update --init --depth=1
  if [[ -d external/oxen-logging ]]; then
    git -C external/oxen-logging submodule update --init --depth=1
  fi

  if [[ -f external/cpr/include/cpr/callback.h ]]; then
    if ! grep -q '#include <cstdint>' external/cpr/include/cpr/callback.h; then
      sed -i '1s/^/#include <cstdint>\n/' external/cpr/include/cpr/callback.h
    fi
  fi
}

build() {
  export CMAKE_POLICY_VERSION_MINIMUM=3.5

  local _cmake_options=(
    -B build
    -S "$_pkgsrc"
    -G Ninja
    -DCMAKE_BUILD_TYPE=Release
    -DCMAKE_INSTALL_PREFIX=/usr
    -DCMAKE_C_FLAGS="$CFLAGS"
    -DCMAKE_CXX_FLAGS="$CXXFLAGS"
    -DCMAKE_POLICY_VERSION_MINIMUM=3.5
    -DCMAKE_WARN_DEPRECATED=OFF
    -Wno-author
    -Wno-deprecated
    -DBUILD_LIBLOKINET=OFF
    -DDOWNLOAD_SODIUM=OFF
    -DFORCE_OXENC_SUBMODULE=ON
    -DFORCE_OXENMQ_SUBMODULE=OFF
    -DGIT_VERSION="v$pkgver"
    -DLOKINET_VERSIONTAG=release
    -DNATIVE_BUILD=OFF
    -DOXENMQ_INSTALL_CPPZMQ=OFF
    -DOXEN_LOGGING_FMT_HEADER_ONLY=ON
    -DOXEN_LOGGING_FORCE_SUBMODULES=ON
    -DOXEN_LOGGING_SPDLOG_HEADER_ONLY=ON
    -DSUBMODULE_CHECK=OFF
    -DUSE_AVX2=OFF
    -DUSE_JEMALLOC=ON
    -DWARNINGS_AS_ERRORS=OFF
    -DWARN_DEPRECATED=OFF
    -DWITH_BOOTSTRAP=ON
    -DWITH_PEERSTATS=OFF
    -DWITH_SETCAP=OFF
    -DWITH_SYSTEMD=ON
    -DWITH_TESTS=OFF
  )

  cmake "${_cmake_options[@]}"
  cmake --build build
}

package() {
  DESTDIR="$pkgdir" cmake --install build

  install -Dm644 lokinet.service        "$pkgdir/usr/lib/systemd/system/lokinet.service"
  install -Dm644 lokinet-vpn@.service   "$pkgdir/usr/lib/systemd/system/lokinet-vpn@.service"
  install -Dm644 lokinet-resume.service "$pkgdir/usr/lib/systemd/system/lokinet-resume.service"
  install -Dm644 lokinet.sysusers       "$pkgdir/usr/lib/sysusers.d/lokinet.conf"
  install -Dm644 lokinet.tmpfiles       "$pkgdir/usr/lib/tmpfiles.d/lokinet.conf"

  install -dm750 "$pkgdir/usr/share/polkit-1/rules.d"
  install -Dm644 lokinet.rules "$pkgdir/usr/share/polkit-1/rules.d/lokinet.rules"

  # REAL config. AUR lokinet.conf is not an INI — never install it here.
  # tmpfiles creates the one-way /var/lib/lokinet/lokinet.ini -> this file.
  install -dm750 "$pkgdir/etc/loki"
  install -dm750 "$pkgdir/var/lib/lokinet"
  cat > "$pkgdir/etc/loki/lokinet.ini" << 'EOF'
[router]
data-dir=/var/lib/lokinet
netid=lokinet
worker-threads=0

[dns]
upstream=1.1.1.1
upstream=9.9.9.9
bind=127.3.2.1:53

[bootstrap]
add-node=/var/lib/lokinet/bootstrap.signed

[api]
enabled=false

[logging]
type=syslog
level=info

[network]
EOF
  chmod 640 "$pkgdir/etc/loki/lokinet.ini"

  if [[ -f "$_pkgsrc/contrib/bootstrap/mainnet.signed" ]]; then
    install -Dm644 "$_pkgsrc/contrib/bootstrap/mainnet.signed" \
      "$pkgdir/var/lib/lokinet/bootstrap.signed"
  fi

  install -Dm644 "$_pkgsrc/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

  rm -rf "$pkgdir/usr/include"
  rm -rf "$pkgdir/usr/lib/cmake"
  rm -rf "$pkgdir/usr/lib/pkgconfig"
  rm -f "$pkgdir/usr/lib"/*.so "$pkgdir/usr/lib"/*.so.* "$pkgdir/usr/lib"/*.a
}
