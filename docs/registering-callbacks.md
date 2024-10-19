# Registering Callbacks

In the [previous example](essential-functions.md), we used a non-blocking delay
to blink an LED at regular intervals. In this example, we will achieve the same
result by registering a callback for tick events.

## Create Application Files

Template source files are provided to speed up development. Create a new
directory for our app and copy `template_app.c` to a file with our app name:

```bash
mkdir cpu/src/apps/my-blink-tick
cp template_app.c cpu/src/apps/my-blink-tick/my_blink_tick.c
```

## Initialise Application

In the `app_init()` function we register a callback for user tick events. The
function `ft_register_tick_callback()` takes 2 arguments: an integer describing
the ratio of user ticks to kernel ticks, and the function to call when user tick
events occur. In this example, we set the 'divisor' to 0, indicating one user
tick for every kernel tick. This will trigger our callback every millisecond. We
pass the name of a function, `_tick_callback`, as the second argument. We will
define this function in a later step.

```c
t_status app_init(void) {

    t_status status = ERROR;

    ft_register_tick_callback(0, _tick_callback);

    status = SUCCESS;
    return status;
}
```

## Define Run Behaviour

Our `app_run()` function is similar to the
[previous example](essential-functions.md), only we do not need to call delay
functions. If a flag has been set, we toggle an LED and clear the flag. We will
set the flag in our callback function below.

```c
void app_run(void) {

    if (g_toggle_led) {
        ft_toggle_led(LED_TAP);

        g_toggle_led = false;
    }
}
```

## Define Tick Callback

In our tick callback, we count the number of ticks and set a flag each time we
reach 1000. We test and clear this flag in the `app_run()` function, toggling
the LED once per second. The tick callback takes no arguments and returns
nothing. It is important that the function prototype matches this form, as
defined in the Freetribe API.

```c
static void _tick_callback(void) {

    static uint16_t led_count;

    if (led_count >= 1000) {

        g_toggle_led = true;
        led_count = 0;

    } else {
        led_count++;
    }
}
```

## Build and Run the Application

Build the application by passing `my-blink-tick` to `make` in the `APP`
environment variable:

```bash
make clean && make APP=my-blink-tick
```

Then run the application using GDB:

```bash
cd cpu && arm-none-eabi-gdb
```

The commands in `cpu/.gdbinit` should take care of attaching to the debugger's
GDB server and loading the ELF.

If all is well, you should see the `[Tap]` LED toggle once per second.

## Things to Try

Repeat the tasks from the [previous example](essential-functions.md), toggling
different LEDs at different rates.

## Next Steps

In the [next example](user-input.md), we will register callbacks for user input
and use them to provide feedback.

## `blink-tick.c`

```c
// Freetribe: blink-tick
// License: AGPL-3.0-or-later

/*----- Includes -----------------------------------------------------*/

#include "freetribe.h"

/*----- Macros and Definitions ---------------------------------------*/

#define USER_TICK_DIV 0   // User tick for every systick (~1ms).
#define LED_TICK_DIV 500 // LED tick per 500 user ticks (~0.5s).

/*----- Static variable definitions ----------------------------------*/

static bool g_toggle_led = false;

/*----- Static function prototypes -----------------------------------*/

static void _tick_callback(void);

/*----- Extern function implementations ------------------------------*/

/**
 * @brief   Initialise application.
 */
t_status app_init(void) {

    t_status status = ERROR;

    ft_register_tick_callback(USER_TICK_DIV, _tick_callback);

    status = SUCCESS;
    return status
}

/**
 * @brief   Run application.
 */
void app_run(void) {

    if (g_toggle_led) {
        ft_toggle_led(LED_TAP);

        g_toggle_led = false;
    }
}

/*----- Static function implementations ------------------------------*/

/**
 * @brief   Callback triggered by user tick events.
 */
static void _tick_callback(void) {

    static uint16_t led_count;

    if (led_count++ >= LED_TICK_DIV) {

        g_toggle_led = true;
        led_count = 0;
    }
}

/*----- End of file --------------------------------------------------*/
```
