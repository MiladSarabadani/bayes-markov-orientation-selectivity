# Bayes–Markov cortical orientation selectivity

Reproducible simulations accompanying the manuscript:

> **A Bayes–Markov Model of Cortical Orientation Selectivity: Computational Re-implementation, Quantitative Evaluation, and Spiking Readouts**<br>
> Abolfazl Moslemi, Milad Sarabadani, Fatemeh Sefidian, and Hossein Peyvandi

This repository provides a self-contained implementation and quantitative evaluation of
the Bayes-Markov orientation-selectivity model introduced by Shirazi (2004). It includes
pixel-wise and vectorized rate-model implementations and downstream leaky
integrate-and-fire (LIF) and Hodgkin-Huxley (HH) spiking readouts.

## Repository contents

| File | Purpose |
|---|---|
| [`bayes_markov_orientation_selectivity.ipynb`](bayes_markov_orientation_selectivity.ipynb) | Complete implementation, experiments, benchmarks, figures, and numerical checks |
| [`requirements.txt`](requirements.txt) | Pinned Python dependencies used for the validated run |
| [`manuscript_figures/`](manuscript_figures/) | Publication-ready copies of code-generated manuscript Figures 2–15 |
| [`CITATION.cff`](CITATION.cff) | Software and manuscript citation metadata |
| [`LICENSE`](LICENSE) | MIT License |

The repository notebook is self-contained and is the authoritative implementation.

## Scientific scope

The Bayes-Markov stage infers a binary SCI maximum-a-posteriori (MAP) field. The LIF and
HH models receive current derived from that inferred field and express it through spikes.
They are therefore **downstream spiking readouts**: their dynamics do not perform the MAP
inference or generate the selective field directly from raw LGN input.

The spiking analyses use spike counts and mean firing rates. They do not test whether
spike latency, order, synchrony, or another timing variable carries information beyond
spike count, and no temporal-code claim is made.

## Validated environment

The repository notebook was clean-run with:

- Python 3.10.7
- NumPy 1.25.1
- SciPy 1.12.0
- Matplotlib 3.8.2

All 31 code cells executed sequentially without exceptions, all numerical assertions
passed, and all 14 code-generated manuscript figures were reproduced.

## Installation

Run the following commands from the repository root:

```bash
python3.10 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m ipykernel install --user --name bayes-markov-v1 --display-name "Bayes-Markov V1"
```

On Windows PowerShell, activate the environment with:

```powershell
.venv\Scripts\Activate.ps1
```

The scientific cells can run with standard Windows Python, but the isolated benchmark
worker imports the Unix `resource` module. Run the complete benchmark on macOS or Linux,
or use WSL2 on Windows.

## Reproducing the results

For an interactive run:

```bash
jupyter lab
```

Open `bayes_markov_orientation_selectivity.ipynb`, select the **Bayes-Markov V1** kernel,
and run all cells from top to bottom.

For a non-interactive clean run from the repository root:

```bash
jupyter nbconvert \
  --to notebook \
  --execute \
  --inplace \
  --ExecutePreprocessor.kernel_name=bayes-markov-v1 \
  --ExecutePreprocessor.timeout=7200 \
  bayes_markov_orientation_selectivity.ipynb
```

The benchmark launches isolated Python subprocesses and expects the notebook to retain
its published filename, so execute it from the repository root without renaming it.
Generated plots are written to `figures/`, while publication-ready copies of manuscript
Figures 2–15 are written to `manuscript_figures/`. The first directory is ignored by Git;
the second is tracked so readers can inspect the final figures without rerunning the
notebook. A complete run is CPU-intensive and may take 30–60 minutes or longer,
depending on the machine.

## Correctness and benchmark checks

The inactive-state recurrence in the baseline `V1Model` evaluates the actual neighboring
SCI state, consistently with the model equation. The vectorized baseline implements the
same recurrence.

The notebook verifies:

- exact equality between pixel-wise and vectorized outputs across deterministic and
  random test cases;
- direct active-state and silent-state recurrence increments;
- exact equality at $16\times16$, $64\times64$, and $128\times128$ benchmark sizes;
- one warm-up plus three measured runs per implementation and size;
- median runtime, interquartile range, process peak RSS, and Python allocation peak; and
- consistency of the active-state and silent-state recurrence contributions.

The benchmark uses Python allocation tracing during every timed repetition. Its results
are therefore instrumented, machine-dependent measurements rather than uninstrumented
performance figures or empirical estimates of asymptotic complexity. The measured
pixel-wise/vectorized speedups combine reduced Python interpreter overhead with the
vectorized implementation's reuse of fixed LGN evidence.

For the validated run, the corrected baseline model had an orientation-selectivity index
of $0.2488$ with preferred orientation $0^\circ$. Benchmark timings are
machine-dependent and should be interpreted using the values produced on the target
machine.

## Model and network configuration

The reported lattice is $128\times128$, corresponding to $16{,}384$ SCI sites or
spiking readout neurons. Each site uses a deterministic Moore neighborhood: interior sites have eight
neighbors, edge sites five, and corner sites three. Permitted local connections are
always present; non-neighbor and self-connections are absent. The model does not sample
a random graph and does not use conduction delays.

Rate-model, LIF, HH, noise, heterogeneity, benchmark, and random-seed parameters are
declared in the notebook. The final cells print firing-rate checks, the corrected OSI
summary, the figure manifest, and the reproducibility assertions.

## Computational complexity

For an $N\times N$ lattice, one local-evidence construction and $K$ fixed-degree MAP
sweeps require $O(N^2(1+K))$ arithmetic work and $O(N^2)$ working-array storage. For
$K\geq1$, the work is conventionally written as $O(N^2K)$. A spiking readout adds
$O(N^2S)$ work for $S$ integration steps. With the reported parameters, each LIF run
uses $400$ integration steps and each HH run uses $3{,}000$ steps. Stored raster events
require additional memory proportional to the number of recorded spikes.

The recorded finite-range timing growth is summarized by

$$
p_{64\to128}=\frac{\log(t_{128}/t_{64})}{\log 2},
\qquad
q_{64\to128}=\frac{t_{128}}{4t_{64}}.
$$

Both recorded V1ModelBio implementations grew by more than fourfold between
$N=64$ and $N=128$. This finite-range observation does not change the fixed-degree
operation bound. No additional benchmark runs were performed for the manuscript
revision.

## Correspondence with Shirazi (2004)

The log-normalized local objective in the manuscript's Eq. (6) is a notationally
separated restatement of the components associated with Shirazi's Eq. (7): observed LGN
clique energy, the log-sum-exp normalizer over candidate LGN site states, and the
SCI-neighbor regularizer. The fully LGN-driven objective in the manuscript's Eq. (7)
has the structure of Shirazi's final Eq. (21), rather than being directly identical to
the original Eq. (7).

This relation uses Shirazi's maximum approximation in Lemma 1 and Eqs. (15)–(17), the
local observed-energy substitutions in Lemma 2 and Eqs. (19)–(20), and the variable SCI
coupling in Eqs. (21)–(23). It is conditional on the stated transformations and
candidate-state assumptions. Because the numerical LGN fields here contain continuous
values normalized to $[0,1]$, general equivalence is not claimed.

## Generated outputs

Figure 1 in the manuscript is an author-created conceptual schematic and is therefore
not generated by the notebook. A clean run saves the remaining manuscript figures under
their final numbering:

| Manuscript figure | Output file |
|---:|---|
| 2 | `figure_02_runtime_memory_benchmark.png` |
| 3 | `figure_03_qualitative_orientation_response.png` |
| 4 | `figure_04_orientation_tuning_curve.png` |
| 5 | `figure_05_normalized_input_contrast.png` |
| 6 | `figure_06_noise_robustness_0deg.png` |
| 7 | `figure_07_noise_robustness_10deg.png` |
| 8 | `figure_08_noise_robustness_45deg.png` |
| 9 | `figure_09_noise_robustness_90deg.png` |
| 10 | `figure_10_rate_model_outputs.png` |
| 11 | `figure_11_model_tuning_curves.png` |
| 12 | `figure_12_hh_spiking_readout.png` |
| 13 | `figure_13_lif_spiking_readout.png` |
| 14 | `figure_14_multitrial_rate_distributions.png` |
| 15 | `figure_15_fi_curves.png` |

Selected manuscript panels use layouts optimized for legibility. These presentation
changes do not alter the underlying simulations or reported numerical results.



## Citation

If you use this software, please cite the repository and the accompanying manuscript.
Machine-readable citation metadata are provided in [`CITATION.cff`](CITATION.cff).

## Reference

Shirazi, M. N. (2004). Emergence of orientation-selective inhibition in the primary
visual cortex: a Bayes-Markov computational model. *Biological Cybernetics*, 91,
115-130. <https://doi.org/10.1007/s00422-004-0483-5>

## License

The software is released under the MIT License. See [`LICENSE`](LICENSE).
