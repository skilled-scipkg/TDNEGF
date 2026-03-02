---
name: tdnegf-inputs-and-modeling
description: This skill should be used when users ask about inputs and modeling in TDNEGF; it prioritizes documentation references and then source inspection only for unresolved details.
---

# TDNEGF: Inputs and Modeling

## High-Signal Playbook

### Route conditions
- Use this skill for model construction: dimensions, Hamiltonian setup, pole/self-energy preparation, lead coupling, and assignment into `ModelParamsTDNEGF`.
- Route to `tdnegf-simulation-workflows` for `ODEProblem` setup, solver tuning, runtime behavior, and post-solve checks.
- Route to `tdnegf-index` when requests span both topics.
- Stay docs-first: `docs/architecture.md`, `docs/observables.md`, then `examples/01_two_terminal_square_lattice.jl`.

### Triage questions
- What are `Nx`, `Ny`, `Nσ`, `N_orb`, `Nα`, `N_λ1`, and `N_λ2`?
- Is the active-region Hamiltonian static (`build_H_ab`) or time-dependent (`update_H_e!`)?
- Which contact channels are coupled (`xcol`, `y_coup`) for each lead?
- Are default poles acceptable (`load_poles_square`) or is a custom pole dataset required?
- What `β` and lead shifts `Δ_α` should be used?
- Which observables are required for acceptance checks after propagation?

### Start commands and required inputs
- Full reference simulation: `julia --project examples/01_two_terminal_square_lattice.jl`
- Interactive setup route:
1. `julia --project`
2. `using TDNEGF`
3. Paste the minimal working example below through conjugate assignment (`χ′`, `Σᴸ′`, `Γ′`).
4. Run the validation snippet below before handing off to propagation.
- Required inputs: lattice sizes (`Nx`, `Ny`), local DoF (`Nσ`, `N_orb`), lead count (`Nα`), poles (`N_λ1`, `N_λ2`), temperature (`β`), and lead shifts (`Δ_α`).

### Canonical workflow
1. Allocate `p = ModelParamsTDNEGF(...)` with final dimensions and pole counts (`docs/architecture.md`, `src/types.jl`).
2. Build and assign `H_ab`/`H0_ab` using `build_H_ab(...)` (`src/hamiltonians.jl`, `examples/01_two_terminal_square_lattice.jl`).
3. Load poles via `load_poles_square`, then build `Σᴸ`, `Σᴳ`, and `χ` (`src/poles.jl`, `src/selfenergy.jl`).
4. Assign per-lead tensors `p.Σᴸ_nλα`, `p.Σᴳ_nλα`, `p.χ_nλα`, then derive `p.Γ_nλα .= 1im .* (p.Σᴳ_nλα .- p.Σᴸ_nλα)` (`examples/01_two_terminal_square_lattice.jl`).
5. Build lead channel vectors with `build_ξ_an` and assign `p.ξ_anα[:,:,α]` for each lead (`src/selfenergy.jl`).
6. Set `p.Δ_α`, then refresh `p.χ′_nλα`, `p.Σᴸ′_nλα`, `p.Γ′_nλα` before solving (`src/types.jl`, `src/eom_tdnegf.jl`).
7. Hand off to `tdnegf-simulation-workflows` for propagation and runtime checks.

### Minimal working example
```julia
using TDNEGF

Rλ, zλ = load_poles_square(49, 20)
p = ModelParamsTDNEGF(Nx=50, Ny=2, Nσ=2, N_orb=1, Nα=2, N_λ1=49, N_λ2=20)

p.H_ab .= build_H_ab(Nx=p.Nx, Ny=p.Ny, Nσ=p.Nσ, N_orb=p.N_orb, γ=1.0, γso=0.5 + 0im)
p.H0_ab .= p.H_ab

ΣL = build_Σᴸ_nλ(Rλ, zλ, p.Ny, p.Nσ, p.N_orb, p.N_λ1, p.N_λ2; β=33.0, γ=1.0)
ΣG = build_Σᴳ_nλ(Rλ, zλ, p.Ny, p.Nσ, p.N_orb, p.N_λ1, p.N_λ2; β=33.0, γ=1.0)
χ  = build_χ_nλ(zλ, p.Ny, p.Nσ, p.N_orb, p.N_λ1, p.N_λ2; β=33.0, γ=1.0)

p.Σᴸ_nλα[:,:,1] .= ΣL; p.Σᴸ_nλα[:,:,2] .= ΣL
p.Σᴳ_nλα[:,:,1] .= ΣG; p.Σᴳ_nλα[:,:,2] .= ΣG
p.χ_nλα[:,:,1]  .= χ;  p.χ_nλα[:,:,2]  .= χ
p.Γ_nλα .= 1im .* (p.Σᴳ_nλα .- p.Σᴸ_nλα)

p.ξ_anα[:,:,1] .= build_ξ_an(p.Nx, p.Ny, p.Nσ, p.N_orb; xcol=1,    y_coup=1:p.Ny)
p.ξ_anα[:,:,2] .= build_ξ_an(p.Nx, p.Ny, p.Nσ, p.N_orb; xcol=p.Nx, y_coup=1:p.Ny)
p.Δ_α .= ComplexF64[0.5, -0.5]
p.χ′_nλα .= conj.(p.χ_nλα); p.Σᴸ′_nλα .= conj.(p.Σᴸ_nλα); p.Γ′_nλα .= conj.(p.Γ_nλα)
```
Source: `examples/01_two_terminal_square_lattice.jl`.

### Validation snippet before propagation
```julia
@assert size(p.H_ab) == (p.Ns, p.Ns)
@assert size(p.ξ_anα) == (p.Ns, p.Nc, p.Nα)
@assert size(p.Σᴸ_nλα) == (p.Nc, p.N_λ, p.Nα)
@assert maximum(abs.(p.Γ_nλα .- 1im .* (p.Σᴳ_nλα .- p.Σᴸ_nλα))) < 1e-10
@assert norm(p.H_ab - p.H_ab') < 1e-10
```

### Pitfalls/fixes
- `load_poles_square` enforces `N_λ1 == 49`; other values fail fast. Fix: keep `N_λ1=49` or add a custom loader (`src/poles.jl`).
- `update_H_e!` assumes `N_loc == 2`; multi-orbital local updates are not implemented there. Fix: avoid this helper or extend it (`src/hamiltonians.jl`).
- Reassigning pole/self-energy arrays without refreshing conjugates (`χ′`, `Σᴸ′`, `Γ′`) yields stale RHS terms. Fix: recompute conjugates after every reassignment (`src/types.jl`, `src/eom_tdnegf.jl`).
- `Δ_α` must match `Nα` length. Fix: assign explicit one-entry-per-lead `ComplexF64` values (`src/types.jl`).
- Wrong `xcol`/`y_coup` for `build_ξ_an` misplaces lead coupling. Fix: verify boundary columns and coupled rows per lead (`src/selfenergy.jl`, example script).

### Convergence/validation checks
- Confirm dimension consistency before solve: `size(p.ξ_anα)`, `size(p.Σᴸ_nλα)`, and `length(p.u)` align with `ModelParamsTDNEGF` fields (`src/types.jl`).
- Check Hermiticity of static Hamiltonian (`norm(p.H_ab - p.H_ab')` near machine precision) (`src/hamiltonians.jl`).
- Confirm sign convention `Γ = i(Σᴳ-Σᴸ)` matches example and RHS usage (`examples/01_two_terminal_square_lattice.jl`, `src/eom_tdnegf.jl`).
- During a short propagation, verify finite non-NaN outputs for `n_i`, `σ_i`, and `Iα` (`docs/observables.md`).

## Scope
- Handle questions about inputs, system setup, models, and physical parameterization.
- Keep responses architectural by default; go function-by-function only when requested.

## Primary documentation references
- `docs/architecture.md`
- `docs/observables.md`

## Workflow
- Start with primary docs.
- If details are missing, inspect `references/doc_map.md` for additional docs/examples.
- If ambiguity remains, inspect `references/source_map.md` for ranked function entry points.
- Cite exact file paths in responses.

## Source entry points for unresolved issues
- `src/types.jl`: `ModelParamsTDNEGF`, `DynamicalVariables`, `pointer`.
  Behavior check: validate `length(p.u) == p.size_u` and `pointer(p.u, p)` reshapes to `p.dims_*`.
- `src/hamiltonians.jl`: `build_H_ab`, `update_H_e!`.
  Behavior check: verify Hermiticity and local block updates for selected sites.
- `src/selfenergy.jl`: `build_Σᴸ_nλ`, `build_Σᴳ_nλ`, `build_χ_nλ`, `build_ξ_an`.
  Behavior check: verify returned shapes `(Nc, N_λ)` and channel-localized support at requested contact columns.
- `src/poles.jl`: `load_poles_square`, `pade_poles`.
  Behavior check: enforce `N_λ1 == 49` and inspect returned vector lengths.
- `src/eom_tdnegf.jl`: `eom_tdnegf!` consumption of `χ′`, `Σᴸ′`, `Γ′`, and `Δ_α`.
  Behavior check: verify no stale conjugates after parameter reassignment.
- `examples/01_two_terminal_square_lattice.jl`: canonical assignment ordering before propagation.

## Optional deeper inspection
- `src/`
