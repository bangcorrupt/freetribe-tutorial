# Display Output

## Create Application Files

```bash
mkdir cpu/src/apps/my-put-pixel
cp template_app.c cpu/src/apps/my-put-pixel/my_put_pixel.c
```

## Initialise Application

In the `app_init()` function we register a callback for trigger pad events and
set the trigger mode to continuous. We will use the data from the first trigger
pad to print a line to the display.

```c
t_status app_init(void) {

    t_status status = ERROR;

    ft_register_panel_callback(TRIGGER_EVENT, _trigger_callback);
    ft_set_trigger_mode(TRIGGER_MODE_CONTINUOUS);

    return SUCCESS;
}
```

## Define Run Behaviour

In our `app_run()` function, we print a line to the display. The length of the
line will be relative to the pressure on the first trigger pad.

```c
void app_run(void) {

    bool pixel_state = 0;
    uint8_t y_pos = 32;

    uint8_t x_pos;

    for (x_pos=0; x_pos < 128; x_pos++;) {

        if (x_pos <= g_pad_pressure) {

            pixel_state = 1;

        } else {
            pixel_state = 0;
        }

        ft_put_pixel(x_pos, y_pos, pixel_state)
    }
}
```

## Define Callback

In our trigger pad callback, we store the pad pressure value in a static global
variable. The trigger callback takes 3 arguments, pad index, pad velocity and
pad state. In continuous mode, the callback will be executed every time the
pressure on the pad changes.

```c
static void _trigger_callback(uint8_t pad, uint8_t vel, bool state) {

    switch(pad) {

        case 0:
            g_pad_pressure = vel;
            break;

        default:
            break;
    }
}
```

## Build and Run the Application

Build the application by passing `my-put-pixel` to `make` in the `APP`
environment variable:

```bash
make clean && make APP=my-put-pixel
```

Then run the application using GDB:

```bash
cd cpu && arm-none-eabi-gdb
```

The commands in `cpu/.gdbinit` should take care of attaching to the debugger's
GDB server and loading the ELF.

If all is well, pushing on the first trigger pad should draw a line across the
screen. The length of the line should change relative to the pressure on the
pad.

## `example.c`

```c
// Freetribe: put-pixel
// License: AGPL-3.0-or-later

/*----- Includes -----------------------------------------------------*/

#include "freetribe.h"

/*----- End of file --------------------------------------------------*/
```
