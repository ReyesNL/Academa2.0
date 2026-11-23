# Principios SOLID en Academa: guía para la exposición

Todos los códigos de esta guía son **los mismos de la diapositiva del grupo**
(`Principios_SOLID_Academa.pptx`). Para cada principio está:

- **Qué dice** el principio.
- **Ejemplo**: el código "En Academa (código real)" de la diapositiva y su explicación.
- **Contraejemplo**: el código "Sin el principio" de la diapositiva y su explicación.
- **Qué decir**: lo que puedes decir en la exposición.

| Letra | Principio | Idea en una línea |
|---|---|---|
| **S** | Responsabilidad Única | Un servicio = una tarea |
| **O** | Abierto / Cerrado | Regla nueva = nueva hija de `Restriccion` |
| **L** | Sustitución de Liskov | El repositorio JSON sustituye al de memoria |
| **I** | Segregación de Interfaces | `RepositorioHorarios`: solo 3 métodos |
| **D** | Inversión de Dependencias | Todo se inyecta desde `main.py` |

---

## S · Responsabilidad Única

> **Cada clase hace una sola cosa y tiene una sola razón para cambiar.**

### ✗ Contraejemplo (sin el principio)

```python
class GestorHorarios:
  generar()
  validar()
  iniciar_sesion()
  guardar_en_json()
  dibujar_tabla()
```

**Explicación:** una sola clase hace cinco trabajos distintos: generar el horario, validarlo, manejar
el login, guardar en un archivo JSON y dibujar la tabla en pantalla. Por eso tiene **cinco razones para
cambiar**. Si se modifica la forma de guardar, se puede romper la generación; la clase crece sin control
y es difícil de probar.

> Ojo: este `GestorHorarios` es un ejemplo **inventado**. El `GestorHorarios` real del proyecto no hace
> esto; solo coordina a los demás servicios.

### ✓ Ejemplo (código real de Academa)

```python
class GeneradorHorarios:
  generar_propuesta()
class ValidadorHorarios:
  detectar_conflictos()
class ServicioAutenticacion:
  iniciar_sesion()
class RepositorioHorarios:
  guardar()
```

📁 `servicios/` · `repositorios/`

**Explicación:** en Academa ese trabajo está repartido en cuatro clases, cada una con **una sola tarea**:

- `GeneradorHorarios.generar_propuesta()` arma automáticamente la propuesta de horario.
- `ValidadorHorarios.detectar_conflictos()` revisa el horario y lista los conflictos (cruces, aula
  incorrecta, docente no disponible).
- `ServicioAutenticacion.iniciar_sesion()` verifica el usuario y la contraseña.
- `RepositorioHorarios.guardar()` guarda los horarios.

Si cambia el login, solo se toca `ServicioAutenticacion`; si cambia la forma de guardar, solo el
repositorio. Cada clase tiene una sola razón para cambiar.

### 🗣️ Qué decir

*"El principio de Responsabilidad Única dice que cada clase debe hacer una sola cosa. En el
contraejemplo, una clase genera, valida, inicia sesión, guarda en JSON y dibuja la tabla: tiene cinco
razones para cambiar. En Academa lo separamos: el GeneradorHorarios genera, el ValidadorHorarios
detecta conflictos, el ServicioAutenticacion maneja el inicio de sesión y el RepositorioHorarios
guarda. Si cambia una de esas cosas, solo modificamos su clase."*

---

## O · Abierto / Cerrado

> **El código debe estar abierto a la extensión y cerrado a la modificación:** agregar una regla nueva
> no obliga a modificar el generador ni el validador.

### ✗ Contraejemplo (sin el principio)

```python
if regla == "espacio":
  ...
elif regla == "disp.":
  ...
# regla nueva = editar aquí
```

**Explicación:** todas las reglas están dentro de un mismo `if / elif`. Cada vez que se quiere agregar
una regla nueva hay que **abrir y modificar** este código, que ya funcionaba, con el riesgo de dañar
las reglas anteriores. Mientras más reglas haya, más largo y frágil se vuelve.

### ✓ Ejemplo (código real de Academa)

```python
class Restriccion(ABC):
  @abstractmethod
  def validar(self, sesion, sesiones)
# 4 hijas: Disponibilidad, Espacio,
# CompatibilidadGrupos,
# DistribucionSemanal
# el generador solo llama validar()
```

📁 `reglas/restricciones.py` · `servicios/`

**Explicación:**

- `Restriccion` es una **clase abstracta** (`ABC`): no se puede crear directamente y obliga a cada
  regla a tener su propio método `validar()` (por eso el `@abstractmethod`).
- Cada regla es una **clase hija**:
  - `RestriccionDisponibilidad`: el docente y el aula están libres.
  - `RestriccionEspacio`: el aula tiene capacidad y es del tipo correcto (aula o laboratorio).
  - `RestriccionCompatibilidadGrupos`: no hay cruces de horario.
  - `RestriccionDistribucionSemanal`: una materia no se repite el mismo día.
- El generador y el validador **solo llaman `validar()`**, sin preguntar qué regla es.

Para agregar una regla nueva (por ejemplo, tutorías) se crea **otra clase hija**; el generador y el
validador no se modifican. El sistema queda abierto a extenderse y cerrado a modificarse.

### 🗣️ Qué decir

*"El principio Abierto/Cerrado dice que el código debe poder extenderse sin modificar lo que ya
funciona. En el contraejemplo, las reglas están en un if/elif y cada regla nueva obliga a editar ese
código. En Academa todas las reglas heredan de la clase abstracta Restriccion, que obliga a
implementar validar(). Tenemos cuatro hijas y el generador solo llama validar(). Para agregar una
regla nueva creamos otra clase hija y no tocamos el generador."*

---

## L · Sustitución de Liskov

> **Una clase hija debe poder usarse en lugar de su clase padre sin romper nada.** Aquí: el repositorio
> JSON puede usarse donde se espera el de memoria.

### ✗ Contraejemplo (sin el principio)

```python
class RepoJSON(RepoMemoria):
  def guardar(horario):
    self._escribir()
    # sin super(): obtener()
    # no encuentra nada
```

**Explicación:** este hijo redefine `guardar()` pero **no llama a `super()`**, es decir, no hace lo que
hacía el padre. Escribe el archivo, pero el horario nunca queda guardado en el repositorio, así que
después `obtener()` no encuentra nada: el horario "desaparece". El hijo **no puede reemplazar** al
padre, porque rompe lo que el padre prometía: *lo que se guarda, se puede obtener*.

> Detalle de la diapositiva: a `def guardar(horario):` le falta `self`; debería ser
> `def guardar(self, horario):`.

### ✓ Ejemplo (código real de Academa)

```python
class RepositorioHorariosMemoria:
  def guardar(self, horario): ...
class RepositorioHorariosJSON(
    RepositorioHorariosMemoria):
  def guardar(self, horario):
    super().guardar(horario)
    self._escribir()
```

📁 `repositorios/horarios.py`

**Explicación:**

- `RepositorioHorariosMemoria` guarda los horarios en memoria mientras el programa está abierto.
- `RepositorioHorariosJSON` **hereda** de él. En `guardar()` primero llama a
  `super().guardar(horario)`, que hace exactamente lo que hacía el padre, y después `self._escribir()`
  guarda además en el archivo JSON para que no se pierda al cerrar el programa.

El hijo **respeta** lo que hacía el padre y solo **agrega** algo. Por eso puede usarse en su lugar sin
problema: la aplicación usa el repositorio JSON y las pruebas usan el de memoria, y el resto del
sistema funciona igual con los dos.

### 🗣️ Qué decir

*"El principio de Liskov dice que una clase hija debe poder reemplazar a su padre sin romper el
programa. En el contraejemplo, el RepoJSON redefine guardar() pero no llama a super(), así que el
horario nunca se guarda en el repositorio y obtener() no encuentra nada. En Academa,
RepositorioHorariosJSON hereda de RepositorioHorariosMemoria y en guardar() primero llama a
super().guardar() y después escribe el archivo. Respeta al padre y solo agrega algo, por eso el JSON
puede usarse donde se espera el de memoria."*

---

## I · Segregación de Interfaces

> **Ninguna clase debe estar obligada a implementar métodos que no usa.** La interfaz del repositorio
> es mínima: solo lo que todos necesitan.

### ✗ Contraejemplo (sin el principio)

```python
class Repositorio(ABC):
  guardar_horario()
  guardar_usuario()
  enviar_correo()
  exportar_pdf()
# todos implementan todo
```

**Explicación:** es una interfaz "gorda", con métodos que no tienen relación entre sí: guardar
horarios, guardar usuarios, enviar correos y exportar PDF. **Todas** las clases que la implementen
estarían obligadas a escribir los cuatro métodos, aunque no los usen. Un repositorio de horarios
tendría que "saber" enviar correos, y terminaría con métodos vacíos o que lanzan errores.

### ✓ Ejemplo (código real de Academa)

```python
class RepositorioHorarios(ABC):
  @abstractmethod
  def guardar(self, horario)
  @abstractmethod
  def obtener(self, carrera_id)
  @abstractmethod
  def listar(self)
```

📁 `repositorios/horarios.py`

**Explicación:** `RepositorioHorarios` es una **interfaz** (clase abstracta con solo métodos
abstractos). Tiene **solo 3 métodos**, que son los que cualquier repositorio de horarios necesita:

- `guardar(horario)`: guarda un horario.
- `obtener(carrera_id)`: busca el horario de una carrera.
- `listar()`: devuelve todos los horarios.

Los repositorios de memoria y JSON implementan los tres y los usan de verdad; ninguno está obligado a
escribir métodos que no le sirven. Enviar correos o exportar PDF serían otras interfaces aparte.

### 🗣️ Qué decir

*"El principio de Segregación de Interfaces dice que una clase no debe estar obligada a implementar
métodos que no usa. En el contraejemplo, la interfaz Repositorio mezcla guardar horarios, guardar
usuarios, enviar correos y exportar PDF, y todos tendrían que implementar todo. En Academa la interfaz
RepositorioHorarios tiene solo tres métodos: guardar, obtener y listar, que son justo los que todos
los repositorios necesitan."*

---

## D · Inversión de Dependencias

> **Las clases no deben crear sus dependencias ni atarse a clases concretas: deben recibirlas desde
> afuera.** El gestor recibe sus dependencias; no las crea él mismo.

### ✗ Contraejemplo (sin el principio)

```python
class GestorHorarios:
  def __init__(self):
    self.repo = RepoJSON()
    self.gen = Generador()
# atado a JSON
```

**Explicación:** el gestor **crea él mismo** su repositorio y su generador dentro del `__init__`, así
que queda **atado a JSON**. Para guardar en una base de datos habría que modificar el gestor, y las
pruebas no podrían usar otro repositorio: escribirían en el archivo real de la aplicación.

### ✓ Ejemplo (código real de Academa)

```python
gestor = GestorHorarios(
  catalogo,
  GeneradorHorarios(catalogo,
                    restricciones),
  ValidadorHorarios(restricciones),
  repositorio)
# pruebas: repositorio en memoria
```

📁 `main.py`

**Explicación:**

- En `main.py` se crean los objetos y se le **pasan** al `GestorHorarios` por el constructor: el
  catálogo, el generador (con sus restricciones), el validador y el repositorio. A esto se le llama
  **inyección de dependencias**.
- El gestor no sabe si el repositorio es JSON o de memoria; solo lo usa.
- Por eso la aplicación le pasa el repositorio JSON y **las pruebas le pasan uno en memoria**, sin
  cambiar ni una línea del gestor. Si mañana se usa una base de datos, solo cambia `main.py`.

### 🗣️ Qué decir

*"El principio de Inversión de Dependencias dice que una clase no debe crear sus dependencias, sino
recibirlas desde afuera. En el contraejemplo, el GestorHorarios crea su RepoJSON y su Generador dentro
del init, y queda atado a JSON. En Academa, en main.py le pasamos al gestor el catálogo, el generador,
el validador y el repositorio por el constructor. Así, en las pruebas le inyectamos un repositorio en
memoria sin modificar el gestor."*

---

## Resumen para repasar

| Principio | Contraejemplo (✗) | Ejemplo en Academa (✓) |
|---|---|---|
| **S** Responsabilidad Única | Una clase con `generar`, `validar`, `iniciar_sesion`, `guardar_en_json` y `dibujar_tabla` | `GeneradorHorarios`, `ValidadorHorarios`, `ServicioAutenticacion` y `RepositorioHorarios`: una tarea cada uno |
| **O** Abierto / Cerrado | `if regla == "espacio": ... elif ...` | `Restriccion` abstracta con 4 hijas; el generador solo llama `validar()` |
| **L** Sustitución de Liskov | `RepoJSON` sin `super()`: `obtener()` no encuentra nada | `RepositorioHorariosJSON` llama `super().guardar()` y luego `_escribir()` |
| **I** Segregación de Interfaces | `Repositorio` con `guardar_usuario`, `enviar_correo`, `exportar_pdf`... | `RepositorioHorarios` con solo `guardar`, `obtener` y `listar` |
| **D** Inversión de Dependencias | `GestorHorarios` crea `RepoJSON()` y `Generador()` en su `__init__` | `main.py` le pasa todo al `GestorHorarios` por el constructor |

**Truco para memorizar el orden:** **S**olo **O**rganiza **L**as **I**deas **D**ivididas.
