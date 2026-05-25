# GPT-OSS 120B on AMD MI300X with vLLM
Deploy OpenAI's GPT-OSS 120B model on AMD MI300X using vLLM with AMD ROCm and AITer

## Hardware
- GPU: AMD Instinct MI300X
- Platform: AMD Developer Cloud

## Stack
- ROCm (rocm/vllm-dev:open-mi300-08052025)
- vLLM with AITer
- OpenAI-compatible REST API

## Quickstart
### 1. Launch Docker Container
```
sudo docker run --rm -it \
  --network=host \
  --device=/dev/kfd \
  --device=/dev/dri \
  --group-add=video \
  --ipc=host \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  --shm-size 32G \
  -v /data:/data \
  -v $HOME:/myhome \
  -w /myhome \
  rocm/vllm-dev:open-mi300-08052025
```
### 2. Set Environment Variables
```
export VLLM_ROCM_USE_AITER=1
export VLLM_USE_AITER_UNIFIED_ATTENTION=1
export VLLM_ROCM_USE_AITER_MHA=0
export GLOO_SOCKET_IFNAME=lo
export TP_SOCKET_IFNAME=lo
```
### 3. Start vLLM Server
```
source /myhome/setup/env_vars.sh

vllm serve openai/gpt-oss-120b \
  --compilation-config '{"full_cuda_graph": true}'
```
### 4. Test Inference
```
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openai/gpt-oss-120b",
    "messages": [
      { "role": "system", "content": "You are a helpful assistant." },
      { "role": "user", "content": "What is AMD Developer Cloud in one sentence" }
    ],
    "temperature": 0.7,
    "max_tokens": 100
  }'
```
### 5. Result
<img width="1437" height="282" alt="first attempt via curl" src="https://github.com/user-attachments/assets/f1707567-1fe2-4811-adbd-1d65b44a60ce" />


## Jupyter Demo
`jupyter lab --allow-root`

<img width="979" height="201" alt="jupyter result" src="https://github.com/user-attachments/assets/ce571bd4-6dba-4219-a010-ff1c3d0057f7" />

