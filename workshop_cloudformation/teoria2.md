# Recursos 

en este laboratorio aprenderemos un poco acerca de CloudFormation top-level sections, incluuyendo la version del formato, descripcion, metadata, parametro y recursos.

## Temas
- entender la estructura de la pantilla(template) de cloudformation, y algunas de sus secciones. 
- desplegar una instancia ec2 via cloudformation. 
- lanzar una consulta para saber el ID de la **AMI** mas reciente de **Amazon Linux**

### Formato

la AWSTemplateFormatVersion identifica las capacidades de la plantilla. la ultima version sigue siendo la 2010-09-09 y es el unico valor valido.

```
AWSTemplateFormatVersion: "2010-09-09"

```

## Descripcion
te habilita para incluir comentarios en tu plantilla.

```
Description: AWS CloudFormation workshop - Resources (uksb-1q9p31idr) (tag:resources).

```

## Metadata
puedes utilizar la seccion de metadata para incluir objetos JSON o YAML. esta seccion es de utilidad para proveer informacion a otras herramientas que interactuan con CloudFormation. por ejemplo, desplegando CloudFormation templates via Consola, puedes mejorar la experiencia de los usuarios especificando el orde, etiquetado y parametros de grupo. 

```
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: 'Amazon EC2 Configuration'
        Parameters:
          - InstanceType
    ParameterLabels:
      InstanceType:
        default: 'Type of EC2 Instance'

```

## Parametros

los parametros te brindan valores personalizados para tu plantilla, como un formulario y las diferentes opciones validas al crear/actualizar tu Stack 

```

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
    Description: 'Enter t2.micro or t2.small. Default is t2.micro.'

```

|Type |	Description	| Example |
| ---------------|---------|-------- |
|String	|A literal string.|	"MyUserName"|
|Number	|An integer or float.|	"123"|
|List<Number> |	An array of integers or floats.|	"10,20,30"|
|CommaDelimitedList|	An array of literal strings.|	"test,dev,prod"|
|AWS-Specific Parameter Types |	AWS values such as Amazon VPC IDs.	|AWS::EC2::VPC::Id |
|SSM Parameter Types |	Parameters that correspond to existing parameters in Systems Manager Parameter Store.|AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>|

## Resources

la seccion recursos requeridos declara que Recursos de AWS seran creados en el stack. 

```
Resources:
  WebServerInstance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: <replace with AMI ID ami-xxxxx>

```

The only required property of the EC2 resource type is ImageId. Let's find the AMI ID via AWS console:

Open AWS EC2 console 
Click Instances -> Launch Instance.
Copy the Amazon Linux 2023 AMI ami-xxxxxxxxx ID.