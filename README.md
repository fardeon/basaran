# Basaran

[![Python](https://github.com/hyperonym/basaran/actions/workflows/python.yml/badge.svg)](https://github.com/hyperonym/basaran/actions/workflows/python.yml)
[![codecov](https://codecov.io/gh/hyperonym/basaran/branch/master/graph/badge.svg?token=8HUSH6HSAN)](https://codecov.io/gh/hyperonym/basaran)
[![Status](https://img.shields.io/badge/status-beta-blue)](https://github.com/hyperonym/basaran)

Basaran is an open-source alternative to the [OpenAI text completion API](https://platform.openai.com/docs/api-reference/completions/create). It provides a compatible streaming API for your [Hugging Face Transformers](https://huggingface.co/docs/transformers/index)-based [text generation models](https://huggingface.co/models?pipeline_tag=text-generation).

The open source community will eventually witness the [Stable Diffusion](https://stability.ai/blog/stable-diffusion-public-release) moment for large language models (LLMs), and Basaran is committed to becoming the [Stable Diffusion web UI](https://github.com/AUTOMATIC1111/stable-diffusion-webui) for LLMs. Basaran allows you to replace OpenAI's service with the latest open-source model to power your application [without modifying a single line of code](https://github.com/hyperonym/basaran/blob/master/README.md#openai-client-library).

The key features of Basaran are:

* Stream generation using various decoding strategies.
* Support both decoder-only and encoder-decoder models.
* Detokenizer that handles surrogates and whitespace.
* Multi-GPU support with optional 8-bit quantization.
* Real-time partial progress using [server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events#Event_stream_format).
* Compatible with OpenAI API and client libraries.
* Comes with a fancy web-based playground!

<img src="https://github.com/hyperonym/basaran/blob/master/docs/assets/playground.gif?raw=true" width="640">

## Quick Start

### Installation

Docker images are available on [Docker Hub](https://hub.docker.com/r/hyperonym/basaran/tags) and [GitHub Packages](https://github.com/orgs/hyperonym/packages?repo_name=basaran).

For GPU acceleration, you also need to install the [NVIDIA Driver](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html) and [NVIDIA Container Runtime](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html). Basaran's image already comes with related libraries such as CUDA and cuDNN, so there is no need to install them manually.

Basaran's image can be used in three ways:

* **Run directly**: By specifying the `MODEL="user/repo"` environment variable, the corresponding model can be downloaded from Hugging Face Hub during the first startup.
* **Pre-packaging**: Create a new Dockerfile to [preload a public model](https://github.com/hyperonym/basaran/blob/master/deployments/bundles/bloomz-560m.Dockerfile) or [bundle a private model](https://github.com/hyperonym/basaran/blob/master/deployments/bundles/private.Dockerfile).
* **Bind mount**: Mount a model from the local file system into the container and point the `MODEL` environment variable to the corresponding path.

### Basic Usage

#### cURL

Basaran's HTTP request and response formats are consistent with the [OpenAI API](https://platform.openai.com/docs/api-reference).

Taking [text completion](https://platform.openai.com/docs/api-reference/completions/create) as an example:

```bash
curl http://127.0.0.1/v1/completions \
    -H 'Content-Type: application/json' \
    -d '{ "prompt": "once upon a time,", "echo": true }'
```

<details>
<summary>Example response</summary>

```json
{
    "id": "cmpl-e08c701b4ba032c09ef080e1",
    "object": "text_completion",
    "created": 1678003509,
    "model": "bigscience/bloomz-560m",
    "choices": [
        {
            "text": "once upon a time, the human being faces a complicated situation and he needs to find a new life.",
            "index": 0,
            "logprobs": null,
            "finish_reason": "length"
        }
    ],
    "usage": {
        "prompt_tokens": 5,
        "completion_tokens": 21,
        "total_tokens": 26
    }
}
```
</details>

#### OpenAI Client Library

If your application uses [client libraries](https://github.com/openai/openai-python) provided by OpenAI, you only need to modify the `OPENAI_API_BASE` environment variable to Basaran's corresponding endpoint:

```bash
OPENAI_API_BASE="http://127.0.0.1/v1" python your_app.py
```

The [examples](https://github.com/hyperonym/basaran/tree/master/examples) directory contains examples of using the Python library.

## Compatibility

Basaran's API format is consistent with OpenAI's, with compatibility differences mainly in parameter support and response fields. The following sections provide detailed information on the compatibility of each endpoint.

### Models

Each Basaran process serves only one model, so the result will only contain this model.

### Completions

Although Basaran does not support the `model` parameter, the OpenAI client library requires this parameter to be present. Therefore, you can fill in any model name you want.

| Parameter | Basaran | OpenAI | Default Value | Maximum Value |
| --- | --- | --- | --- | --- |
| `model` | ○ | ● | - | - |
| `prompt` | ● | ● | `""` | `COMPLETION_MAX_PROMPT` |
| `suffix` | ○ | ● | - | - |
| `min_tokens` | ● | ○ | `0` | `COMPLETION_MAX_TOKENS` |
| `max_tokens` | ● | ● | `16` | `COMPLETION_MAX_TOKENS` |
| `temperature` | ● | ● | `1.0` | - |
| `top_p` | ● | ● | `1.0` | - |
| `n` | ● | ● | `1` | `COMPLETION_MAX_N` |
| `stream` | ● | ● | `false` | - |
| `logprobs` | ● | ● | `0` | `COMPLETION_MAX_LOGPROBS` |
| `echo` | ● | ● | `false` | - |
| `stop` | ○ | ● | - | - |
| `presence_penalty` | ○ | ● | - | - |
| `frequency_penalty` | ○ | ● | - | - |
| `best_of` | ○ | ● | - | - |
| `logit_bias` | ○ | ● | - | - |
| `user` | ○ | ● | - | - |

## Roadmap

- [x] API
    - [x] Models
        - [x] List models
        - [x] Retrieve model
    - [x] Completions
        - [x] Create completion
    - [ ] Chat
        - [ ] Create chat completion
- [x] Model
    - [x] Architectures
        - [x] Encoder-decoder
        - [x] Decoder-only
    - [x] Decoding strategies
        - [x] Random sampling with temperature
        - [x] Nucleus-sampling (top-p)
        - [ ] Contrastive search

See the [open issues](https://github.com/hyperonym/basaran/issues) for a full list of proposed features.

## Contributing

This project is open-source. If you have any ideas or questions, please feel free to reach out by creating an issue!

Contributions are greatly appreciated, please refer to [CONTRIBUTING.md](https://github.com/hyperonym/basaran/blob/master/CONTRIBUTING.md) for more information.

## License

Basaran is available under the [MIT License](https://github.com/hyperonym/basaran/blob/master/LICENSE).

---

© 2023 [Hyperonym](https://hyperonym.org)
