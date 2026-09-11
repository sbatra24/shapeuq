# shapeuq

Code for **ShapeUQ: Propagating 3D Reconstruction Uncertainty Through Scientific PDE Simulations via Shape Calculus**, an oral at the CVPR 2026 3D4S workshop.

One Colab notebook, `ShapeUQ_reproduction.ipynb`, rebuilds the whole method and the SciUQ-3D benchmark. The original experiment code was lost, so this is a clean reimplementation written against the paper's Sections 3 to 7.

[Open in Colab](https://colab.research.google.com/github/sbatra24/shapeuq/blob/main/ShapeUQ_reproduction.ipynb)

## The idea

A neural SDF fitted to noisy 3D measurements is never exactly right, and when you run a physics simulation on the reconstructed geometry that error propagates into the answer without anyone quantifying it. ShapeUQ uses the Hadamard shape derivative to turn a Gaussian-process model of the SDF error into a confidence interval on the PDE quantity of interest. The sensitivity of the QoI to a normal displacement of the boundary is minus the product of the normal derivatives of the forward and adjoint solutions, divided by the SDF gradient norm (Theorem 3), so the whole uncertainty field costs one adjoint solve instead of hundreds of Monte Carlo re-solves.

The paper reports 90% coverage across glacier, protein and coral geometries at 1, 5 and 10% noise, at 14 to 31 times lower cost than MC-500.

## What's implemented

Exact SDFs for the three domains (a slab under a rough heightfield, the van der Waals surface of PDB 1UBQ from real atom coordinates, a branching capsule coral); noisy surface measurements with an 80/20 split; an 8-layer positional-encoded SDF with Eikonal regularisation and geometric initialisation; a Matérn-3/2 GP on the held-out residuals fitted by maximum marginal likelihood (plus RBF, Matérn-1/2, Matérn-5/2 and a spectral mixture for the ablation); meshfree collocation solvers for heat diffusion, linearised Poisson-Boltzmann and Stokes, forward and adjoint, with domain points where phi < 0 and boundary points projected along the SDF gradient; the Eq. 7 sensitivity field, the Eq. 8 variance and the confidence interval; the Deterministic, first-order Perturbation and Monte Carlo baselines; Table 1 (coverage and speedup), Table 2 (kernel ablation) and Figure 2 (bound tightness). Results checkpoint to `results/raw_cases.csv` after every case, so a Colab runtime reset resumes where it stopped.

## Running it

Set the runtime to a T4 GPU and run all. `RUN_MODE = "smoke"` executes every cell at toy scale in a few minutes. `RUN_MODE = "full"` runs 6 noise realisations per cell of Table 1, with Monte Carlo on 2 of them, and takes several hours; the settings are in `CFG` at the top and each one is easy to scale.

## What differs from the paper

The reference solutions in the paper came from FEniCS on a fine mesh; here the reference is the same collocation solver on the exact SDF with more iterations, so coverage measures agreement between two neural solves. The glacier surface is synthetic because the ICESat-2 tracks need an Earthdata login, and the glacier uses a Dirichlet surface temperature rather than the Robin condition so Theorem 3 applies unchanged. The Monte Carlo baseline uses warm-started re-solves with fewer samples than 500, so the measured speedup is much smaller than the paper's; the table also reports the projected speedup against MC-500 full solves, which is the comparison the paper made. All of this is stated again inside the notebook next to the numbers.

## Citation

ShapeUQ: Propagating 3D Reconstruction Uncertainty Through Scientific PDE Simulations via Shape Calculus. CVPR 2026 Workshop on 3D for Science (3D4S), oral.

## License

MIT. Copyright 2026 Soham Batra.
