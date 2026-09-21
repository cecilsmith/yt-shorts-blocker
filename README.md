# Shorts Off

A Firefox extension that removes YouTube Shorts from the home feed, subscriptions, search results, channel pages and the sidebar.

It is one stylesheet. There is no JavaScript, no background page, no settings storage and no permissions — the manifest declares a single content stylesheet and nothing else.

## Why CSS instead of a script

Most Shorts blockers poll the DOM with a `MutationObserver` and delete nodes after YouTube renders them. That costs CPU on every feed update and produces a visible flash of Shorts before they disappear.

The browser injects a declarative content stylesheet before the page paints, so Shorts are never rendered in the first place. The runtime cost is whatever the style engine spends matching a few dozen selectors — effectively nothing — and it keeps working while the tab is in the background without any script running.

The trade-off is that there is no on/off toggle: a settings UI would mean a script, storage and a popup. To customize it, edit `hide-shorts.css` and reload the extension. Each block is labelled, and the navigation block at the bottom can be deleted on its own if you want to keep the Shorts button in the sidebar.

## What it hides

| Surface | How it is matched |
| --- | --- |
| Shorts shelves in home, subscriptions and search | Shorts-only elements, by tag name |
| Shorts cards mixed into feeds and search results | Any card wrapping a `/shorts/` link |
| Watch-page sidebar suggestions | Any card wrapping a `/shorts/` link |
| Shorts tab on channel pages | `tab-title="Shorts"` |
| Shorts button in the collapsed sidebar and mobile nav bar | Nav entry linking to `/shorts` |
| Shorts button in the expanded sidebar | Entry labelled `Shorts` (see below) |

The rules are written in two layers. Layer 1 matches elements that only ever contain Shorts, by tag name. Layer 2 matches a card or shelf by the `/shorts/` link inside it, using `:has()`.

Anchoring on the link rather than on the element name is deliberate: YouTube renames its custom elements often, and an href-based rule survives that. Every layer 2 selector targets a single card or a single shelf section, never a wrapper that also holds regular videos — that distinction is what keeps a stray Shorts link from blanking an entire page of results.

## Browser support

Layer 1 works on Firefox 109+. Layer 2 needs `:has()`, which landed in **Firefox 121** (December 2023); on older builds those rules are ignored and the rest still applies.

Works the same on Firefox derivatives — LibreWolf, Waterfox, Zen, Floorp, Mullvad Browser — and on Firefox for Android 120+, including Mull and IronFox. The stylesheet covers `m.youtube.com` as well as the desktop site.

Two rules depend on the interface language, because the elements give nothing else to grab: the channel **Shorts tab** and the **expanded sidebar's Shorts button**. That sidebar entry is the odd one out — unlike every other nav item it has no `href` at all, just a `title`, so the label is the only available hook. YouTube leaves "Shorts" untranslated in most locales the way it does with product names, but if yours localizes it, change the two `"Shorts"` strings in `hide-shorts.css` to the label you see. Everything else is language-independent.

The collapsed sidebar button does carry a real link, so it is matched by href and works in every language.

## Install

**To try it out**, no signing needed — but it is removed when the browser restarts:

1. Open `about:debugging#/runtime/this-firefox`
2. *Load Temporary Add-on…*
3. Pick `manifest.json` in this folder

**To keep it installed**, release Firefox requires extensions to be signed, so pick one of:

- **Install a signed build from Releases** — see below. Recommended: it auto-updates.
- **Turn signing off.** Only works on Firefox Developer Edition, Nightly and ESR: set `xpinstall.signatures.required` to `false` in `about:config`.
- **Use a derivative that allows unsigned add-ons.** LibreWolf and Waterfox both do.

## Automatic updates

Install the `.xpi` from the [latest release](https://github.com/cecilsmith/yt-shorts-blocker/releases/latest) once, and Firefox keeps it up to date from this repo on its own — it checks roughly once a day.

Two things make that work, and they are easy to confuse:

- `browser_specific_settings.gecko.id` is only a **name**. It looks like an address but is never fetched, and pointing it at a domain does nothing beyond avoiding collisions with other add-ons. Once people have installed the extension, **never change it** — Firefox treats a new id as a different add-on, and everyone would have to reinstall by hand.
- `browser_specific_settings.gecko.update_url` is what actually drives updates. It points at `updates.json` on the latest GitHub release, which lists the current version, the `.xpi` to fetch and a SHA-256 that Firefox verifies before applying.

The manifest also declares `data_collection_permissions: { required: ["none"] }`. AMO has required that field on new submissions since 3 November 2025, and `none` is the honest answer here — the add-on is a stylesheet, with no scripts, no network access and no storage, so there is nothing to collect. Firefox 140+ surfaces this on the install prompt.

**Signing is not optional.** Firefox will not install an unsigned `.xpi`, even one you host yourself, so every release goes to Mozilla to be signed. Submitting to the **unlisted** channel avoids a public listing and human review — it is an automated validation pass, usually a minute or two — and hands back a signed file you host here.

### Cutting a release

One-time setup: generate API credentials at [AMO → Manage API Keys](https://addons.mozilla.org/developers/addon/api/key/), then add them to this repo under *Settings → Secrets and variables → Actions* as `AMO_JWT_ISSUER` and `AMO_JWT_SECRET`.

After that, each release is a version bump and a tag:

```bash
V=1.0.1 && \
  node -e 'const f="manifest.json",m=require("./"+f);m.version=process.argv[1];require("fs").writeFileSync(f,JSON.stringify(m,null,2)+"\n")' "$V" && \
  git commit -am "v$V" && git tag "v$V" && git push origin main "v$V"
```

[`.github/workflows/release.yml`](.github/workflows/release.yml) then checks the tag against `manifest.json`, lints, signs the add-on, generates `updates.json`, and publishes both as release assets. Installed copies pick the new version up on their next check.

The tag must match the version in `manifest.json`, and AMO refuses a version it has already seen — so every release needs a fresh number. The workflow fails early on a mismatch rather than publishing a broken update manifest.

Version notes for each release live in [`CHANGELOG.md`](CHANGELOG.md), and
[`.github/AMO_REVIEWER_NOTES.md`](.github/AMO_REVIEWER_NOTES.md) holds the reviewer notes — including the fact that **no account is needed to test this add-on**, since YouTube serves Shorts to signed-out visitors.

### Checking the manifest locally

```bash
npx --yes web-ext@latest lint --self-hosted --source-dir=. --ignore-files ".github/**" "dist/**" "*.md"
```

`--self-hosted` matters: without it the linter reports `update_url` as an error, because that restriction applies only to add-ons hosted **on** addons.mozilla.org. This one is signed there but hosted here, so the field is allowed. The same flag is used in CI.

Two warnings are expected and harmless — `strict_min_version` is below the versions that introduced `data_collection_permissions` (Firefox 140, Android 142). Older releases simply ignore the key.

### Building the zip by hand

```bash
zip -r -FS shorts-off.zip manifest.json hide-shorts.css icons
```

## When YouTube changes something

If Shorts reappear somewhere, find the element that wraps the Shorts link, and add it to the right block in `hide-shorts.css`. Right-click the Shorts card → *Inspect*, then walk up the tree to the outermost element that contains only that one card.

Keep `:has()` selectors out of the layer 1 rules. A single invalid selector invalidates an entire comma-separated list, so one `:has()` added to a layer 1 block would silently switch that whole block off on browsers that lack support.

## License

MIT — see [LICENSE](LICENSE).
