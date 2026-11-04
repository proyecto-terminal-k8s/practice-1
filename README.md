# Práctica 1: desplegar la calculadora frontend

### Propósito

Crear un `Deployment` del frontend real de la calculadora con dos réplicas, límites
de recursos, sondas y una estrategia `RollingUpdate`. Después se actualizarán sus
imágenes, se provocará un fallo de descarga y se restaurará una revisión válida.

## Archivos

- `namespace.yaml`: aísla los recursos de esta práctica.
- `nginx-deployment.yaml`: versión inicial de la calculadora frontend.
- `nginx-deployment-v2.yaml`: segunda versión para probar el rollout.

## Preparar las imágenes

Para Minikube, construye las dos etiquetas y cárgalas directamente en el clúster.
Así no necesitas crear ni autenticarte en ningún registro:

```bash
cd ../calculadora-frontend
docker build -t calculadora-frontend:v1 .
docker tag calculadora-frontend:v1 calculadora-frontend:v2
minikube image load calculadora-frontend:v1
minikube image load calculadora-frontend:v2
cd ../practice-1
```

Los manifiestos ya usan esas imágenes locales y `imagePullPolicy: IfNotPresent`.

Si prefieres usar un registro externo, construye y publica las etiquetas con un
nombre como `tu-registro/calculadora-frontend:v1`, reemplaza ese nombre en ambos
manifiestos y ejecuta `docker push`.

## 1. Desplegar la versión inicial

```bash
kubectl apply -f namespace.yaml
kubectl apply -f nginx-deployment.yaml
kubectl rollout status deployment/calculadora-frontend -n practice-1
kubectl get deployments,replicasets,pods -n practice-1
```

Debe haber dos Pods de la calculadora `Running` y `Ready`.

## 2. Acceder al frontend

```bash
kubectl port-forward deployment/calculadora-frontend 8080:80 -n practice-1
```

Abre `http://localhost:8080`. Detén el reenvío con `Ctrl+C`.

## 3. Probar el Rolling Update

```bash
kubectl apply -f nginx-deployment-v2.yaml
kubectl rollout status deployment/calculadora-frontend -n practice-1
kubectl rollout history deployment/calculadora-frontend -n practice-1
```

## 4. Probar un fallo y hacer rollback

```bash
kubectl set image deployment/calculadora-frontend frontend=calculadora-frontend:inexistente -n practice-1
kubectl annotate deployment/calculadora-frontend kubernetes.io/change-cause="imagen inexistente" --overwrite -n practice-1
kubectl rollout status deployment/calculadora-frontend --timeout=90s -n practice-1
kubectl rollout undo deployment/calculadora-frontend -n practice-1
kubectl rollout status deployment/calculadora-frontend -n practice-1
```

## 5. Comprobar la autosanación

```bash
kubectl delete pod -l app.kubernetes.io/name=calculadora-frontend -n practice-1
kubectl get pods -n practice-1 --watch
```

El Deployment debe reemplazar el Pod eliminado sin intervención manual.

## Limpieza

```bash
kubectl delete namespace practice-1
```
