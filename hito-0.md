# Hito 0: Propuesta y Modelo Inicial

**Proyecto:** Mesa de Ayuda (Tickets de Soporte Técnico)  
**Autores:** Snyder Diaz, Miguel Saldaña

---

## 1\. Situación y Usuario

En una tienda con dos sucursales, seis cajas registradoras y una oficina administrativa, el soporte técnico lo atiende una sola persona que se mueve entre los locales según el día. Los avisos llegan por el grupo de WhatsApp de los encargados, por llamadas, por notas que alguien deja en la caja o cuando un encargado lo alcanza en la puerta. Cada canal guarda la información a su manera, y casi siempre queda en la memoria de quien atendió el aviso.

Eso complica el seguimiento. Un mismo problema se reporta desde las dos sucursales porque nadie sabe que ya estaba avisado. Un pedido urgente, como una caja que no puede cobrar un sábado por la mañana, se pierde entre mensajes de otros temas. Cuando el técnico arregla algo casi nunca queda constancia de qué hizo ni cuándo, y quien reportó no se entera de que ya está listo. Tampoco hay forma sencilla de contestar preguntas básicas, como cuántos problemas siguen pendientes en cada sucursal o cuál lleva más días sin atención.

Un viernes típico ilustra el problema. Cuando el técnico termina en una sucursal y le avisan de la otra, tiene siete mensajes acumulados. Dos son del mismo lector de código de barras, uno es un mouse que falla y puede esperar, y otro es una caja que no abre el sistema justo antes del fin de semana, que es cuando más se vende. Si todo está repartido entre chats y apuntes sueltos, decidir por dónde empezar depende de recordar cuál llegó primero y no de saber cuál es más urgente. Al final del día, nadie puede decir con certeza qué quedó pendiente.

La propuesta es un registro único de tickets. Un ticket es una solicitud de soporte con un título, una descripción del problema, el nombre de quien la reporta, una prioridad y un estado que cambia mientras se atiende.

La persona que más usaría la aplicación es el técnico de soporte. Necesita abrir la lista al empezar el día, ver qué está pendiente y qué es más urgente, y actualizar el estado a medida que avanza. También la usarían los encargados de cada sucursal, que quieren saber si su problema ya fue visto sin tener que escribirle al técnico. Más adelante podrían registrar sus propios tickets, pero en esta primera versión el registro lo hace el técnico a partir de lo que le reportan.

Para trabajar bien, el técnico necesita saber qué falló y en qué sucursal o caja, quién lo reportó para poder avisarle, qué tan urgente es, en qué estado está y desde cuándo se abrió. No necesita, por ahora, inventario de equipos ni historial de piezas. Dejo esas ideas fuera para no complicar el modelo antes de tiempo.

---

## 2\. Acciones y Resultados

Las cuatro acciones se ejecutan desde una sola pantalla integrada:

### 2.1 Registrar un ticket

El técnico llena título, descripción, solicitante y prioridad (por defecto Media) y pulsa **Guardar**.

* **Entrada válida:** El sistema crea el ticket con el número correlativo correspondiente, estado `Abierto` y la fecha actual. Aparece *"Ticket \#5 registrado correctamente"*, el formulario se limpia y el ticket nuevo aparece en la lista.  
* **Entrada inválida:** No se crea nada. Bajo cada campo con problema aparece un mensaje en rojo y la información escrita se conserva.

### 2.2 Encontrar un ticket en la lista

La lista muestra número, título, prioridad, estado y fecha de creación, ordenados con los más recientes primero. Se puede filtrar por estado (`Todos`, `Abierto`, `En proceso`, `Resuelto`, `Cerrado`) o buscar por número.

* **Con resultados:** Muestra únicamente los tickets coincidentes.  
* **Sin resultados:** Aparece el mensaje *"No hay tickets con ese filtro"*.  
* **Número inválido:** Se muestra un mensaje de error sin alterar la lista.

### 2.3 Consultar un ticket

Al pulsar **Ver** en una fila o buscar por número, se despliega el panel de detalle con título, descripción completa, solicitante, prioridad, estado actual, fecha de creación y fecha de actualización. Consultar no modifica datos.

### 2.4 Cambiar el estado de un ticket

En el detalle del ticket, el técnico presiona uno de los botones de estado.

* **Cambio permitido:** Se actualiza el estado y la fecha de modificación, apareciendo el mensaje *"Ticket \#5 ahora está En proceso"*.  
* **Cambio no permitido:** El estado no cambia y muestra un mensaje explicativo en rojo (ej. *"No se puede pasar de Abierto a Cerrado. Primero póngalo En proceso"*).

---

## 3\. Clases Iniciales

### Clase `Ticket`

Representa una solicitud de soporte y encapsula sus propias reglas de negocio.

| Dato | Tipo | Notas |
| :---- | :---- | :---- |
| **Id** | entero | Asignado por el sistema, no editable |
| **Titulo** | texto | 5 a 80 caracteres |
| **Descripcion** | texto | 10 a 500 caracteres |
| **Solicitante** | texto | 3 a 60 caracteres |
| **Prioridad** | Enumeración | Baja, Media, Alta (Por defecto Media) |
| **Estado** | Enumeración | Abierto, EnProceso, Resuelto, Cerrado (Nace en Abierto) |
| **FechaCreacion** | DateTime | Asignada al crear |
| **FechaActualizacion** | DateTime | Actualizada en cada cambio de estado |

#### Transiciones de Estado Permitidas:

* **Abierto** \$\\rightarrow\$ En proceso  
* **En proceso** \$\\rightarrow\$ Resuelto  
* **Resuelto** \$\\rightarrow\$ Cerrado **o** En proceso *(si el problema reaparece)*  
* **Cerrado** \$\\rightarrow\$ Ninguno *(estado final)*

### Clase `RepositorioTickets`

Mantiene la colección de tickets en memoria durante esta primera fase. Encargada de agregar nuevos tickets, asignar identificadores, buscar por Id y listar con filtros.

---

## 4\. Reglas de Negocio

| \# | Regla | Entrada Válida | Respuesta Válida | Entrada Inválida | Respuesta Inválida |
| :---: | :---- | :---- | :---- | :---- | :---- |
| **1** | **Título obligatorio:** Entre 5 y 80 caracteres. | *"Lector de códigos no lee en caja 2"* | Se guarda. *"Ticket registrado correctamente"*. | *"PC"* o espacios. | *"El título debe tener entre 5 y 80 caracteres."* |
| **2** | **Descripción obligatoria:** Entre 10 y 500 caracteres. | *"Lee algunos productos y otros no desde esta mañana."* | Se guarda. | *"No sirve"* (8 caracteres). | *"La descripción debe tener entre 10 y 500 caracteres."* |
| **3** | **Solicitante obligatorio:** Entre 3 y 60 caracteres. | *"Cajera de muestra"* | Se guarda. | Campo vacío. | *"Escriba el nombre de quien reporta (mín. 3 letras)."* |
| **4** | **Transición de Estado:** Debe respetar el flujo permitido. | De `Abierto` a `En proceso`. | Estado cambia. *"Ticket \#5 ahora está En proceso"*. | De `Abierto` a `Cerrado`. | *"No se puede pasar de Abierto a Cerrado. Primero póngalo En proceso."* |
| **5** | **Búsqueda por Id:** Requiere entero positivo existente. | `3` (existente) | Muestra detalle del ticket \#3. | `"abc"` o `99` (inexistente). | `"abc"`: *"Escriba un número válido."* `99`: *"No existe el ticket \#99."* |

---

## 5\. Boceto de Interfaz

\+----------------------------------------------------------------------------------------------------+

|  Mesa de Ayuda | Tickets de Soporte                                                                |

\+--------------------------------------------------+-------------------------------------------------+

| \[1\] REGISTRAR TICKET                             | \[2\] LISTA DE TICKETS                            |

| (Mensaje Éxito: Ticket \#5 registrado)            | Filtro Estado: \[Todos v\]  Buscar ID: \[5\] \[Buscar\]|

|                                                  | \+----+-----------------------+----------+---------+|

| Título:       \[Lector de códigos falla       \]   | | ID | Título                | Estado   | Acción  ||

| Descripción:  \[Solo falla en caja 2\.         \]   | \+----+-----------------------+----------+---------+|

| Solicitante:  \[Cajera (muestra)              \]   | | 5  | Lector de códigos...  | Abierto  | \[ Ver \] ||

| Prioridad:    ( )Baja  (\*)Media  ( )Alta         | | 4  | Caja 1 no abre...     | EnProc.  | \[ Ver \] ||

|               \[ Guardar \]   \[ Limpiar \]          | \+----+-----------------------+----------+---------+|

\+--------------------------------------------------+-------------------------------------------------+

| \[3\] DETALLE DEL TICKET \#5                        | \[4\] CAMBIAR ESTADO                              |

| Título: Lector de códigos falla                  | Actual: \[Abierto\]                               |

| Solicitante: Cajera (muestra) | Prioridad: Media | Botones: \[Abierto\] \[En proceso\] \[Resuelto\] \[Cerrado\]|

| Creado: 08/10/2026                               | (Error: No se puede pasar de Abierto a Cerrado) |

\+----------------------------------------------------------------------------------------------------+

---

## 6\. Pregunta y Próxima Acción

* **Pregunta para el docente:** En esta primera versión, ¿está bien que los tickets se guarden solo en memoria y se pierdan al cerrar la aplicación, o conviene dejar preparado desde ahora un repositorio con persistencia en archivos local?  
* **Primer paso concreto:** Crear el proyecto Blazor en C\#, implementar la clase `Ticket` con sus reglas de validación y escribir las pruebas unitarias para el método `CambiarEstado()` antes de construir las pantallas de usuario.

---

## 7\. Organización del Trabajo

| Parte | Tiempo % | Contenido / Alcance | Evidencia prevista |
| :---- | :---: | :---- | :---- |
| **Modelo** | 40% | Clase `Ticket`, enumeraciones, reglas 1 a 4 y `RepositorioTickets`. | Código fuente en el repositorio con commits por clase. |
| **Interfaz** | 35% | Página Blazor con formulario, lista, detalle y manejo de mensajes. | Capturas de pantalla de cada acción y validación. |
| **Pruebas** | 25% | Pruebas unitarias de casos válidos e inválidos. | Ejecución de pruebas en verde. |

