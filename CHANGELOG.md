# Changelog: Mini Sumo (ESP32 + 2 × N20 + puente H + LiPo 2S)

Registro de todos los cambios del proyecto. Formato: lo más reciente arriba.
Cada cambio al diseño o a la documentación sube la versión y se anota aquí.

## [0.4.0] - 2026-11-05

### Resultados de pruebas (`MiniSumo_B_definitivo.cir`, LTspice 26.1.1 para Windows)
- La simulación corre sin errores (2 pasos de `driver`, 12.9 s).

| Medida | L293D (`driver=0`) | TB6612FNG (`driver=1`) | Lectura |
|---|---|---|---|
| Corriente del motor en marcha libre | 0.036 A | 0.046 A | Normal |
| Corriente del motor en empuje fuerte | 0.646 A | 0.655 A | Igual: la fija la carga |
| Potencia útil entregada al motor (empuje) | 1.80 W | 3.19 W | TB6612 entrega ~77 % más |
| Pérdida en el driver por canal (empuje) | 1.84 W | 0.57 W | L293D pierde 3.2 veces más |
| Pérdida total en el driver (2 canales) | ≈ 3.7 W | ≈ 1.1 W | L293D excede lo que aguanta su encapsulado |
| Eficiencia del puente (empuje) | ≈ 49 % | ≈ 85 % | |
| Caída mínima de VCC2 | 7.20 V | 7.18 V | Rail estable |
| Corriente media de batería en empuje | 1.23 A | 1.23 A | Dentro de diseño |
| Pico de corriente de batería | 1.40 A | 1.46 A | < 2.6 A de diseño |
| Pico de corriente en el motor (`il_pk`) | 14.9 A | 33 A | No realista; ver observaciones |
| Tensión media en motor, marcha libre | 5.11 V | 7.07 V | Ver observaciones |
| ECHO / ADC Sharp / piso alto / piso bajo | 3.173 V / 2.964 V / 3.297 V / 0.033 V | igual | Correcto |

### Conclusión de viabilidad
- **Viable**, con estos cambios: driver TB6612FNG (o DRV8833) en lugar del L293D; PWM en el pin PWM/IN y no en EN; limitar el PWM a ≈ 80 % si los motores son de 6 V; revisar el condensador de 100 nF en los motores; batería LiPo 2S de ≥ 300 mAh y ≥ 20C.
- El L293D no es viable para empuje sostenido: ≈ 3.7 W en total entre dos canales.

### Observaciones
- **Picos de 14.9 A y 33 A:** son casi seguro un efecto del modelo (flancos ideales de 20 ns cargando el condensador de 100 nF de los terminales del motor). Se verifica en B v1.1 con `tr = 200n` y, si persiste, bajando el condensador a 22 nF.
- **Tensión del motor en vacío (5.11 V y 7.07 V):** con el PWM en EN, en vacío la corriente llega a cero en cada apagado y el motor se acelera hasta la tensión máxima disponible (≈ 5.3 V con L293D, ≈ 7 V con TB6612). El ciclo de trabajo casi no controla la velocidad en vacío. Con TB6612 eso supera los 6 V nominales del N20. Solución: PWM en el pin PWM del TB6612 (frenado corto en el apagado) y límite de ≈ 80 %.

### Batería (criterio)
- Corriente media en empuje ≈ 1.23 A, peor caso con ambos motores trabados ≈ 2.0 A; diseñar para ≥ 2.6 A.
- LiPo 2S (7.4 V) de 300 a 450 mAh y ≥ 20C continuo. Corte recomendado: 3.3 V por celda (6.6 V).
- Autonomía de referencia: 300 mAh × 0.8 / 1.23 A ≈ 11 min de empuje continuo (un combate dura ~3 min).

### Pre-selección de componentes (a confirmar con la lista del equipo)
- Driver: TB6612FNG (alternativa DRV8833).
- Piso: QRE1113 (analógico) o módulo TCRT5000.
- Distancia: Sharp GP2Y0A41SK0F (4 a 30 cm) y/o VL53L0X; el HC-SR04 se conserva como sensor central de largo alcance. Sí es viable un segundo sensor de distancia.
- Pines propuestos del ESP32: PWM motores 25 y 13; IN 26, 27, 16, 17; TRIG 23; ECHO 19; Sharp 32 y 33; piso 34, 35, 36, 39.

### Pendiente
- Lista real de componentes y datos de las baterías del equipo.
- Paso 2 del esquemático: zona de energía completa (buck, rieles y cargas).
- Corregir las tablas V-06 y V-11 del documento del proyecto.
- Preparar B v1.1 (`tr = 200n`, PWM en IN, límite de duty).

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
