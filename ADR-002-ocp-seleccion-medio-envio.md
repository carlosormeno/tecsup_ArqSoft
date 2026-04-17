# ADR-002 — Separar la seleccion del medio de envio de comunicados

**Fecha:** 2026-04-17 
**Estado:** ✅ Aceptado
**Principio SOLID:** O — Open/Closed Principle (OCP)

---

## Contexto

Cuando un usuario administrativo termina de matricular a un alumno, el sistema debe automaticamente enviar al padre de familia un correo, con los datos de la matricula y la fecha de inicio de clases. El evento que llama a envio de mensajes se encuentra dentro de `EnviarMensajes` con una serie de opciones que no permiten el crecimiento o ampliacion sin riesgo:

**Código actual (con el problema):**

```java
// PrestamoService.java — lógica de multas acoplada con condicionales
package v1v.zona_fit.servicio;

import v1v.zona_fit.modelo.Ticbmessage;

import java.util.List;

public class EnviarMensajes implements IEnviarMensajes {

    public String medio = "";

    @Override
    public List<Ticbmessage> listarMensajes() {
        return null;
    }

    @Override
    public Ticbmessage buscarMensajesPorId(Integer idmessage) {
        return null;
    }

    @Override
    public void enviarmensaje(String medio) {
        switch (medio){
            case "_EMAIL" :
                //ejecutarEnvio "Se envio email";
            case "_WHATSAPP" :
                //ejecutarEnvio "Se envio whatsapp";
        }
        
    }
}

```

**¿Cuál es el problema?**

Cada vez que se añade un nuevo tipo de medio de envio (por ejemplo, `_SMS_`), hay que **modificar** `EnviarMensajes`. Esto:
- Rompe código que ya funciona.
- Obliga a revisar y actualizar todas las pruebas existentes del método.

---

## Decisión

Introducimos la interfaz `IEnviarMensajes` con un método `enviarmensaje()`. De acuerdo al medio de envio se utilizara la clase correspondiente de acuerdo almedio.

**Código corregido:**

```java
// EnviarMensajesEmail.java — interfaz (contrato cerrado a modificación)
package v1v.zona_fit.servicio;
import v1v.zona_fit.modelo.Ticbmessage;
import java.util.List;

public abstract class EnviarMensajesEmail implements IEnviarMensajes{

    public String medio = "_EMAIL";

    @Override
    public void enviarmensaje(String medio) {
        // Generar pedido al servidor de correo
    }
}


// EnviarMensajesWhatsapp.java
package v1v.zona_fit.servicio;
import v1v.zona_fit.modelo.Ticbmessage;
import java.util.List;

public class EnviarMensajesWhatsapp implements IEnviarMensajes{

    public String medio = "_WHATSAPP";

    @Override
    public List<Ticbmessage> listarMensajes() {return null;}

    @Override
    public Ticbmessage buscarMensajesPorId(Integer idmessage) {return null;}

    @Override
    public void enviarmensaje(String medio) {
        // Generar pedido al servidor de mensajer[ia wa
    }
}


// EnviarMensajes.java
package v1v.zona_fit.servicio;
import v1v.zona_fit.modelo.Ticbmessage;
import java.util.List;

public class EnviarMensajes implements IEnviarMensajes {

    public String medio = "";

    @Override
    public List<Ticbmessage> listarMensajes() {
        return null;
    }

    @Override
    public Ticbmessage buscarMensajesPorId(Integer idmessage) {
        return null;
    }

    @Override
    public void enviarmensaje(String medio) {
        return null;        
    }
}


```

**¿Cómo se usa en conjunto?**

### Principio SOLID aplicado — OCP

> "Las entidades de software deben estar abiertas para extensión y cerradas para modificación."

**Antes:** En `EnviarMensajes` → modificar `enviarmensajes` (riesgo de error).  
**Después:**  Creamos nuevas clases para enviar los mensajes por cada medio existente.
`EnviarMensajesEmail` y `EnviarMensajesWhatsapp`.

```

**¿Qué está "cerrado"?** La clase `EnviarMensajes` y el método `enviarmensaje`.  
**¿Qué está "abierto"?** Una clase creada por cada medio de pago.

---

## Consecuencias

### Positivas
- Cada medio de envio tiene su propia prueba unitaria independiente.
- Añadir una nueva clase no requiere tocar ni revisar las clases existentes.

### Negativas / trade-offs
- Se crean varias clases pequeñas. En sistemas con muchos medio de pago puede parecer excesivo; valorar si una tabla de configuración en base de datos sería suficiente.
- `MultaFactory` sigue siendo un punto de modificación cuando se añade un tipo nuevo. Aceptable: el cambio es de una sola línea en el `switch`.