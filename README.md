# cloud-itonami-iso3166-jpn-meti

Open ISO 3166 Agency Blueprint for **JPN-METI**: Ministry of Economy, Trade and Industry
(経済産業省, METI) — a Japan-agency-level LEAF under
the `cloud-itonami-iso3166-jpn` country-level coordinator.

This repository designs a forkable OSS business for an independent
compliance consultant: an already-incorporated operator (typically one
already using `cloud-itonami-iso3166-jpn` for general Japan market entry)
gets a Compliance Advisor + independent **Industrial-Policy Compliance Governor** to
navigate export-control classification of controlled goods/technology under the Foreign Exchange and Foreign Trade Act (外国為替及び外国貿易法, commonly "外為法"/FEFTA), and navigation of METI industrial-policy subsidy/grant programs (e.g. GX/DX-related schemes) that a public-sector delivery may rely on.

## No robotics premise — digital/data service exemption

Agency-specific compliance navigation is a pure data/software service with
no physical-domain work — the same exemption class as `cloud-itonami-6310`
and `cloud-itonami-gtin-*`. `blueprint.edn` sets
`:itonami.blueprint/robotics false` and `:required-technologies` lists only
real capabilities (`:identity`, `:forms`, `:dmn`, `:bpmn`, `:audit-ledger`),
no `:robotics`.

## Core Contract

```text
operator intake + prior filing/compliance history
        |
        v
Compliance Advisor -> Industrial-Policy Compliance Governor -> compliance draft, or human sign-off
        |
        v
gated filing / registration / compliance-program submission + audit ledger
```

No automated proposal can submit a filing or registration the governor
refuses, suppress a compliance record, or claim a legal conclusion the
governor has not cleared. `:filing/submit` is never in any phase's `:auto`
set — it always requires human sign-off (mirrors `cloud-itonami-M6910`'s
`filing-submit-never-auto-at-any-phase` invariant).

## What this is NOT

- **Not Ministry of Economy, Trade and Industry (経済産業省) itself, and not the
  government of Japan.** See [`docs/business-model.md`](docs/business-model.md)
  for the boundary with `com-etzhayyim-ooyake`, `matsurigoto`,
  `com-etzhayyim-toritsugi`, `legal-entity.etzhayyim.com`,
  `cloud-itonami-M6910`, and the country-level `cloud-itonami-iso3166-jpn`.
- **Not legal or tax advice.** Every regulatory claim must cite the
  official METI source and route final filings to
  Japan-licensed counsel or a registered agent where the law requires
  licensed representation.

## Capability layer

Resolves via [`kotoba-lang/iso3166`](https://github.com/kotoba-lang/iso3166)
(code `JPN-METI`, `:parent "JPN"`, cross-referenced to ooyake's
`gov.jpn.meti`). Required capabilities:

- :identity
- :forms
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## Regulatory sources

`facts.edn` is the register every regulatory claim in this repository has to
cite. It is tx-data, so it loads the same way as every other EDN corpus here:

```clojure
(d/transact conn (edn/read-string (slurp "facts.edn")))
```

`nbb scripts/verify-facts.cljs` re-fetches all of it against the live
authority and exits `0` verified / `1` the register is wrong / `2` the run
could not answer. The third code carries most of the weight on these hosts,
for the reason below.

**A finding outranks a blockage.** If one source was read and found wrong while
another was blocked, the run exits `1`, not `2` — a page that could not be
reached says nothing about a different page that *was* reached, decoded and
read, so it cannot unmake that finding. The blocked ones are still counted and
still printed, and the exit-`1` message says the failure count is a *floor*, so
it is never read as "everything else verified". Exit `2` is reserved for a run
that established nothing. Getting this backwards is how one intermittently
challenged page hides a real register error for as long as the block lasts;
it did exactly that on 2026-08-29, on a run that had already printed
`FAIL[charset-drift]`.

Every way a run can fail to reach something is emitted as a `BLOCKED\t` token
naming its kind, so a harness reading this file can tell "blocked" from "the
register is wrong" without grepping English.

Each entry names *which* check establishes it, because they are not
interchangeable:

| `:source/verify` | What it establishes |
|---|---|
| `:e-gov-law-id` | The id resolves through the e-Gov law API to the recorded title **and** law number. Status is never consulted: `laws.e-gov.go.jp` answers 200 for `/law/<anything>`. |
| `:page-identity` | 2xx, **and** the final URL is still the page asked for, **and** the declared charset, **and** the recorded `<title>`. |
| `:page-text` | All of the above, plus every string in `:page/must-contain`. |

### The bot challenge is a 2xx

`www.meti.go.jp` is behind AWS WAF. To an automated client it serves, instead
of the page, a 2468-byte JavaScript challenge — and that response answers
**HTTP 202**, at the requested URL, with no redirect and an empty `<title>`:

| Check | Sees | Verdict |
|---|---|---|
| status | `202` is 2xx | pass |
| final URL | served where it was asked for | pass |
| charset | declared utf-8, served utf-8 | pass |
| `<title>` | `""` vs the expected title | **mismatch** |

A verifier that stops there calls this *title drift* and exits `1` — reporting
"the register is wrong" about a run that never reached the authority. The
register is not wrong; the run could not answer. So the challenge is detected
by its own signature **before** the status branch and reported as
`:challenge-interposed` with exit `2`.

This is the defect class this workspace keeps finding, running the other way:
not a check that could not run returning the value of one that ran clean, but
a check that could not run returning the value of one that ran and *failed* —
which sends whoever reads it to edit a register that was correct.

### What that costs this register

**No page on `www.meti.go.jp` is cited.** A cold client reads real pages, but
roughly ten requests trips the challenge, it then holds for about four and a
half minutes, and single requests fourteen seconds apart re-trip it. The
ministry's own site could not be read reliably enough to cite from here — the
pages exist and a browser reads them, so they are *not-cited, not absent*, and
`:coverage/not-covered` says so. One consequence is recorded there rather than
papered over: **nothing in `organization.edn` is confirmed by this register**,
because the page carrying METI's address is on that host.

The pages that *are* cited are on the three extra-ministerial bureau hosts
that answer — 特許庁, 中小企業庁 and 資源エネルギー庁.

### Two more things measured rather than assumed

- **All four hosts answer `403`, not `404`, for a path that does not exist**,
  each with its own real Japanese "this page does not exist" body at the
  requested URL. 403 is not 2xx so status still discriminates — but a checker
  looking for 404 specifically, or one reading 403 as "blocked, cannot
  measure", would misreport four hosts that answered perfectly clearly. The
  status each host returned is pinned in `:host/missing-status` and compared
  on every run.
- **Fetches are serial and paced, and that is a measurement, not a style.**
  The sibling MLIT verifier fetches every page concurrently; doing that here
  is what *trips* the challenge. An early revision of this verifier also drove
  all four page self-tests against a single probe entry, asking one page five
  times in about six seconds — which tripped 特許庁's WAF, and the run refused
  because of its own traffic. The self-tests are now spread across hosts, hosts
  measured to challenge are placed *last* in that rotation, and the verifier
  refuses outright if every `:page-identity` entry is on one host.

  Recovery time is not uniform and no figure for it is written down as though
  it were: `www.meti.go.jp` answered normally again about four and a half
  minutes after being tripped, while `www.jpo.go.jp` was still challenging most
  of an hour later.

The host assumptions live in `:host-behaviour` entities and are re-measured on
every run, rather than written down once with a date beside them.

## License

AGPL-3.0-or-later.
