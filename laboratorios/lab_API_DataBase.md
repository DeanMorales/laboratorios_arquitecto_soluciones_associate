# laboratorio para implementar una API con una base de datos de DynamoDB


## caso de uso 

retomando el uso de una aplicacion web de alquiler de vehiculos para visitantes, actualmentes esta impulsada por una base de datos SQL relacional. las pruebas de estres revelan que la base de datos solo puede manejar un numero limitado de usuarios concurrentes. el personal de TI quiere una solucion de base de datos escalable y rentable que pueda manehar altos volumenes de trafico sin requerir una inversion significativa en infraestructura.

## objetivos

- Crear una tabla en DynamoDB para almacenar datos de vehiculos.
- Crear una funcion Lambda para guardar registros de DynamoDB.
- Crear una REST API usando API Gateway

## Arquitectura del laboratorio

## Conceptos claves

- DynamoDB
- API 
- API Gateway
- API REST
- Microservicios
- SQL vs NoSQL

## Teoria 

migra la aplicacion a una arquitectura sin servidor utilizando Amazon DynamoDB para almacenamiento NoSQL escalable, AWS Lambda para computacion y Amazon API Gateway para manejar solicitudes de API.  

- la aplicacion envia una solicitudes HTTP que contiene una carga util en formato JSON al backend para su procesamiento

- Amazon API Gateway actua como el punto de entrada para que las apliaciones accedan a datos, logica de negocios o funcionalidad de servicios de backend, gestionando solicitudes de API a gran escalable Amazon API Gateway actual como el punto de entrada para que alas aplicaciones accedan a datos, logica de negocios o funcionalidad de servicios de backend, gestionando solicitudes de API a gran escala. Diirefencias entre SQL  y noSQL 

- Definicion de DynamoDB: (puedes agregar los beneficios diferentes de DynamoDB)
- noSQL: son base de datos no relacionades, no llevan una estructura predefinida fija, son para casos de uso podria ser aplicacion de smartphone, web app y gaming,

adeas de sus principales beneficio para administrar grandes volumenes de datos, baja latencia, high performance. y flexibilidad en sus modelos de datos.

- por que utilizar un esquema flexible? R= debido a la rapides y agilidad para desarrollar los documentos y definir las entidades, podemos agregar la informacion necesaria para cada uno de ellos
- noSQL: son base de datos no relacionades, no llevan una estructura predefinida fija, son para casos de uso podria ser aplicacion de smartphone, web app y gaming,

ademas de sus principales beneficio para administrar grandes volumenes de datos, baja latencia, high performance. y flexibilidad en sus modelos de datos.

por que utilizar un esquema flexible? R= debido a la rapides y agilidad para desarrollar los documentos y definir las entidades, podemos agregar la informacion necesaria para cada uno de ellos. 

escalabilidad que brindan, el alto rendimiento. y su alta funcionalidad. 

DynamoDB como servicio de base de datos sin servidor, no requiere servidores para aprovisionar, parchear o administrar, ni software para instalar y mantener u operar

## Query Examples


DynamoDB proporciona disponibilidad y tolerancia a fallos integradas en multiples zonas de disponibilidad, eliminando la necesidad de diseñar aplicaciones especificamente para estas capacidades.

**Descripcion de escaneos de Amazon DynamoDB** 

Un escaneo de DynamoDB lee todos los elementos de una tabla o de un indice secundario. 
- primero se realiza un escaneo en todo la tabla, devolviendo asi los elementos escaneados 
- despues seles puede aplicar un filtro, los filtros pueden tener comparaciones logicas para evaluar los elementos previamente escaneados 
- third al final los elementos filtrados no representa mayor poder computacional, tendremos el atributo count final para todos los elementos que fueron mostrados despues del filtro con compador logico.


## Conceptos
en este laboratorio de practica, podra: 
- Crear una tabla de Amazon DynamoDB.
- second definir el esquema de una tabla
- third crear una funcion Lambda para crear, actualizar y consultar los elementos de la tabla. 
- exponer una API RESTful de Amazon API Gateway a la funcion Lambda.

----------
## laboratorio
se utilizara un archivo de ejemplo escrito en python par aconectar nuestra tabla a la API
> se abordara de maenera muy general pues son 48 pasos. para los puntos importantes. 
>> recordando que configuraciones son escensiales. 




