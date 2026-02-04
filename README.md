# Antibody-Antigen Binding Affinity Prediction

A multimodal deep learning pipeline for predicting antibody-antigen binding affinity by integrating protein sequences, AlphaFold 3 structural predictions, and interface contact maps.

## Highlights

- **606 antibody samples** with heavy and light chain sequences
- **7 immunogen targets** (influenza H3 variants including Hong Kong/1968)
- **AlphaFold 3 integration** for 3D structure prediction of antibody-antigen complexes
- **Automated candidate selection** using differential binding profiles
- **Multimodal transformer architecture** with cross-attention fusion (in development)

## Project Overview

This project develops a computational pipeline to predict antibody binding affinity from sequence and structure. The approach combines:

| Component | Description |
|-----------|-------------|
| **Sequence Analysis** | Heavy (VH) and light (VL) chain amino acid sequences |
| **Structure Prediction** | AlphaFold 3 complexes (mAb + antigen) |
| **Binding Data** | Quantitative binding measurements across immunogen panel |
| **Deep Learning** | Multimodal transformer with cross-attention fusion |

## Pipeline Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                      DATA PREPARATION                            │
├──────────────────────────────────────────────────────────────────┤
│  Antibody Sequences (H+L)  +  Binding Measurements  +  Antigens  │
└─────────────────────────────────┬────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                   STRUCTURE PREDICTION                           │
├──────────────────────────────────────────────────────────────────┤
│  AlphaFold 3: mAb (H+L) + Antigen → 3D Complex Structure         │
│  Outputs: Coordinates, PAE matrices, pLDDT confidence scores     │
└─────────────────────────────────┬────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                    FEATURE EXTRACTION                            │
├──────────────────────────────────────────────────────────────────┤
│  • Sequence embeddings (ESM-2)                                   │
│  • 3D coordinate features (GVP/SE3)                              │
│  • Interface contact maps                                        │
└─────────────────────────────────┬────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                 MULTIMODAL TRANSFORMER                           │
├──────────────────────────────────────────────────────────────────┤
│  Cross-attention fusion → Binding affinity prediction            │
└──────────────────────────────────────────────────────────────────┘
```

## Results

### Candidate Selection for Epitope Conformational Studies

Identified antibodies with selective binding profiles for structural analysis:

| Antibody | 1JPL-T_G3 Binding | 1JPL-WT Binding | Selectivity Ratio |
|----------|-------------------|-----------------|-------------------|
| P40G1 | 199,567 | 8.6 | **23,200x** |
| P38B10 | 28,281 | 4.3 | **6,600x** |
| P40A10 | 29,854 | 7.3 | **4,100x** |

These candidates show strong binding to the T_G3 variant while exhibiting minimal binding to wild-type, enabling comparative structural analysis of epitope conformations.

### AlphaFold 3 Complex Predictions

Successfully generated structural predictions for antibody-antigen complexes:
- 70+ input configurations prepared (top binders × immunogen panel)
- 3-chain complexes: Heavy chain + Light chain + Antigen
- Confidence metrics: PAE matrices and per-residue pLDDT scores

## Project Structure

```
ab_seq_bind_analysis/
├── notebooks/
│   ├── 01_dataset_prep.ipynb              # Data integration pipeline
│   ├── 02_af3_complex_input_prep.ipynb    # AF3 input generation
│   ├── 03_run_af3_prediction.ipynb        # Structure prediction runner
│   ├── 04_separate_chain_inputs.ipynb     # Chain folding analysis
│   ├── 05_modeling_candidate_selection.ipynb  # Differential binding selection
│   └── legacy/                            # Archived development notebooks
├── src/
│   ├── data/                              # Data loading and feature extraction
│   ├── models/                            # Transformer architecture
│   └── utils/                             # Helper functions
└── data/                                  # (not tracked - large files)
    ├── raw/                               # Original datasets
    └── processed/                         # AF3 inputs/outputs, features
```

## Immunogen Panel

| Immunogen | Description | Samples |
|-----------|-------------|---------|
| 1JPL-m4i4-C-G1 | Engineered 1JPL variant | 10 |
| 1JPL-m4i4-T-G3 | Engineered 1JPL variant | 10 |
| 1JPL-4i | Intermediate variant | 10 |
| 1JPL-WT | Wild-type reference | 10 |
| A/Hong Kong/1/1968 | H3N2 influenza | 10 |
| H3/Johannesburg/94 | H3N2 influenza | 10 |

## Technical Stack

- **Structure Prediction**: AlphaFold 3 (Singularity container)
- **Deep Learning**: PyTorch, PyTorch Geometric
- **Protein Language Models**: ESM-2
- **Data Processing**: pandas, BioPython
- **Visualization**: matplotlib, seaborn

## Usage

### Generate AF3 Inputs
```python
# See notebook 02_af3_complex_input_prep.ipynb
# Generates JSON files with H+L+Antigen sequences
```

### Run Structure Predictions
```bash
singularity exec --nv \
  -B /path/to/data:/data \
  alphafold3.sif \
  python run_alphafold.py \
    --json_path input.json \
    --output_dir output/
```

### Select Modeling Candidates
```python
# See notebook 05_modeling_candidate_selection.ipynb
# Identifies antibodies with differential binding profiles
```

## Future Directions

- [ ] Complete AF3 predictions for full candidate set
- [ ] Implement feature extraction from structural outputs
- [ ] Train multimodal transformer on binding affinity task
- [ ] Validate predictions on held-out test set
- [ ] Analyze learned attention patterns for epitope insights

## References

- Abramson, J. et al. (2024). Accurate structure prediction of biomolecular interactions with AlphaFold 3. *Nature*.
- Lin, Z. et al. (2023). Evolutionary-scale prediction of atomic-level protein structure with a language model. *Science*.

## License

See [LICENSE](LICENSE) for details.
