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
    private final NotificadorMatricula notificador;

    public ServicioMatricula(RepositorioMatricula repositorio,
                             NotificadorMatricula notificador) {
        this.repositorio = repositorio;
        this.notificador = notificador;
    }
}
```

En este caso, `ServicioMatricula` es el modulo de alto nivel porque contiene el flujo principal del caso de uso: validar, calcular, registrar y notificar una matricula.

`RepositorioMatriculaMemoria` es un modulo de bajo nivel porque define un detalle tecnico: guardar las matriculas en una lista en memoria.

La regla de DIP es que el modulo de alto nivel no debe depender directamente del modulo de bajo nivel. Por eso el servicio no hace esto:

```java
private final RepositorioMatriculaMemoria repositorio = new RepositorioMatriculaMemoria();
```

En su lugar, depende de la abstraccion:

```java
private final RepositorioMatricula repositorio;
```

La implementacion concreta se entrega desde afuera, al crear el servicio:

```java
RepositorioMatricula repositorio = new RepositorioMatriculaMemoria();
NotificadorMatricula notificador = new NotificadorConsola();

ServicioMatricula servicio = new ServicioMatricula(repositorio, notificador);
```

De esta forma, si luego se crea un repositorio para MySQL, el servicio no cambia:

```java
RepositorioMatricula repositorio = new RepositorioMatriculaMySQL();
ServicioMatricula servicio = new ServicioMatricula(repositorio, notificador);
```

El mismo criterio se aplica a `NotificadorMatricula`: el servicio no depende de `NotificadorConsola` ni de un correo real, sino del contrato `NotificadorMatricula`.

---

## Principio SOLID aplicado

DIP indica que los modulos de alto nivel no deben depender de modulos de bajo nivel. Ambos deben depender de abstracciones.

Con esta decision:

- `ServicioMatricula` no conoce si se guarda en memoria o base de datos.
- `ServicioMatricula` no instancia `RepositorioMatriculaMemoria`; recibe un `RepositorioMatricula`.
- `ServicioMatricula` no instancia `NotificadorConsola`; recibe un `NotificadorMatricula`.
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
