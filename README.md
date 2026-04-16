# nextflow-rnaseq

Reproducing the bulk RNA-seq analysis from Guo et al. 2019 (*Cancer Cell*, PMID 30655535) end-to-end using Nextflow DSL2.

**Status:** Phase 1 in progress — pipeline revision pinning and data staging.

## Overview

- **Upstream:** [nf-core/rnaseq](https://nf-co.re/rnaseq) v3.14+ with Salmon pseudo-alignment.
- **Downstream:** Custom Nextflow DSL2 pipeline — tximport → DESeq2 → fgsea → figure recreation.
- **Dataset:** GSE106305 — 16 RNA-seq samples, LNCaP and PC3 prostate cancer cell lines, ±hypoxia (2×2 factorial).

## What this project demonstrates

An end-to-end reproduction of a published cancer RNA-seq analysis: raw FASTQs through QC, quantification, differential expression, and pathway enrichment, packaged as a reproducible Nextflow pipeline. The design decisions — aligner choice, filtering thresholds, statistical model — are logged in [`DECISIONS.md`](./DECISIONS.md) with reasons, so a reviewer can see *why* each choice was made, not just *what* was run.

## Quick start (placeholder — will flesh out as pipeline stabilizes)

```bash
# upstream (nf-core/rnaseq)
nextflow run nf-core/rnaseq \
  -profile docker \
  --input samplesheet.csv \
  --outdir results/rnaseq \
  --aligner salmon \
  --genome GRCh38

# downstream (custom)
nextflow run main.nf \
  -profile docker \
  --quant_dir results/rnaseq/salmon \
  --outdir results/downstream
```

## Repository layout

```
nextflow-rnaseq/
├── CLAUDE.md            # Project briefing for Claude Code (not user-facing)
├── DECISIONS.md         # Decision log — read this to understand the "why"
├── README.md            # You are here
├── main.nf              # Custom downstream pipeline entry point
├── nextflow.config      # Project-level config
├── conf/                # Profile-specific config
├── modules/             # DSL2 process modules
├── subworkflows/        # Subworkflow definitions
├── bin/                 # Scripts invoked by processes
├── assets/              # Sample sheets, reference lists
├── work/                # Nextflow work dir (gitignored)
├── results/             # Pipeline outputs (gitignored if large)
└── data/                # Symlink to external SSD FASTQs (gitignored)
```

## Dataset

Guo et al. 2019, *Cancer Cell* — prostate cancer hypoxia study (LNCaP and PC3 cell lines). PMID: [30655535](https://pubmed.ncbi.nlm.nih.gov/30655535/). GEO accession: [GSE106305](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE106305). Full citation will be confirmed directly from PubMed before the repo goes public.

## Reproducibility

Container images are pinned in `nextflow.config`. All tool versions are in `DECISIONS.md`. The `work/` directory from a clean run will be preserved as a single snapshot for traceability.
