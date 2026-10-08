# marmalade-stt-engines

Speech recognition model files for the Marmalade STT Android app,
published as GitHub release assets. The app downloads each file when the
user installs that engine and checks its SHA-256 before keeping it.

Release `v1` holds our own conversion of Parakeet Ultra. Releases `v2` to
`v8` re-host third-party int8 ONNX exports unchanged: each file is
byte-identical to the source named for it, with the same SHA-256.

This repository has no single license. Each asset keeps its own license,
listed below and in [NOTICE](NOTICE). License texts are in
[LICENSES/](LICENSES).

## How the app pins files

Each file is pinned by release tag and SHA-256. The download URL is:

```
https://github.com/maxwhipw/marmalade-stt-engines/releases/download/<tag>/<asset>
```

Assets are named `<engine id>-<file name>`. The app stores each one under
its file name, the part after the engine id.

Published assets are never replaced or deleted. New or changed files go
into a new release with a new tag.

## Assets

### v1: Parakeet Ultra, int8 ONNX

| Asset | Bytes | SHA-256 | License | Source |
|---|---|---|---|---|
| `parakeet-ultra-encoder.int8.onnx` | 652,282,526 | `7ef3ca4ffb49d60d4faa757b2930c8cec9927e73c24f6155d5b23a00229a7ed6` | CC-BY-4.0 | Parakeet Ultra |
| `parakeet-ultra-decoder.int8.onnx` | 11,845,274 | `1fab98fe6c12aded87d2da66272cc9e148d0a0044ce3850a12fe56302ec4a922` | CC-BY-4.0 | Parakeet Ultra |
| `parakeet-ultra-joiner.int8.onnx` | 6,355,277 | `8ba94c6919c17a6bd27368fb89533628a72ffc81d4acedebf3cdcb2cae331dbb` | CC-BY-4.0 | Parakeet Ultra |
| `parakeet-ultra-tokens.txt` | 93,939 | `d58544679ea4bc6ac563d1f545eb7d474bd6cfa467f0a6e2c1dc1c7d37e3c35d` | CC-BY-4.0 | parakeet-tdt-0.6b-v3 vocabulary |

[Parakeet Ultra](https://huggingface.co/moondream/parakeet-ultra) is
Moondream's post-trained version of NVIDIA's
[parakeet-tdt-0.6b-v3](https://huggingface.co/nvidia/parakeet-tdt-0.6b-v3).
It uses the same architecture and tokenizer. Moondream publishes it as a
Hugging Face transformers checkpoint. These files are that checkpoint
converted to the ONNX layout used by sherpa-onnx's offline NeMo transducer
(TDT) recognizer. After download the app stores them as
`encoder.int8.onnx`, `decoder.int8.onnx`, `joiner.int8.onnx` and
`tokens.txt`.

How the files were made:

1. Inputs, checked by SHA-256:
   - `moondream/parakeet-ultra` `model.safetensors`, revision
     `73175eb7aeb0d82f1e2a6b53b3aabc10a90bcd0b`, SHA-256
     `c9608f36d0ab956c14bfcc525479b0746b3b42a56f6949ec85c14eb7466717dc`.
   - `nvidia/parakeet-tdt-0.6b-v3` `parakeet-tdt-0.6b-v3.nemo`, revision
     `541d1f99c6b0c3cd0b11a95167540bb8edefd82b`, SHA-256
     `3cbdc85877e668ca7b82d0d56770eb1fac76691f55d6b97545e8d61ca588d10d`.
2. NVIDIA's v3 checkpoint was loaded in NeMo 2.7.3 (PyTorch 2.11.0). It
   supplies the model graph, the preprocessor settings, including the mel
   filterbank and window, and the tokenizer.
3. All 651 learned weights were replaced with Ultra's, using a one-to-one
   name mapping (the inverse of the transformers library's NeMo-to-HF key
   mapping). The six `vad_head` tensors in Ultra's checkpoint, a
   voice-activity head that is not part of the transducer, were left out.
4. The encoder, decoder and joiner were exported to ONNX with NeMo's
   export, with the metadata sherpa-onnx reads added. They were quantized
   with ONNX Runtime's `quantize_dynamic` (onnxruntime 1.23.2, onnx
   1.22.0): the encoder to QUInt8, the decoder and joiner to QInt8. This
   follows sherpa-onnx's NeMo export script for parakeet-tdt-0.6b-v3.
5. `tokens.txt` is written from the v3 tokenizer. It is byte-identical
   to the `tokens.txt` of k2-fsa's parakeet-tdt-0.6b-v3 int8 export.

The conversion involved no retraining or fine-tuning. Two runs of it gave
byte-identical files.

### v2: Parakeet TDT_CTC 110M (English), int8 ONNX

| Asset | Bytes | SHA-256 | License |
|---|---|---|---|
| `parakeet-tdt-110m-en-encoder.int8.onnx` | 131,113,202 | `0f35509ddeb9b39002fb077d979a9fe74f06eb0bc4dd5c34f512f82e5111d657` | CC-BY-4.0 |
| `parakeet-tdt-110m-en-decoder.int8.onnx` | 3,955,863 | `f7c331c5504c2e593c76ed22b728e3f554af6c4a383dde862e719ced08b1da19` | CC-BY-4.0 |
| `parakeet-tdt-110m-en-joiner.int8.onnx` | 1,411,403 | `bf7dff69e9f2cdbe9943d70da358f38b361c115ba0105bae7e908e0d6ec782f6` | CC-BY-4.0 |
| `parakeet-tdt-110m-en-tokens.txt` | 9,953 | `450e56bd2f036fe5b6aa821865838cc5aa9d8b0106134ce9a9ba0664abe6cd10` | CC-BY-4.0 |

NVIDIA's [parakeet-tdt_ctc-110m](https://huggingface.co/nvidia/parakeet-tdt_ctc-110m),
transducer branch, as exported to int8 ONNX by the k2-fsa / sherpa-onnx
project. Source: the files of
`sherpa-onnx-nemo-parakeet_tdt_transducer_110m-en-36000-int8.tar.bz2` in
sherpa-onnx's [`asr-models` release](https://github.com/k2-fsa/sherpa-onnx/releases/tag/asr-models)
(archive SHA-256
`f628312e9fdf8686374cb01a69425c41732529d540860311f16f37cbc32cfe9b`),
unchanged. The bytes were taken from the copy at
[punitd/sherpa-onnx-nemo-parakeet_tdt_transducer_110m-en-36000-int8](https://huggingface.co/punitd/sherpa-onnx-nemo-parakeet_tdt_transducer_110m-en-36000-int8),
commit `66a4fa70643dc7ce25c9b38b2f87e1b35ddad33d`, whose files match the
archive's.

### v3: Parakeet TDT_CTC 0.6B ja (Japanese), int8 ONNX

| Asset | Bytes | SHA-256 | License |
|---|---|---|---|
| `parakeet-ja-model.int8.onnx` | 655,542,604 | `3addd00ef5bd1742078389e540b77394e4a508bdf2f4c9ad1b4a76d93e76598e` | CC-BY-4.0 |
| `parakeet-ja-tokens.txt` | 28,557 | `732f64c53909f2620c713f4106b487d92e6f54a6915b3cd3d1dbd32f9f4f392a` | CC-BY-4.0 |

NVIDIA's [parakeet-tdt_ctc-0.6b-ja](https://huggingface.co/nvidia/parakeet-tdt_ctc-0.6b-ja),
CTC branch, as exported to int8 ONNX by the k2-fsa / sherpa-onnx project.
Source:
[csukuangfj/sherpa-onnx-nemo-parakeet-tdt_ctc-0.6b-ja-35000-int8](https://huggingface.co/csukuangfj/sherpa-onnx-nemo-parakeet-tdt_ctc-0.6b-ja-35000-int8),
commit `bef18eb066808c90bd0f5df5be685767b0732de8`, unchanged.

### v4: Canary 180M Flash, int8 ONNX

| Asset | Bytes | SHA-256 | License |
|---|---|---|---|
| `canary-180m-flash-encoder.int8.onnx` | 132,678,643 | `7a75b4e2a5857a6dcc0819503bbe3fad66943db4a3ccf21d3f27c633667d303f` | CC-BY-4.0 |
| `canary-180m-flash-decoder.int8.onnx` | 74,437,848 | `e41a2ab9c0c2fe81a1e8ade5a45fb02a74bc4db7d1f91b89a54a25e2cf79cba2` | CC-BY-4.0 |
| `canary-180m-flash-tokens.txt` | 53,555 | `2dae6fc7815f9640645e0c765522b278ee0cef49b482d91f6913e334628d3e77` | CC-BY-4.0 |

NVIDIA's [canary-180m-flash](https://huggingface.co/nvidia/canary-180m-flash),
as exported to int8 ONNX by the k2-fsa / sherpa-onnx project. Source:
[csukuangfj/sherpa-onnx-nemo-canary-180m-flash-en-es-de-fr-int8](https://huggingface.co/csukuangfj/sherpa-onnx-nemo-canary-180m-flash-en-es-de-fr-int8),
commit `9077164e0d3dd1d5353743e89ceaa1d3a770838c`, unchanged.

### v5: Nemotron 3.5 ASR Streaming 0.6B (1120 ms chunk), int8 ONNX

| Asset | Bytes | SHA-256 | License |
|---|---|---|---|
| `nemotron-3-5-streaming-1120ms-encoder.int8.onnx` | 657,601,521 | `2fff2166acaa535bd969fb223c1f0783d71029f143cb298bc54c2afe85abf772` | OpenMDW-1.1 |
| `nemotron-3-5-streaming-1120ms-decoder.int8.onnx` | 14,978,075 | `19f9c98fc6d0a2c33a65a43b36fdb2e914c26c0aa9764be3aebc502a1e982fb0` | OpenMDW-1.1 |
| `nemotron-3-5-streaming-1120ms-joiner.int8.onnx` | 9,504,438 | `4101c7c679a0bc30483794b27a059e34e79232aa2068d78d51231a22c8b0d7ce` | OpenMDW-1.1 |
| `nemotron-3-5-streaming-1120ms-tokens.txt` | 131,440 | `729cc103155bafa785f9cd45746cd41cabe97eab7182fc04d594129587958f8a` | OpenMDW-1.1 |

NVIDIA's [nemotron-3.5-asr-streaming-0.6b](https://huggingface.co/nvidia/nemotron-3.5-asr-streaming-0.6b),
as exported to int8 ONNX with a 1120 ms chunk by the k2-fsa / sherpa-onnx
project. Source:
[csukuangfj2/sherpa-onnx-nemotron-3.5-asr-streaming-0.6b-1120ms-int8-2026-06-11](https://huggingface.co/csukuangfj2/sherpa-onnx-nemotron-3.5-asr-streaming-0.6b-1120ms-int8-2026-06-11),
commit `cba1c96ca5ef0e8393b50584ae153a79145dc492`, unchanged.

### v6: Whisper large-v3-turbo, int8 ONNX

| Asset | Bytes | SHA-256 | License |
|---|---|---|---|
| `whisper-turbo-turbo-encoder.int8.onnx` | 674,716,297 | `b02dcdf54f348741e93fe732b67d933c8dcb6735655f710640143081db38878b` | MIT |
| `whisper-turbo-turbo-decoder.int8.onnx` | 361,080,764 | `20accd02388482eb3a46bd615631adfdc85e1eb2c7db9ea3f02a40ffe6b81547` | MIT |
| `whisper-turbo-turbo-tokens.txt` | 816,730 | `b34b360dbb493e781e479794586d661700670d65564001f23024971d1f2fa126` | MIT |

OpenAI's [Whisper](https://github.com/openai/whisper) `large-v3-turbo`
checkpoint, as exported to int8 ONNX by the k2-fsa / sherpa-onnx project.
Source:
[csukuangfj/sherpa-onnx-whisper-turbo](https://huggingface.co/csukuangfj/sherpa-onnx-whisper-turbo),
commit `2ca6ff69fc878651b770880507669577ac41c2ff`, unchanged.

### v7: Granite Speech 5.0 470M TurboCTC, int8 ONNX

| Asset | Bytes | SHA-256 | License |
|---|---|---|---|
| `granite-speech-5-470m-model_int8.onnx` | 551,294,349 | `8173a50ea67f864801971a40622a2e5c7d62fd230912bde94b745f92c74d60e9` | Apache-2.0 |
| `granite-speech-5-470m-tokenizer.json` | 1,137,492 | `3ee80b02f0119a040a70eb909c20fac8271c173d7e71d195a3b35f77780061e6` | Apache-2.0 |

IBM's [granite-speech-5.0-470m-turboctc](https://huggingface.co/ibm-granite/granite-speech-5.0-470m-turboctc),
as exported to int8 ONNX by qwertz92. Source:
[qwertz92/granite-speech-5.0-470m-turboctc-onnx](https://huggingface.co/qwertz92/granite-speech-5.0-470m-turboctc-onnx),
commit `e6e3b4d6f590ac51a05f25028b53591d4b260786` (`onnx/model_int8.onnx`
and `tokenizer.json`), unchanged.

### v8: Qwen3-ASR 0.6B, int8 ONNX

| Asset | Bytes | SHA-256 | License |
|---|---|---|---|
| `qwen3-asr-0-6b-conv_frontend.onnx` | 44,148,281 | `d22dc4423e0940e49884e903d2ea2f7e5567c14fc1aed97e4e26d6b8f208ef9e` | Apache-2.0 |
| `qwen3-asr-0-6b-encoder.int8.onnx` | 182,491,662 | `60748d3e6744a57c9c91e1b17424a6c2990567e8adceb0783940c03ed98fa9d9` | Apache-2.0 |
| `qwen3-asr-0-6b-decoder.int8.onnx` | 755,914,231 | `4f6885be5959ae26af3089d38ee7972c5fafbeeb1cf8d5e76eab6d8b61ca5771` | Apache-2.0 |
| `qwen3-asr-0-6b-vocab.json` | 2,776,833 | `ca10d7e9fb3ed18575dd1e277a2579c16d108e32f27439684afa0e10b1440910` | Apache-2.0 |
| `qwen3-asr-0-6b-merges.txt` | 1,671,853 | `8831e4f1a044471340f7c0a83d7bd71306a5b867e95fd870f74d0c5308a904d5` | Apache-2.0 |
| `qwen3-asr-0-6b-tokenizer_config.json` | 12,487 | `4942d005604266809309cabc9f4e9cb89ce855d59b14681fdc0e1cc62ea26c4c` | Apache-2.0 |

Alibaba Cloud's [Qwen3-ASR 0.6B](https://huggingface.co/Qwen/Qwen3-ASR-0.6B),
as exported to int8 ONNX by [Wasser1462](https://github.com/Wasser1462/Qwen3-ASR-onnx)
and re-hosted by the k2-fsa / sherpa-onnx project. Source:
[csukuangfj2/sherpa-onnx-qwen3-asr-0.6B-int8-2026-03-25](https://huggingface.co/csukuangfj2/sherpa-onnx-qwen3-asr-0.6B-int8-2026-03-25),
commit `68818b2313fe77bd06f6a7c5068ff3ef59d02b8a` (the three tokenizer
files from its `tokenizer/` folder), unchanged.

## License

| Release | License | Text |
|---|---|---|
| v1, v2, v3, v4 | Creative Commons Attribution 4.0 International (CC BY 4.0) | [LICENSES/CC-BY-4.0.txt](LICENSES/CC-BY-4.0.txt) |
| v5 | OpenMDW License Agreement, version 1.1 | [LICENSES/OpenMDW-1.1.txt](LICENSES/OpenMDW-1.1.txt) |
| v6 | MIT License, Copyright (c) 2022 OpenAI | [LICENSES/MIT-openai-whisper.txt](LICENSES/MIT-openai-whisper.txt) |
| v7, v8 | Apache License 2.0 | [LICENSES/Apache-2.0.txt](LICENSES/Apache-2.0.txt) |

Attribution, copyright notices and the list of changes for each release
are in [NOTICE](NOTICE).

The files are provided as-is, without warranties of any kind. See each
license.
