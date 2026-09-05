# Địa chỉ của trang

This file is in English because it is operational and not content. Everything
under `_content_vi` is for readers; this is for whoever deploys.

It holds every hostname the published site answers on and says which one of them
is the real address. `godev publish` in tamnd/godev-vn-translator reads it and
writes `_redirects` and the canonical link tags from what it finds here, so this
is the file to edit and not the exporter.

## The setting

    Canonical: godev-vn.tamnd.com

That line is the whole thing. It is what canonical link tags, the feed and every
absolute reference in the exported HTML point at, and it is what every other
host in the table below is arranged around. Moving the site to a new domain is
changing that one line and nothing else.

## The hosts

| host | role | note |
| ---- | ---- | ---- |
| godev-vn.tamnd.com | canonical | The address today. `tamnd.com` is already on Cloudflare nameservers, so this is one CNAME to the Pages project and no delegation change. |
| godev-vn.pages.dev | redirect | The name Cloudflare gives the Pages project. It answers whether we want it to or not, so it sends readers to the canonical host rather than putting a second copy of the site in a search result. |
| godev.vn | placeholder | The official Vietnamese address, not bought yet. See `DOMAIN.md` for what buying it involves. Listed here from now so that the day it resolves it serves a page explaining itself instead of sitting parked. |

Three roles, and a host whose role is not one of them is treated as a redirect,
which is the safe reading for a host somebody added and did not classify.

**canonical** is ignored as a role. Which host is canonical is the line above,
and that is deliberate: if the role column decided it, moving the site would
mean editing two cells and remembering both.

**redirect** sends a reader to the canonical host with a 301, for every path.

**placeholder** serves `placeholder.html` for every path, with a 200 and a
`noindex`. A domain that is bought and not moved to yet should say what it is to
whoever types it. That is not the same as being parked, and it is not the same
as pretending to be the site.

**mirror** serves the same files with no redirect. That is for the GitHub Pages
deploy, which exists so that one vendor having a bad day is not the whole site
being down. A mirror does not compete with the canonical host in a search index,
because the canonical link tag in every page it serves names the canonical host.

## Moving to godev.vn

When the domain is bought and pointed at the Pages project:

1. Change the `Canonical:` line to `godev.vn`.
2. Merge.

That is the move. The deploy that follows rewrites every canonical tag and every
absolute reference to `godev.vn`, starts serving `godev-vn.tamnd.com` as a 301 to
it, and stops serving the placeholder, because a placeholder role on the
canonical host means nothing. The table does not need touching, and neither does
any code.

There is a test for exactly this in the translator, `TestTheMove` in `site/`, so
that the claim in the paragraph above stays true.
