# SRS — Sistema de Gestión Escolar por Terminal
**Versión:** 1.0.1 
**Fecha:** 31/03/2026 
**Estado:** Borrador aprobado 
**Proyecto:** SchoolApp CLI 
**Clasificación:** Uso interno — Equipo de desarrollo + Product Owner

---

## Tabla de Contenidos

1. [Introducción](#1-introducción) 
2. [Descripción General](#2-descripción-general) 
3. [Objetivos SMART e Indicadores Clave (KPIs)](#3-objetivos-smart-e-indicadores-clave-kpis) 
4. [Requisitos Específicos](#4-requisitos-específicos) 
5. [Modelado Lógico y Modelado de Datos](#5-modelado-lógico-y-modelado-de-datos) 
6. [Matriz de Trazabilidad de Requisitos](#6-matriz-de-trazabilidad-de-requisitos) 
7. [Gestión de Cambios y Control de Versiones](#7-gestión-de-cambios-y-control-de-versiones) 
8. [Glosario](#8-glosario)

---

## 1. Introducción

### 1.1 Propósito

Este documento describe los requisitos del sistema **SchoolApp CLI**, una aplicación de gestión escolar que opera completamente por línea de comandos (terminal). Su objetivo es brindar al colegio una herramienta digital para administrar estudiantes, profesores, materias, tareas y notas, eliminando el uso de registros manuales en papel o en hojas de cálculo desorganizadas.

El documento está dirigido a:
- El **Product Owner** y/o cliente del colegio (para validar que el sistema cubre sus necesidades).
- El **equipo de desarrollo** (para construir el sistema con criterios claros).
- Los **testers** (para diseñar y ejecutar pruebas).

### 1.2 Alcance del Sistema

**SchoolApp CLI** es un sistema de consola que permite:

- Gestionar tres tipos de usuarios: **Administrador**, **Profesor** y **Estudiante**.
- Controlar el acceso mediante un inicio de sesión con correo institucional (`@colegio.edu.co`).
- Administrar estudiantes, materias, tareas y calificaciones.
- Generar un **ranking trimestral** de los 3 estudiantes con mejor promedio.

**Lo que el sistema NO hace en esta versión (v1.0):**
- No tiene interfaz gráfica (web o de escritorio).
- No usa una base de datos real; los datos se almacenan en archivos JSON locales (JSON Server).
- No envía correos electrónicos reales.
- No genera reportes en PDF.

> **Nota para el cliente:** Esta es la primera versión del sistema (MVP). Cada semana se irá mejorando y ampliando, con miras a migrar a base de datos real, servidor web y eventualmente una interfaz gráfica.

### 1.3 Contexto del Problema

Actualmente el colegio gestiona notas, tareas y estudiantes de forma manual o con herramientas no especializadas. Esto genera:

- Pérdida de información.
- Demoras en la consulta de notas por parte de estudiantes.
- Dificultad para identificar a los estudiantes más destacados.
- Falta de trazabilidad en las acciones de cada actor (quién creó qué, cuándo).

**SchoolApp CLI** resuelve estos problemas de forma progresiva, sentando las bases para una plataforma escolar completa.

### 1.4 Supuestos y Dependencias

| # | Supuesto / Dependencia |
|---|------------------------|
| S1 | El correo institucional de todos los usuarios termina en `@colegio.edu.co` |
| S2 | Las 5 materias iniciales son predeterminadas: Matemáticas, Español, Ciencias Naturales, Ciencias Sociales e Inglés |
| S3 | El sistema corre en una máquina local con Java 21 instalado |
| S4 | La persistencia de datos se realiza mediante JSON Server (archivos `.json`) |
| S5 | No existe conexión a internet requerida para el funcionamiento del sistema |
| S6 | El equipo de desarrollo usa Maven como gestor de dependencias |
| S7 | El proyecto evolucionará semanalmente en versiones incrementales |

### 1.5 Restricciones

| # | Restricción |
|---|-------------|
| R1 | Solo se acepta registro con correo que termine en `@colegio.edu.co` |
| R2 | La aplicación solo funciona por terminal (sin interfaz gráfica en v1.0) |
| R3 | El lenguaje de programación es Java 21 |
| R4 | Se deben usar patrones POJO (Java 8) y características modernas (Java 21) de forma combinada |
| R5 | La estructura del proyecto sigue arquitectura de capas organizada por dominio con Maven |

---

## 2. Descripción General

### 2.1 Perspectiva del Producto

SchoolApp CLI es una aplicación independiente (standalone) que no depende de ningún sistema externo en su versión inicial. Es el punto de partida de lo que eventualmente será una plataforma escolar completa con interfaz web, base de datos relacional y acceso remoto.

```
[ Usuario en Terminal ] [ SchoolApp CLI (Java 21) ] [ JSON Server (archivos .json) ]
```

El sistema evoluciona así:

```
v1.0 (actual) → Terminal + JSON Server
v2.0 → Terminal + Base de datos real (MySQL/PostgreSQL)
v3.0 → API REST + Frontend Web
v4.0 → Despliegue en servidor / nube
```

### 2.2 Roles del Sistema

| Rol | Descripción |
|-----|-------------|
| **Administrador (Admin)** | Usuario con acceso total. Gestiona todos los recursos del sistema. Hay al menos uno registrado al iniciar el sistema. |
| **Profesor** | Usuario que gestiona sus propios estudiantes, tareas y notas. No puede eliminar estudiantes ni materias. |
| **Estudiante** | Usuario de solo lectura (para sus propios datos). Puede ver sus notas por tarea, por materia y globales. |

### 2.3 Funciones del Producto

A continuación se describen las funciones principales del sistema de forma clara para el cliente:

#### Autenticación
- Los usuarios **no se auto-registran eligiendo su propio rol**. El flujo es el siguiente:
- El **primer Admin** se crea automáticamente al iniciar el sistema por primera vez (datos predefinidos en configuración).
- El **Admin** es quien crea las cuentas de profesores y estudiantes, asignando el rol correspondiente.
- El registro libre (`Registrarse` en el menú) **no existe** en v1.0; toda cuenta es creada por el Admin.
- Al **iniciar sesión**, el sistema identifica el rol del usuario y muestra el menú correspondiente.
- Si el correo no es institucional, el sistema rechaza el acceso.

#### Funciones del Administrador
- **Gestión de estudiantes:** Crear, ver, editar y eliminar estudiantes del sistema.
- **Gestión de profesores:** Crear, ver, editar y eliminar cuentas de profesores.
- **Gestión de materias:** Crear, ver, editar y eliminar materias. (Las 5 materias predeterminadas vienen cargadas desde el inicio y no pueden eliminarse en v1.0.)
- **Ver ranking trimestral:** Consultar el ranking de los mejores estudiantes del trimestre actual o de trimestres pasados.

> La "Gestión de usuarios" comprende la suma de gestión de estudiantes + gestión de profesores. No es una función separada; ambas se administran desde sus propias secciones del menú.

#### Funciones del Profesor
- **Gestión de estudiantes:** Crear estudiantes y ver la lista de estudiantes.
- **Gestión de tareas:** Crear, ver, editar y eliminar tareas asociadas a una materia.
- **Gestión de notas:** Crear, ver, editar y eliminar notas de estudiantes en tareas específicas.

#### Funciones del Estudiante
- **Ver mis notas por tarea:** Consultar la nota obtenida en cada tarea.
- **Ver mi promedio por materia:** Ver el promedio de notas en cada una de las materias.
- **Ver mi promedio general:** Ver el promedio global de todas las materias.

#### Ranking Trimestral
- Al finalizar cada período de 3 meses, el sistema cierra el trimestre y congela los promedios para el ranking oficial.
- En caso de empate, se muestran todos los estudiantes empatados (el top puede tener más de 3).
- Solo el Administrador puede consultar el ranking.
- **Si el trimestre actual aún no ha cerrado**, el sistema muestra el ranking del último trimestre cerrado disponible, junto con un aviso: _"Mostrando ranking del trimestre anterior. El trimestre actual aún está en curso."_
- Si no existe ningún trimestre cerrado, el sistema muestra: _"Aún no hay trimestres cerrados con datos suficientes para generar el ranking."_

### 2.4 Flujos de Navegación por Rol (Menús de Consola)

#### Menú Principal (sin sesión)
```

SCHOOLAPP CLI v1.0 

1. Iniciar sesión 
2. Salir 

```
> No existe opción de auto-registro. Las cuentas son creadas por el Administrador.

#### Menú Administrador
```

MENÚ ADMINISTRADOR 

1. Gestionar Estudiantes 
2. Gestionar Profesores 
3. Gestionar Materias 
4. Ver Ranking Trimestral 
5. Cerrar sesión 

```

#### Menú Profesor
```

MENÚ PROFESOR 

1. Gestionar Estudiantes 
2. Gestionar Tareas 
3. Gestionar Notas 
4. Cerrar sesión 

```

#### Menú Estudiante
```

MENÚ ESTUDIANTE 

1. Ver mis notas por tarea 
2. Ver promedio por materia 
3. Ver mi promedio general 
4. Cerrar sesión 

```

---

## 3. Objetivos SMART e Indicadores Clave (KPIs)

> Los objetivos SMART son metas que deben ser: **E**specíficas, **M**edibles, **A**lcanzables, **R**elevantes y con **T**iempo definido.

### 3.1 Objetivos SMART

| ID | Objetivo | Específico | Medible | Alcanzable | Relevante | Tiempo |
|----|----------|-----------|---------|------------|-----------|--------|
| OBJ-01 | Digitalizar el registro de notas del colegio | Reemplazar el registro manual de notas con el sistema | 100% de notas ingresadas por terminal | Sí, con 4 developers | Elimina pérdida de datos | Semana 4 |
| OBJ-02 | Permitir a cada estudiante consultar su rendimiento académico en tiempo real | Estudiante accede a sus notas sin depender de un profesor | Tiempo de consulta < 3 segundos | Sí, con JSON Server | Empodera al estudiante | Semana 3 |
| OBJ-03 | Identificar automáticamente a los mejores estudiantes por trimestre | Ranking generado sin intervención manual | Sistema genera ranking correcto en el 100% de los casos | Sí | Motiva el rendimiento | Semana 5 |
| OBJ-04 | Proveer un sistema escalable que migre a web en futuras versiones | Arquitectura por capas y dominios desacoplados | Migración posible sin reescribir lógica de negocio | Sí, con arquitectura por dominio | Base para crecimiento | v2.0 |
| OBJ-05 | Reducir el tiempo que tarda un profesor en registrar notas | Proceso actual vs proceso con el sistema | Reducción del 60% en tiempo de registro | Alcanzable con flujo guiado | Eficiencia docente | Semana 4 |

### 3.2 Indicadores Clave de Desempeño (KPIs)

| KPI | Descripción | Meta |
|-----|-------------|------|
| KPI-01 | Tiempo promedio para registrar una nota | < 30 segundos |
| KPI-02 | Tiempo de respuesta del sistema para consultas | < 3 segundos |
| KPI-03 | Porcentaje de errores de validación capturados correctamente | 100% |
| KPI-04 | Disponibilidad del sistema en sesiones de uso | 99% (sin caídas en terminal) |
| KPI-05 | Cobertura de pruebas unitarias | ≥ 70% en v1.0 |
| KPI-06 | Estudiantes que consultan sus notas sin ayuda del profesor | 100% de los registrados |
| KPI-07 | Ranking trimestral generado correctamente en casos de empate | 100% de los casos |

### 3.3 Beneficios Esperados por Rol

| Rol | Beneficio Principal |
|-----|---------------------|
| **Administrador** | Control total y visibilidad del estado académico del colegio desde un solo lugar |
| **Profesor** | Registro ágil de notas y tareas sin depender de formatos externos |
| **Estudiante** | Acceso inmediato a su rendimiento académico, promoviendo la autorresponsabilidad |
| **Colegio (institución)** | Trazabilidad, orden y base tecnológica para crecer hacia una plataforma web |

---

## 4. Requisitos Específicos

### 4.1 Requisitos Funcionales

#### Módulo de Autenticación

| ID | Requisito | Actor | Prioridad |
|----|-----------|-------|-----------|
| RF-01 | El sistema debe crear automáticamente una cuenta de Administrador por defecto al ejecutarse por primera vez | Sistema | Alta |
| RF-01b | El administrador es el único que puede crear cuentas de profesores y estudiantes; no existe auto-registro en v1.0 | Admin | Alta |
| RF-02 | El sistema debe rechazar el acceso a cuentas con correos que no terminen en `@colegio.edu.co` | Todos | Alta |
| RF-03 | El sistema debe permitir iniciar sesión con correo y contraseña | Todos | Alta |
| RF-04 | El sistema debe mostrar un menú diferente según el rol del usuario autenticado | Todos | Alta |
| RF-05 | El sistema debe permitir cerrar sesión y regresar al menú principal | Todos | Media |
| RF-06 | El sistema debe bloquear el acceso si las credenciales son incorrectas y mostrar mensaje de error | Todos | Alta |

#### Módulo de Gestión de Estudiantes

| ID | Requisito | Actor | Prioridad |
|----|-----------|-------|-----------|
| RF-07 | El administrador puede crear un nuevo estudiante con nombre, correo y contraseña | Admin | Alta |
| RF-08 | El administrador puede ver la lista completa de estudiantes | Admin | Alta |
| RF-09 | El administrador puede editar los datos de un estudiante | Admin | Alta |
| RF-10 | El administrador puede eliminar un estudiante del sistema | Admin | Alta |
| RF-11 | Al eliminar un estudiante, sus **notas** quedan marcadas como huérfanas (no se borran); las tareas no se ven afectadas pues pertenecen al profesor, no al estudiante | Admin | Media |
| RF-12 | El profesor puede crear un nuevo estudiante | Profesor | Alta |
| RF-13 | El profesor puede ver la lista de estudiantes | Profesor | Alta |
| RF-13b | El profesor **NO** puede editar ni eliminar estudiantes; tampoco puede crear, editar ni eliminar materias | Profesor | Alta |

#### Módulo de Gestión de Profesores

| ID | Requisito | Actor | Prioridad |
|----|-----------|-------|-----------|
| RF-34 | El administrador puede crear un nuevo profesor con nombre, correo institucional y contraseña temporal | Admin | Alta |
| RF-35 | El administrador puede ver la lista completa de profesores registrados | Admin | Alta |
| RF-36 | El administrador puede editar los datos de un profesor (nombre y correo) | Admin | Media |
| RF-37 | El administrador puede eliminar un profesor del sistema | Admin | Media |
| RF-38 | Al eliminar un profesor, sus tareas y las notas de esas tareas quedan marcadas como huérfanas y no se eliminan automáticamente; el administrador debe confirmar la acción con un aviso explícito | Admin | Media |

#### Módulo de Gestión de Materias

| ID | Requisito | Actor | Prioridad |
|----|-----------|-------|-----------|
| RF-14 | El sistema debe cargar 5 materias predeterminadas al iniciar por primera vez: Matemáticas, Español, Ciencias Naturales, Ciencias Sociales e Inglés | Sistema | Alta |
| RF-15 | El administrador puede crear nuevas materias | Admin | Media |
| RF-16 | El administrador puede ver la lista de materias | Admin | Alta |
| RF-17 | El administrador puede editar el nombre de una materia | Admin | Media |
| RF-18 | El administrador puede eliminar una materia (excepto las 5 predeterminadas en v1.0) | Admin | Media |

#### Módulo de Gestión de Tareas

| ID | Requisito | Actor | Prioridad |
|----|-----------|-------|-----------|
| RF-19 | El profesor puede crear una tarea asociada a una materia con título, descripción y fecha límite | Profesor | Alta |
| RF-20 | El profesor puede ver todas las tareas de una materia | Profesor | Alta |
| RF-21 | El profesor puede editar una tarea | Profesor | Media |
| RF-22 | El profesor puede eliminar una tarea | Profesor | Media |

#### Módulo de Gestión de Notas

| ID | Requisito | Actor | Prioridad |
|----|-----------|-------|-----------|
| RF-23 | El profesor puede registrar una nota para un estudiante en una tarea específica (valor entre 0.0 y 5.0) | Profesor | Alta |
| RF-24 | El profesor puede ver todas las notas de una tarea | Profesor | Alta |
| RF-25 | El profesor puede editar una nota ya registrada | Profesor | Alta |
| RF-26 | El profesor puede eliminar una nota | Profesor | Media |
| RF-27 | El estudiante puede ver sus notas por tarea | Estudiante | Alta |
| RF-28 | El estudiante puede ver su promedio por materia | Estudiante | Alta |
| RF-29 | El estudiante puede ver su promedio general (todas las materias) | Estudiante | Alta |

#### Módulo de Ranking Trimestral

| ID | Requisito | Actor | Prioridad |
|----|-----------|-------|-----------|
| RF-30 | El sistema debe calcular automáticamente el promedio de cada estudiante al cierre de cada trimestre (cada 3 meses) | Sistema | Alta |
| RF-31 | El administrador puede consultar el ranking trimestral en cualquier momento | Admin | Alta |
| RF-31b | Si el trimestre actual no ha cerrado, el sistema muestra el ranking del último trimestre cerrado con un aviso informativo. Si no hay ningún trimestre cerrado, muestra un mensaje indicándolo | Sistema | Alta |
| RF-32 | El ranking muestra los 3 estudiantes con mayor promedio; en caso de empate se muestran todos los empatados | Sistema | Alta |
| RF-33 | El ranking considera el promedio de las 5 materias predeterminadas | Sistema | Alta |

### 4.2 Requisitos No Funcionales

| ID | Categoría | Requisito |
|----|-----------|-----------|
| RNF-01 | Usabilidad | Los menús de consola deben ser claros, numerados y con instrucciones en español |
| RNF-02 | Usabilidad | Cada operación debe confirmar su éxito o fracaso con un mensaje claro al usuario |
| RNF-03 | Rendimiento | Cualquier consulta o acción debe completarse en menos de 3 segundos |
| RNF-04 | Seguridad | Las contraseñas deben almacenarse cifradas (hash) en el archivo JSON |
| RNF-05 | Seguridad | Un estudiante no puede acceder a las notas de otro estudiante |
| RNF-06 | Seguridad | Un profesor no puede ver ni editar notas de otro profesor |
| RNF-07 | Mantenibilidad | El código debe seguir la arquitectura de capas por dominio (Controller → Service → Repository) |
| RNF-08 | Mantenibilidad | Cada dominio debe tener al menos un test unitario por caso de uso principal |
| RNF-09 | Escalabilidad | La arquitectura debe permitir reemplazar JSON Server por una base de datos real sin reescribir la lógica de negocio |
| RNF-10 | Portabilidad | El sistema debe funcionar en Windows, Linux y macOS con Java 21 instalado |
| RNF-11 | Confiabilidad | El sistema debe manejar entradas inválidas sin cerrarse abruptamente (manejo de excepciones) |
| RNF-12 | Estándares de código | Se deben usar POJOs para modelos de datos (Java 8+) y Records / Sealed Classes donde aplique (Java 21) |

### 4.3 Reglas de Negocio

| ID | Regla |
|----|-------|
| RN-01 | Solo se permiten correos con dominio `@colegio.edu.co` para acceder al sistema |
| RN-02 | Las notas deben estar en el rango de 0.0 a 5.0 |
| RN-03 | El promedio por materia se calcula como la media aritmética de todas las notas de las tareas de esa materia |
| RN-04 | El promedio general se calcula como la media aritmética de los promedios de las materias que **sí tienen al menos una nota registrada**. Las materias sin notas se excluyen del cálculo (no cuentan como 0). Si ninguna materia tiene notas, el promedio general es "Sin datos". |
| RN-05 | En caso de empate en el ranking, se muestran todos los estudiantes con el mismo promedio |
| RN-06 | Un estudiante eliminado conserva sus notas históricas (no se borran en cascada) |
| RN-07 | Las 5 materias predeterminadas no pueden eliminarse en v1.0 |
| RN-08 | El trimestre se calcula desde la fecha de inicio del sistema; cada 90 días se cierra un período |
| RN-09 | **Política de eliminación en cascada:** (a) Al eliminar un **estudiante**: sus notas quedan huérfanas, no se eliminan. Las tareas no se ven afectadas. (b) Al eliminar una **tarea**: sus notas asociadas sí se eliminan en cascada, previa confirmación del profesor. (c) Al eliminar una **materia**: sus tareas y las notas de esas tareas se eliminan en cascada, previa confirmación del administrador. |
| RN-10 | Al eliminar un **profesor**: sus tareas y las notas asociadas a esas tareas quedan **huérfanas** (conservadas con `profesorId` apuntando a un profesor inexistente). El administrador recibe un aviso antes de confirmar. No se eliminan en cascada para preservar trazabilidad histórica. |

---

## 5. Modelado Lógico y Modelado de Datos

### 5.1 Arquitectura del Sistema

El proyecto sigue una **arquitectura de capas organizada por dominio** dentro de un proyecto Maven monolítico. Cada dominio funcional (auth, student, professor, etc.) contiene sus propias capas (controller, service, repository, model), lo que permite separación de responsabilidades sin la complejidad de múltiples módulos Maven.

```
schoolapp-cli/
├── pom.xml                                         <- Proyecto Maven
└── src/
    ├── main/java/com/school/
    │   ├── App.java                                <- Punto de entrada (main)
    │   ├── auth/
    │   │   ├── controller/AuthController.java
    │   │   ├── service/AuthService.java
    │   │   ├── repository/AuthRepository.java
    │   │   └── model/User.java
    │   ├── student/
    │   │   ├── controller/StudentController.java
    │   │   ├── service/StudentService.java
    │   │   ├── repository/StudentRepository.java
    │   │   └── model/Student.java
    │   ├── professor/
    │   │   ├── controller/ProfessorController.java
    │   │   ├── service/ProfessorService.java
    │   │   ├── repository/ProfessorRepository.java
    │   │   └── model/Professor.java
    │   ├── subject/
    │   │   ├── controller/SubjectController.java
    │   │   ├── service/SubjectService.java
    │   │   ├── repository/SubjectRepository.java
    │   │   └── model/Subject.java
    │   ├── task/
    │   │   ├── controller/TaskController.java
    │   │   ├── service/TaskService.java
    │   │   ├── repository/TaskRepository.java
    │   │   └── model/Task.java
    │   ├── grade/
    │   │   ├── controller/GradeController.java
    │   │   ├── service/GradeService.java
    │   │   ├── repository/GradeRepository.java
    │   │   └── model/Grade.java
    │   ├── ranking/
    │   │   ├── controller/RankingController.java
    │   │   ├── service/RankingService.java
    │   │   ├── repository/RankingRepository.java
    │   │   └── model/Period.java
    │   └── shared/
    │       ├── util/JsonFileManager.java
    │       ├── util/MenuHelper.java
    │       ├── util/Validator.java
    │       └── exception/AppException.java
    └── test/java/com/school/                       <- Tests unitarios por dominio
        ├── auth/AuthServiceTest.java
        ├── grade/GradeServiceTest.java
        └── ranking/RankingServiceTest.java
```

**Flujo por capas (ejemplo: registrar una nota):**
```
Usuario en terminal
↓
GradeController ← Recibe la entrada del usuario, llama al servicio
↓
GradeService ← Aplica reglas de negocio (validar rango 0-5, calcular promedio)
↓
GradeRepository ← Lee/escribe en el archivo grades.json
↓
grades.json ← Persistencia de datos
```

### 5.2 Modelo de Datos (Entidades)

#### Usuario (`User`) — Auth
```json
{
"id": "u001",
"nombre": "Carlos Pérez",
"correo": "cperez@colegio.edu.co",
"contrasena": "$2a$10$hashbcrypt...",
"rol": "ADMIN | PROFESOR | ESTUDIANTE",
"activo": true,
"fechaCreacion": "2025-01-15"
}
```

> El modelo `User` es compartido para todos los roles en el archivo `users.json`. El campo `rol` distingue el tipo de cuenta. Los paquetes `student` y `professor` trabajan sobre usuarios filtrados por su rol respectivo.

#### Materia (`Subject`)
```json
{
"id": "m001",
"nombre": "Matemáticas",
"predeterminada": true,
"activa": true
}
```

#### Tarea (`Task`)
```json
{
"id": "t001",
"titulo": "Taller de fracciones",
"descripcion": "Resolver ejercicios del capítulo 3",
"fechaLimite": "2025-02-10",
"materiaId": "m001",
"profesorId": "u002"
}
```
> **Alcance de una tarea:** Una tarea aplica a **todos los estudiantes activos en el sistema** en el momento en que el profesor registra notas. No existe una asignación explícita tarea→estudiante; el profesor simplemente registra la nota de cada estudiante en esa tarea cuando la va calificando.

#### Nota (`Grade`)
```json
{
"id": "g001",
"estudianteId": "u003",
"tareaId": "t001",
"materiaId": "m001",
"valor": 4.5,
"fechaRegistro": "2025-02-11",
"profesorId": "u002"
}
```

#### Período Trimestral (`Period`)
```json
{
"id": "p001",
"nombre": "Trimestre 1 - 2025",
"fechaInicio": "2025-01-01",
"fechaFin": "2025-03-31",
"cerrado": false
}
```

### 5.3 Diagrama de Relaciones (Texto)

```
User (ADMIN) gestiona User (PROFESOR)
User (ADMIN) gestiona User (ESTUDIANTE)
User (ADMIN) gestiona Subject
User (ADMIN) consulta Period (ranking)

User (PROFESOR) 1N Task
User (PROFESOR) 1N Grade (registra)

User (ESTUDIANTE) 1N Grade (recibe)

Subject 1N Task
Task 1N Grade
Period 1N Grade (agrupación temporal por trimestre)
```

**Archivos JSON de persistencia:**

| Archivo | Contenido |
|---------|-----------|
| `users.json` | Todos los usuarios (Admin, Profesor, Estudiante) |
| `subjects.json` | Materias predeterminadas y personalizadas |
| `tasks.json` | Tareas creadas por profesores |
| `grades.json` | Notas registradas por profesores a estudiantes |
| `periods.json` | Períodos trimestrales y sus rankings |

---

## 6. Matriz de Trazabilidad de Requisitos

| ID Req. | Requisito Funcional | HU Relacionada | ID Caso de Prueba |
|---------|--------------------|-----------------|--------------------|
| RF-01 | Crear Admin por defecto al primer inicio | HU-AUTH-01 | CP-AUTH-001 |
| RF-01b | Solo el Admin crea cuentas; no hay auto-registro | HU-AUTH-01 | CP-AUTH-002 |
| RF-02 | Rechazar correo no institucional | HU-AUTH-01 | CP-AUTH-002b |
| RF-03 | Iniciar sesión con credenciales válidas | HU-AUTH-02 | CP-AUTH-003 |
| RF-04 | Mostrar menú según rol | HU-AUTH-02 | CP-AUTH-004 |
| RF-05 | Cerrar sesión | HU-AUTH-03 | CP-AUTH-005 |
| RF-06 | Bloquear acceso con credenciales incorrectas | HU-AUTH-02 | CP-AUTH-006 |
| RF-07 | Admin crea estudiante | HU-STU-01 | CP-STU-001 |
| RF-08 | Admin lista estudiantes | HU-STU-02 | CP-STU-002 |
| RF-09 | Admin edita estudiante | HU-STU-03 | CP-STU-003 |
| RF-10 | Admin elimina estudiante | HU-STU-04 | CP-STU-004 |
| RF-11 | Notas huérfanas al eliminar estudiante | HU-STU-04 | CP-STU-005 |
| RF-12 | Profesor crea estudiante | HU-STU-05 | CP-STU-006 |
| RF-13 | Profesor lista estudiantes | HU-STU-06 | CP-STU-007 |
| RF-13b | Restricciones explícitas del rol Profesor | HU-STU-06 | CP-STU-007b |
| RF-34 | Admin crea profesor | HU-PROF-01 | CP-PROF-001 |
| RF-35 | Admin lista profesores | HU-PROF-02 | CP-PROF-002 |
| RF-36 | Admin edita profesor | HU-PROF-03 | CP-PROF-003 |
| RF-37 | Admin elimina profesor | HU-PROF-04 | CP-PROF-004 |
| RF-38 | Tareas y notas huérfanas al eliminar profesor | HU-PROF-04 | CP-PROF-005 |
| RF-14 | Cargar materias predeterminadas al inicio | HU-SUB-01 | CP-SUB-001 |
| RF-15 | Admin crea materia | HU-SUB-02 | CP-SUB-002 |
| RF-16 | Admin lista materias | HU-SUB-03 | CP-SUB-003 |
| RF-17 | Admin edita materia | HU-SUB-04 | CP-SUB-004 |
| RF-18 | Admin elimina materia no predeterminada | HU-SUB-05 | CP-SUB-005 |
| RF-19 | Profesor crea tarea | HU-TASK-01 | CP-TASK-001 |
| RF-20 | Profesor lista tareas de una materia | HU-TASK-02 | CP-TASK-002 |
| RF-21 | Profesor edita tarea | HU-TASK-03 | CP-TASK-003 |
| RF-22 | Profesor elimina tarea (con aviso de notas en cascada) | HU-TASK-04 | CP-TASK-004 |
| RF-23 | Profesor registra nota (0.0 - 5.0) | HU-GRADE-01 | CP-GRADE-001 |
| RF-24 | Profesor lista notas de una tarea | HU-GRADE-02 | CP-GRADE-002 |
| RF-25 | Profesor edita nota | HU-GRADE-03 | CP-GRADE-003 |
| RF-26 | Profesor elimina nota | HU-GRADE-04 | CP-GRADE-004 |
| RF-27 | Estudiante ve sus notas por tarea | HU-GRADE-05 | CP-GRADE-005 |
| RF-28 | Estudiante ve promedio por materia | HU-GRADE-06 | CP-GRADE-006 |
| RF-29 | Estudiante ve promedio general (excluyendo materias sin notas) | HU-GRADE-07 | CP-GRADE-007 |
| RF-30 | Calcular promedio al cierre de trimestre | HU-RANK-01 | CP-RANK-001 |
| RF-31 | Admin consulta ranking trimestral | HU-RANK-02 | CP-RANK-002 |
| RF-31b | Comportamiento del ranking con trimestre en curso o sin datos | HU-RANK-02 | CP-RANK-002b |
| RF-32 | Mostrar empatados en ranking | HU-RANK-03 | CP-RANK-003 |
| RF-33 | Ranking basado en 5 materias predeterminadas | HU-RANK-01 | CP-RANK-004 |

---

## 7. Gestión de Cambios y Control de Versiones

### 7.1 Roadmap de Versiones

| Versión | Sprint | Descripción | Estado |
|---------|--------|-------------|--------|
| v1.0.0 | Semana 1-2 | MVP: Auth, CRUD estudiantes, materias predeterminadas | En desarrollo |
| v1.1.0 | Semana 3 | CRUD tareas y notas, vista de notas para estudiante | Pendiente |
| v1.2.0 | Semana 4 | Ranking trimestral, promedio general | Pendiente |
| v1.3.0 | Semana 5 | Validaciones robustas, manejo de errores, tests unitarios | Pendiente |
| v2.0.0 | Futuro | Migración a base de datos real (MySQL/PostgreSQL) | Pendiente |
| v3.0.0 | Futuro | API REST con Spring Boot | Pendiente |
| v4.0.0 | Futuro | Frontend web | Pendiente |

### 7.2 Proceso de Control de Cambios

Todo cambio al SRS o al sistema debe seguir este proceso:

1. **Solicitud de cambio:** El Product Owner o un developer documenta el cambio en un issue de GitHub.
2. **Evaluación de impacto:** El equipo analiza qué requisitos y casos de prueba se ven afectados.
3. **Aprobación:** El Product Owner aprueba o rechaza el cambio.
4. **Actualización:** Se actualiza este SRS con la nueva versión y se registra en la tabla de historial.
5. **Implementación:** El developer asignado implementa y actualiza los tests correspondientes.

### 7.3 Historial de Cambios del SRS

| Versión SRS | Fecha | Autor | Descripción del Cambio |
|-------------|-------|-------|------------------------|
| 1.0.0 | 2025 | Equipo SchoolApp | Versión inicial del SRS |
| 1.0.1 | 31/03/2026 | Equipo SchoolApp | Corrección de 12 inconsistencias: flujo de registro/roles, política de cascada (RN-09), promedio general con materias sin notas (RN-04), comportamiento de ranking con trimestre en curso (RF-31b), menú Admin reconciliado, alcance de tareas documentado, tiempo de respuesta unificado (OBJ-02/KPI-02), restricciones explícitas del Profesor (RF-13b), corrección RF-11 (tareas no son huérfanas), fecha completa en encabezado |
| 1.0.2 | 31/03/2026 | Equipo SchoolApp | Agregado dominio completo de Gestión de Profesores: RF-34 a RF-38, RN-10, paquete `professor` en arquitectura por dominio, diagrama de relaciones actualizado, tabla de archivos JSON de persistencia, matriz de trazabilidad actualizada |

---

## 8. Glosario

| Término | Definición |
|---------|-----------|
| **Admin / Administrador** | Rol con acceso total al sistema. Puede gestionar usuarios, materias y ver el ranking. |
| **Caso de Prueba (CP)** | Escenario definido para verificar que una función del sistema trabaja correctamente. |
| **CLI** | Command Line Interface. Interfaz de línea de comandos. El sistema opera completamente en terminal. |
| **CRUD** | Create, Read, Update, Delete. Las cuatro operaciones básicas sobre datos. |
| **Correo institucional** | Dirección de correo electrónico con dominio `@colegio.edu.co`. Requerido para registrarse. |
| **Historia de Usuario (HU)** | Descripción breve de una funcionalidad desde el punto de vista del usuario. |
| **JSON Server** | Herramienta que simula una API REST leyendo y escribiendo archivos `.json`. Usada como capa de persistencia en v1.0. |
| **KPI** | Key Performance Indicator. Indicador clave de desempeño para medir el éxito del sistema. |
| **Maven** | Herramienta de gestión y construcción de proyectos Java. Maneja dependencias y módulos. |
| **MVP** | Minimum Viable Product. Primera versión funcional del sistema con las funciones esenciales. |
| **Materia predeterminada** | Materia cargada automáticamente al iniciar el sistema por primera vez. No puede eliminarse en v1.0. |
| **Dominio** | Paquete del proyecto que encapsula las capas de un área funcional completa (ej. `student/` contiene controller, service, repository y model). |
| **Nota huérfana** | Nota que queda en el sistema luego de que el estudiante asociado es eliminado. Se conserva por trazabilidad. |
| **POJO** | Plain Old Java Object. Clase Java simple sin dependencias de frameworks. Usada para modelos de datos. |
| **Product Owner** | Persona responsable de definir y priorizar los requisitos del sistema (el cliente o su representante). |
| **Promedio general** | Media aritmética de los promedios de todas las materias de un estudiante. |
| **Promedio por materia** | Media aritmética de todas las notas de las tareas de una materia para un estudiante. |
| **Ranking trimestral** | Listado de los 3 (o más, en caso de empate) estudiantes con mayor promedio general al cierre de un período de 3 meses. |
| **Record (Java 21)** | Tipo especial de clase en Java 21 para modelar datos inmutables de forma concisa. |
| **RF** | Requisito Funcional. Describe qué debe hacer el sistema. |
| **RNF** | Requisito No Funcional. Describe cómo debe comportarse el sistema (rendimiento, seguridad, etc.). |
| **RN** | Regla de Negocio. Condición o restricción que el sistema debe respetar según las políticas del colegio. |
| **Rol** | Tipo de usuario en el sistema: Admin, Profesor o Estudiante. Define qué puede hacer cada persona. |
| **Sealed Class (Java 21)** | Clase que restringe qué otras clases pueden heredar de ella. Útil para modelar roles. |
| **Servicio** | Capa del sistema que contiene la lógica de negocio. Se ubica entre el controlador y el repositorio. |
| **SRS** | Software Requirements Specification. Documento que describe todos los requisitos del sistema. |
| **Tarea** | Actividad académica creada por un profesor, asociada a una materia, sobre la cual se registran notas. |
| **Trimestre** | Período de 3 meses (90 días) al cabo del cual se calcula el ranking de estudiantes. |
