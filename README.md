Here's a README file based on your content, written in a way to guide users through each part of the code. 

---

# STM32 LED Blink Example Using Direct Register Access

This project demonstrates how to blink an LED connected to pin 13 on an STM32 microcontroller (GPIOC pin 13) by accessing registers directly, without using higher-level libraries like HAL. This approach provides a deeper understanding of how STM32 peripherals work at a lower level.

---

## Problem Statement

We need to blink the STM32 processor's onboard LED (connected to GPIOC pin 13) and control the blink delay, all through register-level programming.

**Don't worry if this code looks complex at first glance. With a step-by-step breakdown, you'll understand it as easily as enjoying Ali Vai’s cha in Poribohon!** 😊

---

## Code Overview

Here's the core code that enables the LED to blink with a configurable delay:

```c
#define RCC_BASE               0x40021000
#define RCC_APB2ENR            *(volatile unsigned int *)(RCC_BASE + 0x18)
#define RCC_APB1ENR            *(volatile unsigned int *)(RCC_BASE + 0x1C)
#define GPIOC_BASE             0x40011000
#define GPIOC_CRH              *(volatile unsigned int *)(GPIOC_BASE + 0x04)
#define GPIOC_BSRR             *(volatile unsigned int *)(GPIOC_BASE + 0x10)
#define TIM2                   0x40000000
#define TIM2_PSC               *(volatile unsigned int *)(TIM2 + 0x28)
#define TIM2_ARR               *(volatile unsigned int *)(TIM2 + 0x2C)
#define TIM2_SR                *(volatile unsigned int *)(TIM2 + 0x10)
#define TIM2_CR1               *(volatile unsigned int *)(TIM2 + 0x00)
#define DELAY                  100000

void delay(int count){
    TIM2_PSC = 8000 - 1;
    TIM2_ARR = count - 1;
    TIM2_SR &= ~(1);
    TIM2_CR1 |= 1;
    while (!(TIM2_SR & 1));
    TIM2_CR1 &= ~(1);
}

int main(){
    // Enable RCC for GPIO port C
    RCC_APB2ENR |= (1 << 4);

    // Configure mode for PC13
    GPIOC_CRH &= ~(0xF << 20);  // Clear bits 20-23
    GPIOC_CRH |= (1 << 20);
    GPIOC_CRH |= (1 << 21);

    // Enable RCC for TIM2
    RCC_APB1ENR |= 1;

    while (1) {
        GPIOC_BSRR |= (1 << 13);  // Set PC13 high
        delay(1000);
        GPIOC_BSRR |= (1 << 29);  // Reset PC13 (using bit 29)
        delay(1000);
    }

    return 0;
}
```

---

## Step-by-Step Explanation

### Step 1: Identify the LED Port and Enable GPIO Clock
The onboard LED is connected to GPIOC pin 13.

1. **Enable GPIOC Clock**: `RCC_APB2ENR` controls the clocks for various peripherals. Setting the 4th bit in `RCC_APB2ENR` enables the clock for GPIOC.

    ```c
    RCC_APB2ENR |= (1 << 4);
    ```

### Step 2: Configure GPIO Pin Mode
Configure GPIOC's pin 13 to be an output with a specific mode and speed.

- `GPIOC_CRH` (Configuration Register High) manages the configurations of pins 8-15.
- Bits 20-23 control pin 13:
  - **First 2 bits (Mode)**: 00 (input) or 01 (output at 10 MHz).
  - **Second 2 bits (Configuration)**: Set as 11 for output mode at max speed.

    ```c
    GPIOC_CRH &= ~(0xF << 20);  // Clear bits 20-23
    GPIOC_CRH |= (1 << 20);      // Set bit 20 (output mode)
    GPIOC_CRH |= (1 << 21);      // Set bit 21 (max speed)
    ```

### Step 3: Control GPIO Pin Using BSRR
`GPIOC_BSRR` is the Bit Set/Reset Register for GPIOC, allowing us to set or reset specific bits without affecting others.

- **Set Pin 13**: Write to bit 13 in `BSRR` to turn on the LED.
  
    ```c
    GPIOC_BSRR |= (1 << 13);
    ```

- **Reset Pin 13**: Write to bit 29 (13 + 16) to reset it and turn off the LED.

    ```c
    GPIOC_BSRR |= (1 << 29);
    ```

### Step 4: Enable TIM2 Clock
To generate a delay, we use Timer 2 (TIM2), whose clock is controlled by `RCC_APB1ENR`. Setting bit 1 enables TIM2.

```c
RCC_APB1ENR |= 1;
```

### Step 5: Delay Function Setup
Our `delay` function leverages Timer 2 to create a delay by setting values for `PSC`, `ARR`, `SR`, and `CR1` registers:

- **PSC (Prescaler)**: Divides the frequency of TIM2's clock. `PSC = 8000 - 1` creates a 1 ms interval per tick.
  
    ```c
    TIM2_PSC = 8000 - 1;
    ```

- **ARR (Auto-Reload Register)**: Sets the total count. By setting `TIM2_ARR = count - 1`, we control the delay duration.
  
    ```c
    TIM2_ARR = count - 1;
    ```

- **SR (Status Register)**: Flags when counting is complete. We clear it before starting.
  
    ```c
    TIM2_SR &= ~(1);
    ```

- **CR1 (Control Register 1)**: Starts the timer by setting bit 0. It automatically resets after the count completes.

    ```c
    TIM2_CR1 |= 1;
    while (!(TIM2_SR & 1));  // Wait until the timer finishes
    TIM2_CR1 &= ~(1);
    ```

---

## Summary
This example blinks an LED using low-level register manipulations with Timer 2 to control delay timing. By understanding these steps, you gain insights into register-level programming on STM32.

Feel free to tweak delay values and experiment with the code to learn more about STM32's register-based programming!

---

