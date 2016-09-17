pkgname=sks
pkgver=1.1.6
pkgrel=1
arch=('i686' 'x86_64')
license=('GPL')
pkgdesc='Synchronizing OpenPGP Key Server'
makedepends=('ocaml' 'db' 'camlp4')
url='https://bitbucket.org/skskeyserver/sks-keyserver/'
install='sks.install'
backup=('etc/sks/sksconf'
        'etc/sks/forward.exim'
        'etc/sks/forward.postfix'
        'etc/sks/mailsync'
        'etc/sks/membership'
        'etc/sks/procmail')
source=(https://bitbucket.org/skskeyserver/sks-keyserver/downloads/${pkgname}-${pkgver}.tgz{,.asc}
        '500_debian_fhs.patch'
        'sks-db.service'
        'sks-recon.service')
md5sums=('43f0cc8b4b43d798d453fedad840f926'
         'SKIP'
         '9cf5495b95e84ed91788c04c9ce1b8c1'
         'e8c7dcbb7db3ad879d391a7c0127a068'
         'f28a2d0b151996a99bb006b8e1d29408')
validpgpkeys=(C90EF1430B3AC0DFD00E6EA541259773973A612A) # SKS Keyserver Signing Key


prepare() {
  cd "$pkgname-$pkgver"

  # patch path
  patch -Np1 -i "$srcdir/500_debian_fhs.patch"

  cp Makefile.local.unused Makefile.local

  # XXX Due to how ocaml package is generated in arch, we cannot link
  # dynamically, so we workarround the problem using runtime-variant _pic
  # More info:
  #   - https://wiki.ubuntu.com/SteveBeattie/PIENotes#Incompatible_relocation_R_X86_64_32
  #   - https://bugs.archlinux.org/task/42748
  #   - http://caml.inria.fr/mantis/view.php?id=6693
  echo "OCAMLOPT=ocamlopt -runtime-variant _pic" >> Makefile.local

  sed -i -e 's#LIBDB=-ldb-4.6#LIBDB=-ldb-5.3#g' Makefile.local
  sed -i -e "s#/usr/local#$pkgdir/usr#g" Makefile.local
  sed -i -e "s#/usr/share/man#$pkgdir/usr/share/man#g" Makefile.local
}

build() {
  cd "$pkgname-$pkgver"
  unset MAKEFLAGS
  make dep

  # XXX Parallel compiling not supporting for Bdb module, force -j1 always.
  make CFLAGS="$CFLAGS -I`ocamlc -where` -I ." -j1 all
}

package() {
  cd "$pkgname-$pkgver"

  make PREFIX="$pkgdir/usr" MANDIR="$pkgdir/usr/share/man" install

  install -Dm644 "$srcdir/sks-db.service" "$pkgdir/usr/lib/systemd/system/sks-db.service"
  install -Dm644 "$srcdir/sks-recon.service" "$pkgdir/usr/lib/systemd/system/sks-recon.service"

  mkdir -p "$pkgdir/etc" "$pkgdir/var/lib/sks"

  cp -r sampleWeb/OpenPKG "$pkgdir/var/lib/sks"
  cp -r sampleConfig/debian "$pkgdir/etc/sks"

  sed -i -e 's#/usr/lib/sendmail#/usr/sbin/sendmail#g' "$pkgdir/etc/sks/sksconf"
}
