# TouchOSC Scripting 101 - Section 01: The Basics

[Back](../README.md)

TouchOSC scripts are written in the Lua programming language, specifically verion 5.1.

[Lua 5.1 Reference Manual](https://www.lua.org/manual/5.1/)

It should be noted that the scripting engine in TouchOSC has some limitations. The most notable is the fact that global variables are not supported. This is due to the internal architecture of TouchOSC itself. However, there are workarounds; but that's for a later section.

In order for a script to be run within a control, the script code needs to have at least one function, called a 'callback' function. One of the most common callback functions is "onValueChanged". All of the examples in this section use the onValueChanged callback to do something when a button is pressed or released.

## Do something when a button is pressed

When a button is pressed (or released), TouchOSC makes a script callback to the onValueChanged function. A 'key' is passed to the function. This key tells the function what value has changed. In other words, why are we here. For a button, there are two values possible, therefore there are two possible keys. The possible keys are 'x' and 'touch'.

Let's start with entering a function:

```lua
function onValueChanged(key)

end
```

That's it. 'end' is used in Lua to tell the scripting engine we are done with this section of code. Don't worry, we'll get deeper into when to use the 'end' statement.

Next, we need to qualify or test for why we got called:

```lua
function onValueChanged(key)
  if key == 'x' and self.values.x == 1 then
  
  end
end
```

The way to compare two values or strings to see if they are the same is "==".

If we got called with a key of 'x' and also if 'x' is equal to 1 or self.values.x == 1, then do something. Notice that 'end' statement again. That closes out the 'if' section of the code.

Wait? What the heck is self.values.x?  Think of 'self' as 'me' or 'mine' or 'my'. My values,  the 'x' value.

In this case, is my value of 'x' equal to 1? A value of '1' indicates that the button was pressed down. A value of '0' indicates that it was released. More on that in the next example.

Now, let's do something when the button is pressed.

```lua
function onValueChanged(key)
  if key == 'x' and self.values.x == 1 then
    print('Button Pressed') 
  end
end
```

The 'print' function sends information to the Script tab of the Log Viewer in TouchOSC. This is a simple troubleshooting tool that allows us to print messages as well as the contents of variables. We'll use this a lot while testing scripts.

Congratulations. You have written your first TouchOSC script.

## Do something when a button is released

Now, instead of doing something when a button is pressed, say we want to do something when it is released.

Building upon the first example, all we need to do is test for self.values.x being equal to '0' instead of '1'.

```lua
function onValueChanged(key)
  if key == 'x' and self.values.x == 0 then
    print('Button Released') 
  end
end
```

It's that simple.

## Do something when a button is pressed and something else when it is released

Finally, say we want to do something when pressed and something else when released.

```lua
function onValueChanged(key)
  if key == 'x' then
    if self.values.x == 0 then
      print('Button Released')
    elseif self.values.x == 1 then
      print('Button Pressed')
    end
  end
end
```

This example uses the concept of nested 'if' statements. That's where the 'end' becomes important.

Let's break it down.

```lua
  if key == 'x' then

  end
```

The code inside the outer 'if' section will only be executed if 'key' is equal to the string 'x'.

The second or inner 'if' section will test for either self.values.x equal to 0 (released) or to 1 (pressed) and execute the appropriate code.

```lua
    if self.values.x == 0 then
      print('Button Released')
    elseif self.values.x == 1 then
      print('Button Pressed')
    end
```

For more details on the 'if' statement as well as 'if..else' and 'if..elseif..else', please see [Section 2.4.4](https://www.lua.org/manual/5.1/manual.html#2.4.4) of the Lua 5.1 Reference Manual.

## Send a MIDI or OSC Message when a button is pressed or released

Since I'm almost certain you want to do more than print messages to a log viewer, let's look at sending a MIDI or OSC message when a button is pressed.

Starting with the same basic function as above, we simply replace the print function with a sendMIDI fnuction:

```lua
function onValueChanged(key)
  if key == 'x' and self.values.x == 1 then
    sendMIDI({176,0,0})
  end
end
```

This is not a MIDI tutorial so I won't go into the format of a MIDI message other than to say MIDI messages are typically 3 bytes and need to be defined as a table.  In Lua, a table is a collection of values, separated by commas, and enclosed with curly braces.

See [Section 2.5.7](https://www.lua.org/manual/5.1/manual.html#2.5.7) of the Lua 5.1 Reference Manual for more on tables.

To send an OSC message, the function looks almost the same, except we use a sendOSC function instead of sendMIDI. The simple format of an OSC message requires a path and a value. The OSC path is a string while the value could be any valid OSC value type.

```lua
function onValueChanged(key)
  if key == 'x' and self.values.x == 1 then
    sendOSC('/test', self.values.x)
  end
end
```

Finally, let's send a MIDI message when a button is pressed and a different message when it is released.

For this example we'll use NOTE_ON and NOTE_OFF.

```lua
function onValueChanged(key)
  if key == 'x' then
    if self.values.x == 0 then
      sendMIDI({128,60,0}) -- Note Off, Middle C
    elseif self.values.x == 1 then
      sendMIDI({144,60,127}) -- Note On, Middle C
    end
  end
end
```

That's enough for now. In the next section we'll start to explore controls and how to work with controls in scripts.
