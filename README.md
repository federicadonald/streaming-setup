# Streaming Setup

A minimal Kafka-powered streaming playground that pairs the Atlas Streaming API with a matching key generator and protobuf definitions.

## Architecture
- `docker-compose.yml` orchestrates four containers:
  - **Kafka** (`apache/kafka:latest`) exposing the broker on port 9092 and all internal listeners needed by the other services.
  - **Kafka UI** for browsing topics on [http://localhost:8080](http://localhost:8080) with the wrapped cluster at `kafka:9094`.
  - **Stream API** (`atlasplatformdocker/streaming-proto-server-host:latest`) listening for gRPC requests on port 13579 and writing logs under `logs/`.
  - **Key Generator** (`atlasplatformdocker/keygenerator-proto-server:latest`) that resolves streaming keys exposed on 15379.
- `configs/AppConfig.json` holds the runtime configuration (broker address, stream-to-partition mappings, Serilog sinks, gRPC limits, etc.) that the Stream API container reads via a mounted volume.
- `logs/` collects whatever the Stream API emits for offline inspection.
- `ma/streaming` contains the Python-side runtime if you want to expand the stack locally.
- `MA.DataPlatforms.Protocol` keeps the protobuf definition trees plus the README describing Buf-based linting.
- `atlas_kafka_streaming.ipynb` is an exploratory notebook that connects to the stack once it is running.

## Getting started
1. Install Docker Desktop (Docker Engine 20+). 
2. Clone this repository 
3. Install the Jupyter Notebooks VSCode extension to run the notebook [here](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter).
4. (Optional) Customize `configs/AppConfig.json` before launching to adjust partitions, logging, or gRPC limits.
5. Start the stack by interacting with `atlas_kafka_streaming.ipynb`. 
6. Confirm Kafka is healthy via `docker compose ps` and Kafka UI on port 8080.
7. Use any gRPC client against localhost:13579 to interact with the Stream API; the key generator runs on localhost:15379.
8. Tail the Stream API logs from the host by monitoring `logs/` or attaching to the container with `docker compose logs stream-api`.

## Configuration highlights
- `StreamApiConfig` (inside `configs/AppConfig.json`) defines the stream creation strategy, broker URL, partition mappings, session/data-format integrations, and timeouts.
- `GrpcConfig` bumps the send/receive limits to 4 MiB where needed.
- `Serilog` controls console/file sinks and overrides for common namespaces.
- The Stream API container expects the config under `/app/Configs/AppConfig.json`; the compose file already mounts `configs/` accordingly.

## Protobuf and service contracts
- The full proto definitions for the Stream API, key generator, and open data service live under `MA.DataPlatforms.Protocol/proto/ma/streaming`.
- Follow the guidance in [MA.DataPlatforms.Protocol/README.md](MA.DataPlatforms.Protocol/README.md) to install Buf, run `buf lint`, or `buf generate` if you regenerate bindings.

## Working with the notebook
- Use the built-in notebook experience in VS Code to open `atlas_kafka_streaming.ipynb`. Cells are grouped into markdown (explanatory text) and code; run them sequentially to replay the examples or rerun only the sections you want to inspect.
- Keep an eye on the Python kernel chooser in the top-right of the notebook—switch to the workspace interpreter before executing code so it sees the same project dependencies.
- Restarting or resetting the kernel can be helpful if you change project files; rerun the relevant cells afterward to reinitialize connections to the Docker stack.
- The notebook already references the running Docker services (`docker compose ps` can confirm they are up). Cells that call the gRPC services expect the containers to be reachable on the ports described earlier (13579 for the Stream API, 15379 for the key generator).