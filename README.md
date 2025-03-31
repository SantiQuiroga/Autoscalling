# Proyecto de Auto Scaling en Kubernetes

Este proyecto demuestra cómo configurar varios métodos de autoescalado en un clúster de Kubernetes, específicamente utilizando **Horizontal Pod Autoscaler (HPA)**, **Vertical Pod Autoscaler (VPA)** y **KEDA** (Kubernetes Event-Driven Autoscaler). Cada uno de estos métodos tiene sus características y casos de uso ideales para gestionar el escalado de aplicaciones de manera eficiente según el consumo de recursos y eventos.

## **Objetivos del Proyecto**

- **Horizontal Pod Autoscaler (HPA):** Escalar el número de réplicas de un pod en función de métricas como el uso de CPU o memoria. Ideal para escalar aplicaciones con cargas de trabajo que requieren más réplicas conforme aumentan las solicitudes.

- **Vertical Pod Autoscaler (VPA):** Ajustar automáticamente los recursos (CPU y memoria) de los pods, para que se adapten a la carga de trabajo. Es útil para aplicaciones donde el uso de recursos no es constante, pero los pods individuales necesitan más recursos a medida que crecen.

- **KEDA (Kubernetes Event-Driven Autoscaler):** Permite escalar aplicaciones en función de eventos como el número de mensajes en una cola, métricas personalizadas, entre otras. Es ideal para aplicaciones basadas en eventos como microservicios que procesan mensajes o tareas asíncronas.

## **Requisitos**

Antes de comenzar, asegúrate de tener los siguientes requisitos:

- **Docker** instalado y en funcionamiento.
- **Minikube** instalado para ejecutar un clúster de Kubernetes local.
- **kubectl** instalado para interactuar con el clúster de Kubernetes.
- **Helm** instalado para gestionar aplicaciones de Kubernetes mediante charts.
- **KEDA** instalado para el escalado basado en eventos.

## **Estructura de Archivos del Proyecto**

```plaintext
k8s_autoscaling_advanced/
├── deployment.yaml    # Definición del Deployment de Nginx
├── service.yaml       # Configuración del servicio para Nginx
├── vpa.yaml           # Configuración del Vertical Pod Autoscaler
└── scaledobject.yaml  # Configuración de KEDA
```

Cada archivo tiene una función específica:

- **deployment.yaml:** Define el Deployment de la aplicación (Nginx en este caso).
- **service.yaml:** Expone el servicio para que sea accesible dentro del clúster.
- **vpa.yaml:** Configura el Vertical Pod Autoscaler para ajustar los recursos de los pods.
- **scaledobject.yaml:** Configura el KEDA para escalar según los eventos o métricas definidas.

## **Verificación y Monitoreo del Escalado**

A lo largo del proyecto, se incluyen varios comandos para verificar el estado de los pods, deployments, HPA, VPA y KEDA, y así asegurarse de que los métodos de escalado están funcionando correctamente.

## **Pasos para Probar Cada Tipo de Autoescalado**

### **1. Configuración Inicial**

1. Asegúrate de que Minikube está corriendo:

   ```sh
   minikube start --driver=docker
   ```

2. Habilita el complemento de métricas para el escalado horizontal:

   ```sh
   minikube addons enable metrics-server
   ```

3. Aplica los manifiestos básicos para el Deployment y el Service:

   ```sh
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```

### **2. Prueba de Vertical Pod Autoscaler (VPA)**

1. Aplica el manifiesto de VPA:

   ```sh
   kubectl apply -f vpa.yaml
   ```

2. Verifica que el VPA está configurado:

   ```sh
   kubectl get vpa
   ```

3. Genera carga para observar cómo el VPA ajusta los recursos:

   - Usa un contenedor de prueba para generar carga:

     ```sh
     kubectl run -i --tty load-generator --image=busybox -- /bin/sh
     ```

     Dentro del contenedor, ejecuta:

     ```sh
     while true; do wget -q -O- http://nginx-service; done
     ```

4. Observa los ajustes de recursos:

   ```sh
   kubectl describe vpa nginx-vpa
   ```

5. **Desactiva el VPA** antes de probar otro tipo de escalado:

   ```sh
   kubectl delete -f vpa.yaml
   ```

### **3. Prueba de KEDA ScaledObject**

1. Instala KEDA en tu clúster:

   ```sh
   kubectl apply -f https://github.com/kedacore/keda/releases/download/v2.11.0/keda-2.11.0.yaml
   ```

2. Aplica el manifiesto de KEDA ScaledObject:

   ```sh
   kubectl apply -f scaledobject.yaml
   ```

3. Genera carga para activar el escalado:

   - Usa el mismo contenedor de prueba para generar carga:

     ```sh
     kubectl run -i --tty load-generator --image=busybox -- /bin/sh
     ```

     Dentro del contenedor, ejecuta:

     ```sh
     while true; do wget -q -O- http://nginx-service; done
     ```

4. Observa el escalado de réplicas:

   ```sh
   kubectl get pods -w
   ```

5. **Desactiva el ScaledObject** antes de probar otro tipo de escalado:

   ```sh
   kubectl delete -f scaledobject.yaml
   ```

### **4. Prueba de Horizontal Pod Autoscaler (HPA)**

1. Crea un HPA basado en el uso de CPU:

   ```sh
   kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=1 --max=10
   ```

2. Genera carga para activar el escalado:

   - Usa el mismo contenedor de prueba para generar carga:

     ```sh
     kubectl run -i --tty load-generator --image=busybox -- /bin/sh
     ```

     Dentro del contenedor, ejecuta:

     ```sh
     while true; do wget -q -O- http://nginx-service; done
     ```

3. Observa el escalado de réplicas:

   ```sh
   kubectl get hpa
   kubectl get pods -w
   ```

4. **Desactiva el HPA** antes de probar otro tipo de escalado:

   ```sh
   kubectl delete hpa nginx-deployment
   ```

### **Notas Importantes**

- **No combines múltiples tipos de escalado a la vez**, ya que pueden interferir entre sí.
- Asegúrate de eliminar los recursos de un tipo de escalado antes de probar otro.
- Usa `kubectl delete -f <manifiesto>` para limpiar los recursos después de cada prueba.
