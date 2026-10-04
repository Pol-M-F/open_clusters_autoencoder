# Open Clusters Autoencoder

A neural network that estimates the properties of open star clusters from their colour-magnitude diagrams (CMDs), built for a Master's thesis (TFM) in Astrophysics, 2025/2026.

Author: Pol Musté Ferré

## What this project does

Open clusters are groups of stars that formed together. When their stars are plotted by colour and brightness (a CMD), the shape of the diagram depends on how old the cluster is, how far away it is, how many of its stars are binaries, and how massive it is.

This project trains a convolutional autoencoder on simulated clusters, where these properties are known, and then applies it to real clusters observed by Gaia. For each cluster the model predicts:

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

Each file in `data/simulated_data/` is one simulated cluster, with one row per star. The cluster parameters are read from the file name, which must follow this pattern:

```
<name>_<distance in pc>_<...>_<log age>_..._<binary fraction>.parquet
```

The distance is the second field, the log age is the fourth field, and the binary fraction is the last field. Each file must contain the columns `app_BP`, `app_RP`, `app_G`, `app_G_no_error`, `parallax` and `total_cluster_mass`.

### Real clusters

`data/real_data/` contains the catalogues of the 42 real open clusters:

- `masses_q.csv`: one row per star. Columns used: `Cluster`, `app_BP`, `app_RP`, `app_G`, `parallax`, `mass_A_weighted_mode`, `mass_A_mode`, `q_adopted`, `q_thresh_used`.
- `clus_sum.csv`: one row per cluster. Columns used: `Cluster`, `mean_dist` (pc), `mean_logage`, `n_stars_binaries`, `n_stars_MS`.

The "true" values shown for the real clusters are reference values computed from these two catalogues. They are not exact ground truth.

## Output

- `plots/`: parameter distributions, loss curve, CMD reconstruction and prediction plots from training.
- `plots/cluster_results/`: one comparison figure per real cluster, `inference_results.csv` with the predictions for all clusters, and a scatter plot of predicted against reference values.

## Credits

- The algorithm used to generate the simulated clusters was provided by Sagar Malhotra.
- The 42 real open clusters come from the Gaia DS3 sample list provided by Sagar Malhotra.
