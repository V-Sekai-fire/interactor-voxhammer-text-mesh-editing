# interactor-voxhammer-text-mesh-editing

An HTTP model server that edits a masked region of a 3D mesh from a text instruction, run as a taskweft plan.

## What it is for

It inverts a mesh into the 3D latent, edits the masked region from the instruction, pastes the original geometry back outside the mask, and decodes, returning the edit as its own layer so muting it restores the original. The steps are a taskweft domain, so the guard that keeps the unmasked region unchanged lives in the plan. It has no weights of its own and reuses the image-to-mesh backbone's. [RFD 1047](https://github.com/V-Sekai-fire/manuals-weftspun/tree/main/rfd/1047-voxhammer-text-mesh-editing) owns the model image, [RFD 1037](https://github.com/V-Sekai-fire/manuals-weftspun/tree/main/rfd/1037-composite-models-as-taskweft-domains) the composite convention and [RFD 1036](https://github.com/V-Sekai-fire/manuals-weftspun/tree/main/rfd/1036-packaging-convention) the packaging.

## Build and run

```sh
docker build --target worker .
```

The `worker` stage builds on the backbone's base image, which is built first. The `contract` stage serves a stub of the same interface without a GPU or weights.

## Licence

The taskweft domain files are MIT by their SPDX headers. The repository has no licence file and states none for the rest.
