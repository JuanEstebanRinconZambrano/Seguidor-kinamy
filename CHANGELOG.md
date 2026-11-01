# Changelog: Mini Sumo (ESP32 + 2 × N20 + puente H + LiPo 2S)

Registro de todos los cambios del proyecto. Formato: lo más reciente arriba.
Cada cambio al diseño o a la documentación sube la versión y se anota aquí.

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
