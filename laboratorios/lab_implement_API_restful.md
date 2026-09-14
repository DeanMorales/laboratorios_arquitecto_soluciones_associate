## laboratorio para implementar una API Restful 

### asigancion

el equipo de TI  de la ciudad ha desarrollado una aplicacion movil de alquiler de vehiculos con una interfaz de frontend completa y una base de datos de backend que rastrea la ubicacion, el estado y la disponibilidad de autos y scooters electricos. la aplicacion proporciona una forma conveniente para que los visitantes localicen y alquilen vehiculos.
el equipo necesita una solucion para conectar la aplicacion de frontend a la base de datos de backend, permitiendo el acceso a datos de vehiculos en tiempo real de la aplicacion.

### objetivos de aprendizaje

- crear una API de API Gateway y una funcion Lambda.
- Usa la integracion de proxy Lambdade API Gateway para la llamada a Lambda.

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
- selecciona REST API, 

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

ahora crearemos los Methods o metodos HTTP