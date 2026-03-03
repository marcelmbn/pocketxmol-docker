# pocketxmol-docker

## Micromamba image

Build the micromamba-based image:

```bash
docker buildx build -f Dockerfile.pocketxmol.mamba -t pocketxmol-mamba .
```

Run the FastAPI service:

```bash
mkdir -p outputs_api
mkdir -p inputs

# Put your protein PDB file in ./inputs, for example:
# ./inputs/1hsg_clean.pdb

docker run --runtime=nvidia --rm --gpus all \
  -p 8012:8012 \
  -v "$PWD/inputs:/home/mambauser/PocketXMol/inputs:ro" \
  -v "$PWD/outputs_api:/home/mambauser/PocketXMol/outputs_api" \
  pocketxmol-mamba
```

The service is then reachable from the host at `http://localhost:8012`.

Health check:

```bash
curl http://localhost:8012/health
```

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

## FastAPI service from the container

The `feat/api` PocketXMol checkout inside the image includes `app.py`, and the
container starts `uvicorn` on port `8012` by default.

The API resolves file paths from inside the container, not from the host. The
`docker run` command above mounts your local `./inputs` directory at
`/home/mambauser/PocketXMol/inputs` inside the container, so a host file such as
`$PWD/inputs/1hsg_clean.pdb` must be sent to the API as
`/home/mambauser/PocketXMol/inputs/1hsg_clean.pdb`.

Simple sampling request:

```bash
curl -X POST http://localhost:8012/sample/simple \
  -H "Content-Type: application/json" \
  --data '{
    "protein_path": "/home/mambauser/PocketXMol/inputs/1hsg_clean.pdb",
    "pocket_coord": [13.09, 23.142, 6.004],
    "radius": 10.0,
    "num_mols": 50,
    "batch_size": 10,
    "device": "cuda:0",
    "data_id": "1HSG_PocketC1_try2",
    "pdbid": "1HSG",
    "outdir": "/home/mambauser/PocketXMol/outputs_api"
  }'
```

If you already have a long-running PocketXMol container, start the API inside it
instead of launching a new one:

```bash
docker exec -it <container_name> \
  uvicorn app:app --host 0.0.0.0 --port 8012
```

In that case, make sure the container was created with `-p 8012:8012`, otherwise
the API will only be reachable from inside the container.
