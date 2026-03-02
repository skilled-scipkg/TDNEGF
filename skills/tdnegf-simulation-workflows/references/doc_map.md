# TDNEGF documentation map: Simulation Workflows

Generated from documentation roots:
- `docs`
- `examples`

Total docs grouped in this topic: 4

## File inventory
- `docs/propagation.md` | title: Propagation workflow (current one-time solver) | headings: RHS function; high-level stepping logic; method-hook boundary
- `docs/architecture.md` | title: TDNEGF architecture (current implementation) | headings: one-time workflow; `ModelParamsTDNEGF`; flattened state + `pointer`
- `docs/observables.md` | title: Observables pipeline | headings: `obs_n_i!`; `obs_σ_i!`; `obs_Ixα!`; data-flow placement
- `examples/01_two_terminal_square_lattice.jl` | title: two-terminal square lattice example | headings: `ODEProblem`; `solve`; pointer-based observables loop

## Practical start command
- `julia --project examples/01_two_terminal_square_lattice.jl`

## Runtime checklist
- `p` is fully initialized before `ODEProblem` construction.
- Solver options (`adaptive`, `dense`, `save_everystep`, tolerances) match runtime goals.
- Post-solve checks confirm finite state vectors and finite observables.
