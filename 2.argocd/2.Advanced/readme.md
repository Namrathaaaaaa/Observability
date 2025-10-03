->user management , rbac 
argocd account list
kubectl -n argocd patch configmap argocd-cm  --patch='{"data":{"accounts.rahul":"apiKey,login"}}'
argocd account update-password --account rahul  
kubectl -n argocd patch configmap argocd-rbac-cm --patch='{"data":{"policy.default": "read:readonly"}}'

-> Dex Okta connector 
