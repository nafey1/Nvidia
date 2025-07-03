# NIM with GPU Operator on Kubernetes for LLM Deployments



## 1. Download the NIM for Helm Charts
>**Make sure to export the value NGC_API_KEY before attempting the following operations**

```bash
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia && helm repo update
```

## 2. Delploy GPU Operator
```bash
helm install gpu-operator -n gpu-operator nvidia/gpu-operator --create-namespace --wait
```

## 3. Deploy the NIM Service
```bash
helm install nim-operator nvidia/k8s-nim-operator --create-namespace -n nim-operator
```



## 4. Create a ImagePull Secrets

```bash
kubectl create ns nim-service

kubectl create secret -n nim-service docker-registry ngc-secret \
    --docker-server=nvcr.io \
    --docker-username='$oauthtoken' \
    --docker-password=$NGC_API_KEY

kubectl create secret -n nim-service generic ngc-api-secret \
    --from-literal=NGC_API_KEY=$NGC_API_KEY
```

### 4.1. Verify the NGC Secrets

```bash
kubectl get secrets -n nim-service
NAME             TYPE                             DATA   AGE
ngc-api-secret   Opaque                           1      24s
ngc-secret       kubernetes.io/dockerconfigjson   1      25s
```

## 5. Create a NIM Service using the configuration below

```yaml
apiVersion: apps.nvidia.com/v1alpha1
kind: NIMService
metadata:
  name: llama3-8b-instruct # Change as per requirement
  namespace: nim-service
spec:
  image:
    repository: nvcr.io/nim/meta/llama3-8b-instruct # Change as per requirement
    tag: 1.0.3
    pullPolicy: IfNotPresent
    pullSecrets:
      - ngc-secret
  authSecret: ngc-api-secret
  storage:
    pvc:
      create: true
      size: "50G"
      volumeAccessMode: "ReadWriteOnce"
      storageClass: "customer-key" # Change as per requirement
  replicas: 1
  resources:
    limits:
      nvidia.com/gpu: 1 # Change as per requirement
  expose:
    service:
      type: ClusterIP
      port: 8000
```

## 6. Test the Deployed NIM
```bash
kubectl run --rm -it -n default curl --image=curlimages/curl:latest -- sh

curl -X "POST" \
 'http://llama3-8b-instruct.nim-service:8000/v1/chat/completions' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
        "model": "meta/llama3-8b-instruct",
        "messages": [
        {
          "content":"Create a 4 day travel itinerary for Paris?",
          "role": "user"
        }],
        "top_p": 1,
        "n": 1,
        "max_tokens": 6000,
        "stream": false,
        "frequency_penalty": 0.0,
        "stop": ["STOP"]
      }'
```


# Links
[NIM Deployment with Helm](https://docs.nvidia.com/nim/large-language-models/latest/deploy-helm.html)

[NIM Download](https://catalog.ngc.nvidia.com/orgs/nim/helm-charts/nim-llm)

[Nvidia NIM Supported Models](https://docs.nvidia.com/nim/large-language-models/latest/supported-models.html)

[Nvidia Monitoing with Prometheus](https://docs.nvidia.com/datacenter/cloud-native/gpu-telemetry/latest/kube-prometheus.html)

[Installing the NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html)