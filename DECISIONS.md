# Decision Log

Every meaningful choice on this project gets an entry. Target 5-10+ by project end.

**Template for each entry:**

```
## YYYY-MM-DD — <short title>

**Decision:** What did I choose?

**Options considered:** What were the alternatives?

**Reason:** Why this one? Constraints, trade-offs, data that pushed me this way.

**Revisit if:** What would make me change my mind? (optional but strong signal of rigor)
```

---

## 2026-04-16 — Dataset: GSE106305 over GSE188706

**Decision:** Primary dataset is GSE106305 (Guo et al. 2019, *Cancer Cell*, PMID 30655535), prostate cancer hypoxia study, 16 samples across LNCaP and PC3 cell lines in 2×2 factorial design.

**Options considered:**
- GSE188706 (Pal et al. 2022, MCF-7 breast cancer + G1 agonist, 9 samples)
- Several others returned by initial search that turned out to be mis-attributed by an LLM subagent (e.g. GSE240736 is Arabidopsis, not cancer).

**Reason:** GSE106305 has a clean factorial design that maps directly to a DESeq2 `~ cell_line + condition + cell_line:condition` model; published in a top-tier cancer journal; 16 samples is enough to show real statistics but small enough to pseudo-align on a laptop; hypoxia signature recovery is a well-characterized positive control for GSEA.

**Revisit if:** Samples turn out to be single-ended in a way that blocks downstream tximport, or if the published DE gene list is unrecoverable from SRA metadata.

---

## 2026-04-16 — Aligner: Salmon over STAR

**Decision:** Use Salmon pseudo-alignment via nf-core/rnaseq's `--aligner salmon` path.

**Options considered:** STAR (full splice-aware alignment), HISAT2, kallisto.

**Reason:** 32GB RAM laptop cannot run STAR against the full human genome index (~30GB resident). Salmon's transcriptome index fits in ~4-8GB and runs in ~10-20 min/sample on a modern Mac. Loses some splicing/novel-transcript info but irrelevant for a bulk DE re-analysis.

**Revisit if:** Reviewers/interviewers specifically ask "why not STAR" — answer is prepared, but consider running a single-sample STAR comparison as a side-car if time allows.

---

## 2026-04-16 — Pipeline architecture: nf-core upstream + custom DSL2 downstream

**Decision:** Use nf-core/rnaseq unmodified for QC → alignment → quantification. Write custom DSL2 for tximport → DESeq2 → fgsea → figure recreation.

**Options considered:**
- Fork and modify nf-core/rnaseq end-to-end
- Use nf-core/differentialabundance downstream
- Write the whole pipeline from scratch

**Reason:** nf-core/rnaseq is a production-grade reference; forking it signals inexperience, not depth. nf-core/differentialabundance would hide exactly the DSL2 authorship I need to show. Writing the upstream from scratch is a six-month project. Custom downstream gets me authentic DSL2 code (channels, processes, operators, configs) without the cargo-cult trap.

**Revisit if:** Downstream turns out trivial enough that it doesn't demonstrate meaningful DSL2 authorship — at which point add a custom fusion-detection or variant-calling sidecar.

---

## 2026-04-16 — Phase 0 environment: tool versions and config choices

**Decision:** Lock the following local environment for development:

| Tool | Version | Install path | Method |
|---|---|---|---|
| Nextflow | 25.10.4 (build 11173) | `~/.local/bin/nextflow` | `curl -s https://get.nextflow.io \| bash` |
| Java | OpenJDK 17.0.18 (Homebrew) | `/opt/homebrew/opt/openjdk@17/` | `brew install openjdk@17` |
| Docker Desktop | 4.40.0 / Engine 28.0.4 | `/Applications/Docker.app` | Pre-existing install, first-launched today |
| Git | 2.47.1 | `/opt/homebrew/bin/git` | Homebrew (pre-existing) |

Config changes to `~/.zshrc`:
- `JAVA_HOME` set to `/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home`
- `$JAVA_HOME/bin` prepended to `PATH` (overrides system Java 24, which Nextflow rejects)

Docker VM resources: 10 CPUs, 7.65 GiB RAM (default). Sufficient for `-profile test`; will increase to ~16 GiB for the real 16-sample run.

Phase 0 smoke test: `nextflow run nf-core/rnaseq -profile test,docker --outdir results/test` completed successfully — 219 tasks, 0 failures, 14m 47s. Pipeline revision was `master` at `47b3b0d3da` (unpinned); real run will pin a specific release tag.

**Options considered:**
- Java: Eclipse Temurin via `brew install --cask temurin@17` (rejected: cask runs a `.pkg` installer requiring `sudo`; Homebrew `openjdk@17` formula installs without admin privileges and is the same OpenJDK 17 source).
- Nextflow install location: `brew install nextflow` into `/opt/homebrew/bin/`, or `nextflow self-update` in-place at `/usr/local/bin/` (rejected: `/usr/local/bin/` is root-owned on this M1 Mac, a legacy Intel-prefix remnant; `~/.local/bin/` is user-writable and already first on `PATH`).
- Docker Desktop upgrade to 4.69.0 (rejected: 4.40.0 / Engine 28.x is fully compatible; upgrading adds no concrete benefit for this project).

**Reason:** Every choice prioritized a working environment with minimal `sudo` and zero ambiguity. OpenJDK 17 is the nf-core-recommended LTS version. Nextflow 25.10.4 is required because nf-core/rnaseq `master` depends on `nf-schema@2.5.1` (Nextflow ≥25.04.0). A stale Nextflow 24.10.5 remains at `/usr/local/bin/` but is shadowed by PATH ordering.

**Revisit if:** Real 16-sample run hits Docker VM memory limits (bump to 16 GiB). The stale `/usr/local/bin/nextflow` can be cleaned up with `sudo rm` if it causes confusion. Pipeline revision must be pinned before Phase 1.

---

<!-- New entries go below this line, most recent at the top. -->
