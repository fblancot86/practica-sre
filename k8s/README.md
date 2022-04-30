# Instalación en clúster Kubernetes

Se requiere disponer de un clúster Kubernetes.

Para esta parte hay varios manifiestos que habrá que aplicar en el cluster con kubectl.

Primero de todo tendremos que movernos a la carpeta k8s del proyecto.

Una vez dentro, podremos ir desplegando los diferentes recursos, para que funcione todo correctamente, hay que seguir el siguiente orden.

## ConfigMap

Revisar el ConfigMap, podemos modificar el usuario (postgres_user) y contraseña (postgres_password) de la base de datos que usará la aplicación, así como el nombre de la base de datos (postgres_database).

Una vez donfigurados esos valores, procederemos a lanzar el siguiente comando:

```mermaid
kubectl apply -f configMap.yaml
```

> NOTA: En este proyecto también tenemos un secret definido, pero no está en uso debido a un problema con Nginx que devolvía un error 503 .

## Base de datos (StatefulSet)

Ahora ya podemos empezar a lanzar los diferentes recursos, necesarios para el funcionamiento de nuestra aplicación:

```mermaid
kubectl apply -f db.yaml
```

Esto levantará un StatefulSet de una sola replica de un contenedor con una base de datos PostgreSQL.

Además lanzará un Servicio headless para el StatefulSet y un Servicio ClusterIP para que sea visible desde el interior del Cluster.

Por último, este manifiesto creará un Volúmen persistente a través de un PersistentVolumeClaim para garantiar la persistencia de datos de la aplicación.

## Aplicación Web (Deployment)

Una vez tengamos levantado el StatefulSet con la base de datos, lanzaremos la aplicación web.

```mermaid
kubectl apply -f web.yaml
```

Esto levantará una aplicación web Django, también con 1 replica (escalable a 3 más adelante).

También levantará un Servicio ClusterIP, que más adelante se usará para hacer accesible desde el exterior con Ingress.

## Autoescalado Horizontal (HPA)

Para darle a nuestra aplicación de mayor consistencia le daremos escalabilidad horizontal a la web, de manera que crearemos un HPA para que escale desde 1 a un máximo de 3, siempre que supere un 75% de la cpu o la memoria de sus pods.

```mermaid
kubectl apply -f hpa.yaml
```

## Ingress

Por último, para hacerlo accesible desde el exterior necesitaremos seguir los siguientes pasos:
-   Instalaremos un controlador Ingress en nuestro cluster, en este caso el controlador de Nginx siguiendo la URL: https://kubernetes.github.io/ingress-nginx/deploy
-   Cuando se tenga el controlador instalado, tendremos que modificar el fichero ingress.yaml para informarle el host, si no tienes uno disponible, puedes usar como referencia el que viene por defecto, cambiandole la IP por la IP externa de tu controlador Ingress: dev.<TU_IP>.nip.io

Cuando ya tengamos el fichero ingress bien configurado, procedemos a levantarlo:

```mermaid
kubecctl apply -f ingress.yaml
```

