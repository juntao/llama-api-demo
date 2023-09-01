# Run an OpenAI-like API server for your llama2 models

## Download a model

Here is a llama2 7B model with 32K context length:

```bash
curl -s -L --remote-name-all https://huggingface.co/rozek/LLaMA-2-7B-32K_GGUF/resolve/main/LLaMA-2-7B-32K-Q4_0.gguf
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
nohup python3 -m llama_cpp.server --model LLaMA-2-7B-32K-Q4_0.gguf --n_ctx 2048 --host 0.0.0.0 --port 8000 &
```

## Test the API

Try the CLI command to test the API server.

```
curl -X GET http://localhost:8000/v1/models \
  -H 'accept: application/json'

{"object":"list","data":[{"id":"LLaMA-2-7B-32K-Q4_0.gguf","object":"model","owned_by":"me","permissions":[]}]}
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
  "model": "LLaMA-2-7B-32K-Q4_0.gguf",
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

## Notes

If you have installed `llama-cpp-python` before, do this to reinstall

```bash
PREFIX_ENV_VARS pip install llama-cpp-python --force-reinstall --upgrade --no-cache-dir
```

### GPU

[Install Nvidia and CUDA driver](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html#ubuntu-lts) for [AWS `g5` series](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-nvidia-driver.html#gpu-instance-install-cuda) (Tesla GPUs).

```bash
sudo apt-get install linux-headers-$(uname -r)
distribution=$(. /etc/os-release;echo $ID$VERSION_ID | sed -e 's/\.//g')
wget https://developer.download.nvidia.com/compute/cuda/repos/$distribution/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-drivers
```

[Install CUDA tools](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=deb_network).

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda
```

Then, add CUDA to the system path. Add the following line to `~/.profile` or `~/.bashrc`.

```bash
PATH="/usr/local/cuda/bin:$PATH"
```

Check GPU stat.

```bash
nvidia-smi -l 1
```

Make sure that the tools are installed.

```bash
nvcc --version
```

You may need to re-compile and restart the server after loading the GPU driver. Nvidia GPU support is available through CUDA cuBLAS.

```bash
CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install llama-cpp-python
```

Then, when you start the server, you also need to add the `--n_gpu_layers 100` flag to specify that you want to move up to 100 layers of the model to the GPU.

```bash
nohup python3 -m llama_cpp.server --model LLaMA-2-7B-32K-Q4_0.gguf --n_gpu_layers 100 --n_ctx 2048 --host 0.0.0.0 --port 8000 &
```

> If you are running `llama.cpp` from the command line by calling `main`, you should add `-ngl 100` to specify the layers to run on the GPU.

### Mac

To run on Apple Silicon, you need to compile `llama.cpp` using the Metal settings.

```bash
CMAKE_ARGS="-DLLAMA_METAL=on" FORCE_CMAKE=1 pip install llama-cpp-python
```

### huggingface

```bash
sudo apt install git-lfs
git lfs install
```

Then download any model by cloning its GIT repo. 

> Your username and password are required for private repos that you have access to. The password here is your password to log into huggingface web site. It is NOT the huggingface access token.

```bash
git clone https://huggingface.co/meta-llama/Llama-2-7b-chat
Cloning into 'Llama-2-7b-chat'...
Username for 'https://huggingface.co': my-hf-username
Password for 'https://my-hf-username@huggingface.co': my-hf-password
```

### Additional models

A llama2 13B chat model:

```bash
https://huggingface.co/akarshanbiswas/llama-2-chat-13b-gguf/resolve/main/ggml-llama-2-13b-chat-q4_k_m.gguf
```



