# Contenido

- [Integración third-party Evaluaciones](#integración-third-party-evaluaciones)
- [Autorización](#autorización)
- [Configurando evaluaciones](#configurando-evaluaciones)
- [Creando evaluaciones](#creando-evaluaciones)
- [Manejo de errores](#manejo-de-errores)
- [Eliminando evaluaciones](#eliminando-evaluaciones)
- [Cambio de dominio](#cambio-de-dominio)


# Integración third-party Evaluaciones

El API HR está preparado para integrarse con aplicaciones de terceros (third-party). Para ello, debe pedir al asesor comercial, de soporte o tecnología que le facilite los accesos necesarios para poder realizar la integración. Estos accesos son el **client_id** y el **client_secret**. A su vez, debe proveerle a dicho asesor la siguiente información para poder obtener los accesos correspondientes:

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

# Configurando evaluaciones

Para crear una evaluación se requiere una configuración que vincule el assessment del third-party con una vacante de HiringRoom. 
Para ello existen la siguiente manera: 

- Desde la creación de vacante

### Creando evalución desde HiringRoom

El primer paso para que las evaluaciones se puedan configurar desde la creación de vacantes dentro de HiringRoom es crear una batería de tests dentro de HiringRoom para que el usuario pueda elegir el test disponible. 

La batería de tests debería reflejar los test disponibles dentro del third-party, por lo que es recomendable que cada vez que se cree un test se lo envie como bateria a HirinRoom para que el usuario siempre lo tenga disponible. 

El endpoint correspondiente para crear una batería es

```
 PUT /assessment/battery
```

indicando lo siguiente (se omiten campos opcionales): 

```
 {
  "test_id": ID_TEST,
  "test_name": TEST_NAME,
  "internal_id": INTERNAL_ID,
  "url_webhook": URL_WEBHOOK,
  "url_webhook_vacancy": URL_WEBHOOK_VACANCY
}
```
   - **ID_TEST**: Identificador del test del third-party.
   - **TEST_NAME**: Nombre del test del third-party.
   - **INTERNAL_ID**: Identificador interno de la evaluacion del thirdparty.
   - **URL_WEBHOOK**: Url del thirdparty donde enviamos los datos cuando se pida una evaluación para un postulante
   - **URL_WEBHOOK_VACANCY**: Url del thirdparty donde enviamos los datos cuando se cree una configuración desde la creación de vacante dentro de HiringRoom

Una vez creada la batería, obtendremos un JSON similar a este:

```
{
  "data": [
    {
      "id": "string",
      "test_id": "string",
      "test_name": "string",
      "internal_id": "string",
      "url_webhook": "string",
      "url_webhook_vacancy": "string",
      "url_internal": "string",
      "extra_args": [
        {}
      ],
      "created": 0
    }
  ]
}
```

Cuando tengamos baterías de tests del third-party las mismas aparecerán en el formulario de creación de vacante una vez seleccionado el pipeline correspondiente y en donde necesitaremos seleccionar el test y la etapa donde se configurará.

![11](https://i.imgur.com/spvhjQq.png)

Hecho esto, ya tendremos creada una configuración en la vacante indicada. 

Cuando la vacante se haya creado, se enviará al **url_webhook_vacancy** configurado el siguiente JSON: 

```
{
    "config_id" : "string",
    "test_id": "string",
    "action" : "string",
    "account" : "string"
}
```

Donde **action** puede ser: 
- created : Se ha creado una configuración
- edited : Se ha modificado una configuración
- unlinked : Se ha desvinculado una configuración de la vacante


# Creando evaluaciones 

Una vez que un postulante sea enviado a la etapa configurada (Por ejemplo etapa ENTREVISTA), se creará una evaluación pendiente para este postulante y se enviará al **url_webhook** configurado el pedido de test para ese postulante. 

![9](https://i.imgur.com/KroJiDS.png)

El JSON que se envía al webhook es el siguiente:

```
{
   "postulante": {
     "id" : "string",
     "nombre": "string",
     "apellido": "string",
     "genero": "M" | "F" | "O",
     "email": "string",
     "telefono" : "string"
   },
   "webhook" : {
     "config_id" : "string",
     "assessment_id" : "string",
     "internal_id" : "string",
     "vacante_id" : "string",
     "extra_args" : []
   },
   "account" : "string"
}
```
   - **config_id**: Identificador de la configuración creada.
   - **assessment_id**: Identificador del test creado en estado pendiente.
   - **internal_id**: Identificador interno de la evaluacion del thirdparty.
   - **vacante_id**: Identificador de la vacante configurada para el test.
   - **extra_args**: Argumentos extras que cargó el third-party en la configuración.
   - **telefono**: Si el postulante cargó un número de teléfono de tipo "Celular/Movil", entonces se envía al webhook con el formato "prefijo+telefono" (ej: "+543880303456"). Si no cumple las condiciones devuelve **null**

Para enviar el resultado de la evaluación a HiringRoom, deberemos utilizar el endpoint **POST /assessment/{assessment_id}**. El mismo debe tener la siguiente estructura:

```
{
  "nombre_test": "string",
  "resultado_general": "aprobado",
  "nota_general": 10,
  "detalle_general": [
    {
      "title": "Competencias y Liderazgo",
      "details": [
        {
          "title": "string",
          "value": "string",
          "type": "STRING"
        }
      ]
    }
  ],
  "descripcion": "string",
  "url_reporte": "string"
}
```

Cabe aclarar que se pueden enviar múltiples evaluaciones mediante el endpoint anterior. Las mismas se mostrarán en el perfil del postulante de la siguiente manera:

![10](https://i.imgur.com/fX4rB78.png)

# Manejo de errores

### Webhook de postulaciones (url_webhook)

Una vez creada la evaluación para el postulante, y enviada la información correspondiente al **url_webhook**, Todo third-party que reciba un webhook deberá responder un JSON siguiendo RFC 9457 (Problem Details for HTTP APIs) siendo los mismos estructurados, consistentes y auto-explicativos.

Cualquier respuesta en el rango del 2XX se considera exitosa, es decir que el third-party recibió la información y que la misma será procesada según sus propios criterios.

```
{
  "success": true
}
```

Cualquier respuesta no exitosa, será considerada como un error que se mostrará en el perfil del postulante en la respectiva sección de assessments:

<img width="1350" height="108" alt="image" src="https://github.com/user-attachments/assets/13652a95-29bc-45d3-96a7-46d622f4a24f" />

Importante: Si bien el email es un requisito obligatorio a la hora de postularse en Hiring Room, pueden llegar a existir postulantes sin este requisito. En caso de que un postulante no tenga email y el mismo sea enviado a una etapa configurada, el perfil mostrará un error, pero hasta que el reclutador no agregue el dato de email, no se enviará al third-party la información a traves de webhook (en este caso el reclutador deberá hacerlo manualmente). 


<img width="396" height="133" alt="image" src="https://github.com/user-attachments/assets/6433ffb2-51fe-4970-ab2f-9c962776c8f4" />

Si desde el lado del third-party hubo un error de negocio o validación, vamos a requerir para el estándar la siguiente información como respuesta:

```
{
  "title": "<Descripción corta y entendible del problema>",
  "status": <código HTTP>,
  "detail": "<Descripción detallada del error>",
  "code": "<Código interno de error.>",
}
```
Para detallar el response del webhook, se recomienda seguir la siguiente tabla:

<img width="619" height="280" alt="image" src="https://github.com/user-attachments/assets/546078b5-08b6-436f-a771-c98cbba43f78" />

Ejemplos de uso:

⚠️ Ejemplo — Créditos insuficientes

```
{
  "title": "No tiene créditos disponibles.",
  "status": 402,
  "detail": "No dispone de créditos suficientes para completar la operación.",
  "code": "INSUFFICIENT_CREDITS"
}
```
⚠️ Ejemplo — Error de validación (422)
```
{
  "title": "Error de validación.",
  "status": 422,
  "detail": "El campo 'apellido' es obligatorio.",
  "code": "VALIDATION_ERROR"
}
```
Ejemplo en JS + express

```
async function responseWebhook(req, res, next) {
  try {
    // Lógica de negocio...
    // ...

    const errorValidacion = true; // Ejemplo

    if (errorValidacion) {
      return res.status(402).json({
        title: "Créditos insuficientes.",
        status: 402,
        detail: "No dispone de créditos suficientes para completar la operación.",
        code: "INSUFFICIENT_CREDITS"
      });
    }

    return res.status(200).json({
      message: "OK"
    });

  } catch (err) {
    next(err);
  }
}
```

# Eliminando evaluaciones 

Si el third-party desea eliminar una batería de su sistema o base de datos, ya sea por pedido o interacción del usuario en la plataforma third-party, o por necesidad del third-party de hacerlo, se debe informar esto al API-HR mediante el siguiente endpoint: 

```
 DELETE /assessment/battery/{battery_id}
```
indicando lo siguiente : 

```
 {
  "description": DESCRIPTION,
 }
```
- **DESCRIPTION**: motivo por el cual se esta eliminando el test, este motivo se mostrará a los usuarios de la cuenta de HiringRoom.

Cuando se elimine un test de la batería del third-party las misma ya no aparecerán en el formulario de creación de vacante. Además se desvincularán de forma automática todas las vacantes relacionadas a ese test de batería. En el caso de que la vacante tenga una configuración previamente con un test eliminado, el mismo pasara al estado  "No activo" dando la posibilidad al usuario de poder realizar una nueva configuración (siempre y cuando haya stock de tests del third-party disponibles en esa cuenta de Hiring Room).

Para los postulantes de la/s vacante/s que tengan las evaluaciones enviadas de algún test eliminado, que tengan los estados:
 - **En Progreso**: estas evaluaciones se eliminan y no se podrán visualizar.
 - **Completado**: estas evaluaciones permanecerán y se podrán visualizar los resultados (si el third-party adjunta una url para visualizar el reporte correspondiente y esa url queda inaccesible, queda a discreción del third-party).


# Cambio de dominio
Dentro de la plataforma de Hiringroon, se realizan cambios de dominio. Este dominio se utiliza para la autorización en el endpoint **/oauth2/authorization**. Si en los procesos internos de su plataforma se guarda este campo, se debe proporcionar un endpoint para enviarles el nuevo dominio al que se ha cambiado la cuenta. Los parámetros necesarios para este endpoint son los siguientes:

```
curl -X 'PATCH' 
  '{UrlEndpoint}?oldAccountName=oldaccount&newAccountName=newAccount' 
  -H 'Authorization: Bearer {accessToken}'
```
  - **UrlEndpoint**: url del endpoint.
   - **oldAccountName**: dominio actual.
   - **newAccountName**: dominio nuevo .
   - **Authorization**: Si el endpoint requiere de autorización,debe proporcionar dicho endpoint junto con las credenciales necesarias para obtenerla. Sin embargo, es importante destacar que esta acción es opcional.







