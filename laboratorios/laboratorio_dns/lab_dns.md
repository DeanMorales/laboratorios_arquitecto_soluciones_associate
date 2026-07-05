# laboratorio DNS

## caso de uso

nuestro cliente cuenta ya con un servidor interno que sirve como fuente de noticias, tienen reporteros y empleados que la usan para consultar fuente de informacion, atravez de una direccion IP statica 

* agregaremos un nombre de dominio por medio de Route53 
* el servidor solo debe tener acceso privado a lo empreados
* implementaremos una zona privada que no exponga al servidor a internet

### diagrama del laboratorio 

![ laboratorio_dns ](  ".\imagenes_laboratorios\lab_dns.png"  "laboratorio_dns titulo")

## pasos a seguir 

### solicitud de solucion
> crear una zona hospedada privada en Amazon Route 53 con registros A para asignar direcciones IP a nombre de dominio y registro CNAME para alias de dominio, lo que permite el trafico enterno de DNS dentro de la VPC.

algo asi para manejar (www.ejemplo.com) en direcciones IP que las computadoras utilizan

Amazon Route 53 gestiona los nombres de dominio y ademas funciona como servidor DNS privado para el acecso exclusivo para recursos internos de una VPC privada denominadas (Hosted Zones)

* la private hosted zone ccontiene 2 tipos de regristos DNS:
    1. registros "A": mapean nombres de dominio directamente a direcciones IP
    2. registros CNAME: registros de nombres cannonicos, crean alias que apuntan a un nombre de dominio a otro.
* el registro A asina la direccion iup de la instancia EC2 con ip 10.10.0.10 al nombre legible por humanos tipo newspaper.internal.news.org

* el CNAME crea un alias al mapear database.thewhitepaper.internal.news.org al dominio existente

newspaper.internal.news.org, para que los empleados puedan usar multiples nombre de dominio para acceder al mismo recurso para diferentes propositos o contextos.


1. en primer lugar en la consola vamos a dirigirnos a nuestras instancias EC2 buscnado EC2 en nuestra barra de busqueda. 
2. los patrones de arquitecturas seguras señalan a tener una aplicacion central de la empresa en privado, y utilizar un intermediario para el acceso administrativo. 

EC2 Servidor  --> EC2 Host Bastion --> Solicitud de usuario

copiamos la direccion IP del EC2 Servidor 
ip : 10.10.2.10

> un servidor bastion o tambien conocido como servidores de salto. proporcionan acceso seguro a instancias privadas dentro de su VPC. controlan el flujo  de solicitudes a travez del aislamiento de red.

3. accedemos al bastion host por medio de sesion manager(system manager)
    * solo acemos un ping (direccion ip)
comprobamos que tenemos acceso a nuestro servidor privado
    * intentamos hacer ping utilizando un CNAME 
    * ping thewhitepaper.internal.news.org 
    fallara pues aun no esta configurado 

    ```
        sh-5.2$ ping thewhitepaper.internal.news.org
        ping: thewhitepaper.internal.news.org: Name or service not known
        sh-5.2$ ^C
    ``` 

4. nos dirigimos a route53 
* configuremos una nueva Hosted Zone dando clic en "create hosted zone"
* en nombres de dominio:  internal.news.org
* en descripcion_: Nombre de dominio del servidor de noticias interno
* type: **Private hosted zone**
* Vpcs to associate with the hosted zone: seleccionamos la region y la vpc que asociaremos, en este caso ***dns-lab**
en **virginia**
* click en add vpc 
* y crear hosted zone
* **muy importante** agregar tags siempre para todos los proyectos en tu cuenta real para llevbar mayor control de los recursos

> Route 53 automaticamente crea 2 tipos de registro en nuevas zonas alojadas: registros NS(servidor de nombres) que identifican que servidores responden a las consultas DNS y un registro SOA(inicio de autoridad) que contiene informacion de administracion de zona, como intervalos de actualizacion y contactos de las partes responsables.

5. creacion de un nuevo record
* clic en nuevo record 
* para nombre del record 1 : thewhitepaper
* para tipo de record: A- routes traffic to an IPv4
* para value: la direccion ip del servidor privado
* para TTL mantenga el valor en 300
* para Routing policy, mantena **Simple routing**
* clic en crear record
* en automatico debe mostrarse el nuevo record en la lista 

6. comprobamos regresando al servidor bastion y volvemos a hacer ping con la direccion ahora del registro A thewhitepaper.internal.news.org

```
    sh-5.2$ ping thewhitepaper.internal.news.org
ping: thewhitepaper.internal.news.org: Name or service not known
sh-5.2$ ^C
sh-5.2$ ping thewhitepaper.internal.news.org
PING thewhitepaper.internal.news.org (10.10.2.10) 56(84) bytes of data.
64 bytes from ip-10-10-2-10.ec2.internal (10.10.2.10): icmp_seq=1 ttl=127 time=0.295 ms
64 bytes from ip-10-10-2-10.ec2.internal (10.10.2.10): icmp_seq=2 ttl=127 time=0.317 ms
64 bytes from ip-10-10-2-10.ec2.internal (10.10.2.10): icm

``` 


## Do it Yourself
