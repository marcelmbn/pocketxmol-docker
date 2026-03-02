# pocketxmol-docker

## Micromamba image

Build the micromamba-based image:

```bash
docker buildx build -f Dockerfile.micromambapocketxmol -t pocketxmol-mamba .
```

Run the default GPU smoke tests:

```bash
docker run --runtime=nvidia --rm --gpus all pocketxmol-mamba
```

This runs checks for:

- `torch.__version__`, `torch.version.cuda`, and `torch.cuda.is_available()`
- `torch_geometric`, `torch_cluster`, and `torch_scatter` imports
- CUDA device count and a small CUDA tensor allocation

Run the PocketXMol sampling example:

```bash
docker run --runtime=nvidia --rm --gpus all \
  pocketxmol-mamba \
  micromamba run -n base python scripts/sample_use.py \
    --config_task configs/sample/examples/dock_smallmol.yml \
    --outdir outputs_examples \
    --device cuda:0
```

To keep generated outputs on the host:

```bash
mkdir -p outputs_examples

docker run --runtime=nvidia --rm --gpus all \
  -v "$PWD/outputs_examples:/home/mambauser/PocketXMol/outputs_examples" \
  pocketxmol-mamba \
  micromamba run -n base python scripts/sample_use.py \
    --config_task configs/sample/examples/dock_smallmol.yml \
    --outdir outputs_examples \
    --device cuda:0
```
