# First-party recipe for the starnix aur-mirror. Repackages Objective
# Development's official Arch package. Nothing is pinned in this file: every
# build resolves the current stable release, verifies the vendor's signed hash
# list with Sigstore, and then requires the package to match it.
# The license allows redistribution only of the unmodified binary, with its
# license text, so never strip or patch it.
pkgname=littlesnitch
pkgver=0  # placeholder; pkgver() reports the verified release
pkgrel=1
pkgdesc="Network monitor that lets you decide which applications may connect (first-party, signature-verified official binary)"
arch=('x86_64')
url="https://obdev.at/littlesnitch-linux"
license=('GPL-2.0-only AND LicenseRef-Obdev-Freeware')
depends=('glibc' 'libgcc' 'pam' 'sqlite')
makedepends=('cosign' 'curl')
options=('!strip' '!debug')
install=littlesnitch.install

_page="https://obdev.at/products/littlesnitch-linux/download.html"
_base="https://obdev.at/downloads/littlesnitch-linux"
# Signing identity, documented in the vendor's hashes file.
_identity="littlesnitch-linux@obdev.at"
_issuer="https://github.com/login/oauth"

prepare() {
  cd "$srcdir"
  local ver pkg hashes sum
  ver=$(curl -fsSL "$_page" \
    | grep -oE "littlesnitch-[0-9]+(\.[0-9]+)+-1-x86_64\.pkg\.tar\.zst" \
    | sed -E 's/^littlesnitch-([0-9.]+)-1-.*/\1/' | sort -uV | tail -n1) || true
  [[ $ver =~ ^[0-9]+(\.[0-9]+)+$ ]] \
    || { echo "could not resolve the current version" >&2; return 1; }
  pkg="littlesnitch-$ver-1-x86_64.pkg.tar.zst"
  hashes="littlesnitch-$ver.hashes.txt"
  curl -fsSL -o "$hashes" "$_base/$hashes"
  curl -fsSL -o "$hashes.sig.json" "$_base/$hashes.sig.github.json"

  TUF_ROOT="$srcdir/tuf" cosign verify-blob \
    --bundle "$hashes.sig.json" \
    --certificate-identity="$_identity" \
    --certificate-oidc-issuer="$_issuer" "$hashes" \
    || { echo "hash list signature is invalid" >&2; return 1; }

  sum=$(awk -v f="$pkg" '$2 == f { print $1 }' "$hashes")
  [[ $sum =~ ^[0-9a-f]{64}$ ]] \
    || { echo "no single hash for $pkg" >&2; return 1; }
  curl -fsSL -o "$pkg" "$_base/$pkg"
  echo "$sum  $pkg" | sha256sum -c --quiet - \
    || { echo "package does not match the signed hash" >&2; return 1; }

  rm -rf extracted && mkdir extracted
  bsdtar -xf "$pkg" -C extracted --exclude '.[A-Z]*'
  echo "$ver" > version
}

pkgver() {
  cat "$srcdir/version"
}

package() {
  cp -dpr --no-preserve=ownership "$srcdir/extracted/." "$pkgdir/"
  install -Dm644 "$srcdir/extracted/usr/share/doc/littlesnitch/copyright" \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
