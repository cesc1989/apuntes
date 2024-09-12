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

## Tocó sin Luna CLI

Se instaló ahora por Brew

```
brew install kubectl
brew install kubectx
```

Ahora se ejecutan unos comandos para configurar el archivo `~/.kube/config`.

NOTA: fíjate que la carpeta sea `.kube`. Antes había configurado manualmente otra carpeta.

```bash
aws eks update-kubeconfig --region us-west-2 --name alpha-primary-v3 --profile alpha

aws eks update-kubeconfig --region us-west-2 --name omega-primary-v3 --profile omega
```

Finalmente, con el comando `kubectx` se elige el cluster y luego sí se puede pasar a los comandos para elegir un namespace y el pod al cual iniciar sesión.

```bash
# gets namespaces
kubectl get ns

# gets pods within a namespace
kubectl get po -n [NAMESPACE-NAME]

# starts a session inside of an application pod
kubectl exec -it -n [NAMESPACE-NAME] [POD-NAME] -- sh

# streams logs from application, alternatively, without the `-f` to see logs up until that point
kubectl logs -n [NAMESPACE-NAME] [POD-NAME] -f
```

# Ejemplos de Comandos

Para listar e ingresar a los pods.

## Patient Self Report (backend)

```bash
kubectl get ns

kubectl get po -n patient-forms-backend

kubectl exec -it -n patient-forms-backend patient-forms-backend-[ID] -- sh
```

## Edge

```bash
kubectl get ns

kubectl get po -n backend

kubectl exec -it -n backend backend-[ID] -- sh
```

## Clinical Dashboard

```bash
kubectl get ns

kubectl get po -n physician-portal

kubectl exec -it -n physician-portal physician-portal-[ID] -- sh
```

## Therapist Signup (backend)

```bash
kubectl get ns

kubectl get po -n therapist-credentialing-backend

kubectl exec -it -n therapist-credentialing-backend therapist-credentialing-backend-[ID] -- sh
```

## Marketplace

```bash
kubectl get ns

kubectl get po -n marketplace

kubectl exec -it -n marketplace marketplace-[ID] -- sh
```