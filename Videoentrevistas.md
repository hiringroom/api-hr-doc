# Contenido

- [Integración third-party Videoentrevistas](#integración-third-party-videoentrevistas)
- [Autorización](#autorización)
- [Autenticación mediante Bearer Token](#autenticación-mediante-bearer-token)
- [Configurando videoentrevistas](#configurando-videoentrevistas)
- [Creando videoentrevistas](#creando-videoentrevistas)
- [Manejo de errores](#manejo-de-errores)
- [Eliminando videoentrevistas](#eliminando-videoentrevistas)
- [Cambio de dominio](#cambio-de-dominio)

# Integración third-party Videoentrevistas

El API HR está preparado para integrarse con aplicaciones de terceros (third-party) especializadas en videoentrevistas. Para ello, debe pedir al asesor comercial, de soporte o tecnología que le facilite los accesos necesarios para poder realizar la integración. Estos accesos son el **client_id** y el **client_secret**. A su vez, debe proveerle a dicho asesor la siguiente información para poder obtener los accesos correspondientes:

- **fullname** : Nombre completo de la aplicación third-party.
- **description** : Descripción completa de lo que hace la aplicación third-party.
- **description_link** (opcional) : URL donde se describa o se redirija a la aplicación third-party.
- **url_pic** : URL del logo del third-party.
- **redirect_uri**: URL de redirección en donde el third-party obtendrá el codigo de autorización una vez que el cliente autorice la aplicación.
- **user_agent**: Identificador que se le dará a cada request que haga el third-party al API HR. El mismo se configurará en la conexión entre las plataformas, y será validado en cada request.

El protocolo de autenticación que usa el API HR para integraciones es el protocolo OAuth2.

# Autorización

Para que la aplicación third-party obtenga el **AccessToken** correspondiente para realizar acciones con la cuenta del cliente, primeramente debe pasar por un proceso de autorización en donde el cliente autoriza al third-party para que pueda hacer uso de los datos requeridos, una vez obtenido el codigo de autorización, se puede obtener el **AccessToken**.

El link de autorización , tiene la siguiente forma:

```
 https://api.hiringroom.com/v0/oauth2/authorization?response_type=authcode&client_id=[CLIENT_ID]&account=[ACCOUNTNAME]&state=[STATE]
```

- **CLIENT_ID**: client_id que se provee.
- **ACCOUNTNAME**: Subdominio de la cuenta del cliente
- **STATE**: Estado que el third-party desee mantener en la url (ejemplo: un parametro, un email, un nombre, etc)

Una vez que se genera la url para el cliente, el cliente (una vez tenga una sesión iniciada en HiringRoom o inicie una nueva en el proceso) verá esta pantalla para dar permiso al third-party:

![8](https://i.imgur.com/8x3GgvV.png)

Cuando el cliente acepte, se enviará al **redirect_uri** provisto anteriormente el codigo de autorizacion

```
[redirect_uri]?authCode=[AUTH_CODE]&state=[STATE]
```

El codigo de autorización sólo dura unos minutos, así que si no se pide el **AccessToken** correspondiente lo antes posible, el codigo de autorización quedará inválido y será necesario realizar nuevamente el proceso de autorización.

Para obtener el **AccessToken**, se utiliza el endpoint

```
 POST /oauth2/token
```

**Aclaración importante**, el API HR utiliza el concepto de token rotation y el mecanismo se basa en el estándar del protocolo OAuth2, por lo que, por motivos de seguridad los diferentes tokens tienen fecha de caducidad:

```
 authCode: 5 minutos
 accessToken: 1 día
 refreshToken: 2 meses
```

El **refreshToken** se actualiza cada vez que se solicita un nuevo **accessToken**. Es crucial que el mecanismo del thirdparty guarde y utilice siempre el **refreshToken** más reciente para evitar cualquier tipo de errores referidos a la autenticación.

# Autenticación mediante Bearer Token

⚠️ **IMPORTANTE**: A diferencia de otros endpoints del API HR, **todos los endpoints de videoentrevistas requieren que el token de acceso se envíe exclusivamente en el header HTTP `Authorization` con el formato Bearer Token**.

**NO se acepta el token como query parameter** en los endpoints de videoentrevistas.

## Formato correcto del header

```
Authorization: Bearer {accessToken}
```

### Ejemplo de request correcto:

```bash
curl -X GET "https://api.hiringroom.com/v0/ve/battery?resp=json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### ❌ Ejemplo INCORRECTO (no funciona para VE):

```bash
# Esto NO funcionará para endpoints de videoentrevistas
curl -X GET "https://api.hiringroom.com/v0/ve/battery?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...&resp=json"
```

## Configuración en Swagger UI

Cuando utilice la documentación Swagger para probar los endpoints de videoentrevistas:

1. Haga clic en el botón **"Authorize"** en la parte superior derecha de la interfaz Swagger
2. En el campo de autorización, ingrese su token **incluyendo el prefijo "Bearer "**:
   ```
   Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   ```
3. Haga clic en "Authorize" y luego en "Close"
4. Ahora todos los requests a endpoints de VE incluirán automáticamente el header de autorización correcto

**Nota**: Si no incluye el prefijo "Bearer " en Swagger, los requests fallarán con error 401 (Unauthorized).

## Manejo de errores de autenticación

Si el token no se envía en el header o el formato es incorrecto, recibirá una respuesta de error:

```json
{
  "error": "The request does not contain the authentication token"
}
```

Status HTTP: **401 Unauthorized**

# Configurando videoentrevistas

Para crear una videoentrevista se requiere una configuración que vincule la videoentrevista del third-party con una vacante de HiringRoom.
La misma se hace desde la creación de vacante

### Creando videoentrevista desde HiringRoom

⚠️ **IMPORTANTE** en HiringRoom tanto para Videoentrevistas como para assessments se utiliza el concepto de "baterías" ya que, en este caso, se pueden enviar tanto videoentrevistas individuales, como un conjunto de videoentrevistas. Esta lógica es pura y exclusivamente responsabilidad del third-party, siendo Hiring Room el responsable solamente de dar flexibilidad a estos casos de uso mencionados.
A partír de aquí se mencionarán "baterias de videentrevistas", pudiendo flexibilizarse al caso de uso anteriormente mencionado. 

El primer paso para que las videoentrevistas se puedan configurar desde la creación de vacantes dentro de HiringRoom es crear una batería de videoentrevistas dentro de HiringRoom para que el usuario pueda elegir la videoentrevista disponible.

La batería de videoentrevistas debería reflejar las videoentrevistas disponibles dentro del third-party, por lo que es recomendable que cada vez que se cree una videoentrevista se la envíe como batería a HiringRoom para que el usuario siempre la tenga disponible.

El endpoint correspondiente para crear una batería es

```
 PUT /ve/battery
```

indicando lo siguiente (se omiten campos opcionales):

```json
{
  "ve_id": "VE_ID",
  "ve_name": "VE_NAME",
  "internal_id": "INTERNAL_ID",
  "url_webhook": "URL_WEBHOOK",
  "url_webhook_vacancy": "URL_WEBHOOK_VACANCY"
}
```

- **VE_ID**: Identificador de la videoentrevista del third-party.
- **VE_NAME**: Nombre de la videoentrevista del third-party.
- **INTERNAL_ID**: Identificador interno de la videoentrevista del thirdparty.
- **URL_WEBHOOK**: Url del thirdparty donde enviamos los datos cuando se solicite una videoentrevista para un postulante
- **URL_WEBHOOK_VACANCY**: Url del thirdparty donde enviamos los datos cuando se cree una configuración desde la creación de vacante dentro de HiringRoom

Una vez creada la batería, obtendremos un JSON similar a este:

```json
{
  "data": [
    {
      "id": "string",
      "ve_id": "string",
      "ve_name": "string",
      "internal_id": "string",
      "url_webhook": "string",
      "url_webhook_vacancy": "string",
      "url_internal": "string",
      "url_ve": "string",
      "extra_args": [
        {
          "name": "string",
          "value": "string"
        }
      ],
      "created": 0
    }
  ]
}
```

### Editando una batería existente

Para editar una batería de videoentrevistas ya creada, utilice el endpoint:

```
 POST /ve/battery
```

**Nota**: Aunque no es el estándar REST típico, este comportamiento se mantiene por compatibilidad con el sistema de assessments existente (legacy).

Cuando tengamos baterías de videoentrevistas del third-party, las mismas aparecerán en el formulario de creación de vacante una vez seleccionado el pipeline correspondiente y en donde necesitaremos seleccionar la videoentrevista y la etapa donde se configurará.

<img width="938" height="198" alt="image" src="https://github.com/user-attachments/assets/b3ba9674-b89b-4add-9c6c-9711ac4af25d" />

Hecho esto, ya tendremos creada una configuración en la vacante indicada.

Cuando la vacante se haya creado, se enviará al **url_webhook_vacancy** configurado el siguiente JSON:

```json
{
  "config_id": "string",
  "ve_id": "string",
  "action": "string",
  "account": "string"
}
```

Donde **action** puede ser:

- **created**: Se ha creado una configuración
- **edited**: Se ha modificado una configuración
- **unlinked**: Se ha desvinculado una configuración de la vacante

# Creando videoentrevistas

Una vez que un postulante sea enviado a la etapa configurada (Por ejemplo etapa ENTREVISTA), se creará una videoentrevista pendiente para este postulante y se enviará al **url_webhook** configurado el pedido de videoentrevista para ese postulante.

<img width="1352" height="303" alt="image" src="https://github.com/user-attachments/assets/e2e21168-305d-484f-9d0b-dfdb235671a9" />


El JSON que se envía al webhook es el siguiente:

```json
{
  "postulante": {
    "id": "string",
    "nombre": "string",
    "apellido": "string",
    "genero": "M" | "F" | "O",
    "email": "string"
  },
  "webhook": {
    "config_id": "string",
    "ve_id": "string",
    "internal_id": "string",
    "vacante_id": "string",
    "extra_args": []
  },
  "account": "string"
}
```

- **config_id**: Identificador de la configuración creada.
- **ve_id**: Identificador de la videoentrevista creada en estado pendiente.
- **internal_id**: Identificador interno de la videoentrevista del thirdparty.
- **vacante_id**: Identificador de la vacante configurada para la videoentrevista.
- **extra_args**: Argumentos extras que cargó el third-party en la configuración.

## Enviando resultados de la videoentrevista

Para enviar el resultado de la videoentrevista a HiringRoom, deberemos utilizar el endpoint:

```
POST /ve/{ve_id}
```

El mismo debe tener la siguiente estructura:

```json
{
  "nombre_ve": "Videoentrevista Técnica - Desarrollador",
  "resultado_general": "El candidato demostró excelentes habilidades técnicas y comunicación clara",
  "nota_general": 85,
  "url_informe": "https://plataforma-ve.com/informes/abc123",
  "preguntas": [
    {
      "texto_pregunta": "¿Cuál es tu experiencia con JavaScript?",
      "sintesis_respuesta": "5 años de experiencia, especializado en React y Node.js",
      "evaluacion_respuesta": "Respuesta clara y concisa, demuestra conocimiento profundo",
      "url_video": "https://plataforma-ve.com/videos/pregunta1.mp4"
    }
  ]
}
```

### Campos del resultado:

- **nombre_ve** (obligatorio, string, máx. 200 caracteres): Nombre de la videoentrevista
- **resultado_general** (obligatorio, string, máx. 1000 caracteres): Resumen general del resultado
- **nota_general** (opcional, number, 0-100): Nota general de la videoentrevista
- **url_informe** (opcional, string, máx. 500 caracteres): URL del informe completo en la plataforma del third-party
- **preguntas** (obligatorio, array, mínimo 1 elemento): Array de preguntas y respuestas

### Campos de cada pregunta:

- **texto_pregunta** (obligatorio, string, máx. 255 caracteres): Texto de la pregunta realizada
- **sintesis_respuesta** (opcional, string, máx. 500 caracteres): Resumen de la respuesta del postulante
- **evaluacion_respuesta** (opcional, string, máx. 255 caracteres): Evaluación de la respuesta
- **url_video** (opcional, string, formato URI): URL del video de la respuesta

### ⚠️ Requisitos importantes:

1. **Al menos una pregunta es obligatoria**: El array `preguntas` debe contener al menos un elemento con `texto_pregunta`.
2. **URLs de video públicas**: Si se proporciona `url_video`, se recomienda enfáticamente que sea una URL pública para que el reclutador pueda acceder sin problemas de autenticación o permisos.
3. **Múltiples videoentrevistas**: Se pueden enviar múltiples resultados de videoentrevistas para el mismo `ve_id`. Cada envío agregará una nueva videoentrevista al array `interviews` del postulante.

### Ejemplo de respuesta exitosa:

```json
{
  "data": {
    "id": "string",
    "customer_account": "string",
    "vacancy_id": "string",
    "postulant_id": "string",
    "config_id": "string",
    "created": 1234567890,
    "updated": 1234567890,
    "interviews": [
      {
        "nombre_ve": "string",
        "resultado_general": "string",
        "nota_general": 85,
        "url_informe": "string",
        "preguntas": [
          {
            "texto_pregunta": "string",
            "sintesis_respuesta": "string",
            "evaluacion_respuesta": "string",
            "url_video": "string"
          }
        ],
        "completed_date": 1234567890
      }
    ]
  },
  "error": null
}
```

Las videoentrevistas completadas se mostrarán en el perfil del postulante dentro de HiringRoom.

# Manejo de errores

### Webhook de postulaciones (url_webhook)

Una vez creada la videoentrevista para el postulante, y enviada la información correspondiente al **url_webhook**, el third-party deberá responder con un código de estado HTTP apropiado y, en caso de error, un JSON estructurado que permita identificar y comunicar el problema de forma clara.

## Respuestas exitosas

Cualquier respuesta en el rango **2XX** se considera exitosa, indicando que el third-party recibió la información correctamente y la procesará según sus propios criterios.

**Ejemplo de respuesta exitosa:**

```json
{
  "success": true
}
```

## Respuestas de error

Cualquier respuesta **no exitosa** (fuera del rango 2XX) será considerada como un error y se mostrará en el perfil del postulante en la sección de videoentrevistas.

### Formato recomendado: RFC 9457 (Problem Details for HTTP APIs)

**Recomendamos enfáticamente** utilizar el estándar RFC 9457, que proporciona respuestas estructuradas, consistentes y auto-explicativas:

```json
{
  "title": "<Descripción corta y entendible del problema>",
  "status": "<código HTTP>",
  "detail": "<Descripción detallada del error>",
  "code": "<Código interno de error>"
}
```

**Campos:**

- **`title`** (obligatorio): Resumen breve del error (ej: "Error de validación")
- **`status`** (obligatorio): Código de estado HTTP que refleja el tipo de error
- **`detail`** (obligatorio): Descripción detallada que se mostrará al reclutador (debe ser autoexplicativa, corta y clara)
- **`code`** (obligatorio): Código interno de error (se encuentra en la tabla de errores)

### Importancia del estándar RFC 9457

Si bien el usuario podrá reintentar manualmente el envío de la videoentrevista ocurrido un error, la experiencia de usuario se verá afectada negativamente, ya que **no podrá identificar claramente** qué error ocurrió al enviar las videoentrevistas y deberá **comunicarse directamente con el third-party** para entender qué está pasando.

### Validación interna

⚠️ **Importante**: Si bien el email es un requisito obligatorio a la hora de postularse en Hiring Room, pueden llegar a existir postulantes sin este requisito. En caso de que un postulante no tenga email y el mismo sea enviado a una etapa configurada, el perfil mostrará un error, pero hasta que el reclutador no agregue el dato de email, no se enviará al third-party la información a través de webhook (en este caso el reclutador deberá hacerlo manualmente).

### Tabla de códigos

Para detallar el response del webhook, se recomienda seguir la siguiente tabla:

| Código de Error (`code`)        | Status HTTP | Descripción Sugerida                                             |
| :------------------------------ | :---------- | :--------------------------------------------------------------- |
| `MALFORMED_JSON`                | 400         | El cuerpo de la solicitud no es un JSON válido.                  |
| `INVALID_CREDENTIALS`           | 401         | Sesión expirada o no autorizada.                                 |
| `INSUFFICIENT_CREDITS`          | 402         | No dispone de créditos suficientes para completar la operación.  |
| `ACCOUNT_PLAN_UPGRADE_REQUIRED` | 402         | Se requiere una actualización de plan para realizar esta acción. |
| `FORBIDDEN_ACTION`              | 403         | Acción no permitida o permisos insuficientes.                    |
| `ACCOUNT_PLAN_EXPIRED`          | 403         | El plan de su cuenta ha expirado.                                |
| `ACCOUNT_PLAN_NOT_ACTIVE`       | 403         | Su cuenta no posee un plan activo o créditos suficientes.        |
| `ACCOUNT_NOT_FOUND`             | 404         | No se encontró la empresa en nuestros registros.                 |
| `CLIENT_NOT_FOUND`              | 404         | Cliente no encontrado.                                           |
| `SCALES_NOT_FOUND`              | 404         | No se encontraron escalas configuradas en el perfil.             |
| `CANDIDATE_VACANCY_MAX_REACHED` | 409         | Se ha alcanzado el máximo de candidatos para esta vacante.       |
| `PROCESS_EXPIRED`               | 410         | El proceso de envío para este candidato ha expirado.             |
| `VALIDATION_ERROR`              | 422         | Error de validación en los campos enviados.                      |
| `POSTULANT_EMAIL_MISSING`       | 422         | El postulante no posee un email válido.                          |
| `POSTULANT_PHONE_INVALID`       | 422         | El postulante no posee un teléfono válido.                       |
| `PROCESS_CREATION_ERROR`        | 500         | Error interno al intentar crear el proceso.                      |
| `SERVER_ERROR`                  | 500         | Error interno en el servidor del proveedor de videoentrevistas.  |
| `WEBHOOK_TIMEOUT`               | 504         | El webhook tardó más de 30 segundos en responder.                |
| `SERVICE_UNAVAILABLE`           | 503         | Servicio de videoentrevistas temporalmente fuera de servicio.    |

## Ejemplos de respuestas de error

**Ejemplo 1 — Créditos insuficientes (402)**

```json
{
  "title": "Créditos insuficientes",
  "status": 402,
  "detail": "No dispone de créditos suficientes para completar la operación.",
  "code": "INSUFFICIENT_CREDITS"
}
```

**Ejemplo 2 — Error de validación (422)**

```json
{
  "title": "Error de validación",
  "status": 422,
  "detail": "El campo 'apellido' es obligatorio.",
  "code": "VALIDATION_ERROR"
}
```

**Ejemplo 3 — Servicio no disponible (503)**

```json
{
  "title": "Servicio temporalmente no disponible",
  "status": 503,
  "detail": "El servicio de videoentrevistas está en mantenimiento. Intente nuevamente en 30 minutos.",
  "code": "SERVICE_UNAVAILABLE"
}
```

**Ejemplo 4 — Timeout del webhook (504)**

```json
{
  "title": "Timeout del webhook",
  "status": 504,
  "detail": "El webhook tardó más de 30 segundos en responder.",
  "code": "WEBHOOK_TIMEOUT"
}
```

⚠️ **Importante**: En caso de enviar un `code` que no se encuentre en la tabla de errores, se mostrará un error genérico en el detalle del error en el postulante.

## Implementación en código

### Ejemplo en JavaScript + Express (RFC 9457)

```javascript
async function responseWebhook(req, res, next) {
  try {
    // Lógica de negocio...
    // ...

    const errorValidacion = true; // Ejemplo

    if (errorValidacion) {
      return res.status(422).json({
        title: "Error de validación.",
        status: 422,
        detail: "El campo 'apellido' es obligatorio.",
        code: "VALIDATION_ERROR",
      });
    }

    return res.status(200).json({
      success: true,
    });
  } catch (err) {
    next(err);
  }
}
```

### Timeout de webhooks

Todos los webhooks de videoentrevistas tienen un timeout configurado de **30 segundos** por defecto. Si su webhook necesita más tiempo para procesar la solicitud, puede configurar un timeout mayor mediante la variable de entorno:

```bash
WEBHOOK_TIMEOUT=60000  # 60 segundos
```

# Eliminando videoentrevistas

Si el third-party desea eliminar una batería de videoentrevistas de su sistema o base de datos, ya sea por pedido o interacción del usuario en la plataforma third-party, o por necesidad del third-party de hacerlo, se debe informar esto al API-HR mediante el siguiente endpoint:

```
 DELETE /ve/battery/{battery_id}
```

indicando lo siguiente:

```json
{
  "description": "DESCRIPTION"
}
```

- **DESCRIPTION**: motivo por el cual se está eliminando la videoentrevista, este motivo se mostrará a los usuarios de la cuenta de HiringRoom.

Cuando se elimine una videoentrevista de la batería del third-party, la misma ya no aparecerá en el formulario de creación de vacante. Además se desvincularán de forma automática todas las vacantes relacionadas a esa videoentrevista de batería. En el caso de que la vacante tenga una configuración previamente con una videoentrevista eliminada, la misma pasará al estado "No activo" dando la posibilidad al usuario de poder realizar una nueva configuración (siempre y cuando haya stock de videoentrevistas del third-party disponibles en esa cuenta de Hiring Room).

Para los postulantes de la/s vacante/s que tengan las videoentrevistas enviadas de alguna batería eliminada, que tengan los estados:

- **En Progreso**: estas videoentrevistas se eliminan y no se podrán visualizar.
- **Completado**: estas videoentrevistas permanecerán y se podrán visualizar los resultados (si el third-party adjunta una url para visualizar el reporte correspondiente y esa url queda inaccesible, queda a discreción del third-party).

- **oldAccountName**: dominio actual.
- **newAccountName**: dominio nuevo.
- **Authorization**: Si el endpoint requiere de autorización, debe proporcionar dicho endpoint junto con las credenciales necesarias para obtenerla. Sin embargo, es importante destacar que esta acción es opcional.
