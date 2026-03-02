# TDNEGF source map: Simulation Workflows

Generated from source roots:
- `src`
- `examples`

Use this map only after exhausting topic docs in `doc_map.md`.

## Topic query tokens
- `propagation`
- `simulation`
- `ODEProblem`
- `solver`
- `tolerance`
- `pointer`
- `observables`
- `runtime`

## Fast source navigation
- `rg -n "function eom_tdnegf!|pointer|ObservablesTDNEGF|obs_n_i!|obs_σ_i!|obs_Ixα!" src examples`
- `rg -n "ODEProblem|solve\(|Vern7|Tsit5|reltol|abstol|dense|save_everystep" docs examples`
- `rg -n "χ′_nλα|Σᴸ′_nλα|Γ′_nλα|Π_abα|Ψ_anλα|Ω_nλ" src`

## Suggested source entry points (function-level)
- `src/eom_tdnegf.jl` | symbol: `eom_tdnegf!`
  Check: inspect `ρ`, `Ψ`, and `Ω` blocks when numerical behavior diverges from docs/examples.
- `src/types.jl` | symbols: `ModelParamsTDNEGF`, `pointer`
  Check: verify `pointer(sol.u[k], p)` produces consistent tensor views for every saved state.
- `src/observables.jl` | symbols: `ObservablesTDNEGF`, `obs_n_i!`, `obs_σ_i!`, `obs_Ixα!`, `cal_Π_abα`
  Check: validate finite observable outputs and expected array dimensions after solve.
- `examples/01_two_terminal_square_lattice.jl` | symbols: `init_params`, `ODEProblem`, `solve`
  Check: canonical order of model setup, solve call, and observables extraction.
- `src/TDNEGF.jl` | module export surface
  Check: runtime-facing symbols are exported and available from `using TDNEGF`.
- `src/selfenergy.jl` and `src/hamiltonians.jl` | setup dependencies for runtime
  Check: only inspect when propagation failures trace back to malformed inputs.

## Exclusions for canonical behavior checks
- Avoid `src/.ipynb_checkpoints/*`; they are snapshots, not primary implementation entry points.
