
# Modificación: Control de Turnos y Tirada de Dado en la Carrera de Camellos 🐪

## Descripción General

Este proyecto implementa una modificación al juego original de la Carrera de Camellos, transformándolo de un sistema automático a uno basado en turnos. La modificación principal introduce dos cambios fundamentales:

1. Los camellos avanzan únicamente cuando el jugador (cliente) realiza una tirada de dado
2. Se implementa un sistema de control de turnos para gestionar las tiradas de los jugadores

## Implementación

### 1. Servidor (`Servidor.java`)

#### Nuevas Características
- Control de turno mediante variable `turnoActual`
- Métodos sincronizados para gestión de turnos:
  ```java
  public synchronized int getTurnoActual()
  public synchronized void siguienteTurno()
  ```
- Sistema de avance basado en el valor del dado

### 2. Gestión de Clientes (`GestionClientes.java`)

#### Modificaciones Principales
- Verificación de turno antes de la tirada
- Sistema de notificación al cliente cuando es su turno
- Procesamiento de la tirada y actualización del avance

### 3. Cliente (`Cliente.java`)

#### Nuevas Funcionalidades
- Interpretación de mensajes de turno
- Sistema de tirada de dado mediante interfaz gráfica
- Comunicación del resultado al servidor

### 4. Interfaz Gráfica (`ClienteVentanaCarrera.java`)

#### Elementos Añadidos
- Botón "Lanzar Dado"
- Indicadores visuales de turno
- Sistema de espera para tirada (`esperarTirada()`)

## Diagrama de Flujo del Turno

1. Servidor determina el turno actual
2. Cliente recibe notificación de turno
3. Jugador lanza el dado
4. Servidor procesa el resultado
5. Se actualiza el avance del camello
6. Turno pasa al siguiente jugador

## Detalles Técnicos

### Sincronización

```java
synchronized (servidor) {
    while (servidor.getTurnoActual() != idCamello && !servidor.isFinCarrera()) {
        servidor.wait();
    }
}
```

### Control de Turnos

```java
public synchronized void siguienteTurno() {
    do {
        turnoActual = (turnoActual + 1) % NUM_MAX_JINETES;
    } while (avances[turnoActual] >= 100 && !finCarrera);
    notifyAll();
}
```

## Mejoras y Extensiones Posibles

- Ampliación del número de jugadores
- Personalización del dado
- Mejoras en la interfaz gráfica
- Sistema de puntuación
- Modos de juego alternativos

## Pruebas Recomendadas

1. Verificación de sincronización entre clientes
2. Prueba de control de turnos
3. Validación de avance de camellos
4. Comprobación de estados finales de carrera

## Notas para Desarrolladores

- Revisar la documentación en el código fuente para entender la lógica detallada
- Probar cada modificación de forma aislada
- Mantener la sincronización entre cliente y servidor
- Documentar cualquier cambio adicional

## Requisitos del Sistema

- Java Runtime Environment (JRE)
- Interfaz gráfica compatible
- Conexión de red para modo multijugador

## Contribución

Se anima a los desarrolladores a probar y experimentar con el código. Para contribuir:

1. Realizar fork del repositorio
2. Crear una rama para nuevas características
3. Enviar pull request con los cambios propuestos

---

*Este README forma parte de la documentación del proyecto de modificación de la Carrera de Camellos. Para más detalles, consultar el código fuente y la documentación adicional.*
