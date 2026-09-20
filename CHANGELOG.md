# Changelog

## 1.0.0

First release.

Hides YouTube Shorts across the site:

- Shorts shelves in the home feed and subscriptions
- Individual Shorts mixed into the feed, search results and watch-page suggestions
- The Shorts button in both the expanded and collapsed sidebars
- The Shorts tab on channel pages
- The Shorts entry in the mobile navigation bar

The add-on is a single stylesheet. It runs no JavaScript, requests no
permissions, makes no network requests and stores nothing. Because the
browser applies the stylesheet before the page paints, Shorts are never
rendered rather than being removed after the fact, so there is no flicker
and no ongoing CPU cost.

Requires Firefox 121 or later for full coverage. On Firefox 109-120 the
shelf rules still apply, but rules that rely on `:has()` do not.
