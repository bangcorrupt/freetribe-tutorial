# User Input

In the [previous example](registering-callbacks.md) we regisetered a callback for tick events.
In this example, we will register callbacks for knob and button input, 
then use them to send MIDI CC messages and toggle an LED. 

## Create Application Files

```
mkdir cpu/src/apps/midi-knob
cp template_app.c cpu/src/apps/blink-tick/midi-knob.c
```

## Initialise Application  

In the `app_init()` function, we register callbacks for knob and button input events.

``` c
t_status app_init(void) {
    
    t_status status = ERROR;

    ft_register_panel_callback(KNOB_EVENT, _knob_callback);
    ft_register_panel_callback(BUTTON_EVENT, _button_callback);
    
    status =  SUCCESS;
    return status
}
```
## Define Run Behaviour

Our `app_run()` function is very similar to the [previous example](registering-callbacks.md).
If a flag is set, we toggle an LED and clear the flag. The flag will be set 
in our button callback below. In this example, we toggle the `[Play]` button LED.

``` c
void app_run(void) {

    if (g_toggle_led) {
        ft_toggle_led(LED_PLAY);

        g_toggle_led = false;
    }
}
```

## Define Knob Callback

The knob callback takes 2 arguments and returns nothing.  The first argument is the index
of the knob and the second is it's value.  In this example, we send a MIDI CC message
on channel 1, with CC number equal to knob index and CC value equal to half knob value.
The panel MCU sends 8 bit unsigned values in the range 0...255 for knobs, so we right shift 
by 1 for MIDI CC values.  We could save the knob index and value to a static global variable
and send MIDI messages in the `app_run()` function, but as the `ft_send_cc()` function is 
non-blocking there is little advantage in this small application.

``` c

void _knob_callback(uint8_t index, uint8_t value) {

    ft_send_cc(0, index, value >> 1);
}

```

## Define Button Callback

``` c
void _button_callback(uint8_t index, bool state) {

    switch (index) {

    case BUTTON_PLAY:
	if (state == 1) {
	    g_toggle_led == true;
	}
        break;
    }
}
```

## Build and Run the Application

Build the application by passing `midi-knob` to `make` in the `APP` environment variable:

```
make clean && make APP=midi-knob
```

Then run the application using GDB:

```
cd cpu && arm-none-eabi-gdb
```

The commands in `cpu/.gdbinit` should take care of
attaching to the debugger's GDB server and loading the ELF.

If all is well, moving any of the knobs should send MIDI CC messages.
Pressing the `[Play]` button should toggle it's LED.

## `midi-knob.c`

``` c
// Freetribe: midi-knob 
// License: AGPL-3.0-or-later

/*----- Includes -----------------------------------------------------*/

#include "freetribe.h"

/*----- Macros and Definitions ---------------------------------------*/

#define BUTTON_PLAY 0xa

/*----- Static variable definitions ----------------------------------*/

static bool g_toggle_led = false;

/*----- Extern variable definitions ----------------------------------*/

/*----- Static function prototypes -----------------------------------*/

void _knob_callback(uint8_t index, uint8_t value); 
void _button_callback(uint8_t index, bool state); 

/*----- Extern function implementations ------------------------------*/

/**
 * @brief   Initialise application. 
 */
t_status app_init(void) {

    t_status status = ERROR;

    ft_register_panel_callback(KNOB_EVENT, _knob_callback);
    ft_register_panel_callback(BUTTON_EVENT, _button_callback);

    status = SUCCESS;
    return status;
}

/**
 * @brief   Run application.
 */
void app_run(void) {

    if (g_toggle_led) {
        ft_toggle_led(LED_PLAY);

        g_toggle_led = false;
    }
}

/*----- Static function implementations ------------------------------*/

/**
 * @brief   Callback triggered by panel knob events.
 */
void _knob_callback(uint8_t index, uint8_t value) {

    ft_send_cc(0, index, value >> 1);
}

/**
 * @brief   Callback triggered by panel button events.
 */
void _button_callback(uint8_t index, bool state) {

    switch (index) {

    case BUTTON_PLAY:
	if (state == 1) {
	    g_toggle_led == true;
	}
        break;
    }
}

/*----- End of file --------------------------------------------------*/

```
