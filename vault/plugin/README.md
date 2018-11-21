# Install the Kubernetes plugin to Vault

Create a ServiceAccount and ClusterRoleBinding for Vault to use, when communicating with Kubernetes API
```
kubectl apply -f vault-sa.yaml -n sekira
kubectl apply -f vault-sa-crb.yaml -n sekira
```

Authenticate with Root token to Vault, through the port-forward made in the main Vault README.md
```
vault login 3a067f6c-f7c2-0606-e027-fe933a6cf8d7
```

Get the token from the service account created for Vault to communicate with the API server
```
token=$(kubectl get serviceaccounts -n sekira vault-auth -o=jsonpath='{.secrets[].name}' | xargs -0 -I {} kubectl get secret {} -o json -n sekira -o=jsonpath='{.data.token}' | base64 -d)
ca=$(kubectl get serviceaccounts -n sekira vault-auth -o=jsonpath='{.secrets[].name}' | xargs -0 -I {} kubectl get secret {} -o json -n sekira -o=jsonpath='{.data.ca\.crt}' | base64 -d)
echo $token
echo $ca
```

Connect Vault to Kubernetes
```
vault auth enable kubernetes
vault write auth/kubernetes/config toeken_reviewer_jwt="$token" kubernetes_host=https://kubernetes kubernetes_ca_cert="$ca"
```
