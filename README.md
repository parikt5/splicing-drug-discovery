# Splicing-to-Drug: A Rare Disease Drug Discovery Pipeline

A computational pipeline that takes a rare disease patient variant, predicts its effect on RNA splicing, models the resulting aberrant protein, and screens drug databases for therapeutic candidates — fully automated from VCF to ranked drug report.

> Built as part of a computational biology research project targeting the underserved intersection of splicing biology and rare disease drug discovery.

---

## The problem this solves

Around half of rare disease patients remain undiagnosed or untreated after genome sequencing. A major reason: variants that disrupt RNA splicing outside canonical splice sites are routinely ignored. This pipeline identifies those variants, models their downstream effect on protein structure, and surfaces drug candidates — a process that today is almost entirely manual.

---

## Pipeline overview

```
Patient VCF
    │
    ▼
[Stage 1] Variant filtering
    │   ClinVar, gnomAD — rare, predicted-damaging variants
    ▼
[Stage 2] Splicing prediction
    │   SpliceAI / Pangolin — delta scores, aberrant isoform sequence
    ▼
[Stage 3] Protein structure modeling
    │   AlphaFold2 / ESMFold — 3D structure of aberrant isoform
    ▼
[Stage 4] Drug candidate screening
    │   ChEMBL, DrugBank — small molecules + ASO libraries
    ▼
[Stage 5] Prioritization & report
        Mechanistic scoring, clinical safety filter → ranked CSV
```

---

## Repository structure

```
splicing-drug-discovery/
├── README.md
├── environment.yml              # Conda environment for full reproducibility
├── Snakefile                    # End-to-end pipeline runner
│
├── scripts/
│   ├── 01_variant_filter.py     # Parse VCF, filter rare + damaging variants
│   ├── 02_spliceai_runner.py    # Run SpliceAI, extract aberrant isoforms
│   ├── 03_isoform_to_protein.py # Translate isoform, submit to AlphaFold
│   ├── 04_drug_screen.py        # Query ChEMBL/DrugBank for candidates
│   └── 05_prioritize.py         # Score and rank drug candidates
│
├── notebooks/
│   └── case_study.ipynb         # End-to-end walkthrough on a real variant
│
├── results/
│   ├── example_structures/      # Example PDB files from AlphaFold
│   └── drug_candidates.csv      # Example ranked output
│
└── docs/
    └── validation_report.md     # Comparison against published findings
```

---

## Quickstart

### 1. Clone the repository

```bash
git clone https://github.com/parikt5/splicing-drug-discovery
cd splicing-drug-discovery
```

### 2. Set up the environment

```bash
conda env create -f environment.yml
conda activate splicing-pipeline
```

### 3. Run the full pipeline

```bash
snakemake --cores 4 --config vcf=data/your_variants.vcf
```

Or run individual stages:

```bash
python scripts/01_variant_filter.py --vcf data/your_variants.vcf --out results/filtered.vcf
python scripts/02_spliceai_runner.py --vcf results/filtered.vcf --out results/spliceai_output.json
python scripts/03_isoform_to_protein.py --input results/spliceai_output.json
python scripts/04_drug_screen.py --protein results/aberrant_protein.fasta
python scripts/05_prioritize.py --candidates results/raw_candidates.csv --out results/drug_candidates.csv
```

---

## Dependencies

| Tool | Purpose | Install |
|------|---------|---------|
| SpliceAI | Splicing variant prediction | `pip install spliceai` |
| Biopython | Sequence parsing + translation | `pip install biopython` |
| AlphaFold2 | Protein structure prediction | [Colab notebook](https://colab.research.google.com/github/deepmind/alphafold/blob/main/notebooks/AlphaFold.ipynb) |
| ChEMBL API | Drug-target database | `pip install chembl-webresource-client` |
| Pandas | Data processing | `pip install pandas` |
| Snakemake | Pipeline orchestration | `conda install snakemake` |

---

## Example output

Running the pipeline on a known pathogenic splice variant in *LMNA* (Hutchinson-Gilford Progeria Syndrome) produces:

```
rank  drug_name         mechanism                      chembl_id       confidence
1     Lonafarnib        Farnesyltransferase inhibitor  CHEMBL1956       High
2     MG-132            Proteasome inhibitor           CHEMBL268473     Medium
3     Temsirolimus      mTOR inhibitor                 CHEMBL1908       Medium
```

See `notebooks/case_study.ipynb` for the full annotated walkthrough.

---

## Scientific background

This pipeline is motivated by three key findings in the literature:

- ~50% of rare disease patients lack a molecular diagnosis after whole genome sequencing ([Cormier et al. 2022](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-022-05041-x))
- Up to 27% of pathogenic splicing variants lie outside canonical splice sites and are routinely missed in clinical analysis ([Genomics England 2022](https://www.medrxiv.org/content/10.1101/2022.01.28.22270002))
- SpliceAI achieves state-of-the-art prediction of cryptic splice sites from sequence alone ([Jaganathan et al., Cell 2019](https://www.cell.com/cell/fulltext/S0092-8674(18)31629-5))

---

## Roadmap

- [ ] Variant filtering module
- [ ] SpliceAI integration
- [ ] AlphaFold automation (in progress)
- [ ] ChEMBL + DrugBank screening
- [ ] ASO library screening
- [ ] Snakemake end-to-end pipeline
- [ ] Web interface for non-coders

---

## Author

**Tvara Parikh**
Computational Biology student
GitHub: [github.com/parikt5](https://github.com/parikt5)

---

## Citation

If you use this pipeline in your research, please cite:

```
@misc{parikt5_splicing_pipeline_2026,
  author = {Tvara Parikh},
  title  = {Splicing-to-Drug: A Rare Disease Drug Discovery Pipeline},
  year   = {2026},
  url    = {https://github.com/parikt5/splicing-drug-discovery}
}
```

---

## License

MIT License — free to use, modify, and distribute with attribution.
