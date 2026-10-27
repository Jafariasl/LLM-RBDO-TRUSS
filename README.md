LLM-RBDO-TRUSS

Code and data for the paper "Large language models as knowledge-guided proposers for reliability-based design optimization of truss structures" (Advances in Engineering Software).

This repository contains everything needed to reproduce the experiments reported in the paper, namely a closed-loop framework in which a large language model proposes candidate truss designs, while every proposal is screened by a deterministic finite element and first-order reliability (FORM) evaluator before it can be accepted. The language model acts only as a proposer of candidate cross sections. The feasibility and the reliability of every reported design are guaranteed by the physics-based evaluator rather than by the model itself.

Overview

At each step the framework writes the current structural state, the relevant member-group information, and the recent search history into a prompt, asks the model to return an executable update rule, and applies that rule to the incumbent design. A candidate is retained only when it lowers the weight while satisfying both the deterministic stress constraint and the probabilistic displacement constraint, the latter evaluated through FORM. A short deterministic coordinate-descent step refines the best feasible design at the end of the search. The same evaluator, acceptance rule, and refinement step are applied identically to every method compared in the study, so that any difference reflects the proposer rather than the surrounding machinery.

Benchmarks

The framework is evaluated on four sizing benchmarks of increasing size and complexity:

10-bar cantilever truss (continuous areas)
25-bar space truss (discrete catalogue)
120-bar space dome truss (continuous areas, series-system reliability)
137-bar Burro Creek bridge (real structure, SI units)
Baselines

Each benchmark is compared, under identical FE and reliability settings and under both a native and a matched evaluation budget, against particle swarm optimization (PSO), a genetic algorithm realized through differential evolution (GA), sequential least-squares programming (SLSQP), and a Kriging-based surrogate-assisted optimizer with the expected-improvement criterion (Kriging-EGO).

Repository structure

The scripts are organized by experimental phase and named by benchmark:

phase1_*_ablation.py — the framework and the deterministic control proposers (fixed heuristic rule, random perturbation, local refinement only) under an identical loop
phase3_*_benchmark.py — the PSO, GA, and SLSQP baselines under native and matched budgets
phase4_*_surrogate.py — the Kriging-EGO surrogate-assisted baseline
phase5_*_multillm.py — the model-sensitivity study across DeepSeek-V3, GPT-4o mini, and Claude Haiku 4.5
phase6_*_promptsensitivity.py — the surface-wording study (four prompt variants with identical technical content)
mc_validation_*.py — the Monte Carlo validation of the FORM reliability estimates
phase7_stats_*.py — the statistical analysis (Mann-Whitney U tests, effect sizes, Fisher exact tests)

Each script is self-contained, saves its results incrementally to CSV and Excel, and resumes from where it stopped if interrupted.

Requirements

The experiments were run with Python 3.12. The exact package versions are pinned in requirements.txt. To set up the environment:

pip install -r requirements.txt
Running the experiments

The language-model experiments query the providers through their HTTPS APIs, so an API key is required for each model used. The keys are read from environment variables and are never stored in the code. Set the ones you need before running:

export DEEPSEEK_API_KEY=...      # primary model (all main results)
export OPENAI_API_KEY=...        # GPT-4o mini, model-sensitivity study only
export ANTHROPIC_API_KEY=...     # Claude Haiku 4.5, model-sensitivity study only

Then run any phase script directly, for example:

python phase1_120bar_ablation.py
python phase3_120bar_benchmark.py

The deterministic baselines in phase3 and phase4 need no API key.

A note on reproducibility

The deterministic parts of the framework, the FE solver, the FORM analysis, the acceptance rule, and the refinement, are fully reproducible for a fixed seed. The language model's own token sampling is performed by the provider's inference service and is not bit-for-bit reproducible, even at a temperature of zero, because the order of floating-point reductions and the batching of concurrent requests vary between calls. For this reason every quantity in the paper is reported as a statistic over twenty independent runs rather than as a single realization, and an independent replication should be expected to reproduce the reported behaviour as a distribution rather than as identical individual weights.

Citation

If you use this code, please cite the paper:

J. Jafari-Asl, Y. Dong, and P. Spyridis, "Large language models as knowledge-guided proposers for reliability-based design optimization of truss structures," Advances in Engineering Software.

(The full bibliographic details will be updated once the article is published.)

Contact and contributions

Questions, bug reports, and suggestions are very welcome. If you run into any problem reproducing the results, or if something is unclear, please open an issue or contact Jafar Jafari-Asl at jafariaslyu@gmail.com.

We also warmly welcome collaboration. If you would like to build on this framework, extend it to other structures or reliability methods, or carry out new studies together, please get in touch. We are happy to help and look forward to seeing where this line of work can go.
