# interactor-voxhammer-text-mesh-editing

Model image for `voxhammer_text_mesh_editing`, per
[weftspun's RFD 0036](https://github.com/weftspun/request-for-discussion/tree/main/0036-packaging-convention)

- [RFD 0037](https://github.com/weftspun/request-for-discussion/tree/main/0037-composite-models-as-taskweft-domains)
  (composite models as taskweft domains). Facts from
  [RFD 0047](https://github.com/weftspun/request-for-discussion/tree/main/0047-voxhammer-text-mesh-editing).

## This is a composite

`domain.ex`, `problem.ex`, `plan.ex` are **ported verbatim from RFD 0047** — a real taskweft
HTN domain, not invented here. Per RFD 0037: "the server calls the plan, and it runs one action
per step." `server.py`'s `PLAN` list is the same 7 steps `plan.ex` records; regenerate both
together if the domain changes (`plan.ex`'s own header: "GENERATED. Do not edit by hand.").

**The guard that matters**: `a_splice` must run before `a_decode`. Inversion (`a_invert`) is
lossy — a decode of an unedited latent doesn't return the input mesh exactly, so a naive
implementation moves vertices the caller never selected. `a_splice` pastes the original
geometry back outside the mask first. The domain states this as a hard guard
(`a_decode requires /have/preserved_outside`); this dispatcher's fixed step order reproduces it.

## Model

| Property   | Value                                                                                                                                                          |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Upstream   | [Nelipot-Lee/VoxHammer](https://github.com/Nelipot-Lee/VoxHammer) (3DV 2026 Oral), on [microsoft/TRELLIS.2](https://github.com/microsoft/TRELLIS.2)'s backbone |
| License    | MIT — independently checked, matches RFD 0047 (VoxHammer's own code; TRELLIS.2's code it depends on carries its own MIT license too)                           |
| Parameters | 0 — shares [`interactor-trellis2-image-to-textured-mesh`](https://github.com/weftspun/interactor-trellis2-image-to-textured-mesh)'s weights                    |
| bf16       | 8.0 GB, the RFD 0038 cost                                                                                                                                      |

## Interface

`POST /predict`:

| Input         | Type            | Default  | Note                                                      |
| ------------- | --------------- | -------- | --------------------------------------------------------- |
| `mesh`        | Path/URL/base64 | required |                                                           |
| `instruction` | str             | required |                                                           |
| `region`      | Path/URL/base64 | required | A mask — see `decisions/api/api.md`'s supported mask list |
| `seed`        | int             | -1       |                                                           |

Returns `{layer, plan, seed, stub}` — `layer` is the edit sublayer (RFD 0053: muting it returns
the original mesh unchanged), `plan` is the step list actually executed, included so a caller
can confirm the guard order held.

## Build

Builds `FROM` `weftspun/trellis2-base`'s worker stage — build that image first.

```sh
docker build --target contract -t interactor-voxhammer-text-mesh-editing:contract .
docker run --rm -p 8000:8000 interactor-voxhammer-text-mesh-editing:contract
curl -X POST localhost:8000/predict -d @test_input.json -H 'Content-Type: application/json'
```

## Status

**Scaffolded from the RFD, not yet built or run.** The domain/plan/problem `.ex` files are real
and verbatim. `server.py`'s dispatcher runs the plan's step order correctly in stub mode
(proving the sequencing), but each individual step's call into VoxHammer's real Python API is
**not yet wired up** — `_run_plan` raises `NotImplementedError` outside stub mode. Confirm
VoxHammer's actual inversion/edit/splice/decode functions against the upstream repo before
trusting the worker stage.
