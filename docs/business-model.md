# Business Model: Independent METI-Regulated Trade & Industrial-Policy Compliance Service — Japan (METI)

## Classification

- Repository: `cloud-itonami-iso3166-jpn-meti`
- ISO 3166 (agency-level): `JPN-METI`, parent `JPN`
- Ooyake cross-reference: `gov.jpn.meti` (Ministry of Economy, Trade and Industry / 経済産業省)
- Activity: export-control classification of controlled goods/technology under the Foreign Exchange and Foreign Trade Act (外国為替及び外国貿易法, commonly "外為法"/FEFTA), and navigation of METI industrial-policy subsidy/grant programs (e.g. GX/DX-related schemes) that a public-sector delivery may rely on
- Social impact: [:export-control-clarity :subsidy-access :public-spend-transparency]

## Customer

- an operator whose public-sector delivery includes goods or technology that may fall under FEFTA export-control classification
- an operator applying for a METI subsidy or grant program as part of winning or servicing a public contract
- a foreign supplier needing to confirm whether their product/technology requires an export license before delivery to a Japanese public-sector buyer

## Offer

- FEFTA export-control classification checklist and screening walkthrough for the operator's specific goods/technology
- METI subsidy/grant program eligibility navigation and application checklist
- ongoing regulatory-change monitoring for FEFTA control-list updates
- compliance-audit export package for the operator's own records

## Revenue

- per-engagement compliance-review fee
- recurring regulatory-change monitoring subscription
- compliance-audit export package

## Trust Controls

- any actual filing, registration, or compliance-program submission
  requires Industrial-Policy Compliance Governor clearance and always escalates to human
  sign-off (`:filing/submit` is never automated at any phase)
- a false or fabricated regulatory-requirement claim is a HARD hold that
  cannot be overridden by human approval alone — it must be corrected
  against a cited METI source first
- this service does **not** provide legal or tax advice; characterization
  and filing on the client's behalf beyond checklist/draft assistance
  routes to Japan-licensed counsel or a registered agent
- every requirement cites the official METI source or
  regulation, never invented

## Boundary with adjacent actors (read before forking)

- **`cloud-itonami-iso3166-jpn`**: the COUNTRY-level coordinator (general
  Japan public-sector market entry). This repo is a narrower, deeper
  AGENCY-level leaf — most operators need the country-level blueprint plus
  only the agency-level blueprints that actually apply to their contract.
- **`com-etzhayyim-ooyake`** (etzhayyim/root): read-only civic-wayfinding
  mirror of government structure, non-commercial, barred from acting as or
  for the government (G3 impersonation ban). This blueprint is commercial
  and never claims to be Ministry of Economy, Trade and Industry or an official channel.
- **`matsurigoto`** (etzhayyim/root): sovereign e-government statecraft —
  literally the government. This blueprint is an independent operator that
  engages with METI under its public rules — never the
  agency itself.
- **`com-etzhayyim-toritsugi`** (etzhayyim/root): guides a consenting
  INDIVIDUAL citizen through their OWN procedure, non-profit,
  donation-only. This blueprint's client is a business operator, not an
  individual citizen, and it is commercial.
- **`cloud-itonami-M6910`**: helps a client BECOME a legal entity
  (incorporation, ISIC 6910) — a prior, different regulatory phase (company
  law). This blueprint assumes incorporation is already done and handles
  METI-specific compliance (a different regulatory domain).
