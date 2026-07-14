# Implementacion de una API Restful

## caso de uso

tenemos una aplicacion de tipo API Restful, y necesitamos desplegarla en los servicios de AWS. El escenario nos ubica en una aplicacion dirigida a los clientes de un parque de diversiones, que les ayuda a rentar y ubicar su flota de scooters para poder moverse dentro de todo el parque. 

La interfaz FrontEnd ya esta lista y han creado una base de datos  que guaras su posicion, el estado y la disponibilidad de cada vehiculo.

necesitan ahora conectar el front con el backend, utilizando los servicios de AWS podemos poner el front dentro de API Gateway y conectarlo con el servicio de backend mediante funciones lambda. **Amazon API Gateway** es un servicio totalmente gestionado que puede utilizar para crear API seguras a cualquier escala. 

* usaremos la integracion de **proxy Lambda** de **API Gateway** para llamar a lambda. 


## Conceptos Clave

### ¿Qué es una API RESTful?
Una API RESTful (Representational State Transfer) es una interfaz de programación de aplicaciones que sigue los principios arquitectónicos REST. Se basa en el protocolo HTTP y utiliza métodos estándar como GET, POST, PUT, DELETE para realizar operaciones sobre recursos identificados por URLs. Las APIs RESTful son stateless, cacheables y utilizan una estructura uniforme que facilita la comunicación entre clientes y servidores. En el contexto de aplicaciones web, permite que el frontend se comunique con el backend mediante solicitudes HTTP para obtener, crear, actualizar o eliminar datos de forma estandarizada.

### ¿Qué es AWS API Gateway?
AWS API Gateway es un servicio de Amazon Web Services que permite crear, publicar, mantener, monitorizar y asegurar APIs a cualquier escala. Actúa como puerta de entrada para aplicaciones web y móviles, gestionando todas las tareas relacionadas con la recepción y procesamiento de miles de llamadas concurrentes a la API. Ofrece características como autenticación, autorización, limitación de tasa (throttling), caching, transformación de datos y registro de métricas. Es un servicio completamente gestionado que elimina la necesidad de aprovisionar y mantener servidores para el manejo de APIs.

### Conexión con Lambda y Base de Datos
API Gateway se integra con AWS Lambda para procesar solicitudes y conectar el frontend con la base de datos del backend. Cuando un usuario realiza una solicitud desde el frontend (como consultar scooters disponibles), API Gateway recibe la solicitud HTTP y la dirige a una función Lambda específica mediante la integración de proxy Lambda. La función Lambda ejecuta la lógica de negocio: puede consultar, insertar o actualizar datos en la base de datos (como DynamoDB o RDS) y devolver una respuesta estructurada. API Gateway luego formatea esta respuesta y la envía de vuelta al frontend. Esta arquitectura serverless permite escalar automáticamente según la demanda y pagar solo por el tiempo de ejecución real.

## arquitectura del laboratorio 


### que haremos ?

Crearemos una API REST utilizando API Gateway que invoque una funcion **Lambda** para que procese solicitudes **HTTP** y conecte el frontend de la aplicacion movil con la base de datos del backend 

### estrategias de implementacion de control de versiones de amazon API Gateway
**API Gateway** maneja las implementacion por **Stages**, una *etapa* es un referencia con nombre a una implementacion, que es realmente una *instantanea*(snapshot) de la API.
puedes utilizar estas *Stages* para realizar una administracion de despliegue de tu API, Como en desarrollo y en produccion, una implementacion especifica en varios entornos  

#### Stage variables Example 
como metodo para automatizar la integracion y despliegue (CI/CD)
