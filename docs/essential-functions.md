# Essential Functions

This minimal example is reproduced from the [Getting Started](https://bangcorrupt.github.io/freetribe-docs/getting-started) section of the [Freetribe Documentation](https://bangcorrupt.github.io/freetribe-docs/).
We demonstrate the essential functions for Freetribe applications, use a non-blocking delay, and toggle an LED.


## Create Application Files

Create a directory for the app and a file for the code.  The app directory must
be under the `freetribe/cpu/src/apps/` directory for the build system to work.

```
mkdir cpu/src/apps/blink
touch cpu/src/apps/blink.c
```

## Include Freetribe API

First, we must include the Freetribe API:

``` c
#include "freetribe.h"
```

This gives us access to all the functions our 
application should need for interacting with the device.


## Override Initialisation Function 

The function `app_init()` is defined as a weak reference in the Freetribe API.
We override it to do any one time initialisation required by our app. 
This is a good place to register callbacks for events we are interested in, 
and do any setup required by external libraries.

In this example, we initialise a static global variable to hold the start time of our delay. 
The `app_init()` function takes no arguments and returns `t_status`, an integer error code.

We also define a macro `DELAY_TIME` as the number of microseconds in 1 second.

``` c
// 1 second in microseconds.
#define DELAY_TIME 1000000 

static uint32_t g_start_time; 

t_status app_init(void) {

    // Set start time.
    g_start_time = ft_get_delay_current();

    status =  SUCCESS;
    return status
}
```

## Override Run Function

The `app_run()` function is also defined as a  weak reference in the Freetribe API. 
We override it to do any continuous processing required by our app.
In this example, we toggle an LED on the panel if 1 second has passed, 
and reset the delay start time. 

The `app_run` function takes no arguments and returns nothing.

``` c
void app_run(void) {

    // Wait for delay.
    if (ft_delay(g_start_time, DELAY_TIME)) {

        // Toggle LED.
        ft_toggle_led(LED_TAP);

        // Reset start time.
        g_start_time = ft_get_delay_current();

    }
}
```

Each time the `app_run()` function is executed, we test the return value of `ft_delay()`.
The function `ft_delay()` returns `true` if `delay_time` microseconds have passed since `start_time`.
Once we enter the `if(...)` block, we toggle an LED and reset the start time 
to the current value of the delay timer.

## Build the Application

We build the application by running `make` in the root `freetribe` directory.
We pass the name of our application directory (not the path) in the `APP` environment variable.

```
make clean && make APP=blink
```

## Run the Application

First we must [attach a debugger](https://bangcorrupt.github.io/freetribe-docs/debugging) and start a GDB server.  Then we can connect to the GDB server and run the application:

```
cd cpu && arm-none-eabi-gdb
```

The commands in `freetribe/cpu/.gdbinit` should load the application and set a breakpoint at `main`.
Once we hit the breakpoint, input `c` and press `Enter` to continue.  The LED on the `[Tap]` button should
blink at a rate of 1 Hz.

## Things to Try

Try modifying the LED blink rate by passing different values to `ft_delay()`.

Try toggling a different LED by passing different values to `ft_toggle_led()`.

Try toggling multiple LEDs at different rates.  It may help to use separate `start_time`
variables for each LED.

## Next Steps

In the [next example](registering-callbacks.md), we will register a callback for user tick events, 
providing a regular sense of time to our application.

## `blink.c`

``` c
// Freetribe: blink
// License: AGPL-3.0-or-later

#include "freetribe.h"

// 1 second in microseconds.
#define DELAY_TIME 1000000 

static uint32_t g_start_time; 

t_status app_init(void) {

    // Set start time.
    g_start_time = ft_get_delay_current();

    status =  SUCCESS;
    return status
}

void app_run(void) {

    // Wait for delay.
    if (ft_delay(g_start_time, DELAY_TIME)) {

        // Toggle LED.
        ft_toggle_led(LED_TAP);

        // Reset start time.
        g_start_time = ft_get_delay_current();

    }
}
```
