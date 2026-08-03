# Instrinsic Functions

este laboratorio muestra como utilizar las *Funciones intrinsecas* en tu template. 

Las funciones intrinsecas son funciones integradas que le ayudan a administrar sus pilas. sin ellos, estaras limitado a plantillas muy basicas. similares al la un template basico de un **S3**

## Temas tratados

- utilizar la **Referencia**  *!Ref* para asignar dinamicamente los valores a una propiedad de recurso.
- etiqueta una instancia con Fn::join
- agregar una etiqueta a la instancias usando funciones.

### inicio laboratorio

> en los laboratorios pasados habiamos hardcodeado* las AMI ID directamente para levantar una EC2

ahora crearemos una funcion con referencia para hacerlo mas dinamico y flexible, y pasar esta variable al runtime. 

1. primero debemos crear un nuevo parametro llamado *AmiID* y despues debemos meterla dentro de la seccion de *Parametros* de tu template. 

```
AmiID:
  Type: AWS::EC2::Image::Id
  Description: 'The ID of the AMI.'

```
2. utilizar la funcion intrinsica de *Ref* para pasar el parametro de *AmiID*¨dentro de las proppiedades de la EC2

```
Resources:
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      # Use !Ref function in ImageId property
      ImageId: !Ref AmiID
      InstanceType: !Ref InstanceType

```

#### Fn::Join

para ayudar a la administracion de tus recursos de AWS, tu puedes opcionalmente asignar tu propia metadata a cada uno de los recursos en forma de **tags**, Cada etiqueta es un nombre simple que consisten en una llave definida por usuario, y un valor opcional que te ayuda a categorizar los recursos por proposito, dueño o ecosistema, u otro criterio, vamos a usuar una funcion intrinseca **Fn::Join** para nombrar la instancia. 

- agregar una propieda *Tags a la *Properties* seccion
- Referenciar el parametro **InstanceType** y la palabra webserver, delimitado por un dash - a la propiedad de la etiqueta.
```
    Resources:
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref AmiID
      InstanceType: !Ref InstanceType
      Tags:
        - Key: Name
          Value: !Join [ '-', [ !Ref InstanceType, webserver ] ]
```

3. procedemos a realizar los comandos para subirlo mediante la AWS **CLI**

### Primero, crea un bucket (si no tienes uno)
```
aws s3 mb s3://mi-bucket-cloudformation --region us-east-1
```
### Sube la plantilla
```
aws s3 cp cloudformation03.yaml s3://mi-bucket-cloudformation/
```
### Crear stack en cloudformation
Use the AWS CLI to create the stack. The required parameter 
```
aws cloudformation create-stack --stack-name cfn-workshop-intrinsic-functions --template-body file://intrinsic-functions.yaml --parameters ParameterKey="AmiID",ParameterValue="MyAmiId"
```
>Actualiza "MyAmiId" a una un ID de AMI de tu preferencia en este caso es: ami-02b64aa047cb5edf5
>> actualiza la ruta de archivo con el link https:// del archivo del bucket


### Ver estado del stack
aws cloudformation describe-stacks --stack-name MiInstanciaEC2 --region us-east-1

### Ver eventos del stack
aws cloudformation describe-stack-events --stack-name MiInstanciaEC2 --region us-east-1

