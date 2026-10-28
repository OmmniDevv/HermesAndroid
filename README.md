# HermesAndroid (fixed mirror, v1.0.1)

> **Credit / kredit:** karya asli oleh **TDat** di
> https://github.com/dat-hoangnguyentuandat/HermesAndroid — repo ini adalah
> salinan yang sama persis (plek ketiplek) dengan satu perbaikan, diterbitkan
> ulang oleh OmmniDevv supaya bisa di-flash dari repo ini. Lisensi tetap MIT
> atas nama TDat (lihat LICENSE). Jangan klaim karya asli ini sebagai buatan
> OmmniDevv; OmmniDevv hanya memperbaiki dan menerbitkan ulang salinan ini.

## Perbaikan v1.0.1 (satu-satunya perubahan dari v1.0.0 asli)

- **Error yang diperbaiki:** install gagal berulang di langkah
  `import hermes_cli, fastapi, openai, yaml` dengan
  `ModuleNotFoundError: No module named 'fastapi'`.
- **Penyebab:** upstream Hermes v0.22.5 hanya mendukung Python 3.14 —
  semua dependensi intinya di `pyproject.toml` ditandai
  `python_version >= '3.14'`. Module asli masih memakai CPython 3.12,
  jadi pip tidak menginstal apa pun kecuali paket Hermes itu sendiri.
- **Perbaikan:** runtime CPython dinaikkan ke **3.14.10**
  (python-build-standalone tag 20261026), termasuk fallback-nya.
  Install lama yang masih 3.12 tidak lagi dipertahankan — runtime-nya
  diganti 3.14 saat flash ulang. Semua file lain tidak diubah, kecuali
  `module.prop` (versi 1.0.1, versionCode 101, updateJson ke repo ini).

---

# HermesAndroid

Magisk/KernelSU module for rooted aarch64 Android. Runs upstream Hermes through
bundled glibc with CPython and Node.js. No Termux or proot required.

## Install

1. Download `HermesAndroid-v1.0.0.zip` from [Releases](https://github.com/dat-hoangnguyentuandat/HermesAndroid/releases/latest), or run `python build.py` from this repository to build it in the sibling `version/` directory.
2. Flash in Magisk/KernelSU with an Internet connection, then reboot.
3. Open the Action button or `http://127.0.0.1:9119/`.

The ZIP contains scripts and glibc, not Hermes source or UI. Installation downloads
CPython/Node, resolves GitHub's latest Hermes release to an exact commit, downloads
its source and builds dashboard/TUI on the phone using upstream's lockfile and build
scripts. Python dependencies are isolated per revision. No upstream source patches.

Allow several minutes, a stable connection and at least 3 GB free on `/data` for
builds, caches and rollback. A tested revision uses about 760 MB including source,
dependencies and generated assets; sizes vary upstream.

## Use and update

```sh
su -c 'hermes setup'
su -c 'hermes'
su -c 'hermes update'                  # download/build latest GitHub release
su -c 'hermes update --rollback'       # switch to previously active revision
su -c 'hermes update --ref main'       # explicitly opt into upstream main
su -c 'hermes update --build-only'     # prepare latest without switching
```

Normal Hermes releases no longer require repacking or reflashing the module.
The module's update wrapper builds source, UI and Python dependencies together.
The old release runs during the build; activation restarts previously running
services and checks dashboard health. Failed builds do not activate; failed
activation attempts to restore the previous release. An exclusive lock prevents
concurrent updates. Credentials, configuration and chat history are not replaced.

Use this shell command for updates. Upstream GUI update buttons or direct
`python -m hermes_cli.main update` do not invoke the module updater.

Rollback switches code/dependencies, not user data, and cannot reverse migrations
made by newer upstream Hermes. Old releases and npm's cache remain on disk.
Upgrading from a legacy installation keeps it available for the first rollback.

## Layout

```
/data/adb/hermes/
  bin/hermes             launcher / update entry point
  bin/python             bootstrap Python wrapper
  python/                base CPython
  node/                  Node.js and npm
  glibc/lib/             Linux loader and libraries
  releases/<commit>/     upstream source, built UI, isolated Python packages
  current -> releases/<commit>
  previous-release       rollback target (or legacy installation)
  update.py              module updater
  npm-cache/             reusable download cache
  home/.hermes/          user configuration, credentials and history
```

Ordinary uninstall removes runtimes/releases and preserves `home/`.

## Compatibility

Module-owned wrappers launch Python, Node and esbuild through glibc. Generated
npm command shims use Android's shell instead of `/usr/bin/env`. Python package
paths are isolated per revision. Android's CA store and a DNS-over-HTTPS proxy
provide TLS/DNS for the Linux runtimes.

This community runtime differs from upstream's documented Termux installation.
Future Python/Node minimums, workspace layout changes or new native dependencies
can still require a module update. Build failures leave the current release
available. Automatic updates on boot are not enabled.

Dashboard binds localhost. No systemd crash supervision: boot and the Action
button start services. See `/data/adb/hermes/install.log` for installation errors.

## Credits

- [Hermes Agent](https://github.com/NousResearch/hermes-agent), Nous Research.
- [python-build-standalone](https://github.com/astral-sh/python-build-standalone).
- GlibClaw's glibc-loader approach for rooted Android.

MIT © TDat
