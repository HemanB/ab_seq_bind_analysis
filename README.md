# Antibody-Antigen Binding Prediction

Transformer-based prediction of antibody-antigen binding affinity using multimodal representations. This project combines molecular graphs, 3D structural coordinates, and interface contact maps with cross-attention fusion to predict binding interactions.

## Overview

This project develops a multimodal transformer architecture for predicting antibody-antigen binding affinity (n=606 antibody samples). The approach integrates multiple molecular representations:

| Modality | Description |
|----------|-------------|
| **Molecular Graphs** | Atom/residue connectivity with chemical features |
| **3D Coordinates** | Spatial structure from AlphaFold 3 predictions |
| **Contact Maps** | Residue-residue distances at the binding interface |

These modalities are fused using cross-attention mechanisms to learn structure-function relationships.

## Project Structure

```
ab_seq_bind_analysis/
├── notebooks/
│   ├── 01_dataset_prep.ipynb           # Data cleaning and merging
│   ├── 02_af3_complex_input_prep.ipynb # AF3 input generation
│   ├── 03_run_af3_prediction.ipynb     # Structure prediction
│   └── legacy/                         # Archived notebooks
├── src/
│   ├── data/                           # Data loading, feature extraction
│   ├── models/                         # Transformer architecture
│   └── utils/                          # Helper functions
├── data/                               # (gitignored)
│   ├── raw/                            # Original antibody/binding data
│   └── processed/
│       ├── af3_inputs/                 # AlphaFold 3 input JSONs
│       ├── af3_outputs/                # Predicted structures
│       └── features/                   # Extracted features (graphs, embeddings)
└── README.md
```

## Data

### Input Data
- **Antibody sequences**: 606 samples with heavy (H) and light (L) chain sequences
- **Binding measurements**: Quantitative binding against 7 immunogens (influenza H3 variants)
- **Immunogen sequences**: 5 antigen sequences for structural modeling

### Immunogen Panel
- 1JPL variants (m414_C_G1, m414_T_G3, 4I, WT)
- H3 influenza strains (Hong Kong/1968, Johannesburg/94)

## Pipeline

### 1. Data Preparation
`notebooks/01_dataset_prep.ipynb`
- Merge antibody sequences with binding measurements
- Quality control and filtering

### 2. Structure Prediction
`notebooks/02_af3_complex_input_prep.ipynb` & `03_run_af3_prediction.ipynb`
- Generate AlphaFold 3 input files (H + L + Antigen complexes)
- Run structure predictions via Singularity

### 3. Feature Extraction (WIP)
`src/data/`
- Extract molecular graphs from sequences
- Parse 3D coordinates from AF3 outputs
- Compute interface contact maps

### 4. Model Training (WIP)
`src/models/`
- Multimodal transformer with cross-attention fusion
- Binding affinity regression

## Model Architecture

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Sequence   │  │     3D      │  │   Contact   │
│   Encoder   │  │  Encoder    │  │    Map      │
│  (ESM-2)    │  │  (GVP/SE3)  │  │   Encoder   │
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │
       └────────┬───────┴────────┬───────┘
                │                │
         ┌──────▼────────────────▼──────┐
         │     Cross-Attention Fusion    │
         └──────────────┬───────────────┘
                        │
                 ┌──────▼──────┐
                 │  Prediction │
                 │    Head     │
                 └─────────────┘
```

## Requirements

Key dependencies (see `requirements.txt`):
- PyTorch
- PyTorch Geometric
- ESM (protein language model)
- BioPython
- AlphaFold 3 (for structure prediction)

## Usage

### Running Notebooks
```bash
cd notebooks
jupyter notebook 01_dataset_prep.ipynb
```

### Running AF3 Predictions
```bash
singularity exec --nv \
  -B /cwork:/cwork \
  -B /path/to/AF3_dbs:/path/to/AF3_dbs \
  alphafold3.sif \
  python /app/alphafold/run_alphafold.py \
    --json_path data/processed/af3_inputs/sample.json \
    --output_dir data/processed/af3_outputs/sample \
    --db_dir /path/to/AF3_dbs
```

## License

See [LICENSE](LICENSE) for details.
