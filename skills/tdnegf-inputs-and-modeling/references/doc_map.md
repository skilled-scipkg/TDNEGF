# TDNEGF documentation map: Inputs and Modeling

Generated from documentation roots:
- `docs`
- `examples`

Total docs grouped in this topic: 3

## File inventory
- `docs/architecture.md` | title: TDNEGF architecture (current implementation) | headings: overall workflow; `ModelParamsTDNEGF`; flattened state + `pointer`
- `docs/observables.md` | title: Observables pipeline | headings: observable kernels (`obs_n_i!`, `obs_σ_i!`, `obs_Ixα!`); data-flow placement
- `examples/01_two_terminal_square_lattice.jl` | title: two-terminal square lattice example | headings: `init_params`; model assignment order; `ODEProblem`/`solve` usage

## Practical start command
- `julia --project examples/01_two_terminal_square_lattice.jl`

## Inputs checklist before handoff to propagation
- Dimensions are finalized: `Nx`, `Ny`, `Nσ`, `N_orb`, `Nα`, `N_λ1`, `N_λ2`.
- Pole and embedding arrays are assigned to every lead index `α`.
- `Γ_nλα`, `χ′_nλα`, `Σᴸ′_nλα`, and `Γ′_nλα` are refreshed after edits.
- Coupling vectors `ξ_anα` target the intended boundary columns/rows.
