# Shorts Off

A Firefox extension that removes YouTube Shorts from the home feed, subscriptions, search results, channel pages and the sidebar.

It is one stylesheet. There is no JavaScript, no background page, no settings storage and no permissions — the manifest declares a single content stylesheet and nothing else.

## Why CSS instead of a script

Most Shorts blockers poll the DOM with a `MutationObserver` and delete nodes after YouTube renders them. That costs CPU on every feed update and produces a visible flash of Shorts before they disappear.

The browser injects a declarative content stylesheet before the page paints, so Shorts are never rendered in the first place. The runtime cost is whatever the style engine spends matching a few dozen selectors — effectively nothing — and it keeps working while the tab is in the background without any script running.

The trade-off is that there is no on/off toggle: a settings UI would mean a script, storage and a popup. To customise it, edit `hide-shorts.css` and reload the extension. Each block is labelled, and the navigation block at the bottom can be deleted on its own if you want to keep the Shorts button in the sidebar.

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

Two rules depend on the interface language, because the elements give nothing else to grab: the channel **Shorts tab** and the **expanded sidebar's Shorts button**. That sidebar entry is the odd one out — unlike every other nav item it has no `href` at all, just a `title`, so the label is the only available hook. YouTube leaves "Shorts" untranslated in most locales the way it does with product names, but if yours localises it, change the two `"Shorts"` strings in `hide-shorts.css` to the label you see. Everything else is language-independent.

The collapsed sidebar button does carry a real link, so it is matched by href and works in every language.

## Install

**To try it out**, no signing needed — but it is removed when the browser restarts:

1. Open `about:debugging#/runtime/this-firefox`
2. *Load Temporary Add-on…*
3. Pick `manifest.json` in this folder

**To keep it installed**, release Firefox requires extensions to be signed, so pick one of:

- **Sign it yourself.** Submit the zip to [addons.mozilla.org](https://addons.mozilla.org/developers/) as an unlisted add-on. You get a signed `.xpi` back to install and keep, without publishing it.
- **Turn signing off.** Only works on Firefox Developer Edition, Nightly and ESR: set `xpinstall.signatures.required` to `false` in `about:config`.
- **Use a derivative that allows unsigned add-ons.** LibreWolf and Waterfox both do.

Before submitting to AMO, change the `id` under `browser_specific_settings.gecko` in `manifest.json` to a domain you control.

To build the zip:

```bash
zip -r -FS shorts-off.zip manifest.json hide-shorts.css icons
```

## When YouTube changes something

If Shorts reappear somewhere, find the element that wraps the Shorts link, and add it to the right block in `hide-shorts.css`. Right-click the Shorts card → *Inspect*, then walk up the tree to the outermost element that contains only that one card.

Keep `:has()` selectors out of the layer 1 rules. A single invalid selector invalidates an entire comma-separated list, so one `:has()` added to a layer 1 block would silently switch that whole block off on browsers that lack support.

## License

MIT — see [LICENSE](LICENSE).
