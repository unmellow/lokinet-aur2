# Maintainer: Unmellow <amazingminecrafter2015 at gmail dot com>
# Contributor: xiota
# Contributor: nekgem2 <nekgem2 at firemail dot cc>
#
# Local overlay of AUR/lokinet. Do NOT push this pkgbase to aur.archlinux.org:
# lokinet 0.9.14 is already maintained by xiota. Send GCC 16 / CMake 4.4
# fixes as comments on https://aur.archlinux.org/packages/lokinet
#
# Differences from AUR/lokinet:
#   lokinet.service   — CAP_NET_ADMIN + ExecStart=/etc/loki/lokinet.ini
#   lokinet.tmpfiles  — one-way link /var/lib/lokinet/lokinet.ini -> /etc/loki/lokinet.ini
#   lokinet.install   — never reverse-link /etc -> /var/lib
#   CMAKE_POLICY_VERSION_MINIMUM=3.5 for CMake 4
#   cpr-cstdint.patch for GCC 16

_pkgname=lokinet
pkgname=$_pkgname
pkgver=0.9.14
pkgrel=5
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
  'python'
)
conflicts=('lokinet-bin')
install='lokinet.install'
backup=('etc/loki/lokinet.ini')
_commit='90f2fde60009691cfe58eb5b8d90fc4a71c18347'
_pkgsrc="$_pkgname"
source=(
  "$_pkgsrc"::"git+$url.git#commit=$_commit"
  'lokinet.ini'
  'lokinet.service'
  'lokinet-vpn@.service'
  'lokinet-resume.service'
  'lokinet.sysusers'
  'lokinet.tmpfiles'
  'lokinet.rules'
  'cpr-cstdint.patch'
)
sha256sums=('SKIP'
            '5a64bccc13152cb78b4243f6a98ddf3437c881e2875f35353dba6ef8e93611cd'
            'cb594dfac267d0759a34a91e74f6d7e2e85232b0e17ef5eb7760b33295f786f0'
            '1c90e7e362bf33d824af70fcf7da509dcc166f9d1f9c90111d25c28905b81857'
            'bcf4bd7b38d2f054e25cc243353d3c9a56d1948b42ad07ee5c0260de06e8dd6c'
            '137cf7eeebc8737d62f3ccfad2398fb1c442a91cb9db7d650429b218dd949a00'
            '8026813dc5d420a2d9320b23d7afc46daebde1f728c17d4b5a9c087c6e850838'
            '6ea4d917ce2e46b2c31af31b8c8c28054c5f977bab5b050c44e2029ab3248713'
            '6e14400832f2691a37e56cbd045bf5e98adb4f6d5c96dff782847479c8b795e8')

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

  local _cpr_cb=external/cpr/include/cpr/callback.h
  if [[ -f $_cpr_cb ]] && ! grep -q '#include <cstdint>' "$_cpr_cb"; then
    patch -Np1 -i "$srcdir/cpr-cstdint.patch"
  fi
}

build() {
  export CMAKE_POLICY_VERSION_MINIMUM=3.5

  local _cmake_options=(
    -B build
    -S "$_pkgsrc"
    -G Ninja
    -DCMAKE_BUILD_TYPE=None
    -DCMAKE_INSTALL_PREFIX=/usr
    -DCMAKE_C_FLAGS="$CFLAGS"
    -DCMAKE_CXX_FLAGS="$CXXFLAGS"
    -DCMAKE_POLICY_VERSION_MINIMUM=3.5
    -DCMAKE_WARN_DEPRECATED=OFF
    -Wno-dev
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

  install -dm750 "$pkgdir/etc/loki"
  install -dm750 "$pkgdir/var/lib/lokinet"
  install -Dm640 lokinet.ini "$pkgdir/etc/loki/lokinet.ini"

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
