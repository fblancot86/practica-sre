# liberando-productos-francesc
Práctica del Módulo Liberando Productos de Francesc

# Requisitos

Para la correcta ejecución de los comandos necesarios apra la correción de la práactica, hace falta tener isntalado lo siguiente:

```
minikube
helm
kubectl
```

# Aplicación simple-server

Se ha creado el endpoint /bye, añadiendolo a los endpoints ya existentes.

![Endpoints](https://https://github.com/KeepCodingCloudDevops5/liberando-productos-francesc/blob/main/ejemplos/endpoints.png)


# Github Actions

Se han creado 2 Workflows, uno de Testing y otro de Release. En el apartado Actions se pueden ver ejecuciones exitosas de ambos Workflows.

https://github.com/KeepCodingCloudDevops5/liberando-productos-francesc/actions

![Ejemplo Test](https://https://github.com/KeepCodingCloudDevops5/liberando-productos-francesc/blob/main/ejemplos/test_ghactions)

![Ejemplo Release](https://https://github.com/KeepCodingCloudDevops5/liberando-productos-francesc/blob/main/ejemplos/release_ghactions)


# Creación Cluster en Minikube

Para la instalación de la aplicación con la integración con Prometheus, deberemos iniciar un Cluster en Minikube:

```sh
minikube start --kubernetes-version='v1.21.1' --memory=4096 -p practica-francesc
```

# Instalación Prometheus y Aplicación

Instalaremos Prometheus y Grafana con Helm:

```sh
helm -n monitoring upgrade --install prometheus prometheus-community/kube-prometheus-stack -f custom_values_prometheus.yaml --create-namespace --wait --version 34.1.1
```

A continuación activaremos el addon metrics-server en Minikube:

```sh
minikube addons enable metrics-server -p practica-francesc
```

Por último instalaremos, también a través de Helm, la aplicación simple.server:

```sh
helm install simple-server simple-server/.
```

# Dashboard Grafana

Para comprobar que se están generando correctamente las métricas, hay que hacer port-forward de los siguientes puertos:

```sh
kubectl -n monitoring port-forward svc/prometheus-grafana 3000:80
kubectl port-forward svc/simple-server 8081:8081
```

Entrando a http://localhost:8081/docs, se pueden forzar ejecuciones a los 3 endpoints, con la opción "Try it out".

Entrando a http://localhost:3000, con usuario admin y password prom-operator se podrá acceder al Grafana instanciado en el cluster.

Importando el dashboard personalizado incluido en el repositorio, `grafana-dashborad.json` se pueden ver el número de llamadas a los 3 endpoints.

# Alertas en Slack

Para comprobar este apartado se ha usado la siguiente estrategia:

Exportamos el nombre del Pod:

```sh
export POD_NAME=$(kubectl get pods -l "app.kubernetes.io/name=simple-server,app.kubernetes.io/instance=simple-server" -o jsonpath="{.items[0].metadata.name}")
```

Nos conectamos al contenedor del pod:

```sh
kubectl exec --stdin --tty $POD_NAME -- /bin/sh
```

Una vea dentro, instalamos Git y Go:

```sh
apk update && apk add git go
```

Entramos en la carpeta NodeWrecker:

```sh
cd NodeWrecker
```

Compilamos la aplicación Go:

```sh
go build -o extress main.go
```

Por último ejecutamos la aplicación para que se dispares las alertas:

```sh
./extress -abuse-memory -escalate -max-duration 10000000
```

![Alertas Slack](https://https://github.com/KeepCodingCloudDevops5/liberando-productos-francesc/blob/main/ejemplos/alertas_slack)
