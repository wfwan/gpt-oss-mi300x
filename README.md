# GPT-OSS 120B on AMD MI300X with vLLM
Deploy OpenAI's GPT-OSS 120B model on AMD MI300X using vLLM with AMD ROCm and AITer

## Hardware
- GPU: AMD Instinct MI300X
- Platform: AMD Developer Cloud
<img width="1085" height="377" alt="GPU Droplet AMD Developer Cloud" src="https://github.com/user-attachments/assets/e20b2581-15d5-4032-a141-53e40a4fa999" />


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
<img width="717" height="350" alt="started vllm on AMD Developer Cloud" src="https://github.com/user-attachments/assets/9bcc4b86-2b38-4b08-88f6-67c9595d2c13" />


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
<img width="720" height="102" alt="result" src="https://github.com/user-attachments/assets/5fc2517e-c70b-4d31-ab1a-4ff1a445e0c6" />

|Avg prompt throughput|Avg generation throughput|
|---------------------|-------------------------|
|         9.0 tokens/s|      9.7 tokens/s       |

## Jupyter Demo
`jupyter lab --allow-root`

<img width="979" height="201" alt="jupyter result" src="https://github.com/user-attachments/assets/ce571bd4-6dba-4219-a010-ff1c3d0057f7" />

