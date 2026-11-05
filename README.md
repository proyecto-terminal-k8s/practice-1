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
```

Cambia al directorio del proyecto frontend para construir sus imágenes.

```bash
docker build -t calculadora-frontend:v1 .
```

Crea la imagen local con la etiqueta inicial usada por el Deployment.

```bash
docker tag calculadora-frontend:v1 calculadora-frontend:v2
```

Reutiliza la misma imagen con la etiqueta de la segunda versión.

```bash
minikube image load calculadora-frontend:v1
```

Carga `v1` dentro de Minikube para que el nodo pueda ejecutarla sin descargarla de Internet.

```bash
minikube image load calculadora-frontend:v2
```

Carga también la etiqueta `v2`.

```bash
cd ../practice-1
```

Los manifiestos ya usan esas imágenes locales y `imagePullPolicy: IfNotPresent`.

Si prefieres usar un registro externo, construye y publica las etiquetas con un
nombre como `tu-registro/calculadora-frontend:v1`, reemplaza ese nombre en ambos
manifiestos y ejecuta `docker push`.

## 1. Desplegar la versión inicial

```bash
kubectl apply -f namespace.yaml
```

Crea el namespace `practice-1`, que aísla los recursos de esta práctica.

```bash
kubectl apply -f nginx-deployment.yaml
```

Crea el Deployment inicial con dos réplicas y las sondas configuradas.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-1
```

Espera a que Kubernetes termine de crear las réplicas y deje el Deployment disponible.

```bash
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
```

Aplica la nueva imagen y crea una revisión del Deployment.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-1
```

Espera hasta que finalice el rollout de la nueva versión.

```bash
kubectl rollout history deployment/calculadora-frontend -n practice-1
```

## 4. Probar un fallo y hacer rollback

```bash
kubectl set image deployment/calculadora-frontend frontend=calculadora-frontend:inexistente -n practice-1
```

Modifica temporalmente la imagen del contenedor para simular que una etiqueta nueva no existe.

```bash
kubectl annotate deployment/calculadora-frontend kubernetes.io/change-cause="imagen inexistente" --overwrite -n practice-1
```

Asocia un texto descriptivo al cambio para que aparezca en el historial de revisiones.

```bash
kubectl rollout status deployment/calculadora-frontend --timeout=90s -n practice-1
```

Confirma que la nueva revisión no puede descargar la imagen, por lo que el Deployment quedará bloqueado.

```bash
kubectl rollout undo deployment/calculadora-frontend -n practice-1
```

Restaura la revisión anterior que sí utiliza una imagen válida.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-1
```

## 5. Comprobar la autosanación

```bash
kubectl delete pod -l app.kubernetes.io/name=calculadora-frontend -n practice-1
```

Elimina los Pods que coincidan con el selector; el Deployment los reemplazará.

```bash
kubectl get pods -n practice-1 --watch
```

El Deployment debe reemplazar el Pod eliminado sin intervención manual.

## Limpieza

```bash
kubectl delete namespace practice-1
```
