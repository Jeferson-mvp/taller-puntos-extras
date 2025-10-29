# taller-puntos-extras
Este juego demuestra cómo combinar la programación gráfica con Python, aplicando los conceptos de hilos, mutex y semáforos de forma práctica.
Cada obstáculo, movimiento y evento de colisión ocurre en tiempo real gracias al manejo paralelo de tareas.
El mutex asegura que los hilos no interfieran entre sí, y el semáforo limita los recursos disponibles, haciendo que el juego sea estable, fluido y seguro frente a errores de sincronización.

##  Variables Principales

El juego mantiene varias variables globales que controlan la lógica general del sistema: número de vidas, velocidad de los obstáculos, nivel actual, cantidad de obstáculos esquivados y la lista de obstáculos activos.

<img width="567" height="212" alt="image" src="https://github.com/user-attachments/assets/0639e53e-b6da-4886-960f-ad035b264eef" />
**Fragmento de las variables globales**

Estas variables son compartidas entre diferentes hilos, lo que requiere el uso de mecanismos de sincronización (como el `mutex`) para evitar conflictos de acceso.  
Por ejemplo, `vidas` disminuye cuando el jugador colisiona con un obstáculo, y `esquivados` aumenta cuando un obstáculo sale de la pantalla sin tocar al jugador.  
La variable `game_over` detiene todos los procesos cuando el jugador pierde todas sus vidas.


## Movimiento del Jugador

El jugador se representa mediante un rectángulo rojo dentro del juego.  
Se controla con las flechas del teclado (`←` y `→`), lo cual le permite desplazarse lateralmente para esquivar los obstáculos.  
<img width="666" height="267" alt="image" src="https://github.com/user-attachments/assets/a2a87387-b28d-49ab-94f3-4a4a7145e0e5" />
**Fragmento del movimiento del jugador**


El método `canvas.coords()` se utiliza para obtener las coordenadas actuales del jugador, y `canvas.move()` realiza el desplazamiento.  
La función se asegura de que el jugador no salga de los límites del Canvas.  
Estas funciones están vinculadas a las teclas mediante `canvas.bind()`, lo que permite una respuesta inmediata a la entrada del usuario.


## Hilos, Mutex y Semáforos

El juego utiliza **hilos** para ejecutar tareas en paralelo y mantener la interfaz fluida.  
El bucle principal de Tkinter maneja la interfaz, mientras que los hilos secundarios se encargan de generar y mover los obstáculos de forma concurrente.

Para coordinar estos hilos y evitar errores, se emplean dos herramientas fundamentales: **el mutex** y **el semáforo**.

<img width="663" height="70" alt="image" src="https://github.com/user-attachments/assets/3c3b1603-8165-48c1-b0aa-7d2151a806e5" />
**Fragmento de inicialización de los hilos y mecanismos de concurrencia:**

El **mutex (bloqueo mutuo)** garantiza que solo un hilo acceda a un recurso compartido —como la lista de obstáculos— en un momento dado.  
Por su parte, el **semaforo** limita la cantidad de obstáculos que pueden estar activos simultáneamente.  
Si se llega al número máximo permitido, el hilo que intenta crear un nuevo obstáculo queda en espera hasta que otro desaparezca.

---

## Generación de Obstáculos (Primer Hilo)

El primer hilo se dedica a crear los obstáculos en posiciones aleatorias dentro del Canvas.  
Cada vez que crea un obstáculo, adquiere un permiso del semáforo para asegurarse de no superar el límite de objetos activos.  
Durante la creación, el hilo entra en una **sección crítica** protegida por el `mutex`.

<img width="908" height="263" alt="image" src="https://github.com/user-attachments/assets/17490a3a-fd9f-4d46-8b69-09f2238f5682" />
**Fragmento del hilo generador de obstáculos:**


Dentro de esta sección crítica se crea el obstáculo gráfico y se agrega a la lista compartida `obstaculos`.  
El uso de `with mutex:` impide que otro hilo modifique la lista al mismo tiempo, evitando errores o inconsistencias visuales.

---

## 🌀 Movimiento y Colisiones (Segundo Hilo)

El segundo hilo se encarga de actualizar el movimiento de los obstáculos en pantalla.  
Cada obstáculo se desplaza hacia abajo, y se verifica constantemente si alguno colisiona con el jugador.  
En caso de colisión, el jugador pierde una vida; si un obstáculo sale de la pantalla, se cuenta como esquivado.

<img width="1041" height="778" alt="image" src="https://github.com/user-attachments/assets/10bbf4a9-6a2d-47f9-a995-24a47ca27259" />
**Fragmento del hilo de movimiento y colisiones:**

Esta parte del programa también está protegida por el `mutex`, ya que accede y modifica la misma lista `obstaculos` que usa el hilo generador.  
Además, cuando un obstáculo desaparece, se libera un permiso del semáforo (`release()`), permitiendo que se genere un nuevo obstáculo.

## La Sección Crítica

En programación concurrente, una **sección crítica** es un bloque de código que accede a un recurso compartido y que solo puede ser ejecutado por un hilo a la vez.  
En este programa, las secciones críticas aparecen cuando se crean, mueven o eliminan obstáculos del Canvas, ya que estas acciones afectan estructuras de datos compartidas.

<img width="931" height="172" alt="image" src="https://github.com/user-attachments/assets/6a08faf2-a121-4048-b7ce-7803e560fcd2" />
<img width="881" height="97" alt="image" src="https://github.com/user-attachments/assets/ce515d68-2747-4534-b339-e130b1a071d0" />
**Ejemplos de sección crítica en el código:**


El uso del `mutex` garantiza exclusión mutua, evitando que varios hilos modifiquen el Canvas o la lista `obstaculos` de forma simultánea.  
Esto es crucial para mantener la estabilidad del juego y evitar errores gráficos o bloqueos inesperados.


## Aumento de Dificultad

El juego incrementa su dificultad de manera progresiva.  
Cada vez que el jugador esquiva 10 obstáculos, el nivel aumenta automáticamente, lo que se traduce en una mayor velocidad de caída y en un menor intervalo entre la aparición de nuevos obstáculos.

<img width="1016" height="161" alt="image" src="https://github.com/user-attachments/assets/8ae183fb-ed46-4ea4-ac32-e4ee9c7e0764" />
**Fragmento del código que gestiona el aumento de dificultad:**


Además, con cada nuevo nivel también se incrementa el número máximo de obstáculos activos permitidos por el semáforo, haciendo que el desafío crezca de forma constante conforme el jugador mejora su desempeño.

---

## 💀 Fin del Juego (Game Over)

Cuando el jugador pierde sus tres vidas, el juego se detiene completamente y aparece en pantalla el mensaje **“GAME OVER”**.  
Esto se logra estableciendo la variable `game_over` en `True`, lo que provoca que los hilos terminen sus ciclos de ejecución.

<img width="1107" height="145" alt="image" src="https://github.com/user-attachments/assets/cefb13de-f780-42b0-87e7-0c04db581909" />
**Fragmento del código del Game Over:**

El mensaje se muestra directamente en el Canvas utilizando `canvas.create_text()`.  
Esta condición marca el fin del juego y detiene el movimiento de todos los elementos.
