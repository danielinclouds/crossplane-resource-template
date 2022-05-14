# Crossplane Resource Template
A template that you can use for crating more complex compositions.  
<br>


### Create GCP service account for Crossplane

```sh
PROJECT_ID=""

gcloud iam service-accounts create "crossplane" --project "$PROJECT_ID"

gcloud iam service-accounts keys create "credentials.json" \
    --iam-account="crossplane@${PROJECT_ID}.iam.gserviceaccount.com"
```
<br>


### Create kind cluster with Crossplane

```sh
kind create cluster

kubectl create namespace crossplane-system

# Secret for Crossplane
kubectl create secret generic crossplane -n crossplane-system --from-file=credentials.json

# Install Crossplane
helm upgrade --install \
    crossplane crossplane-stable/crossplane \
    --namespace crossplane-system \
    --wait

# Install GCP provider
cat <<EOF | kubectl apply -f -
---
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-gcp
  namespace: crossplane-system
spec:
  package: crossplane/provider-gcp:v0.21.0
EOF

cat <<EOF | kubectl apply -f -
---
apiVersion: gcp.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: provider-gcp
  namespace: crossplane-system
spec:
  projectID: ${PROJECT_ID}
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: crossplane
      key: credentials.json
EOF
```
<br>


### Install composition

```sh
kubectl apply -f MyBucket/composition.yaml
kubectl apply -f MyBucket/xrd.yaml
```
<br>


### Create a claim

```sh
kubectl apply -f claim.yaml
```
<br>

