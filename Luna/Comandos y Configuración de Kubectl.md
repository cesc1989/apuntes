# Instalación y configuración de kubectl

Instalación en Apple Sillicon

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

sudo chown root: /usr/local/bin/kubectl
```

## Uso para Luna CLI

Necesito permisos suficientes para poder ejecutar este comando.

```
luna eks configure --profile alpha
```

Con los permisos obtenidos:

```
kubectl config current-context

kubectl get po -n REPO_NAME
```

Y para ingresar al pod

```
kubectl exec -it -n POD_NAME ID -- bash
```