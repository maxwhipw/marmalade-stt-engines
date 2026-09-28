# marmalade-stt-engines

Speech recognition model files that are converted or trained for the
Marmalade STT Android app, published as GitHub release assets. The app
downloads each file when the user installs that engine and checks its
SHA-256 before keeping it.

This repository has no single license. Each asset keeps its own license,
listed below and in [NOTICE](NOTICE). License texts are in
[LICENSES/](LICENSES).

## How the app pins files

Each file is pinned by release tag and SHA-256. The download URL is:

```
https://github.com/maxwhipw/marmalade-stt-engines/releases/download/<tag>/<asset>
```

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

## License

The Parakeet Ultra files are licensed under the
[Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)
(CC BY 4.0). The full legal code is in
[LICENSES/CC-BY-4.0.txt](LICENSES/CC-BY-4.0.txt). Attribution and the
list of changes are in [NOTICE](NOTICE).

The files are provided as-is, without warranties of any kind. See
Section 5 of the license.
