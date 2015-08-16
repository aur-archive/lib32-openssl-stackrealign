# $Id$
# Maintainer: Pierre Schmitz <pierre@archlinux.de>

_pkgbasename=openssl
pkgname=lib32-${_pkgbasename}-stackrealign
_ver=1.0.1c
# use a pacman compatible version scheme
pkgver=${_ver/[a-z]/.${_ver//[0-9.]/}}
#pkgver=$_ver
pkgrel=1.1
pkgdesc='The Open Source toolkit for Secure Sockets Layer and Transport Layer Security (32-bit). Added -mstackrealign so that it works OK with wine.'
arch=('x86_64')
url='https://www.openssl.org'
license=('custom:BSD')
depends=('lib32-zlib' "${_pkgbasename}")
provides=('lib32-openssl')
conflicts=('lib32-openssl')
optdepends=('ca-certificates')
makedepends=('gcc-multilib')
options=('!makeflags')
source=("https://www.openssl.org/source/${_pkgbasename}-${_ver}.tar.gz"
        "https://www.openssl.org/source/${_pkgbasename}-${_ver}.tar.gz.asc"
        'no-rpath.patch'
        'ca-dir.patch')
md5sums=('ae412727c8c15b67880aef7bd2999b2e'
         'a3d90bc42253def61cd1c4237f1ce5f7'
         'dc78d3d06baffc16217519242ce92478'
         '3bf51be3a1bbd262be46dc619f92aa90')

build() {
	export CC="gcc -m32"
	export CXX="g++ -m32"
	export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"

	cd $srcdir/$_pkgbasename-$_ver

	# remove rpath: http://bugs.archlinux.org/task/14367
	patch -p0 -i $srcdir/no-rpath.patch
	# set ca dir to /etc/ssl by default
	patch -p0 -i $srcdir/ca-dir.patch
	# mark stack as non-executable: http://bugs.archlinux.org/task/12434
	# workaround for PR#2771: OPENSSL_NO_TLS1_2_CLIENT
	./Configure --prefix=/usr --openssldir=/etc/ssl --libdir=lib32 \
		shared zlib enable-md2 \
		linux-elf \
		-Wa,--noexecstack "${CFLAGS}" "${LDFLAGS}" \
		-mstackrealign \
		-DOPENSSL_NO_TLS1_2_CLIENT

	make MAKEDEPPROG="${CC}" depend
	make
}

check() {
	cd $srcdir/$_pkgbasename-$_ver
	# the test fails due to missing write permissions in /etc/ssl
	# revert this patch for make test
	patch -p0 -R -i $srcdir/ca-dir.patch
	make test
	patch -p0 -i $srcdir/ca-dir.patch
}

package() {
	cd $srcdir/$_pkgbasename-$_ver
	make INSTALL_PREFIX=$pkgdir install

	rm -rf ${pkgdir}/{usr/{include,share,bin},etc}
	mkdir -p $pkgdir/usr/share/licenses
	ln -s $_pkgbasename $pkgdir/usr/share/licenses/$pkgname
}
