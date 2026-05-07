# Changelog

All notable changes to BlasCh are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and
this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-05-07

First public release of BlasCh as a standalone tool.

### Features
- Per-sample BLAST self-database construction from input FASTA files
- Optional reference database integration (accepts pre-built BLAST DB or raw FASTA)
- Four-category sequence classification:
  - **Non-chimeric** — high-confidence match to DB or self-sample
  - **Borderline** — moderate match, optionally rescued
  - **Multiple alignments** — first non-self hit spans multiple HSPs (fragmented alignment)
  - **Chimeric** — confirmed false positive
- Smart rerun: detects existing BLAST XML results (raw or zipped) and skips BLAST step
- Optional merge step: combines input non-chimeric reads with BlasCh-rescued sequences per sample
- Parallel XML parsing using `multiprocessing`
- Per-run README and CSV sequence-level detail reports
- XML compression to ZIP after processing to save disk space
- Automatic BLAST database cleanup after run

### Notes
- Developed within the [PipeCraft2](https://github.com/SuvalineVana/pipecraft) pipeline
  and extracted here as a standalone, independently citable tool
- Tested on Illumina amplicon datasets (16S, ITS, COI)
