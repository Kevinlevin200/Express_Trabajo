# Uber 2: Más Conductores Que Nunca 🏎️🚗🚗

## Autor: Kevin Santiago Rivero Rueda
## Proyecto: Plataforma de transporte tipo Uber

---

## Reflexiones antes de diseñar

1. **¿Qué es un endpoint en una API?**  
   Es básicamente una dirección dentro de la API que permite acceder a una función específica, como pedir un viaje o ver tu historial de pagos.

2. **¿Diferencia entre endpoint público y privado?**  
   El público lo puede usar cualquiera (como registrarse), mientras que el privado necesita autenticación (como ver tus viajes).

3. **¿Qué datos de usuario son confidenciales?**  
   Contraseñas, ubicación en tiempo real, datos de pago, número de teléfono… todo lo que pueda comprometer la privacidad.

4. **¿Por qué es clave definir bien los métodos HTTP?**  
   Porque cada uno tiene su propósito: GET para consultar, POST para crear, PUT/PATCH para actualizar, DELETE para eliminar. Usarlos bien evita confusiones y errores.

5. **¿Qué información requiere autenticación?**  
   Todo lo que sea personal o sensible: viajes, pagos, calificaciones, ubicación, etc.

6. **¿Cómo proteger la ubicación de conductores y pasajeros?**  
   Solo compartirla durante el viaje, cifrarla, y usar tokens para controlar quién accede.

7. **¿Qué pasa si no hay conductores disponibles?**  
   La API debería responder con algo como “No hay conductores disponibles en este momento. Intenta más tarde.”, idealmente con un código 200 o 404.

8. **¿Cómo identificar los recursos principales?**  
   Pensando en los objetos clave del sistema: usuarios, viajes, pagos, calificaciones, vehículos…

9. **¿Ventajas de versionar la API desde el inicio?**  
   Evita dolores de cabeza en el futuro. Puedes hacer cambios sin romper lo que ya funciona.

10. **¿Por qué documentar errores también?**  
   Porque los errores pasan, y es mejor saber cómo manejarlos que quedarse en blanco.

---

## Roles y lo que pueden hacer

- **Pasajero:** pedir viajes, pagar, calificar conductores, ver su historial.
- **Conductor:** aceptar viajes, cambiar estados, recibir pagos, calificar pasajeros.
- **Administrador:** gestionar usuarios, monitorear viajes, revisar pagos, bloquear cuentas si es necesario.



## Recursos principales

- users
- rides
- payments
- ratings
- vehicles
- auth
- admin

---

## Endpoints (mínimo 20)

| Método | Ruta                         | Qué hace                                 | Parámetros                          | Autenticación |
|--------|------------------------------|------------------------------------------|-------------------------------------|----------------|
| POST   | /v1/auth/register            | Registro de usuario                      | nombre, email, rol, password        | No             |
| POST   | /v1/auth/login               | Inicio de sesión                         | email, password                     | No             |
| GET    | /v1/users/me                 | Ver tu perfil                            | token                               | Sí             |
| PUT    | /v1/users/me                 | Actualizar tu perfil                     | nombre, teléfono                    | Sí             |
| GET    | /v1/users/:id                | Ver perfil público limitado              | id                                  | Sí             |
| POST   | /v1/rides/request            | Pedir un viaje                           | origen, destino                     | Pasajero       |
| GET    | /v1/rides/available          | Ver viajes disponibles                   | ubicación                           | Conductor      |
| POST   | /v1/rides/:id/accept         | Aceptar viaje                            | id                                  | Conductor      |
| PATCH  | /v1/rides/:id/status         | Cambiar estado del viaje                 | status                              | Conductor      |
| POST   | /v1/rides/:id/cancel         | Cancelar viaje                           | id                                  | Pasajero/Conductor |
| POST   | /v1/rides/:id/complete       | Finalizar viaje                          | id                                  | Conductor      |
| GET    | /v1/rides/history            | Ver historial de viajes                  | rol                                 | Sí             |
| POST   | /v1/payments/:rideId         | Pagar un viaje                           | rideId, método                      | Pasajero       |
| GET    | /v1/payments/history         | Ver pagos anteriores                     | fecha                               | Sí             |
| GET    | /v1/payments/:id/receipt     | Ver recibo de pago                       | id                                  | Sí             |
| POST   | /v1/ratings/:rideId          | Calificar viaje                          | score, comentario                   | Sí             |
| GET    | /v1/ratings/user/:id         | Ver calificaciones de alguien            | id                                  | Sí             |
| GET    | /v1/vehicles/me              | Ver tu vehículo                          | token                               | Conductor      |
| PUT    | /v1/vehicles/me              | Actualizar datos del vehículo            | modelo, placa                       | Conductor      |
| GET    | /v1/admin/users              | Ver lista de usuarios                    | rol                                 | Admin          |
| PATCH  | /v1/admin/users/:id/block    | Bloquear usuario                         | id                                  | Admin          |

---

## Flujos de uso

### 1. Pedir y completar un viaje
- El pasajero pide -> `/rides/request`
- El conductor ve y acepta -> `/rides/available` + `/rides/:id/accept`
- Cambia estado -> `/rides/:id/status`
- Finaliza -> `/rides/:id/complete`
- El pasajero paga -> `/payments/:rideId`

### 2. Cancelación y devolución parcial
- El pasajero cancela -> `/rides/:id/cancel`
- La API calcula penalización
- Se devuelve parte del pago -> `/payments/:rideId`

### 3. Calificación mutua
- Pasajero califica -> `/ratings/:rideId`
- Conductor también -> `/ratings/:rideId`
- Se pueden consultar -> `/ratings/user/:id`

---

## Decisiones de diseño

- Usamos `/v1/` desde el inicio para evitar líos en el futuro.
- Seguridad con JWT y roles bien definidos.
- Ubicación solo visible durante el viaje.
- Errores bien documentados para que nadie se quede en el aire.

---

## Manejo de errores

**Formato de error:**
```json
{
  "error": true,
  "code": 403,
  "message": "Acceso denegado"
}
```

| Código | Cuándo pasa                     | Ejemplo de mensaje              |
|--------|----------------------------------|---------------------------------|
| 400    | Datos mal enviados               | "Faltan campos obligatorios"    |
| 401    | No estás autenticado             | "Token no proporcionado"        |
| 403    | No tienes permiso                | "Acceso denegado para este rol" |
| 404    | No se encontró lo que buscabas   | "Viaje no disponible"           |
| 500    | Algo se rompió en el servidor    | "Ocurrió un error inesperado"   |

---

## Ideas para mejorar

- Agregar más métodos de pago (efectivo, billeteras digitales).
- Notificaciones en tiempo real.
- Integración con mapas.
- Recompensas por buena conducta.
- Panel de estadísticas para conductores y admins.