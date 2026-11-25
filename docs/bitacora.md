# Bitácora del laboratorio

## Sesión

- Número de práctica: 1 (Vectores y movimiento)
- Commit o rama: 3f61b2f (rama master) feat: completar practica 1 de movimientos
- Datos del mock utilizados: escenario polinizadores, energiaInicial 100, viento (0, 0), flores F1 = (2, 2, néctar 30) y F2 = (5, 6, néctar 50); resultado: ruta [1, 2], distancia 5, energiaFinal 80, exito true; visual: velocidad 2, dirección (0.6, 0.8)

## Pruebas

| Caso y parámetros | Predicción antes de ejecutar | Resultado observado | Explicación o corrección |
|---|---|---|---|
| P=(2,2), Q=(5,6), v=2, dt=0.5 | Avanzará 1 unidad hacia la flor y quedará en (2.6, 2.8). | `OK Movimiento 3-4-5: <2.6, 2.8>` | Coincide con la hoja: avanza 1 u de las 5 u de distancia. |
| P=Q=(2,2), v=2, dt=0.5 | Ya está en la flor, así que no se moverá. | `OK Vector cero: <2, 2>` | La condición de llegada se revisa antes de dividir. |
| P=(0,0), Q=(0.1,0), v=2, dt=0.5 | Solo falta 0.1 unidad, así que avanzará solo eso y se detendrá justo en la flor, sin pasarse. | `OK Sin sobrepasar: <0.1, 0>` | `MathF.Min` limita el paso a la distancia restante. Sin él falla (ver Error documentado). |
| Horizontal y luego diagonal: P=(2,2), v=2; tecla 3 y luego tecla 2 | Con la tecla 3 solo se moverá hacia la derecha, sin subir ni bajar. Con la tecla 2 se moverá en diagonal, cambiando x y y al mismo tiempo. | Con la tecla 3, y se mantuvo en 2.00 y solo cambió x. Al pulsar 2, x y y cambiaron a la vez. | La dirección (1, 0) solo cambia x. Tarda más porque son 10 u en vez de 5 u. |
| Rapidez 1 y 2: de F1 a F2 | Con 1 u/s tardará el doble que con 2 u/s. La dirección y la trayectoria serán las mismas. | Con 1 u/s tardó ≈ 4.75 s y con 2 u/s ≈ 2.25 s (cronómetro manual) | 5 u / 1 u/s = 5 s y 5 u / 2 u/s = 2.5 s; la diferencia de ≈ 0.25 s es por medir a mano. Al duplicar la rapidez, el tiempo se reduce a la mitad. |

## Error documentado

1. Intento: quité `MathF.Min` de la línea 15 de `MoverHacia` (`Matematicas2D.cs`), dejando `float paso = MathF.Max(0, rapidez) * MathF.Max(0, dt);` y la original comentada, y corrí `dotnet run -- --probar`.
2. Evidencia: `ERROR Sin sobrepasar: <1, 0>, esperado <0.1, 0>` y `Errores: 1`; las otras 10 pruebas siguieron en OK. En pantalla (F1, rapidez 10.00) el agente oscila alrededor de Q sin detenerse: P = (4.91, 6.04), Q = (5.00, 6.00), Distancia = 0.102. Capturas en la sección "Evidencia: capturas" del documento de la práctica.
3. Causa: con P=(0,0), Q=(0.1,0), rapidez 2 y dt 0.5, el paso es 1 u pero solo faltan 0.1 u. `posicion + direccion * paso` se pasa 0.9 u del objetivo, y en el siguiente cuadro la dirección se invierte.
4. Corrección: restauré la línea 15 original, `float paso = MathF.Min(MathF.Max(0, rapidez) * MathF.Max(0, dt), distancia);`, y dejé comentada la temporal. `dotnet run -- --probar` volvió a dar las 11 pruebas en OK y `Errores: 0`. En pantalla, con la misma rapidez 10.00, el agente se detiene en Q: P = (5.00, 6.00) y Distancia = 0.000.
5. Explicación: el paso escalar de la hoja es min(rapidez · dt, distancia). El mínimo evita sobrepasar el destino: cuando el paso calculado supera la distancia restante, el agente avanza solo esa distancia y queda exactamente en Q.

## Datos del proyecto final

| Campo y origen | Unidad o rango | Variable gráfica | Regla de mapeo |
|---|---|---|---|
| `flores[i].x`, `flores[i].y` (escenario) | Unidades de mundo (u). Ejemplo: x de 2 a 5, y de 2 a 6 | `objetivo` (Vector2) | Se busca la flor por su id y su (x, y) es el destino. Se convierte a pantalla solo al dibujar. |
| `visual.velocidad` (respuesta) | u/s. Ejemplo: 2 | `rapidez` (float) | paso = rapidez · dt, con dt de `GetFrameTime()`; se limita con `MathF.Min`. |
| `resultado.ruta` (respuesta) | Enteros que existen en `flores`. Ejemplo: [1, 2] | Lista de objetivos | El cliente resuelve cada id con `BuscarFlor`; con J, la primera flor es la posición inicial y la última el objetivo. |

__________________________________________________________________________
## Práctica 2: Matrices homogéneas 2D

### Sesión

- Número de práctica: 2 (Matrices homogéneas 2D)
- Commit o rama: `ad19e61` (rama master); `feat: completar practica 2 de transformaciones`
- Datos utilizados: puntos originales A = (1, 2), B = (0, 0), C = (-1, 2); matriz del ejemplo M = [[2, 0, 3], [0, 1, -1], [0, 0, 1]]; origen visual (80, 700) px, k = 40 px por unidad; ancla visual (8, 5)

### Pruebas

| Caso y parámetros | Predicción antes de ejecutar | Resultado observado | Explicación o corrección |
|---|---|---|---|
| Identidad aplicada a (1,2) (F2, tecla I) | Queda igual: (1, 2) | Panel: `M p = (1.00, 2.00)` | La identidad no cambia nada; coincide con la hoja. |
| M del ejemplo aplicada a (0,0) (F2, tecla E) | (3, -1) | Panel: `M(0,0) = (3.00, -1.00)` | Con (0, 0) solo queda la columna de desplazamiento. |
| M del ejemplo aplicada a (-1,2) (F2, tecla E) | (1, 1): 2·(-1) + 3 = 1 y 2 - 1 = 1 | Panel: `M(-1,2) = (1.00, 1.00)` | Coincide con el cálculo de la hoja. |
| Mouse (280,460) al mundo | (5, 6): x = (280 - 80) / 40 y y = (700 - 460) / 40 | En la captura 2 el panel dijo `Mouse mundo: 6.00, 5.00` con el mouse sobre otro punto | La misma fórmula inversa da (6, 5) para el píxel (320, 500) y (5, 6) para (280, 460). |

### Error documentado

1. Intento: en `Program.cs`, línea 213 (`DibujarMatrices`), cambié la celda `m[0,2]` de la matriz del ejemplo de 3 a 4: `{ 2, 0, 4 }`. Predije que todo se correría 1 unidad a la derecha.
2. Evidencia: con E el panel dijo `M p = (6.00, 1.00)`, `M(0,0) = (4.00, -1.00)` y `M(-1,2) = (2.00, 1.00)`, y el triángulo azul se movió 1 u a la derecha sin cambiar de forma. Ya no coincidía con la hoja, que da (5, 1). Captura 3 en la sección "Evidencia: capturas" del documento de la práctica.
3. Causa: `m[0,2]` es el desplazamiento en x. Se suma a la x de todos los puntos (w = 1) y no cambia la escala ni la forma.
4. Corrección: restauré el 3 en la línea 213. `dotnet run -- --probar` volvió a dar las 11 pruebas en OK y `Errores: 0`, y el panel volvió a dar `M p = (5.00, 1.00)`.
5. Explicación: en la matriz, la última columna de las dos primeras filas es el desplazamiento. Cambiarla mueve la figura entera, mientras que `m[0,0]` y `m[1,1]` la estiran. Un vector con w = 0 no se vería afectado por este cambio.

### Datos del proyecto final (Problema 6, residuos)

| Campo y origen | Unidad o rango | Variable gráfica | Regla de mapeo |
|---|---|---|---|
| `resultado.clasificados[i].destino` (respuesta) | Texto: reciclaje, organico o especial | Posición (x, y) del contenedor, `objetivo` (Vector2) | Cada destino es un contenedor con su posición en el mundo; el residuo se mueve hacia ella. Se convierte a pantalla solo al dibujar. |
| `residuos[i].peso` (escenario) | Número; unidad y rango por acordar con primero | Escala del objeto | Factor > 0 que se pasa a `Escala(s, s)` o al tamaño del icono. |
| `visual.velocidad` (respuesta) | Número, u/s | `rapidez` (float) | paso = rapidez · dt, con `dt` de `GetFrameTime()`; se limita con `MathF.Min`. |