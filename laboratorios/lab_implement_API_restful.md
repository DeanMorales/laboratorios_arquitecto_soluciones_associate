## laboratorio para implementar una API Restful 

### asigancion

el equipo de TI  de la ciudad ha desarrollado una aplicacion movil de alquiler de vehiculos con una interfaz de frontend completa y una base de datos de backend que rastrea la ubicacion, el estado y la disponibilidad de autos y scooters electricos. la aplicacion proporciona una forma conveniente para que los visitantes localicen y alquilen vehiculos.
el equipo necesita una solucion para conectar la aplicacion de frontend a la base de datos de backend, permitiendo el acceso a datos de vehiculos en tiempo real de la aplicacion.

### objetivos de aprendizaje

- crear una API de API Gateway y una funcion Lambda.
- Usa la integracion de proxy Lambdade API Gateway para la llamada a Lambda.


## Diagrama del laboratorio

![ lab_api_Restful ](  /imagenes_laboratorios/lab_api_Restful.png  "lab_api_Restful titulo")


## laboratorio
---
### Implementacion de API RESTful

#### solicitud de solucion.

crear una API REST usando Amazom API gateway que invoque funciones de AWS Lambda para procesar solicitudes y conectar el frontend de la aplicacion movil a la base de datos del backend.

esta solucion utiliza Amazon API Gateway como el punto de entrada para que la aplicacion movil acceda a los datos del vehiculo, la logica de negocio y la funcionalidad de los servicios de backend 

la aplicacion movil realiza una solicitud http al punto final de API Gateway para recuperar los datos de ubicacion, estado y disponibilidad del vehiculo. 

API Gateway recibe la solicitud¿ y maneja todas las tareas involucadras en aceptar, procesar y enruta la llamada de API al servicio backend apropiado. 

API Gateway invoca una funcion de AWS Lambda, que procesa la solicitud consultando la base de datos del backend y devolviendo los datos del vehiculo. Api Gateway recibe la respuesta de Lambda y la devuelve a la aplicacion movil. 

Con llamadas sincronas desde API Gateway a Lambda, la solucion crea una aplicacion completamente sin servidor que se escala automaticamente sin administrar servidores o infraestructura. 

Usando API Gateway, se puede crear multiples puntos finaels de API REST, cada uno apuntando a diferentes funciones Lambda que manejan operaciones especificas como recuperar ubicaciones de vehiculos, actualizar el estado de alquiler o procesar reservas.

##### microservicios. 

## Concepto 

En este laboratorio practico, podrá:
- Cree una API REST de Amazon API gateway.
- Implementar la API REST.
- Invoque una funcion de AWS Lambda mediante la integracion de proxy de API Gateway. 

## Pasos

### Implementar una funcion Lambda 

una funcion de lambda consta de codigo y de cualquier dependencia asociada. Tambien se asocia informacion de confifguracion con cada funcion de Lambda, como el entorno de ejecucion, la asignacion de memorai y el rol de ejecucion. 

para esto haremos lo siguiente: 

- abrir la consola de AWS 
- buscar el servicio de AWS Lambda

dentro de lambda: 
- observar la barra lateral izquierda, donde tenemos diferentes opciones
- en la categoria de **Functions**  damos clck.
- damos click en crear nueva funcion. 

creando nueva funcion: 

1. Para Create function, elige Author from scratch.
2. Para Function name, escribe:

    labFunction

3. Para Runtime, en la lista desplegable, elige Python 3.x.

    - La versión de Python disponible en la AWS Management Console puede ser diferente a la que se muestra en el ejemplo de la captura de pantalla.

4. Vaya al siguiente paso.

---

Custom setting

siguiendo con la configuracion debemos activar "Custom execution role".
el rol de ejecucion de una funcion lambda es un rol de AWS Identity and Access Management (IAM) que otorga a la funcion permiso para acceder a servicios y recursos de AWS
Proporcionas este rol cuando creas una funcion y Lambda asume el rol cuando se invoca tu funcion. Este modelo de seguridad sigue el principio de privilegio minimo al otorgar solo los permisos necesarios para que la funcion realices sus tareas.

- elija activar el custom execution role
- selecione el rol dentro del droplist disponible y
- seleccione Lab_function_role
- al dar click en crear funcion  se mostrara un resumen de sus configuracion 
-click en crear funcion y listo! 

Utilizando el archivo Sample_code.py es un archivo que contiene una funcion lambda para el laboratorio. 

**dentro de la funcion**

tenemos una barra superios con diferentes categorias 

| code | test | monitor |  configuracion | aliases | versions |
| ---  | ---  | --- | --- | --- | --- |

en la seccion *code* borrramos el contenido y aqui pegaremos nuestra funcion de sample_code.py 

El editor de codigo en la consola de lambda proporciona un entorno de desarrollo integrado IDE donde puedes escribir, probar y ver los resultados de ejecucion del codigo de tu funcion lambda, lambda almacena el codigo de la funcion en Amazon simple storage Service S3 y lo encripta en reposo para mayor seguridad. 

```python

import json
import logging

# AWS Lambda Function Logging in Python - https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    '''Demonstrates Amazon API Gateway Lambda proxy integration. You have full
    access to the request and response payload, including headers and
    status code.
    https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html
    '''
    logger.debug(event) # Mind logger.setLevel at line 6. Check Event printed at CloudWatch

    #/pets/{petId}
    pets = [
        { "id": "1", "name": "Peach"},
        { "id": "2", "name": "Chuck"},
        { "id": "3", "name": "Lelow"}
    ]
    
    
    # Input Format https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format
    resource = event['resource']
    # Uncomment to print the event
    # print("Received event: " + json.dumps(event, indent=2))

    err = None
    # /pets List all pets
    response_body = {}
    if (resource == "/pets"):
        response_body = {
            "pets": pets
        }
    # /pets/petId find pet by Id    
    elif (resource == "/pets/{id}"):
        petId = event['pathParameters']['id']
        value = next((item for item in pets if item["id"] == str(petId)), False)
        if( value == False ):
            err = "Pet not found"
        else:
            response_body = {
                "pet": value
            }

        
    response =  response_payload(err, response_body)

    return response
  
  
    
'''
In Lambda proxy integration, API Gateway sends the entire request as input to a backend Lambda function. 
API Gateway then transforms the Lambda function output to a frontend HTTP response.
Output Format: https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format
'''
def response_payload(err, res=None):
    return {
        'statusCode': '400' if err else '200',
        'body': err if err else json.dumps(res),
        'headers': {
            'Content-Type': 'application/json',
        },
    }
# aqui termina el codigo. 
```
---
asegurarnos de respetar nuestra identacion para python, en breve hablaremos de lo que hace cada parte del codigo por lineas de codigo, 

- damos click en Deploy para conpilar nuestro codigo. 

-probar funciones lambda con los test, es inyectar  evento en formato JSON,  para validar la funcion y que haga lo que debe hacerse. que nos devuelva la respuesta esperada. esta logica de programacion nos ayuda a validar y sea correctamente acoplada a otros servciios de AWS como API Gateway. 

- en primera instancia debe arrojar una notificacion de Actualizada correctamente. 

se puede crear hasta 10 eventos necesarios para provar nuestra funcion lambda, con diferentes escenarios y condiciones, de manera rapida te ayudan a detectar varios casos de uso sin requerir actidores externos.
 
---

**Creando un test** 

*Test para provar el API Gateway AWS Proxy*

las plantailla de eventos de muestra proporciona estructuras JSON preconfiguradas para las integraciones de servicios de AWS comunes, como las solicitudes proxy de API Gateways. puedes personalizar estas plantillas para simular llamadas a recursos especificos y probar como tu funcion procesa diferentes patrones de solicitud. 

1. Para la acción del evento de prueba, elige Crear nuevo evento.
2. Para el nombre del evento, escribe:

FindAllPets

3. Para la configuración de uso compartido del evento, elige o mantén el valor en Privado.
4. Para la plantilla, elige apigateway-aws-proxy.
5. Para el JSON del evento, en la línea 3, escribe:

"resource": "/pets",

- Asegúrate de escribirlo exactamente como se muestra.

6. En la parte superior de la sección de evento de prueba, haz clic en Guardar.
7. Haz clic en Probar.
8. Vaya al siguiente paso.

---
Cuando invocas una prueba. Lambda ejecuta la funcion y procesa el evento de muestra que proporciono, el controlador de la funcion recibe el evento, ejecuta la logica del codigo y devuelve una respuesta, incluido  el codigo de estado, los encabezados y el cuerpo, para que pueda verificar que la funcion se comporta correctamente antes de la implementacion. 

- al finalizar el test, debe aparecer un resumen y una respuesta del servidor. 

*Creando un test para probar FindALLPets*

- de igual manera crea un nuevo Test event

1. Para Test event action, elige Create new event.
2. Para Event name, escribe:

FindPetById

3. Para Event sharing settings, elige o mantén Private.
4. Para Template, elige FindAllPets.
5. Vaya al siguiente paso.

1. Para Event JSON, en la línea 3, escriba:

"resource": "/pets/{id}",

2. En la línea 16, escriba:

"id": 1

- Asegúrese de escribir ambos datos exactamente como se muestra.

3. Ve al siguiente paso.

- espera a que arroge "Executing funtion: succeeded"
y lee el log
---

### Implementacion de Api Gateway

API Gateway cierra la brecha entre los clientes HTTP y las funciones de Lambda al traducir solicitudes web en eventos de invocacion de Lambda. esta ingrecion transforma tu funcion sin servidor en una API web de acceso publico sin que tengas que administrar servidores web o balanceadores de carga. 

- navega en la consola de AWS 
- busca API Gateway.
- click en crear una nueva API 

se nos muestran diferentes diferentes opciones para elegir de api, tenemos WebSocket API, Rest API y REST API Private.
- **selecciona REST API** 

como servicio complemtamente gestionado, maneja la complejidad operativa de ejecutar API de produccion a escala, la gestion de trafico distribuye las solicitudes entrantes de manera eficiente. los controles de autorizacion aseguran el acceso a sus recursos y el monitoreo proporciona visibilida en el rendimiento de la API, lo que le permite centrarse en la logica del negocio en lugar de infraestructura.

- en la tarjeta REST API hacer clic en Build.
- No uses la tarjeta REST API.

#### Creando la rest API

1. Para API details, elija New API.
2. Para API name, escriba:

ApiLab

3. Para Description, escriba:

API to support lab

4. Para API endpoint type, elija o mantenga Regional.
5. Desplácese hacia abajo hasta la parte inferior de la página y luego haga clic en Create API (no se muestra).
6. Vaya al siguiente paso.

7. En la alerta de éxito, revise el mensaje.
8. Para cerrar la alerta, haga clic en la X.
9. En el panel Resources, haga clic en **Create resource**.
10. Vaya al siguiente paso.

los recursos representan la estructura jerarquica de su AP, correspondiento a las rutas de URL que los clientes utilizan para acceder a diferentes funcionalidades. el recursos /pets que esta creando establece la base para organizar operaciones relacionadas, como listar todas las mascotas o recuperar detalles de mascotas individuales. 

#### creando recursos con verbos HTTP

los metodos definen los verbos HTTP(GET, POST, PUT DELETE) que las aplicaciones cliente pueden usar para interactuar con cada recurso. el metodo GET que esta agregando a /pets manejara las solicitudes de lectura, siguiendo las convenciones de REST donde las operaciones GET recuperan datos sin modificar el estado del servidor. 

1. En Nombre del recurso, escriba:

Mascotas

2. Haga clic en Create Resource (Crear recurso)
3. Ve al siguiente paso.

---

**ahora crearemos los Methods o metodos HTTP**

debe surgir una notificacion de exito, yt revisa el mensaje, puedes cerrar la alerta. 

en la seccion de Metodos, haz clic en **Crear Metodo**

El ciclo de solicitud-respuesta en las API REST sigue un patron predecible: los clientes envias solicitudes HTTP a rutas de recursos especificias, su API procesa esas solicitudes y devuelve respuestas estructuradas como JSON, con codigos de estado apropiados. Este enfoque estandarizado hace que las API sean intuitivas para que los desarrolladores las consuman e integren en las aplicaciones. 

---

1. En Method type (Tipo de método), elija GET.
2. En Integration type (Tipo de integración), seleccione Lambda function (Función Lambda)
3. Active la integración de proxy de Lambda.
4. Ve al siguiente paso.

la integracion de proxy de lambda optimiza la conexion entre API Gateway y tu funcion lambda. cuando activas la integracion de proxy, API Gateway pasa la solicitudHTTP completa, incluidos los encabeazados, los parametros de consulta, las variables de ruta y el cuerpo, directamente a tu funcion como un objeto de evento estructurado, y tu funcion devuelve una respuesta formateada que API Gateway traduce de nuevo al clinete. 

1. Para la función Lambda, revise para confirmar que la región de AWS us-east-1 esté seleccionada.
2. En el siguiente cuadro de búsqueda, escriba:

lab

y elija el ARN labFunction.

- Creó esta función Lambda en un paso anterior.

3. Haga clic en Crear método.
4. Vaya al siguiente paso.
 
 el nombre de recurso de Amazon **ARN** de funcion Lambda identifica de forma exclusiva su funcion dentro de AWS. API Gateway usa este ARN para establecer el objetivo de integracion, configurando los permisos y el enrutmaient oencesarios para invocar su funcion especifica cuando llegan solicitudes al recursos /pets. 

 Probar la inegracion entre API Gateway y Lambda valida que el fujo de solicitu-respuesta funciona correctamente antes de implementar la API para usuarios externos, las pruebas internas detectan errores de configuracion, problemas de permisos y problemas de formato de respuesta en un entrono controlado. 

 1. Debajo de eso, elija la pestaña Test.
2. Haga clic en Test.
3. Vaya al siguiente paso.

La simulacion de prueba imita como las aplicaciones cliente interactuaran con su API implementeada,. API Gateway construye una solicitud http simulada, invoca su funcion Lambda con la ruta de recursos /pets y muestra el ciclo de respuesta completo, lo que demuestra la integracion de extremo a extremo que hemos configurado.

1. Revisa los resultados.

- En particular, observa los resultados en Status y Response body. El cuerpo de la respuesta podría tener un diseño diferente al del ejemplo de la captura de pantalla.

2. Vaya al siguiente paso.

La respuesta de Lambda incluye tres componentes  criticos: el codigo de estado (que indica exito o falla), los encabezados(que proporcionan metadatos sobre la respuesta), y el cuerpo (que contiene los datos reales.), revisar estos elementos confirma que su funcion devuelve respuestas correctamente formateadas que las aplicaciones cliente pueden analizar y mostrar. 

los recursos forman una estructura de arbol jerarquica que refleja la organizacion logica de su API. El recursos /pets que creó anteriormente representa una coleccion y ahora esta agregando un recursos secundario para representar elementos individuales dentro de esa coleccion, un patron comun en el diseño de API RESTful.

- crearemos otros recursos dentro de pets. para citar /pets/{id}

los parametros de reuta como {id}, crean rutas de recursos dinamicas que captan valores de variables de la URL. Cuando un cliente solicita /pets/3, API Gateway extrae el valro 3 y lo pasa a su funcion Lambda. que luego puede recuperar la mascota especifica con ese identificador. este parton admite API flexibles y basadas en datos sin requerir recursos separados para cada valor de ID posible. 
- en nombre de recursos escriba {id}

- vamos a lo siguiente. 

Las solicitudes de metodo y las respuestas de metodo definen, el contrato completo apra cada operacion de API. La solicitud especifica los datos que la aplicacion cliente debe proporcionar (parametros de ruta, encabezados, cadenas de consulta o cuerpo), mientras que la respuesta define lo que el cliente debe esperar a cambio. este contrato ayuda a los desarrolladores de clientes a comprender como interactuar correctamente con su API. 

en la **API: ApiLab** en Resources y en el metodo **GET** y {id} seleccionamos y seleccionamos **Create method**. 

configurar la solicitud del metodo establece las reglas de validacion y requisitos de datos para las solciones entrantes. los parametros de cadena de consulta filtran los resultados, los encabezados proporcionan credenciales de autenticacion o preferencia de contenido, el cuerpo de la solicitud transporta datos para las operaciones que crean o modifican recursos. Definir estos elementos por adelantado ayuda a Api Gateway a validar las solicitudes antes de invocar tu funcion Lambda. 

- para metood type elegimos GET.
- integracion con Lambda funtion
- prendemos *Lambda proxy integratio*
- seleccionamos nuestra lambda 
- clic en create method

para finalizar podemos testear nuestro metodo en la seccion **Test** dentro del metodo {id}

podemos ingresar en el path , el id , query strings y Headers.

- en este caso solo agregamos un id 1 para verificar que funcione nuestro get por id de nuestro recurso pets. 

el tiempo de ejecucione de Lambda convierte el objeto de respuesta de la funcion en formato JSON,  y lo devuelve a API Gateway. API Gateway luego construye una respuesta HTTP correctamente formateada. que incluye el codigo de estado, los encabezados y el cuerpo, y la entrega a la aplicacion clientes solicitante, completando el cciclo de solicitud- respuesta.

---

### Despiegue

- en nuestro panel clic en deploy API 

el despliegue de tu API la hace accesible fuera de la consola de API Gateway al generar una URL de invocacion publica. Hasta el despliegue, tu API existe solo como una configuracion dentro de API Gateway; el despliegue transforma esa configuracion en un punto final activo (endpoint) y seleccionable al que las aplicaciones cliente pueden acceder atraves de internet. 

al desplegar podemos personalizar nuestra etapa, una etapa representa una instantanea con nombre de su API en un entorno especifico. Las etapas admiten multiples entornos de implementacion como desarrollo, pruebas y produccion. manteninedo configuraciones separadas. por lo que puede probar cambios en una etapa antes de promocionarlos a otra.

**invoke link** 
cada etapa genera una URL de invocacion unica que enruta las solicitudes a la version especifica de API implementada en esa etapa. puedes administrar la limitacion, almacenamiento en cache y registro de fomra independiente.

---

### prueba final

en un navegador o modo incognito puedes pegar este endpoint, y agregar el recurso /pets la final para enviar una solicitud a nuestra API. 

enviar una solicitud GET sin parametros a un recurso, debe recuperar la coleccion completa de elementos, como lo dicta el diseño de una API RESTful. 

- agrega lab/pets/3

agregar un id a la ruta del recursos. recupera un solo elemento especifico de la coleccion. este patron demuesta como las API RESTful usan la estructura de URL para expresar las relaciones de los recursos: la jerarquia de rutoas refleja la jerarquia de datos, lo que hace que las API sean intuitivas y predecibles para los desarrolladores. 

---

## Conclusiones. 

