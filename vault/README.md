# Installing and setup of Vault in Kubernetes

## Prerequsits
Install Helm and Tiller

## Install the Operator
helm install stable/vault-operator --name sekira \
  --set etcd-operator.enabled=true \
  --namespace=sekira

## Deploy a Vault cluster with ETCD
```
kubectl apply -f vault.yaml -n sekira

```

Documentation
```
https://github.com/helm/charts/tree/master/stable/etcd-operator
https://github.com/helm/charts/tree/master/stable/etcd-operator#configuration
```

## Get the vault client
```
cd /tmp
wget https://releases.hashicorp.com/vault/0.11.1/vault_0.11.1_linux_amd64.zip
unzip vault_0.11.1_linux_amd64.zip
sudo mv vault /usr/bin/

```

## Initialize Vault
In a tmp windows, run the following port-forward
```
kubectl -n sekira get vault sekira -o jsonpath='{.status.vaultStatus.sealed[0]}' | xargs -0 -I {} kubectl -n sekira port-forward {} 8200
```

In primary windows, initialize Vault
```
export VAULT_ADDR='https://localhost:8200'
export VAULT_SKIP_VERIFY="true"
vault operator init

Unseal Key 1: 9cPP/n9Jo6xR1dbsVNyre8bLI7UNa9Wmp65PXcYxlAg7
Unseal Key 2: TA5h4kYvBSr3JLEryNY2c9YP2EDQy8jEiqsv/oDRHfFp
Unseal Key 3: 0P/PQmhbGH5lJNzim0AdCU0cnKOYre0sL9p4KbdcqDUB
Unseal Key 4: 3C3rhimJJOUPmGN2FmAcXjiYJhRrX4nRBpO4aYEXkkex
Unseal Key 5: BZgLMif9EW9TYErVTSQjfBJsZSGBp8qLK4JFQPX10UKF

Initial Root Token: 3a067f6c-f7c2-0606-e027-fe933a6cf8d7

```

## Unseal Vault
```
vault operator unseal
vault operator unseal
vault operator unseal

```

Brake the port-forward, and run it again, and then run the unseal again. This will connect to the other node, that is still sealed, and unseal that as well.


Now install the Kubernetes plugin as described in ```plugin/README.md```
