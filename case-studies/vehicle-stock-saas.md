# A multi-tenant SaaS over a certificate-only government API

**Product:** my own, built with a technical partner · **Role:** engineer and founder
**Status:** in production, small number of paying customers
**Headline:** 11 of 41 documented government operations wrapped · 6,256 lines · no local copy of the data

Most of the engineering in this product is not the product. It is the layer that makes an
undocumented, certificate-only federal API usable at all.

## The problem

Brazil runs a mandatory federal register of vehicles held in stock — RENAVE, the *Registro
Nacional de Veículos em Estoque*. When a manufacturer or an authorised dealer takes a vehicle
into stock, and again when it sells one, that movement must be registered with the government
**before ownership can legally change hands**. The register is operated by SERPRO, the federal
government's IT corporation, on behalf of the national traffic authority and the state vehicle
agencies.

Three things make it hard for the businesses obliged to use it.

**The only credential is a digital certificate.** No API key, no OAuth. Authentication is
mutual TLS with an ICP-Brasil A1 certificate — a password-protected PKCS#12 file issued to a
specific company's tax ID, and separately credentialed with the traffic authority for a
specific RENAVE profile. A certificate is a legal signing identity, not an API key.

**There are two incompatible profiles.** A manufacturer and a dealer talk to different URL
prefixes, with different operations and different mandatory fields on the same logical action.
A dealer's sale requires a buyer; a manufacturer's requires a tax-benefit classification of
that buyer, from a closed list of eight. The payloads are not interchangeable, and a company's
certificate determines which one it is allowed to be — a fact the company's own staff usually
do not know.

**There is no documentation to read.** The OpenAPI specification exists but its location is
not published. The government's own interface is referred to by the people who have to use it
as the black-screen system.

My customers are trailer and semi-trailer manufacturers and dealers — towed units are
separately registered vehicles in Brazil — and the proposition is compliance as a service:
they upload a certificate, and the stock work happens in a web interface instead of in the
black screen.

## What it does

- Takes a company's PKCS#12 certificate, **asks the government which profile it is credentialed
  for**, and configures the tenant from the answer rather than from a dropdown.
- Lists a company's stock, its sold vehicles and, for manufacturers, the vehicles pre-registered
  and awaiting entry — each read live from the register, with no local copy.
- Registers a vehicle into stock, cancels an entry, and registers a direct sale, with the
  payload shape and the available operations determined by the tenant's profile.
- Downloads the ownership-transfer authorisation as a PDF for a completed sale.
- Sells prepaid credits, topped up by instant bank transfer, credited by a payment-gateway
  webhook.
- Exposes a diagnostics endpoint that answers, in one request, the question that costs the most
  hours on a mutual-TLS integration: *is the right certificate loaded on this deploy?*

## Architecture

```mermaid
flowchart TB
  COOKIE[session cookie
  carries the active company] --> PAGE[server component]
  PAGE --> DB[(Postgres
  users, companies, credit orders)]
  DB -->|that tenant's certificate
  and profile| FACTORY[build a client
  PKCS#12 parsed in memory]
  FACTORY --> PREFIX{profile}
  PREFIX -->|dealer| ITE[/api/ite/*]
  PREFIX -->|manufacturer| MONT[/api/montadora/*]
  ITE --> GOV[(federal register
  the system of record)]
  MONT --> GOV
  GOV --> PROPS[plain props]
  PROPS --> TABLE[client table
  filter, sort, page in the browser]
  TABLE -->|modal| ACTION[server action
  Zod, then POST]
  ACTION --> FACTORY
  PAY[payment gateway webhook] --> ORDER[(credit order
  keyed on the gateway's own id)]
```

Next.js App Router, React, Postgres through Prisma, deployed on a small platform-as-a-service.
Thirteen routes, fifteen server actions, three database models.

The read path is deliberately short and has no client-side data layer: a server component
reads the session, looks up that company's certificate, builds a client, calls the government,
and hands plain data to a client component that does the filtering and paging in the browser.
Mutations invert it — a modal calls a server action, which re-derives the client from the
session, validates, posts, and returns either a success or a message.

**There is no vehicle table.** The government register is the system of record, and a second
copy of it would only ever be wrong. Every page render is a live call. The cost is a hard
dependency on an API whose availability I do not control; the compensation is that every read
path returns data *and* an error field rather than throwing, so an outage degrades to a banner
above an empty table rather than to an error page.

## Design decisions

**The certificate is per tenant, and the system asks the government what it is.** Onboarding a
company is four fields: tax ID, legal name, drag a certificate file, type its password. The
system then fires the authenticated-client probe at *both* profile roots concurrently and reads
the profile off whichever answers. Neither answering is an error that reports both status codes
and tells the operator to check their credentialing.

The user never picks their profile from a dropdown, because they usually do not know it, and
because a declared profile can drift from reality while a detected one cannot. That one stored
string then drives five separate behaviours: the URL prefix, which operations the client
exposes, which fields the sale form renders, whether the cancel button appears, and whether the
manufacturer-only page will load at all.

**A directory of brute-force probes, kept.** There was no documentation, so I wrote six
standalone scripts: dump a certificate's subject and validity; point a certificate at two known
endpoints and print the status codes; sweep a list of candidate paths with emoji-tagged results;
scrape the Swagger UI's HTML for the specification URL and then try five candidate paths; and —
the one that took longest — brute-force sixteen candidate names and eight candidate URL prefixes
to find out what the second profile is even *called*.

Those scripts are not build tooling and they are not dead code. They are the written record of
how the API was discovered, and they are the first thing I would reach for if an endpoint
started behaving differently.

**404 means empty.** The stock endpoint answers `404 Estoque não encontrado` for an empty stock
list rather than `200 []`. One line in the transport layer normalises that to an empty array,
because without it a dealer with no vehicles sees a hard error on their first login. The
diagnostics endpoint surfaces the reasoning rather than hiding it: when the list comes back
empty it attaches a note saying the upstream returned 404 and that this was read as an empty
list for the default three-month window.

Documenting a workaround at the point of observation, rather than in a wiki, is the only version
of that habit that survives.

**Hand-rolled HTTP, because the abstraction did not fit.** Node's `fetch` cannot carry a
per-request client certificate, and the certificate is the entire authentication scheme. So the
transport is `https.request` directly, with the promise-wrapping boilerplate that implies. Four
HTTP-capable dependencies sit unused in the manifest, left over from assuming otherwise.

**Idempotency by borrowing the upstream primary key.** A credit order's primary key has no
default: it *is* the payment gateway's charge identifier. Webhook idempotency is therefore a
primary-key lookup and a status check — no dedupe table, no event log, no nonce. Gateways retry
aggressively, and this makes double-crediting structurally hard rather than merely unlikely. The
credit itself is one transaction with an atomic increment, so concurrent deliveries cannot lose
an update, and money is a fixed-point decimal throughout.

**A diagnostics endpoint as a product decision.** It runs two independent probes, each in its own
error handler, **accumulating failures into a list rather than stopping at the first** — so one
broken thing does not mask another. It returns the environment, the base URL, whether a
certificate is configured, the tax ID the system expected, the tax ID the government says you
authenticated as, and a three-valued verdict on whether those match: true, false, or null when
the check could not run.

That one field answers the question that burns the most hours on this kind of integration. It
also echoes back the exact method, path and URL it called, which turns an incident into a
copy-pasteable reproduction. Transport failures return 502 rather than 500, because "the
government API failed" and "we failed" are different facts.

**Validation that encodes the domain, not just the types.** The chassis pattern excludes I, O and
Q — the letters the vehicle identification standard forbids because they are confusable with 1
and 0 — with the reason in the error message. Invoice keys are exactly forty-four digits. The
corporate tax ID gets a full check-digit calculation, so a mathematically impossible one never
reaches the database.

**Performance care where it is nearly free.** The marketing page checks the device's reported
memory and Android version and disables its continuous animations on low-end hardware, jumping
the counters straight to their final values. The customers are trailer dealerships browsing on
cheap Android phones.

## Scale and state

| | |
| --- | --- |
| Hand-written TypeScript and TSX | 6,256 lines across 68 files |
| Government operations wrapped | 11 of 41 documented |
| Routes | 13 (1 public, 12 authenticated) |
| Server actions | 15 |
| Database models | 3 and one enum |
| API-discovery scripts | 6 |
| Automated tests | none |

The two saved specification dumps describe nineteen and twenty-two paths respectively, with
thirty-nine and forty-one definitions. What is *not* wrapped is the product's roadmap, and it
clusters: the multi-party transfer choreography by which a manufacturer hands a vehicle to a
dealer, the cancellation of a registered sale, the signed entry and exit instruments, and — the
one that stings — the path for selling an unfinished vehicle, which is a trailer chassis sold
before its bodywork, and is precisely my customers' product category.

The specification also describes a far richer stock record than the application reads: lien and
theft restrictions, registration-certificate generations, odometer readings, and the identifiers
that chain a stock record across transfers. The client declares a six-field subset of it.

## Data and privacy

In production the system processes the personal data of buyers — names and tax IDs appear in a
sale payload — along with each customer company's own registration details and its digital
certificate. The certificate is the sensitive asset by a wide margin: an A1 certificate is a
legal signing identity capable of transferring vehicle ownership, not merely an API credential.

Vehicle and buyer data is never persisted locally. Everything comes from the register at request
time and is discarded when the response is rendered, which means there is no shadow database of
other people's vehicles to leak.

The certificate is a different story, and an honest one: it is held in the application's own
database so that the service can act for a tenant without that tenant being present. That is a
real design tension rather than an oversight, and getting it right is the top item on the list
below.

## What I would do differently

**Finish the application layer to the standard of the integration layer.** This is the summary
of everything else here. The part that talks to the government had the care: probes,
auto-detection, error accumulation, a diagnostics endpoint. The part that decides *who is
asking* did not get the same attention, and it is the half that matters more.

Concretely, and in the order I would do them: the session scheme needs to be signed rather than
trusted, the tenant check needs to be structural — middleware over the authenticated route
group, not a line copy-pasted into each page — and the certificate needs envelope encryption
with the key held somewhere other than next to the data.

**Fix the sentence before the code.** The company-creation form tells the customer that the
certificate is used only to authenticate and is not stored on our servers. It is stored on our
servers. That copy came from a true statement about the *file system* — the key is parsed in
memory and never written to disk — which became a false statement about storage on its way into
the interface. Whatever else gets done, that sentence is wrong and goes first.

**Migrations from the first schema change.** There is no migrations directory. Schema changes
were applied by pushing the schema or, on at least one occasion, by hand in a database console,
because the push command kept failing against the connection pooler on Windows. The predictable
thing then happened: a deploy shipped code referencing three new columns before the columns
existed, and the site was down. There is consequently no schema history and no reproducible way
to stand the database up from scratch.

**Any tests at all.** There are none, and the riskiest gap is the profile branching: a dealer
payload sent on a manufacturer path, or a sale missing the tax-benefit field, against an API
whose operations the interface itself warns the operator are irreversible. A handful of tests
around that branch would have cost an afternoon.

**Close the billing loop or remove it.** Credits are incremented by the webhook and displayed in
the header. Nothing decrements them, no price list exists, and no operation checks the balance.
Half a billing system is worse than none, because it looks finished.

**Verify the server, not just the client.** The transport disables verification of the
government's own certificate — almost certainly to get past a chain issue during integration,
and never revisited. That forfeits the server half of mutual TLS. The correct fix is to supply
the root and intermediate bundle, and it is the change I would make first in that file. The
standalone library extracted from this project
([mtls-pkcs12-agent](https://github.com/fillipeml/mtls-pkcs12-agent)) does it the right way, and
names the insecure option so that reaching for it has to be deliberate.

## Why this is a case study and not a repository

This one is mine to publish, which makes it the most interesting of the three to decide about.

It stays closed for two reasons. The first is commercial: it is a live product with paying
customers, and the half that is worth reading — the integration layer — is also the half that
took the longest to work out. The second is that the repository is not in a publishable state
independently of any decision about the code. Dead code in it carries real buyers' names and tax
IDs, the seed script creates a real named user, and the handover document contains working
credentials. Sanitising that is a day's work that produces no artefact anyone would clone.

So the reusable part was extracted instead, rewritten against generated certificates, and
published on its own: [mtls-pkcs12-agent](https://github.com/fillipeml/mtls-pkcs12-agent) is the
certificate handling from this product, done properly, with the server-verification mistake
fixed and sixty-one tests. That repository is the code behind this write-up, as far as any code
behind it exists in public.

What is left here is the thing a repository would not have shown anyway: that the shape of the
whole product follows from one constraint. Hand-rolled HTTP because the platform's client cannot
carry a certificate. A directory of brute-force probes because there was nothing to read.
Profile detection because the user cannot be expected to know the answer. Empty-means-404 because
the API says so. No vehicle table because the government already has one.
