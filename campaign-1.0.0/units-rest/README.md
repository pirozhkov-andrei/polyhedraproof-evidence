# Units planned again

The open part of a unit whose worker stopped before closing it, planned again from a certificate
of all returned journals (`cubeproof shard --from-open`) and cut to that unit, so that it does not
overlap units other workers still hold. One file per unit, `rest-KEY-K.json`; the SHA-256 of each is
in [`../units-rest.sha256`](../units-rest.sha256). Empty until a unit is abandoned.
