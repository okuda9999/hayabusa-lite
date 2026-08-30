# Hayabusa Glow spike

This branch is an isolated technical spike based on Breez Glow. It is not a production release.

## Fixed inputs

- `glow-app`: `c5a88666db1e02fb6e7f339536eda4d5542fd391`
- `glow-web`: `fcaff4e71cd094b37aed41674f17519c5e72b494`
- Breez SDK - Spark: `0623571b57bf76996360800e58d691171475c0d4`
- Node.js: 22
- Validation bundle ID: `com.r-heit.hayabusa.glowspike`
- Validation URL scheme: `hayabusa-spike`
- Candidate Passkey RP: `keys.hayabusawallet.com`

The `upstream` remotes are fetch-only. The writable Hayabusa forks are:

- app `origin`: `https://github.com/okuda9999/hayabusa-lite.git`
- web `origin`: `https://github.com/okuda9999/hayabusa-lite-web.git`

The parent `.gitmodules` file points at the Hayabusa web fork. Breez remains available only through each repository's `upstream` remote.

## Reproduce the verified build

Use a checkout path without spaces. The upstream Spark SDK Makefile does not quote every path.

```sh
git submodule update --init --recursive
PATH=/opt/homebrew/opt/node@22/bin:$PATH npm ci
PATH=/opt/homebrew/opt/node@22/bin:$PATH npm --prefix glow-web ci
PATH=/opt/homebrew/opt/node@22/bin:$PATH npm --prefix glow-web test -- --run
PATH=/opt/homebrew/opt/node@22/bin:$PATH npm --prefix glow-web run build
make resolve-sdk
make sdk-ios
PATH=/opt/homebrew/opt/node@22/bin:$PATH npx cap sync ios
xcodebuild -project ios/App/App.xcodeproj -scheme App \
  -destination 'generic/platform=iOS' -configuration Debug \
  -derivedDataPath /tmp/hayabusa-spike-derived \
  CODE_SIGNING_ALLOWED=NO build
```

The verified `sdk-ios` target creates an iPhone device framework. It does not create the Simulator slice required for a Simulator build.

## Release blockers

- Do not use Breez's Passkey RP. The code points at the verified Hayabusa-owned RP `keys.hayabusawallet.com`.
- Do not put a Breez API key in source control. Supply a Hayabusa-owned key through local or CI secrets only.
- Do not switch to the existing production bundle ID or overwrite the existing Hayabusa TestFlight app.
- Do not treat the spike as production-ready. Separate-device recovery, payment-safety work, legal/App Store/security review, and deliberately small Mainnet interoperability testing remain mandatory.

## Current verification

- Glow Web: 34 test files and 218 tests passed.
- Glow Web production build: passed.
- Spark iOS device SDK and Swift bindings: built at the pinned commit.
- Capacitor iOS sync: passed with 17 plugins.
- Generic iPhone Debug build without signing: passed with the validation bundle ID.
- Apple Development signed build: passed with Team ID `C258H6NS96` and `webcredentials:keys.hayabusawallet.com`.
- Physical iPhone install and launch: passed on iPhone 13 Pro.
- Breez SDK initialization with the Git-ignored API-key configuration: passed.
- New wallet, receive QR, Passkey registration with Face ID, and same-device full restart: passed.
- No real Bitcoin has moved. Separate-device recovery and small-value Mainnet send/receive are not yet verified.

The technical-spike decision is **Go** for using Glow as Hayabusa's base. This is not a TestFlight or App Store release decision. The legacy SwiftUI Regtest wallet is retained as a source for safety tests; transferring from that wallet to this Mainnet-oriented fork is not an active release gate.

See the workspace ADRs and `docs/07_48時間スパイク実施記録.md` for decisions and open gates.
