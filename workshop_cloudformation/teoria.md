# Workshop CloudFormation - Notas del Curso

## Configuración previa

Antes de comenzar, asegúrate de tener lo siguiente:

- Una cuenta activa en AWS (es obligatorio)
- Credenciales configuradas correctamente en tu máquina local con **AWS CLI**
- Un IDE de tu preferencia instalado
- **CloudFormation Linter** instalado para validar plantillas

> **Nota:** Solo pagarás por los recursos que utilices. CloudFormation en sí no tiene costo adicional; se paga únicamente por el tiempo de los recursos desplegados.

---

## Teoría del Workshop

### ¿Qué es un Template de CloudFormation?

Un **template** (plantilla) es un documento de texto donde se especifican los recursos que se desplegarán en AWS. Puede estar en formato YAML o JSON.

### Estructura de una plantilla

```yaml
AWSTemplateFormatVersion: 'version date' (opcional)
# Versión del template. El único valor válido es '2010-09-09'

Description: 'String' (opcional)
# Descripción del template

Metadata: 'template metadata' (opcional)
# Objetos con información adicional sobre el template

Parameters: 'set of parameters' (opcional)
# Conjuntos de entradas para personalizar el template

Rules: 'set of rules' (opcional)
# Reglas para validar los parámetros durante el despliegue/actualización

Mappings: 'set of mappings' (opcional)
# Mapeo de claves y valores asociados

Conditions: 'set of conditions' (opcional)
# Condiciones que controlan si ciertos recursos se crean

Transform: 'set of transforms' (opcional)
# Para aplicaciones serverless

Resources: 'set of resources' (obligatorio)
# Componentes de tu infraestructura - ¡es el único requisito obligatorio!

Hooks: 'set of hooks' (opcional)
# Usado para ECS Blue/Green Deployments

Outputs: 'set of outputs' (opcional)
# Valores que se retornan al ver las propiedades del stack
```

> **IMPORTANTE:** El único elemento obligatorio es `Resources`, ya que necesitamos al menos un recurso a crear en AWS.

---

## ¿Qué es un Stack?

Un **stack** es la infraestructura desplegada en la consola de AWS. Sus características principales son:

- **Reutilizable:** Se puede crear y eliminar múltiples veces
- **Rápido:** La eliminación completa toma solo unos minutos
- **Rollback automático:** Si AWS no puede crear un recurso, por defecto se eliminan todos los recursos creados hasta el momento del fallo. Este comportamiento se puede configurar.

---

## Flujo de creación de un Stack

1. Definir los recursos en tu template (YAML o JSON)
2. Guardar el archivo en un bucket S3 (opcional, pero recomendado)
3. Abrir el servicio **CloudFormation** en la consola
4. Ingresar los parámetros y crear un nuevo stack
5. Monitorear el despliegue hasta obtener el estado **CREATE_COMPLETE**

---

## Práctica: Crear tu primer S3 con CLI

### 1. Configurar credenciales

Primero, asegúrate de tener tus credenciales de AWS configuradas:

```bash
aws configure
```

### 2. Crear un bucket S3

```bash
# Crear bucket en la región por defecto
aws s3 mb s3://mi-nombre-de-bucket-unico

# Crear bucket en una región específica
aws s3 mb s3://mi-nombre-de-bucket-unico --region us-west-2
```

### 3. Subir el template a S3

```bash
# Usar ruta absoluta
aws s3 cp /ruta/local/archivo.yaml s3://nombre-del-bucket/

# O desde la raíz del proyecto (ruta relativa)
aws s3 cp workshop_cloudformation/cloudformation01.yaml s3://practicas-cloudformation-deanmorales
```

---

## Crear el Stack con CloudFormation CLI

```bash
aws cloudformation create-stack \
  --stack-name cfn-workshop-template-and-stack \
  --template-body file://cloudformation01.yaml
```

**Respuesta:**

```json
{
    "StackId": "arn:aws:cloudformation:us-east-1:826496717630:stack/cfn-workshop-template-and-stack/c3b97a40-7fa5-11f1-849c-1229f76eacc3",
    "OperationId": "c3bbc430-7fa5-11f1-849c-1229f76eacc3"
}
```

> **Nota:** El comando retorna un JSON con el ID del stack. Puedes verificar su estado en la consola de CloudFormation.

---

## Challenge: Agregar Versionamiento al Bucket S3

Para habilitar el versionamiento en el bucket, agrega la siguiente propiedad:

```yaml
VersioningConfiguration:
  Status: Enabled
```

### Pasos actualizados:

1. Modifica el template con la configuración de versionamiento
2. Sube el template actualizado al bucket S3:

```bash
aws s3 cp workshop_cloudformation/cloudformation01.yaml s3://practicas-cloudformation-deanmorales
```

3. Actualiza el stack:

```bash
aws cloudformation update-stack \
  --stack-name cfn-workshop-template-and-stack \
  --template-body file://cloudformation01.yaml
```

### Template final:

```yaml
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      # Versionamiento del bucket
      VersioningConfiguration:
        Status: Enabled
      # Cifrado del bucket
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
```

---

## Eliminar el Stack con CLI

```bash
aws cloudformation delete-stack --stack-name cfn-workshop-template-and-stack
```

### Opciones adicionales:

```bash
# Eliminar y esperar a que termine
aws cloudformation delete-stack \
  --stack-name cfn-workshop-template-and-stack \
  --wait

# Eliminar pero retener ciertos recursos
aws cloudformation delete-stack \
  --stack-name cfn-workshop-template-and-stack \
  --retain-resources RecursoLogico1 RecursoLogico2
```

---

## Recomendaciones

1. **Usa el CLI de CloudFormation**: Te permite automatizar despliegues y es más rápido que usar la consola.

2. **Valida tu template antes de desplegar**:
   ```bash
   cfn-lint cloudformation01.yaml
   ```

3. **Habilita Termination Protection**: Evita eliminaciones accidentales del stack.

4. **Usa Change Sets**: Antes de actualizar un stack importante, revisa los cambios que se aplicarán:
   ```bash
   aws cloudformation create-change-set \
     --stack-name mi-stack \
     --template-body file://template.yaml \
     --change-set-name my-change-set
   ```

5. **Monitorea los eventos**: Puedes ver el progreso en tiempo real:
   ```bash
   aws cloudformation describe-stack-events \
     --stack-name cfn-workshop-template-and-stack
   ```

6. ** limpia tus recursos**: Siempre elimina los stacks que no uses para evitar cargos inesperados.