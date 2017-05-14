# Understanding Menus

## Instantiating an instance of the menu class

The menu framework provided by the dot3k library enables us to create a menu instance and pass a list of menu items for display on the dot3k, a corresponding function to call when each menu option is selected or a sub-menu to display

First we need to import the sys and time modules so that we can use their functions here.

```python
import sys
import time
```

Since we want to use the backlight, joystick and lcd with our menu we'll need to import those frameworks also.

```python
import dot3k.backlight as backlight
import dot3k.joystick as jstick
import dot3k.lcd as lcd
```

Next we should import Menu and MenuOption to enable us to create a Menu instance and use MenuOption for any menu plugins.

```python
from dot3k.menu import Menu, MenuOption
```

Any existing menu plugins we intend to call from our menu should also be imported here.  For example we could import a plugin to display a clock on the lcd and plugins to display our IP address, show CPU and GPU temperature and reboot or shutdown our Raspberry Pi.  

Before we do that we also need to point Python to the directory where the plugins are located.  When we import a module Python checks first to see if it's a built-in module then searches for a python file with that name (e.g. clock.py). If not Python checks the current directory / directory where the python file importing the module exists then it checks the $PYTHONPATH (sys.path) and finally it checks the installation dependent default location.

We could simply append to the existing sys.path the absolute path to where the Pimoroni dot3k plugins code resides on our Pi, for example:

```python
sys.path.append('/home/pi/Pimoroni/displayotron/dot3k/')
```

However we're not going to do this as it is generally considered poor programming practise.  If we share our code or try to execute it on another device it's likely importing the plugins will fail because python can't locate the modules in the plugins directory.

As an alternative we can create our python file in a specific location (e.g. /home/pi/Pimoroni/displayotron/dot3k/examples/advanced) and use the following to have Python look for the plugins directory with our modules to import by appending a relative path pointing to two directories up from where our main python file is located.  Once we've done that we can also import any plugins we need for our menu.

```python
# Add the root examples dir so Python can find the plugins
sys.path.append('../../')

from plugins.clock import Clock
from plugins.graph import IPAddress, GraphTemp, GraphCPU, GraphSysReboot, GraphSysShutdown
```

Also we could write our own plugin and include it directly in the body of our main Python code to use in our menu instance.  To do this we create a class which inherits from the **MenuOption** class and at a minimum override the methods that initialise the plugin and redraw it.  See Plugins below for further information on creating your own plugins.

```python
class AwesomePlugin(MenuOption):
    """
    An awesome plugin that displays awesome on the dot3k.
    """

    def __init__(self):
        
        MenuOption.__init__(self)

    def redraw(self, menu):
        
        menu.clear_row(0)
        menu.write_row(1, 'Awesome!!!')
        menu.clear_row(2)
```

Assuming we plan on using the AwesomePlugin class we also need to create an instance of the class to call from our menu as follows.

```python
awesome = AwesomePlugin()
```

Next we need to create our Menu instance, populate the menu options, any sub-menus and the plugins they call, define the lcd instance to pass to the menu, the instance to call when the menu is idle (or specify none) and the number of seconds to wait before the menu is considered idle as follows.  

```python
menu = Menu({
        'Awesome': awesome,
        'Clock': Clock(),
        'System': {
            'Reboot Pi': GraphSysReboot(),
            'Shutdown Pi': GraphSysShutdown()
            },
        'Status': {
            'CPU-GPU Temp': GraphTemp(),
            'CPU Usage': GraphCPU(),
            'IP Address': IPAddress()
        	}
        }
    },
    lcd,
    Clock(),
    30)
```

This will display a menu with the following options.

* Awesome
* Clock
* System
    * Reboot Pi
    * Shutdown Pi
* Status
	* CPU-GPU Temp
	* CPU Usage
	* IP Address

At this juncture it's worth mentioning that creating the menu as we did above, the menu options may not be presented to the user in the order defined in the menu instance.  To present the menu options in a specific order we need to use a slightly different method described in the *Ordered Menu Example* below.

We can also present a menu to the user and automatically navigate that menu without user input (for example to demo the functionality of our menu) as shown in the *Auto Menu Example* below.  

## Controlling the Menu with the Joystick

Following on with our example of using the menu frawework we need to include some additional code to allow the user to navigate the menu using the dot3k's joystick.

Here we have handlers for when the joystick is pushed up, down, left, right or the button is pressed, each of which perform the corresponding action in our menu instance.  `REPEAT_DELAY` is used along with `jstick.repeat` to keep calling (for example) menu.up repeatedly if the user holds the joystick in the up position waiting 0.5 seconds before each call and providing a ramp mutliplier of 0.9 to the joystick framework's repeat function to speed up the menu navigation the longer we hold the joystick in that position.

```python
REPEAT_DELAY = 0.5

@jstick.on(jstick.UP)
def handle_up(pin):
    menu.up()
    jstick.repeat(jstick.UP, menu.up, REPEAT_DELAY, 0.9)


@jstick.on(jstick.DOWN)
def handle_down(pin):
    menu.down()
    jstick.repeat(jstick.DOWN, menu.down, REPEAT_DELAY, 0.9)


@jstick.on(nav.LEFT)
def handle_left(pin):
    menu.left()
    jstick.repeat(jstick.LEFT, menu.left, REPEAT_DELAY, 0.9)


@jstick.on(jstick.RIGHT)
def handle_right(pin):
    menu.right()
    jstick.repeat(jstick.RIGHT, menu.right, REPEAT_DELAY, 0.9)


@jstick.on(jstick.BUTTON)
def handle_button(pin):
    menu.select()
```

## Drawing the Menu

Finally we need to draw our menu.  To do this we call `menu.redraw()` however because we want to redraw the menu to the lcd periodically as the user interacts with it and as the plugins update the lcd, we call this within a while loop.  We want the while loop to continue indefinitely so we pass it a condition that will always evaluate to True.  In addition to `menu.redraw()` we also use the sleep method to suspend execution for approximately half a second each pass through our while loop.  Without this our while loop would be contantly redrawing the menu likely causing noticeable flickering on the lcd and preventing the user from interacting with the menu.  

```python
while True:
    menu.redraw()
    time.sleep(0.05)
```

## Plugins

To add additional plugins to our menu we simply need to add additional import statements to locate the plugins' python file in the plugins directory then add a menu option that calls it.  We can utilise existing plugins or create our own.

See [dot3k Plugin Guide](../examples/plugins/README.md) for information on creating your own plugins

## Advanced Menu Setup

As we already saw above, an unordered menu is the simplest alternative to instantiate a menu, pass the structure for our menu, the lcd instance, a method to call when idle and the number of seconds before the menu is considered idle.

### Unordered Menu Example

See [menu.py](../examples/dot3k/advanced/menu.py) for an additional example of an unordered menu.

### Ordered Menu Example

To create an ordered menu we could instead instantiate our menu with the following.  Note this would replace the **menu = ...** code earlier in the document. 

```python
menu = Menu(
    None,
    lcd,
    None,
    5)
```

Next we would need to build out our menu structure in the order it will appear:

```python
menu.add_item('Awesome', awesome)
menu.add_item('Clock', Clock())
menu.add_item('System/Reboot Pi', GraphSysReboot())
menu.add_item('System/Shutdown Pi', GraphSysShutdown())
menu.add_item('Status/CPU-GPU Temp', GraphTemp())
menu.add_item('Status/CPU Usage', GraphCPU())
menu.add_item('Status/IP Address', IPAddress())
```

See [orderedmenu.py](../examples/dot3k/advanced/orderedmenu.py) for a further example of an ordered menu.

### Auto Menu Example

Once we've instantiated our menu, added our code to control it with the joystick and our while loop to redraw it we could also add additional code to automatically navigate the menu.  

To do this we could add a `millis()` function to return the current time in milliseconds and an 'advance()'' function to advance through the menu.  Adding the 'advance()'' function to our while loop steps through menu prior to redrawing it.  

```python
def millis():
    return int(round(time.time() * 1000.0))


def advance():
    global last
    if millis() > last + (delay * 1000.0):
        menu.cancel()
        menu.down()
        menu.right()
        last = millis()


last = millis()
delay = 2  # In seconds

while 1:
    advance()
    menu.redraw()
    time.sleep(0.05)

```

See [automenu.py](../examples/dot3k/advanced/automenu.py) for a further example of an automenu.

### Passing `*args` or `**kwargs` When Instantiating Our Menu  

So far when instantiating our menu we've used `*args' with the following structure:

* args[0] - structure for the menu of type `dict` or `OrderedDict()` or None
* args[1] - lcd instance or None
* args[2] - idle handler to call when menu is idle or None
* args[3] - seconds to wait before menu is considered idle

Alternatively or in addition to this we can pass optional `**kwargs` with the following structure:

```python
menu = Menu(
    None,
    lcd,
    None,
    0,
    {'structure'={'Contrast': Contrast(lcd), 'Backlight(backlight)': 2, 'Clock': Clock()}, 'lcd'=lcd, 'idle_handler'=Clock(), 'idle_time'= 5.0, 'input_handler'= '', 'config_file'= 'blah.cfg'})
```

* structure - structure for the menu of type `dict` or `OrderedDict()`
* lcd - instance of lcd for menu
* idle_handler - instance or class to call when menu is idle
* idle_time - number of seconds before menu is considered idle
* input_handler - instance or class to call when requesting user input
* config_file - text file containing menu configuration items located in user's home directory (defaults to dot3k.cfg if not present)

### Menu Configuration File

By default the menu utilises dot3k.cfg located in the user's home directroy to store and retrieve menu configuration items for each plugin.

For example if we're using the Utils module with the Contrast class in our menu we could set the contrast in our configuration file as follows:

```
[Display]
contrast = 45
```

The setup method in the Contrast class attempts to retrieve this value from the configuration file and sets the contrast of the lcd accordingly or defaults to a value of 40 if no configuration entry is found.

#### Storing a menu option configuration item in the configuration file

```
set_option(self, section, option, value)
```

Sets a value for a configuration option within a specific section of the configuration file for a plugin.

* self - instance of plugin
* section - section name within configuration file (e.g. "[Display]")
* option - configuration option being set (e.g. "contrast")
* value - value to set for configuration option (e.g. "45")

#### Retrieving a menu option onfiguration item from the configuration file

```
get_option(self, section, option, default=None)
```
Gets a value for a confiuration option from a specific section of the configuration file for a plugin returning the default if no entry exists.

* self - instance of plugin
* section - section name within configuration file (e.g. "[Display]")
* option - configuration option being retrieved (e.g. "contrast")
* default - None or a value to use in the absence of an entry in the configuration file (e.g. "40")
