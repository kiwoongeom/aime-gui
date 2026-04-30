# Aime GUI Wallet

> Qt-based graphical wallet for Aime, forked from [monero-project/monero-gui](https://github.com/monero-project/monero-gui).
> Bundles patched Aime source as the libwallet backend.

**Maintained by:** Kiwoong Eom (eric.eom@gmail.com) — GitHub [@kiwoongeom](https://github.com/kiwoongeom)
**Companion project:** [aime-core](https://github.com/kiwoongeom/aime-core) (the daemon)

---

> 🚀 **New here?** Start with the [Aime Getting Started Guide](https://github.com/kiwoongeom/aime-core/blob/main/GETTING_STARTED.md) — step-by-step instructions for the whole project (mining, node, wallet, all 4 components).
## Features

- Create / restore wallets (25-word seed)
- Send / receive AIME with full privacy (RingCT, ring size 16)
- View transaction history
- Built-in mining controls
- Multi-language support (translations from Monero ecosystem)
- Address book
- Hardware wallet support (Ledger, Trezor — via Monero compatibility)
- Daemon manager (auto-start/stop aimed)

---

## Build

### Prerequisites

```bash
sudo apt-get install -y \
  qttools5-dev qttools5-dev-tools qtdeclarative5-dev libqt5svg5-dev \
  libqt5x11extras5-dev libqt5quick5 libgcrypt20-dev \
  qml-module-qtquick-controls qml-module-qtquick-controls2 \
  qml-module-qtquick-dialogs qml-module-qtquick2 qml-module-qtgraphicaleffects \
  qml-module-qt-labs-settings qml-module-qt-labs-platform \
  qml-module-qt-labs-folderlistmodel qml-module-qtmultimedia \
  libqt5multimedia5-plugins qml-module-qtquick-templates2 \
  qml-module-qtquick-extras qml-module-qtquick-window2 \
  qml-module-qtquick-xmllistmodel qml-module-qtquick-shapes \
  qml-module-qtquick-virtualkeyboard qml-module-qtquick-particles2 \
  qml-module-qtquick-localstorage qml-module-qt-labs-qmlmodels
```

Plus all the Aime daemon build dependencies (this fork bundles libwallet).

### Compile

```bash
git clone <aime-gui-repo> aime-gui
cd aime-gui
MANUAL_SUBMODULES=ON make
# → ~15-30 minutes (Qt + libwallet + GUI)
```

Output: `build/bin/monero-wallet-gui` (~27 MB)

---

## Run

### Basic launch

```bash
DISPLAY=:0 ./build/bin/monero-wallet-gui
```

Under WSLg (Windows 11), `:0` works automatically.
On native Linux, also works with X11 / Wayland.

### Connect to local daemon

If you have aimed running on `127.0.0.1:17081`, the GUI auto-connects.

To connect to a remote daemon:
1. Launch GUI
2. Settings → Node → "Configure"
3. Enter: `<NODE_IP>:17081`

### CLI options

```
--verify-update          Verify update binary signatures
--disable-check-updates  No update checks
--socks5-proxy <addr>    Use SOCKS5 proxy
--log-file <file>        Custom log location
```

---

## Aime-Specific Patches

This fork modifies upstream monero-gui in 4 ways:

1. **Bundled `monero/` submodule replaced** with Aime source — gives libwallet our NETWORK_ID, address prefix 56, custom genesis
2. **`src/main/Logger.cpp`** — `.bitmonero` → `.aime` (data dir for desktop integration)
3. **`src/libwalletqt/Wallet.cpp`** — `bitmonero.log` → `aime.log`
4. **`src/daemon/DaemonManager.cpp`** — `monerod` / `monerod.exe` → `aimed` / `aimed.exe` (binary path + process management)
5. **QML files** — replaced "Monero" with "Aime" in user-visible text (window title, wizard screens, settings dialogs) while preserving namespace identifiers (MoneroComponents, MoneroEffects)

The upstream Monero GUI repo can be merged with care — most rebrand changes are isolated.

---

## Architecture

```
┌─────────────────────────────────────┐
│       monero-wallet-gui (Qt/QML)    │
│  ┌────────────────────────────────┐ │
│  │  Wallet UI (pages/, wizard/)   │ │
│  │  Settings, Transfer, etc.      │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │  libwalletqt (Qt/C++ bridge)   │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │  libwallet (from Aime source)  │ │
│  │  - Address validation (Aime)   │ │
│  │  - RPC client                  │ │
│  │  - Transaction construction    │ │
│  └────────────────────────────────┘ │
└─────────────────────────────────────┘
            │
            ▼ RPC (port 17081)
   ┌──────────────────┐
   │  aimed daemon    │
   └──────────────────┘
```

The GUI doesn't need its own copy of aimed — but for "embedded daemon mode", it can launch aimed automatically.

---

## Troubleshooting

### "QQmlApplicationEngine failed to load component"
Missing QML modules. Re-run apt-get install command above.

### Window appears but blank
- Qt Quick Layouts warnings can usually be ignored (cosmetic).
- For severe issues, set `QT_DEBUG_PLUGINS=1` and check logs.

### "Module 'XXX' not installed"
Install corresponding `qml-module-*` package.

### Daemon not connecting
- Check `aimed` is running: `pgrep aimed`
- Check port: `netstat -tnlp | grep 17081`
- Try restart from GUI's Settings → Node

### Wallet won't open
- Check password
- Check wallet file permissions
- Use CLI to verify: `aime-wallet-cli --wallet-file mywallet`

---

## Building for Other Platforms

### Windows (cross-compile from Linux)

Use `make depends` like Monero. See Monero GUI's official docs.

### macOS

```bash
brew install qt@5 boost openssl libsodium
MANUAL_SUBMODULES=ON make
```

(Untested on Aime — should work if Monero GUI does.)

### Android

Significant effort, see Monero's Dockerfile.android. Would need similar adaptation for Aime.

---

## Distribution Packaging

For releasing GUI to end users:

### Linux AppImage

```bash
cd build
./linuxdeploy --appdir AppDir --executable bin/monero-wallet-gui --plugin qt
./linuxdeploy --appdir AppDir --output appimage
# Output: AimeWallet-x86_64.AppImage
```

### Code signing (recommended)

Use Authenticode (Windows), codesign (macOS), GPG (Linux) to sign release binaries. This prevents user warnings and proves authenticity.

---

## License

GPL v3 (inherited from upstream monero-gui).

---

## Acknowledgments

- The Monero GUI maintainers — for years of UI/UX refinement
- Localization Workgroup — for translations (still attributed)
