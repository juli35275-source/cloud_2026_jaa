## Práctica 203 de Julián ALmarza Álvarez

1. Paso 1: Configuración de los Security Groups encadenados
En la consola de AWS, navegar a EC2 > Network & Security > Security Groups.
Crear el Security Group para el balanceador:
Name: ```sg-alb-publico```
Description: Permitir entrada HTTP publica al balanceador
VPC: VPC por defecto del laboratorio.
Inbound rule: Tipo HTTP, Puerto 80, Source 0.0.0.0/0.
Guardar el grupo. Copiar el identificador generado (ejemplo: sg-0123456789abcdef0).


![](./imagenes/securitygroup.png)


Crear el Security Group para las instancias EC2:
Name: sg-ec2-backend
Description: Permitir trafico HTTP unicamente desde el ALB y SSH para administracion
VPC: VPC por defecto del laboratorio.
Inbound rules:
Regla 1 (HTTP): Tipo HTTP, Puerto 80, Source: escribir sg- y seleccionar el grupo sg-alb-publico creado en el paso anterior.
Regla 2 (SSH): Tipo SSH, Puerto 22, Source My IP (para pruebas de diagnóstico).


![](./imagenes/segundogrupodeseguridad.png)


2. Paso 2: Creación de la plantilla de lanzamiento
En el menú lateral izquierdo de EC2, ir a Instances > Launch Templates y pulsar en Create launch template.
Launch template name: lt-servidor-web-asir.
Application and OS Images: seleccionar Amazon Linux 2023 AMI.
Instance type: t2.micro o t3.micro.
Key pair: seleccionar un par de claves
Network settings:
No asignar subred aquí (el ASG se encargará de distribuirlas).
Firewall (security groups): seleccionar sg-ec2-backend.


![](./imagenes/imagen2.1.png)
![](./imagenes/imagen2.3.png)
![](./imagenes/iamgen2.4.png)


3. Paso 3: Creación del Target Group
Ir a Load Balancing > Target Groups y hacer clic en Create target group.
Basic configuration:
Target type: Instances.
Target group name: tg-web-asir.
Protocol: HTTP, Port: 80.
VPC: VPC por defecto.
Health checks:
Health check protocol: HTTP.
Health check path: /.


![](./imagenes/targetgroup.png)


En Advanced health check settings, ajustar para acelerar las pruebas:
Healthy threshold: 2.
Unhealthy threshold: 2.
Interval: 10 segundos.
Pulsar en Next. En la pantalla de registro de destinos (Register targets), no registrar ninguna instancia manualmente, ya que el Auto Scaling Group se encargará de esta tarea.
Hacer clic en Create target group.


![](./imagenes/healtchecks.png)


4. Paso 4: Creación del Application Load Balancer (ALB)
Ir a Load Balancing > Load Balancers y hacer clic en Create load balancer.
Seleccionar Application Load Balancer (pulsar en Create).
Basic configuration:
Load balancer name: alb-web-asir.
Scheme: Internet-facing (público).
IP address type: IPv4.
Network mapping:
VPC: VPC por defecto.
Mappings: Marcar al menos dos subredes correspondientes a dos Zonas de Disponibilidad distintas (por ejemplo, us-east-1a y us-east-1b).
Security groups:
Eliminar el grupo por defecto.
Seleccionar exclusivamente sg-alb-publico.
Listeners and routing:
Protocol HTTP, Port 80.
Default action: Reenviar a (Forward to) tg-web-asir.
Pulsar en Create load balancer.
El balanceador tardará entre 2 y 3 minutos en aprovisionarse. Localizar en su panel de detalles el campo DNS name (ejemplo: alb-web-asir-123456789.us-east-1.elb.amazonaws.com).


![](./imagenes/balanceador.png)


5. Paso 5: Creación del Auto Scaling Group (ASG)
Ir a Auto Scaling > Auto Scaling Groups y pulsar en Create Auto Scaling group.
Paso 1 (Choose launch template or configuration):
Auto Scaling group name: asg-web-asir.
Launch template: Seleccionar lt-servidor-web-asir (versión Default o Latest). Pulsar en Next.
Paso 2 (Choose instance type and network):
VPC: VPC por defecto.
Availability Zones and subnets: Seleccionar las mismas dos subredes elegidas para el balanceador en el Paso 4. Pulsar en Next.
Paso 3 (Configure advanced options):
Load balancing: Marcar Attach to an existing load balancer.
Seleccionar Choose from your load balancer target groups y elegir tg-web-asir.
Health checks: Marcar la casilla Turn on Elastic Load Balancing health checks.
Health check grace period: 120 segundos (tiempo para que el script User Data instale Apache antes de que el ASG considere la máquina fallida). Pulsar en Next.
Paso 4 (Configure group size and scaling policies):
Desired capacity: 2.
Minimum capacity: 2.
Maximum capacity: 4.
Scaling policies: Seleccionar Target tracking scaling policy.
Policy name: politica-cpu-50.
Metric type: Average CPU utilization.
Target value: 50 (escalará si la media de CPU del grupo supera el 50%).
Warmup period: 60 segundos. Pulsar en Next.
Omitir notificaciones y etiquetas pulsando en Next hasta la pantalla final.
Pulsar en Create Auto Scaling group.