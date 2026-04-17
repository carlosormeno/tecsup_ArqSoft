# ADR-003 - Separar estudiantes generales de estudiantes con beca

**Fecha:** 2026-04-15
**Estado:** Aceptado
**Principio SOLID:** L - Liskov Substitution Principle (LSP) / Principio de Sustitucion de Liskov

---

## Contexto

El sistema maneja estudiantes regulares y estudiantes con beca. Todos pueden tener codigo, nombre y capacidad de matricularse, pero no todos pueden renovar beca ni reportar porcentaje de beca.

Codigo problematico:

```java
public interface Estudiante {
    String codigo();
    String nombre();
    void renovarBeca();
}

public class EstudianteRegular implements Estudiante {
    public void renovarBeca() {
        throw new UnsupportedOperationException("El estudiante no tiene beca");
    }
}
```

El problema es que cualquier cliente que reciba un `Estudiante` podria llamar `renovarBeca()`, pero no todas las implementaciones pueden cumplir ese contrato.

---

## Decision

Separar el contrato general `Estudiante` del contrato especializado `EstudianteConBeca`.

Codigo aplicado:

```java
public interface Estudiante {
    String codigo();
    String nombre();
    boolean puedeMatricularseEn(Curso curso);
}

public interface EstudianteConBeca extends Estudiante {
    double porcentajeBeca();
    void renovarBeca();
}
```

---

## Principio SOLID aplicado

LSP indica que los subtipos deben poder sustituir a sus tipos base sin romper el comportamiento esperado.

Con esta decision:

- `EstudianteRegular` puede usarse como `Estudiante` sin fallar.
- `EstudianteBecado` puede usarse como `EstudianteConBeca`.
- El compilador impide renovar beca sobre un estudiante que no tiene ese contrato.

---

## Alternativas consideradas

| Alternativa | Por que se descarto |
|-------------|---------------------|
| Poner `renovarBeca()` en `Estudiante` | Obliga a estudiantes regulares a implementar una operacion no aplicable |
| Dejar `renovarBeca()` vacio | Oculta un error de modelado |
| Lanzar `UnsupportedOperationException` | Traslada el problema a tiempo de ejecucion |
| Validar con `instanceof` en todo el codigo | Hace que el cliente conozca detalles concretos |

---

## Consecuencias

### Positivas

- Se evita una violacion de LSP.
- Los contratos expresan capacidades reales.
- El codigo cliente trabaja con tipos mas precisos.

### Negativas / trade-offs

- Hay mas interfaces que en una jerarquia unica.
