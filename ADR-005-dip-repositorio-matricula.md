# ADR-005 - Abstraer persistencia de matriculas

**Fecha:** 2026-04-15
**Estado:** Aceptado
**Principio SOLID:** D - Dependency Inversion Principle (DIP) / Principio de Inversion de Dependencias

---

## Contexto

La matricula debe guardarse. En una demo se puede usar memoria, pero en un sistema real podria usarse MySQL, PostgreSQL o un servicio externo.

Codigo problematico:

```java
public class ServicioMatricula {
    private final RepositorioMatriculaMySQL repositorio = new RepositorioMatriculaMySQL();
}
```

El servicio queda acoplado a una implementacion concreta.

---

## Decision

Hacer que `ServicioMatricula` dependa de la abstraccion `RepositorioMatricula`.

Codigo aplicado:

```java
public interface RepositorioMatricula {
    void guardar(Matricula matricula);
    boolean existe(String codigoEstudiante, String codigoCurso);
    List<Matricula> listar();
}

public class ServicioMatricula {
    private final RepositorioMatricula repositorio;

    public ServicioMatricula(RepositorioMatricula repositorio,
                             NotificadorMatricula notificador) {
        this.repositorio = repositorio;
        this.notificador = notificador;
    }
}
```

---

## Principio SOLID aplicado

DIP indica que los modulos de alto nivel no deben depender de modulos de bajo nivel. Ambos deben depender de abstracciones.

Con esta decision:

- `ServicioMatricula` no conoce si se guarda en memoria o base de datos.
- La implementacion puede cambiar sin modificar el caso de uso.
- Las pruebas pueden usar un repositorio falso o en memoria.

---

## Alternativas consideradas

| Alternativa | Por que se descarto |
|-------------|---------------------|
| Instanciar repositorio concreto dentro del servicio | Acopla el caso de uso a infraestructura |
| Usar variables globales | Oculta dependencias reales |
| Guardar directamente desde `Main` | Rompe el flujo de aplicacion |

---

## Consecuencias

### Positivas

- Menor acoplamiento.
- Mayor facilidad para probar.
- Se puede cambiar la infraestructura sin tocar la logica principal.

### Negativas / trade-offs

- Requiere inyectar dependencias al crear el servicio.
