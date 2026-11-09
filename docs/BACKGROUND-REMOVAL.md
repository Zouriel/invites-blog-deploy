# Background removal model

The designer's "Remove background" runs BiRefNet general "lite" (MIT licence, ZhengPeng7/BiRefNet) on the API's CPU.
The model file is not in any repo or image: it lives on the server and is mounted read-only at `/app/models`.

Put it there once (or after moving servers):

```sh
sudo mkdir -p /opt/apps/models && cd /opt/apps/models
curl -L -H 'Accept: application/octet-stream' -o birefnet-general-lite.onnx \
  "$(curl -s https://api.github.com/repos/danielgatis/rembg/releases/tags/v0.0.0 \
     | python3 -c "import json,sys;print([a['url'] for a in json.load(sys.stdin)['assets'] if a['name']=='BiRefNet-general-bb_swin_v1_tiny-epoch_232.onnx'][0])")"
md5sum birefnet-general-lite.onnx   # 4fab47adc4ff364be1713e97b7e66334 (rembg's published checksum)
```

Then `docker compose -f compose.prod.yml up -d api`. Each cut-out takes ~10 s on 4 threads and about 1–2 GB of memory;
one runs at a time.
