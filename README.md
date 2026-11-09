# Práctica 1: desplegar la calculadora frontend

### Propósito

Crear un `Deployment` del frontend real de la calculadora con dos réplicas, límites
de recursos, sondas y una estrategia `RollingUpdate`. Después se actualizarán sus
imágenes, se provocará un fallo de descarga y se restaurará una revisión válida.

## Archivos

- `namespace.yaml`: aísla los recursos de esta práctica.
- `nginx-deployment.yaml`: versión inicial de la calculadora frontend.
- `nginx-deployment-v2.yaml`: segunda versión para probar el rollout.

## 1. Entrar al proyecto frontend

Cambia al directorio del proyecto frontend para construir las imágenes localmente.

```bash
cd ../calculadora-frontend
```

## 2. Construir la versión inicial

Crea la imagen local con la etiqueta inicial usada por el Deployment.

```bash
docker build -t calculadora-frontend:v1 .
```

## 3. Crear la etiqueta de actualización

Reutiliza la misma imagen con la etiqueta de la segunda versión.

```bash
docker tag calculadora-frontend:v1 calculadora-frontend:v2
```

## 4. Cargar la versión inicial en Minikube

Carga `v1` dentro de Minikube para que el nodo pueda ejecutarla sin descargarla de Internet.

```bash
minikube image load calculadora-frontend:v1
```

## 5. Cargar la actualización en Minikube

Carga también la etiqueta `v2`.

```bash
minikube image load calculadora-frontend:v2
```

## 6. Volver a los manifiestos

Regresa al directorio de la práctica.

```bash
cd ../practice-1
```

Los manifiestos ya usan esas imágenes locales y `imagePullPolicy: IfNotPresent`.

Si prefieres usar un registro externo, construye y publica las etiquetas con un
nombre como `tu-registro/calculadora-frontend:v1`, reemplaza ese nombre en ambos
manifiestos y ejecuta `docker push`.

## 7. Crear el namespace

Crea el namespace `practice-1`, que aísla los recursos de esta práctica.

```bash
kubectl apply -f namespace.yaml
```

## 8. Crear el Deployment inicial

Crea el Deployment con dos réplicas y las sondas configuradas.

```bash
kubectl apply -f nginx-deployment.yaml
```

## 9. Esperar el despliegue inicial

Espera a que Kubernetes termine de crear las réplicas y deje el Deployment disponible.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-1
```

## 10. Verificar Deployment, ReplicaSets y Pods

Comprueba que existan dos Pods de la calculadora en estado `Running` y `Ready`.

```bash
kubectl get deployments,replicasets,pods -n practice-1
```

## 11. Abrir el acceso local al frontend

Mantén este proceso abierto. Abre `http://localhost:8080` y detén el reenvío con `Ctrl+C`.

```bash
kubectl port-forward deployment/calculadora-frontend 8080:80 -n practice-1
```

## 12. Aplicar la versión nueva

Aplica la nueva imagen y crea una revisión del Deployment.

```bash
kubectl apply -f nginx-deployment-v2.yaml
```

## 13. Esperar el Rolling Update

Espera hasta que finalice el rollout de la nueva versión.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-1
```

## 14. Consultar el historial

Comprueba las revisiones registradas por Kubernetes.

```bash
kubectl rollout history deployment/calculadora-frontend -n practice-1
```

## 15. Simular una imagen inexistente

Modifica temporalmente la imagen del contenedor para provocar un fallo de descarga.

```bash
kubectl set image deployment/calculadora-frontend frontend=calculadora-frontend:inexistente -n practice-1
```

## 16. Registrar el motivo del cambio

Asocia un texto descriptivo al cambio para que aparezca en el historial de revisiones.

```bash
kubectl annotate deployment/calculadora-frontend kubernetes.io/change-cause="imagen inexistente" --overwrite -n practice-1
```

## 17. Comprobar el fallo

Confirma que la nueva revisión no puede descargar la imagen.

```bash
kubectl rollout status deployment/calculadora-frontend --timeout=90s -n practice-1
```

## 18. Restaurar la revisión anterior

Restaura la revisión anterior que utiliza una imagen válida.

```bash
kubectl rollout undo deployment/calculadora-frontend -n practice-1
```

## 19. Verificar el rollback

Espera a que la revisión restaurada quede disponible.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-1
```

## 20. Provocar la eliminación de los Pods

Elimina los Pods seleccionados; el Deployment los reemplazará.

```bash
kubectl delete pod -l app.kubernetes.io/name=calculadora-frontend -n practice-1
```

## 21. Observar la autosanación

Mantén visible la lista hasta comprobar que aparecen Pods nuevos.

```bash
kubectl get pods -n practice-1 --watch
```

## 22. Limpiar la práctica

Elimina el namespace y todos sus recursos asociados.

```bash
kubectl delete namespace practice-1
```
