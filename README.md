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

Construye y publica dos etiquetas distintas de la misma aplicación. La segunda
imagen solo necesita existir para demostrar que Kubernetes cambia entre etiquetas;
podrás sustituirla por una versión propia del frontend cuando amplíes la práctica:

```bash
cd ../calculadora-frontend
docker build -t tu-registro/calculadora-frontend:v1 .
docker push tu-registro/calculadora-frontend:v1
docker tag tu-registro/calculadora-frontend:v1 tu-registro/calculadora-frontend:v2
docker push tu-registro/calculadora-frontend:v2
cd ../practice-1
```

Reemplaza `your-registry` por `tu-registro` en ambos manifiestos antes de aplicarlos.

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
kubectl set image deployment/calculadora-frontend frontend=tu-registro/calculadora-frontend:inexistente -n practice-1
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
