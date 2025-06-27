# NIM on Kubernetes for LLM Deployments

## 1. Download the NIM for Helm Charts
>**Make sure to export the value NGC_API_KEY before attempting the following operations**

```bash
helm fetch https://helm.ngc.nvidia.com/nim/charts/nim-llm-1.7.0.tgz --username='$oauthtoken' --password=$NGC_API_KEY
```

### 1.1 Check the Helm Values

```bash
helm show readme nim-llm-1.7.0.tgz | glow -p -

helm show values nim-llm-1.7.0.tgz
```

## 2 Create Secrets

```bash
kubectl create ns nim

kubectl create secret docker-registry ngc-secret -n nim --docker-server=nvcr.io --docker-username='$oauthtoken' --docker-password=$NGC_API_KEY

kubectl create secret generic ngc-api -n nim --from-literal=NGC_API_KEY=$NGC_API_KEY
```

### 2.1 Verify Secrets

```bash
$kubectl get secrets -n nim
NAME         TYPE                             DATA   AGE
ngc-api      Opaque                           1      8s
ngc-secret   kubernetes.io/dockerconfigjson   1      22s
```

### 2.2 Verify GPU Node(s) Labels & Taints

```bash
$kubectl describe node 172.16.122.203
Name:               172.16.122.203
Roles:              node
Labels:             beta.kubernetes.io/instance-type=VM.GPU.A10.1
                    name=gpu
                    node.kubernetes.io/instance-type=VM.GPU.A10.1
                    nvidia.com/gpu=true
Taints:             nvidia.com/gpu=present:NoSchedule
```
### 2.3 Deploy NIM
Create the Yaml file for Deployment and save as **LLM_deployment.yaml**

```yaml

image:
  repository: nvcr.io/nim/meta/llama3-8b-instruct
  pullPolicy: IfNotPresent
imagePullSecrets:
  - name: ngc-secret 
replicaCount: 1 # replicaCount Specify static replica count for deployment.
statefulSet:
  enabled: false
tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"
nodeSelector:
  nvidia.com/gpu: "true"
livenessProbe:
  enabled: true
  method: http
  command:
    - myscript.sh
  path: /v1/health/live
  initialDelaySeconds: 15
  timeoutSeconds: 1
  periodSeconds: 10
  successThreshold: 1
  failureThreshold: 3

readinessProbe:
  enabled: true
  path: /v1/health/ready  
  initialDelaySeconds: 15
  timeoutSeconds: 1
  periodSeconds: 10
  successThreshold: 1
  failureThreshold: 3

startupProbe:
  enabled: true
  path: /v1/health/ready  
  initialDelaySeconds: 40
  timeoutSeconds: 1
  periodSeconds: 10
  successThreshold: 1
  failureThreshold: 180
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
persistence:
  enabled: true
  storageClass: "customer-key"
  accessMode: ReadWriteOnce 
  stsPersistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain
    whenScaled: Retain
  size: 50Gi
```



```bash
helm install llama -n nim nim-llm-1.7.0.tgz -f LLM_deployment.yaml
NAME: llama
LAST DEPLOYED: Mon May 26 01:48:29 2025
NAMESPACE: nim
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing nim-llm.

**************************************************
| It may take some time for pods to become ready |
| while model files download                     |
**************************************************

Your NIM version is: 1.0.3
```

### 2.4 List Model Profiles

```bash
$kubectl exec -it -n nim llama-nim-llm-0 -- sh
$ list-model-profiles
SYSTEM INFO
- Free GPUs: <None>
- Non-free GPUs:
  -  [2236:10de] (0) NVIDIA A10 [current utilization: 85%]
MODEL PROFILES
- Compatible with system and runnable: <None>
- Compatible with system but not runnable due to low GPU free memory
  - 8835c31752fbc67ef658b20a9f78e056914fdef0660206d82f252d62fd96064d (vllm-fp16-tp1)
  - With LoRA support:
    - 8d3824f766182a754159e88ad5a0bd465b1b4cf69ecf80bd6d6833753e945740 (vllm-fp16-tp1-lora)
- Incompatible with system:
  - dcd85d5e877e954f26c4a7248cd3b98c489fbde5f1cf68b4af11d665fa55778e (tensorrt_llm-h100-fp8-tp2-latency)
  - f59d52b0715ee1ecf01e6759dea23655b93ed26b12e57126d9ec43b397ea2b87 (tensorrt_llm-l40s-fp8-tp2-latency)
  - 30b562864b5b1e3b236f7b6d6a0998efbed491e4917323d04590f715aa9897dc (tensorrt_llm-h100-fp8-tp1-throughput)
  - 09e2f8e68f78ce94bf79d15b40a21333cea5d09dbe01ede63f6c957f4fcfab7b (tensorrt_llm-l40s-fp8-tp1-throughput)
  - a93a1a6b72643f2b2ee5e80ef25904f4d3f942a87f8d32da9e617eeccfaae04c (tensorrt_llm-a100-fp16-tp2-latency)
  - e0f4a47844733eb57f9f9c3566432acb8d20482a1d06ec1c0d71ece448e21086 (tensorrt_llm-a10g-fp16-tp2-latency)
  - 879b05541189ce8f6323656b25b7dff1930faca2abe552431848e62b7e767080 (tensorrt_llm-h100-fp16-tp2-latency)
  - 24199f79a562b187c52e644489177b6a4eae0c9fdad6f7d0a8cb3677f5b1bc89 (tensorrt_llm-l40s-fp16-tp2-latency)
  - 751382df4272eafc83f541f364d61b35aed9cce8c7b0c869269cea5a366cd08c (tensorrt_llm-a100-fp16-tp1-throughput)
```

### 2.5 Port Forward

```bash
kubectl port-forward -n nim svc/llama-nim-llm 8000:8000
```

### 2.6 Send a Request
```bash
curl "http://localhost:8000/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{
      "model": "meta/llama3-8b-instruct",
      "messages": [
        {
          "role": "user",
          "content": "Hello!",
          "max_tokens": 600,
          "temperature": 0.3
        }
      ]
    }'



{"id":"cmpl-3c4b4ac38a2341ed98cff645702d8963","object":"chat.completion","created":1748242494,"model":"meta/llama3-8b-instruct","choices":[{"index":0,"message":{"role":"assistant","content":"Hello! It's nice to meet you. Is there something I can help you with, or would you like to chat?"},"logprobs":null,"finish_reason":"stop","stop_reason":128009}],"usage":{"prompt_tokens":12,"total_tokens":38,"completion_tokens":26}}
```

# Links
[NIM Deployment with Helm](https://docs.nvidia.com/nim/large-language-models/latest/deploy-helm.html)

[NIM Download](https://catalog.ngc.nvidia.com/orgs/nim/helm-charts/nim-llm)

[Nvidia NIM Supported Models](https://docs.nvidia.com/nim/large-language-models/latest/supported-models.html)

[Nvidia Monitoing with Prometheus](https://docs.nvidia.com/datacenter/cloud-native/gpu-telemetry/latest/kube-prometheus.html)