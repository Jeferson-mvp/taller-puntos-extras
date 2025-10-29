# taller-puntos-extras
Este juego demuestra cómo combinar la programación gráfica con Python, aplicando los conceptos de hilos, mutex y semáforos de forma práctica.
Cada obstáculo, movimiento y evento de colisión ocurre en tiempo real gracias al manejo paralelo de tareas.
El mutex asegura que los hilos no interfieran entre sí, y el semáforo limita los recursos disponibles, haciendo que el juego sea estable, fluido y seguro frente a errores de sincronización.

##  Variables Principales

El juego mantiene varias variables globales que controlan la lógica general del sistema: número de vidas, velocidad de los obstáculos, nivel actual, cantidad de obstáculos esquivados y la lista de obstáculos activos.
<img width="567" height="212" alt="image" src="https://github.com/user-attachments/assets/0639e53e-b6da-4886-960f-ad035b264eef" />

*Fragmento de las variables globales*
