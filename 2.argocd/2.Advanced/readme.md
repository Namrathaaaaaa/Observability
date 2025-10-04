->user management , rbac 
argocd account list
kubectl -n argocd patch configmap argocd-cm  --patch='{"data":{"accounts.rahul":"apiKey,login"}}'
argocd account update-password --account rahul  
kubectl -n argocd patch configmap argocd-rbac-cm --patch='{"data":{"policy.default": "read:readonly"}}'

-> Dex Okta connector 

-> sealed secrets 
download bitnami link 
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
install kubeseal 
brew install kubeseal cli
kubectl get secret -n kube-system sealed-secrets-key98fn9 -o json| jq -r .data'."tls.crt"' | base64 -d > /root/sealedSecret-publicCert.crt

kubectl create secret generic app-crds --from-literal=apikey=zaCELgL-0imfnc8mVLWwsAawjYr4Rx-Af50DDqtlx --from-literal=username=admin-dev-group --from-literal=password=paSsw0rD-1erT-diS -o yaml --dry-run=client > mysql-password_k8s-secret.yaml
pass public secret -> gives encryptes value

kubeseal -o yaml --scope cluster-wide --cert sealedSecret-publicCert.crt < mysql-password_k8s-secret.yaml > mysql-password_sealed-secret.yaml

cat mysql-password_sealed-secret.yaml

then replace the secrets file 

-> hashicorp vault with argocd plugin

argocd-vault-plugin generate -c /root/vault.env - < /root/secret.yaml > /root/secret_updated.yaml
