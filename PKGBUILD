# Maintainer: DJ Lucas <dj_AT_linuxfromscratch_DOT_org>
pkgname=openchange-git
pkgver=2.5
pkgrel=1
pkgdesc="A portable, open source implementation of Microsoft Exchange server and Exchange protocols."
arch=('i686' 'x86_64')
url="http://www.openchange.org"
license=('GPL3')
depends=('samba>=4.0.2-1' 'libical' 'sqlite3' 'file' 'boost' 'python2'
         'mod_wsgi' 'python2-pycurl' 'python2-pylons' 'python2-lxml')
makedepends=('ccache' 'python2' 'docbook-xsl' 'libxslt' 'python2-pylons' 'unzip')
options=(!makeflags emptydirs)
conflicts=('openchange')
replaces=('openchange')
backup=('etc/httpd/conf/extra/ocsmanager.conf'
        'etc/httpd/conf/extra/rpcproxy.conf'
        'etc/ocsmanager/ocsmanager.ini')
# Releases are mirrored at http://tracker.openchange.org/projects/openchange/files
source=("http://downloads.sourceforge.net/sourceforge/flex/flex-2.5.35.tar.bz2"
        "ocsmanager.service")
sha256sums=('0becbd4b2b36b99c67f8c22ab98f7f80c9860aec70f0350a0018f29a88704e7b'
            '45bd19e2a5725a94692ae606086be6d57423375c9b1c0eb5322c6e09ef2b5fb3')

build() {
    # Build a temporary flex of the proper version
    cd "${srcdir}/flex-2.5.35"
    ./configure --prefix="${srcdir}/buildtools"
    make
    make install

    cd "${srcdir}"
    wget https://github.com/openchange/openchange/archive/master.zip
    unzip "${srcdir}/master.zip"
    cd openchange-master

    PYTHON_CALLERS="$(find ${srcdir}/openchange-master -name '*.py')
                    $(find ${srcdir}/openchange-master -name 'configure.ac')
                    setup/openchange_newuser setup/openchange_provision
                    mapiproxy/services/web/rpcproxy/rpcproxy.wsgi"
    sed -i -e "s|/usr/bin/env python$|/usr/bin/env python2|" \
           -e "s|python-config|python2-config|" \
           -e "s|bin/python|bin/python2|" \
        ${PYTHON_CALLERS}

    # Fix linking of boost_thread in autoconf test
    sed -i -e "s|-lboost_thread\$BOOST_LIB_SUFFIX|-lboost_thread\$BOOST_LIB_SUFFIX -lboost_system\$BOOST_LIB_SUFFIX|" \
        configure.ac

    export PYTHON=/usr/bin/python2

    export PKG_CONFIG_PATH="/usr/samba/lib/pkgconfig:/usr/lib/pkgconfig"
    ./autogen.sh
    FLEX="${srcdir}/buildtools/bin/flex" \
    ./configure --prefix=/usr --disable-pymapi \
                --with-modulesdir=/usr/lib/samba/modules \
                --datadir=/usr/share/samba
    make
}

package() {
    _pyver=`python2 -c 'import sys; print(sys.version[:3])'`

    cd "${srcdir}/openchange-master"
    make DESTDIR="${pkgdir}/" install
    install -vdm755 ${pkgdir}/usr/include
    cp -r libmapi++ ${pkgdir}/usr/include
    install -vdm755 ${pkgdir}/usr/share/man
    cp -r doc/man/man1 ${pkgdir}/usr/share/man/
    cd ${pkgdir}/usr/lib/
    ln -s libmapi.so libmapi.so.0
    ln -s libocpf.so libocpf.so.0

    # Install OCSManager
    cd ${srcdir}/openchange-master/mapiproxy/services/ocsmanager
    install -vdm755 ${pkgdir}/etc/ocsmanager
    install -vm644 ocsmanager.ini ${pkgdir}/etc/ocsmanager/ocsmanager.ini
    install -vdm755 ${pkgdir}/etc/httpd/conf/extra
    install -vm644 ocsmanager-apache.conf \
                   ${pkgdir}/etc/httpd/conf/extra/ocsmanager.conf
    install -vdm755 ${pkgdir}/usr/lib/systemd/system
    install -vm644 ${srcdir}/ocsmanager.service ${pkgdir}/usr/lib/systemd/system
    install -vdm700 -o http -g http ${pkgdir}/var/cache/ntlmauthhandler
    python2.7 setup.py install --root=${pkgdir} --prefix=/usr \
                               --single-version-externally-managed
    find ${pkgdir} -name '*.pth' -delete

    # Install rpcproxy
    cd ${srcdir}/openchange-master/mapiproxy/services/web/rpcproxy
    install -vdm755 ${pkgdir}/etc/httpd/conf/extra
    install -vm644 rpcproxy.conf ${pkgdir}/etc/httpd/conf/extra
    install -vdm755 ${pkgdir}/usr/lib/openchange/web/rpcproxy
    install -vm644 rpcproxy.wsgi \
                   ${pkgdir}/usr/lib/openchange/web/rpcproxy/rpcproxy.wsgi
    python2.7 setup.py install --install-lib=/usr/lib/openchange/web/rpcproxy \
        --root ${pkgdir} --install-scripts=/usr/lib/openchange/web/rpcproxy
    rm -f ${pkgdir}/usr/lib/openchange/web/rpcproxy/rpcproxy/*pyc

    find ${pkgdir}/usr/lib/python${_pyver}/site-packages/ -name '*.py' | \
         xargs sed -i "s|#!/usr/bin/env python$|#!/usr/bin/env python2|"
}
