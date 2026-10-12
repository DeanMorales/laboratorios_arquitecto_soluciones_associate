# Laboratorio: Respaldos Automatizados con EBS y AWS Backup

---

## Caso de Uso

Una empresa de consultoría de marketing utiliza múltiples scripts personalizados para realizar copias de seguridad de sus aplicaciones. Estos scripts se están volviendo cada vez más difíciles de mantener. La empresa necesita hacer copias de seguridad de su almacenamiento de host bastión, almacenamiento de servidor web centralizado y snapshots de base de datos de diferentes servicios de AWS. Buscan una solución de copia de seguridad unificada y automatizada que agilice la gestión y proporcione una protección de datos consistente.

---

## Objetivos de Aprendizaje

- Crear un plan de respaldo para los volúmenes de Amazon EBS mediante etiquetas.
- Realizar un respaldo bajo demanda.

---

## Arquitectura del Laboratorio

![ laboratorio_respaldos_datos ](  /imagenes_laboratorios/lab_respaldos_datos.png  "laboratorio_respaldos_datos")

---

## Conceptos Clave

| Concepto | Descripción |
|---|---|
| **AWS Backup** | Servicio administrado que centraliza y automatiza la protección de datos en todos los servicios de AWS |
| **Backup Vault (Bóveda de respaldo)** | Contenedor que almacena y organiza los respaldos, con políticas de acceso y cifrado propias |
| **Backup Plan (Plan de respaldo)** | Política que define cuándo y cómo se respaldan los recursos: frecuencia, retención y ciclo de vida |
| **Resource Assignment** | Mecanismo para vincular recursos (por ID o etiqueta) a un plan de respaldo |
| **On-Demand Backup** | Respaldo manual e inmediato, fuera del calendario programado |
| **Amazon EBS** | Elastic Block Store; almacenamiento en bloque persistente para instancias EC2 |
| **Snapshot** | Copia puntual de un volumen EBS almacenada en S3 (gestionada por AWS) |
| **AWS KMS** | Key Management Service; servicio administrado para crear y controlar claves de cifrado |
| **CMK** | Customer Managed Key; clave de cifrado creada y administrada por el cliente en KMS |
| **Etiqueta (Tag)** | Par clave-valor que se puede asignar a recursos de AWS para clasificarlos y aplicarles políticas |
| **RTO / RPO** | Recovery Time Objective (tiempo máximo de recuperación) y Recovery Point Objective (pérdida máxima de datos aceptable) |
| **Crash-consistent backup** | Copia de seguridad coherente en caso de fallo; captura todos los volúmenes de una instancia en el mismo instante |

---

## Teoría

### Respaldos de Datos con AWS Backup

AWS Backup permite centralizar y automatizar la protección de datos en todos los servicios de AWS, eliminando la necesidad de scripts personalizados y procesos de respaldo manuales.

**Descripción de AWS Backup**

AWS Backup es un servicio totalmente administrado por Amazon, basado en políticas, por lo que no es necesario preocuparse por el escalado de las copias de seguridad.

Actualmente, AWS Backup cubre los siguientes servicios:

- Amazon EBS (Elastic Block Store)
- Amazon EFS (Elastic File System)
- Amazon FSx
- Amazon EC2 (Elastic Compute Cloud)
- Amazon RDS
- Amazon DynamoDB
- Amazon Aurora
- Amazon S3

Ofrece protección de datos centralizada y automatizada, y brinda soporte para regulaciones de cumplimiento y políticas empresariales. Junto con AWS Organizations, permite desplegar diferentes políticas de protección de datos para configurar, administrar y gobernar múltiples cuentas y sus recursos.

**Beneficios principales**

| Beneficio | Descripción |
|---|---|
| **Centralización** | Mediante la consola, AWS CLI y APIs, puedes configurar políticas de respaldo de forma rápida y simple, asegurando recursos a través de toda la cuenta |
| **Automatización de procesos** | Basado en políticas: programa respaldos, gestiona la retención y el ciclo de vida automáticamente, eliminando scripts personalizados y reduciendo costos operativos |
| **Cumplimiento (Compliance)** | Aplica cifrado, auditoría y control de acceso; permite diseñar una estrategia alineada con los requerimientos regulatorios (HIPAA, PCI DSS, etc.) |
| **Integración con Organizations** | Permite gobernar las políticas de backup a nivel de organización completa desde una sola cuenta de gestión |

**Componentes clave de AWS Backup**

- **Bóveda de respaldo (Backup Vault)**: contenedor que almacena y organiza los respaldos. Tiene su propia política de acceso y clave de cifrado KMS.
- **Plan de respaldo (Backup Plan)**: define cuándo y cómo se realizan las copias de seguridad, incluyendo la frecuencia, los períodos de retención y las políticas de ciclo de vida. Se pueden crear múltiples planes para distintos requisitos.
- **Asignación de recursos**: los recursos se vinculan a un plan de respaldo usando etiquetas o IDs de recursos. AWS Backup los respalda automáticamente de acuerdo con el calendario y las reglas de retención del plan.

---

### AWS KMS (Key Management Service)

Cada bóveda de respaldo requiere una clave de cifrado de AWS KMS para cifrar todos los respaldos almacenados dentro de ella, proporcionando seguridad de los datos en reposo.

**Descripción de AWS KMS**

AWS KMS es un servicio administrado por AWS que facilita la administración del cifrado de datos y controla el uso de claves criptográficas en una amplia gama de servicios de AWS (S3, EC2, RDS, EBS, etc.). Las claves pueden generarse en KMS o importarse desde una infraestructura de administración de claves externa.

KMS opera bajo el estándar **FIPS 140-2**, que permite validar, aislar y proteger los módulos criptográficos.

**Características principales**:

- Puedes utilizar KMS para cifrar y descifrar datos en tus servicios; los recursos de AWS pueden usar tus claves para proteger la información almacenada.
- Las claves de KMS **nunca salen de la bóveda de KMS**, por lo que siempre tendrás acceso centralizado para administrarlas y darles mantenimiento en un solo lugar.
- Puedes crear políticas de uso de claves (Key Policies), lo que permite determinar qué usuarios y servicios tienen acceso a cada clave y cómo pueden utilizarla.
- Todas las llamadas al servicio de KMS se registran como logs en **AWS CloudTrail** para seguimiento y auditoría.

**Beneficios**

| Beneficio | Descripción |
|---|---|
| **Seguridad física** | AWS gestiona la seguridad física, la durabilidad y el refuerzo por políticas de los módulos HSM |
| **Cifrado y firma digital** | Integra cifrado mediante el SDK de AWS; usa claves asimétricas para firma digital y garantiza la integridad de la información |
| **Cumplimiento** | AWS KMS está validado bajo múltiples estándares: PCI DSS, FIPS 140-2, HIPAA y otros marcos regulatorios |
| **Instancias HSM** | Utiliza módulos de seguridad de hardware (HSM) dedicados para el almacenamiento seguro de claves |

> Para respaldar una base de datos de Amazon Aurora, esta solución utiliza el almacén de respaldo predeterminado que AWS Backup crea automáticamente para cada cuenta de AWS.

---

## Laboratorio

### Respaldo de Datos con AWS Backup

En este laboratorio realizaremos las siguientes acciones:

- Crear una bóveda de respaldo personalizada.
- Configurar una copia de seguridad bajo demanda de los volúmenes de Amazon EBS adjuntos a instancias EC2, usando el ID del recurso.
- Crear planes de respaldo automatizados para volúmenes de EBS usando etiquetas.

---

### Paso 1: Revisar las instancias EC2 y etiquetar el volumen EBS

Nos dirigimos al servicio **EC2** en la consola de AWS para revisar las instancias actuales. Seleccionamos la instancia a la que deseamos realizar el backup.

1. En la configuración de la instancia, revisamos la pestaña de **Storage** para identificar el volumen EBS conectado.
2. Copiamos el **Volume ID** del volumen asociado (por ejemplo: `vol-0gf0d987fg09d7h`, device name `/dev/xvda`).

---

### Paso 2: Agregar etiqueta al volumen EBS

Nos dirigimos a la sección **Elastic Block Store → Volumes** y seleccionamos el volumen cuyo ID copiamos anteriormente.

1. Dentro de su configuración, abrimos la pestaña **Tags** y hacemos clic en **Manage tags**.
2. Agregamos la siguiente etiqueta y hacemos clic en **Save**:

| Key | Value |
|---|---|
| `Backup` | `True` |

---

### Paso 3: Agregar etiqueta a la instancia EC2

Regresamos a nuestra instancia EC2 y le agregamos la misma etiqueta (`Backup: True`) para agruparla con sus volúmenes y facilitar la asignación al plan de respaldo.

---

### Paso 4: Etiquetar el Bastion Host

Repetimos el proceso de etiquetado para la instancia **Bastion Host**, con el objetivo de agrupar todos los recursos de la aplicación y respaldarlos de forma conjunta.

---

### Paso 5: Crear la Bóveda de Respaldo en AWS Backup

Nos dirigimos al servicio **AWS Backup** en la consola. Abrimos la sección **Vaults** y creamos una nueva bóveda con los siguientes parámetros:

| Parámetro | Valor |
|---|---|
| Nombre | `any_company_backup` |
| Vault type | Backup Vault |
| Encryption key | `Default (aws/backup)` |

Hacemos clic en **Create vault**.

---

### Paso 6: Crear un Respaldo Bajo Demanda

Nos dirigimos a **Protected Resources**.

> La página **Protected Resources** muestra los recursos de AWS que AWS Backup ha respaldado al menos una vez. Los recursos asignados a planes de respaldo que aún no han sido respaldados no aparecerán hasta que se complete su primer trabajo de respaldo programado o bajo demanda.

Creamos un respaldo bajo demanda con los siguientes parámetros:

| Parámetro | Valor |
|---|---|
| Resource type | EBS |
| Resource ID | *(ID del volumen copiado anteriormente)* |
| Backup window | Create backup now |
| Total retention period | 7 days |
| Backup vault | `any_company_backup` |
| IAM role | `backup_role` |

Hacemos clic en **Create on-demand backup**.

> AWS Backup crea copias de seguridad **coherentes en caso de bloqueo** (*crash-consistent*) de los volúmenes de EBS conectados a instancias EC2 de forma predeterminada. Esto significa que las instantáneas de todos los volúmenes conectados a la misma instancia se capturan en el mismo momento, lo que ayuda a mantener la integridad de los datos en configuraciones de múltiples volúmenes.

> AWS Backup se integra con **AWS Identity and Access Management (IAM)** para controlar las operaciones de backup. Los roles de IAM definen los permisos que AWS Backup necesita para acceder y proteger los recursos en el entorno de AWS.

El proceso de backup tarda aproximadamente **5 a 6 minutos**.

---

### Paso 7: Crear un Plan de Respaldo Automatizado

Dentro de las opciones de AWS Backup, accedemos a **Backup plans** y creamos un plan personalizado.

> Un plan de respaldo es una expresión de política que define horarios de respaldo, períodos de retención y reglas de ciclo de vida para los recursos de AWS. Los planes automatizan la protección en varios recursos y tipos de recursos, como tablas de Amazon DynamoDB, sistemas de archivos de Amazon EFS y volúmenes de Amazon EBS.

Seleccionamos **Build a new plan** y configuramos:

| Parámetro | Valor |
|---|---|
| Plan name | `any_company_backup_plan` |

**Configuración de la regla de respaldo**:

| Parámetro | Valor |
|---|---|
| Rule name | `backup_rule` |
| Backup vault | `any_company_backup` |
| Backup frequency | Weekly — Sunday |
| Start time | Default |

Hacemos clic en **Create plan**.

> Las **ventanas de backup** definen cuándo pueden iniciarse los trabajos de respaldo. La configuración predeterminada inicia los backups dentro de una ventana de 8 horas a partir de las 5:00 AM UTC, con un requisito de finalización dentro de las 8 horas posteriores a su hora de inicio.

---

### Paso 8: Asignar Recursos al Plan

> Las **asignaciones de recursos** conectan los planes de respaldo a recursos específicos de AWS. Cada asignación requiere un rol de IAM que otorgue a AWS Backup los permisos necesarios para realizar operaciones de respaldo en los recursos asignados.

Configuramos la asignación con los siguientes parámetros:

| Parámetro | Valor |
|---|---|
| Assignment name | *(nombre de preferencia)* |
| IAM role | `backup_role` |
| Resource selection | Tagged resources |
| Include all resource types | ✅ Activado |

**Definir selección por etiquetas**:

| Key | Condition | Value |
|---|---|---|
| `Backup` | Equals | `True` |

Hacemos clic en **Assign resources**.

> ⚠️ Ten en cuenta que las etiquetas **distinguen mayúsculas y minúsculas**. Asegúrate de que el valor `True` coincida exactamente con el que asignaste en los pasos anteriores.

Al confirmar, AWS Backup puede mostrar el siguiente aviso:

> *"Seleccionando todos los recursos, proteges todos los recursos de tu cuenta. Tu selección actual podría respaldar varios de ellos e incurrir en implicaciones de costos."*

Aceptamos y continuamos.

---

### Paso 9: Verificar los Jobs de Respaldo

Nos dirigimos a la sección **Jobs** de AWS Backup.

> Los **trabajos de backup (Jobs)** representan operaciones de respaldo individuales. La página muestra el estado de todas las operaciones de backup y restauración, incluyendo backups bajo demanda y ejecuciones de planes de respaldo programados.

Verificamos que el último trabajo aparezca como **Completed**. Cada trabajo tiene su propio ID de seguimiento.

Luego navegamos a **Protected Resources** y confirmamos que el volumen EBS respaldado aparece en la lista. Hacemos clic sobre él para ver el historial de puntos de recuperación disponibles.

> Las organizaciones suelen crear **múltiples planes de respaldo** para abordar diferentes objetivos de recuperación según la carga de trabajo. Por ejemplo: las bases de datos de producción pueden requerir respaldos diarios con retención de 30 días, mientras que los recursos de desarrollo solo necesitan respaldos semanales con retención de 7 días.

---

## DIY

*(Sección pendiente de completar con ejercicios propios del laboratorio.)*

---

## Recursos Adicionales

- [Documentación de AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
- [Documentación de Amazon EBS Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html)
- [Documentación de AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
- [Planes de respaldo en AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/creating-a-backup-plan.html)
- [AWS Backup con AWS Organizations](https://docs.aws.amazon.com/aws-backup/latest/devguide/manage-cross-account.html)
- [Mejores prácticas de AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/best-practices.html)
