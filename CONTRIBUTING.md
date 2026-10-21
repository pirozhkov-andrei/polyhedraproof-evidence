# Computing units of the Polyhedraproof campaign: a guide for contributors

Thank you for contributing computing time. This guide takes you from the files you download to
the results you send back. It needs about half an hour of your attention; the computer then
works on its own.

## What you are doing

The question is whether a polyhedron with twelve faces exists in which every face shares an
edge with every other face. The project reduces it to 59 Boolean formulas
([README](https://github.com/pirozhkov-andrei/polyhedraproof/blob/v1.0.0/README.md)), and all of
them are computed in one *campaign*, recorded in the registry
[polyhedraproof-evidence](https://github.com/pirozhkov-andrei/polyhedraproof-evidence). The
campaign is divided into units of work. In reply to your offer you received the numbers of one or
more units and a **worker name** (`wNNN`); everything else comes from the registry and the
release, each file with its published SHA-256. Your computer runs one worker in a Docker container
with the release image of Polyhedraproof 1.0.0. The worker refutes the cases of your
units with the SAT solver CaDiCaL, and every refutation is checked by the formally verified
checker cake_lpr. The worker writes everything it does into a hash-chained journal, and the
journal is what you send back.

One unit has 1024 cases of one formula. **How long a unit takes is known only roughly.** In the
pilot computation, the eight units of the formulas of maps 1 and 2 took between **57 and 117
core-hours** of processor time, 80 on average; with `--jobs 8`, that is 7 to 15 hours. The time
goes to a few hard cases: 1 % of the cases took half of it, and a single case took 22 core-hours.
So units differ a lot even within one formula, and the units of the other 57 formulas may be
several times cheaper or more expensive: nobody has measured them. Take these numbers as a rough
guide, not a promise. If a unit takes much longer than you can give, stop and send what you have:
partial work is not lost, the rest is given to someone else.

## What you need

- **Linux on x86-64**, or Windows 10/11 with **WSL 2** and Docker Desktop (WSL integration on).
  macOS and ARM machines are not supported: the verified checker is x86-64 machine code.
- **Docker**, usable by your user without `sudo` (`docker run --rm hello-world` works).
- For each parallel job: one CPU core, **2 GiB of RAM** and **5 GiB of free disk**, plus 10 GiB
  for the image and the journal.
- Under WSL, keep every directory of this guide **below your Linux home** (`~`), not on `/mnt/c`.
  Docker there gets only the memory WSL allows (by default half of the RAM; `.wslconfig` changes
  it), so choose `--memory` below that.

## 1. Download and verify

```sh
mkdir -p ~/faces12/release ~/faces12/campaign && cd ~/faces12/release
for f in faces12-campaign-1.0.0.tar polyhedraproof-1.0.0-source.tar.gz SHA256SUMS; do
  curl -fLO https://github.com/pirozhkov-andrei/polyhedraproof/releases/download/v1.0.0/$f
done
sha256sum -c --ignore-missing SHA256SUMS
cd ~/faces12/campaign
for f in campaign-1.0.0.tar.gz SHA256SUMS; do
  curl -fLO https://github.com/pirozhkov-andrei/polyhedraproof-evidence/releases/download/campaign-1.0.0/$f
done
sha256sum -c SHA256SUMS
```

Both checks must print `OK`. The release of the software is also archived on Zenodo
([doi:10.5281/zenodo.23491042](https://doi.org/10.5281/zenodo.23491042)), with the same SHA-256.

## 2. Load and check the image

```sh
docker load -i ~/faces12/release/faces12-campaign-1.0.0.tar
docker image inspect --format '{{.Id}}' faces12-campaign:1.0.0
docker run --rm --init --network none faces12-campaign:1.0.0 selfcheck
```

The ID must be one of these two (the second appears with Docker's classic image store):

```text
sha256:2af5b6382b43b4fd3dd0811d88711fd3cdad98983e26fd1597eb2e9f130c39d7
sha256:0b3680264a8ef18f3c75de639ccde69c4b9b15db0974c51b23abe5b6035e5ce2
```

`selfcheck` must end with `11 of 11 checks hold`. If it doesn't, stop and write to the
organizer.

## 3. Set up your worker

Put in your worker name and the list of your units, exactly as in the reply you received
(at most 10 units; they all run in one worker, launched once):

```sh
W=wNNN
UNITS="map07-1 map07-2 map07-3"
```

Then run, unchanged. Its first two lines save your settings in `~/faces12/wNNN.env`: in any new
terminal later (the next day, after a reboot), `. ~/faces12/wNNN.env` sets them again.

```sh
printf 'W=%s\nUNITS="%s"\nIO="%s"\nWRAP="bash %s"\nIMAGE=faces12-campaign:1.0.0\n' "$W" "$UNITS" "$HOME/faces12/$W" "$HOME/faces12/polyhedraproof-1.0.0/pipeline/faces12-worker.sh" > ~/faces12/$W.env
echo 'unit_file() { case $1 in rest-*) echo "units-rest/$1.json" ;; *) echo "units/${1%-*}/shard-${1##*-}.json" ;; esac; }' >> ~/faces12/$W.env
. ~/faces12/$W.env
mkdir -p "$IO/inbox" "$IO/units-rest"
tar -xzf ~/faces12/campaign/campaign-1.0.0.tar.gz -C "$IO"
tar -xzf ~/faces12/release/polyhedraproof-1.0.0-source.tar.gz -C ~/faces12
head -c -1 "$IO/campaign.json" | sha256sum   # must be the campaign ID in the registry's README
```

A unit named like `map07-2` is the file `units/map07/shard-2.json` of the campaign archive. A unit
whose name starts with `rest-` (the open part of an abandoned unit, planned again) is not in the
archive; this downloads such units, if you have any:

```sh
for u in $UNITS; do case $u in rest-*) curl -fsSL "https://github.com/pirozhkov-andrei/polyhedraproof-evidence/raw/main/campaign-1.0.0/units-rest/$u.json" -o "$IO/units-rest/$u.json" ;; esac; done
```

Check every unit against the SHA-256 published in the registry, and put all of them into the
worker's inbox:

```sh
for f in units.sha256 units-rest.sha256; do curl -fsSL "https://github.com/pirozhkov-andrei/polyhedraproof-evidence/raw/main/campaign-1.0.0/$f" -o ~/faces12/campaign/$f; done
for u in $UNITS; do f=$(unit_file "$u"); awk -v f="$f" '$2 == f' ~/faces12/campaign/units.sha256 ~/faces12/campaign/units-rest.sha256 | grep . || echo "0000000000000000000000000000000000000000000000000000000000000000  $f"; done > ~/faces12/$W-units.sha256
(cd "$IO" && sha256sum -c ~/faces12/$W-units.sha256) && for u in $UNITS; do cp "$IO/$(unit_file "$u")" "$IO/inbox/$u.json"; done && ls "$IO/inbox"
```

Every unit must print `OK`, and the inbox must then hold one file per unit. A unit that prints
`FAILED` is misspelled or not published: check the reply and write to the organizer.

## 4. Launch (once)

Choose the number of jobs `J` (usually the number of physical cores, minus one if you use the
machine meanwhile), then give it `2·J` GiB of memory and `5·J + 5` GiB of disk. For 8 jobs:

```sh
FIRST=$(unit_file $(set -- $UNITS; echo "$1"))
test "$(ls "$IO/inbox" | wc -l)" -eq "$(echo $UNITS | wc -w)" && $WRAP launch --worker $W --io "$IO" --image $IMAGE --shard "$FIRST" -- --jobs 8 --memory 16G --disk 45G
```

(`test` refuses to launch unless the inbox holds exactly one file per unit of your list.)

The worker takes every unit from its inbox when it starts. `--shard` names the first unit once more
as a safeguard (the copy in the inbox is then recognized as the same and skipped): a worker launched
with neither a shard nor a unit in its inbox would take the *whole campaign* as its own, which is
not what you were asked to do. If that ever happens, stop it (`$WRAP stop --worker $W`) and write
to the organizer.

`launch` returns at once; the container keeps running in the background. Check that it started and
took all your units (they move from `inbox/` to `inbox/accepted/` within a minute):

```sh
$WRAP logs --worker $W --tail 30
$WRAP status --worker $W --image $IMAGE
ls "$IO/inbox/accepted"
```

Right after the launch, `status.absent` for a few seconds is normal.

## 5. While it runs

In a new terminal, first: `. ~/faces12/wNNN.env` (with your worker name).

- **Status:** `$WRAP status --worker $W --image $IMAGE` shows the closed and open cases and the
  export of the journal.
- **Pause and resume** (also after a reboot): `$WRAP stop --worker $W`, later
  `$WRAP start --worker $W`. The worker resumes from its journal. Never run `launch` again for
  the same worker, and never delete its Docker volumes.
- **If it runs for more than a day:** make a backup now and then (it pauses the worker for a
  moment), and copy `~/faces12/backups` and `$IO/export` to another disk:

  ```sh
  $WRAP stop --worker $W && mkdir -p ~/faces12/backups && $WRAP backup --worker $W --to ~/faces12/backups && $WRAP start --worker $W
  ```

  A run of a few hours needs none of this.
- **Has it finished?**

  ```sh
  docker inspect -f '{{.State.Status}} {{.State.ExitCode}}' faces12-$W
  ```

| Shows | Meaning | What to do |
|---|---|---|
| `running 0` | still working | wait |
| `exited 0` | all your units are closed | step 6 |
| `exited 130` | stopped by you | `$WRAP start --worker $W` to continue |
| `exited 1` | work left open: a time limit you gave (`--max-hours`), or the resources did not fit | `$WRAP logs --worker $W --tail 50`, write in your issue |
| `exited 2` | a model was found | send the results now (step 6): it needs examination |
| `exited 3`, `4`, `5`, `6` | configuration, integrity, tool or anomaly | paste the output of `logs --tail 100` and `status` into a comment in your issue; don't start again |

## 6. Send the results

When the container has exited with 0, or when you have to stop for good (`$WRAP stop --worker $W`
first; in a new terminal, `. ~/faces12/wNNN.env` before anything):

```sh
JOURNAL=$(grep -o '"file":"journal\.[0-9]*\.jsonl"' "$IO/export/export.json" | cut -d'"' -f4) && tar --owner=0 --group=0 --numeric-owner -czf ~/results-$W.tar.gz -C "$IO" export/export.json "export/$JOURNAL" && (cd ~ && sha256sum results-$W.tar.gz) && cat "$IO/export/export.json"
```

The archive holds only the export descriptor and the journal it names. Write a comment in your
offer issue: attach the file `results-wNNN.tar.gz` from your home directory (drag it into the
comment box; about 1–2 MB per unit, GitHub takes up to 25 MB) and paste the two lines the command
printed, the `sha256sum` line and the `export.json` line. Only comments by the author of the issue
are taken, so post from the account that opened it. Under WSL, `cd ~ && explorer.exe .` opens your
home directory in Windows; if the machine has no browser, copy the file to one that has (for
example with `scp`).

If the file is larger than 25 MB (with at most 10 units it should not be), do not send it: write
in your issue, and the organizer will tell you how. Please send no files by email and no links to
other sites.

The file is public from the moment you attach it: anyone with the link can download it. The reply
in your issue confirms that the journal arrived intact and repeats its head: it is your record of
what you sent.

**Keep** the container, its volumes, `$IO` and your backups until the organizer confirms that
the journal arrived intact.

## 7. More work

One issue is one worker with at most 10 units, so that its archive stays well below GitHub's
25 MB. To compute more, open a new [offer issue](https://github.com/pirozhkov-andrei/polyhedraproof-evidence/issues/new?template=offer.yml) once the organizer has confirmed the
results of this one, and name this issue in it. You get a new worker name and new units: start at
step 3 with them (the downloads and the image of steps 1 and 2 stay), and send their results from
the new issue. Up to about 3000 core-hours in all go this way, one issue after another; for more,
write to the organizer (pirozhkov.andrey.u@gmail.com) first.

## 8. When everything is confirmed

```sh
docker rm faces12-$W
docker volume rm faces12-results-$W faces12-work-$W
```

## Please do not

- change the policy (`--policy-override`), the proof mode or the image;
- run two workers with the same name, or give your worker another name;
- edit anything in `$IO` except what step 3 does (unit files into `inbox/` and `units-rest/`);
- delete the volumes before the organizer confirms;
- send files by email or links to other sites: results go only into your offer issue.

## What is in your journal, and what will be published

The journal records the worker name, the processor model, the hashes of the tools and, for each
case, its outcome, the hash and size of its proof, and the processor time and memory used. It
records no host name, user name or host path. The container runs without network access. Your
journal is public from the moment you attach it to your issue, and all the journals of the
campaign are published (CC BY 4.0) with its evidence when the campaign ends, so that anyone can
check the result. You are credited as you choose in your offer, or not named at all.

Questions: ask in your issue, or write to the organizer. The procedure in full is in
[RUNNING](https://github.com/pirozhkov-andrei/polyhedraproof/blob/v1.0.0/docs/RUNNING.md) and
[COMPUTE](https://github.com/pirozhkov-andrei/polyhedraproof/blob/v1.0.0/docs/COMPUTE.md).
