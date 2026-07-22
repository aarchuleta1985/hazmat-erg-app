# Hazmat + ERG Field Reference — Windows desktop wrapper

This folder wraps `Hazmat Plus ERG 2024 - Offline Field Reference.html` in a small
[Tauri](https://tauri.app) desktop app, so it can be submitted to the Microsoft Store
through Partner Center as an installable Windows app instead of a file people have to
double-click open in a browser.

The web app itself is untouched — it's just sitting at `dist/index.html` and is loaded
into a native window. Nothing about it needs the internet; the desktop wrapper doesn't
add any network access either.

## Before you build anything

Open `src-tauri/tauri.conf.json` and change two placeholder values:

- `"identifier"` — currently `com.hazmaterg.fieldreference`. Change the domain-style
  prefix to something tied to you/your company (e.g. `com.yourcompany.hazmaterg`).
  This must stay the same for the life of the app once you submit it.
- `"bundle.publisher"` — currently `"Change Me LLC"`. Set this to your actual publisher
  name. It must be different from the `productName` (`Hazmat ERG Field Reference`) —
  Microsoft Store rejects submissions where they match.

The icon is a generic placeholder (orange hazmat-diamond + document glyph) in
`src-tauri/icons/`. Swap in your own artwork any time by replacing those PNG/ICO files,
or regenerate the whole set from one image with `npx @tauri-apps/cli icon path/to/logo.png`
if you install the Tauri CLI locally.

## Update: if you reserved the app as "MSIX or PWA app"

If Partner Center's Packages page only accepts `.msix`/`.msixbundle`/`.appx`-family
files, you reserved the app under the **"MSIX or PWA app"** product type, not "EXE or
MSI app." Tauri doesn't generate MSIX directly (it produces a normal `.exe`/`.msi`), so
that installer can't be dropped on that page as-is — it needs one extra step: wrapping
it into a real MSIX with Microsoft's free **MSIX Packaging Tool**. See "Submitting to
Partner Center (MSIX or PWA app)" below. The good news: for this product type,
**Microsoft automatically re-signs your package after certification, so you don't need
to buy a code-signing certificate at all.**

If your Packages page instead asks for an installer URL (no drag-and-drop), you're on
the "EXE or MSI app" type — use the older "Submitting to Partner Center (EXE or MSI
app)" section further down instead, which does need your own code-signing certificate.

## Building it

**Option A — GitHub Actions (recommended, no Windows PC needed).**
Push this folder to a GitHub repo, then go to the Actions tab and run the
"Build Windows installer" workflow (or push a tag like `v1.0.0`, which also creates a
GitHub Release with the installers attached — that Release asset URL is what you'll
paste into Partner Center). It builds two variants:
- `artifacts/general/` — the everyday installer, downloads WebView2 if needed.
- `artifacts/msstore/` — the Store-specific build, with WebView2 bundled in offline
  (the Store requires this variant; see `tauri.microsoftstore.conf.json`).

**Option B — your own Windows machine.** Install
[Rust](https://rustup.rs) and Node.js, then from this folder:

```
npm install
npm run build:win
```

The installer(s) land in `src-tauri/target/release/bundle/nsis/` and `/msi/`.

## Submitting to Partner Center (MSIX or PWA app) — the path that matches a
## drag-and-drop Packages page

1. **Build the installer** using the GitHub Actions workflow above. Use the
   `artifacts/msstore/` variant (WebView2 bundled in, so wrapping it doesn't depend on
   internet access) — grab the `.exe` from its `nsis` folder.
2. **Look up your exact app identity.** Partner Center dashboard → your app → Product
   management → **Product identity**. Copy the three values shown: `Package/Identity/Name`,
   `Package/Identity/Publisher`, and `Package/Properties/PublisherDisplayName`. These
   have to match *exactly* in the MSIX you upload, or Partner Center rejects it.
3. **Install the MSIX Packaging Tool** — it's free, from the Microsoft Store (search
   "MSIX Packaging Tool"). Needs a Windows machine; this is the tool that wraps an
   existing installer into a real MSIX without you writing any manifest XML by hand.
4. **Create the package:** Application package → "Create package on this computer" →
   point it at the `.exe` from step 1 → click through the installer once (it's silent,
   so this should just finish on its own) → on the "Package information" screen, paste
   in the three identity values from step 2 (this is the part that must match).
5. **App visual assets:** the tool will ask for logo images at several sizes — use the
   files already in `src-tauri/icons/` (`Square44x44Logo.png`, `Square150x150Logo.png`,
   `StoreLogo.png`, `Square310x310Logo.png`, etc.) rather than generating new ones.
6. **Save/finish** — output as `.msix`. Drag that file onto Partner Center's Packages
   page. No code-signing certificate needed; Microsoft re-signs it after certification.
7. Fill in the rest of the listing (description, screenshots, age rating — this is a
   reference tool, not medical/dispatch software) and submit. Certification is
   typically 24–48 hours.

## Submitting to Partner Center (EXE or MSI app) — only if your Packages page instead
## asked for an installer URL

1. **Reserve the app name.** Partner Center dashboard → Apps and games → New product →
   **EXE or MSI app**. Reserve the name you want (this can differ from the internal
   `productName`).
2. **Code sign the installer.** The Store requires Win32 installers to be signed; an
   unsigned `.exe`/`.msi` will be rejected. If you don't have a certificate yet, the
   simplest current option is **Azure Trusted Signing** (a few dollars/month, no
   hardware token, integrates with CI) — or a traditional OV/EV code-signing
   certificate from any CA. Once you have one, wire it into the GitHub Actions workflow
   using the commented-out steps in `.github/workflows/build-windows.yml`.
3. **Host the signed installer** somewhere with a stable public URL — a GitHub Release
   (from the Actions workflow) works well for this.
4. **Fill in the installer details in Partner Center:**
   - Installer URL → your hosted `.exe`/`.msi` link (use the `msstore` variant, the one
     with the offline WebView2 installer baked in).
   - Silent install switch → `/S` (capital S) if you're using the NSIS `.exe`, or the
     standard `/quiet` if you're using the `.msi`. The Store requires silent install to
     work, or your submission gets rejected with a "Win32 products must install silently"
     error.
   - Since the wrapper only launches a local file — no telemetry, no network calls —
     the privacy/data-collection questions in the listing should all be "no."
5. **Store listing.** Placeholder listing icons (300×300 and 1080×1080 PNG) are in
   `../store-assets/`. Add a short description, a couple of screenshots of the app
   (launch it and grab a few), and pick an age rating (this is a reference/informational
   tool, not medical or emergency-dispatch software — the disclaimer already built into
   the cover screen should be quoted or referenced in your listing description so buyers
   understand it's a reference aid, not an official regulatory source).
6. **Submit.** Certification is typically 24–48 hours.

## Reminder about the content itself

The app's cover screen already discloses this, but worth repeating for your listing
copy: this tool reproduces 49 CFR Title 49 Parts 100–177 (revised as of October 1, 2024)
and the DOT/PHMSA Emergency Response Guidebook 2024. It's a reference aid, not a
replacement for the official CFR, the printed ERG, or hazmat training — Store reviewers
sometimes ask about this for anything touching safety/compliance content, so having that
disclaimer visible up front is worth keeping.
