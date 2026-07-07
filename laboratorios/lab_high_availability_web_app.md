# Laboratorio de aplicacion de alta disponibilidad

#### caso de uso
cuando implementar una aplicacion web en multiples zonas de disponibilidad(AZ) con un Load Balancer de aplicacion y verificaciones de estado para proporcionar alta disponibilidad 

>ademas de estar utilizando servicios como Amazon CloudWatch para verificar el estado de la instancia, CloudFront para entrega de contenido y Targets junto con el Grupo de Auto Scaling

### Diagrama del laboratorio

### pasos seguir

#### paso 1

en este laboratorio ya contamos con varios elementos ya desplegados. como en este caso tenemos un Auto Scaling Group, pero le falta configurar algunas reglas. 

* uno de los puntos importantes a tener en cuenta es a que instancia esta apuntado el grupo de auto scalamiento, si checamos dando check a nuestro grupo y en la etiqueta **Instance Management** podemos ver a donde apunta y cuales son los  hooks y warm pool que podemos configurar. 

* procedemos a crear un nuevo **Target Group** dando clic sobre el apartado **Load Balancing**. 
y clic en *Create target Group*

#### paso 2 

configuracion de un *Target Group* para el lab: 

* **Target type:** Instances
* **Target group name**: TravelAgencyWebServer-TG
* **protocolo:** HTTP 
* **Port:** 80, IPv4
* **VPC:** Lab/TravelAgencyVpc

clic en *CReate target group*
> osea saltate todas la configuraciones restantes jaja hasta incluso seleccionar la instancia dentro de la vpc.

* procedemos a crear un balanceador de carga

#### Paso 3 

creamos el balanceador de carga en *Load Balancing* 

el tipo de *Load balancer* en este laboratorio es el de aplicacion, tambien existe por red y por gateway  y por ulitmo el clasico, ya en una generacion previa. 

>los componentes de ALB son el balanceador, los oyentes osea listeners para determinar el enrutamiento y los target groups para enviar a los objetivos registrados. 

* **LoadBalanceNAme**: TravelAgencyServers-ALB
* **Seleccionar el SCheme por Internet-facing**.
* **VPC** vamos a seleccionar normalmente hasta 3 zonas de disponibilidad en este caso
    * us-east-1a
    * us-east-1b
    * us-east-1c 

ademas,s de siempre seleccionar una **subnet publica**

* seleccionamos el grupo de seguridad correcto 

* agregamos los listeners y configuramos el routing
    en este caso seleccionamos la action **Forward to target groups**
    es el target group que creamos previamente.
* bajamos hastas summary y clic en *Create load balancer* 

#### paso 4 

vamos a conectar este ELB en nuestro ASG osea poner el balanceador frente al grupo de auto escalado. 

en nuestro grupo de auto estacalado, nos vamos a la pestaña de integraciones. y seleccionamos nuestro nuevo ELB

ahora vamos a crear un grupo de segurodad, con el objetivo de filtrar el trafico.
- el trafico permitido al balanceador de carga
- el trafico permitido desde el balanceador de carga a los servidores EC2 que contienen la app web 

#### pas 5
 
creamos un grupo de seguridad con 1 regla de outbound 1 para inbound

inbound:

1. regla tipo **HTTP** con destino 0.0.0.0/0

outbound
1. regla tipo **HTTP** con destino al **SecurityGroup del Web server**

* clic en crear grupo de seguridad.
* aprovechamos y editamos las reglas de otro grupo de seguridad, en este caso el de las instancias EC2 para evitar que entren directamente  desde cualquier direccion ip4 y solo reciban trafico desde el load balancer 

para eso seleccionamos edit en las inbound rules
1. regla tipo **HTTP** con destino a **SG-del load balancer**

* debemos cambiar el grupo de seguridad de nuestro balanceador de carga directo en, editar y selecionamos el nuevo security que acabamos de modificar. 

*abrimos el enlace DNS del ELB y deberia aparecer nuestra app desplegada. 

#### paso 6 

crear las reglas de health check y despliegue en multiples AZ. 

esto se hace en la categoria de *LoadBalancing*, el servicio *Target groups* y dentro seleccionamos nuestro tarjet y dentro de la pestaña *Health checks* clic en **Edit**

1. En Unhealthy threshold, escriba: 2
2. En Timeout, escriba: 2
3. En Interval, escriba: 5
4. Haga clic en Save change

#### paso 7 

crear nuestro despliegue automatico en *Auto Scaling* dentro de *Auto Scaling Groups*, seleccionamos nuestro ASG y editamos **Network** 
y despues **Capacity Overview**

>solo como datos a tener en cuenta, debemos selecionar nuestra VPC y despues cada subnet donde querramos tener desplegada nuestra app.
>>podemos probar nuestro health desde nuestro navegador y ademas agregar cuantos servidores necesitamos para solventar la carga
>>> la capacidad minima, deseada y maxima son diferentes por proyecto.
