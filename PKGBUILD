# $Id$
# Maintainer: Andrew Soutar <andrew@andrewsoutar.com>

pkgbase=systemd-fix-6381
pkgname=('systemd-fix-6381' 'libsystemd-fix-6381' 'systemd-sysvcompat-fix-6381')
# Bump this to latest major release for signed tag verification,
# the commit count is handled by pkgver() function.
pkgver=234.13
pkgrel=2
arch=('i686' 'x86_64')
url="https://www.github.com/andrewsoutar/systemd"
makedepends=('acl' 'cryptsetup' 'docbook-xsl' 'gperf' 'lz4' 'xz' 'pam' 'libelf'
             'intltool' 'iptables' 'kmod' 'libcap' 'libidn' 'libgcrypt'
             'libmicrohttpd' 'libxslt' 'util-linux' 'linux-api-headers'
             'python-lxml' 'quota-tools' 'shadow' 'gnu-efi-libs' 'git'
             'meson')
options=('strip')
source=("git://github.com/andrewsoutar/systemd-stable.git#branch=v${pkgver%.*}-stable"
        'git://github.com/systemd/systemd.git' # pull in for tags
        'initcpio-hook-udev'
        'initcpio-install-systemd'
        'initcpio-install-udev'
        'arch.conf'
        'loader.conf'
        'splash-arch.bmp'
        'systemd-user.pam'
        'systemd-hwdb.hook'
        'systemd-sysusers.hook'
        'systemd-tmpfiles.hook'
        'systemd-update.hook')
sha512sums=('SKIP'
            'SKIP'
            'f0d933e8c6064ed830dec54049b0a01e27be87203208f6ae982f10fb4eddc7258cb2919d594cbfb9a33e74c3510cfd682f3416ba8e804387ab87d1a217eb4b73'
            '691acebb243b9cd7fb63272662f34bdb9aead710c69aee9361ab2322f9f108600ad5b0214fc00b7cb2d9c95db8abd748030625d60d6567efd98663c56ba28c65'
            'a25b28af2e8c516c3a2eec4e64b8c7f70c21f974af4a955a4a9d45fd3e3ff0d2a98b4419fe425d47152d5acae77d64e69d8d014a7209524b75a81b0edb10bf3a'
            '61032d29241b74a0f28446f8cf1be0e8ec46d0847a61dadb2a4f096e8686d5f57fe5c72bcf386003f6520bc4b5856c32d63bf3efe7eb0bc0deefc9f68159e648'
            'c416e2121df83067376bcaacb58c05b01990f4614ad9de657d74b6da3efa441af251d13bf21e3f0f71ddcb4c9ea658b81da3d915667dc5c309c87ec32a1cb5a5'
            '5a1d78b5170da5abe3d18fdf9f2c3a4d78f15ba7d1ee9ec2708c4c9c2e28973469bc19386f70b3cf32ffafbe4fcc4303e5ebbd6d5187a1df3314ae0965b25e75'
            'b90c99d768dc2a4f020ba854edf45ccf1b86a09d2f66e475de21fe589ff7e32c33ef4aa0876d7f1864491488fd7edb2682fc0d68e83a6d4890a0778dc2d6fe19'
            '2c1f765e7cefc50f07ad994634ea25d9396e6b9c0de46e58f18377e642a471517a0dbf5eb547070a38c6ecf84ec8e030f650a6cee010871cd7a466a32534adda'
            '9d27d97f172a503f5b7044480a0b9ccc0c4ed5dbb2eb3b2b1aa929332c3bcfe38ef0c0310b6566f23b34f9c05b77035221164a7ab7677784c4a54664f12fca22'
            '0f4efddd25256e09c42b953caeee4b93eb49ecc6eaebf02e616b4dcbfdac9860c3d8a3d1a106325b2ebc4dbc6e08ac46702abcb67a06737227ccb052aaa2a067'
            '10190fba9f39a8f4b620a0829e0ba8ed63bb4dbeca712966011ee7807880d01ab2abff1a80baafeb6674db70526a473fe585db8190e864f318fc4d6068552618')
validpgpkeys=(
  '63CDA1E5D3FC22B998D20DD6327F26951A015CC4'  # Lennart Poettering
)

_backports=(
)

_reverts=(
)

_validate_tag() {
  local success fingerprint trusted status tag=v${pkgver%.*}

  parse_gpg_statusfile /dev/stdin < <(git verify-tag --raw "$tag" 2>&1)

  if (( ! success )); then
    error 'failed to validate tag %s\n' "$tag"
    return 1
  fi

  if ! in_array "$fingerprint" "${validpgpkeys[@]}" && (( ! trusted )); then
    error 'unknown or untrusted public key: %s\n' "$fingerprint"
    return 1
  fi

  case $status in
    'expired')
      warning 'the signature has expired'
      ;;
    'expiredkey')
      warning 'the key has expired'
      ;;
  esac

  return 0
}

pkgver() {
  local version count

  cd "systemd-stable"

  version="$(git describe --abbrev=0 --tags)"
  count="$(git rev-list --count ${version}..)"
  printf '%s.%s' "${version#v}" "${count}"
}

prepare() {
  cd "systemd-stable"

  git remote add upstream ../systemd/
  git fetch --all

  _validate_tag || return

  local _commit
  for _commit in "${_backports[@]}"; do
    git cherry-pick -n "$_commit"
  done
  for _commit in "${_reverts[@]}"; do
    git revert -n "$_commit"
  done

  # core: store the invocation ID in the per-service keyring
  # core: run each system service with a fresh session keyring
  git show 'b3415f5daef49642be3d5f417b8880c078420ff7' \
    '74dd6b515fa968c5710b396a7664cac335e25ca8' \
    -- . ':!src/test/test-id128.c' | git apply --reverse --index
}

build() {
  local timeservers=({0..3}.arch.pool.ntp.org)

  local meson_options=(
    -Daudit=false
    -Dgnuefi=true
    -Dima=false
    -Dlz4=true
    -Db_lto=true

    -Ddbuspolicydir=/usr/share/dbus-1/system.d
    -Ddefault-dnssec=no
    # TODO(dreisner): consider changing this to unified
    -Ddefault-hierarchy=hybrid
    -Ddefault-kill-user-processes=false
    -Dfallback-hostname='archlinux'
    -Dntp-servers="${timeservers[*]}"
    -Drpmmacrosdir=no
    -Dsysvinit-path=
    -Dsysvrcnd-path=
  )

  meson "systemd-stable" build "${meson_options[@]}"

  ninja -C build
}

package_systemd-fix-6381() {
  pkgdesc="system and service manager"
  license=('GPL2' 'LGPL2.1')
  depends=('acl' 'bash' 'cryptsetup' 'dbus' 'iptables' 'kbd' 'kmod' 'hwids' 'libcap'
           'libgcrypt' 'libsystemd-fix-6381' 'libidn' 'lz4' 'pam' 'libelf' 'libseccomp'
           'util-linux' 'xz')
  provides=('systemd' 'nss-myhostname' "systemd-tools=$pkgver" "udev=$pkgver")
  replaces=('systemd' 'nss-myhostname' 'systemd-tools' 'udev')
  conflicts=('systemd' 'nss-myhostname' 'systemd-tools' 'udev')
  optdepends=('libmicrohttpd: remote journald capabilities'
              'quota-tools: kernel-level quota management'
              'systemd-sysvcompat-fix-6381: symlink package to provide sysvinit binaries'
              'polkit: allow administration as unprivileged user')
  backup=(etc/pam.d/systemd-user
          etc/systemd/coredump.conf
          etc/systemd/journald.conf
          etc/systemd/journal-remote.conf
          etc/systemd/journal-upload.conf
          etc/systemd/logind.conf
          etc/systemd/system.conf
          etc/systemd/timesyncd.conf
          etc/systemd/resolved.conf
          etc/systemd/user.conf
          etc/udev/udev.conf)
  install="systemd.install"

  DESTDIR="$pkgdir" ninja -C build install

  # don't write units to /etc by default. some of these will be re-enabled on
  # post_install.
  rm -r "$pkgdir/etc/systemd/system/"*.wants
  rm -r "$pkgdir/etc/systemd/system/"*.service

  # add back tmpfiles.d/legacy.conf
  install -m644 "systemd-stable/tmpfiles.d/legacy.conf" "$pkgdir/usr/lib/tmpfiles.d"

  # Replace dialout/tape/cdrom group in rules with uucp/storage/optical group
  sed -i 's#GROUP="dialout"#GROUP="uucp"#g;
          s#GROUP="tape"#GROUP="storage"#g;
          s#GROUP="cdrom"#GROUP="optical"#g' "$pkgdir"/usr/lib/udev/rules.d/*.rules
  sed -i 's/dialout/uucp/g;
          s/tape/storage/g;
          s/cdrom/optical/g' "$pkgdir"/usr/lib/sysusers.d/basic.conf

  # add mkinitcpio hooks
  install -Dm644 "$srcdir/initcpio-install-systemd" "$pkgdir/usr/lib/initcpio/install/systemd"
  install -Dm644 "$srcdir/initcpio-install-udev" "$pkgdir/usr/lib/initcpio/install/udev"
  install -Dm644 "$srcdir/initcpio-hook-udev" "$pkgdir/usr/lib/initcpio/hooks/udev"

  # ensure proper permissions for /var/log/journal. This is only to placate
  chown root:systemd-journal "$pkgdir/var/log/journal"
  chmod 2755 "$pkgdir/var/log/journal"

  # match directory owner/group and mode from extra/polkit
  chown root:102 "$pkgdir"/usr/share/polkit-1/rules.d
  chmod 0750     "$pkgdir"/usr/share/polkit-1/rules.d

  # we'll create this on installation
  rmdir "$pkgdir/var/log/journal/remote"

  # ship default policy to leave services disabled
  echo 'disable *' >"$pkgdir"/usr/lib/systemd/system-preset/99-default.preset

  # manpages shipped with systemd-sysvcompat
  rm "$pkgdir"/usr/share/man/man8/{telinit,halt,reboot,poweroff,runlevel,shutdown}.8

  # runtime libraries shipped with libsystemd
  rm "$pkgdir"/usr/lib/lib{nss,systemd,udev}*.so*

  # allow core/filesystem to pristine nsswitch.conf
  rm "$pkgdir/usr/share/factory/etc/nsswitch.conf"
  sed -i '/^C \/etc\/nsswitch\.conf/d' "$pkgdir/usr/lib/tmpfiles.d/etc.conf"

  # add example bootctl configuration
  install -Dm644 "$srcdir/arch.conf" "$pkgdir"/usr/share/systemd/bootctl/arch.conf
  install -Dm644 "$srcdir/loader.conf" "$pkgdir"/usr/share/systemd/bootctl/loader.conf
  install -Dm644 "$srcdir/splash-arch.bmp" "$pkgdir"/usr/share/systemd/bootctl/splash-arch.bmp

  install -Dm644 "$srcdir/systemd-hwdb.hook" "$pkgdir/usr/share/libalpm/hooks/systemd-hwdb.hook"
  install -Dm644 "$srcdir/systemd-sysusers.hook" "$pkgdir/usr/share/libalpm/hooks/systemd-sysusers.hook"
  install -Dm644 "$srcdir/systemd-tmpfiles.hook" "$pkgdir/usr/share/libalpm/hooks/systemd-tmpfiles.hook"
  install -Dm644 "$srcdir/systemd-update.hook" "$pkgdir/usr/share/libalpm/hooks/systemd-update.hook"

  # overwrite the systemd-user PAM configuration with our own
  install -Dm644 systemd-user.pam "$pkgdir/etc/pam.d/systemd-user"
}

package_libsystemd-fix-6381() {
  pkgdesc="systemd client libraries"
  replaces=('libsystemd')
  conflicts=('libsystemd')
  depends=('glibc' 'libcap' 'libgcrypt' 'lz4' 'xz')
  license=('GPL2')
  provides=('libsystemd' 'libsystemd.so' 'libudev.so')

  # meson does not support installing subsets of files, no?
  # So do a full install to temporary directory, then install what we need.
  DESTDIR="$srcdir"/full-install ninja -C build install

  install -dm755 "$pkgdir"/usr/lib/
  cp --archive "$srcdir"/full-install/usr/lib/lib{nss_*,systemd,udev}.so* "$pkgdir"/usr/lib/
}

package_systemd-sysvcompat-fix-6381() {
  pkgdesc="sysvinit compat for systemd"
  license=('GPL2')
  groups=('base')
  provides=('systemd-sysvcompat')
  replaces=('systemd-sysvcompat')
  conflicts=('systemd-sysvcompat' 'sysvinit')
  depends=('systemd-fix-6381')

  install -dm755 "$pkgdir"/usr/share/man/man8
  cp -d --no-preserve=ownership,timestamp \
    build/man/{telinit,halt,reboot,poweroff,runlevel,shutdown}.8 \
    "$pkgdir"/usr/share/man/man8

  install -dm755 "$pkgdir/usr/bin"
  for tool in runlevel reboot shutdown poweroff halt telinit; do
    ln -s 'systemctl' "$pkgdir/usr/bin/$tool"
  done

  ln -s '../lib/systemd/systemd' "$pkgdir/usr/bin/init"
}

# vim: ft=sh syn=sh et
