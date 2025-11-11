# comments-infra-openshift

Repositorio de infraestructura que define el despliegue completo del sistema de comentarios en OpenShift mediante un chart de Helm y un workflow de GitHub Actions.

## Contenido

- `charts/comments-system`: Chart de Helm con todos los manifiestos (namespace, Deployments, Services, Route, PVC, ConfigMaps, Secrets, NetworkPolicies).
- `.github/workflows/openshift-deploy.yaml`: pipeline que instala Helm, oc y despliega el chart en OpenShift usando únicamente `helm upgrade --install`.

## Configuración de secretos en GitHub

Definir los siguientes secretos en el repositorio:

- `OPENSHIFT_SERVER`: URL del servidor OpenShift (por ejemplo `https://api.cluster:6443`).
- `OPENSHIFT_TOKEN`: token de acceso con permisos para crear recursos en el namespace objetivo.
- `DOCKERHUB_USERNAME`: usuario que aloja las imágenes `comments-frontend`, `comments-backend-api` y `comments-backend-data`.
- `DOCKERHUB_TOKEN`: token o contraseña del usuario de Docker Hub (se usa para autenticación en el workflow).
- `COMMENTS_NAMESPACE`: namespace de OpenShift donde se desplegará el sistema.

## Ejecución local de Helm

1. Iniciar sesión en OpenShift con `oc login`.
2. Ajustar los valores de imágenes según tu usuario de Docker Hub:

```bash
helm upgrade --install comments-system charts/comments-system \
  --namespace comments-system \
  --create-namespace \
  --set namespace=comments-system \
  --set image.frontend.repository=myuser/comments-frontend \
  --set image.backendApi.repository=myuser/comments-backend-api \
  --set image.backendData.repository=myuser/comments-backend-data
```

El pipeline oficial ejecuta el mismo comando sin usar `oc apply`, garantizando que todo proviene del chart.
