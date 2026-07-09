# laboratorio de fundamentos sin servidor

## Caso de uso

El departamento de TI de la ciudad está construyendo la infraestructura para el nuevo parque de diversiones, incluyendo un sistema para recopilar y organizar las calificaciones y comentarios de los visitantes en tiempo casi real para una rápida respuesta a las quejas. El equipo está en una fase temprana de desarrollo y preocupado por los costos de hardware y mantenimiento asociados con los servidores físicos o la gestión de instancias de Amazon EC2. Quieren una solución que pueda ejecutar código para procesar los comentarios de los clientes sin aprovisionar o mantener servidores, reduciendo tanto los costos como los gastos operativos.

## Diagrama del laboratorio

![ lab_lambda ](  /imagenes_laboratorios/lab_lambda.png  "laboratorio_lambda titulo")


## Teoria 

>basicamente el termino "serverless" no quiere decir que no haya servidor por detras, quiere decir que tu proveedor de servicios en la nube se encarga del mantenimiento, parcheo, escalado automatico. nos ayuda a reducir la operacion necesarita dejando al equipo de desarrollo con mayor tiempo en crear la solucion optima del proyecto 

en este lab crearemos una funcion lambda que se encargue de recibir los comentarios desde el front, y llevarlos a la base de datos que podria estar almacenada en RDS, DynamoDB, etc...

### AWS Lambda

 es un servicio totalmente adminsitrado de amazon, se encarga de procesa toda la adminstracion del servidor por detras, tu solo te encargar de escribir la funcion, puede activarse mediante *Triggers*
* como por ejemplo: cuando dentro de un S3 (almacenamiento) se crea, destruye o modifical algun objeto. podemos invocar una funcion lambda. 
* beneficios
    1. no servers to manage
    2. continuous scaling
    3. subsecond metering

#### plataforma serverless de aws

ademas de *lambda*, tenemos otros servicios como **AWS Fargate** que aprovisiona la infraestructura para orquestar contenedores. 

En la categoriga de *almacenamiento* tenemos a **S3** que es un almancen de objetos, **EFS**(elastic file system).

Para *Data stores* tenemos **DynamoDB** para bases de datos NOSQL, y segun **Amazon Aurora Serverless**.

En *API Proxy* tenemos **Amazon API Gateway**.

*Aplication integration* tenemos **SNS** servicio de subscripcion y mensajeria, **SQS** para servicio de colas, **AWS AppSync GraphQL API** y **Amazon EventBridge**.

En *Analytics* tenemos a la familia **Amazon Kinesis**, y **Amazon Athena**.

En *Orquestation*, tenemos como no a **AWS step Funtions**


#### invocacion

aws **Lambda** permite varias maneras de invocacion, desde usar la consola de AWS, la API de lambda, desde la *CLI de AWS* , la **AWS Toolkit** . otra forma es invocarla desde otros recursos o servicios.

se puede invocar por ejemplo desde un evento en S3, como agregar un nuevo objeto o eliminarlo.

tambien puede leer eventos que esten pasando desde una cola con **SQS** o en un flujo entrante de datos com **kinesis**,

De forma **Asincrona** o **sincrona**, de esta forma hay servicios que invocan a lambda de forma predeterminara sincrona, es decir que esperan una respuesta: 
* Api gateway
* Alexa
* cognito
* Kinesis Data Firehose
* lex
* Stepfuntions
* elastic load balancing
* aplication load balancing 

de forma asincrona(trabaja con colas y no espera respuesta): 
- S3
- Amazon Simple email service
- CloudWatch logs, events
- aws config
- SNS
- cloudformation
- codecommit
- AWS IoT events

de forma pretederminada lambda solo se puede invocar un maximo de **1000** veces por region al mismo tiempo, si necesitas mas invocaciones, necesitas llamar a soporte tecnico 

y podemos disponer de un maximo de concurrencia, por lo que necsitamos reservar y siempre tener disponbiles *capacidad reservada* para poder solventar siempre nuevos pedidos o invocaciones.


## pasos a seguir

### paso 1 
    vamos acrear una funcion lambda con python3 
    nos cercioramos de estar en la region de virginia y nos dirigimos a Lambda
* Clic en nueva funtion
* Creation funtion: Author form scratcj
* le ponemos su nombre: 
* la version mas reciente de python en el runtime: python 3.14* 
* por ultimo en la parte de configuracion adicional, seleccionamos **custom execution role** recordemos que es el rol que lambda tomara para consultar los demas servicios
* clic en crear 

### paso 2 

aqui tienes un ejemplo de codigo python para laboratorio, pegamos esto en la seccion de codigo de lambda 

` sample_code.py`

```
# Runtime: Python 3.8
# Please review the comments for each code block to help you understand the execution of this Lambda function.
# At the time you create a Lambda function, you specify a handler, 
# which is a function in your code, that AWS Lambda can invoke when the service executes your code.
# The Lambda handler is the first function that executes in your code.
# Python programming model - https://docs.aws.amazon.com/lambda/latest/dg/python-programming-model.html
def lambda_handler(event, context):
    
    # context – AWS Lambda uses this parameter to provide runtime information to your handler.
    # event – AWS Lambda uses this parameter to pass in event data to the handler. This parameter is usually of the Python dict type.
    # AWS Lambda function Python handler - https://docs.aws.amazon.com/lambda/latest/dg/python-handler.html
    
    # When you invoke your Lambda function you can determine the content and structure of the event. 
    # When an AWS service invokes your function, the event structure varies by service.
    # 
    # In this lab, the lambda_handler "event" parameter has the following structure of name/value pairs:
    # {
    #     "emoji_type": number,
    #     "message": "string"
    # }
        
    # In a name-value pair you can extract the "value" of the pair using the "name" of the pair.   
    # Here the emoji_type value is extracted from the event Lambda parameter and passed to a variable called emoji_type.
    emoji_type = event["emoji_type"]
    # Here the message value is extracted from the event Lambda parameter and passed to a variable called message.
    message = event["message"]
    
    # The variables are printed here, which means the variable values will be displayed in CloudWatch logs and the Execution Results panel.
    print(emoji_type)
    print(message)
    
    # In Python, you can create a variable with no value using the Built-in constant None. This means the variable "custom_message" currently has no value.
    custom_message = None
    
    # In Python, we use the if-elif-else to create a conditional execution. https://docs.python.org/3/reference/compound_stmts.html#the-if-statement
    # That is, if the value of emoji_type is equal to 0, we execute the statement inside its block
    if emoji_type == 0:
        # Only execute if emoji_type is equal to 0
        # The variable custom_message combines "Message for code 0:" string with the variable message
        custom_message = "Message for code 0: " + message

    elif emoji_type == 1:
        # The variable custom_message combines "Message for code 1:" string with the variable message
        custom_message = "Message for code 1: " + message

    else:
        # The variable custom_message combines "Message for all other codes:" string with the variable message
        custom_message = "Message for all other codes: " + message
        
    # Optionally, the handler can return a value. What happens to the returned value depends on the invocation type you use when invoking the Lambda function
    # In this lab we use synchronous execution, so we need to create a response for the lambda function.
    # We created a "response" variable that has a structure of name-value pairs using the emoji_type and custom_message created earlier.
    response = {
        # In this name-value pair, the literal value "message" will store the value of the original message variable.
        "message": message,
        # In this name-value pair, the literal value "custom_message" will store the value of the custom_message variable.
        "custom_message": custom_message,
    }
    
    # Finally, we return the values from response variable to the caller, which could be a test event or an AWS service performing a synchronous call.
    # The execution of the lambda function is finished.
    return response

```
- puedes pasara a dar clic en Deploy, para actualizar la lambda

### paso 3

vamos a testearlo en la pestaña Test
- es un test de tipy synchronous
- le ponemos su nombre
- es privado
- para template es : mobile-backend-echo (mobile backend)

en evento JSON 

```
    {
   "emoji_type" : 1,
   "message": "Hello world!"
    }
``` 
* clic en **Save**
*una vez creado corectamente clic en **Test**

*puedes revisar el executing function: succeded

### paso 4 
nos dirigimos a la pestaña de **Monitor** justo al lado derecho de Test. y podemos dar clic en **abrir cloudwatch logs**

em este panel podemos guardar los direntes logs, y es capaz de manejar las invocaciones asincronas, para un mejor seguimiento de nuestras llamadas a lambda. 

los *logs* contienen etiquetas de tipo START, END y REPORT generadas por el tiempo de ejecucion, REPORT proporciona metricas de ejecucion como la duracion, la duracion facturada, la memoria asignada y la memoria maxima utilizada , datos criticos para optimizar el rendimiento y el costo de la funcion. 
## DIY

para el doit yourself unicamente modificaremos la lambda para cambiar el JSON que retorna, con etiquetas de sentimientos 

``` 
# Runtime: Python 3.8
# Please review the comments for each code block to help you understand the execution of this Lambda function.
# At the time you create a Lambda function, you specify a handler, 
# which is a function in your code, that AWS Lambda can invoke when the service executes your code.
# The Lambda handler is the first function that executes in your code.
# Python programming model - https://docs.aws.amazon.com/lambda/latest/dg/python-programming-model.html
def lambda_handler(event, context):
    
    # context – AWS Lambda uses this parameter to provide runtime information to your handler.
    # event – AWS Lambda uses this parameter to pass in event data to the handler. This parameter is usually of the Python dict type.
    # AWS Lambda function Python handler - https://docs.aws.amazon.com/lambda/latest/dg/python-handler.html
    
    # When you invoke your Lambda function you can determine the content and structure of the event. 
    # When an AWS service invokes your function, the event structure varies by service.
    # 
    # In this lab, the lambda_handler "event" parameter has the following structure of name/value pairs:
    # {
    #     "emoji_type": number,
    #     "message": "string"
    # }
        
    # In a name-value pair you can extract the "value" of the pair using the "name" of the pair.   
    # Here the emoji_type value is extracted from the event Lambda parameter and passed to a variable called emoji_type.
    emoji_type = event["emoji_type"]
    # Here the message value is extracted from the event Lambda parameter and passed to a variable called message.
    message = event["message"]
    
    # The variables are printed here, which means the variable values will be displayed in CloudWatch logs and the Execution Results panel.
    print(emoji_type)
    print(message)
    
    # In Python, you can create a variable with no value using the Built-in constant None. This means the variable "custom_message" currently has no value.
    custom_message = None
    
    # In Python, we use the if-elif-else to create a conditional execution. https://docs.python.org/3/reference/compound_stmts.html#the-if-statement
    # That is, if the value of emoji_type is equal to 0, we execute the statement inside its block
    if emoji_type == 0:
        # Only execute if emoji_type is equal to 0
        # The variable custom_message combines "Message for code 0:" string with the variable message
        feeling= "positive"

    elif emoji_type == 1:
        # The variable custom_message combines "Message for code 1:" string with the variable message
        feeling= "neutral"

    else:
        # The variable custom_message combines "Message for all other codes:" string with the variable message
        feeling ="negative"
        
    # Optionally, the handler can return a value. What happens to the returned value depends on the invocation type you use when invoking the Lambda function
    # In this lab we use synchronous execution, so we need to create a response for the lambda function.
    # We created a "response" variable that has a structure of name-value pairs using the emoji_type and custom_message created earlier.
    response = {

        # In this name-value pair, the literal value "custom_message" will store the value of the custom_message variable.
        "feeling": feeling,
        # In this name-value pair, the literal value "message" will store the value of the original message variable.
        "message": message,
    }
    
    # Finally, we return the values from response variable to the caller, which could be a test event or an AWS service performing a synchronous call.
    # The execution of the lambda function is finished.
    return response

```

y el test 

```
{
   "emoji_type" : 0,
   "message": "I love the park"
}


```


