# $Id$
# Maintainer : Figue <ffigue@gmail.com>
# Contributor : Ionut Biru <ibiru@archlinux.org>
# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=basilisk
pkgver=55.0.0
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org, Extended Support Release"
arch=(x86_64)
license=(MPL GPL LGPL)
url="https://www.mozilla.org/en-US/firefox/organizations/"
depends=(gtk3 gtk2 mozilla-common libxt startup-notification mime-types dbus-glib alsa-lib ffmpeg
         libvpx libevent nss hunspell sqlite ttf-font icu)
makedepends=(unzip zip diffutils python2 yasm mesa imake gconf libpulse inetutils xorg-server-xvfb
             autoconf2.13 rust)
optdepends=('networkmanager: Location detection via available WiFi networks'
            'libnotify: Notification integration'
            'speech-dispatcher: Text-to-Speech')
options=(!emptydirs !makeflags)
com=029f8127bd714fecb607b13a2896fed5198e09ff
source=("basil::git+https://github.com/MoonchildProductions/moebius.git#commit=$com"
        basilisk.desktop basilisk-symbolic.svg
        0001-Bug-54395-remove-hardcoded-flag-lcrmf.patch
        fix-wifi-scanner.diff
        glibc-2.26-fix.diff
        rust-i686.patch
        make_SystemResourceMonitor.stop_more_resilient_to_errors.patch
        nvidia-GLSL-version.patch
https://raw.githubusercontent.com/bn0785ac/firefox-beta/master/firefox-52-disable-data-sharing-infobar.patch
https://raw.githubusercontent.com/bn0785ac/firefox-beta/master/firefox-52-disable-location.services.mozilla.com.patch
https://raw.githubusercontent.com/bn0785ac/firefox-beta/master/firefox-52-disable-telemetry.patch
https://raw.githubusercontent.com/bn0785ac/firefox-beta/master/firefox-install-dir.patch)
sha256sums=('7b27825a7446f98e59296f4a46328c65913ffd50d839e0b4359b71ec7250ca4f'
            'c202e5e18da1eeddd2e1d81cb3436813f11e44585ca7357c4c5f1bddd4bec826'
            'a2474b32b9b2d7e0fb53a4c89715507ad1c194bef77713d798fa39d507def9e9'
            '93c495526c1a1227f76dda5f3a43b433bc7cf217aaf74bd06b8fc187d285f593'
            'd86e41d87363656ee62e12543e2f5181aadcff448e406ef3218e91865ae775cd'
            '9765bca5d63fb5525bbd0520b7ab1d27cabaed697e2fc7791400abc3fa4f13b8'
            'cd7ff441da66a287f8712e60cdc9e216c30355d521051e2eaae28a66d81915e8'
            'f61ea706ce6905f568b9bdafd1b044b58f20737426f0aa5019ddb9b64031a269'
            '7760ebe71f4057cbd2f52b715abaf0d944c14c39e2bb2a5322114ad8451e12d9'
            'd8c5c30589c0e176d260a5814f3cb99e94267b3185ab40ff01bf33a58f375d6a')



prepare() {
  mkdir path
  ln -s /usr/bin/python2 path/python

  cd basil

msg2 'Edgy patches'

patch -Np1 -i ../firefox-52-disable-data-sharing-infobar.patch
patch -Np1 -i ../firefox-52-disable-location.services.mozilla.com.patch
patch -Np1 -i ../firefox-52-disable-telemetry.patch

msg2 'Workin'


  # https://bugzilla.mozilla.org/show_bug.cgi?id=1314968
  patch -Np1 -i ../fix-wifi-scanner.diff

  # https://bugs.archlinux.org/task/54395 // https://bugzilla.mozilla.org/show_bug.cgi?id=1371991
  patch -Np1 -i ../0001-Bug-54395-remove-hardcoded-flag-lcrmf.patch

  # https://bugzilla.mozilla.org/show_bug.cgi?id=1385667
  # https://bugzilla.mozilla.org/show_bug.cgi?id=1394149
  patch -d toolkit/crashreporter/google-breakpad/src/client -Np4 < ../glibc-2.26-fix.diff

  # Build with the rust targets we actually ship




  cat >.mozconfig <<END
ac_add_options --enable-application=browser

ac_add_options --prefix=/usr
ac_add_options --enable-release
ac_add_options --enable-gold
ac_add_options --enable-pie
ac_add_options --enable-optimize="-O2"

# Branding
ac_add_options --enable-official-branding
ac_add_options --enable-update-channel=release
ac_add_options --with-distribution-id=org.archlinux
export MOZILLA_OFFICIAL=1
export MOZ_TELEMETRY_REPORTING=1
export MOZ_ADDON_SIGNING=1
export MOZ_REQUIRE_SIGNING=0


# System libraries
ac_add_options --with-system-nspr
ac_add_options --with-system-nss
ac_add_options --with-system-icu
ac_add_options --with-system-zlib
ac_add_options --with-system-bz2
ac_add_options --enable-system-hunspell
ac_add_options --enable-system-sqlite
ac_add_options --enable-system-ffi
ac_add_options --enable-system-pixman

# Features
ac_add_options --enable-startup-notification
ac_add_options --enable-crashreporter
ac_add_options --enable-alsa
ac_add_options --disable-updater


# Imported from Waterfox
ac_add_options --disable-stylo
ac_add_options --disable-tests


# If you want to have text-to-speech support, comment this line:
ac_add_options --disable-webspeech

# If you want to have geolocation support, comment this line:
ac_add_options --disable-necko-wifi 

# If you have some problems with Skype Web or other web chat, comment this line:
ac_add_options --disable-webrtc

# If you want to have gamepad support, comment this line:
ac_add_options --disable-gamepad


# please put 1.25 times your number of threads

mk_add_options MOZ_MAKE_FLAGS="-j10"


END
}

build() {
  cd basil

  # _FORTIFY_SOURCE causes configure failures
  CPPFLAGS+=" -O2"

#  # Hardening
#  LDFLAGS+=" -Wl,-z,now"

  export PATH="$srcdir/path:$PATH"

  # Do PGO
  #xvfb-run -a -n 95 -s "-extension GLX -screen 0 1280x1024x24" \
  #  MOZ_PGO=1 ./mach build
  ./mach build
  #  ./mach buildsymbols
}

package() {
  cd basil
  DESTDIR="$pkgdir" ./mach install

  for i in 16 22 24 32 48 256; do
    install -Dm644 browser/branding/official/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/basilisk.png"
  done
  install -Dm644 browser/branding/official/content/icon64.png \
    "$pkgdir/usr/share/icons/hicolor/64x64/apps/basilisk.png"
  install -Dm644 browser/branding/official/mozicon128.png \
    "$pkgdir/usr/share/icons/hicolor/128x128/apps/basilisk.png"
  install -Dm644 browser/branding/official/content/about-logo.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/basilisk.png"
  install -Dm644 browser/branding/official/content/about-logo@2x.png \
    "$pkgdir/usr/share/icons/hicolor/384x384/apps/basilisk.png"
  install -Dm644 ../basilisk-symbolic.svg \
    "$pkgdir/usr/share/icons/hicolor/symbolic/apps/basilisk-symbolic.svg"

  install -Dm644 ../basilisk.desktop \
    "$pkgdir/usr/share/applications/basilisk.desktop"

  # Use system-provided dictionaries

  ln -srf "$pkgdir/usr/bin/basilisk" \
    "$pkgdir/usr/lib/basilisk-$pkgver/basilisk-bin"

}