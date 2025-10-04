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
pass public secret -> gives encryptes value

-> hashicorp vault with argocd plugin
