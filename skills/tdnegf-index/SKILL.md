---
name: tdnegf-index
description: This skill should be used when users ask how to use TDNEGF and the correct generated documentation skill must be selected before going deeper into source code.
---

# TDNEGF Skills Index

## Route the request
- Use `tdnegf-inputs-and-modeling` for model dimensions, Hamiltonian/self-energy/pole preparation, lead coupling setup, and `ModelParamsTDNEGF` initialization.
- Use `tdnegf-simulation-workflows` for `ODEProblem` setup, solver/tolerance choices, runtime diagnostics, and observables extraction.
- If the request spans both phases, start with inputs/modeling and then hand off to simulation workflows.

## Generated topic skills
- `tdnegf-inputs-and-modeling`: Inputs and Modeling (inputs, system setup, models, and physical parameterization)
- `tdnegf-simulation-workflows`: Simulation Workflows (simulation setup, execution flow, and runtime controls)

## Quick simulation start (repo root)
- Full reference run: `julia --project examples/01_two_terminal_square_lattice.jl`
- Interactive route:
1. `julia --project`
2. Build `p::ModelParamsTDNEGF` using `tdnegf-inputs-and-modeling`.
3. Run propagation and observables checks using `tdnegf-simulation-workflows`.

## Repository anchors
- Docs: `docs/`
- Example entry point: `examples/01_two_terminal_square_lattice.jl`
- Source for function-level checks: `src/`
- Prefer canonical `src/*.jl`; avoid `src/.ipynb_checkpoints/*` for behavior verification.

## Test roots for behavior checks
- None discovered.

## Escalation order
1. Open the selected topic `SKILL.md`.
2. Read the selected topic doc map:
   `skills/tdnegf-inputs-and-modeling/references/doc_map.md` or
   `skills/tdnegf-simulation-workflows/references/doc_map.md`.
3. Use the selected topic source map:
   `skills/tdnegf-inputs-and-modeling/references/source_map.md` or
   `skills/tdnegf-simulation-workflows/references/source_map.md`.
4. Run targeted search, for example:
   `rg -n "eom_tdnegf!|pointer|build_Σ|build_H_ab|obs_" src examples`.
