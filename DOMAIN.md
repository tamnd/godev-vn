# Buying godev.vn

This file is the note `SITE.md` points at. It is in English for the same reason
that one is: it is for whoever runs the deploy, not for readers.

`godev.vn` is the address this site should end up on. It is the Vietnamese
national domain, it says what the site is without a subdomain explaining it, and
it is the one thing in this project that cannot be done with a pull request.
Everything below is what buying it involves, gathered in September 2026. Prices
and registrar lists move, so check the VNNIC pages linked at the bottom before
paying anybody.

## Is it still free

As of 5 September 2026 `godev.vn` has no nameservers and no SOA record, so it is
not delegated:

    dig +short NS godev.vn
    dig +short SOA godev.vn

Both come back empty. That is consistent with the name being unregistered, and
it is not proof, because a registered name that nobody has pointed anywhere
looks exactly the same. There is no public WHOIS for `.vn` that answers over the
protocol; IANA has no referral server for the zone. The way to actually check is
the availability search on any accredited registrar, which queries VNNIC's
registry directly.

`.vn` is first come first served. VNNIC states plainly that domain registration
is not governed by the Law on Intellectual Property, so holding a trademark on a
name does not reserve it and does not get it back from somebody who registered
it first. If we want this name, the answer to "when" is now, not after the site
is finished.

## Who can register it

Foreign individuals and organisations can register `.vn` names. There is no
requirement to have a company, an office or a representative in Vietnam, which
is the thing most people assume and get wrong.

The registrar system has two halves and the address on the application decides
which one is available:

- Ten domestic registrars, accredited by VNNIC and based in Vietnam: Back Kim
  Network Solution, GMO RunSystem, PA Vietnam, Mat Bao, NhanHoa Software, iNET
  Software, Online Solution, Vinahost, Tino Group and Long Van System Solution.
- Six overseas registrars: Megazone, Hi-Tek, Instra, Web Commerce
  Communications, EuroDNS and InternetX.

An applicant with a Vietnamese address can use either half. An applicant with a
foreign address can use either half as well, but the overseas registrars exist
because they are set up for it and will not ask for paperwork that only makes
sense for a Vietnamese resident. Registering with a Vietnamese address through a
domestic registrar is the cheapest and the least friction, and it is the obvious
route for anyone who has one.

## What it costs

VNNIC's fees for an ordinary second level `.vn` name are set by Circular
20/2023/TT-BTC:

| what | amount |
| ---- | ------ |
| registration, once | 100,000 VND |
| maintenance, per year | 350,000 VND |

`godev.vn` is an ordinary second level name. It is six characters, so it is not
in the short name band that VNNIC prices separately, and it is not one of the
two character names VNNIC has been auctioning through 2026 with a 10,000,000 VND
starting price. None of that applies here.

On top of the VNNIC fees the registrar charges its own service fee, which is
where the price differences between them come from, and Vietnamese registrars
add 10% VAT. Budget somewhere around 700,000 to 900,000 VND for the first year
and around 400,000 to 600,000 VND a year after that. That is under 40 US dollars
to start and under 25 a year to keep, which is cheap enough that the argument
for buying it early is not really about money.

Registrars usually discount the first year and quietly charge full price on
renewal, so compare the renewal number and not the banner.

## What the paperwork is

`.vn` is not a name you buy by typing a card number. Every registration comes
with a Domain Name Registration Declaration, which the registrar generates and
the registrant signs, plus identity documents. For an individual that is a copy
of a passport or a national ID; for a company it is the business registration
certificate. Some registrars accept a scan and an electronic signature, some
still want paper.

The rule to actually remember: the supporting documents must reach the registrar
within seven days of registration. Miss it and the registrar locks the domain,
and there is no refund of what has already been paid. So do not register the
name on a day when the passport scan is not to hand.

Renewal is not automatic in the way `.com` renewal is. Pay the maintenance fee
before the anniversary. A `.vn` name that lapses goes back to first come first
served and, given the name, somebody will take it.

## What to set once it resolves

Cloudflare is where this site is served from, so the goal is Cloudflare running
DNS for `godev.vn`, the same as it already does for `tamnd.com`.

1. Add `godev.vn` as a zone in Cloudflare. Cloudflare gives back two
   nameservers.
2. Change the nameservers at the registrar to those two. Every accredited
   registrar supports this, though some want a signed form for it rather than a
   checkbox in a control panel, and it can take a business day. This is the step
   to ask about before choosing a registrar.
3. Add `godev.vn` as a custom domain on the Cloudflare Pages project, and
   `www.godev.vn` alongside it. Cloudflare creates the records itself once it
   runs the zone.
4. Leave the row in `SITE.md` as `placeholder`. From this point the name
   resolves and serves the placeholder page, which says what it is and carries a
   `noindex`, so nothing gets into a search index under an address that is not
   ready.
5. When the site is ready to move, change the `Canonical:` line in `SITE.md` to
   `godev.vn` and merge. That is the whole move, and `TestTheMove` in the
   translator keeps it true.

If the nameserver change turns out to be impossible at a given registrar, the
fallback is to use the registrar's own DNS and point the name at the Pages
project with a CNAME. That works for `www.godev.vn` and not for the apex,
because a CNAME cannot sit at the top of a zone unless the provider offers an
ALIAS or ANAME record. It is a worse arrangement and the reason to pick a
registrar that will move the nameservers instead.

## Sources

- [VNNIC, the `.vn` domain](https://www.vnnic.vn/en/domain-name-vn)
- [VNNIC, the registrar system](https://vnnic.vn/en/domain-name-vn/registras/registras-system)
- [VNNIC, fee schedule](https://vnnic.vn/en/domain-name-vn/domain-name/fee-schedule)
- [PA Vietnam, registrar and registrant agreement](https://www.pavietnam.vn/en/registrar-registrant-agreement-vnnic.html)
