# Maintainer:  Giovanni 'ItachiSan' Santini <giovannisantini93@yahoo.it>
# Contributor: Filipe Laíns (FFY00) <lains@archlinux.org>
# Contributor: Pieter Goetschalckx <3.14.e.ter <at> gmail <dot> com>

pkgname=franz
#pkgver=${_pkgver//-/_} # Leaving it here for possible dev/beta package :)
pkgver=5.10.0
pkgrel=1
# Due to the previous "_beta" naming
epoch=1
pkgdesc='Free messaging app for services like WhatsApp, Slack, Messenger and many more.'
arch=(x86_64 i686)
url='https://meetfranz.com'
license=(Apache)
# Allow to easily switch between Electron versions.
# Expected one is 'electron25' (Electron 25). May change soon.
# This is automatically replaced in `franz.sh` with the package name, as
# the executable matches the package name (as of 2023-09-11).
_electron='electron25'
depends=($_electron)
makedepends=(expac git nvm python)
source=("git+https://github.com/meetfranz/$pkgname#tag=v$pkgver"
        franz.desktop
        franz.sh.in
        disable-prompt.patch)
sha512sums=('SKIP'
            '049c4bf2e0f362f892e8eef28dd18a6c321251c686a9c9e49e4abfb778057de2fc68b95b4ff7bb8030a828a48b58554a56b810aba078c220cb01d5837083992e'
            '7ccf058421b173830493f35417d204e3a735fc20f801283dad3f658abeb484f6244bc535634c2f02ab2cb8e35a0e1a92dd3d06be5943e121ddccbbee7ad74b48'
            'a0c83c33ba9544fb9a2c7bb8a8c1b3200e3377fb3a97848c031da53368520ab9a41e4ff2c37e6e7d86944dbb8d7068cf6330c00caca2c494ff8de6a0c78bd1d7')

# Helper function for setting up nvm nicely.
# Found here: https://wiki.archlinux.org/title/Node.js_package_guidelines#Using_nvm
# Personal extras at the bottom of the function :)
_ensure_nvm_setup() {
  # let's be sure we are starting clean
  which nvm >/dev/null 2>&1 && nvm deactivate && nvm unload
  export NVM_DIR="$srcdir/.nvm"

  # The init script returns 3 if version specified
  # in ./.nvrc is not (yet) installed in $NVM_DIR
  # but nvm itself still gets loaded ok
  source /usr/share/nvm/init-nvm.sh || [[ $? != 1 ]]

  # Personal extras
  # Avoid installing Electron as we use the system one
  export ELECTRON_SKIP_BINARY_DOWNLOAD=1
  # For safety, define the NPM cache in $srcdir
  # Better configuration for npm cache and calling installed binaries
  export npm_config_cache="$srcdir/npm_cache"
}

prepare() {
  # Small patching
  cd "$pkgname"

  # Adjust the electron version to use when building
  echo "--> Using Electron package:   $_electron"
  electron_version="`expac %v $_electron | cut -d'-' -f1`"
  echo "--> Electron package version: $electron_version"
  sed -i -E "s|(\s+\"electron\":).*,|\1 \"$electron_version\",|" package.json

  # Prevent Franz from being launched in development mode
  # This changes all the occurences where 'isDevMode' is set to a value.
  grep -lr 'isDevMode =' src | xargs sed -E 's|^(.*isDevMode =) .*$|\1 false|' -i

  # Apply patches
  echo "--> Applying patches"
  git apply --ignore-whitespace --recount ../../disable-prompt.patch

  # Setup nvm
  _ensure_nvm_setup

  echo "--> Install toolchain with nvm"
  nvm install

  echo "--> Install updated node-gyp"
  npm install node-gyp@9

  echo "--> Install modules dependencies"
  # The author still uses old dependencies resolution.
  # Luckily it is "documented" in the Appveyor CI file.
  # Also in the Github CI:
  # https://github.com/meetfranz/franz/blob/master/.github/workflows/build.yml#L56
  # About the double double escape, that is because the first '--' is for
  # npm exec, while the second is for lerna.
  # See: https://github.com/lerna/lerna/issues/2921
  # npm exec --package="lerna@3.8" --yes -- lerna bootstrap
  # However, after the installation above, all works fine ;)
  npm install
}

build() {
  cd "$pkgname"

  # Be sure we are correctly setup
  _ensure_nvm_setup

  # Actually build the package
  echo "--> Building the package"
  npm exec lerna run build
  npm exec gulp build
  npm exec -- electron-builder --linux dir
}

check() {
  cd "$pkgname"

  # Be sure we are correctly setup
  _ensure_nvm_setup

  # Run the tests
  echo "--> Running the tests"
  npm run test
}

package() {
  cd $pkgname

  # Point the proper Electron package version, so that people can complain when it's updated.
  # This is for extra safety & reminds me of upgrading the package.
  # Keeping this here if I'll switch back to `electron`, which is more unstable.
  #electron_version="`expac %v $_electron | cut -d'-' -f1`"
  #depends=("${_electron}=${electron_version}")

  # Install the .asar files
  install -dm 755 "$pkgdir/usr/lib/$pkgname"
  cp -r --no-preserve=ownership --preserve=mode out/linux-unpacked/resources "$pkgdir/usr/lib/$pkgname/"

  # Install icon
  install -Dm 644 "$srcdir/franz.desktop" "$pkgdir/usr/share/applications/franz.desktop"
  install -Dm 644 build-helpers/images/icon.png "$pkgdir/usr/share/icons/franz.png"

  # Install run script
  sed "s|@ELECTRON@|$_electron|" "$srcdir/franz.sh.in" > franz.sh
  install -Dm 755 franz.sh "$pkgdir/usr/bin/franz"
}
