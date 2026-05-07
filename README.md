# BlasCh — BLAST-based chimera recovery

**BlasCh** detects and recovers false positive chimeric sequences from metabarcoding and environmental DNA (eDNA) datasets. It uses BLAST alignment against per-sample self-databases (and optionally a reference database) to classify sequences flagged as chimeras by upstream denoisers (DADA2, VSEARCH, etc.) into four categories:

| Category | Meaning |
|---|---|
| **non_chimeric** | High-confidence true sequence — rescued |
| **borderline** | Moderate confidence — optionally rescued |
| **multiple_alignments** | Fragmented alignment pattern — likely chimeric |
| **chimeric** | Confirmed false positive |

---

## Requirements

- Python ≥ 3.8
- [NCBI BLAST+](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download) (`blastn` and `makeblastdb` must be in `$PATH`)
- Python packages: `biopython`, `psutil`

```bash
pip install biopython psutil
```

---

## Installation

```bash
git clone https://github.com/alihkz94/BlasCh.git
cd BlasCh
pip install -r requirements.txt
```

No compilation or installation step is required — `blasch.py` is a self-contained script.

---

## Input requirements

BlasCh expects the output of a standard chimera detection step.

| Argument | Description |
|---|---|
| `--input_chimeras_dir` | Directory containing per-sample chimera FASTA files. Files must follow the naming `<sample>.<ext>` or `<sample>.chimeras.<ext>`, where `<ext>` is `.fasta`, `.fa`, or `.fas`. |
| `--self_fasta_dir` | Directory with the original (pre-chimera-detection) FASTA files for building per-sample self-databases. BlasCh auto-detects files and prioritises original sample files over `.chimeras` files. |
| `--reference_db` |  Path to a BLAST database prefix or a FASTA file. If a FASTA is provided, BlasCh builds the database automatically. |
| `--nonchimeric_dir` | *(Optional)* Directory of per-sample non-chimeric FASTA files (from the denoiser). When provided, rescued sequences are merged with these into `nonchimeric+rescued_reads/`. |

---

## Usage

```bash
python blasch.py \
    --input_chimeras_dir /path/to/chimeras \
    --self_fasta_dir     /path/to/original_fastas \
    --reference_db       /path/to/reference.fasta \
    --output_dir         ./blasch_output \
    --threads            16 \
    --high_identity_threshold        99.0 \
    --high_coverage_threshold        99.0 \
    --borderline_identity_threshold  80.0 \
    --borderline_coverage_threshold  89.0
```

### All arguments

| Argument | Default | Description |
|---|---|---|
| `--input_chimeras_dir` | `./` | Directory with `.chimeras` FASTA files |
| `--self_fasta_dir` | `./` | Directory with original FASTA files for self-databases |
| `--reference_db` | — | Reference DB prefix or FASTA file (optional) |
| `--output_dir` | `./blasch_output` | Output directory |
| `--threads` | `8` | BLAST threads |
| `--high_identity_threshold` | `99.0` | Identity (%) for high-confidence classification |
| `--high_coverage_threshold` | `99.0` | Coverage (%) for high-confidence classification |
| `--borderline_identity_threshold` | `80.0` | Identity (%) for borderline rescue |
| `--borderline_coverage_threshold` | `89.0` | Coverage (%) for borderline rescue |
| `--nonchimeric_dir` | — | Pre-existing non-chimeric reads for merging (optional) |
| `--version` | — | Print version and exit |

---

## Classification logic

```
For each sequence in each .chimeras file:

1. Run BLAST against self-database (+ reference DB if provided)
2. Inspect the best non-self hit:
   a. Multiple HSPs AND coverage ≤ 85%  →  chimeric
   b. Multiple HSPs AND coverage > 85%  →  multiple_alignments
3. No non-self hits at all              →  non_chimeric
4. Only self-sample hits                →  chimeric
5. identity ≥ HIGH and coverage ≥ HIGH  →  non_chimeric
6. identity ≥ BORDERLINE and coverage ≥ BORDERLINE  →  non_chimeric (rescued)
7. Single taxonomy, no threshold met    →  borderline
8. Multiple taxonomies, no threshold met  →  chimeric
```

---

## Output structure

```
blasch_output/
├── non_chimeric/                   # Rescued sequences (rules 3, 5, 6)
├── borderline/                     # Borderline sequences (rule 7)
├── nonchimeric+rescued_reads/      # (only if --nonchimeric_dir provided)
│                                   # per-sample merge of denoiser non-chimeric + rescued
├── detailed_results/
│   ├── *_chimeric.fasta            # Confirmed chimeras
│   ├── *_multiple_alignments.fasta
│   └── *_sequence_details.csv      # Per-sequence classification table
├── xml/
│   └── blast_results.zip           # Compressed BLAST XML (kept for reanalysis)
├── chimera_recovery_report.txt     # Overall summary statistics
└── README.txt                      # Run-specific documentation with parameters
```

### Smart rerun

If `xml/blast_results.zip` (or uncompressed XML files) already exist from a previous run, BlasCh **skips the BLAST step entirely**. You can re-run classification with different thresholds in seconds without re-running BLAST.

---

## Dependencies to cite

- **BLAST+**: Camacho, C., Coulouris, G., Avagyan, V., Ma, N., Papadopoulos, J., Bealer, K., & Madden, T. L. (2009). BLAST+: architecture and applications. *BMC Bioinformatics*, 10, 421. https://doi.org/10.1186/1471-2105-10-421
- **Biopython**: Cock, P. J. A., et al. (2009). Biopython: freely available Python tools for computational molecular biology and bioinformatics. *Bioinformatics*, 25(11), 1422–1423. https://doi.org/10.1093/bioinformatics/btp163

---

## Integration with PipeCraft2

BlasCh is also available as a module within [PipeCraft2](https://github.com/SuvalineVana/pipecraft), a GUI-based pipeline for metabarcoding data processing. This standalone version is provided for users who want to run BlasCh independently.

---

## Citation

If you use BlasCh, please cite:

> Hakimzadeh A, Mikryukov V, Metsoja M, Tedersoo L, Anslan S. 2025. Are we throwing away good data? Evaluation of chimera detection algorithms on long-read amplicons reveals high false-positive rates across algorithms. *PeerJ* 13:e20456. https://doi.org/10.7717/peerj.20456

A `CITATION.cff` file is included for automated citation export via GitHub's **Cite this repository** button.

---

## License

MIT — see [LICENSE](LICENSE).
