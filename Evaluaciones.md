# Contenido

- [Integración third-party](#integración-third-party)
- [Creando evaluaciones](#creando-evaluaciones)


# Integración third-party

El API HR está preparado para integrarse con aplicaciones de terceros (third-party). Para ello, debe pedir al asesor comercial, de soporte o tecnología que le facilite los accesos necesarios para poder realizar la integración. Estos accesos son el **client_id** y el **client_secret**. A su vez, debe proveerle a dicho asesor la siguiente información para poder obtener los accesos correspondientes:

   - **fullname** : Nombre completo de la aplicación third-party.
   - **description** : Descripción completa de lo que hace la aplicación third-party.
   - **description_link** (opcional) : URL donde se describa o se redirija a la aplicación third-party.
   - **url_pic** : URL del logo del third-party.
   - **redirect_uri**: URL de redirección en donde el third-party obtendrá el codigo de autorización una vez que el cliente autorice la aplicación.

El protocolo de autenticación que usa el API HR para integraciones es el protocolo OAuth2. 

### Obteniendo el token de autorización 

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
El codigo de autorización sólo dura unos segundos, así que si no se pide el **AccessToken** correspondiente lo antes posible, el codigo de autorización quedará inválido. 

Para obtener el **AccessToken**, se utiliza el endpoint

```
 POST /oauth2/token
```

# Creando evaluaciones

Para crear una evaluación, el primer paso es realizar una configuración en la cual se indica en qué vacante se desea obtener las evaluaciones correspondientes y también en qué etapa de la misma.

El endpoint que deberemos utilizar es: 

```
 PUT /assessment/config
```

indicando lo siguiente (se omiten campos opcionales): 

```
 {
  "vacancy_id": ID_VACANTE,
  "stage": ID_ETAPA,
  "internal_id": INTERNAL_ID,
  "url_webhook": URL_WEBHOOK
}
```

   - **ID_VACANTE**: Identificador de la vacante de HR. Lo podemos obtener del endpoint **GET /vacancies**
   - **ID_ETAPA**: Etapa de la vacante en la que se quiere configurar el test. Lo podemos obtener del endpoint **GET /pipeline/{pipelineId})**
   - **INTERNAL_ID**: Identificador interno de la evaluacion del thirdparty.
   - **URL_WEBHOOK**: Url del thirdparty donde enviamos los datos cuando se pida una evaluación para un postulante

Una vez creada la configuración, obtendremos un JSON similar a este:

```
{
  "data": [
    {
      "config_id": "string",
      "vacancy_id": "string",
      "stage": 0,
      "internal_id": "string",
      "url_webhook": "string",
      "url_internal": "string",
      "extra_args": [
        {}
      ],
      "created": 0
    }
  ]
}
```

Hecho esto, ya tendremos creada una configuración en la vacante indicada. 

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
   },
   "webhook" : {
     "config_id" : "string",
     "assessment_id" : "string",
     "internal_id" : "string",
     "vacante_id" : "string",
     "extra_args" : []
   }
}
```
   - **config_id**: Identificador de la configuración creada.
   - **assessment_id**: Identificador del test creado en estado pendiente.
   - **internal_id**: Identificador interno de la evaluacion del thirdparty.
   - **vacante_id**: Identificador de la vacante configurada para el test.
   - **extra_args**: Argumentos extras que cargó el third-party en la configuración.

### Crear la evaluación 

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




