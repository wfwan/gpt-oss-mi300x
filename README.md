# GPT-OSS on AMD Instinct MI300X with vLLM Serving Benchmark
Deployment and benchmarking of OpenAI GPT-OSS 20B and 120B on a single AMD Instinct MI300X using vLLM with ROCm platform and AITer backend. Achieves 424 tok/s output throughput on GPT-OSS 20B and 275 tok/s on GPT-OSS 120B across benchmark runs.

## Hardware
- GPU: AMD Instinct MI300X
- Platform: [AMD Developer Cloud]([url](https://www.amd.com/en/developer/resources/cloud-access/amd-developer-cloud.html))
<img width="1085" height="399" alt="Snipaste_2026-05-26_01-15-08" src="https://github.com/user-attachments/assets/ed8e2d3c-218b-4ecc-ac3e-eaaf5bbb8550" />

## Stack
- ROCm ([rocm/vllm-dev:open-mi300-08052025]([url](https://hub.docker.com/layers/rocm/vllm-dev/open-mi300-08052025/images/sha256-4bbe1ae90ca9f006975fd4236e99fd51d060f26dc8277072026652cc19b8b090)))
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
### 5. Benchmark Result (10 concurrent requests)

| Metric  | GPT-OSS 120B | GPT-OSS 20B |
|---------|--------------|-------------|
|Mean TTFT|     128.97ms |    73.01ms (43% faster)|
|Mean TPOT|     16.24ms  | 9.63ms     (41% faster)|
|Mean ITL |    14.04ms   |   8.84ms   (37% faster)| 

## Jupyter Demo
`jupyter lab --allow-root`

<img width="979" height="201" alt="jupyter result" src="https://github.com/user-attachments/assets/ce571bd4-6dba-4219-a010-ff1c3d0057f7" />

