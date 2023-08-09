# Run an OpenAI-like API server for your llama2 models

## Download a model

Here is a llama2 7B chat model file:

```bash
curl -s -L --remote-name-all https://huggingface.co/localmodels/Llama-2-7B-Chat-ggml/resolve/main/llama-2-7b-chat.ggmlv3.q4_0.bin
```

Here is an alpaca-lora 65B model:

```bash
curl -s -L --remote-name-all https://huggingface.co/TheBloke/alpaca-lora-65B-GGML/resolve/main/alpaca-lora-65B.ggmlv3.q5_1.bin
```

## Setup the software on Ubuntu

Make sure that you have dev tools including the C++ compiler toolchain and Python installed.

```bash
sudo ACCEPT_EULA=Y apt-get update
sudo ACCEPT_EULA=Y apt-get upgrade
sudo apt-get install git curl software-properties-common build-essential libopenblas-dev ninja-build pkg-config cmake-data
sudo apt-get install python3 python3-pip python-is-python3
```

Install Python packages.

```bash
python -m pip install --upgrade pip pytest cmake scikit-build setuptools fastapi uvicorn sse-starlette pydantic-settings
```

Build `llama.cpp` and install its Python wrapper package.

```bash
CMAKE_ARGS="-DLLAMA_BLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS" FORCE_CMAKE=1 pip install llama-cpp-python
```

## Run the API server

```bash
nohup python3 -m llama_cpp.server --model llama-2-7b-chat.ggmlv3.q4_0.bin --n_ctx 2048 --host 0.0.0.0 --port 8000 &
```

## Test the API

Try the CLI command to test the API server.

```
curl -X GET http://localhost:8000/v1/models \
  -H 'accept: application/json'

{"object":"list","data":[{"id":"vendor/llama.cpp/models/7B/ggml-model-q4_0.bin","object":"model","owned_by":"me","permissions":[]}]}
```

Send in an HTTP API request to prompt the model and ask it to answer a question.

```
curl -X POST http://localhost:8000/v1/chat/completions \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{"messages":[{"role":"system", "content": "You are a helpful assistance. Provide helpful and informative responses in a concise and complete manner. Please avoid using conversational tags and only reply in full sentences. Ensure that your answers are presented directly and without the human of '\''Human:'\'' or '\''###'\''. Thank you for your cooperation"}, {"role":"user", "content": "What was the significance of Joseph Weizenbaum'\''s ElIZA program?"}], "max_tokens":64}'
```

The response is

```
{
  "id": "chatcmpl-0cfe6cdf-28be-4bb0-a39e-9572e678c0d1",
  "object": "chat.completion",
  "created": 1690825931,
  "model": "/llama-cpp-python/vendor/llama.cpp/models/7B/ggml-model-q4_0.bin",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The eliza program is an artificial intelligence program. It simulates a Rogerian therapist and was created by Weizanbam in 1964. The user can ask it questions or give it statements to respond with replies that are designed to make them feel better. However, the computer"
      },
      "finish_reason": "length"
    }
  ],
  "usage": {
    "prompt_tokens": 92,
    "completion_tokens": 64,
    "total_tokens": 156
  }
}
```

