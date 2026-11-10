# Face grouping (GPU)

Albums whose host switched on "Group photos by face" get a People filter. The API finds faces with two
OpenCV-zoo models, both free for commercial use, on the server's GPU (GeForce GTX 1050 Ti, 4 GB, Pascal):

| File | Model | Licence |
|---|---|---|
| `face_detection_yunet_2023mar.onnx` | YuNet detector (230 KB) | MIT |
| `face_recognition_sface_2021dec.onnx` | SFace recogniser (37 MB) | Apache-2.0 |

## One-time server setup

1. **GPU for Docker** (done 2026-11-11): `pacman -S nvidia-container-toolkit`, then
   `nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml` (re-run after an NVIDIA driver update).
   No daemon restart is needed; check with `docker run --rm --device nvidia.com/gpu=all ubuntu:24.04 nvidia-smi -L`.
2. **Models** in `/opt/apps/models` (mounted read-only at `/app/models`): run the backend's
   `tools/faces/fetch.sh /opt/apps/models` (downloads both and checks their SHA-256).

## Why the versions are pinned

The API image carries ONNX Runtime 1.26 (the newest whose GPU build uses CUDA 12) and cuDNN 9.8 with CUDA 12.9
libraries (see the backend Dockerfile, stage `gpulibs`). CUDA 13 (ONNX Runtime 1.27+) dropped Pascal, and newer
cuDNN releases have no kernels for it ("no kernel image is available for execution on the device"). Measured on
the card: detection 8 ms, fingerprint 3 ms per face.

## Memory

The models load when there is work and are released 60 s after the last photo (`FACES_IDLE_SECONDS`). Check
with `nvidia-smi` that GPU memory falls back after a minute. The GPU is used only by this (Dhirumi may use it later).

## Switches (.env)

- `FACES_DEVICE`: `auto` (default: GPU, else CPU), `gpu`, `cpu`.
- `FACES_INDEX`: `off` stops the background worker (albums keep what they have).
- Building without the GPU libraries (~2 GB smaller image): `--build-arg GPU_LIBS=off`.
