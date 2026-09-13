# Progress Software

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Progress Software Corporation (NASDAQ: PRGS) is a Burlington, Massachusetts enterprise software
company that builds and acquires infrastructure and digital-experience products. This repository is
an independent, third-party profile of the API surface Progress publishes to the public.

## What this profile holds

Three real, first-party machine-readable contracts were harvested — **558 operations** in total:

| API | Spec | Operations | Where it runs |
|---|---|---|---|
| Chef Automate API | Swagger 2.0, 215 paths | 277 | customer-hosted |
| MOVEit Transfer REST API | Swagger 2.0, 83 paths | 113 | customer-hosted, optionally licensed |
| WhatsUp Gold REST API | Swagger 2.0, 121 paths | 168 | customer-hosted, port 9644 |

Two further API surfaces are documented but publish no downloadable contract: the **ShareFile API v3**
(OData, per-tenant `$metadata`) and **Sitefinity CMS headless OData services** (per-installation
`$metadata`).

## What stood out

- **Progress publishes a real `llms.txt`, and publishes it six times over** — on `progress.com`,
  `telerik.com`, `sharefile.com`, `chef.io`, `whatsupgold.com` and `kemptechnologies.com`. This is a
  deliberate per-brand rollout, and it is more agent-facing discovery work than most of the catalog
  does at all.
- **Every MCP server Progress ships is one a human has to install first.** Five of them — KendoReact,
  Kendo Angular, Kendo jQuery, Telerik Blazor and OpenEdge. None is a hosted endpoint an agent can
  call; `mcp.progress.com` and `mcp.telerik.com` do not resolve.
- **The MCP surface and the API surface do not touch.** Zero of the 558 published REST operations has
  an MCP tool; the MCP tools all generate UI code. See `mcp/progress-software-tool-crosswalk.yml`.
- **No idempotency anywhere.** The string `idempoten` appears zero times across all 558 operations.
- **The RFC 9116 `security.txt` is served on `telerik.com` only** — not on `progress.com`, `chef.io`,
  `sharefile.com`, `whatsupgold.com` or `kemptechnologies.com`. The corporate disclosure policy and
  the Bugcrowd DevTools VDP both exist; they are just not advertised where the standard says to look.
- **`status.progress.com` 302s to an explicitly inactive Statuspage.** Three live per-product status
  pages exist instead (Chef, ShareFile, MOVEit Cloud).
- **Internal build addresses shipped in the contracts** — MOVEit Transfer declares `host: 127.0.0.1`,
  WhatsUp Gold declares `host: 10.40.67.158:9644` with a `tokenUrl` of `http://localhost:8734/...`.
- These are four separately-acquired API programs that agree on nothing: three auth headers, four
  pagination vocabularies, three error envelopes. See `conventions/progress-software-conventions.yml`.
