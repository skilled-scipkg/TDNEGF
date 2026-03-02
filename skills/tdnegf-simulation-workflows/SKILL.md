---
name: tdnegf-simulation-workflows
description: This skill should be used when users ask about simulation workflows in TDNEGF; it prioritizes documentation references and then source inspection only for unresolved details.
---

# TDNEGF: Simulation Workflows

## High-Signal Playbook

### Route conditions
- Use this skill for propagation execution: `ODEProblem` setup, solver choice, tolerances, state unpacking, runtime diagnostics, and observables extraction.
- Route to `tdnegf-inputs-and-modeling` when static inputs (`H_ab`, `Σ`, `χ`, `ξ`, `Δ`) are incomplete.
- Route to `tdnegf-index` when a request spans multiple topics.
- Start with `docs/propagation.md`, then cross-check `examples/01_two_terminal_square_lattice.jl`, then inspect `src/eom_tdnegf.jl`.

### Triage questions
- Is `ModelParamsTDNEGF` fully initialized (Hamiltonian, self-energies, poles, couplings, conjugates)?
- What time span and output granularity are required (`tspan`, save cadence, final state only)?
- Which integrator and tolerances are targeted (`Vern7`, `Tsit5`, `reltol`, `abstol`)?
- Should the run be adaptive+dense, or memory-minimized (`dense=false`, `save_everystep=false`)?
- Which observables are needed (`n_i`, `σ_i`, `Iα`, `Iαx`)?
- Is this one-time TDNEGF ODE propagation (not a two-time contour-grid solver)?

### Start commands and required inputs
- Full reference simulation: `julia --project examples/01_two_terminal_square_lattice.jl`
- Fast runtime smoke route:
1. `julia --project`
2. `using TDNEGF, DifferentialEquations`
3. Build a fully initialized `p` using the inputs skill.
4. Run the short solve snippet below and execute the post-solve validation snippet.
- Required runtime inputs: initialized `p`, `tspan`, integrator choice, and tolerances.

### Canonical workflow
1. Confirm model state is ready (or route back to inputs skill) (`docs/architecture.md`, `examples/01_two_terminal_square_lattice.jl`).
2. Construct `prob = ODEProblem(eom_tdnegf!, p.u, tspan, p)` (`docs/propagation.md`, `src/eom_tdnegf.jl`).
3. Solve with DifferentialEquations.jl, for example `solve(prob, Vern7(); adaptive=true, dense=false, reltol=1e-6, abstol=1e-8)`.
4. Post-process each saved state via `dv = pointer(ut, p)` and observable kernels (`docs/observables.md`, `src/observables.jl`).
5. Validate stability/accuracy with tolerance and integrator cross-checks before physical interpretation.
6. Escalate into `eom_tdnegf!` internals (`ρ`, `Ψ`, `Ω` blocks) only when docs/example behavior and results disagree.

### Minimal working example
```julia
using TDNEGF, DifferentialEquations

prob = ODEProblem(eom_tdnegf!, p.u, (0.0, 100.0), p)
sol = solve(
    prob, Vern7();
    adaptive=true,
    dense=false,
    reltol=1e-6,
    abstol=1e-8,
)

obs = ObservablesTDNEGF(p; N_tmax=length(sol.t), N_leads=p.Nα)
obs.t .= sol.t
for (it, ut) in enumerate(sol.u)
    obs.idx = it
    dv = pointer(ut, p)
    obs_n_i!(dv, p, obs)
    obs_σ_i!(dv, p, obs)
    obs_Ixα!(dv, p, obs)
end
```
Source: `examples/01_two_terminal_square_lattice.jl`.

### Short solve snippet (smoke run)
```julia
prob = ODEProblem(eom_tdnegf!, p.u, (0.0, 0.2), p)
sol = solve(prob, Tsit5(); adaptive=true, dense=false, save_everystep=false, reltol=1e-5, abstol=1e-7)
```

### Post-solve validation snippet
```julia
@assert !isempty(sol.t)
@assert all(u -> all(x -> isfinite(real(x)) && isfinite(imag(x)), u), sol.u)

dv = pointer(sol.u[end], p)
obs = ObservablesTDNEGF(p; N_tmax=1, N_leads=p.Nα)
obs.idx = 1
obs_n_i!(dv, p, obs)
obs_σ_i!(dv, p, obs)
obs_Ixα!(dv, p, obs)
@assert all(isfinite, obs.n_i[:,1])
@assert all(isfinite, obs.Iα[:,1])
```

### Pitfalls/fixes
- Running propagation with partially assigned model inputs gives unphysical trajectories. Fix: complete all static assignments first (`docs/architecture.md`, example).
- Treating this as a two-time solver leads to wrong state/output expectations. Fix: use one-time `u(t)` semantics (`docs/propagation.md`).
- Reusing `p.u` across experiments without reset can leak prior state. Fix: reinitialize `ModelParamsTDNEGF` or reset `p.u` before each run (`src/types.jl`).
- Loose tolerances can hide instabilities; overly strict tolerances can dominate runtime. Fix: run at least one tolerance sweep (`docs/propagation.md`).
- Long runs with dense output and full saves can exhaust memory. Fix: set `dense=false` and optionally `save_everystep=false` when endpoints are enough.
- Skipping `pointer(ut, p)` before observable calls breaks shape assumptions. Fix: unpack each `ut` first (`docs/observables.md`).

### Convergence/validation checks
- Repeat solves with tighter tolerances (`1e-6/1e-8` vs `1e-7/1e-9`) and compare key observables.
- Cross-check one trajectory with a second integrator (`Vern7` vs `Tsit5`) for qualitative agreement.
- Inspect sampled density matrices from `pointer(ut, p)` for finite values and Hermiticity trends.
- Ensure observables remain finite and NaN/Inf-free over the stored timeline (`docs/observables.md`).

## Scope
- Handle questions about simulation setup, execution flow, and runtime controls.
- Keep responses architectural by default; go function-by-function only when requested.

## Primary documentation references
- `docs/propagation.md`
- `docs/observables.md`

## Workflow
- Start with primary docs.
- If details are missing, inspect `references/doc_map.md` for additional docs/examples.
- If ambiguity remains, inspect `references/source_map.md` for ranked function entry points.
- Cite exact file paths in responses.

## Source entry points for unresolved issues
- `src/eom_tdnegf.jl`: `eom_tdnegf!` (`ρ`, `Ψ`, `Ω` update blocks).
  Behavior check: confirm `du` stays finite for a short solve and that conjugate arrays are consumed as expected.
- `src/types.jl`: `ModelParamsTDNEGF`, `pointer`.
  Behavior check: verify vector-to-view unpacking is consistent across `sol.u` states.
- `src/observables.jl`: `ObservablesTDNEGF`, `obs_n_i!`, `obs_σ_i!`, `obs_Ixα!`.
  Behavior check: finite outputs with expected shapes for one or more saved time slices.
- `examples/01_two_terminal_square_lattice.jl`: full flow `init_params -> ODEProblem -> solve -> observables`.
  Behavior check: use as canonical reference for call order and defaults.
- `src/TDNEGF.jl`: module exports and include wiring.
  Behavior check: exported runtime symbols (`eom_tdnegf!`, `pointer`, observable kernels) are available from `using TDNEGF`.

## Optional deeper inspection
- `src/`
