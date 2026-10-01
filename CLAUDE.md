# CLAUDE.md

Working rules for AI-assisted changes in this repository.

## Non-negotiable rules

1. **Nothing from a source system is ever copied in here.** Not a file, not a snippet, not a
   schema dump, not a screenshot. These are write-ups, and CI fails if a `.py`, `.ts`,
   `.tsx`, `.sql` or `.prisma` file appears in the tree.
2. **No real identifier, ever.** No person's name, CPF, address, phone or e-mail; no company
   name or CNPJ; no certificate serial; no chassis, plate or RENAVAM; no valid CNJ case
   number; no token, app id, tenant id, hostname or bucket name. Where a data shape matters,
   describe it by its column names and its row count.
3. **Every number is checkable or absent.** Line counts, record counts, test counts and
   durations are fine when they were measured. A performance claim, a cost saving or an
   accuracy figure goes in only with the method that produced it. No rounded-up impact
   numbers.
4. **Clients and employers are described, never named.** "A law firm's controllership
   team", "a vehicle dealer", "a municipal revenue programme". The anonymisation is the
   point of the repository existing at all.
5. **Be honest about what is unfinished and what is wrong.** A case study that only lists
   successes reads as marketing. Every write-up carries a "what I would do differently"
   section, and the weaknesses in it are real ones.
6. **Each case study keeps the seven required headings**, in order, because CI checks them:
   `## The problem`, `## What it does`, `## Architecture`, `## Design decisions`,
   `## Data and privacy`, `## What I would do differently`,
   `## Why this is a case study and not a repository`. Add `## How AI was used` wherever a
   language model was in the loop, and `## Scale and state` wherever there are measured
   numbers to report.

## Conventions

- British English. Prose, not bullet soup: a list is for genuinely parallel items.
- A diagram is a Mermaid block, so it renders on GitHub and stays diffable.
- Link to the published repositories where a component was extracted, so a reader can see
  the code that does exist.
- Commits: English, Conventional Commits, one logical change each, no AI attribution
  trailers. `docs:` for a new or edited case study.
