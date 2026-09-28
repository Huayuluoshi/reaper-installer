# reaper-installer

一个 REAPER Linux 安装器的 Debian 虚包。包本身不含 REAPER 二进制，`postinst`
会在安装时从 [reaper.fm](https://www.reaper.fm/download.php) 下载对应架构的官方
tarball，并运行官方 `install-reaper.sh`。

## 支持的架构

- amd64  ↔  x86_64
- i386   ↔  i686
- arm64  ↔  aarch64
- armhf  ↔  armv7l

GitHub Actions 每天 03:00 UTC 检查一次 REAPER 版本，页面出现哪些架构就构建哪些
deb，并发布到 Releases。

## 手动构建

```bash
cd reaper-installer

# 写入版本号
VERSION=7.80
sed -i "s|^Version: .*|Version: ${VERSION}|" DEBIAN/control
sed -i "s|__REAPER_VERSION__|${VERSION}|" DEBIAN/postinst

# 为每个架构各构建一份
for ARCH in amd64 arm64 i386 armhf; do
  BUILD_DIR="build-${ARCH}"
  rm -rf "${BUILD_DIR}"
  cp -a reaper-installer "${BUILD_DIR}"
  sed -i "s|^Architecture: .*|Architecture: ${ARCH}|" "${BUILD_DIR}/DEBIAN/control"
  chmod 755 "${BUILD_DIR}/DEBIAN/postinst" \
            "${BUILD_DIR}/DEBIAN/prerm" \
            "${BUILD_DIR}/DEBIAN/postrm"
  chmod 644 "${BUILD_DIR}/DEBIAN/control"
  dpkg-deb --build --root-owner-group "${BUILD_DIR}" \
    "reaper-installer_${VERSION}_${ARCH}.deb"
done
```

## 安装

```bash
sudo apt install ./reaper-installer_7.80_amd64.deb
```

安装过程中会：

1. 从 reaper.fm 下载 `reaper780_linux_x86_64.tar.xz`。
2. 解压到临时目录。
3. 运行官方 `install-reaper.sh`，你可以在终端里选择安装路径、桌面集成等。

## 卸载

```bash
sudo apt remove reaper-installer
# 或
sudo apt autopurge reaper-installer
```

卸载时：

- 若 REAPER 在 `/opt/REAPER` 或 `~/opt/REAPER`，直接清理。
- 若不在默认位置，会询问你 REAPER 的实际安装目录，并二次确认后删除。
- `~/.config/REAPER` 不会被删除，保留用户配置。
