# 03 - Configuración de Balanceador de Carga

## Objetivo
El objetivo de esta etapa del laboratorio es implementar un **Application Load Balancer (ALB)** para distribuir el tráfico HTTP entrante entre dos instancias EC2 privadas que ejecutan Apache, mejorando la disponibilidad de la aplicación y permitiendo validar un escenario básico de alta disponibilidad en múltiples zonas de disponibilidad. [cite:1834][page:1][page:2]

## Componentes configurados
- **Application Load Balancer (ALB):** balanceador de carga configurado para recibir solicitudes HTTP y distribuirlas hacia los destinos registrados según las reglas definidas en sus listeners. [page:1]
- **Security Group del ALB:** grupo de seguridad asociado al balanceador para permitir tráfico HTTP desde Internet hacia el ALB.
- **Target Group:** grupo lógico donde se registran las instancias EC2 que recibirán el tráfico enviado por el balanceador. [page:1][page:2]
- **Instancias privadas:** `ServerA` y `ServerB`, ubicadas en subredes privadas y configuradas previamente con Apache. [cite:1834]
- **Subredes públicas:** subredes `publica1` y `publica2`, seleccionadas para desplegar el balanceador en diferentes zonas de disponibilidad.
- **Arquitectura final:** clientes en Internet -> ALB público -> Target Group -> instancias privadas con Apache.

## Configuración del Balanceador de Carga

### 1. Configuración del esquema, AZs y Security Group
Para este laboratorio se seleccionó un **Application Load Balancer** porque el servicio que se desea publicar corresponde a una aplicación web basada en **HTTP**, y este tipo de balanceador trabaja a nivel de solicitud, permitiendo recibir tráfico HTTP/HTTPS y enrutarlo hacia los destinos configurados. [page:1]

El balanceador se creó con el nombre `training-lab-app-load-bal` y con las siguientes opciones iniciales:

- **Scheme:** `internet-facing`
- **IP address type:** `IPv4`

Después de esto, se seleccionó la VPC del laboratorio y se asociaron las subredes públicas disponibles en distintas zonas de disponibilidad, en este caso `publica1` y `publica2`. Esta configuración permite que el ALB quede accesible desde Internet y, al mismo tiempo, distribuya tráfico hacia recursos desplegados en más de una AZ.

Posteriormente, se creó y asoció un **Security Group** para el balanceador que permite tráfico de entrada **HTTP (puerto 80)** desde `0.0.0.0/0`, con el fin de aceptar solicitudes web provenientes de cualquier origen.

### 2. Configuración del listener y del Target Group
Un **listener** es el proceso del balanceador que espera solicitudes de conexión en un **protocolo** y **puerto** específicos. Si un load balancer no tiene al menos un listener configurado, no puede recibir tráfico de los clientes. Además, las reglas definidas en el listener determinan cómo se enrutan las solicitudes hacia los destinos registrados, por ejemplo, instancias EC2. [page:1]

En este laboratorio se configuró un listener para **HTTP en el puerto 80**, ya que el contenido publicado por las instancias se sirve mediante Apache.

A continuación, se creó un **Target Group**, que es el componente donde se registran los recursos de destino que recibirán el tráfico proveniente del ALB. En este caso, los destinos registrados fueron las instancias privadas del laboratorio. [page:1][page:2]

Dentro del Target Group se definieron los siguientes parámetros de health check:

- **Healthy threshold: 3**
  Número de verificaciones exitosas consecutivas necesarias para que un destino sea considerado saludable. [page:2]

- **Unhealthy threshold: 2**
  Número de verificaciones fallidas consecutivas necesarias para que un destino sea marcado como no saludable. [page:2]

- **Timeout: 10 segundos**
  Tiempo máximo que el balanceador espera una respuesta del servidor antes de considerar fallido un health check. [page:2]

- **Interval: 20 segundos**
  Tiempo aproximado entre una verificación de estado y la siguiente para cada destino registrado. [page:2]

Finalmente, se registraron como targets las dos instancias EC2 privadas del laboratorio, correspondientes a `ServerA` y `ServerB`, para que el balanceador pudiera enviar tráfico a ambas una vez superadas las verificaciones iniciales de salud. AWS indica que, tras registrar un target, este debe aprobar sus health checks iniciales antes de empezar a recibir solicitudes. [cite:1834][page:2]

### 3. Permitir al ALB verificar la salud de los servidores privados
Para que el Application Load Balancer pudiera comprobar el estado de salud de las instancias privadas y, posteriormente, enrutar tráfico hacia ellas, fue necesario ajustar las reglas del **Security Group** asociado a los servidores privados.

Se agregó una regla de entrada (**inbound rule**) que permite tráfico **HTTP** desde el Security Group del balanceador, `sg_app_load-bal`. Con esto, el ALB pudo acceder al puerto 80 de las instancias privadas para ejecutar los health checks y confirmar que Apache estaba respondiendo correctamente.

Después de aplicar esta regla, en la consola de AWS fue posible observar que ambos targets cambiaron a estado **healthy**, lo que indica que el balanceador ya podía utilizarlos para distribuir solicitudes. [page:2]

### 4. Pruebas de acceso mediante el ALB
Con el balanceador ya configurado, se utilizó el **DNS name** generado automáticamente por AWS para realizar las pruebas de acceso desde el navegador.

Al abrir esta dirección, se verificó que el tráfico HTTP era distribuido entre `ServerA` y `ServerB`, ambas instancias privadas registradas en el Target Group. Esta prueba permitió confirmar el funcionamiento del ALB, la correcta configuración del Target Group y la conectividad hacia los servidores web internos. [cite:1834][page:1][page:2]

## Datos técnicos relevantes

| Recurso | Nombre | CIDR / IP | Zona |
|---|---|---|---|
| Application Load Balancer | `training-lab-app-load-bal` | IPv4 público asignado por AWS | `publica1` / `publica2` |
| Instancia privada 1 | `ServerA` | `10.0.1.79` | AZ privada 1 [cite:1834] |
| Instancia privada 2 | `ServerB` | `10.0.2.53` | AZ privada 2 [cite:1834] |
| Listener | HTTP : 80 | - | - [page:1] |
| Health checks | Interval 20s / Timeout 10s / Healthy 3 / Unhealthy 2 | - | - [page:2] |

## Aprendizajes
- Un **Application Load Balancer** es apropiado cuando el tráfico que se desea distribuir corresponde a una aplicación web basada en HTTP o HTTPS. [page:1]
- Un **listener** define el protocolo y el puerto por el cual el balanceador recibe solicitudes, y sus reglas determinan cómo se enrutan hacia los targets. [page:1]
- Un **Target Group** agrupa los recursos de destino del balanceador y utiliza health checks para determinar cuáles instancias pueden recibir tráfico de forma segura. [page:2]
- Los **Security Groups** cumplen un papel fundamental en la comunicación entre el ALB y las instancias privadas, ya que sin las reglas adecuadas los health checks fallan y los targets no pueden ser usados.
- Distribuir la aplicación entre múltiples instancias y múltiples zonas de disponibilidad mejora la tolerancia a fallos y representa una arquitectura más cercana a escenarios reales en AWS.

## Resultado
Como resultado de esta práctica, quedó implementado un **Application Load Balancer público** asociado a dos subredes públicas y conectado a un **Target Group** con dos instancias privadas que ejecutan Apache. El laboratorio permitió validar la creación del balanceador, la configuración de listeners, el registro de targets, las verificaciones de salud y la distribución básica de tráfico web entre `ServerA` y `ServerB`. [cite:1834][page:1][page:2]

## Siguiente etapa
Como siguiente mejora, este laboratorio puede evolucionar hacia una arquitectura más robusta incorporando **HTTPS con certificados**, redirección automática de HTTP a HTTPS, integración con **Auto Scaling**, y monitoreo con métricas y logs para observar el comportamiento del balanceador y de los targets. AWS indica que los listeners de ALB soportan HTTP y HTTPS, y que un listener HTTPS permite descargar al balanceador el trabajo de cifrado y descifrado. [page:1]

---
**Joel David Gonzalez - AWS Practitioner Lab - 17/04/2026**
