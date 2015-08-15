# Maintainer: Limao Luo <luolimao+AUR@gmail.com>
# Contributor: piojo <aur@zwell.net>
# Contributor: MiakoMiyamura <miyamiyamura@gmail.com>
# Contributor: Frederic Bezies <fredbezies@gmail.com>

pkgname=chromium-browser-svn
pkgver=212598
pkgrel=1
pkgdesc="An open-source browser project that aims to build a safer, faster, and more stable way for all users to experience the web"
arch=(i686 x86_64)
url=http://www.chromium.org
license=(BSD)
depends=(alsa-lib cairo desktop-file-utils gconf gtk2 hicolor-icon-theme lib{event,xslt,xss} nspr)
makedepends=(gperf libgnome-keyring mesa python2 rsync subversion ttf-ms-fonts yasm)
provides=(${pkgname%-*})
conflicts=(${pkgname%-*})
options=(!makeflags)
install=chromium.install
source=($pkgname::svn+http://src.chromium.org/svn/trunk/src
    svn+http://src.chromium.org/svn/trunk/tools/depot_tools
    chromium.desktop
    $pkgname.sh)
sha256sums=('SKIP'
    'SKIP'
    'e2cf2c8bd2c4930a6a9f37e5a6e87a031199d1585cba475da6c1f04a8985ef35'
    'd787d965e09c6a12dbcd5ba3053bc40098ea9930abdaa121d1a6175988cf0afe')
sha512sums=('SKIP'
    'SKIP'
    '816ae2cefe8f1ae6334bb3d2f2284ca8d338d6aba969cc4b8e943f8820140a09201a9ca71f53b3966f1bc166116f519727b880b3913341767acba82843d28f68'
    'e877a1a546251b5e34d46c5023be7e4569bf1e3b7a95045d78a7fe014c58ef5d60e78a20e2cd0b94618cdcbcc3671c782a9d446c65cd93b9705960b4a732664d')

pkgver() {
    svnversion "$SRCDEST"/$pkgname/
}

prepare() {
    [[ $CHROMIUM_WATERFALL_REMINDER == "" ]] || read -p "Remember to check http://build.chromium.org/buildbot/waterfall/console"

    _pyregex='s:^#!/usr/bin/(env )?python$:&2:g'
    find depot_tools/ -type f -exec sed -ri "$_pyregex" '{}' \;

    cd $pkgname/
    ### Write out correct revision number in 'About Chromium'
    find -type f -execdir sed -ri -e "$_pyregex" -e 's:python :python2 :g' '{}' \;
    echo $pkgver > build/LASTCHANGE.in
}

build() {
    ### http://code.google.com/p/chromium/issues/detail?id=41887
    ### http://code.google.com/p/chromium/issues/detail?id=94518
    export CXXFLAGS+=" -fno-ipa-cp"
    export GYP_GENERATORS='make'
    export GYP_DEFINES="
     gcc_version=46
     no_strict_aliasing=1
     linux_sandbox_path=/usr/lib/chromium/chromium-sandbox
     linux_strip_binary=1
     proprietary_codecs=1
     remove_webcore_debug_symbols=1
     use_system_bzip2=1
     use_system_ffmpeg=0
     use_system_icu=0
     use_system_libevent=1
     use_system_libjpeg=1
     use_system_libpng=1
     use_system_libxml=0
     use_system_ssl=0
     use_system_yasm=1
     use_system_zlib=1
     use_gconf=0
     werror=
     disable_nacl=1
    "

    [[ $_shared_build = "true" ]] && GYP_DEFINES+=" library=shared_library"
    ### 64-bit instructions from
    ### http://code.google.com/p/chromium/wiki/LinuxBuildInstructions
    [[ $CARCH = 'x86_64' ]] && GYP_DEFINES+=" target_arch=x64"

    msg "Running gyp in builddir using these settings: "
    echo $GYP_DEFINES
    cd $pkgname/build/
    ./gyp_chromium -f make all.gyp --depth=..

    cd ../
    msg "Now starting build..."
    make BUILDTYPE=Release V=1 chrome chrome_sandbox
}

package() {
    desktop-file-install chromium.desktop --dir "$pkgdir"/usr/share/applications/
    install -Dm755 $pkgname.sh "$pkgdir"/usr/bin/chromium

    cd depot_tools/src/
    install -Dm755 out/Release/chrome "$pkgdir"/usr/lib/chromium/chromium
    install -Dm4555 -o root -g root out/Release/chrome_sandbox "$pkgdir"/usr/lib/chromium/chromium-sandbox
    install -Dm644 out/Release/chrome.pak "$pkgdir"/usr/lib/chromium/chrome.pak
    install -Dm644 out/Release/resources.pak "$pkgdir"/usr/lib/chromium/resources.pak
    cp -a out/Release/locales "$pkgdir"/usr/lib/chromium/
    cp -a out/Release/resources  "$pkgdir"/usr/lib/chromium/
    find "$pkgdir"/usr/lib/chromium/ -name '*.d' -type f -delete
    install -Dm755 out/Release/libffmpegsumo.so "$pkgdir"/usr/lib/chromium/libffmpegsumo.so
    install -Dm644 out/Release/chrome.1 "$pkgdir"/usr/share/man/man1/chromium.1
    install -Dm644 LICENSE "$pkgdir"/usr/share/licenses/chromium/LICENSE

    for size in 16 22 24 32 48 128 256; do
        install -Dm644 chrome/app/theme/chromium/product_logo_$size.png "$pkgdir"/usr/share/icons/hicolor/${size}x$size/apps/chromium.png
    done
}
