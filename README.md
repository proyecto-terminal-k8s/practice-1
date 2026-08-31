# Práctica 0: crear un clúster con Minikube

## Propósito

Crear un clúster local de tres nodos con el driver de Docker, comprobar sus
componentes básicos y practicar las operaciones de pausa, detención y borrado de
Minikube.

Esta práctica no contiene manifiestos YAML: el clúster se crea con Minikube antes
de poder enviar objetos a la API de Kubernetes.

## 1. Comprobar los requisitos

Docker debe responder sin errores y las dos herramientas deben estar instaladas:

```bash
docker info
minikube version
kubectl version --client
```

Si `docker info` devuelve un error de permisos, corrige el acceso de tu usuario al
daemon de Docker antes de continuar.

## 2. Crear el clúster

Desde una terminal ejecuta:

```bash
minikube start --driver=docker --memory=2048 --nodes=3
```

Se usa el perfil predeterminado `minikube`. Minikube crea un nodo de plano de
control y dos nodos de trabajo. Cada nodo recibe 2048 MiB de memoria.

## 3. Verificar el resultado

```bash
minikube status
kubectl cluster-info
kubectl get nodes -o wide
```

El resultado esperado de `kubectl get nodes` es una lista de tres nodos en estado
`Ready`. Para observar los recursos que Minikube instaló por defecto:

```bash
kubectl get namespaces
kubectl get pods --all-namespaces
```

## 4. Administrar el ciclo de vida

Ejecuta estos comandos uno por uno cuando quieras probar cada operación:

```bash
minikube pause
minikube unpause
minikube stop
minikube start
```

- `pause` suspende los recursos del clúster sin eliminarlo.
- `unpause` reanuda un clúster pausado.
- `stop` apaga sus nodos y conserva su configuración.
- `start` vuelve a iniciar el perfil existente.

## Limpieza

No elimines el clúster si vas a continuar con las demás prácticas. Cuando hayas
terminado todo el recorrido puedes borrarlo con:

```bash
minikube delete
```

Este último comando elimina los nodos y datos del clúster local.
