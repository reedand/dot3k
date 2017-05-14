# Dot3k Plugin Guide

## MenuOption

So, you want to make a plugin, huh?

Dot3k has a handy class, MenuOption, you can inherit from to build menu plugins.

When a menu plugin is run, it'll take over the screen and all joystick input will be passed to it.

### Example

This super simple example is what you need to get something displaying from your plugin on the Dot3k LCD. Easy!
```python
    class MyPlugin(MenuOption):
      def redraw(self, menu):
        menu.clear_row(0)
        menu.write_row(1,'Hello World')
        menu.clear_row(2)
```
Every time redraw is called, the parent menu sends itself and lets you call write_row, clear_row or write_option to draw things to the LCD.

**You should always call menu.clear_row() on any rows you don't use.**

menu.write_row and menu.write_option will always truncate any text that doesn't fit on the screen. You can implement a scrolling marquee, but doing this automatically is on the feature roadmap!

### Methods

MenuOption has the following methods which you can override:

```python
def __init__(self):
	# actions to perform when the MenuOption is initialised

	MenuOption.__init__(self)
```

**__init__** is called once when the instance of MenuOption is first created and is used to initialise the plugin

	* self - instance of the MenuOption plugin which is passed to the method

```python
def up(self):
	# action to perform when joystick up is pressed
	return value
```
**up** is called when the joystick is pressed up, or the menu gets the up command.
		* self - instance of the MenuOption plugin which is passed to the method

```python
def down(self):
	# action to perform when joystick down is pressed
	return value
```
**down** - is called when the joystick is pressed down, or the menu gets the down command.
		* self - instance of the MenuOption plugin which is passed to the method

```python
def left(self):
	# action to perform when joystick left is pressed
	return value
```

**left** - is called when the joystick is pressed left, or the menu gets the left command.
		* self - instance of the MenuOption plugin which is passed to the method

```python
def right(self):
	# action to perform when joystick right is pressed
	return value
```

**right** - is called when the joystick is pressed right, or the menu gets the right command.
		* self - instance of the MenuOption plugin which is passed to the method

```python
def select(self):
	# action to perform when joystick button is pressed
	return value
```

**select** - by default this returns true and causes the menu to exit your plugin, return false to prevent.
		* self - instance of the MenuOption plugin which is passed to the method

```python
def redraw(self, menu):
	#actions to perform every draw pass
```

**redraw** - called every draw pass, you can draw using menu.write_row here, **make sure to clear_row rows you don't use.**
	* self - instance of the MenuOption plugin which is passed to the method

```python
def input_prompt(self):
        """
        Returns the prompt/title for the input plugin
        """
        return 'Prompt:'
```

**input_prompt** - provides the prompt used by the input plugin and presented to the user when requesting input.
		* self - instance of the MenuOption plugin which is passed to the method

```python
def receive_input(self, value):
        """
        The value we get back when text input finishes
        """
        # perform action with value which contains the input entered by the user
```

**receive_input** - called when the user has entered a value with the input plugin
	* self - instance of the MenuOption plugin which is passed to the method [required]
  	* value - input received from the input plugin [required]

**begin** - called when the menu enters your plugin, use to start games, reset things, etc.
		* self - instance of the MenuOption plugin which is passed to the method

```python
def cleanup(self):
        # action to perform when exiting plugin.
```

**cleanup** - called when the menu leaves your plugin.
	* self - instance of the MenuOption plugin which is passed to the method [required]

```python
def setup(self, config):
        MenuOption.setup(self, config)
        #other setup actions to perform here 
```

**setup(self, config)** - the config is passed to this method by default, you should overide using this method
	* self - instance of the MenuOption plugin which is passed to the method [required]
	* config - configuration data [required]

```python
	menu.write_row(row, string)
```
*write_row(row, string)* - writes a string to the lcd at the row it is passed
	* row - defines which row to write to (0..2 where 0 is the first row and 2 is the last row) [required]
	* string -  string of text to write to the lcd [required]

```python
menu.write_option(row=0, text='menu option', scroll=True)
menu.write_option(row=1, text='other')
menu.write_option(row=1, icon='>', text='this', scroll=True)
```

**write_option(row,icon,text,scroll)** - writes a menu option to the lcd 
	* row - defines which row to write to (0..2 where 0 is the first row and 2 is the last row) [required]
	* icon - defines the icon to display with the option [optional]
	* text - string of text to write to the lcd [required]
	* scroll - True or False value determining whether the text should scroll on the lcd [optional - defaults to false]

```python
menu.clear_row(row)
```

**clear_row(row)* - called to clear the specified row on the lcd.  Each pass of the redraw method should call this for any rows not being written to in order to prevent artifacts on the lcd from other plugins, etc.
	* row - defines which row to clear (0..2 where 0 is the first row and 2 is the last row) [required]

### Options

MenuOption also includes the methods set_option and get_option for saving and loading options from the configuration file (dot3k.cfg or other file as defined in Menu instance).

The Menu class automatically saves the current settings to the configuration file when it exits.
