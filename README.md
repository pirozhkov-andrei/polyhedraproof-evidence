# Polyhedraproof: the campaign of the 59 formulas

This repository is the public record of the computation that tries to settle a question of
discrete geometry: is there a polyhedron with twelve faces in which every face shares an edge
with every other face? The software and the article are in
[pirozhkov-andrei/polyhedraproof](https://github.com/pirozhkov-andrei/polyhedraproof)
([doi:10.5281/zenodo.23434449](https://doi.org/10.5281/zenodo.23434449)). The article reduces the
question to 59 Boolean formulas; this campaign refutes them case by case, each refutation checked
by the formally verified checker cake_lpr, so that anyone can verify the result without trusting
the program that ran it.

**Status:** running. The journals, the certificates, the audits and the list of assignments are
published together, on Zenodo, when the campaign ends.

## How to help

The campaign needs about 19,000 core-hours in total, an order of magnitude from the pilot
computation ([doi:10.5281/zenodo.23464843](https://doi.org/10.5281/zenodo.23464843)). It is divided
into 236 units of 1024 cases. In the pilot a unit took 57 to 117 core-hours, 80 on average, because
a few hard cases take most of the time; the units of the other formulas have not been measured and
may be several times cheaper or more expensive. With 8 parallel jobs, the pilot's units took 7 to
15 hours each: a rough guide, not a promise. Partial work counts: if a unit takes longer than you
can give, the rest goes to someone else.

1. Read the guide: [CONTRIBUTING.md](CONTRIBUTING.md).
   You need Linux on x86-64 (or Windows with WSL 2) and Docker.
2. Offer time: open an issue **[Offer computing time](https://github.com/pirozhkov-andrei/polyhedraproof-evidence/issues/new?template=offer.yml)**, or write
   to pirozhkov.andrey.u@gmail.com.
3. You receive a worker name and the numbers of your units. Everything else is here, with its
   SHA-256: nothing is sent by email.
4. Compute, then email the archive of the journal (about 1–2 MB per unit).

Without Linux on x86-64, a Linux x86-64 virtual machine with Docker, for example in a cloud,
works.

## Files

| Where | What |
|---|---|
| release [`campaign-1.0.0`](https://github.com/pirozhkov-andrei/polyhedraproof-evidence/releases/tag/campaign-1.0.0): `campaign-1.0.0.tar.gz` | the campaign: `campaign.json`, the 59 formula bundles and the 236 units. SHA-256 `32d7badfbf805bde8ac13bf5e17e6e17b21af42717f78c5068f171453e3339fb` |
| Zenodo, when the campaign ends | the final evidence package: every journal, the certificates of both certifiers, the audit reports, the journal heads recorded on arrival and the list of assignments |
| [`campaign-1.0.0/PLAN.md`](campaign-1.0.0/PLAN.md) | the plan, published before the first unit was handed out: what is computed, how it is checked, how the audit samples are chosen |
| [`campaign-1.0.0/campaign.json`](campaign-1.0.0/campaign.json) | the campaign document; its ID is `0f8b35ed0805b2df8100ea5745998f9e636fe1194a2299db2ee44472179dda2f` (`head -c -1 campaign.json \| sha256sum`) |
| [`campaign-1.0.0/units.sha256`](campaign-1.0.0/units.sha256) | the SHA-256 of every unit file of the archive |
| [`campaign-1.0.0/units-rest/`](campaign-1.0.0/units-rest/) | the open parts of abandoned units, planned again (`units-rest.sha256`) |

Every commit of this repository is signed, and the release `campaign-1.0.0` is immutable: the plan
and the campaign were fixed here before the first unit was handed out.

## Software and image

All workers and audits use Polyhedraproof 1.0.0, release image `faces12-campaign:1.0.0`, image ID
`sha256:2af5b6382b43b4fd3dd0811d88711fd3cdad98983e26fd1597eb2e9f130c39d7`
(`sha256:0b3680264a8ef18f3c75de639ccde69c4b9b15db0974c51b23abe5b6035e5ce2` with Docker's classic
store), from the [release v1.0.0](https://github.com/pirozhkov-andrei/polyhedraproof/releases/tag/v1.0.0).

Data in this repository and the evidence of the campaign: CC BY 4.0. Contact: Andrei Pirozhkov,
pirozhkov.andrey.u@gmail.com, [ORCID 0009-0008-6901-7843](https://orcid.org/0009-0008-6901-7843).
