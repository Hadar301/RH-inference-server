# Set infernce server on RHEL AI (GA) VM

Based on the instuctions [here](https://docs.redhat.com/en/documentation/red_hat_ai_inference_server/3.0/html-single/getting_started/index).
I went to the [catalog](https://catalog.demo.redhat.com/) and started a RHEL AI machine with L4 Nvidia GPU.

Following those steps:
1. SSH to your machine.
2. Open a terminal and `python3 -m venv .venv` -> `source .venv/bi/activate`.
3. run the command `pip install vllm pandas datasets` you don't need to wait for the installation to end.
4. Open a second terminal (of couse make sure you're logged in to the machine too).
5. Following the instructions of the demo:
    1. podman login registry.redhat.io
    2. podman pull registry.redhat.io/rhaiis/vllm-cuda-rhel9:3.0.0
    3. mkdir -p rhaiis-cache
    4. chmod a+rwX rhaiis-cache
    5. echo "export HF_TOKEN=<your_HF_token>" > private.env
    6. source private.env
    7. Start the continer:
    podman run --rm -it \
    --device nvidia.com/gpu=all \
    --security-opt=label=disable \
    --shm-size=4GB -p 8000:8000 \
    --env "HUGGING_FACE_HUB_TOKEN=$HF_TOKEN" \
    --env "HF_HUB_OFFLINE=0" \
    --env=VLLM_NO_USAGE_STATS=1 \
    -v ./rhaiis-cache:/opt/app-root/src/.cache \
    registry.redhat.io/rhaiis/vllm-cuda-rhel9:3.0.0 \
    --model RedHatAI/Llama-3.2-1B-Instruct-FP8

    You can verify that the pod is running by using the command `podman ps` from another terminal.
6. Go to your `.venv` terminal and test the pod:
    curl -X POST -H "Content-Type: application/json" -d '{
        "prompt": "What is the capital of France?",
        "max_tokens": 50
    }' http://<your_server_ip>:8000/v1/completions | jq
7. git clone https://github.com/vllm-project/vllm.git
8. python vllm/benchmarks/benchmark_serving.py --backend vllm --model RedHatAI/Llama-3.2-1B-Instruct-FP8 --num-prompts 100 --dataset-name random  --random-input 1024 --random-output 512 --port 8000
And my output was:
============ Serving Benchmark Result ============
Successful requests:                     100       
Benchmark duration (s):                  17.32     
Total input tokens:                      102300    
Total generated tokens:                  38827     
Request throughput (req/s):              5.77      
Output token throughput (tok/s):         2241.42   
Total Token throughput (tok/s):          8147.03   
---------------Time to First Token----------------
Mean TTFT (ms):                          5399.10   
Median TTFT (ms):                        6003.49   
P99 TTFT (ms):                           6013.25   
-----Time per Output Token (excl. 1st token)------
Mean TPOT (ms):                          36.13     
Median TPOT (ms):                        22.10     
P99 TPOT (ms):                           324.17    
---------------Inter-token Latency----------------
Mean ITL (ms):                           23.88     
Median ITL (ms):                         22.00     
P99 ITL (ms):                            26.00     
==================================================


