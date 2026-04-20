---
tags:
  - aws
  - practitioner
  - lab
date: 2026-04-18
---

# 03 - Configuración de Balanceador de carga

## Objetivo
Describe aquí el propósito del laboratorio de forma breve y clara (agregar descripción).

## Componentes configurados
- Balanceador de carga
- SG del balanceador
- Target Group

## Arquitectura
![Arquitectura](../Diagrams/01-nombre-del-diagrama.png)

## Configuracion de Balanceador de carga

### 1. Configuracion del esquema, AZs y SG
(Explicación breve de por que se escogió un aplication load balancer). 

1. Se procede a configurar el aplicación load Blancer en este caso llamado `training-lab-app-load-bal` para lo cual se definen las opciones:
- internet facing:
- Ipv4 only
![Captura 1](../Screenshots/03-app-load-bal-1.png)
Posteriormente se selecciona la VPC y en subredes seleccionamos las redes públicas alojadas en las diferentes AZ que tenemos disponibles, en este caso son `pública1` y `pública2`.  Posteriormente creamos un security groups para el balanceador que acepte el trafico en este caso HTTP abierto a internet (0.0.0.0/0) y procedemos a seleccionarlo en la configuración del balanceador.
![Captura 1](../Screenshots/03-app-load-bal-2.png)

### 2. Configuración del listener y target group
Aquí se define el protocolo y el puerto que recibirá el balanceador (mejorar explicación breve de que es listener) y se define el target groups (explicación breve de que es el target group).
En el target group se se definen los siguientes parámetros:
- Healthy threshold: 3 (chequeos consecutivos para verificar que el server esta en buena salud).
- Unhealthy threshold: 2 (chequeos consecutivos para definir que el servidor esta caído)
- Timeout: 10s (tiempo de espera antes de considerarse cheque fallido)
- Intervalos: 20s (definir brevemente)
![Captura 1](../Screenshots/03-target-group.png)
finalmente se seleccionan las instancias privadas que serán el destino del target (mejorar esta explicación),  las cuales son `privada1` y `privada2` y se finaliza la creación del target group.

### 3. Permitir al ALB chequear salud de servidores privados
Para que el balanceador pueda verificar el estado de salud de los servidores privados y poderles enviar tráfico se necesita agregar una regla de entrada (inbound rule) al security group de la subred privada que los administra. Se procede a agregar regla que permita trafico http desde el security groups del balanceador `sg_app_load-bal`.

![Captura 3](../Screenshots/03-sg-privadas-rules.png)
Ahora podemos volver al target groups y ver que los 2 servidores se han verificado en buen estado para recibir solicitudes. 
![Captura 3](../Screenshots/03-target-group-3.png)
### 4. Pruebas de carga con el ALB
Procedemos a ir al balanceaodr de carga y buscamos el `DNS name` generado por el balanceador, ese enlace lo pegamos al navegador y verificamos que el balanceador este utilizando ambos servidores configurados `serverA` y `serverB.

![Captura 3](../Screenshots/03-verification-serverA.png)

![Captura 3](../Screenshots/03-verification-serverB.png)
## Datos técnicos relevantes

| Recurso | Nombre | CIDR | Zona |
| --- | --- | --- | --- |
| Recurso 1 | `nombre-1` | `x.x.x.x/x` | `region-1a` |
| Recurso 2 | `nombre-2` | `x.x.x.x/x` | `region-1b` |
| Recurso 3 | `nombre-3` | `-` | `-` |

## Aprendizajes
- Aprendizaje 1.
- Aprendizaje 2.
- Aprendizaje 3.

## Resultado
Describe aquí el resultado final del laboratorio y qué quedó listo.

## Siguiente etapa
Describe cuál sería el siguiente laboratorio o la siguiente mejora.

---
*Joel David Gonzalez - AWS Practitioner Lab - 17/04/2026*
