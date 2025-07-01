# Set infernce service using OpenShift

Based on the instuctions [here](https://docs.redhat.com/en/documentation/red_hat_ai_inference_server/3.0/html-single/getting_started/index).

### 1. Log in to your cluster oc login ...

### 2. apply the infernce service runtime:
    oc apply -f vllm-serving-runtime.yaml

### 3. apply the infernce service:
    oc apply -f vllm-inference-service.yaml


### 6. Testing:
    git clone https://github.com/vllm-project/vllm.git
    cd vllm
    python3 -m venv .venv
    source .venv/bin/activate
    pip install vllm pandas datasets
    python benchmarks/benchmark_serving.py --backend vllm \
    --model RedHatAI/Llama-3.2-1B-Instruct-FP8 --num-prompts 100 \
    --dataset-name random  --random-input 1024 \
    --random-output 512 --port 8000


    And the output should look like this:
    ============ Serving Benchmark Result ============
    Successful requests:                     100       
    Benchmark duration (s):                  5.61      
    Total input tokens:                      102300    
    Total generated tokens:                  40087     
    Request throughput (req/s):              17.83     
    Output token throughput (tok/s):         7149.26   
    Total Token throughput (tok/s):          25393.81  
    ---------------Time to First Token----------------
    Mean TTFT (ms):                          653.51    
    Median TTFT (ms):                        656.28    
    P99 TTFT (ms):                           789.38    
    -----Time per Output Token (excl. 1st token)------
    Mean TPOT (ms):                          10.10     
    Median TPOT (ms):                        9.59      
    P99 TPOT (ms):                           14.11     
    ---------------Inter-token Latency----------------
    Mean ITL (ms):                           9.56      
    Median ITL (ms):                         9.46      
    P99 ITL (ms):                            17.72     
    ==================================================


