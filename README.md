
# Miércoles: Despliegue y Gestión de Aplicaciones en Kubernetes

## Duración: 3 horas

### Contenido:
1. **Despliegue de Aplicaciones Complejas**
   - **Deployments**: Un Deployment en Kubernetes gestiona la creación y escalado de Pods, asegurando que siempre haya una cantidad específica de Pods en ejecución.
   - **StatefulSets**: Utilizado para aplicaciones que requieren identidad persistente, como bases de datos.
   - **DaemonSets**: Asegura que un pod se ejecute en cada nodo del clúster, útil para tareas como monitoreo o almacenamiento de logs.

2. **Gestión de actualizaciones y rollback**
   - Kubernetes permite hacer actualizaciones con **rolling updates**, sin tiempo de inactividad.
   - Si algo falla, puedes revertir a una versión anterior usando **rollback**.

3. **Escalado y Autoscaling**
   - **Horizontal Pod Autoscaler (HPA)**: Escala automáticamente el número de réplicas de un Deployment en función de la carga de recursos, como la CPU.
   - **Cluster Autoscaler**: Ajusta el número de nodos en el clúster cuando no hay suficientes recursos para manejar los pods.

### Descripción de los Componentes:

- **Frontend (Nginx)**:
  El frontend está compuesto por un servidor web Nginx que sirve contenido web. Utiliza un **Deployment** con dos réplicas y un **Service** de tipo **LoadBalancer** para exponer la aplicación.

- **Backend (Node.js)**:
  El backend está compuesto por una aplicación Node.js. Se configura con un **Deployment** de dos réplicas y un **Service** de tipo **ClusterIP** para comunicarse con el frontend. El backend utiliza **ConfigMaps** y **Secrets** para manejar la configuración y las credenciales de manera segura.

- **Base de Datos (MySQL)**:
  La base de datos MySQL está configurada con un **Deployment** de una réplica y un **Service** de tipo **ClusterIP** para que el backend pueda conectarse. Los credenciales de la base de datos están almacenados en **Secrets**.

- **Horizontal Pod Autoscaler (HPA)**:
  Se configura para el backend, escalando el número de réplicas automáticamente según la utilización de CPU.

### Archivos Involucrados:

1. **frontend-deployment.yaml**: Despliega el servidor Nginx.
2. **backend-deployment.yaml**: Despliega la aplicación Node.js.
3. **db-deployment.yaml**: Despliega la base de datos MySQL.
4. **frontend-service.yaml**: Exponer el frontend con un LoadBalancer.
5. **backend-service.yaml**: Permite la comunicación entre el frontend y el backend.
6. **db-service.yaml**: Proporciona acceso al backend a la base de datos.
7. **hpa.yaml**: Configura el autoscaling para el backend.
8. **configmap.yaml**: Configura las variables de entorno para el backend.
9. **backend-secrets.yaml**: Almacena las credenciales del backend.
10. **db-secrets.yaml**: Almacena las credenciales de la base de datos MySQL.

### Paso a Paso para Desplegar la Aplicación:

1. **Clonar el repositorio** o acceder a los archivos YAML necesarios.

2. **Crear el clúster en AWS EKS** usando `eksctl`:

   - Instalar `eksctl` (si no lo tienes instalado):
     ```bash
     curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
     sudo mv /tmp/eksctl /usr/local/bin
     ```

   - Configurar tus credenciales de AWS:
     ```bash
     aws configure
     ```

   - Crear el clúster:
     ```bash
     eksctl create cluster \
       --name bootcamp-cluster \
       --region us-east-1 \
       --nodegroup-name linux-nodes \
       --node-type t3.medium \
       --nodes 3 \
       --nodes-min 1 \
       --nodes-max 4 \
       --managed
     ```

   - Configurar `kubectl` para apuntar al clúster:
     ```bash
     aws eks update-kubeconfig --region us-east-1 --name bootcamp-cluster
     ```

   - Verificar que el clúster está funcionando:
     ```bash
     kubectl get nodes
     ```

3. **Desplegar el frontend**:
   ```bash
   kubectl apply -f frontend-deployment.yaml
   kubectl apply -f frontend-service.yaml
   ```

4. **Desplegar el backend**:
   ```bash
   kubectl apply -f backend-deployment.yaml
   kubectl apply -f backend-service.yaml
   kubectl apply -f configmap.yaml
   kubectl apply -f backend-secrets.yaml
   ```

5. **Desplegar la base de datos MySQL**:
   ```bash
   kubectl apply -f db-deployment.yaml
   kubectl apply -f db-service.yaml
   kubectl apply -f db-secrets.yaml
   ```

6. **Configurar el Horizontal Pod Autoscaler (HPA)** para el backend:
   ```bash
   kubectl apply -f hpa.yaml
   ```

7. **Verificar que todo está corriendo correctamente**:
   ```bash
   kubectl get pods
   kubectl get services
   ```

8. **Probar la aplicación** accediendo a la IP externa del **LoadBalancer** creado por el frontend:
   - Usa el comando `kubectl get services` para obtener la **EXTERNAL-IP** del servicio **frontend-service** y accede a esa dirección desde el navegador.

---

Este ejercicio despliega una aplicación multi-servicio con frontend, backend y base de datos, y configura autoscaling para gestionar la carga automáticamente.
