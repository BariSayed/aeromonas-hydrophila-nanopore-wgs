# aeromonas-hydrophila-nanopore-wgs

Long-read whole-genome assembly and annotation of *Aeromonas hydrophila* from publicly
available Oxford Nanopore sequencing data, using the nf-core/bacass pipeline.

## Overview

This repository documents a reproducible long-read assembly and annotation workflow applied
to secondary (publicly archived) data. No new sequencing was generated. The input reads were
obtained from the European Nucleotide Archive (ENA); the pipeline, configuration files, run
script, and summary outputs are provided so the analysis can be reproduced.

## Input data

| | |
|---|---|
| Organism | *Aeromonas hydrophila* |
| Sequencing platform | Oxford Nanopore (MinION) |
| ENA run accession | [`SRR38923685`](https://www.ebi.ac.uk/ena/browser/view/SRR38923685) |
| Reads | 133,986 |
| Read N50 | 16,008 bp |
| Approximate coverage | ~201x |

## Workflow

Pipeline: **nf-core/bacass 2.6.1**, executed with **Nextflow** and **Apptainer** containers.

1. Read QC and adapter trimming — Porechop and NanoPlot
2. De novo assembly — Flye (`--nano-hq`)
3. Long-read polishing — Medaka
4. Assembly QC — QUAST (contiguity) and BUSCO (`bacteria_odb10`, gene completeness)
5. Annotation — Prokka
6. Aggregate report — MultiQC

The assembly was polished with Medaka using the same long reads.

### Running the workflow

```bash
nextflow run nf-core/bacass -r 2.6.1 \
  -profile apptainer \
  -c config/run.config \
  -params-file config/params.yaml \
  -work-dir work -resume
```

Parameters are defined in `config/params.yaml`. The sample sheet (`config/samplesheet.tsv`)
provides long reads only, so the short-read and Fast5 columns are set to `NA`.

## Results

| Metric | Value |
|---|---|
| Contigs | 4 |
| Total length | 5,431,448 bp |
| Largest contig | 5,332,215 bp |
| N50 | 5,332,215 bp |
| GC content | 60.26% |
| BUSCO completeness | C:99.2%[S:97.6%,D:1.6%],F:0.0%,M:0.8% (bacteria_odb10, n=124) |
| CDS (Prokka) | 4,947 |
| tRNA / rRNA | 130 / 31 |

The final assembly is provided at `results/assembly/aeromonas_nanopore-flye-medaka_polished_genome.fa.gz`. Full QC and annotation outputs are in
`results/` (`qc/`, `annotation/`, and `multiqc_report.html`).

## Downstream characterization

The assembly was further characterized with standard in-silico tools (StaPH-B
containers). All results are screening of publicly archived sequence data and are
reported without clinical interpretation.

| Analysis | Tool | Result |
|---|---|---|
| Species identity (ANI vs *A. hydrophila* ATCC 7966<sup>T</sup>) | fastANI | **97.02%** average nucleotide identity (> 95% species threshold) |
| Sequence type | mlst (PubMLST *aeromonas* scheme) | **ST709** |
| Acquired / intrinsic AMR genes | AMRFinderPlus | aac(6')-Va, aph(3'')-Ib, aph(6)-Id, bla_OXA, cepS, cphA, floR, sul2, tet(A) |
| Virulence genes | ABRicate / VFDB | 157 hits, including aerolysin (*aerA/act*) |
| Plasmid / mobile content | Flye circularity + PlasmidFinder | contig_5 (37,631 bp, circular) plus contig_2 (43,007 bp) and contig_4 (18,595 bp); no PlasmidFinder replicon matched the database |

The two genomes in these two repositories share 96.95% ANI and carry different
sequence types, consistent with distinct *A. hydrophila* isolates. Full tables are
in `results/downstream/` (`mlst/`, `amr_vir/`, `plasmid/`).

## Repository contents

```
config/        params.yaml, samplesheet.tsv, run.config
scripts/       run script for the pipeline
results/       assembly FASTA, QUAST and BUSCO summaries, read statistics,
               Prokka annotation, MultiQC report
results/downstream/  fastANI, MLST, AMR/virulence (ABRicate, AMRFinderPlus),
               plasmid replicon and Flye circularity
```

Raw reads and large intermediate files (Nextflow `work/`, full `04_results/`) are not tracked;
see `.gitignore`.

## Tools and versions

Nextflow 26.04 · nf-core/bacass 2.6.1 · Apptainer 1.1.9 · Flye 2.9.5 · Medaka, QUAST 5.3.0 ·
BUSCO 5.8.3 (bacteria_odb10) · Prokka · MultiQC 1.32 · seqkit 2.1.0 · fastANI · mlst · ABRicate · AMRFinderPlus (StaPH-B containers)

## Data availability and reproducibility

Input reads are publicly available from ENA under accession `SRR38923685`:

```bash
wget -c https://ftp.sra.ebi.ac.uk/vol1/fastq/SRR389/085/SRR38923685/SRR38923685_1.fastq.gz
```

## License

Released under the MIT License. Input sequence data are public ENA records.
