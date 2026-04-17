# ADR-004 - Segregar servicios academicos por rol

**Fecha:** 2026-04-15
**Estado:** Aceptado
**Principio SOLID:** I - Interface Segregation Principle (ISP) / Principio de Segregacion de Interfaces

---

## Contexto

El sistema tiene distintos actores: estudiante, docente y secretaria academica. Cada rol necesita operaciones diferentes.

Codigo problematico:

```java
public interface GestionAcademica {
    void solicitarMatricula(Estudiante estudiante, Curso curso);
    void consultarHorario(Estudiante estudiante);
    void registrarNota(String codigoEstudiante, String codigoCurso, double nota);
    void aprobarMatricula(String codigoMatricula);
    void generarReporteMatriculas();
}
```

El problema es que un estudiante no registra notas y un docente no aprueba matriculas.

---

## Decision

Separar los servicios por rol.

Codigo aplicado:

```java
public interface PortalEstudiante {
    void solicitarMatricula(Estudiante estudiante, Curso curso);
    void consultarHorario(Estudiante estudiante);
}

public interface PortalDocente {
    void registrarNota(String codigoEstudiante, String codigoCurso, double nota);
    void consultarCursosAsignados(String codigoDocente);
}

public interface PortalSecretaria {
    void aprobarMatricula(String codigoMatricula);
    void generarReporteMatriculas();
}
```

---

## Principio SOLID aplicado

ISP indica que los clientes no deben depender de metodos que no usan.

Con esta decision:

- El estudiante depende solo del portal de estudiante.
- El docente depende solo del portal docente.
- Secretaria depende solo del portal administrativo.

---

## Alternativas consideradas

| Alternativa | Por que se descarto |
|-------------|---------------------|
| Una unica interfaz academica | Obliga a cada rol a conocer operaciones que no usa |
| Una clase con todos los metodos publicos | Aumenta acoplamiento y reduce claridad |
| Validar permisos dentro de cada metodo | No resuelve la dependencia innecesaria |

---

## Consecuencias

### Positivas

- Interfaces mas pequenas y faciles de entender.
- Cada rol depende solo de sus capacidades.
- Menos impacto cuando cambia un proceso de otro rol.

### Negativas / trade-offs

- Puede haber mas interfaces y mas archivos.
