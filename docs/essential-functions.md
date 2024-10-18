# Essential Functions

This minimal example is reproduced from the
[Getting Started](https://bangcorrupt.github.io/freetribe-docs/getting-started)
section of the
[Freetribe Documentation](https://bangcorrupt.github.io/freetribe-docs/). We
demonstrate the essential functions for Freetribe applications, use a
non-blocking delay, and toggle an LED.

## Create Application Files

Create a directory for the app and a file for the code. The app directory must
be under the `freetribe/cpu/src/apps/` directory for the build system to work.

```bash
mkdir cpu/src/apps/my-blink
touch cpu/src/apps/my_blink.c
```

## Include Freetribe API

First, we must include the Freetribe API:

```c
#include "freetribe.h"
```

This gives us access to all the functions our application should need for
interacting with the device.

## Override Initialisation Function

The `app_init()` function is defined as a weak reference in the Freetribe API,
it takes no arguments and returns `t_status`, an integer error code. We override
it to do any one time initialisation required by our app. This is a good place
to register callbacks for events we are interested in, and do any setup required
by external libraries.

In this example, we initialise a static global variable of type `t_delay_state`
to hold the parameters of our delay. The `ft_start_delay()` function sets the
delay time in microseconds and resets the delay state to zero.

```c
// 0.5 seconds in microseconds.
#define DELAY_TIME 500000

static t_delay_state g_blink_delay;

t_status app_init(void) {

    // Initialise delay.
    ft_start_delay(&g_blink_delay, DELAY_TIME);

    return SUCCESS;
}
```

## Override Run Function

The `app_run()` function is called continuously by the kernel in the main loop,
after `app_init()` has completed. It takes no arguments and returns nothing.

In this example, we toggle an LED on the panel if one second has passed, and
reset the delay start time. The `ft_delay()` function returns `true` if the
delay time has expired since `ft_start_delay()` was executed. The
`ft_toggle_led()` function takes the index of an led, and toggles it on or off.
We can call `ft_start_delay()` again with the same struct to reset the delay to
zero.

```c
void app_run(void) {

    // Wait for delay.
    if (ft_delay(&g_blink_delay)) {

        // Toggle LED.
        ft_toggle_led(LED_TAP);

        // Reset start time.
        ft_start_delay(&g_blink_delay, DELAY_TIME);
    }
}
```

## Build the Application

We build the application by running `make` in the root `freetribe` directory. We
pass the name of our application directory (not the path) in the `APP`
environment variable.

```bash
make clean && make APP=my-blink
```

## Run the Application

First we must
[attach a debugger](https://bangcorrupt.github.io/freetribe-docs/debugging) and
start a GDB server. Then we can connect to the GDB server and run the
application:

```bash
cd cpu && arm-none-eabi-gdb
```

The commands in `freetribe/cpu/.gdbinit` should load the application and set a
breakpoint at `main`. Once we hit the breakpoint, input `c` and press `Enter` to
continue. The LED on the `[Tap]` button should blink at a rate of 1 Hz.

## Things to Try

Try modifying the LED blink rate by passing different values to
`ft_start_delay()`.

Try toggling a different LED by passing different values to `ft_toggle_led()`.

Try toggling multiple LEDs at different rates, perhaps using separate
`t_delay_state` variables for each LED.

## Next Steps

In the [next example](registering-callbacks.md), we will register a callback for
user tick events, providing a regular sense of time to our application.

## `blink.c`

```c
// Freetribe: Minimal Example
// License: AGPL-3.0-or-later

#include "freetribe.h"

// 0.5 seconds in microseconds.
#define DELAY_TIME 500000

static t_delay_state g_blink_delay;

t_status app_init(void) {

    // Initialise delay.
    ft_start_delay(&g_blink_delay, DELAY_TIME);

    return SUCCESS;
}

void app_run(void) {

    // Wait for delay.
    if (ft_delay(&g_blink_delay)) {

        // Toggle LED.
        ft_toggle_led(LED_TAP);

        // Reset start time.
        ft_start_delay(&g_blink_delay, DELAY_TIME);
    }
}
```
