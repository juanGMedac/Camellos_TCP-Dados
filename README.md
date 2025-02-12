# Guía de Modificación: Control de Turnos y Tirada de Dado en la Carrera de Camellos

## Introducción

En este proyecto original, la carrera de camellos se ejecuta de forma automática, es decir, cada hilo (representando a un camello) avanza en cada iteración sin la intervención del usuario.  
La **modificación** que vamos a implementar consiste en que:
- **Cada camello avance solo cuando el jugador (cliente) realice una tirada de dado.**
- **Se controle el turno** para que únicamente el jugador que le corresponde lance el dado, mientras que los demás esperen.

Esta guía explica, de manera paso a paso y a nivel conceptual, cómo lograr esta modificación y qué partes del código se deben cambiar.

---

## Paso 1: Controlar el Turno en el Servidor

### 1.1. Agregar una variable para el turno

- **Objetivo:** Permitir que el servidor sepa qué camello (jugador) tiene el turno para lanzar el dado.
- **Dónde cambiar:**  
  En la clase `Servidor.java`, declara una variable `turnoActual` (tipo `int`) que indique el identificador (id) del camello que tiene el turno.

```java
// Declaración en la clase Servidor
private int turnoActual = 0; // El primer turno se asigna al camello 0

1.2. Crear métodos sincronizados para consultar y actualizar el turno
Objetivo: Asegurar que la consulta y actualización del turno sean seguras en un entorno multihilo.

Dónde cambiar:
En Servidor.java, añade métodos como:

public synchronized int getTurnoActual() {
    return turnoActual;
}

public synchronized void siguienteTurno() {
    // Avanza al siguiente camello; si el siguiente ya ha terminado, continúa buscando
    do {
        turnoActual = (turnoActual + 1) % NUM_MAX_JINETES;
    } while (avances[turnoActual] >= 100 && !finCarrera);
    // Notifica a todos los hilos que se ha cambiado el turno
    notifyAll();
}
```
---
1.3. Actualizar el avance según la tirada
Objetivo: Usar el valor del dado para actualizar el avance del camello correspondiente.

Dónde cambiar:
En el método realizarAvance(int idCamello, int avance) de Servidor.java, el valor del dado (entre 1 y 6) se utiliza para actualizar el avance del camello. Se deben conservar las comprobaciones para que el avance no supere 100 y asignar la posición final cuando corresponda.

Paso 2: Modificar el Hilo de Gestión de Clientes (GestionClientes.java)
2.1. Esperar hasta que sea el turno del camello
Objetivo: Cada hilo debe verificar que es su turno antes de proceder a lanzar el dado.

Dónde cambiar:
En el método run() de GestionClientes.java, antes de realizar cualquier acción, se utiliza un bloque synchronized(servidor) y se espera (wait()) hasta que servidor.getTurnoActual() == idCamello.

synchronized (servidor) {
    while (servidor.getTurnoActual() != idCamello && !servidor.isFinCarrera()) {
        servidor.wait();
    }
}

2.2. Notificar al cliente que es su turno
Objetivo: Informar al cliente (jugador) que le corresponde lanzar el dado.

Dónde cambiar:
Una vez que el hilo determina que es su turno, se envía un código especial (por ejemplo, -2) al cliente para que active la funcionalidad de "Lanzar Dado".

// Notificar con el código -2 que es el turno del jugador
out.writeInt(-2);
out.flush();

2.3. Esperar la tirada y actualizar el avance
Objetivo: Recibir el valor del dado del cliente y actualizar el avance del camello.

Dónde cambiar:
Tras enviar el código, el hilo espera la respuesta del cliente (el valor del dado) y lo utiliza para actualizar el avance mediante servidor.realizarAvance(idCamello, dado). Posteriormente, se envían las actualizaciones a ese cliente y se cambia el turno usando servidor.siguienteTurno().

Paso 3: Modificar el Cliente (Cliente.java)
3.1. Interpretar el mensaje de turno
Objetivo: Detectar el código especial enviado por el servidor que indica "¡Es tu turno, lanza el dado!" (por ejemplo, -2).

Dónde cambiar:
En el bucle de comunicación de Cliente.java, al leer el primer entero se debe comprobar si es -2.

int codigo = in.readInt();
if (codigo == -2) {
    // Es el turno del jugador: se debe activar la funcionalidad de lanzar el dado
    // ...
}

3.2. Implementar la tirada del dado mediante la interfaz gráfica
Objetivo: Permitir que el jugador interactúe con la GUI para lanzar el dado.

Dónde cambiar:
En la clase ClienteVentanaCarrera.java, se debe agregar un botón "Lanzar Dado" (deshabilitado por defecto) que se active cuando se reciba el código -2. Además, se implementa un método bloqueante (por ejemplo, esperarTirada()) que espera hasta que el usuario pulse el botón y retorne el valor del dado (un número entre 1 y 6).

// Ejemplo de método en ClienteVentanaCarrera.java:
public int esperarTirada() {
    activarTirada(); // Habilita el botón "Lanzar Dado"
    synchronized(this) {
        while (!dadoLanzado) {
            try {
                wait();
            } catch (InterruptedException ex) {
                ex.printStackTrace();
            }
        }
        dadoLanzado = false;
        return resultadoDado; // Valor generado al pulsar el botón
    }
}

3.3. Enviar el valor del dado al servidor
Objetivo: Una vez que el jugador realiza la tirada, el valor se envía al servidor para actualizar el avance.

Dónde cambiar:
En el bucle de Cliente.java, cuando se reciba el código -2 se llama a esperarTirada() y, una vez obtenido el resultado, se envía al servidor:

int dado = ventana.esperarTirada();
out.writeInt(dado);
out.flush();

Paso 4: Modificar la Interfaz Gráfica (ClienteVentanaCarrera.java)
4.1. Agregar y controlar el botón "Lanzar Dado"
Objetivo: Permitir al jugador lanzar el dado únicamente cuando es su turno.

Dónde cambiar:

Añade un botón "Lanzar Dado" a la interfaz.

El botón debe estar deshabilitado por defecto y habilitarse cuando se reciba el mensaje de turno.

Al pulsar el botón, se genera el número aleatorio (entre 1 y 6) y se notifica al hilo bloqueado mediante notifyAll().

4.2. Indicadores visuales
Objetivo: Mejorar la experiencia del usuario, por ejemplo, cambiando el color del texto del nombre del jugador para indicar que es su turno.

Conclusión
Siguiendo estos pasos, el juego se modificará de tal forma que:

El avance de cada camello depende de la acción del jugador (tirada del dado), en lugar de avanzar automáticamente.

Solo el jugador cuyo turno corresponda puede lanzar el dado, mientras que los demás quedan esperando.

Se utiliza la sincronización con wait() y notifyAll() para coordinar la comunicación y el control de turnos entre el servidor y los clientes.

Esta guía paso a paso debe ayudar a comprender conceptualmente qué partes del código se deben cambiar y cómo implementar la nueva funcionalidad. ¡Anímense a probar y experimentar!

