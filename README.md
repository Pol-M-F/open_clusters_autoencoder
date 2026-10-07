# Open Clusters Autoencoder

A neural network that estimates the properties of open star clusters from their colour-magnitude diagrams (CMDs). This repository contains the code, trained model and data of the Master's thesis:

> Musté Ferré, P. (2026). *Convolutional Autoencoders to estimate open cluster astrophysical parameters*. Master's thesis, Universitat de Barcelona. https://hdl.handle.net/2445/231630

The thesis explains the method, the results and their limits in detail. This README is a short guide to the code.

Author: Pol Musté Ferré. Advisors: Dr. Alfred Castro Ginard, Sagar Malhotra and Dr. Núria Miret Roig.

## What this project does

Open clusters are groups of stars that formed together. When their stars are plotted by colour and brightness (a CMD), the shape of the diagram depends on how old the cluster is, how far away it is, how many of its stars are binaries, and how massive it is.

The model is trained on simulated clusters, where these properties are known, and then applied to 42 real clusters observed by Gaia. For each cluster it predicts:

| Parameter | Unit |
|---|---|
| Distance | kpc |
| Age | log10 of the age in years |
| Binary fraction | between 0 and 1 |
| Total mass | log10 of the mass in solar masses |

## How it works

1. Each cluster is turned into two 100x100 images: the CMD star counts, and the inverse of the mean G-magnitude error in each CMD bin.
2. The encoder compresses both images into a 4-dimensional latent vector. The mean parallax and the number of stars of the cluster are added as two extra inputs.
3. The decoder rebuilds the CMD star counts from the latent vector (Poisson loss).
4. Four small linear layers read distance, age, binary fraction and mass from the same latent vector.

## Results

Mean absolute error (MAE) of the predictions, as reported in the thesis. These values change slightly every time the model is retrained, so retraining will not reproduce them exactly.

| Parameter | Simulated validation set | 42 real clusters |
|---|---|---|
| Distance (kpc) | 0.057 | 0.322 |
| log(Age) (dex) | 0.065 | 0.343 |
| Binary fraction | 0.094 | 0.230 |
| log(Mass) (dex) | 0.031 | 0.160 |

On simulated clusters the model recovers all four parameters well, except the binary fraction, which is the hardest to measure. On real clusters the mass stays reliable, while distance, age and binary fraction are mostly overestimated. The thesis discusses why, and how to improve it.

## Repository layout

```
.
├── README.md
├── autoencoder_code.ipynb       Notebook with all the code
├── autoencoder_model.pth        Trained model (weights and settings)
├── data/
│   ├── simulated_data/          Simulated clusters (.parquet), used for training and validation
│   └── real_data/               Real clusters: masses_q.csv and clus_sum.csv
└── plots/                       Figures from the training and testing sections
    └── cluster_results/         Figures and results table for the real clusters
```

## Requirements

Python 3 with the following packages:

```
pip install torch numpy pandas scipy scikit-learn matplotlib pyarrow jupyter
```

A GPU is optional. The notebook uses it automatically if one is available. Training for 250 epochs is much faster on a GPU.

## How to run

Open `autoencoder_code.ipynb` from the root folder of this repository and run the cells in order. All paths are relative to that folder.

| Section | Content | Needs |
|---|---|---|
| 1. Setup | Libraries, dataset classes, model and helper functions | Nothing |
| 2. Training and Evaluation | Trains the model and saves `autoencoder_model.pth` | `data/simulated_data/` |
| 3. Model Testing | Compares predictions with the true values for simulated clusters | `autoencoder_model.pth`, `data/simulated_data/` |
| 4. Real Open Clusters | Applies the model to the real clusters | `autoencoder_model.pth`, `data/real_data/` |

Section 4 does not need training. It only needs the saved model and the real data, so you can skip section 2 and use the provided `autoencoder_model.pth`.

## Data

### Simulated clusters

The thesis used 10,000 simulated clusters. This repository contains 5,000 of them (half), because of GitHub size restrictions. The simulated distances range from 150 pc to 4 kpc.

Each file in `data/simulated_data/` is one simulated cluster, with one row per star. The cluster parameters are read from the file name, which follows this pattern:

```
<IMF slope>_<distance in pc>_<...>_<log age>_<...>_<...>_<binary fraction>.parquet
```

For example, `-2.3_1000.8_0.0_7.3_0.0_0.0_0.52.parquet` is a cluster at 1000.8 pc with log age 7.3 and binary fraction 0.52. The notebook reads only the distance (second field), the log age (fourth field) and the binary fraction (last field). Each file must contain the columns `app_BP`, `app_RP`, `app_G`, `app_G_no_error`, `parallax` and `total_cluster_mass`.

### Real clusters

`data/real_data/` contains the catalogues of the 42 real open clusters:

- `masses_q.csv`: one row per star. Columns used: `Cluster`, `app_BP`, `app_RP`, `app_G`, `parallax`, `mass_A_weighted_mode`, `mass_A_mode`, `q_adopted`, `q_thresh_used`.
- `clus_sum.csv`: one row per cluster. Columns used: `Cluster`, `mean_dist` (pc), `mean_logage`, `n_stars_binaries`, `n_stars_MS`.

The real clusters are all closer than about 1.5 kpc, well inside the simulated distance range. The "true" values shown for the real clusters are reference values computed from these two catalogues. They are not exact ground truth.

## Output

- `plots/`: parameter distributions, loss curve, CMD reconstruction and prediction plots from training.
- `plots/cluster_results/`: one comparison figure per real cluster, `inference_results.csv` with the predictions for all clusters, and a scatter plot of predicted against reference values.

## Credits

- The simulated clusters were generated with the PARSEC-based simulation pipeline of Malhotra et al. (2026), provided by Sagar Malhotra.
- The 42 real open clusters come from the sample of Malhotra et al. (2026): Malhotra, S., Castro-Ginard, A., Anders, F., et al., *Stellar masses and mass ratios for Gaia open cluster members*, Astronomy & Astrophysics, 706, A62. doi: 10.1051/0004-6361/202557529.
