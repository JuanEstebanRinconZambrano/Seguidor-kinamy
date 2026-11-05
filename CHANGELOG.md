# Changelog: Mini Sumo (ESP32 + 2 × N20 + puente H + LiPo 2S)

Registro de todos los cambios del proyecto. Formato: lo más reciente arriba.
Cada cambio al diseño o a la documentación sube la versión y se anota aquí.

## [0.3.0] - 2026-11-05

### Resultados de pruebas (`MiniSumo_A_base.cir`, LTspice 26.1.1 para Windows)
- La simulación corre sin errores (3 pasos de `Eb`, 3.8 s).

| Prueba | Medido | Esperado | ¿Pasa? |
|---|---|---|---|
| V-02 Divisor ECHO (máx.) | 3.173 V | ≈ 3.17 V | Sí |
| V-03 Divisor Sharp (máx.) | 2.916 V | ≈ 2.98 V | Sí (el filtro de 100 nF retrasa la rampa ~0.06 V) |
| V-04 Piso, nivel alto | 3.297 V | 3.3 V | Sí |
| V-04 Piso, nivel bajo | 0.033 V | < 0.1 V | Sí |
| V-06 Stall, PWM 80 % en EN: corriente | 0.429 A | 0.55 a 0.6 A | No (menor) |
| V-06 Stall, PWM 80 % en EN: tensión motor | 2.57 V | 3.4 a 3.6 V | No (menor) |
| Carga media (`Eb = 2.5`) | 0.166 A / 3.49 V | n/a | n/a |
| Casi vacío (`Eb = 4.2`) | 0.066 A / 4.60 V | 50 a 100 mA | Sí |
| Caída mínima de VCC2 | 6.80 V | > 6.0 V | Sí |

### Hallazgo
- Al aplicar el PWM en el pin **EN** del L293D, el apagado deja las salidas «sueltas» y la corriente del motor se devuelve hacia la batería (decaimiento rápido). Durante ese tiempo el motor ve una tensión **negativa** de ≈ −(Vbat + 2 caídas de diodo) ≈ −8 a −9 V, y eso baja el promedio. Por eso la tensión media quedó en 2.57 V y no en ~3.5 V. Además la respuesta es no lineal (a poca corriente la corriente llega a cero y el efecto cambia).
- Mi estimación manual de V-06 no consideró este efecto. Prevalece la simulación.

### Cambiado
- Criterio V-06 corregido: con PWM en EN y `Eb = 0`, `duty = 0.8`, se espera ≈ 0.43 A y ≈ 2.6 V.

### Pendiente
- Actualizar la tabla V-06 en el documento del proyecto.
- Probar en el circuito B el PWM en el pin **IN1** con EN fijo en 3.3 V (decaimiento lento, el motor queda en corto en el apagado) y comparar corriente, tensión media y pérdidas.
- Seguir con el Paso 2 del esquemático A (condensadores y PWM).

## [0.2.0] - 2026-11-01

### Añadido
- Explicación en lenguaje común de los términos técnicos (voltaje, corriente, divisor de voltaje, pull-up, PWM, puente H, etc.).
- Aclaración de qué es un archivo `.cir` (netlist) y por qué LTspice no muestra un dibujo al abrirlo.
- Inicio del paso a paso para dibujar el **Circuito A** en LTspice.
  - Paso 1: batería con resistencia interna y una carga de prueba (verificación del punto de operación).
- Este archivo `CHANGELOG.md`.

### Pendiente
- Primera corrida de `MiniSumo_A_base.cir` y `MiniSumo_B_definitivo.cir` en LTspice (las netlists se revisaron a mano, no se simularon).
- Pasos 2 en adelante del Circuito A (PWM, medio puente, motor, sensores).

## [0.1.0] - 2026-10-21

### Añadido
- `MiniSumo_A_base.cir`: circuito base (1 motor, medio puente L293D, divisor del HC-SR04, divisor del Sharp, 1 IR de piso).
- `MiniSumo_B_definitivo.cir`: circuito definitivo (2 motores electromecánicos, driver L293D o TB6612FNG seleccionable, batería con protecciones, buck modelado, 4 IR de piso).
- Documento de proyecto con cálculos, decisiones de diseño (D-01 a D-08), plan de verificación (V-01 a V-16), bitácora y lista de materiales preliminar.
- Cálculos base:
  - Corriente de stall con L293D: 0.77 A por motor a 7.4 V (tensión útil ≈ 4.6 V).
  - Divisor ECHO: 1.8 kΩ y 3.3 kΩ (≈ 3.2 V).
  - Divisor Sharp: 6.8 kΩ y 10 kΩ (≈ 2.98 V máximo).
  - Regulador de 5 V: buck en lugar de AMS1117.

---

## Plantilla para nuevas entradas

```
## [x.y.z] - AAAA-MM-DD

### Añadido
-

### Cambiado
-

### Corregido
-

### Resultado de pruebas
- V-xx: valor medido, ¿pasa?
```

## Estructura sugerida del repositorio

```
mini-sumo/
├── README.md
├── CHANGELOG.md
├── ltspice/
│   ├── MiniSumo_A_base.cir
│   ├── MiniSumo_B_definitivo.cir
│   ├── MiniSumo_A_v0.2.asc      (esquemáticos que vayas dibujando)
│   └── MiniSumo_B_v0.1.asc
└── docs/
    └── (documento del proyecto exportado a PDF o Markdown)
```
