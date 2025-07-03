# Instal LLM using from Hugging Face

## 1. Get the Helm Chart Repository
```bash
helm repo add hugs https://raw.githubusercontent.com/huggingface/hugs-helm-chart/main/charts/hugs
helm repo update hugs
```

## 2. Explore the Hugs Chart Repository
```bash
helm search repo hugs
NAME     	CHART VERSION	APP VERSION	DESCRIPTION
hugs/hugs	0.0.4        	1.16.0     	Helm Chart for Hugging Face Generative AI Servi...
```

## 3. Get the default values.yaml configuration file:
```bash
helm show values hugs/hugs > values.yaml
```

## 4. Modify the values.yaml file appropiatley
```bash
image:
  repository: hfhugs
  name: meta-llama/llama-3.1-8B-Instruct
  tag: latest
resources:
  limits:
    nvidia.com/gpu: 1
  requests:
    nvidia.com/gpu: 1
tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"
```    
## 5. Create Secrets for Hugging Face

### 5.1. Install Hugging Face CLI
To generate a custom token for the Hugging Face Hub, you can follow the instructions at https://huggingface.co/docs/hub/en/security-tokens; and the recommended way of setting it is to install the huggingface_hub Python SDK as follows:

```bash
pip install huggingface_hub
```
### 5.2. Login in with the generated token with read-access over the gated/private model:

```bash
huggingface-cli login
```

### 5.3. Finally, you can create the Kubernetes secret with the generated token for the Hugging Face Hub as follows using the huggingface_hub Python SDK to retrieve the token

```bash
kubectl create namespace hugs

export HF_TOKEN=hf_xxxxxxxxxxx

kubectl create secret generic hf-secret -n hugs --from-literal=hf_token=$HF_TOKEN
```

## 6. Deploy the LLM


### 5.2 Deploy the LLM
```bash
helm install hugs -n hugs hugs/hugs --values values.yaml
```





```bash
docker login registry.hf.space
username: nafey1@gmail.com
password: <HF-TOKEN>

# References
[HUGS on Kubernetes](https://huggingface.co/docs/hugs/how-to/kubernetes)