# Notes to Reviewer

Paste the section below into the "Notes to Reviewer" field if you submit
through the AMO website. Tagging a release signs through the API instead
and never shows that form, but unlisted add-ons are occasionally pulled for
manual review, so this is kept here and kept accurate.

---

No account is required to test this add-on. YouTube serves Shorts to
signed-out visitors, so a fresh profile is enough to see it work.

**To verify:**

1. Install the add-on and open `https://www.youtube.com` signed out.
2. The "Shorts" button is absent from the left sidebar, in both its
   expanded and collapsed states.
3. Scroll the home feed. No Shorts shelves appear; regular videos are
   untouched.
4. Open any channel page, for example `https://www.youtube.com/@MrBeast`.
   The "Shorts" tab is absent from the tab row; Home, Videos, Shows and
   Posts remain.

To see the difference, disable the add-on and reload — the Shorts shelf
and sidebar button reappear.

**Implementation:**

The add-on is one static stylesheet, `hide-shorts.css`, injected as a
declarative content script on `*://*.youtube.com/*`. It sets
`display: none` on the elements that hold Shorts.

- No JavaScript of any kind. The manifest declares one `content_scripts`
  entry, but it carries only a `css` array and no `js` — the package
  contains no `.js` file at all. There is no background page and no
  event page.
- No permissions and no host permissions are requested.
- No network requests, no storage, no cookies, no telemetry. Declared as
  `data_collection_permissions: { required: ["none"] }`.
- No build step, no bundler, no minification and no transpilation. The
  four files in the package — `manifest.json`, `hide-shorts.css`,
  `icons/icon.svg` and `LICENSE` — are the complete, readable source,
  byte-for-byte identical to the repository. Nothing is generated,
  combined or templated.
- `web-ext` is used in CI, but only to zip the directory and submit it for
  signing. It does not transform, rewrite or generate any file.
- CI also produces an `updates.json` for self-distribution. It is published
  as a release asset and is **not** part of the extension package.

The stylesheet is organised in three commented layers: elements matched by
tag name, containers matched by the `/shorts/` link they wrap using
`:has()`, and the navigation entries. Selectors deliberately target
individual cards and dedicated Shorts shelves only, never a wrapper that
also holds regular videos.

Two selectors match on the visible label `Shorts` (the channel tab and the
expanded sidebar entry) because those elements expose no stable
language-independent attribute — the sidebar entry has no `href` at all.

**On `update_url`:** the add-on is self-distributed on the unlisted
channel. `update_url` points to an update manifest published on the
project's GitHub releases, which is permitted for add-ons signed on this
channel.

Source: https://github.com/cecilsmith/yt-shorts-blocker
