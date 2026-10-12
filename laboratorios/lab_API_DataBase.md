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

![Imagen Arquitectura](/imagenes_laboratorios/lab_api_DynamoDB_lambda.png "Diagrama de arquitectura: API Gateway → Lambda → DynamoDB")

---

## Conceptos Clave

| Concepto | Descripción |
|---|---|
| **DynamoDB** | Base de datos NoSQL totalmente administrada por AWS, con latencia de milisegundos a cualquier escala |
| **API** | Interfaz de Programación de Aplicaciones; contrato que define cómo se comunican dos sistemas de software |
| **API Gateway** | Servicio de AWS para crear, publicar, proteger y gestionar APIs a escala |
| **API REST** | Estilo arquitectónico para servicios web que opera sin estado (stateless) usando HTTP |
| **Microservicios** | Arquitectura donde la aplicación se divide en componentes pequeños, independientes y desplegables de forma autónoma |
| **SQL vs NoSQL** | SQL usa esquemas fijos y escalado vertical; NoSQL usa esquemas flexibles y escalado horizontal |
| **Integración Lambda Proxy** | Modo de integración donde API Gateway pasa el evento HTTP completo a Lambda sin transformaciones |
| **Clave de partición** | Atributo primario en DynamoDB que determina la partición física donde se almacena un ítem |
| **Clave de ordenación** | Atributo secundario en DynamoDB que permite organizar ítems con la misma clave de partición |
| **Variables de entorno** | Valores de configuración externos inyectados en el entorno de ejecución de Lambda en tiempo de arranque |
| **CORS** | Cross-Origin Resource Sharing; mecanismo que permite o restringe solicitudes HTTP entre dominios distintos |
| **Stage (API Gateway)** | Entorno de despliegue de una API (por ejemplo: dev, test, prod); cada stage genera su propia URL |
| **IAM Role** | Conjunto de permisos que otorga a un servicio de AWS (como Lambda) acceso controlado a otros servicios |

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

**Tipos de API disponibles en API Gateway:**

| Tipo | Descripción | Casos de uso |
|---|---|---|
| **HTTP API** | Más liviana y económica; soporta OIDC y OAuth 2.0 | Backends simples, latencia mínima |
| **REST API** | Completa; soporta todas las funciones (caché, WAF, validación) | Aplicaciones empresariales |
| **WebSocket API** | Conexión bidireccional persistente | Chat en tiempo real, notificaciones |

---

### SQL vs NoSQL

| Característica | SQL (Relacional) | NoSQL (No relacional) |
|---|---|---|
| Esquema | Fijo y predefinido | Flexible y dinámico |
| Escalabilidad | Vertical (más CPU/RAM) | Horizontal (más nodos) |
| Consultas | SQL estándar | API propia por servicio |
| Transacciones | ACID completas | Eventual consistency (por defecto) |
| Casos de uso | Transacciones complejas, reportes | Apps móviles, web, gaming, IoT |
| Ejemplos AWS | Amazon RDS, Aurora | DynamoDB, ElastiCache |

---

### Amazon DynamoDB

**Definición**: DynamoDB es una base de datos NoSQL totalmente administrada por AWS, diseñada para aplicaciones que requieren baja latencia a cualquier escala.

**Principales beneficios**:

- **Sin servidores**: no requiere aprovisionar, parchear ni administrar servidores, ni instalar software.
- **Alto rendimiento**: latencia en milisegundos de un solo dígito, incluso con millones de solicitudes por segundo.
- **Escalabilidad automática**: maneja grandes volúmenes de datos sin intervención manual; puede escalar hasta petabytes.
- **Alta disponibilidad**: disponibilidad y tolerancia a fallos integradas en múltiples zonas de disponibilidad (AZ), sin necesidad de diseñar esa lógica en la aplicación.
- **Flexibilidad de esquema**: permite agregar atributos a los ítems de forma independiente, agilizando el desarrollo.
- **Seguridad integrada**: cifrado en reposo habilitado por defecto, integración con IAM para control de acceso granular.
- **Streams y eventos**: DynamoDB Streams captura los cambios en la tabla en tiempo real para procesamiento posterior con Lambda u otros servicios.

**Modelo de datos**:

- **Tabla**: colección de ítems (equivale a una tabla en SQL).
- **Ítem**: conjunto de atributos (equivale a una fila en SQL).
- **Atributo**: par clave-valor dentro de un ítem (equivale a una columna en SQL, pero flexible).
- **Clave primaria**: puede ser solo la clave de partición (*simple*) o la combinación de clave de partición + clave de ordenación (*compuesta*).

---

### Escaneos en DynamoDB

Un escaneo (`Scan`) lee todos los elementos de una tabla o de un índice secundario. El proceso ocurre en tres etapas:

1. **Escaneo completo**: se recorre toda la tabla y se devuelven todos los elementos encontrados.
2. **Aplicación de filtros**: se aplica una expresión de filtro con comparaciones lógicas sobre los elementos escaneados.
3. **Resultado final**: el atributo `Count` refleja el número de elementos devueltos después del filtro.

> ⚠️ Los filtros **no** reducen el costo del escaneo: DynamoDB cobra por los elementos leídos **antes** de aplicar el filtro, no por los elementos devueltos. Para consultas eficientes, utiliza operaciones `Query` con claves de partición conocidas en lugar de `Scan`.

---

### Métodos HTTP y Códigos de Estado

Las APIs REST utilizan métodos HTTP estándar para expresar la intención de una operación:

| Método | Operación CRUD | Descripción |
|---|---|---|
| `GET` | Read | Recuperar un recurso o lista de recursos |
| `POST` | Create | Crear un nuevo recurso |
| `PUT` | Update | Reemplazar un recurso existente completo |
| `PATCH` | Update parcial | Actualizar solo algunos campos de un recurso |
| `DELETE` | Delete | Eliminar un recurso |

**Códigos de estado HTTP comunes**:

| Código | Significado |
|---|---|
| `200 OK` | Solicitud exitosa |
| `201 Created` | Recurso creado exitosamente |
| `400 Bad Request` | Solicitud inválida (error del cliente) |
| `401 Unauthorized` | Requiere autenticación |
| `403 Forbidden` | Sin permisos para el recurso |
| `404 Not Found` | Recurso no encontrado |
| `500 Internal Server Error` | Error en el servidor/backend |

---

### CORS (Cross-Origin Resource Sharing)

CORS es un mecanismo de seguridad del navegador que controla si las aplicaciones web de un dominio pueden acceder a recursos de un dominio diferente. Las APIs a las que acceden aplicaciones basadas en navegador típicamente requieren la configuración de CORS para especificar qué orígenes pueden hacer solicitudes.

**Ejemplo**: si tu frontend está en `https://miaplicacion.com` y tu API en `https://api.miaplicacion.com`, el navegador bloqueará las solicitudes a menos que la API incluya los encabezados CORS adecuados.

**Encabezados CORS principales**:
- `Access-Control-Allow-Origin`: dominios autorizados a realizar solicitudes.
- `Access-Control-Allow-Methods`: métodos HTTP permitidos (GET, POST, etc.).
- `Access-Control-Allow-Headers`: encabezados personalizados permitidos en la solicitud.

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

---

## Práctica: Pasos del Laboratorio

En este laboratorio podrás:

1. Crear una tabla de Amazon DynamoDB.
2. Definir el esquema de la tabla (clave de partición y clave de ordenación).
3. Crear una función Lambda para **crear**, **actualizar** y **consultar** elementos de la tabla.
4. Exponer la función Lambda como una API RESTful mediante Amazon API Gateway.

---

### Paso 1: Crear la tabla en DynamoDB

Nos dirigimos al servicio de DynamoDB en la consola de AWS y damos clic en **Create table**.

Requisitos de la tabla:

| Parámetro | Valor |
|---|---|
| Nombre de la tabla | `rental_app` |
| Partition key | `record_type` |
| Sort key | `id` |
| Table settings | Default settings |

Hacemos clic en **Create table**.

---

### Paso 2: Esperar a que la tabla esté activa

Solo queda esperar a que el *status* de la tabla pase a **Active**.

> DynamoDB aplica una función hash al valor de la clave de partición para determinar la partición física donde se almacenan los datos. Este mecanismo de distribución admite el escalado horizontal y el acceso de alto rendimiento. Cuando una tabla incluye tanto clave de partición como de ordenación, varios ítems pueden compartir la misma clave de partición, pero deben tener valores de clave de ordenación únicos.

---

### Paso 3: Crear la función Lambda

Nos dirigimos al servicio Lambda. Creamos una nueva función con los siguientes parámetros:

| Parámetro | Valor |
|---|---|
| Método de creación | Author from scratch |
| Function name | `labFunction` |
| Runtime | Python 3.12 |
| Architecture | x86_64 |
| Execution role | Use an existing role → `lab_function_role_xxxxx` |

Hacemos clic en **Create function**.

---

### Paso 4: Desplegar el código de la función

Dentro del entorno de Lambda, pegamos nuestra función en la sección de **Code**. El editor integrado es similar a Visual Studio Code.

**¿Cómo gestiona Lambda el código?**

- Lambda almacena la función en Amazon S3 cifrado en reposo.
- Proporciona un almacenamiento de código seguro y duradero.
- Al implementar código a través de la consola, Lambda maneja automáticamente la carga en S3 y mantiene el historial de versiones para capacidades de reversión.

Siempre es necesario actualizar y desplegar la función. Para eso, hacemos clic en **Deploy**.

> Nuestra función obtiene el nombre de la tabla de DynamoDB mediante una variable de entorno de Lambda, por lo que a continuación debemos definirla en la sección de **Configuration**.

---

### Paso 5: Definir variable de entorno

En la sección **Configuration → Environment variables**, hacemos clic en **Edit** y agregamos una nueva variable:

| Key | Value |
|---|---|
| `TABLE_NAME` | `rental_app` |

Hacemos clic en **Save**.

---

### Paso 6: Pruebas (Testing)

Una parte fundamental es testear nuestra función Lambda antes de exponerla como API. En la sección **Code**, hacemos clic en el botón **Test** y luego en **Create new test event**.

Configuramos el evento de prueba:

| Parámetro | Valor |
|---|---|
| Event name | `create_location` |
| Event sharing settings | Private |
| Template | hello-world |
| Event JSON | *(ver código de referencia del laboratorio, líneas 178–182)* |

Hacemos clic en **Save** y luego en **Test**. Revisamos la consola de **Output** para ver los resultados.

Como resultado del testing, se debería haber creado un elemento en la tabla DynamoDB. Para verificarlo:

1. Vamos al servicio DynamoDB.
2. Abrimos la tabla `rental_app`.
3. En la sección **Explore table items**, el nuevo elemento debe aparecer en la lista.

> Para los siguientes pasos del laboratorio, copia el `id` de la ubicación creada.

---

### Paso 7: Creación de la API en API Gateway

El servicio de API Gateway ofrece tres tipos de API: **HTTP**, **REST** y **WebSocket**. Todas admiten patrones RESTful con métodos HTTP estándar (GET, POST, PUT, PATCH, DELETE) para la comunicación cliente-servidor sin estado.

Seleccionamos la tarjeta de **REST API** y configuramos:

| Parámetro | Valor |
|---|---|
| Tipo | New API |
| API name | `rental_app` |
| Description | *(opcional)* |
| Security policy | TLS 1.2 |

Hacemos clic en **Create API**.

---

#### Paso 7.1: Creación de recursos

Los recursos representan segmentos de la ruta URL en la estructura de una API y sirven como contenedores para los métodos HTTP. Los recursos se pueden anidar para crear estructuras jerárquicas, por ejemplo: `/locations/{id}/vehicles`.

Creamos el recurso:

| Parámetro | Valor |
|---|---|
| Resource path | `/` |
| Resource name | `locations` |

Hacemos clic en **Create resource**.

> **Nota sobre CORS**: Cross-Origin Resource Sharing es un mecanismo de seguridad del navegador que controla si las aplicaciones web de un dominio pueden acceder a recursos de un dominio diferente. Las APIs a las que acceden aplicaciones basadas en navegador típicamente requieren configuración de CORS para especificar qué orígenes pueden hacer solicitudes.

---

#### Paso 7.2: Crear el método GET

Dentro del recurso `/locations`, definimos el método **GET**:

| Parámetro | Valor |
|---|---|
| Tipo de integración | Lambda Function |
| Lambda proxy integration | Activado ✅ |
| Lambda function | `labFunction` |
| Integration timeout | 29,000 ms (máximo) |

> La integración timeout tiene una duración máxima de **29,000 ms (29 segundos)**. Los sistemas que requieran mayor tiempo de procesamiento deben migrar a una arquitectura asíncrona con mecanismos de devolución de llamada (callbacks).

Como resultado, veremos el método `GET` creado debajo de `/locations`.

**Testing del método GET**: Al igual que en Lambda, podemos testear nuestra API directamente desde la pestaña **Test**. Podemos ingresar headers opcionales y hacer clic en **Test**. Un **status 200** indica que la solicitud fue exitosa.

---

#### Paso 7.3: Crear el método POST

Seguimos los mismos pasos del método GET para crear el método **POST** en el recurso `/locations`.

Para el testing, usamos el siguiente cuerpo de solicitud:

```json
{
  "name": "viper roller coaster",
  "vehicles_available": 5
}
```

El resultado esperado es un **status 201**, que indica que el recurso fue creado exitosamente.

Volvemos a DynamoDB y realizamos un escaneo de la tabla para verificar que el nuevo elemento fue ingresado correctamente.

---

### Paso 8: Desplegar la API

Por último, desplegamos nuestra API mediante la opción **Deploy API**. Definimos el stage de destino:

| Parámetro | Valor |
|---|---|
| Stage | New stage |
| Stage name | `Test` |

Hacemos clic en **Deploy**. API Gateway generará la URL base del stage, que tendrá la forma:

```
https://{api-id}.execute-api.{region}.amazonaws.com/Test
```

---

## DIY

*(Sección pendiente de completar con ejercicios propios del laboratorio.)*

---

## Recursos Adicionales

- [Documentación de Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [Documentación de AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Documentación de Amazon API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [Integración Lambda con API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-lambda-function-handler.html)
- [DynamoDB: operaciones Scan](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Scan.html)
- [DynamoDB: operaciones Query vs Scan](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-query-scan.html)
- [CORS en API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)
