# Especificación 001 — MVP de habits-cli

Estado: borrador para revisión
Alcance: primera funcionalidad entregable
Rige sobre esta spec la constitución en `docs/CONSTITUTION.md`.

---

## 1. Contexto y objetivo

Quien estudia por su cuenta pierde continuidad porque no tiene una señal visible
de su propio esfuerzo sostenido. Anotar los días en papel o en una hoja de
cálculo funciona hasta que se rompe la cadena: entonces nadie sabe cuántos días
consecutivos llevaba ni cuál fue su mejor tramo.

**Objetivo:** que una persona pueda registrar, en menos de cinco segundos y sin
salir de la terminal, que hoy cumplió un hábito de estudio, y ver de un vistazo
cuántos días consecutivos lleva y cuál ha sido su mejor racha.

**Por qué la racha y no el total:** el total acumulado premia el volumen; la
racha premia la constancia, que es la conducta que este proyecto quiere
reforzar. La *mejor racha* añade una segunda motivación —superarse a uno mismo—
que sobrevive a una cadena rota.

**Por qué una CLI:** el registro debe costar menos esfuerzo que saltárselo. Una
persona que ya tiene la terminal abierta para estudiar no debería cambiar de
contexto para anotar.

---

## 2. Usuarios

| Usuario | Descripción | Necesita |
|---|---|---|
| **Estudiante autodidacta** (principal) | Persona que estudia por su cuenta, cómoda con la terminal. Registra a diario, normalmente al cerrar su sesión de estudio. | Marcar rápido y ver su racha sin fricción. |
| **Desarrollador junior** (mantenedor) | Quien continúa el proyecto. No participó en las decisiones originales. | Que cada comportamiento esté escrito aquí antes de existir en el código. |

No hay usuario administrador, ni varias personas compartiendo un mismo registro:
cada instalación pertenece a una sola persona.

---

## 3. Historias de usuario

- **HU-1.** Como estudiante, quiero **crear un hábito** con un nombre que yo
  elija, para empezar a llevar su cuenta.
- **HU-2.** Como estudiante, quiero **marcar un hábito como hecho hoy** con un
  solo comando, para que registrar no me cueste más que cumplirlo.
- **HU-3.** Como estudiante, quiero **ver todos mis hábitos con su racha actual
  y su mejor racha**, para saber qué me falta hoy y cuánto tengo en juego.
- **HU-4.** Como estudiante, quiero **registrar un día pasado que olvidé
  anotar**, para que mi historial refleje lo que de verdad hice.
- **HU-5.** Como estudiante, quiero **deshacer una marca** que hice por error,
  para que mis rachas no mientan a mi favor.
- **HU-6.** Como estudiante, quiero **renombrar o borrar un hábito**, para
  corregir un nombre mal escrito o retirar algo que ya no practico.

---

## 4. Requisitos funcionales

Criterios de aceptación en notación EARS: **CUANDO** (evento), **SI…ENTONCES**
(condición no deseada), **MIENTRAS** (estado), **DONDE** (opción), y la forma
ubicua **El sistema DEBE**.

### RF-1 — Crear un hábito

El sistema DEBE permitir crear un hábito indicando un nombre libre.

- CUANDO el usuario crea un hábito con un nombre válido, el sistema DEBE
  registrarlo, asignarle un identificador numérico y confirmar al usuario el
  identificador asignado y el nombre.
- El sistema DEBE asignar identificadores enteros positivos, únicos y crecientes
  en orden de creación.
- El sistema DEBE aceptar nombres repetidos: dos hábitos distintos pueden
  llamarse igual, porque su identidad es el identificador, no el nombre.
- SI el nombre está vacío o compuesto solo por espacios, ENTONCES el sistema
  DEBE rechazar la creación e informar de que el nombre es obligatorio.
- SI el nombre supera los 100 caracteres, ENTONCES el sistema DEBE rechazar la
  creación e informar del límite.
- El sistema DEBE conservar el nombre tal como lo escribió el usuario, salvo los
  espacios sobrantes al principio y al final, que DEBE eliminar.
- Un hábito recién creado DEBE tener racha actual 0 y mejor racha 0.

### RF-2 — Marcar un hábito como hecho hoy

El sistema DEBE permitir registrar que un hábito se cumplió hoy.

- CUANDO el usuario marca un hábito existente que aún no estaba marcado hoy, el
  sistema DEBE registrar la fecha de hoy y confirmar la operación mostrando la
  racha actual resultante.
- SI el hábito ya estaba marcado hoy, ENTONCES el sistema DEBE dejar el registro
  sin cambios, informar de que ya estaba marcado y **tratar la operación como
  éxito**: marcar dos veces no es un error del usuario.
- SI el identificador indicado no corresponde a ningún hábito, ENTONCES el
  sistema DEBE rechazar la operación e indicar cómo consultar los
  identificadores disponibles.
- El sistema DEBE registrar como máximo una marca por hábito y día: un hábito
  está hecho o no lo está, sin grados ni cantidades.

### RF-3 — Marcar una fecha pasada

El sistema DEBE permitir registrar un día anterior a hoy que el usuario olvidó
anotar.

- DONDE el usuario indique una fecha, el sistema DEBE registrar la marca en esa
  fecha en lugar de en hoy.
- El sistema DEBE aceptar las fechas en formato `AAAA-MM-DD`.
- SI la fecha indicada es posterior a hoy, ENTONCES el sistema DEBE rechazar la
  operación: no se puede registrar algo que aún no ha ocurrido.
- SI la fecha indicada no existe en el calendario o no respeta el formato,
  ENTONCES el sistema DEBE rechazar la operación e indicar el formato esperado.
- SI ese día ya estaba marcado, ENTONCES se aplica la misma regla de RF-2: sin
  cambios, con aviso y como éxito.
- CUANDO se registra una fecha pasada, el sistema DEBE recalcular la racha
  actual y la mejor racha teniendo en cuenta la marca añadida, porque rellenar
  un hueco puede unir dos tramos en uno solo.

### RF-4 — Desmarcar un día

El sistema DEBE permitir deshacer una marca registrada por error.

- CUANDO el usuario desmarca un hábito cuyo día indicado estaba marcado, el
  sistema DEBE eliminar esa marca y confirmar la operación mostrando la racha
  actual resultante.
- El sistema DEBE desmarcar hoy cuando el usuario no indique ninguna fecha.
- DONDE el usuario indique una fecha, el sistema DEBE desmarcar ese día, con las
  mismas reglas de validación de fecha de RF-3.
- SI el día indicado no estaba marcado, ENTONCES el sistema DEBE informar de
  ello sin modificar nada y tratar la operación como éxito, por simetría con
  RF-2.
- CUANDO se elimina una marca, el sistema DEBE recalcular la racha actual y la
  mejor racha, incluida la posibilidad de que la mejor racha disminuya.

### RF-5 — Listar los hábitos con sus rachas

El sistema DEBE permitir ver todos los hábitos registrados.

- CUANDO el usuario pide el listado, el sistema DEBE mostrar, para cada hábito:
  identificador, nombre, racha actual, mejor racha y si está marcado hoy.
- El sistema DEBE ordenar el listado por identificador ascendente, es decir, en
  orden de creación. El orden DEBE ser idéntico entre dos ejecuciones
  consecutivas si no ha habido cambios.
- El sistema DEBE distinguir de forma visible los hábitos ya marcados hoy de los
  pendientes.
- MIENTRAS no exista ningún hábito, el sistema DEBE mostrar un mensaje que
  explique cómo crear el primero, y DEBE tratarlo como éxito: no tener hábitos
  todavía no es un error.

### RF-6 — Cálculo de la racha actual

El sistema DEBE calcular la racha actual como el número de días consecutivos
marcados que terminan hoy o ayer.

- CUANDO el día más reciente marcado es hoy, la racha actual DEBE ser el número
  de días consecutivos marcados hacia atrás desde hoy.
- CUANDO el día más reciente marcado es ayer, la racha actual DEBE ser el número
  de días consecutivos marcados hacia atrás desde ayer: la racha sigue viva
  porque el día en curso todavía puede salvarla.
- SI el día más reciente marcado es anterior a ayer, ENTONCES la racha actual
  DEBE ser 0.
- SI el hábito no tiene ninguna marca, ENTONCES la racha actual DEBE ser 0.
- El sistema DEBE contar días naturales completos: dos marcas separadas por un
  día sin marcar nunca pertenecen a la misma racha.

### RF-7 — Cálculo de la mejor racha

El sistema DEBE calcular la mejor racha como el tramo más largo de días
consecutivos marcados en todo el historial del hábito.

- El sistema DEBE incluir la racha actual en el cálculo: si el tramo en curso es
  el más largo, mejor racha y racha actual coinciden.
- El sistema DEBE mantener la mejor racha aunque la cadena se rompa: es un
  registro histórico, no un estado presente.
- SI el hábito no tiene ninguna marca, ENTONCES la mejor racha DEBE ser 0.

### RF-8 — Renombrar un hábito

El sistema DEBE permitir cambiar el nombre de un hábito existente.

- CUANDO el usuario renombra un hábito existente con un nombre válido, el
  sistema DEBE actualizar el nombre y confirmar el cambio.
- El sistema DEBE aplicar al nuevo nombre las mismas reglas de validez de RF-1.
- El sistema DEBE conservar íntegros el identificador y todo el historial de
  marcas: renombrar no altera ninguna racha.
- SI el identificador no corresponde a ningún hábito, ENTONCES el sistema DEBE
  rechazar la operación sin modificar nada.

### RF-9 — Borrar un hábito

El sistema DEBE permitir eliminar un hábito que el usuario ya no practica.

- CUANDO el usuario borra un hábito existente y confirma la operación, el
  sistema DEBE eliminar el hábito y todo su historial de marcas, y DEBE
  confirmar qué se borró.
- El sistema DEBE exigir una confirmación explícita antes de borrar, porque es
  la única operación que destruye historial y no se puede deshacer.
- El sistema NO DEBE reutilizar nunca el identificador de un hábito borrado: los
  identificadores se retiran de forma permanente.
- SI el identificador no corresponde a ningún hábito, ENTONCES el sistema DEBE
  rechazar la operación sin borrar nada.

### RF-10 — Persistencia entre ejecuciones

El sistema DEBE conservar los hábitos y sus marcas entre ejecuciones.

- CUANDO una operación modifica los datos y termina con éxito, el cambio DEBE
  seguir presente en la siguiente ejecución.
- MIENTRAS no exista todavía ningún registro previo, el sistema DEBE comportarse
  como si hubiera cero hábitos, sin error.
- El sistema DEBE almacenar los datos únicamente en la máquina del usuario y NO
  DEBE realizar ninguna comunicación por red.
- SI los datos guardados están dañados o son ilegibles, ENTONCES el sistema DEBE
  detenerse con un mensaje que explique el problema, y NO DEBE sobrescribirlos
  ni descartarlos por su cuenta.

### RF-11 — Comportamiento ante errores

El sistema DEBE tratar todo fallo de forma uniforme y legible.

- CUANDO una operación falla, el sistema DEBE emitir el mensaje por la salida de
  error y terminar con código de salida 1.
- CUANDO una operación termina con éxito —incluidos los avisos de RF-2 y RF-4 y
  el listado vacío de RF-5—, el sistema DEBE terminar con código de salida 0.
- El sistema NO DEBE mostrar nunca al usuario una traza técnica del error.
- Todo mensaje de error DEBE decir qué ocurrió y cuál es la acción siguiente.
- SI el usuario invoca el programa sin indicar operación, o con una operación
  desconocida, ENTONCES el sistema DEBE mostrar la ayuda de uso.

### RF-12 — Idioma

- El sistema DEBE dirigirse al usuario en español en todos sus mensajes,
  incluidos los de error y la ayuda.
- El sistema DEBE aceptar nombres de hábito con tildes, eñes y otros caracteres
  no ingleses, y DEBE mostrarlos sin alterar.

---

## 5. Requisitos no funcionales

- **RNF-1 — Rapidez percibida.** Cualquier operación DEBE responder en menos de
  un segundo con 50 hábitos y dos años de historial. Si registrar cuesta más que
  cumplir el hábito, el usuario deja de registrar.
- **RNF-2 — Datos inspeccionables.** El usuario DEBE poder abrir y entender sus
  propios datos con un editor de texto, sin herramientas especiales.
- **RNF-3 — Mantenibilidad por un junior.** Toda regla de esta spec DEBE ser
  comprobable con un test automático, sin intervención manual.
- **RNF-4 — Funcionamiento sin conexión.** El sistema DEBE funcionar por
  completo sin red.
- **RNF-5 — Privacidad.** Los datos NO DEBEN salir de la máquina del usuario ni
  enviarse a terceros.
- **RNF-6 — Puesta en marcha sin fricción.** Empezar a usar la herramienta NO
  DEBE requerir servicios externos ni configuración previa por parte del
  usuario.

---

## 6. Casos límite

1. **Marca a medianoche.** Una marca a las 23:59 y otra a las 00:01 pertenecen a
   dos días distintos. El día se determina por la fecha local del sistema.
2. **Cambio de zona horaria o del reloj del sistema.** El usuario viaja o
   corrige la fecha y esta retrocede. El sistema no detecta el caso: usa siempre
   la fecha local vigente. Documentado, no gestionado.
3. **Racha que cruza un cambio de mes o de año.** El 31 de diciembre y el 1 de
   enero son días consecutivos.
4. **Año bisiesto.** El 28 y el 29 de febrero son consecutivos en año bisiesto;
   el 28 de febrero y el 1 de marzo lo son en año no bisiesto.
5. **Rellenar un hueco con una fecha pasada.** Dos tramos separados por un único
   día sin marcar se convierten en uno solo al marcar ese día, y la mejor racha
   puede crecer de golpe.
6. **Desmarcar en mitad de una racha.** Partir un tramo puede reducir tanto la
   racha actual como la mejor racha.
7. **Hábito creado hoy y nunca marcado.** Aparece en el listado con racha 0 y
   mejor 0; no se oculta.
8. **Borrado del último hábito.** El listado vuelve al estado vacío de RF-5.
9. **Identificador inexistente o retirado por un borrado.** Se tratan igual: no
   existe, error según RF-11.
10. **Identificador no numérico.** Argumento inválido, error según RF-11.
11. **Nombre con solo espacios, o con emojis.** Solo espacios se rechaza
    (RF-1); cualquier otro carácter imprimible se acepta y se muestra tal cual.
12. **Dos ejecuciones simultáneas del programa.** Fuera del alcance de este MVP:
    se asume un único uso a la vez.

---

## 7. Fuera de alcance

Queda explícitamente fuera de esta primera funcionalidad:

- Hábitos con frecuencia distinta de la diaria (x veces por semana, días
  laborables, días concretos).
- Metas, cantidades o duraciones: un hábito está hecho o no lo está.
- Recordatorios, notificaciones o cualquier ejecución automática.
- Estadísticas más allá de la racha actual y la mejor racha: medias, porcentajes
  de cumplimiento, gráficos, vista de calendario.
- Archivar o pausar un hábito sin borrarlo.
- Categorías, etiquetas, notas o comentarios por día.
- Exportar, importar o sincronizar entre dispositivos.
- Varios usuarios, cuentas o perfiles.
- Deshacer un borrado, papelera o historial de cambios.
- Configuración del usuario: formato de fecha, idioma alternativo, colores.
- Interfaz gráfica o web.

---

## 8. Criterios de finalización

Esta funcionalidad se considera terminada cuando:

1. Las seis historias de usuario (HU-1 a HU-6) se pueden completar de principio
   a fin desde la terminal.
2. Cada requisito funcional RF-1 a RF-12 tiene al menos un test automático por
   criterio de aceptación, incluidos los criterios SI…ENTONCES.
3. Cada caso límite del apartado 6 tiene su test, salvo los declarados no
   gestionados (2 y 12).
4. La suite completa pasa en verde, según el principio 4 de la constitución.
5. Ninguna operación muestra al usuario una traza técnica; todo fallo termina
   con código 1 y todo éxito con código 0.
6. Los datos sobreviven a cerrar y reabrir la terminal.
7. Todos los mensajes dirigidos al usuario están en español.
8. No queda sin resolver ninguna duda del apartado 9 marcada como bloqueante.

---

## 9. Dudas abiertas

- **[NECESITA ACLARACIÓN] (bloqueante)** — *Forma de la confirmación de
  borrado.* RF-9 exige confirmación explícita, pero no fija su forma: ¿una
  pregunta interactiva que el usuario responde, o una señal expresa en la misma
  orden? Lo segundo mantiene la herramienta utilizable en scripts; lo primero
  protege mejor a quien se equivoca. **Afecta a: RF-9.**
- **[NECESITA ACLARACIÓN] (bloqueante)** — *Límite hacia atrás en fechas
  pasadas.* RF-3 prohíbe el futuro, pero no fija suelo. ¿Se puede marcar una
  fecha anterior a la creación del hábito? ¿Hay un tope, por ejemplo un año?
  Sin suelo, un error de tecleo (`2025-09-13` → `2015-09-13`) entra sin aviso y
  distorsiona la mejor racha. **Afecta a: RF-3, RF-7.**
- **[NECESITA ACLARACIÓN] (no bloqueante)** — *Desmarcar fechas pasadas.* RF-4
  asume simetría con RF-3 y admite indicar fecha. Si se prefiere limitar el
  deshacer a hoy, el MVP se simplifica. **Afecta a: RF-4.**
- **[NECESITA ACLARACIÓN] (no bloqueante)** — *Nombres largos en el listado.*
  Con nombres de hasta 100 caracteres, ¿la tabla los recorta o ensancha la
  columna? **Afecta a: RF-5.**
- **[NECESITA ACLARACIÓN] (no bloqueante)** — *Aviso de racha en peligro.* Con
  la regla de gracia de RF-6, un hábito con racha viva pero sin marcar hoy puede
  perderse esta noche. ¿Debe el listado señalarlo de forma distinta a un simple
  «pendiente»? **Afecta a: RF-5, RF-6.**
