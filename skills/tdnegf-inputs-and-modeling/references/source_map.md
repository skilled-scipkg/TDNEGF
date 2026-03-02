# TDNEGF source map: Inputs and Modeling

Generated from source roots:
- `src`
- `examples`

Use this map only after exhausting topic docs in `doc_map.md`.

## Topic query tokens
- `model`
- `inputs`
- `hamiltonian`
- `selfenergy`
- `poles`
- `coupling`
- `ModelParamsTDNEGF`
- `pointer`

## Fast source navigation
- `rg -n "^(function|struct|mutable struct|@inline function)" src/*.jl`
- `rg -n "ModelParamsTDNEGF|pointer|build_H_ab|update_H_e!|build_Σᴸ_nλ|build_Σᴳ_nλ|build_χ_nλ|build_ξ_an|load_poles_square" src examples`
- `rg -n "Γ_nλα|χ′_nλα|Σᴸ′_nλα|Γ′_nλα|Δ_α" src examples`

## Suggested source entry points (function-level)
- `src/types.jl` | symbols: `ModelParamsTDNEGF`, `DynamicalVariables`, `pointer`
  Check: `length(p.u) == p.size_u`; `pointer(p.u, p)` reshapes to `p.dims_ρ_ab`, `p.dims_Ψ_anλα`, and `p.dims_Ω_*`.
- `src/hamiltonians.jl` | symbols: `build_blocks`, `build_H_ab`, `update_H_e!`
  Check: `build_H_ab` output is square `(Ns, Ns)` and Hermitian for static setup.
- `src/selfenergy.jl` | symbols: `build_Σᴸ_nλ`, `build_Σᴳ_nλ`, `build_χ_nλ`, `build_ξ_an`
  Check: all embedding arrays return `(Nc, N_λ)`; `build_ξ_an` localizes support at requested `xcol`/`y_coup`.
- `src/poles.jl` | symbols: `load_poles_square`, `pade_poles`
  Check: `load_poles_square` asserts `N_λ1 == 49`; output lengths match `N_λ1 + N_λ2`.
- `src/eom_tdnegf.jl` | symbol: `eom_tdnegf!`
  Check: RHS uses primed conjugate arrays (`χ′`, `Σᴸ′`, `Γ′`) and `Δ_α`; stale values produce inconsistent dynamics.
- `examples/01_two_terminal_square_lattice.jl` | symbol: `init_params`
  Check: confirms canonical assignment ordering before `ODEProblem` construction.
- `src/TDNEGF.jl` | module export surface
  Check: verify expected constructors/helpers are exported for top-level usage.

## Exclusions for canonical behavior checks
- Avoid `src/.ipynb_checkpoints/*`; they are snapshots, not primary implementation entry points.
