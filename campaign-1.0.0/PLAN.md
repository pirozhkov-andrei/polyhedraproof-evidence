# Polyhedraproof 1.0.0: the campaign of the 59 formulas (plan)

Published before any unit of work is handed out; the signed commit that adds this file, and the
immutable release `campaign-1.0.0` of this repository, are the record of the plan. Afterwards only
the running records change.

## What is computed

- **Software:** Polyhedraproof 1.0.0, [doi:10.5281/zenodo.23491042](https://doi.org/10.5281/zenodo.23491042),
  tag [`v1.0.0`](https://github.com/pirozhkov-andrei/polyhedraproof/tree/v1.0.0) (signed).
- **Image:** `faces12-campaign:1.0.0`, image ID
  `sha256:2af5b6382b43b4fd3dd0811d88711fd3cdad98983e26fd1597eb2e9f130c39d7` (containerd image store;
  `sha256:0b3680264a8ef18f3c75de639ccde69c4b9b15db0974c51b23abe5b6035e5ce2` with the classic store);
  every worker and every audit uses this image.
- **Campaign:** the 59 formulas of the article, default policy, proof mode `transient`, solver
  arguments `["-q", "--lrat"]`, root split `proof-prefix-v1` (4096 cases per formula).
  Campaign ID `0f8b35ed0805b2df8100ea5745998f9e636fe1194a2299db2ee44472179dda2f`;
  created 2026-10-20T08:36:35.839Z by Andrei Pirozhkov.
- **Files:** `campaign-1.0.0.tar.gz`, SHA-256
  `32d7badfbf805bde8ac13bf5e17e6e17b21af42717f78c5068f171453e3339fb` (`campaign.json`,
  `campaign-init.json`, `prefixes.json`, `formulas/`, `units/`), the asset of the release
  [`campaign-1.0.0`](https://github.com/pirozhkov-andrei/polyhedraproof-evidence/releases/tag/campaign-1.0.0)
  of this repository.

## Units of work

Each formula is divided into 4 units of 1024 cases (`cubeproof shard --formula KEY --machines 4`):
236 units, files `units/KEY/shard-K.json` (KEY from `map01` to `map59`, K from 1 to 4), SHA-256
in `units.sha256`. A unit is assigned to a worker with a name `wNNN` that is used once; a worker
gets at most 10 units, through one offer issue of this repository; the
organizer keeps the list of assignments (unit, worker, dates) and publishes it, without personal
data, with the final evidence. The open part of an abandoned unit is planned again from a
certificate of all returned journals and assigned to a new worker; its file goes into
`units-rest/` here, its SHA-256 into `units-rest.sha256`.

The pilot computation (maps 1 and 2 with 1.0.0rc1,
[doi:10.5281/zenodo.23464843](https://doi.org/10.5281/zenodo.23464843)) puts a unit at 57 to 117
core-hours (80 on average; the eight units of those two formulas only) and the campaign at about 19,000 core-hours, an order of magnitude. The units of the other
formulas have not been measured and may differ several-fold; the estimate is revised from the
returned journals as the campaign goes on.

## How the result will be checked

- **Journals and heads:** every returned journal is attached by its contributor to their offer
  issue in this repository and checked on arrival (`cubeproof journal verify`); its head, record
  count and length are recorded, and the organizer's receipt in the same issue states them, so
  that every contributor holds a dated, public record of the journal they sent. The list of heads
  is published with the final evidence.
- **Certification:** after every return, `cubeproof certify` and the independent
  `tools/certify_min.py` on all returned journals; they must agree byte for byte.
- **Audit of every closed formula:** a strict audit (`cubeproof audit --sample 50`) of the
  formula's leaves by the organizer, on a machine that did not compute them. The seed of formula KEY is the SHA-256 of the ASCII string `SECRET:KEY` (for example
  `SECRET:map07`), where SECRET is a string of 64 hexadecimal characters, committed here now by
  its SHA-256 and revealed at the end:

  `SHA-256(SECRET) = 5dd697cf5beec3e1ed4bfc0eb1bd9a4fe873c7fe43d3da977799d6e4c51bacd7`

  A `MISMATCH` or `FAILED` leaf leads to a larger audit of the journal that recorded it, and that
  journal does not enter the result until the leaf is explained.
- **Final:** when all 59 formulas are closed, the evidence package of all journals, both
  certifiers, a final strict audit of 500 leaves whose seed is the SHA-256 of the ASCII string
  `SECRET:final`, SECRET revealed (anyone can then check its SHA-256 above and every seed), and an
  invitation to recompute every leaf independently. The mathematical claim needs, in addition, the
  [claim review](https://github.com/pirozhkov-andrei/polyhedraproof/blob/v1.0.0/docs/COMPUTE.md#the-claim-review).

## Publication

- before the start, here: this plan, the campaign files and the guide for contributors (signed
  commits), and the campaign archive as the immutable release `campaign-1.0.0`; during the
  campaign, the files of units planned again (`units-rest/`) and, in the issues, each returned
  journal with its receipt;
- when the campaign ends, on Zenodo with a DOI: the final evidence package (campaign, bundles,
  every journal, the certificates of both certifiers), the audit reports, the list of journal
  heads recorded on arrival (`utc  journal  seq  head  offset  worker`), the list of assignments
  (`unit  formula  part  cases  status  worker  sent  due  returned  head`) and SECRET. Data under
  CC BY 4.0.

Contact: Andrei Pirozhkov, pirozhkov.andrey.u@gmail.com,
[ORCID 0009-0008-6901-7843](https://orcid.org/0009-0008-6901-7843).
