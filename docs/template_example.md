# Example Template

## Create Application Files

```
mkdir cpu/src/apps/example
cp template_app.c cpu/src/apps/blink-tick/blink-tick.c
```

## Initialise Application  

In the `app_init()` function ``` c
t_status app_init(void) {
    
    t_status status = ERROR;
    
    return SUCCESS;
}
```
## Define Run Behaviour

Our `app_run()` function 

``` c
void app_run(void) {

}
```

## Define Callback

In our callback, 
``` c
static void _callback(void) {

}
```

## Build and Run the Application

Build the application by passing `example` to `make` 
in the `APP` environment variable:

```
make clean && make APP=example
```

Then run the application using GDB:

```
cd cpu && arm-none-eabi-gdb
```

The commands in `cpu/.gdbinit` should take care of
attaching to the debugger's GDB server and loading the ELF.

If all is well, 

## `example.c`

``` c
// Freetribe: blink-tick 
// License: AGPL-3.0-or-later

/*----- Includes -----------------------------------------------------*/

#include "freetribe.h"

/*----- End of file --------------------------------------------------*/
```
