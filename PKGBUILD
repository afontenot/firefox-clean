# Maintainer: afontenot <adam.m.fontenot@gmail.com>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=firefox-clean
_pkgname=firefox
pkgver=67.0.3
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org, with defaults for more privacy"
arch=(x86_64)
license=(MPL GPL LGPL)
url="https://www.mozilla.org/firefox/"
depends=(gtk3 mozilla-common libxt startup-notification mime-types dbus-glib
         ffmpeg nss ttf-font libpulse)
makedepends=(unzip zip diffutils python2-setuptools yasm mesa imake inetutils
             xorg-server-xvfb autoconf2.13 rust mercurial clang llvm jack gtk2
             python nodejs python2-psutil cbindgen nasm)
optdepends=('networkmanager: Location detection via available WiFi networks'
            'libnotify: Notification integration'
            'pulseaudio: Audio support'
            'speech-dispatcher: Text-to-Speech'
            'hunspell-en_US: Spell checking, American English')
options=(!emptydirs !makeflags)
conflicts=('firefox')
provides=("firefox=$pkgver")
_repo=https://hg.mozilla.org/mozilla-unified
source=("hg+$_repo#tag=FIREFOX_${pkgver//./_}_RELEASE"
        0001-bz-1521249.patch
        $_pkgname.desktop firefox-symbolic.svg
	disable-bad-addons.diff disable-newtab-ads.diff add-restart.diff)
sha256sums=('SKIP'
            'd0673786a6a1f1b9f6f66a3a1356afa33f1f18f59dabd92bd193c88c52a1d04c'
            '4a783dca1f88e003c72f32d22719a0915f3fa576adbc492240e7cc250246ce10'
            '9a1a572dc88014882d54ba2d3079a1cf5b28fa03c5976ed2cb763c93dabbd797'
            'f68dd4cf6e9ae902daaf0c218b7ac3d0cf04c0cca68cc2c4d2f33bf8a5be2b81'
            '54185ac9b1a5e7bf39315670751e29640630e229ed5f8591dbb239a338612de4'
            '5408e978b2873cac06a6c9e9f6b6bcecab3628882a41ea996e9ea5dfe2857634')

prepare() {
  mkdir -p mozbuild
  cd mozilla-unified

  # https://bugzilla.mozilla.org/show_bug.cgi?id=1521249
  patch -Np1 -i ../0001-bz-1521249.patch
  
  # Disable anti-features
  patch -Np1 -i ../disable-bad-addons.diff
  
  # Disable junk on the new tab page
  patch -Np1 -i ../disable-newtab-ads.diff

  # Add restart to file menu
  patch -Np1 -i ../add-restart.diff

  # I recommend we take off and nuke the site from orbit.
  # It's the only way to be sure.
  rm -r browser/components/pocket

  cat >.mozconfig <<END
ac_add_options --enable-application=browser

ac_add_options --prefix=/usr
ac_add_options --enable-release
ac_add_options --enable-hardening
ac_add_options --enable-optimize
ac_add_options --enable-rust-simd
ac_add_options --enable-lto
export MOZ_PGO=1
export CC=clang
export CXX=clang++
export AR=llvm-ar
export NM=llvm-nm
export RANLIB=llvm-ranlib

# Branding
ac_add_options --enable-official-branding
ac_add_options --enable-update-channel=release
ac_add_options --with-distribution-id=org.archlinux
export MOZILLA_OFFICIAL=1
export MOZ_APP_REMOTINGNAME=${pkgname//-/}
export MOZ_TELEMETRY_REPORTING=0
export MOZ_REQUIRE_SIGNING=0

# System libraries
ac_add_options --with-system-nspr
ac_add_options --with-system-nss

# Features
ac_add_options --enable-alsa
ac_add_options --enable-jack
ac_add_options --enable-startup-notification
ac_add_options --enable-crashreporter
ac_add_options --disable-gconf
ac_add_options --disable-updater
END
}

build() {
  cd mozilla-unified

  export MOZ_SOURCE_REPO="$_repo"
  export MOZ_NOSPAM=1
  export MOZBUILD_STATE_PATH="$srcdir/mozbuild"

  # LTO needs more open files
  ulimit -n 4096

  xvfb-run -a -n 97 -s "-screen 0 1600x1200x24" ./mach build
  ./mach buildsymbols
}

package() {
  cd mozilla-unified
  DESTDIR="$pkgdir" ./mach install

  _vendorjs="$pkgdir/usr/lib/$_pkgname/browser/defaults/preferences/vendor.js"
  install -Dm644 /dev/stdin "$_vendorjs" <<END
// Use LANG environment variable to choose locale
pref("intl.locale.requested", "");

// Use system-provided dictionaries
pref("spellchecker.dictionary_path", "/usr/share/hunspell");

// Disable default browser checking.
pref("browser.shell.checkDefaultBrowser", false);

// Don't disable our bundled extensions in the application directory
pref("extensions.autoDisableScopes", 11);
pref("extensions.shownSelectionUI", true);

// DuckDuckGo instead of Yahoo
pref("browser.search.defaultenginename", "DuckDuckGo");
pref("browser.search.defaultenginename.US", "DuckDuckGo");
pref("browser.search.order.1", "DuckDuckGo");
pref("browser.search.order.2", "Google");
pref("browser.search.order.US.1", "DuckDuckGo");

// Disable Google's safe browsing by default
// Note: Safe Browsing has blocked entire legitimate sites
pref("browser.safebrowsing.enabled", false);
pref("browser.safebrowsing.malware.enabled", false);
pref("browser.safebrowsing.phishing.enabled", false);
pref("browser.safebrowsing.downloads.enabled", false);

// Disable suggested sites
pref("browser.newtabpage.enhanced", false);
pref("browser.newtabpage.activity-stream.feeds.snippets", false);
pref("browser.newtabpage.activity-stream.feeds.section.highlights", false);
pref("browser.discovery.enabled", false);
pref("browser.newtabpage.activity-stream.feeds.discoverystreamfeed", false);
pref("browser.discovery.containers.enabled", false);

// Don't assume user wants to search when typing URLs
pref("browser.urlbar.suggest.searches", false);

// Mozilla has proven they can't be trusted with experiments
pref("app.normandy.enabled", false);
pref("app.shield.optoutstudies.enabled", false);
pref("browser.onboarding.shieldstudy.enabled", false);

// Mozilla now enables telemetry not covered by other policies
// https://blog.mozilla.org/data/2018/08/20/effectively-measuring-search-in-firefox/
pref("toolkit.telemetry.enabled", false)
pref("toolkit.telemetry.coverage.opt-out", true);

// Disable uploading screenshots
pref("extensions.screenshots.upload-disabled", true);

// Secure URLbar
pref("browser.urlbar.oneOffSearches", false);
pref("browser.urlbar.searchSuggestionsChoice", false);
pref("browser.search.widget.inNavBar", true);

// Disable requesting notifications access by default
pref("permissions.default.desktop-notification", 2);

// Security hardening
pref("security.tls.version.min", 3);
pref("security.ssl3.rsa_aes_128_sha", false);
pref("security.ssl3.rsa_aes_256_sha", false);
pref("security.ssl3.rsa_des_ede3_sha", false);
pref("security.ssl.require_safe_negotiation", true);
pref("network.IDN_show_punycode", true);
END

  _policies="$pkgdir/usr/lib/$_pkgname/distribution/policies.json"
  install -Dm644 /dev/stdin "$_policies" <<END
{
  "policies": {
    "DisablePocket": true
  }
}
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

# vim:set sw=2 et:
