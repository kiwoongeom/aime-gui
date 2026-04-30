# Aime GUI Wallet Patches

Patches against upstream monero-gui (monero-project/monero-gui).

| Patch | Description |
|---|---|
| 0001-Aime-GUI-patches.patch | Data dir, log path, daemon binary name (`monerod` → `aimed`) |
| 0002-Phase-11-QML-user-text-rebrand.patch | QML files: user-visible "Monero" → "Aime" |

Apply: `git am AIME_PATCHES/*.patch` against monero-gui master.
