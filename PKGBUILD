#
# Tsekhovoy Eugene <8552246@gmail.com>
#

pkgname=wine-wot
pkgver=1.5.14
pkgrel=1

_pkgname=wine
_pkgbasever=${pkgver/rc/-rc}

source=(
    http://prdownloads.sourceforge.net/$_pkgname/$_pkgname-$_pkgbasever.tar.bz2
    0002-disable_dynamic_vertex_buffers.patch
)
md5sums=(
    'f84c54bd7422328e96b6cf14ee6e163c'
    '73c5c245a31ac7a21ee7f5100bf9821b'
)

pkgdesc="Wine with patches for World Of Tanks"
url="http://www.winehq.com"
arch=(i686 x86_64)
license=(LGPL)
install=wine.install

depends=(
  fontconfig      lib32-fontconfig
  mesa            lib32-mesa 
  libxcursor      lib32-libxcursor
  libxrandr       lib32-libxrandr
  libxdamage      lib32-libxdamage
  libxi           lib32-libxi
  gettext         lib32-gettext
  desktop-file-utils
)

makedepends=(autoconf ncurses bison perl fontforge flex prelink
  'gcc>=4.5.0-2'  'gcc-multilib>=4.5.0-2'
  giflib          lib32-giflib
  libpng          lib32-libpng
  gnutls          lib32-gnutls
  libxinerama     lib32-libxinerama
  libxcomposite   lib32-libxcomposite
  libxmu          lib32-libxmu
  libxxf86vm      lib32-libxxf86vm
  libxml2         lib32-libxml2
  libldap         lib32-libldap
  lcms            lib32-lcms
  mpg123          lib32-mpg123
  openal          lib32-openal
  v4l-utils       lib32-v4l-utils
  alsa-lib        lib32-alsa-lib
  oss
  samba
)
  
optdepends=(
  giflib          lib32-giflib
  libpng          lib32-libpng
  libldap         lib32-libldap
  gnutls          lib32-gnutls
  lcms            lib32-lcms
  libxml2         lib32-libxml2
  mpg123          lib32-mpg123
  openal          lib32-openal
  v4l-utils       lib32-v4l-utils
  libpulse        lib32-libpulse
  alsa-plugins    lib32-alsa-plugins
  alsa-lib        lib32-alsa-lib
  oss             cups
  samba
)

if [[ $CARCH == i686 ]]; then
  # Strip lib32 etc. on i686
  depends=(${depends[@]/*32-*/})
  makedepends=(${makedepends[@]/*32-*/})
  makedepends=(${makedepends[@]/*-multilib*/})
  optdepends=(${optdepends[@]/*32-*/})
  provides=("wine=$pkgver")
  conflicts=(wine)
else
  provides=("wine=$pkgver" "bin32-wine=$pkgver" "wine-wow64=$pkgver")
  conflicts=('wine' 'bin32-wine' 'wine-wow64')
  replaces=('wine' 'bin32-wine')
fi

build() {
  # Patching
  cd "$srcdir/$_pkgname-$_pkgbasever"

  # up fps
  msg2 "Applaing patch for up fps..."
  patch -p1 < "$startdir/0002-disable_dynamic_vertex_buffers.patch" || exit 1

  ./tools/make_requests

  cd "$srcdir"

  # Allow ccache to work
  mv $_pkgname-$_pkgbasever $_pkgname

  # Get rid of old build dirs
  rm -rf $_pkgname-{32,64}-build
  mkdir $_pkgname-32-build

  # These additional CFLAGS solve FS#27662
  export CFLAGS="${CFLAGS/-D_FORTIFY_SOURCE=2/} -D_FORTIFY_SOURCE=0"
  export CXXFLAGS="${CFLAGS/-D_FORTIFY_SOURCE=2/} -D_FORTIFY_SOURCE=0"

  if [[ $CARCH == x86_64 ]]; then
    msg2 "Building Wine-64..."

    mkdir $_pkgname-64-build
    cd "$srcdir/$_pkgname-64-build"
    ../$_pkgname/configure \
      --prefix=/usr \
      --sysconfdir=/etc \
      --libdir=/usr/lib \
      --with-x \
      --enable-win64

    make

    _wine32opts=(
      --libdir=/usr/lib32
      --with-wine64="$srcdir/$_pkgname-64-build"
    )

    export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"
  fi

  msg2 "Building Wine-32..."
  cd "$srcdir/$_pkgname-32-build"
  ../$_pkgname/configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --with-x \
    "${_wine32opts[@]}"

  # These additional CFLAGS solve FS#27560
  make CFLAGS+="-mstackrealign" CXXFLAGS+="-mstackrealign"
}

package() {
  msg2 "Packaging Wine-32..."
  cd "$srcdir/$_pkgname-32-build"

  if [[ $CARCH == i686 ]]; then
    make prefix="$pkgdir/usr" install
  else
    make prefix="$pkgdir/usr" \
      libdir="$pkgdir/usr/lib32" \
      dlldir="$pkgdir/usr/lib32/wine" install

    msg2 "Packaging Wine-64..."
    cd "$srcdir/$_pkgname-64-build"
    make prefix="$pkgdir/usr" \
      libdir="$pkgdir/usr/lib" \
      dlldir="$pkgdir/usr/lib/wine" install
  fi
}

# vim:set ts=8 sts=2 sw=2 et:
