# Contenido

- [Introducción](#introducción)
- [Client ID y Client secret](#client-id-y-client-secret)
- [Interfaz swagger](#interfaz-swagger)
- [Inicio de Sesión](#inicio-de-sesión)
- [Creación de una vacante](#creación-de-una-vacante)
- [Manejo de errores](#manejo-de-errores)

# Introducción

Para empezar,aparte de sus credenciales para acceder a HiringRoom (username y password) el cliente debe obtener sus credenciales para el API, su **client_id** y su **client_secret** que puede obtenerlos desde HiringRoom.

El **API-HR** posee una interfaz swagger para poder ver y probar los endpoints disponibles , la misma se encuentra en https://api.hiringroom.com/

### Client ID y Client secret

El **client_id** y el **client_secret** son datos fundamentales a la hora de usar el API de Hiringroom (a partir de ahora API-HR). Si ellos no es posible crear el **access_token** requerido para usar el API-HR. 
Para poder obtenerlos, debe iniciar sesión con su cuenta de HiringRoom y nos dirigimos a **Configuracion->Preferencias del sistema**

![6](https://i.imgur.com/YW3yMMD.png)

Una vez dentro de las preferencias del sistema nos dirigimos a **Integración API HR** y hacemos click en **Solicitar credenciales**. Obtendremos un Json con las credenciales

![7](https://i.imgur.com/5KkYhUU.png)

```
{
	"client_id" : "[CLIENT_ID]"
	"client_secret" : "[CLIENT_SECRET]"
}
```

Nota: Una vez obtenidas las credenciales deben ser almacenadas en algún lugar de confianza. Una vez fuera de esa vista, o si se refresca la pagina, las credenciales no serán visibles nuevamente, debiendo repetir el proceso para adquirirlas nuevamente. _**Se debe tener en cuenta que si se adquieren nuevas credenciales, las anteriores quedaran inutilizables**_. 


### Interfaz swagger

Una pequeña explicación de la interfaz para entender como funciona
- Estructura basica

![1](https://i.imgur.com/wdbatUn.png)
- Ejemplo modelo de response code 200

![2](https://i.imgur.com/lPT0RvB.png)
- Ejemplo de response de algun endpoint

![3](https://i.imgur.com/AvX1MxV.png)

## Inicio de Sesión

Para acceder a los recursos privados que provee el API de Hiringroom (a partir de ahora API-HR) es necesario autenticarse iniciando sesión en la misma. El API-HR soporta dos tipos de sesión: a nivel de usuario y a nivel aplicación de terceros.

### Inicio de sesión como usuario

1. Desplegamos el apartado **autenticacion-usuarios**, el endpoint a usar es **POST /authenticate/login/users**

2. Se completa el body con la información requerida: 

```
{
  "grand_type": "password",
  "client_id": "[CLIENT_ID]",
  "client_secret": "[CLIENT_SECRET]",
  "username": "[EMAIL]",
  "password": "[PASSWORD]"
}
```
![4](https://i.imgur.com/t4AFkeL.png)

3. Y haciendo click en el botón **Try it out!** y obtendremos el resultado del request

4. Obtendremos un json con este formato: 

```
    {
     "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IktaXNlbnRyZXZpc3RhcyIsInNjcCI6WyJ1c2VyczpyZWFkIiwicG9zdHVsYW50czpyZWFkIiwicG9zdHVsYW50czp3cml0ZSIsInZhY2FuY2llczpyZWFkIiwidmFjYW5Njb3VudF9sb2NhbGl0aWVzOnJlYWQiLCJhY2NvdW50X2xvZ29zOnJlYWQiLCJhY2NvdW50X3N0YWdlczpyZWFkIiwiY29tdW5lczpyZWFkIiwiYWNjb3V",
     "expiresIn": 86370,
     "tokenType": "bearer",
     "refreshToken": "zAKGpl0ETvrMZyy57C0Ty2Qn3iaQiG4WoSq8W8lkJpj34Et45BIbQzbwWIgRkvXF0jpcQGyrfYRSjavcw0XC81EBLP2zy79qSdHBKAiLtT8HtF4ucSl85nN2Cn7HoHd9RSqCEG0TFTKd85ZMxU1GftwDJzewNsccGciSXDINOI9Mx5W5Lf6zZmNAffKRGqYbo939Xi1pC"
    }
```
   - **token**: es el access_token (Token de Acceso) necesario para poder hacer las consultas en el API.
   - **expiresIn**: fecha de expiracion del token expresada en segundos
   - **tokenType**: tipo de token
   - **refreshToken**: token de refresco

5. Una vez obtenido el token podemos setearlo en el input superior del swagger y haciendo click en el botón **Set Token**: 

![5](https://i.imgur.com/vPnVHRs.png)

### Uso de refresh token

Para la seguridad de los usuarios creamos este endpoint para que puedan obtener un nuevo access_token una vez vencido el actual. De esta forma, el usuario no necesita ingresar siempre al endpoint de login con sus accesos (user, password, client_id, client_secret) para obtener un nuevo access_token sino que, ingresando al endpoint de refresh_token puede obtenerlos de una manera mas facil y segura. Solo es necesario el **refresh_token** obtenido en el login y el **client_id**.

El Refresh token tiene una duracion de 6 meses, y puede ser usado una sola vez, luego de eso, se obtiene un nuevo refresh_token

Uso del refresh token: 

1. Desplegamos el apartado **autenticacion-usuarios**, el endpoint a usar es **POST /authenticate/login/refresh_token**

2. El body lo completamos con la informacion requerida: 

```
{
  "grand_type": "refresh_token",
  "client_id": "string",
  "refresh_token": "string"
}
```
- **grand_type** : en este endpoint siempre es **refresh_token**
- **client_id** : el **client_id** que solicita el token
- **refresh_token** : el **refresh_token** obtenido en el login. 

3. Nuevamente, obtendremos un Json como el que obtenemos al hacer un login, con un nuevo access_token y un nuevo refresh_token

```
    {
     "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IktaXNlbnRyZXZpc3RhcyIsInNjcCI6WyJ1c2VyczpyZWFkIiwicG9zdHVsYW50czpyZWFkIiwicG9zdHVsYW50czp3cml0ZSIsInZhY2FuY2llczpyZWFkIiwidmFjYW5Njb3VudF9sb2NhbGl0aWVzOnJlYWQiLCJhY2NvdW50X2xvZ29zOnJlYWQiLCJhY2NvdW50X3N0YWdlczpyZWFkIiwiY29tdW5lczpyZWFkIiwiYWNjb3V",
     "expiresIn": 86370,
     "tokenType": "bearer",
     "refreshToken": "zAKGpl0ETvrMZyy57C0Ty2Qn3iaQiG4WoSq8W8lkJpj34Et45BIbQzbwWIgRkvXF0jpcQGyrfYRSjavcw0XC81EBLP2zy79qSdHBKAiLtT8HtF4ucSl85nN2Cn7HoHd9RSqCEG0TFTKd85ZMxU1GftwDJzewNsccGciSXDINOI9Mx5W5Lf6zZmNAffKRGqYbo939Xi1pC"
    }
```

## Creación de una vacante

Aquí se ejemplifica la creación de una vacante, explicando los pasos involucrados necesarios.

### Introducción

Desplegamos el apartado **vacantes**, el endpoint a usar es **PUT /vacancies**

 Antes de poder completar el json para crear la vacante debemos tener algunas cosas en cuenta: 

- En el modelo del json de vacante tenemos todos los campos obligatorios de la misma
- Hay campos que requieren consultar otros endpoints para poder completarlos (por ejemplo **ubicacionId**). En el modelo estan especificados los endpoints necesarios para cada campo.
- HiringRoom maneja 2 tipos de cuentas: **consultoras** y **empresas**, y es necesario que el usuario del API-HR tenga en cuenta esto a la hora de agregar ciertos parámetros (como área o cliente)

### Campos que consultan otros endpoints

* ubicacionId
* logoId
* area/cliente 
* subarea/subcliente
* areaTrabajo 
* subareaTrabajo 
* jerarquia 
* educacionMinima 
* estadoEducacion 
* tipoEmpleo 
* razonBusqueda 

Nota: Puede que haya campos específicos para cada tipo de cuenta (al hacer el request obtendrá un error de validación indicándolo), por ejemplo area/subarea son parámetros para cuentas tipo **empresa** mientras que cliente/subcliente son para cuentas tipo **consultora**.

### Consultando otros endpoints

Para completar ,por ejemplo, el campo requerido **ubicacionId** es necesario consultar el endpoint **GET /account/localities**
El único campo que nos pide este endpoint es el access_token obtenido en el [login](#inicio-de-sesión). Ejecutamos el request y obtendremos un Json con el siguiente formato: 

```
{
  "localidades": {
    "fechaCreacion": "12-05-2020",
    "pais": "Argentina",
    "provincia": "Jujuy",
    "ciudad": "San Salvador de Jujuy",
    "id": "5ebb0053f40cf01b74f1b274"
  }
}
```
En este caso, el campo que necesitamos para completar **ubicacionId** es **id**.

Nota: Si no se obtienen localidades quiere decir que la cuenta no posee ninguna creada, puede crear una desde HiringRoom al crear/editar una vacante o desde el endpoint **PUT /account/localities**

### Preguntas simples

Las preguntas simples son aquellas que tienen una respuesta abierta por parte del postulante.  
Para adjuntar una pregunta simple en el Json de vacante basta con añadirle el campo **preguntasSimples** que es un array de objetos: 

```
"preguntasSimples": [
    {
      "pregunta": "string"
    }
  ]
``` 

Por lo que, el Json de ejemplo nos podría quedar de este modo, agregándole 2 preguntas simples: 

```
{
 "preguntasSimples": [
    {
      "pregunta": "Experiencia con Api REST"
    },
    {
      "pregunta": "Experiencia con Swagger"
    }
  ]
}
```

### Preguntas múltiples

Las preguntas múltiples son aquellas que tienen opciones predefinidas, se pueden marcar las respuestas incorrectas para que los postulantes puedan ser automáticamente rechazados al contestarlas.

Para adjuntar una pregunta múltiple en el Json de vacante basta con añadirle el campo **preguntasMultiples** que es un array de objetos, pero con una estructura particular: 

```
"preguntasMultiples": [
    {
      "pregunta": "string",
      "opciones": [
        {
          "opcion": "string",
          "descalifica": true
        }
      ]
    }
  ]
``` 

* **pregunta** es el texto de la pregunta
* **opciones** array de opciones para esa pregunta, las opciones pueden descalificar o no un postulante (mínimo debe haber una opción que no descalifique, ademas no es obligatorio. El mínimo de opciones a agregar son 2). 

Por lo que, el Json de ejemplo nos podría quedar de este modo, agregándole 2 preguntas múltiples : 

```
{
  "preguntasMultiples": [
    {
      "pregunta": "Sabe usar swagger?",
      "opciones": [
        {
          "opcion": "Si",
          "descalifica": false
        },
        {
          "opcion": "No",
          "descalifica": true
        }
      ]
    },
    {
      "pregunta": "Entiende la documentacion sobre el API?",
      "opciones": [
        {
          "opcion": "Mucho",
          "descalifica": false
        },
        {
          "opcion": "Poco",
          "descalifica": false
        },
        {
          "opcion": "Nada",
          "descalifica": false
        }
      ]
    }
  ] 
}
```

### Crear la vacante

Una vez obtenidos todos los campos necesarios para crear una vacante, podemos tener un Json similar a este (se omitieron algunos campos que no son obligatorios) 

```
{
  "nombre": "Nueva vacante desde el API",
  "ubicacionId": "5ebb0053f40cf01b74f1b274",
  "area": "5c813949377caf355c95bc2e",
  "subarea": "5e66aa546e04883414002a8d",
  "descripcionPuesto": "Este puesto es un ejemplo de creacion de vacante desde el API",
  "areaTrabajo": 19,
  "subareaTrabajo": 2621,
  "jerarquia": 1,
  "tags": [
    "test", "api"
  ],
  "razonBusqueda": 1,
  "tipoBusqueda": "externa",
  "preguntasSimples": [
    {
      "pregunta": "Experiencia con Api REST"
    },
    {
      "pregunta": "Experiencia con Swagger"
    }
  ],
  "preguntasMultiples": [
    {
      "pregunta": "Sabe usar swagger?",
      "opciones": [
        {
          "opcion": "Si",
          "descalifica": false
        },
        {
          "opcion": "No",
          "descalifica": true
        }
      ]
    },
    {
      "pregunta": "Entiende la documentacion sobre el API?",
      "opciones": [
        {
          "opcion": "Mucho",
          "descalifica": false
        },
        {
          "opcion": "Poco",
          "descalifica": false
        },
        {
          "opcion": "Nada",
          "descalifica": false
        }
      ]
    }
  ] 
}
```

Haciendo el request de este Json podremos obtener (por ejemplo) un json con un error de validación (code 422)

```
{
  "message": "Validation Error",
  "errors": [
    {
      "field": "nombre",
      "message": "Param 'nombre' is required"
    }
  ]
}
```

O el request puede haberse hecho satisfactoriamente (code 201) obteniendo un Json similar a este: 

```
{
  "result": {
    "result": "success",
    "id": "5ebc047012b4d57678385ef8"
  }
}
```

Nota: el id que obtenemos el es Id de la nueva vacante creada, que confirma que fue creada satisfactoriamente.

Si se visita el dashoboard de la cuenta de HiringRoom se puede observar que efectivamente la vacante esta creada. 

### Manejo de errores

La lista de status codes que maneja el HR-API son

* **200** Successfull (Operación exitosa)
* **201** Successfull (creacion exitosa de algún recurso en el API)
* **202** Successfull, no data (Operación exitosa sin resultado)
* **400** Bad Request (Algun dato proveido es invalido)
* **401** Authentication Error (Error de autenticación)
* **403** Forbidden Error (No se permite acceder a ese recurso)
* **404** Not Found Error (Recurso no encontrado)
* **422** Validation Error (Error de validación)
* **500** Internal Server Error (Error interno)

La estructura que devuelve el API-HR para los codes 200, 201 se especifica en cada endpoint en el swagger

La estructura que devuelve el API-HR para los codes 202 es la siguiente: 

```
{
    "result": {
        "message": "There are no results for this search"
    }
}
```

La estructura que devuelve el API-HR para los codes 400, 401, 403 y 404 es la siguiente: 

```
{
  "message": "string",
  "errors": {
    "message": "string"
  }
}
```
La estructura que devuelve el API-HR para el code 422 es el siguiente: 

```
{
  "message": "Validation Error",
  "errors": [
    {
      "field": "string",
      "message": "string"
    }
  ]
}
```

