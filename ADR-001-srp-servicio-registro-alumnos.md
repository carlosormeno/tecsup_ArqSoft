# ADR-001 — Separar el envío de mensajes y registro de alumnos

**Fecha:** 2026-04-17
**Estado:** ✅ Aceptado
**Principio SOLID:** S — Single Responsibility Principle (SRP)

---

## Contexto

La interface `ISgeAlumnosServicio` es responsable del mantenimiento, busqueda y visualizacion de Empresas, además, de enviar correos electrónicos al usuarios y rergistrar la trazabilidad de los procesos.

**Código actual (con el problema):**

```java
// IEmpresaServicio.java — tiene más de una responsabilidad
package v1v.zona_fit.servicio;

import v1v.zona_fit.modelo.SgeAlumnos;
import java.util.List;

public interface ISgeAlumnosServicio {

    public List<SgeAlumnos> listarAlumnos();

    public SgeAlumnos buscarAlumnoPorId(Integer idAlumno);

    public void guardarAlumno(SgeAlumnos alumno);

    public void eliminarAlumno(SgeAlumnos alumno);

    void enviarMensajes();

}
```

**¿Cuál es el problema?**

`ISgeAlumnosServicio` tiene **una razon para cambiar**:
- Esta incluyendo el metodo para ejecutar el envio de notificaciopnes a traves de distintos medios.

Esto también dificulta las pruebas: para testear `ReclamoServicio` hay que configurar un servidor SMTP real o usar mocks complejos, ya que incluye el metodo de envio de correos.

---

## Decisión

Extraemos el envío de notificaciones a una interfaz dedicada `IEnviarMensajes`. A partir de esta interfaz podemos crear clases que solo hagan esta funcionalidad.


**Código corregido:**

```java
// IEmpresaServicio.java — responsabilidad única:Mantenimiento de Empresas
package v1v.zona_fit.servicio;

import v1v.zona_fit.modelo.SgeAlumnos;
import java.util.List;

public interface ISgeAlumnosServicio {

    public List<SgeAlumnos> listarAlumnos();

    public SgeAlumnos buscarAlumnoPorId(Integer idAlumno);

    public void guardarAlumno(SgeAlumnos alumno);

    public void eliminarAlumno(SgeAlumnos alumno);

}


// IEnviarMensajes.java — responsabilidad única: gestionar mensajes
package v1v.zona_fit.servicio;

import v1v.zona_fit.modelo.Ticbmessage;
import java.util.List;

public interface IEnviarMensajes {
    public List<Ticbmessage> listarMensajes();

    public Ticbmessage buscarMensajesPorId(Integer idmessage);

    public void enviarmensaje(String medio);
    
}
```

### Principio SOLID aplicado — SRP

> "Un módulo debe tener una, y solo una, razón para cambiar."

| Clase | Única razón de cambio |
|-------|----------------------|
| `IEnviarMensajes`       | Solo contiene los metodos para gestionar los mensajes |

**Antes:** dentro de una sola interfaz se incluia metodo que realizaba multiples funcionalidad 
sin relacion con la idea principal de la interface `ISgeAlumnosServicio`.  
**Después:** Se eliminaron los metodos de la interface en cuestion  `ISgeAlumnosServicio`.
             Se creo la interface para manejo de mensajes `IEnviarMensajes`


## Consecuencias

### Positivas
- `ISgeAlumnosServicio` se puede probar con un mock simple de `EmpresaServicioMock`, sin configurar SMTP.
- `IEnviarMensajes` Debemos configurar SMTP, solo cuando probemos esta funcionalidad


### Negativas / trade-offs
- Puede que existan otras interfaces con el mismo problema de diseno que el original, por deteccion 
    tardia del problema.
- Se ha creado una interface adicional, con respecto a la configuracion original, y se debera crear una nueva clase separada para implementar sus metodos, pero lo que generara a la larga mas orden y menos contaminacion.
