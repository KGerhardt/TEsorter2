# TEsorter2 container

Docker and Apptainer images with TEsorter2 and every search engine of its
cascade: pyhmmer/HMMER, nail (+MMseqs2) and BATH. BATH is not on conda, so it
is compiled from source, pinned to the release used in the TEsorter2
benchmarks: BATH v2.0.0-rc7 (`b65b6bb9`) with easel `74e45a88`, the same
sources as the bioconda `bath` recipe (see `/opt/bath/VERSION` inside the
image).

## Build

Docker, from the repository root:

```bash
docker build -f container/Dockerfile -t tesorter2:2.0.0 .
```

Apptainer, from the repository root. This needs no Docker and works unprivileged on most HPC
systems (about 5 minutes, ~800 MB image):

```bash
apptainer build tesorter2_2.0.0.sif container/tesorter2.def
```

If it stops at once with `newuidmap was not found in PATH` (an unprivileged Apptainer, such as the
conda package, on a system that lists your user in `/etc/subuid`), add `--ignore-subuid`:

```bash
apptainer build --ignore-subuid tesorter2_2.0.0.sif container/tesorter2.def
```

With a Docker image already built, `apptainer build tesorter2_2.0.0.sif docker-daemon://tesorter2:2.0.0`
converts it instead.

The bundled protein databases are converted to BATH format while the image is built, so runs
start searching immediately even though the image is read-only.

## Run

```bash
# Docker: mount the working directory, run as your own user so the
# outputs are not owned by the container's user
docker run --rm -u $(id -u):$(id -g) -v "$PWD":/data tesorter2:2.0.0 \
    TEsorter2 TEs.fa -p 8

# TEsorter's own pipeline (TEsorter is an alias of TEsorter2)
docker run --rm -u $(id -u):$(id -g) -v "$PWD":/data tesorter2:2.0.0 \
    TEsorter2 TEs.fa -db rexdb-plant -p 8 --legacy

# Apptainer; -q hides its own notices (e.g. "fuse2fs not found")
apptainer -q exec tesorter2_2.0.0.sif TEsorter2 TEs.fa -p 8
```
