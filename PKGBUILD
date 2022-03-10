# Maintainer: afontenot <adam.m.fontenot@gmail.com>
# Contributor: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=firefox-clean
_pkgname=firefox
pkgver=98.0
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org, with defaults for more privacy"
arch=(x86_64)
license=(MPL GPL LGPL)
url="https://www.mozilla.org/firefox/"
depends=(gtk3 libxt mime-types dbus-glib ffmpeg4.4 nss ttf-font libpulse)
makedepends=(unzip zip diffutils yasm mesa imake inetutils xorg-server-xvfb
             autoconf2.13 rust clang llvm jack nodejs cbindgen nasm
             python-setuptools python-psutil python-zstandard lld dump_syms
             wasi-compiler-rt wasi-libc wasi-libc++ wasi-libc++abi)
optdepends=('networkmanager: Location detection via available WiFi networks'
            'libnotify: Notification integration'
            'pulseaudio: Audio support'
            'speech-dispatcher: Text-to-Speech'
            'hunspell-en_US: Spell checking, American English'
            'xdg-desktop-portal: Screensharing with Wayland')
options=(!emptydirs !makeflags !strip !lto !debug)
source=(https://archive.mozilla.org/pub/firefox/releases/$pkgver/source/firefox-$pkgver.source.tar.xz{,.asc}
        $_pkgname.desktop
        identity-icons-brand.svg disable-pocket-addon.diff
        disable-discoverystream.diff add-restart.diff
        allow-removing-menu-button.diff disable-topsite-sponsors.diff)
sha256sums=('fd0a4c11d007d9045706667eb0f99f9b7422945188424cb937bfef530cb6f4dd'
            'SKIP'
            '298eae9de76ec53182f38d5c549d0379569916eebf62149f9d7f4a7edef36abf'
            'a9b8b4a0a1f4a7b4af77d5fc70c2686d624038909263c795ecc81e0aec7711e9'
            '028caedaf0c66a401c30b34ad267daeb4288482d6c59bc2926567e2abe8ebe9b'
            'fce0b197a2e120fd94329d02ca23b0f74eb3befca61507da94a72a6baf9679dd'
            'c18fb7e9f17c0eac8487c0dffa3286b0ab56e66d18ecc8a8f3376e687f2d48fc'
            'f53cac8cb4885758a446a7c9ed9d951a524524df5147594b50469fc1749368cc'
            'ed4f34b0ed1d37713d77fe078a696b58db47be387ecb043652f31447090046f9')
conflicts=('firefox')
provides=("firefox=$pkgver")
validpgpkeys=('14F26682D0916CDD81E37B6D61B7B526D98F0353') # Mozilla Software Releases <release@mozilla.com>

prepare() {
  mkdir -p mozbuild
  cd firefox-$pkgver

  # Disable anti-features
  patch -Np1 -i ../disable-pocket-addon.diff
  
  # Disable junk on the new tab page
  patch -Np1 -i ../disable-discoverystream.diff

  # Disable topsite sponsored entries
  patch -Np1 -i ../disable-topsite-sponsors.diff

  # Add restart to file menu
  patch -Np1 -i ../add-restart.diff

  # Allow user to remove menu button
  # Work in progress, not finished
  # patch -Np1 -i ../allow-removing-menu-button.diff

  # I recommend we take off and nuke the site from orbit.
  # It's the only way to be sure.
  rm -r browser/components/pocket

  cat >../mozconfig <<END
ac_add_options --enable-application=browser
mk_add_options MOZ_OBJDIR=${PWD@Q}/obj

ac_add_options --prefix=/usr
ac_add_options --enable-release
ac_add_options --enable-hardening
ac_add_options --enable-optimize
ac_add_options --enable-rust-simd
ac_add_options --enable-linker=lld
ac_add_options --disable-elf-hack
ac_add_options --disable-bootstrap
ac_add_options --with-wasi-sysroot=/usr/share/wasi-sysroot

# Branding
ac_add_options --enable-official-branding
ac_add_options --enable-update-channel=release
ac_add_options --with-distribution-id=org.archlinux
ac_add_options --with-unsigned-addon-scopes=app,system
ac_add_options --allow-addon-sideload
export MOZILLA_OFFICIAL=1
export MOZ_APP_REMOTINGNAME=${_pkgname//-/}
export MOZ_REQUIRE_SIGNING=""

# System libraries
ac_add_options --with-system-nspr
ac_add_options --with-system-nss

# Features
ac_add_options --enable-alsa
ac_add_options --enable-jack
ac_add_options --enable-crashreporter
ac_add_options --disable-updater
ac_add_options --disable-tests
END
}

build() {
  cd firefox-$pkgver

  export MOZ_NOSPAM=1
  export MOZBUILD_STATE_PATH="$srcdir/mozbuild"
  export MOZ_ENABLE_FULL_SYMBOLS=1
  export MACH_USE_SYSTEM_PYTHON=1

  # LTO needs more open files
  ulimit -n 4096

  # Do 3-tier PGO
  echo "Building instrumented browser..."
  cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-profile-generate=cross
END
  ./mach build

  echo "Profiling instrumented browser..."
  ./mach package
  LLVM_PROFDATA=llvm-profdata \
    JARLOG_FILE="$PWD/jarlog" \
    xvfb-run -s "-screen 0 1920x1080x24 -nolisten local" \
    ./mach python build/pgo/profileserver.py

  stat -c "Profile data found (%s bytes)" merged.profdata
  test -s merged.profdata

  stat -c "Jar log found (%s bytes)" jarlog
  test -s jarlog

  echo "Removing instrumented browser..."
  ./mach clobber

  echo "Building optimized browser..."
  cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-lto=cross
ac_add_options --enable-profile-use=cross
ac_add_options --with-pgo-profile-path=${PWD@Q}/merged.profdata
ac_add_options --with-pgo-jarlog=${PWD@Q}/jarlog
END
  ./mach build

  echo "Building symbol archive..."
  ./mach buildsymbols
}

package() {
  cd firefox-$pkgver
  DESTDIR="$pkgdir" ./mach install

  local vendorjs="$pkgdir/usr/lib/$_pkgname/browser/defaults/preferences/vendor.js"
  install -Dvm644 /dev/stdin "$vendorjs" <<END
// Use LANG environment variable to choose locale
pref("intl.locale.requested", "");

// Use system-provided dictionaries
pref("spellchecker.dictionary_path", "/usr/share/hunspell");

// Disable default browser checking.
pref("browser.shell.checkDefaultBrowser", false);

// Don't disable extensions in the application directory
pref("extensions.autoDisableScopes", 11);

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
pref("toolkit.telemetry.enabled", false);
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
pref("security.certerrors.mitm.auto_enable_enterprise_roots", false);
END

  local distini="$pkgdir/usr/lib/$pkgname/distribution/distribution.ini"
  install -Dvm644 /dev/stdin "$distini" <<END
[Global]
id=archlinux
version=1.0
about=Mozilla Firefox for Arch Linux

[Preferences]
app.distributor=archlinux
app.distributor.channel=$pkgname
app.partner.archlinux=archlinux
END

  local i theme=official
  for i in 16 22 24 32 48 64 128 256; do
    install -Dvm644 browser/branding/$theme/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$_pkgname.png"
  done
  install -Dvm644 browser/branding/$theme/content/about-logo.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/$_pkgname.png"
  install -Dvm644 browser/branding/$theme/content/about-logo@2x.png \
    "$pkgdir/usr/share/icons/hicolor/384x384/apps/$_pkgname.png"
  install -Dvm644 browser/branding/$theme/content/about-logo.svg \
    "$pkgdir/usr/share/icons/hicolor/scalable/apps/$_pkgname.svg"
  install -Dvm644 ../identity-icons-brand.svg \
    "$pkgdir/usr/share/icons/hicolor/symbolic/apps/$_pkgname-symbolic.svg"

  install -Dvm644 ../$_pkgname.desktop \
    "$pkgdir/usr/share/applications/$_pkgname.desktop"

  # Install a wrapper to avoid confusion about binary path
  install -Dvm755 /dev/stdin "$pkgdir/usr/bin/$_pkgname" <<END
#!/bin/sh
exec /usr/lib/$_pkgname/firefox "\$@"
END

  # Replace duplicate binary with wrapper
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -srfv "$pkgdir/usr/bin/$_pkgname" "$pkgdir/usr/lib/$_pkgname/firefox-bin"

  # Use system certificates
  local nssckbi="$pkgdir/usr/lib/$_pkgname/libnssckbi.so"
  if [[ -e $nssckbi ]]; then
    ln -srfv "$pkgdir/usr/lib/libnssckbi.so" "$nssckbi"
  fi
}

# vim:set sw=2 et:
