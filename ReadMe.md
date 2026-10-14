# Proof and Clay Alignment Audit

Status: complete. The user confirms the audited files were downloaded from Git https://github.com/openai/NavierStokesAndEuler and
that the project compiles successfully.

Author: Obrian Mc Kenzie (aka Auro Zera online) in collaboration with AI (GPT-6 Luna running in copilot).
Small donations in Ethereum are quite welcome. Please send any such gifts to my ETH address sails.eth (active until the year 2121)

## Audit Targets

- [x] Compare alternatives (C) and (D), including the Clay erratum, in the
  supplied official statement PDF against
  `NavierStokes/ComparatorDefinitions.lean` and
  `ComparatorChallenges/NavierStokes.lean`.
- [x] Build status: user reports the pinned project compiles; this audit did
  not independently rerun Lake.
- [x] Audit both bridges from the comparator solution classes to the project's
  global-solution classes, especially PDE translation, time-zero behavior,
  and energy normalization: `NavierStokes/R3/ComparatorBridge.lean`,
  `NavierStokes/ComparatorBridge.lean`, and
  `NavierStokes/PeriodicComparatorSolution.lean`.
- [x] Compare the reported Lemma 8.6 derivative-loss discrepancy with the
  cited Lean theorem and inspect the local call site.
- [x] Compare the reported equation (10.19) pressure-flux discrepancy with
  the Lean estimate and trace how that estimate is consumed downstream.
- [x] Compare the OpenAI manuscript's Lemma 8.6 and equation (10.19) with the
  saved `navier-stokes.pdf` text supplied in the workspace attachment.
- [x] Pin the related Formal Conjectures link to the exact source revision
  cited by `NavierStokes/ComparatorDefinitions.lean`.
- [x] Keep the separate unforced Euler results out of the Navier-Stokes Clay
  (C)/(D) conclusion; Euler is outside this audit's scope.

## Initial Local Findings

- The saved OpenAI manuscript text states Theorem 1.1 for every positive
  viscosity with a smooth compactly supported force, zero initial data,
  divergence-free smooth fields, finite-time velocity blowup, and a uniform
  kinetic-energy bound; it explicitly says this establishes alternative (C).
  Corollary 10.6 periodizes the construction and explicitly identifies it as
  alternative (D). The paper also notes the pressure-periodicity erratum.
- Against the repository's copied Comparator predicates, the constructed zero
  datum meets the smoothness, divergence, rapid spatial-decay, and periodicity
  conditions. The compactly supported smooth force meets the whole-space
  space-time decay and periodic time-decay conditions. The (C) competitor class
  includes the PDE, smoothness, finite energy at each time, and uniform energy;
  the (D) competitor class includes the PDE, smoothness, and periodic velocity
  and pressure. The bridges map any such Comparator competitor into the
  project's excluded competitor classes, so their negations have the needed
  direction for (C)/(D).
- Direct comparison with the supplied official Clay PDF (conditions (4)-(11),
  alternatives (C)/(D), and the appended errata) found the local comparator
  predicates aligned on the tested scope. For (C), conditions (4)-(7) map to
  smooth divergence-free rapidly decaying data, a smooth rapidly decaying
  force, smooth solutions, and finite uniformly bounded energy. For (D),
  conditions (8), (9), and (10)-(11) map to periodic data/force, rapid force
  time decay, and smooth periodic solutions; the erratum explicitly requires
  periodic pressure, which the comparator includes. The erratum's correction
  to weak-solution identity (10) is unrelated to alternatives (C)/(D).
- The upstream Formal Conjectures source at commit
  `8bf45ed70d48b2b2a501de9c00b26bfa38c573ee` independently documents the same
  conditions, including pressure periodicity from the Clay erratum. It is
  corroboration; the supplied Clay PDF is the primary source checked here.
- `formalization.yaml` now links to that fixed Formal Conjectures commit rather
  than mutable `main`, matching the snapshot cited in
  `NavierStokes/ComparatorDefinitions.lean`.
- `formalization.yaml` maps (C) and (D) to
  `NavierStokes.Comparator.navier_stokes_breakdown_R3` and
  `NavierStokes.Comparator.navier_stokes_breakdown_periodic` in
  `NavierStokes/ComparatorSolution.lean`; it reports the standard Lean axioms
  and zero `sorry`s.
- A source scan under `NavierStokes/` found no executable `sorry`, `admit`,
  explicit `axiom`, or unsafe escape hatch; matches were comments or
  `#print axioms` commands. The challenge reference modules do contain
  intentional `sorry` placeholders, but `ComparatorSolution` imports
  `ComparatorR3Theorem` and `ComparatorTheorem`; the adapter path imports
  `NavierStokes.ComparatorDefinitions`, not `ComparatorChallenges.*`.
- The public-facing adapters state the same predicates as the independent
  challenge module and prove them via `ComparatorBridge`.
- The copied comparator definitions encode smooth divergence-free initial
  data; all-order spatial decay for (C); smooth force with all-order
  space-time decay for (C); and smooth, spatially periodic, all-order
  time-decaying force for (D). Global solution classes include smoothness and
  the relevant PDE; (C) additionally includes finite energy at each time and
  uniformly bounded energy. (D) requires periodic velocity and pressure.
- The whole-space bridge maps any comparator solution to
  `GlobalFiniteEnergySolution`, including its energy bound. The periodic
  bridge maps any comparator solution to `GlobalSmoothSolution`, preserving
  periodicity of velocity and pressure. The theorem adapters use zero initial
  data and the constructed force.
- The equation translations rewrite the comparator's Laplacian, gradient,
  divergence, and half-line time derivative into the project's physical
  operators, preserving the Navier-Stokes signs. Both viscosity-normalization
  bridges rescale space and fields explicitly and prove the equation for
  positive times. The whole-space bridge converts the comparator's strict
  uniform integral bound to the project's half-energy convention; the periodic
  bridge preserves periodicity of both velocity and pressure. The target
  exclusions only need the competing equation for positive times.
- `SpeedUnboundedAtOne` states that for every positive threshold and every
  positive left-neighborhood radius there is a time below one in that
  neighborhood and a point exceeding the threshold. In (C), the uniqueness
  argument identifies a hypothetical competitor with the candidate before
  time one; compact spatial support then bounds the competitor on the compact
  candidate support, contradicting this predicate. In (D), periodic uniqueness
  gives equality on every `[0,T]`, `T < 1`, and smooth extension through time
  one contradicts the same blowup predicate. No bounded-energy assumption is
  added to the periodic competitor.
- The saved manuscript's Lemma 8.6, equation (8.19), states
  `||N^{-1} F||_{C^m_y} <= C_m ||F||_{C^{m+4}_y}`. Its accompanying use
  applies this inverse to the zero-auxiliary-mean parts of the angular residual.
  The formal theorem `SmoothFamilyTorusInverse.inverse_finiteJets` requires
  `n + 5` input jets. At the inspected call in
  `PhysicalMeanDomain.meanClass_temporalInverse`, the premise comes from
  `hclass.bounds (m + 5)` and `MeanClass` supplies bounds at every order.
  Therefore the additional derivative is available at this use, but the Lean
  estimate is not the same theorem-strength as the manuscript's `m + 4` bound.
  This is a real formalization discrepancy, not a demonstrated gap in this
  downstream call.
- The whole-space pressure-flux route is concrete in
  `NavierStokes/R3/PressureRecovery.lean` and
  `NavierStokes/R3/PressureFlux.lean`. `PressureRecovery.Hypotheses` requires
  smoothness, divergence-free fields, the PDE, and uniform finite energy, but
  no pressure decay or pressure-at-infinity bound. `WholeSpaceUniqueness`
  derives the required difference `L²` bound from energy, the reference `L³`
  bound from compact support, and the tensor-difference `L¹` bound from energy,
  then discharges the flux estimate. This supports a valid alternate formal
  route locally; it does not prove equality with the paper's (10.19).
- Independent critique investigated: Bastounis, Circelli, and Hansen,
  arXiv:2610.08144v1, Examples 3.1 and 3.3. Their critique says it evaluates
  fidelity of selected Lean declarations to intermediate manuscript claims,
  not correctness of the entire natural-language proof. The revision/commit
  pairing cited in the conversation has not been independently replayed here.
- In Example 3.1, the critique notes the same `m + 4` versus `m + 5`
  discrepancy verified in the saved manuscript. The all-order `MeanClass`
  premise at the actual call provides the extra order, so no local regularity
  gap was found there. The statement-strength mismatch remains; no Lean source
  was changed to disguise it.
- The saved manuscript's equation (10.19) states, with its notation for the
  localized quantity `B_R`,
  `|integral pi w . grad chi_R| <= C_T [R^-1 (B_R + 1) B_R^(1/2) + R^-3/4 B_R^(3/4)]`.
  The formal `PressureFlux.exists_uniform_actual_pressure_flux_bound` instead
  proves a bound proportional to
  `((B_R^(1/2) + 1) (A_R/R + 1/R^2) + R^-7/4 B_R^(3/4))`, where `A_R` is
  `dissipationRoot` and `B_R` is `cutoffL6`. This is not a verbatim
  formalization of equation (10.19): it has a different set of quantities,
  exponents, and proof route.
- The differing Lean bound is used directly. `WholeSpaceUniqueness` derives
  the difference `L2` bound from uniform energy, the reference `L3` bound from
  compact support, and the tensor-difference `L1` bound from energy. It passes
  the resulting flux envelope into
  `WholeSpaceComparisonClosure.eq_of_pressure_flux_bound`, where the localized
  Sobolev estimate and energy inequality close the comparison. The local
  dependency chain therefore proves the bound needed by the Lean closure from
  the stated comparison hypotheses; this inspection does not verify every
  analytic lemma in that chain or prove equivalence with the manuscript's
  equation (10.19).
- The absorption step is explicit in `ComparisonRateBound`: the cutoff
  Sobolev estimate supplies `B_R <= S (A_R + M/R)`, and `ComparisonYoung`
  absorbs the resulting subquadratic powers `1/2`, `3/4`, and `3/2` of `A_R`
  into `delta * A_R^2 + D/R`. Thus the additional localized dissipation term
  in the Lean pressure estimate is handled by the downstream energy closure;
  no separate pressure or dissipation bound is assumed of the competitor.
- Sign/index check: `tensorDiff u v t i j` is `u_i u_j - v_i v_j`.
  `ConservativeDifference.conservative_difference_equation` gives the component
  equation with `-div(tensorDiff) - grad(p-q)`. Its `weak_pressure_poisson`
  theorem proves
  `integral (p-q) * Delta(psi) = -sum_{k,i} integral tensorDiff[k,i] * partial_i(partial_k(psi))`,
  which agrees with the distributional Poisson equation and the Riesz pairing
  convention used by pressure recovery. No sign or tensor-index mismatch was
  found in this chain.
- Pressure recovery's final gradient identity is proved only against compact
  spatial tests. It time-averages the pressure-gradient difference, proves the
  averaged functional annihilates compact Laplacian tests using the two Poisson
  identities, bounds that functional in a Fourier `H^3` test norm using `L2`
  velocity averages and `L1` tensor averages, and applies compact-test density
  plus Fourier uniqueness. `PressureFluxIdentity` then uses compact integration
  by parts to identify the actual cutoff flux with the canonical Riesz pairing.
  This route imposes no pressure decay, pressure normalization, or integrability
  at spatial infinity; its equation, smoothness, incompressibility, and energy
  inputs are visible in `PressureRecovery.Hypotheses`.
- These two findings concern correspondence with the OpenAI manuscript, not a
  mismatch between the local Comparator statements and the supplied Clay
  alternatives (C)/(D). Do not equate proposition alignment, Lean kernel
  checking, manuscript fidelity, and mathematical correctness.
- These observations support structural agreement with the comparator
  proposition and the supplied Clay statement/erratum, but do not certify that
  every analytic proof obligation is correct or that the Lean proof is fully
  faithful to the informal paper.
- A diagnostics check found no editor errors in the comparator theorem,
  bridge, inverse-estimate, pressure-recovery, flux, and comparison-closure
  files. The user reports successful compilation; this audit did not rerun the
  build or independently replay a certificate.
- `lake-manifest.json` resolves Comparator to
  `19e111e2141cf333c7daff0f64c5f24acc91dd2e` and mathlib to
  `85e3a25e006c35636f0e53b0e9296caca2685bc0`, both requested as
  `v4.34.0-rc2`.

## Repository Entry Points

- Metadata and claimed theorem mapping: `formalization.yaml`.
- Build pins and targets: `lean-toolchain`, `lakefile.toml`.
- Comparator statements and axioms configuration:
  `ComparatorChallenges/NavierStokes.lean`,
  `ComparatorChallenges/NavierStokes.json`.
- Result adapters and bridges: `NavierStokes/ComparatorSolution.lean`,
  `NavierStokes/ComparatorR3Theorem.lean`,
  `NavierStokes/ComparatorTheorem.lean`, and the bridge files listed above.
- Reported analytic objections: search source modules for the estimate
  statements corresponding to paper Lemma 8.6 and equation (10.19); findings
  and source references are recorded above. Do not infer resolution from
  theorem names or successful compilation alone.