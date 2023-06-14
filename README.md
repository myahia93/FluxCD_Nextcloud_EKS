## Kubernetes
```shell
# Create cluster & set kubeconfig
eksctl create cluster -f cluster.yml
aws eks update-kubeconfig --name projet-cluster --region us-east-1

# Create the namespace
kubectl create ns nextcloud
kubectl config set-context --current --namespace=nextcloud
```

## Nextcloud
```shell
# Adding the helm repo for nextcloud
helm repo add nextcloud https://nextcloud.github.io/helm/

# Installing & Updating
helm install nextcloud nextcloud/nextcloud -f values.yaml -n nextcloud
helm install nextcloud nextcloud/nextcloud -f values.yaml -n nextcloud

# Connect to nextcloud
export POD_NAME=$(kubectl get pods --namespace nextcloud -l "app.kubernetes.io/name=nextcloud" -o jsonpath="{.items[0].metadata.name}")
echo http://127.0.0.1:8080/
kubectl port-forward $POD_NAME 8080:80 -n nextcloud

# Get credentials
kubectl get secret --namespace nextcloud nextcloud -o jsonpath="{.data.nextcloud-password}" | base64 --decode
```

## Helm
```shell
# List all helm releases
helm list --all-namespaces

# Uninstall a helm release
helm uninstall <release> -n <namespace>
```

## Flux CD
```shell
# Install Flux
export GITHUB_TOKEN="<token>"
flux bootstrap github --owner=myahia93 --repository=FluxCD_Nextcloud_EKS --path=clusters/project --personal --private=false

# Deploying the UI
flux create source helm ww-gitops \
 --url=https://helm.gitops.weave.works \
 --export > ./clusters/project/weave-gitops-source.yml

flux create helmrelease ww-gitops \
 --source=HelmRepository/ww-gitops \
 --chart=weave-gitops \
 --values=./weave-gitops-values.yml \
 --export > ./clusters/project/weave-gitops-helmrelease.yaml

# Access to the UI
kubectl port-forward pod/ww-gitops-weave-gitops-6fb6f5fb57-skcmn 9001:9001 -n flux-system
```

## Nextcloud Using Flux
```shell
# Create the Helm source
flux create source helm nextcloud \
  --url=https://nextcloud.github.io/helm/ \
  --export > ./clusters/project/nextcloud-source.yaml

# Create the HelmRelease
flux create helmrelease nextcloud \
  --source=HelmRepository/nextcloud \
  --chart=nextcloud \
  --target-namespace=nextcloud \
  --values=./nextcloud-values.yaml \
  --interval=1m \
  --export > ./clusters/project/nextcloud-helmrelease.yaml

# Connect to Nextcloud
k port-forward svc/nextcloud-nextcloud 8080:8080
```