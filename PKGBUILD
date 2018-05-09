# $Id$
# Maintainer: afontenot <adam.m.fontenot@gmail.com>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=firefox-clean
_pkgname=firefox
pkgver=60.0
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org, with defaults for more privacy"
arch=(x86_64)
license=(MPL GPL LGPL)
url="https://www.mozilla.org/firefox/"
depends=(gtk3 mozilla-common libxt startup-notification mime-types dbus-glib ffmpeg
         nss hunspell sqlite ttf-font libpulse libvpx icu)
makedepends=(unzip zip diffutils python2 yasm mesa imake gconf inetutils xorg-server-xvfb
             autoconf2.13 rust mercurial clang llvm jack gtk2)
optdepends=('networkmanager: Location detection via available WiFi networks'
            'libnotify: Notification integration'
            'pulseaudio: Audio support'
            'speech-dispatcher: Text-to-Speech')
options=(!emptydirs !makeflags !strip)
conflicts=('firefox')
provides=("firefox=$pkgver")
_repo=https://hg.mozilla.org/mozilla-unified
source=("hg+$_repo#tag=FIREFOX_${pkgver//./_}_RELEASE"
        complete-csd-window-offset-mozilla-1457691.patch.xz
        0001-Bug-1435212-Add-support-for-FFmpeg-4.0.-r-bryce.patch.xz
        $_pkgname.desktop firefox-symbolic.svg no-crmf.diff
	disable-pocket.diff disable-newtab-ads.diff add-restart.diff)
sha256sums=('SKIP'
            'a3fb3c3b6fb775c99afdbad507848b77c5e4bbaac2e8ceeb1bfb47699c4b6268'
            '8422030440032535d918844263fbd92d39bff207acb5fff55ed0afee38bcf582'
            '677e1bde4c6b3cff114345c211805c7c43085038ca0505718a11e96432e9811a'
            '9a1a572dc88014882d54ba2d3079a1cf5b28fa03c5976ed2cb763c93dabbd797'
            '02000d185e647aa20ca336e595b4004bb29cdae9d8f317f90078bdcc7a36e873'
            '9b3f96dde8a5d07cd7fce209a95d7d9b7c3e0dfdc760f97e770af00da3cfcc4e'
            '8addb2cd51c9a6dbf75f4cc587f9dfffc280e2f8d25e25e1b413f4284c368712'
            '3089db5b7cac7702e29ee60109805081797d3a38fcb9907b71211bcb02f1e7c0')

# Google API keys (see http://www.chromium.org/developers/how-tos/api-keys)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact foutrelis@archlinux.org for
# more information.
_google_api_key=AIzaSyDwr302FpOSkGRpLlUpPThNTDPbXcIn_FM

# Mozilla API keys (see https://location.services.mozilla.com/api)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact heftig@archlinux.org for
# more information.
_mozilla_api_key=16674381-f021-49de-8622-3021c5942aff

prepare() {
  if [ ! -d "path" ]; then
    mkdir path
    ln -s /usr/bin/python2 path/python
  fi

  cd mozilla-unified

  # https://bugzilla.mozilla.org/show_bug.cgi?id=1283299#c158
  patch -Np1 -i ../complete-csd-window-offset-mozilla-1457691.patch

  # https://bugzilla.mozilla.org/show_bug.cgi?id=1435212
  patch -Np1 -i ../0001-Bug-1435212-Add-support-for-FFmpeg-4.0.-r-bryce.patch

  # https://bugzilla.mozilla.org/show_bug.cgi?id=1371991
  patch -Np1 -i ../no-crmf.diff

  # Disable anti-features
  patch -Np1 -i ../disable-pocket.diff
  patch -Np1 -i ../disable-newtab-ads.diff

  # Add restart to file menu
  patch -Np1 -i ../add-restart.diff

  echo -n "$_google_api_key" >google-api-key
  echo -n "$_mozilla_api_key" >mozilla-api-key

  cat >.mozconfig <<END
ac_add_options --enable-application=browser

ac_add_options --prefix=/usr
ac_add_options --enable-release
ac_add_options --enable-gold
ac_add_options --enable-pie
ac_add_options --enable-optimize="-O2"
ac_add_options --enable-rust-simd

# Branding
ac_add_options --enable-official-branding
ac_add_options --enable-update-channel=release
ac_add_options --with-distribution-id=org.archlinux
export MOZILLA_OFFICIAL=1
export MOZ_TELEMETRY_REPORTING=1
export MOZ_ADDON_SIGNING=1
export MOZ_REQUIRE_SIGNING=0

# Keys
ac_add_options --with-google-api-keyfile=${PWD@Q}/google-api-key
ac_add_options --with-mozilla-api-keyfile=${PWD@Q}/mozilla-api-key

# System libraries
ac_add_options --with-system-zlib
ac_add_options --with-system-bz2
ac_add_options --with-system-icu
ac_add_options --with-system-jpeg
ac_add_options --with-system-libvpx
ac_add_options --with-system-nspr
ac_add_options --with-system-nss
ac_add_options --enable-system-hunspell
ac_add_options --enable-system-sqlite
ac_add_options --enable-system-ffi

# Features
ac_add_options --enable-alsa
ac_add_options --enable-jack
ac_add_options --enable-startup-notification
ac_add_options --enable-crashreporter
ac_add_options --disable-updater
ac_add_options --disable-stylo
END
}

build() {
  cd mozilla-unified

  # _FORTIFY_SOURCE causes configure failures

  # CPPFLAGS+=" -O2"

  export PATH="$srcdir/path:$PATH"
  export MOZ_SOURCE_REPO="$_repo"

  # Do PGO
  #xvfb-run -a -n 95 -s "-extension GLX -screen 0 1280x1024x24" \
  #  MOZ_PGO=1 ./mach build
  ./mach build
  ./mach buildsymbols
}

package() {
  cd mozilla-unified
  DESTDIR="$pkgdir" ./mach install

  _vendorjs="$pkgdir/usr/lib/$_pkgname/browser/defaults/preferences/vendor.js"
  install -Dm644 /dev/stdin "$_vendorjs" <<END
// Use LANG environment variable to choose locale
pref("intl.locale.requested", "");

// Disable default browser checking.
pref("browser.shell.checkDefaultBrowser", false);

// Don't disable our bundled extensions in the application directory
pref("extensions.autoDisableScopes", 11);
pref("extensions.shownSelectionUI", true);

// Opt all of us into e10s, instead of just 50%
pref("browser.tabs.remote.autostart", true);

// DuckDuckGo instead of Yahoo
pref("browser.search.defaultenginename", "DuckDuckGo")
pref("browser.search.defaultenginename.US", "DuckDuckGo")
pref("browser.search.order.1", "DuckDuckGo")
pref("browser.search.order.2", "Google")
pref("browser.search.order.US.1", "DuckDuckGo")

// Disable Google's safe browsing by default
pref("browser.safebrowsing.malware.enabled", false)
pref("browser.safebrowsing.phishing.enabled", false)
pref("browser.safebrowsing.downloads.enabled", false)

// Disable suggested sites
pref("browser.newtabpage.enhanced", false)
pref("browser.newtabpage.activity-stream.feeds.snippets", false)
pref("browser.newtabpage.activity-stream.feeds.section.highlights", false)

// Mozilla has proven they can't be trusted with experiments
pref("app.shield.optoutstudies.enabled", false)
pref("browser.onboarding.shieldstudy.enabled", false)
END

  _distini="$pkgdir/usr/lib/$_pkgname/distribution/distribution.ini"
  install -Dm644 /dev/stdin "$_distini" <<END
[Global]
id=archlinux
version=1.0
about=Mozilla Firefox for Arch Linux

[Preferences]
app.distributor=archlinux
app.distributor.channel=$_pkgname
app.partner.archlinux=archlinux
END

  for i in 16 22 24 32 48 64 128 256; do
    install -Dm644 browser/branding/official/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$_pkgname.png"
  done
  install -Dm644 browser/branding/official/content/about-logo.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/$_pkgname.png"
  install -Dm644 browser/branding/official/content/about-logo@2x.png \
    "$pkgdir/usr/share/icons/hicolor/384x384/apps/$_pkgname.png"
  install -Dm644 ../firefox-symbolic.svg \
    "$pkgdir/usr/share/icons/hicolor/symbolic/apps/$_pkgname-symbolic.svg"

  install -Dm644 ../$_pkgname.desktop \
    "$pkgdir/usr/share/applications/$_pkgname.desktop"

  # Use system-provided dictionaries
  rm -r "$pkgdir/usr/lib/$_pkgname/dictionaries"
  ln -Ts /usr/share/hunspell "$pkgdir/usr/lib/$_pkgname/dictionaries"
  ln -Ts /usr/share/hyphen "$pkgdir/usr/lib/$_pkgname/hyphenation"

  # Install a wrapper to avoid confusion about binary path
  install -Dm755 /dev/stdin "$pkgdir/usr/bin/$_pkgname" <<END
#!/bin/sh
exec /usr/lib/$_pkgname/firefox "\$@"
END

  # Replace duplicate binary with wrapper
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -srf "$pkgdir/usr/bin/$_pkgname" \
    "$pkgdir/usr/lib/$_pkgname/firefox-bin"
}
