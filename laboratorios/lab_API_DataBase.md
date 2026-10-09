# Laboratorio: API REST con DynamoDB

## Caso de Uso

Retomando la aplicación web de alquiler de vehículos para visitantes, actualmente impulsada por una base de datos SQL relacional. Las pruebas de estrés revelan que la base de datos solo puede manejar un número limitado de usuarios concurrentes. El personal de TI busca una solución de base de datos escalable y rentable que pueda manejar altos volúmenes de tráfico sin requerir una inversión significativa en infraestructura.

---

## Objetivos

- Crear una tabla en **DynamoDB** para almacenar datos de vehículos.
- Crear una función **Lambda** para guardar registros en DynamoDB.
- Crear una **REST API** usando **API Gateway**.

---

## Arquitectura del Laboratorio

> *(Diagrama por agregar)*

---

## Conceptos Clave

| Concepto | Descripción breve |
|---|---|
| **DynamoDB** | Base de datos NoSQL totalmente administrada por AWS |
| **API** | Interfaz de programación de aplicaciones |
| **API Gateway** | Servicio para crear, publicar y gestionar APIs |
| **API REST** | Estilo arquitectónico para servicios web stateless |
| **Microservicios** | Arquitectura de componentes independientes y desacoplados |
| **SQL vs NoSQL** | Bases de datos relacionales vs no relacionales |

---

## Teoría

### Arquitectura de la Solución

La solución migra la aplicación a una arquitectura serverless utilizando tres servicios principales:

- **Amazon DynamoDB** — almacenamiento NoSQL escalable
- **AWS Lambda** — cómputo sin servidores
- **Amazon API Gateway** — punto de entrada para solicitudes HTTP

El flujo básico es:

```
Cliente → HTTP Request (JSON) → API Gateway → Lambda → DynamoDB
```

---

### Amazon API Gateway

Amazon API Gateway actúa como el punto de entrada para que las aplicaciones accedan a datos, lógica de negocios o funcionalidad de servicios de backend. Gestiona solicitudes de API a gran escala, incluyendo autenticación, throttling, caché y monitoreo.

---

### SQL vs NoSQL

| Característica | SQL (Relacional) | NoSQL (No relacional) |
|---|---|---|
| Esquema | Fijo y predefinido | Flexible y dinámico |
| Escalabilidad | Vertical | Horizontal |
| Consultas | SQL estándar | API propia por servicio |
| Casos de uso | Transacciones complejas | Apps móviles, web, gaming |

---

### Amazon DynamoDB

**Definición**: DynamoDB es una base de datos NoSQL totalmente administrada por AWS, diseñada para aplicaciones que requieren baja latencia a cualquier escala.

**Principales beneficios**:

- **Sin servidores**: no requiere aprovisionar, parchear ni administrar servidores, ni instalar software.
- **Alto rendimiento**: latencia en milisegundos de un solo dígito.
- **Escalabilidad automática**: maneja grandes volúmenes de datos sin intervención manual.
- **Alta disponibilidad**: disponibilidad y tolerancia a fallos integradas en múltiples zonas de disponibilidad (AZ), sin necesidad de diseñar esa lógica en la aplicación.
- **Flexibilidad de esquema**: permite agregar atributos a los ítems de forma independiente, agilizando el desarrollo.

---

### Escaneos en DynamoDB

Un escaneo (`Scan`) lee todos los elementos de una tabla o de un índice secundario. El proceso ocurre en tres etapas:

1. **Escaneo completo**: se recorre toda la tabla y se devuelven todos los elementos encontrados.
2. **Aplicación de filtros**: se aplica una expresión de filtro con comparaciones lógicas sobre los elementos escaneados.
3. **Resultado final**: el atributo `Count` refleja el número de elementos devueltos después del filtro.

> ⚠️ Los filtros no reducen el costo del escaneo: DynamoDB cobra por los elementos leídos **antes** de aplicar el filtro, no por los elementos devueltos.

---

## Laboratorio

Se utilizará un archivo de ejemplo escrito en **Python** para conectar la tabla DynamoDB a la API.

> El laboratorio consta de **48 pasos**; a continuación se destacan los puntos y configuraciones esenciales.

### Configuraciones esenciales a recordar

- Definir correctamente la **clave de partición** (`Partition Key`) al crear la tabla en DynamoDB.
- Asignar un **rol de ejecución IAM** a la función Lambda con permisos sobre DynamoDB.
- Configurar la **integración Lambda proxy** en API Gateway para pasar el evento HTTP directamente a la función.
- Realizar el **deploy del API** a un stage antes de probarlo.
- Habilitar **CloudWatch Logs** en API Gateway para facilitar el debugging.




## Práctica: Pasos del Laboratorio

En este laboratorio podrás:

1. Crear una tabla de Amazon DynamoDB.
2. Definir el esquema de la tabla (clave de partición y clave de ordenación).
3. Crear una función Lambda para **crear**, **actualizar** y **consultar** elementos de la tabla.
4. Exponer la función Lambda como una API RESTful mediante Amazon API Gateway.

1. nos dirigimos a crear una tabla en el servicio de DynamoDB, en la consola.  y damos clic en "create table".
 - los requisitos de la tabla: 
 - nombre de la tabla: rental_app
 - partition key: record_type
 - sort key: id 
 - table settings: 
    - default settings

  click en create table. 

2. solo queda esperar a que el *status* de la tabla pase a **Active** 
> DynamoDB aplica una funcion hash al valor de la clave de partition  para determinar la particion fisica donde se almacenan los datos. Este mecanismo de distribucion admite el escalado horizonal y el acceso a datos de alto rendimiento. Cuando una tabla incluye tanto claves de partition como de ordenacion, varios elementos pueden compartir la mism aclave de particion, pero deben tener valores de clave de ordenacion unicos.

3. nos dirigimos a la lambda, seguramente aqui pondremos la funcion que acontinuacion se muestra, es un ejemplo de un handler.py para manejar las respuestas de la api, para que actualice la tabla de base de datos en DynamoDB

- Creamos nueva funcion:
  - Author from scratch. empiezas por un simple hellow world
  - en informacion basica: 
    - Function name: labFuntion 
    - runtime: Python 3.14 
    - Architecture: x86_64
    - clic en  **Change default execution role** 
      - seleciona **Use an existing role** 
      - lab_function_role_xxxxx

- clic en crear funcion

4. dentro del entorno ya listo para trabajar, pegaremos en la seccion de *code* nuestra funcion lambda. es un entorno muy similar a visual studio 

como funciona: 
- lambda almacena nuestra funcion en un S3 en cifrado en reposo
- proporciona un almaenamiento de codigo seguro y duradero. al implementar codigo a traves de la consola, lambda maneja automaticamente la carga en Amazon S3 y mantiene el historial de versiones para las capacidades de reversion. 

siempre necesitamos actualizar y desplegar nuestra funciuon. dando click en **Deploy** 

>  como vemos nuestra funcion obtiene el nombre de la tabla de DynamoDB mediante una variable de entorno de Lambda, por lo que acontinuacion debemos definirla en la seccion de **Configuracion**.

5. definir variable de entorno. en la configuracion en *environment variables*  editar la configuracion
  - agregar una nueva variable:
  - key: TABLE_NAME 
  - value: rental_app

  damos click en *Save* 

de esta menera checamos el codigo y analizamos como maneja estas llamadas a la base de datos.

6. Testing: 
  una parte fundamental es testear nuestra api. asi que debemos de mandar algunos test para ver que este funcionando de manera correcta. en la seccion de **code** en el boton **Test** . podremos crear todo un evento de prueba  dando click en **Create new test event**  

- configuramos nuestro evento de testing. 
    - event name: create_location
    - Event sharing settings: prinvate 
    - template_ hello world
    - Event JSON: El del codigo de la linea 178-182

  damos click en *save* y clicl en test*. bajamos a la consola de **output**  y vemos los resultados que arroja la consola. 
  como resultados del testing se debio haber creado un elemento en nuestra tabla DynamoDB

 ingresamos a DynamoDB 
 y exploramos nuestros elementos de la tabla. deberia aparecer ahi mismo. 
 para este laboratorio debes copiar  el id de la locacion, 

### creacion de la API Gateway

 una parte fundamental es el servicio de API Gateway, pues ofrece 3 tipos de API la de tipo HTTP, API REST y API Websocket. admiten patrones RESTful y con metodos HTTP estandar (GET, POST, PUT, PATCH, DELETE) para la comunicacion cliente-servidor sin estado.

- selecicona la tarjeta de  REST API.
  - New API
  - APIname : rental_app
  - description -optional 
  - Security policy: TLS_1_0
  - click en create API.

  
#### creacion de recursos 
los recursos representan segmentos de la ruta URL en la estructura de una API y sirven como contenedores para los metodos HTTP. los recurrsos se pueden anidar para crear estructuras gerarquicas como /locations/{id}/vehicles.

> investigar los CORS Cross-Origin Resource Sharing es un mecanismo de seguridad del navegador que controla si las aplicaciones web de un dominio pueden acceder a recursos de un dominio diferente. las API a las que acceden las aplicaciones basadas en navegador tipicamente requieren la configuracion de CORS para especificar que origenes pueden hacer solicitudes.

creamos el recursos:
- resource path: / 
- resource name: locations 
-click en crear recursos. 

dentro de el  recurso location definiremos los meotodos:
- meotod GET: 
- tipo de integracion con Lambda. 
- turn on: lambda proxy integration.

- seleccionamos la funcion lambda que creamos en un inicio. 
- integration timeout.

la integracion de timeout tiene una duracion maxima de 29000 ms , 29 sec pues los sistemas que necesitan mayor tiempo deben migrar a una arquitectura o patrones asincronicos con mecanismos de devolucion de llamada.

como resultado debemos ver el methodo creado GET justo debajo de /locations

como en lambda tambien tenemos la forma de testear nuestra API por medio de la pesatañ **test**. podemos ingresar un **headers**  y click en test. 
analizamos el test. 

un status 200 indica que son exitosas. los codigos 400 indican problemas de solicitud no validas , y los errores 500 son para errores internos del b backend como no disponibilidad del serivico. 
---rental_app

sigamos creando el metodo POST con los mismos pasos. incluye test con el siguiente request body: 

```Python  
{
  "name": "viper roller coaster",
  "vehicles_available": 5
}
```

testeamos y nos debe dar un status de 201 como resultado exitoso. 
volvemos a nuestro dynamos y hacemos un scaneo para ver los elementos en la tabla. y debe estar ingresado ese elemto nuevo. 


- por ultimo deployamos nuestra API  por medio de la implemetnacion de API. definimos el Stage y su nombre de **Test** 


---
## DIY 



---

## Recursos Adicionales

- [Documentación de Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [Documentación de AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Documentación de Amazon API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [Integración Lambda con API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-lambda-function-handler.html)
- [DynamoDB: operaciones Scan](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Scan.html)
