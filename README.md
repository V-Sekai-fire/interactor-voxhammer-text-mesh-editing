# interactor-voxhammer-text-mesh-editing

An HTTP model server for editing a masked region of a 3D mesh from a text instruction, run as a taskweft plan.

## What it is for

The plan inverts a mesh into the 3D latent, edits the masked region from the instruction, pastes the original geometry back outside the mask, and decodes, returning the edit as its own layer so muting it restores the original. The steps are a taskweft domain, so the guard that keeps the unmasked region unchanged lives in the plan. It has no weights of its own and reuses the image-to-mesh backbone's. [RFD 1047](https://github.com/V-Sekai-fire/manuals-weftspun/tree/main/rfd/1047-voxhammer-text-mesh-editing) owns the model image, [RFD 1037](https://github.com/V-Sekai-fire/manuals-weftspun/tree/main/rfd/1037-composite-models-as-taskweft-domains) the composite convention and [RFD 1036](https://github.com/V-Sekai-fire/manuals-weftspun/tree/main/rfd/1036-packaging-convention) the packaging.

## Status

The worker's steps are not wired to the model; each raises `NotImplementedError`. The `contract` stage runs the plan as a stub without a GPU or weights, which shows the step order.

## Build and run

```sh
docker build --target contract -t voxhammer-contract .
docker run --rm -p 8000:8000 voxhammer-contract
```

The `worker` stage builds on the backbone's base image, which is built first.

## Licence

MIT. See [LICENSE](LICENSE).
