# Maintainer: David Runge <dvzrv@archlinux.org>
# Maintainer: Sven-Hendrik Haase <svenstaro@archlinux.org>
# Contributor: Lauri Niskanen <ape@ape3000.com>
# Contributor: Sebastian.Salich@gmx.de
# Contributor: Doc Angelo

# NOTE: needs to be built using multilib for mumble-overlay!
pkgbase=mumble
#pkgname=(mumble mumble-server)
pkgname=mumble
pkgver=1.6.870
pkgrel=6
pkgdesc="An Open Source, low-latency, high quality voice chat software"
arch=(x86_64)
url="https://www.mumble.info/"
license=(BSD-3-Clause)
# shared depends
depends=(
  abseil-cpp
  gcc-libs
  glibc
  qt5-base
)
# shared makedepends
makedepends=(
  avahi
  boost
  cmake
  openssl
  protobuf
  python
  qt5-tools
)
# mumble makedepends
makedepends+=(
  alsa-lib
  hicolor-icon-theme
  jack
  lib32-gcc-libs
  libpulse
  libsndfile
  libspeechd
  libx11
  libxi
  mesa
  nlohmann-json
  opus
  poco
  qt5-svg
  rnnoise
  speech-dispatcher
  speexdsp
  xdg-utils
)
# mumble-server makedepends
makedepends+=(
  libcap
  systemd
  zeroc-ice
)
checkdepends=(
  xorg-server-xvfb
)
source=(
#  https://github.com/mumble-voip/mumble/releases/download/v$pkgver/$pkgbase-$pkgver.tar.gz{,.sig}
  https://github.com/mumble-voip/mumble/releases/download/v$pkgver/$pkgbase-$pkgver.tar.gz
  $pkgbase-1.5.517-config_defaults.patch
  0001-fix-tab-completion.patch
  0002-fix-markdown.patch
  0003-add-paste-to-send-dialog.patch
)
sha512sums=(
            'SKIP'
            'c12f6269c5745532031f09fba5b9e3118e6beaf387ae0aaba6ff8380a1452b47f9f0d1cae04472a5763b3da695e03467de152a98bf03c01ae59bd6d553ec7100'
            'SKIP'
            'SKIP'
            'SKIP'
)
# See https://github.com/mumble-voip/mumble-gpg-signatures
validpgpkeys=(
#  'CEB6BB9EC81FA7105F22A017CABE1DB1F21021AE'  # Mumble Automatic Build Infrastructure 2025 <mumble-auto-build-2025@mumble.info>
)

prepare() {
  # add default configuration options
#  patch -Np1 -d $pkgbase-$pkgver -i ../$pkgbase-1.5.517-config_defaults.patch
  patch -p1 -d $pkgbase-$pkgver -i ../0001-fix-tab-completion.patch
  patch -p1 -d $pkgbase-$pkgver -i ../0002-fix-markdown.patch
  patch -p1 -d $pkgbase-$pkgver -i ../0003-add-paste-to-send-dialog.patch
  # Ensure the _mumble-server user is fully locked.
  printf 'u! _mumble-server - "Mumble server user" - -\n' > $pkgbase-$pkgver/auxiliary_files/config_files/mumble-server.sysusers
  # ensure the default server directory is created
  printf "d /var/lib/mumble-server 0750 _mumble-server _mumble-server -\n" >> $pkgbase-$pkgver/auxiliary_files/config_files/mumble-server.tmpfiles.in
}

build() {
  local default_options=(
    -D CMAKE_INSTALL_PREFIX=/usr
    # protobuf 23 requires C++17
    -D CMAKE_CXX_STANDARD=17
    -D CMAKE_C_STANDARD=99
    -D CMAKE_BUILD_TYPE=None
    # upstream requires adding arbitrary build number specifically, as otherwise the version string is wrong:
    # https://github.com/mumble-voip/mumble/issues/5538
    -D BUILD_NUMBER="${pkgver/*./}"
    -D tests=OFF
    -D warnings-as-errors=OFF
    -S $pkgbase-$pkgver
    -W no-dev
  )
  local cmake_options_client=(
    -D update=OFF
    -D server=OFF
    -B build-client
    -D bundled-json=OFF
    -D bundled-rnnoise=OFF
    -D bundled-speex=OFF
    -D rnnoise=ON
  )
  local cmake_options_server=(
    -D MUMBLE_INSTALL_ABS_SYSCONFDIR=/etc/mumble-server
    -D CMAKE_INSTALL_SYSCONFDIR=/etc
    -D use-pkgconf-install-paths=ON
    -D client=OFF
    -B build-server
  )

#  cmake "${default_options[@]}" "${cmake_options_server[@]}"
#  cmake --build build-server --verbose

  cmake "${default_options[@]}" "${cmake_options_client[@]}"
  cmake --build build-client --verbose
}

#check() {
#  xvfb-run ctest --test-dir build-client --output-on-failure
#  ctest --test-dir build-server --output-on-failure
#}

package_mumble() {
  pkgdesc+=" (client)"
  # NOTE: jack, libpulse, and pipewire are dlopen'ed
  depends+=(
    alsa-lib libasound.so
    avahi libdns_sd.so
    hicolor-icon-theme
    jack
    protobuf libprotobuf.so
    libpulse
    libsndfile libsndfile.so
    libspeechd
    libx11
    libxi
    openssl libcrypto.so libssl.so
    opus libopus.so
    poco
    qt5-svg
    rnnoise
    speexdsp libspeexdsp.so
    xdg-utils
  )
  optdepends=(
    'bash: for mumble-overlay'
    'lib32-glibc: for mumble-overlay'
    'espeak-ng: Text-to-speech support'
    'speech-dispatcher: Text-to-speech support'
  )

  DESTDIR="$pkgdir" cmake --install build-client
  install -vDm 644 $pkgbase-$pkgver/LICENSE -t "$pkgdir/usr/share/licenses/$pkgname/"
}

# vim: sw=2:ts=2 et:
