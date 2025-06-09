
# Instalación de AWX en Debian 12.

En esta documentación desplegamos AWX en un Debian12 usando Kubernetes y k3s como orquestador.

## 1. Actualizamos el sistema

```bash
apt update && apt upgrade -y
```

## 2. Instalamos las herramientas básicas

```bash
apt install -y curl wget vim git net-tools gnupg2 lsb-release ca-certificates apt-transport-https
```

## 3. Instalamos Kubernetes con k3s.

En este caso optamos como orquestador por k3s, que es mucho más simple y menos pesado que kubeadm o minikube.

```bash
curl -sfL https://get.k3s.io | sh -
```

Verificamos que funciona:

```bash
kubectl get nodes
```

## 4. Instalamos Kustomize (para despliegues con AWX Operator)

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
mv kustomize /usr/local/bin/
```

## 5. Instalamos AWX Operator

**Clonamos el repositorio oficial**

```bash
git clone https://github.com/ansible/awx-operator.git
cd awx-operator
git checkout 2.14.0
```

**Creamos el namespace para AWX**

```bash
kubectl create namespace awx
```

**Instalamos el operador**


```bash
make deploy NAMESPACE=awx
```

:::tip
El comando make deploy usa Kustomize para instalar el operador.
:::

## 6. Instalamos la instancia de AWX

Creamos un archivo `awx.yml` con el siguiente contenido:

```yaml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
  namespace: awx
spec:
  service_type: nodeport
  ingress_type: none
```

Lo aplicamos:

```bash
kubectl apply -f awx.yml
```

## 7. Comprobamos el estado de los pods

```bash
kubectl get pods -n awx -w
```

Veremos algo como:

* `awx-postgres-*` running
* `awx-web` running
* `awx-task` running

## 8. Accedemos a la interfaz web de AWX

Buscamos el puerto externo asignado (NodePort):

```bash
kubectl get svc -n awx
```

Verás algo como:

```
NAME          TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
awx-service   NodePort   10.43.X.X       <none>        80:31XXX/TCP     3m
```

El número tras los dos puntos (`31XXX`) es el puerto NodePort externo.

Accedemos a AWX, en mi caso:

[https://192.168.1.129:31441](https://daedalusawx.duckdns.org)

## 9. Credenciales iniciales de AWX

El usuario por defecto es:

```
admin
```

Obtenemos la contraseña:

```bash
kubectl get secret awx-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode
echo
```

Y ya estaría, con el usuario admin y la contraseña obtenida tenemos nuestro AWX funcionando y acceso a la plataforma.
