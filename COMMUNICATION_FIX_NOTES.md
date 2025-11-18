# Correcciones aplicadas para restaurar la comunicación entre servicios

Este documento resume los cambios realizados en los distintos repositorios para resolver el error de timeout del frontend al llamar `http://backend-api:8080/api/comments`.

## 1. Servicio frontend
- **Repositorio:** `comments-frontend`
- **Cambios:**
  - La aplicación Express ahora escucha en el puerto 8080 (`src/index.js`).
  - El Dockerfile expone el puerto 8080 y define `PORT=8080` para evitar permisos restringidos.
  - Se actualizó la documentación (`README.md`) para reflejar el nuevo puerto.
- **Motivo:** OpenShift no permite que contenedores sin privilegios abran el puerto 80, provocando el crash inicial del pod.

## 2. Chart de infraestructura
- **Repositorio:** `comments-infra-openshift`
- **Cambios clave:**
  - `values.yaml` ahora fija `frontend.port: 8080` para alinear Service y Route con el contenedor.
  - El Deployment del frontend define explícitamente `PORT` a partir de los valores del chart, garantizando que la variable de entorno llegue al contenedor aunque la imagen se cachee.
  - Se reforzaron las NetworkPolicies:
    - Se mantuvieron las políticas "deny-all" pero se añadieron reglas de egress específicas (`allow-frontend-egress`, `allow-backend-api-egress`, `allow-backend-data-egress`) para la comunicación frontend → backend-api → backend-data → db.
    - Se añadió permiso de egress hacia los pods de `openshift-dns` dentro de `deny-all-egress` para permitir la resolución de nombres de servicio.
- **Motivo:** Las políticas originales bloqueaban todo el tráfico saliente y la resolución DNS, lo que impedía que los servicios se encontraran aunque los endpoints fueran correctos.

## 3. Validaciones realizadas
- Se ejecutaron pods temporales (`curlimages/curl`) con las mismas etiquetas que los servicios para confirmar conectividad:
  - `frontend` pudo obtener `HTTP 200` al llamar `http://backend-api:8080/api/comments`.
  - `backend-api` confirmó acceso a `backend-data` (mediante la misma técnica).
- Se verificaron los ConfigMaps de frontend y backend-api para asegurar que apuntan a los servicios internos correctos.

## Próximos pasos sugeridos
1. Ejecutar nuevamente el workflow `Deploy to OpenShift` (o `helm upgrade --install`) para aplicar cualquier cambio pendiente del chart.
2. Validar desde el router público: `http://frontend-darkblade-x-dev.apps.rm2.thpm.p1.openshiftapps.com:8080`.
3. Si se realizan cambios futuros en puertos o políticas, repetir las pruebas con pods temporales para descartar bloqueos de red o DNS.

Con estos ajustes, los pods pueden resolver los servicios internos y comunicarse respetando las políticas de seguridad establecidas.
