# Deploy-an-AI-model-on-GKE-with-NVIDIA-NIM
Deploy an AI model on GKE with NVIDIA NIM

1. Create NVIDIA API key
2. Create a GCP account
3. Create GKE cluster with GPU
4. Open a cloud shell
5. export PROJECT_ID=<YOUR PROJECT ID>
export REGION=<YOUR REGION>
export ZONE=<YOUR ZONE>
export CLUSTER_NAME=nim-demo
export NODE_POOL_MACHINE_TYPE=g2-standard-16
export CLUSTER_MACHINE_TYPE=e2-standard-4
export GPU_TYPE=nvidia-l4
export GPU_COUNT=1
6. Please note you may have to change the values for NODE_POOL_MACHINE_TYPE, CLUSTER_MACHINE_TYPE and GPU_TYPE based on what type of Compute Instance and GPUs you are using.
7. gcloud container clusters create ${CLUSTER_NAME} \
    --project=${PROJECT_ID} \
    --location=${ZONE} \
    --release-channel=rapid \
    --machine-type=${CLUSTER_MACHINE_TYPE} \
    --num-nodes=1
8. gcloud container node-pools create gpupool \
    --accelerator type=${GPU_TYPE},count=${GPU_COUNT},gpu-driver-version=latest \
    --project=${PROJECT_ID} \
    --location=${ZONE} \
    --cluster=${CLUSTER_NAME} \
    --machine-type=${NODE_POOL_MACHINE_TYPE} \
    --num-nodes=1
9. Configure NVIDIA NGC API Key
10. export NGC_CLI_API_KEY="<YOUR NGC API KEY>"
11. Deploy and test NVIDIA NIM
12. Helm chart fetch https://helm.ngc.nvidia.com/nim/charts/nim-llm-1.3.0.tgz --username='$oauthtoken' --password=$NGC_CLI_API_KEY
13. kubectl create namespace nim
14. kubectl create secret docker-registry registry-secret --docker-server=nvcr.io --docker-username='$oauthtoken'     --docker-password=$NGC_CLI_API_KEY -n nim
15. kubectl create secret generic ngc-api --from-literal=NGC_API_KEY=$NGC_CLI_API_KEY -n nim
16. cat <<EOF > nim_custom_value.yaml
image:
  repository: "nvcr.io/nim/meta/llama3-8b-instruct" # container location
  tag: 1.0.0 # NIM version you want to deploy
model:
  ngcAPISecret: ngc-api  # name of a secret in the cluster that includes a key named NGC_CLI_API_KEY and is an NGC API key
persistence:
  enabled: true
imagePullSecrets:
  -   name: registry-secret # name of a secret used to pull nvcr.io images, see https://kubernetes.io/docs/tasks/    configure-pod-container/pull-image-private-registry/
17. helm install my-nim nim-llm-1.1.2.tgz -f nim_custom_value.yaml --namespace nim
18. kubectl get pods -n nim
19. Testing NIM deployment: kubectl port-forward service/my-nim-nim-llm 8000:8000 -n nim
20. curl -X 'POST' \
  'http://localhost:8000/v1/chat/completions' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "messages": [
    {
      "content": "You are a polite and respectful chatbot helping people plan a vacation.",
      "role": "system"
    },
    {
      "content": "What should I do for a 4 day vacation in Spain?",
      "role": "user"
    }
  ],
  "model": "meta/llama3-8b-instruct",
  "max_tokens": 128,
  "top_p": 1,
  "n": 1,
  "stream": false,
  "stop": "\n",
  "frequency_penalty": 0.0
}'
21. Clean up gcloud container clusters delete $CLUSTER_NAME --zone=$ZONE
   
