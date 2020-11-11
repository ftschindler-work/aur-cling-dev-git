# Maintainer: Felix Schindler <aur at felixschindler dot net>
# Contributor: Sebastian Gsänger <sebastian_gsaenger@web.de>

pkgname=(cling-dev-git cling-jupyter-dev-git)
pkgver=0.7.r19.ge84dc544
pkgrel=1
pkgdesc="Interactive C++ interpreter (includes custom clang and llvm for dev)."
arch=('i686' 'x86_64')
url="https://root.cern.ch/cling"
license=('custom:Cling Release License')
makedepends=('cmake' 'libffi' 'git' 'python' 'python2' 'jupyter')
source=(
    "llvm::git+http://root.cern.ch/git/llvm.git#branch=cling-patches"
    "clang::git+http://root.cern.ch/git/clang.git#branch=cling-patches"
    "cling::git+http://root.cern.ch/git/cling.git#branch=master"
)
sha256sums=(
    'SKIP'
    'SKIP'
    'SKIP'
)

pkgver() {
  cd "${srcdir}"/cling
  git describe --long | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

prepare() {
    if [ ! -h "$srcdir/llvm/tools/clang" ]; then
        ln -s "$srcdir/clang" "$srcdir/llvm/tools/clang"
    fi

    if [ ! -h "$srcdir/llvm/tools/cling" ]; then
        ln -s "$srcdir/cling" "$srcdir/llvm/tools/cling"
    fi
}

build() {
    mkdir -p "$srcdir/cling-build"
    cd "$srcdir/cling-build"

    cmake \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX="/opt/cling" \
        -DLLVM_TARGETS_TO_BUILD="host;NVPTX" \
        -DLLVM_BUILD_LLVM_DYLIB=OFF \
        -DLLVM_ENABLE_RTTI=ON \
        -DLLVM_ENABLE_FFI=ON \
        -DLLVM_BUILD_DOCS=OFF \
        -DLLVM_BUILD_TOOLS=OFF \
        -DLLVM_ENABLE_SPHINX=OFF \
        -DLLVM_ENABLE_DOXYGEN=OFF \
        -DFFI_INCLUDE_DIR=$(pkg-config --cflags-only-I libffi | cut -c3-) \
        "$srcdir/llvm"

    cmake --build .
}

package_cling-dev-git() {
    provides=("cling=$pkgver" "cling-dev=$pkgver")
    conflicts=('cling')
    depends=('libffi')

    cd "$srcdir/cling-build"
    DESTDIR="$pkgdir" cmake --build . -t install

    install -d "$pkgdir/usr/bin"
    ln -s "/opt/cling/bin/cling" "$pkgdir/usr/bin/cling"
    install -d "$pkgdir/usr/include"
    ln -s "/opt/cling/include/cling" "$pkgdir/usr/include/cling"

    install -Dm644 "$srcdir/llvm/tools/cling/LICENSE.TXT" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_cling-jupyter-dev-git() {
    depends=('cling-dev-git' 'jupyter')
    provides=("cling-jupyter=$pkgver" "cling-jupyter-dev=$pkgver")

    cd "$srcdir/cling/tools/Jupyter/kernel"
    python3 setup.py install --prefix=/usr --root="$pkgdir"
    jupyter-kernelspec install --prefix="$pkgdir/usr" .
}
