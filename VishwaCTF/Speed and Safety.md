# Speed and Safety

## Challenge

Find the company and F1 team from this chain:

1. Yann LeCun is Executive Chairman at AMI Labs.
2. AMI Labs raised a `$1.03B` seed round co-led by 5 investors.
3. One of those investors also led a `$79M Series B` into an identity security company in October 2025.
4. The CEO of that company previously worked at an identity giant that sponsors an F1 team.

Flag format: `VishwaCTF{company_team}`

## Recon

TechCrunch's March 9, 2026 report on AMI Labs says the `$1.03B` round was co-led by:

- Cathay Innovation
- Greycroft
- Hiro Capital
- HV Capital
- Bezos Expeditions

Source:

- https://techcrunch.com/2026/03/09/yann-lecuns-ami-labs-raises-1-03-billion-to-build-world-models/

That gives the investor shortlist.

## Pivot on the October 2025 `$79M Series B`

Searching those investors against the clue about an identity security company leads to ConductorOne.

ConductorOne's October 28, 2025 press release states:

- ConductorOne raised `$79M` in Series B funding
- the round was led by `Greycroft`

Source:

- https://www.conductorone.com/news/press-release/conductorone-raises-79-million-series-b/

So the identity company is `ConductorOne`.

## CEO clue

Now verify the CEO's previous employer.

TechCrunch's 2021 ConductorOne seed article says Alex Bovee and Paul Querna led Zero Trust strategies and products at `Okta`, and that Bovee left Okta before starting ConductorOne.

Source:

- https://techcrunch.com/2021/04/05/conductorone-raises-5m-in-seed-round-led-by-accel-to-automate-your-access-requests/

So the "identity giant" is `Okta`.

## F1 sponsor clue

Okta's January 28, 2025 press release says it announced a multi-year partnership with the `McLaren Formula 1 Team`.

Source:

- https://www.okta.com/es-mx/newsroom/press-releases/okta-announces-multi-year-partnership-with-the-mclaren-racing-formula-1-team/

Therefore the F1 team is `McLaren`.

## Answer

- Company: `conductorone`
- Team: `mclaren`

Flag:

`VishwaCTF{conductorone_mclaren}`
