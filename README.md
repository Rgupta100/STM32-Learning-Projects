# STM32 Learning Projects

Three small STM32CubeIDE projects built while learning the STM32 HAL, each
isolating one peripheral. Target is the **STM32G431KB** (Nucleo-32).

These are deliberately minimal — one peripheral per project, no abstraction —
so the generated CubeMX init code and the hand-written part stay easy to tell
apart.

## Projects

### `PWM/` — timer PWM output

Triangle-wave LED fade. Starts TIM2 channel 1 with `HAL_TIM_PWM_Start`, then
sweeps the compare register 0 → 255 → 0 in 5 ms steps:

```c
for (int i = 0; i < 255; i++) {
    __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, i);
    HAL_Delay(5);
}
```

`__HAL_TIM_SET_COMPARE` writes the CCR directly rather than re-initialising the
timer, which is the cheap way to change duty cycle at runtime. Also brings up
COM1 at 115200 8N1 via the board support package.

Roughly 2.5 s per full fade cycle.

### `IR_Sensor/` — digital input

Reads an IR obstacle/reflectance sensor as a GPIO input.

### `Ultrasound_Sensor/` — input capture / timed echo

HC-SR04 ranging: trigger pulse out, echo pulse width measured and converted to
distance.

## Building

Each folder is a standalone STM32CubeIDE project — import them individually
(File → Import → Existing Projects into Workspace), don't open the parent as one
workspace.

The `.ioc` file in each project is the CubeMX configuration. Open it to change
pin assignments or clock setup and regenerate; hand edits belong strictly
between the `/* USER CODE BEGIN */` and `/* USER CODE END */` markers or CubeMX
will overwrite them.

## Repo layout

```
PWM/                 TIM2 PWM LED fade
IR_Sensor/           IR sensor digital read
Ultrasound_Sensor/   HC-SR04 distance measurement
```

Each contains the usual generated tree: `Core/Src`, `Core/Inc`,
`Core/Startup/startup_stm32g431kbtx.s`, and `Drivers/` (CMSIS + STM32G4 HAL).
The HAL and CMSIS sources are ST's, vendored by CubeMX.

## Notes

- **These are learning exercises, not library code.** No error handling beyond
  the generated `Error_Handler()`, and everything is blocking.
- The vendored `Drivers/` tree is most of the repo by size and is generated, not
  written here.
- A follow-on using two of these boards over CAN is at
  [`STM32-CAN-System`](https://github.com/Rgupta100/STM32-CAN-System) — note
  that one targets the **STM32F103**, not the G431.
