# Maintainer: Michael Gallagher <michael.ray.gallagher@gmail.com>
pkgver=28.0.50.149472
pkgrel=1
pkgdesc="GNU Emacs. Development master branch. Minimal options and optimized compiling."
arch=('x86_64')
url="http://www.gnu.org/software/emacs/"
license=('GPL3')
depends=('alsa-lib' 'gnutls' 'libxml2' 'jansson' 'gpm' 'libgccjit')
makedepends=('git')
provides=('emacs')
conflicts=('emacs' 'emacs26-git' 'emacs-27-git' 'emacs-seq' 'emacs-nox')
replaces=('emacs' 'emacs26-git' 'emacs-27-git' 'emacs-seq' 'emacs-nox')
source=("emacs-git::git://github.com/emacs-mirror/emacs.git")
options=(!strip)
pkgname=emacs-minimal-git 
install="$pkgname".install
b2sums=('SKIP')
################################################################################
CLANG="NO"
LTO="YES"

if [[ $CLANG == "YES" ]]; then
  export CC="/usr/bin/clang" ;
  export CXX="/usr/bin/clang++" ;
  export CPP="/usr/bin/clang -E" ;
  export LD="/usr/bin/lld" ;
  export AR="/usr/bin/llvm-ar" ;
  export AS="/usr/bin/llvm-as" ;
  export CCFLAGS+=' -fuse-ld=lld' ;
  export CXXFLAGS+=' -fuse-ld=lld' ;
  makedepends+=( 'clang' 'lld' 'llvm') ;
else            
  export CFLAGS="-pipe -march=native -mtune=native -Ofast -fomit-frame-pointer -fno-finite-math-only"
  export LDFLAGS="-pipe -march=native -mtune=native -Ofast -fno-finite-math-only"
fi

if [[ $LTO == "YES" ]]; then
  if [[ $CLANG != "YES" ]]; then
  CFLAGS+=" -flto -fuse-linker-plugin"
  CXXFLAGS+=" -flto -fuse-linker-plugin"
else
  CFLAGS+=" -flto"
  CXXFLAGS+=" -flto"
  fi
fi

################################################################################
pkgver() {
  cd "$srcdir/emacs-git"

  printf "%s.%s" \
    "$(grep AC_INIT configure.ac | \
    sed -e 's/^.\+\ \([0-9]\+\.[0-9]\+\.[0-9]\+\?\).\+$/\1/')" \
    "$(git rev-list --count HEAD)"
}

# There is no need to run autogen.sh after first checkout.
# Doing so, breaks incremental compilation.
prepare() {
  cd "$srcdir/emacs-git"
  [[ -x configure ]] || ( ./autogen.sh git && ./autogen.sh autoconf )
}

if [[ $CHECK == "YES" ]]; then
check() {
  cd "$srcdir/emacs-git"
  make check
}
fi

build() {
  cd "$srcdir/emacs-git"

  local _conf=(
    --prefix=/usr
    --sysconfdir=/etc
    --libexecdir=/usr/lib
    --localstatedir=/var
    --mandir=/usr/share/man
    --with-gameuser=:games
    --with-sound=alsa
    --with-modules
    --without-gconf
    --without-gsettings
    --with-libsystemd 
    --localstatedir=/var
    --with-compress-install
    --without-x
    --without-sound
    --without-xpm
    --without-jpeg
    --without-png
    --without-gif
    --without-imagemagick
    --with-mailutils
    --with-pop
    --without-rsvg
    --without-lcms2
    --without-xml2
    --without-json
    --without-xft
    --without-harfbuzz
    --without-libotf
    --without-m17n-flt
    --without-toolkit-scroll-bars
    --without-xaw3d
    --without-xim
    --without-gpm
    --without-dbus
    --without-gsettings
    --without-selinux
    --with-modules
    --without-makeinfo
    --without-libgmp
    --without-tiff
    --without-cairo
    --disable-largefile
    --enable-link-time-optimization
    --with-native-compilation
  )

################################################################################

if [[ $CLANG == "YES" ]]; then
  _conf+=( '--enable-autodepend' );
fi

if [[ $LTO == "YES" ]]; then
  _conf+=( '--enable-link-time-optimization' );
fi

# ctags/etags may be provided by other packages, e.g, universal-ctags
_conf+=('--program-transform-name=s/\([ec]tags\)/\1.emacs/')

################################################################################

  ./configure "${_conf[@]}"

  # Using "make" instead of "make bootstrap" enables incremental
  # compiling. Less time recompiling. Yay! But you may
  # need to use bootstrap sometimes to unbreak the build.
  # Just add it to the command line.
  #
  # Please note that incremental compilation implies that you
  # are reusing your src directory!
  #
  make NATIVE_FULL_AOT=1 -j $(nproc)

}

package() {
  cd "$srcdir/emacs-git"

  make DESTDIR="$pkgdir/" install

  # Install optional documentation formats
  if [[ $DOCS_HTML == "YES" ]]; then make DESTDIR="$pkgdir/" install-html; fi
  if [[ $DOCS_PDF == "YES" ]]; then make DESTDIR="$pkgdir/" install-pdf; fi

  # fix user/root permissions on usr/share files
  # MOVED to install script
  # find "$pkgdir"/usr/share/emacs/ | xargs chown root:root

  # fix permssions on /var/games
  mkdir -p "$pkgdir"/var/games/emacs
  chmod 775 "$pkgdir"/var/games
  chmod 775 "$pkgdir"/var/games/emacs
  # MOVED to install script
  # chown -R root:games "$pkgdir"/var/games

}
