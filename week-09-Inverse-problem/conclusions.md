Experiment 1: Single learnable κ, equal loss weights
* κ drifted to 1.01e-7 (2.5× Spohn's value)
* Surface fit was terrible (RMSE 19 K)
* Diagnosis: optimizer exploited the residual form ∂T/∂t / κ_eff, inflating κ to trivially shrink the physics loss without solving the equation correctly
Experiment 2: Single learnable κ, strong BC weighting (λ_BC=100, λ_data=50)
* κ converged to 4.19e-8 m²/s (~7% above Spohn's 3.93e-8)
* Surface fit excellent (RMSE 0.58 K)
* Mole fit still poor (RMSE 2.63 K) with clear phase lag
* Conclusion: independently validated Spohn's measurement, but constant κ insufficient to capture the mole signal
Experiment 3: Three piecewise κ values, same weights, mole at 9.4 cm
* κ values barely differentiated (4.50, 4.00, 3.72 × 10⁻⁸)
* Phase lag essentially unchanged
* Conclusion: depth-dependent κ alone doesn't fix the phase mismatch
Experiment 4: Three piecewise κ, mole depth changed to 1.7 cm
* κ values crashed to ~7-8 × 10⁻⁹ (5× too small)
* Phase corrected but amplitude wildly wrong (RMSE 6.68 K)
* Diagnosis: at 1.7 cm the physics demands ~60% of surface amplitude, but the real TEM-A averages over the full mole and shows much smaller amplitude. This creates an irreconcilable contradiction.
Key open question: Is the mole depth issue truly the bottleneck, or are we overlooking something else? The jump from 9.4 cm to 1.7 cm was based on the skin depth argument — that the diurnal signal is dominated by the shallowest part. But the TEM-A is a physical average over the mole, not a reading from just the shallow end.
What we're confident about:
* The single-κ recovery agreeing with Spohn is solid
* The 1D heat equation with constant κ can't fully capture the mole signal
* The mole data representation is currently our biggest modelling weakness
What needs more thought:
* Is 9.4 cm actually defensible for some reason we dismissed too quickly?
* Should we model the spatial averaging explicitly rather than guessing a single depth?

Next step model spatial average to see if it works.
