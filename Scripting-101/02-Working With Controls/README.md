# TouchOSC Scripting 101 - Section 02: Working With Controls

[Back](../README.md)

TouchOSC templates are made up of controls. Controls provide for user interaction and can also display useful information to the user.

Controls are accessed with scripting by reference. This reference can be relative to the control's position within a group, or its position within the template itself, or the reference can be absolute, found using a property of the control, such as the control's name.  A future section will focus on control references. To keep things simple for this section, I'll use absolute control references by name. Here we will introduce the '[findByName](https://hexler.net/touchosc/manual/script-objects-control#functions-findbyname)' function.

\** Important **

When using the finByName function, you'll want to turn off (uncheck) the editor setting "Assign new names on copy/paste", otherwise your controls will get assigned new names when copying and pasting. Renaming controls will almost certainly break findByName script references.

## Explaining how findByName works

This function returns a reference to the FIRST control found with the name specified in the function call, starting at the target of the function. If the optional boolean is true, the search will be recursive until a match is found or all controls have been searched.

```lua
myButton2 = root:fndByName('button2', true)
```

The code above starts at the document root and searches recursively for a control named 'button2' and returns its reference if found. If no match is found, the result will be a 'nil' value.

Note: findByName is the simple equivalent of the more general findByProperty function. We'll explore that in a later section when we deep dive into control references.

## Get the value of a control from another control

For this example, we'll have a button do something when pressed, based on the state or value of another button. We will have 'button1' do something based on the state of 'button2'.

From the previous section, we know to use the onValueChanged callback function to handle button presses.

```lua
-- Code for 'button1'

-- First, get our refernce outside of any function
-- since we only need to do this once, not on every
-- callback. 
myButton2 = root:findByName('button2', true)

function onValueChanged(key)

end
```

Now, let's use the value of 'button2' to decide what to do when 'button1' is pressed.

```lua
-- Code for 'button1'

-- First, get our referemce outside of any function
-- since we only need to do this once, not on every
-- callback. 
myButton2 = root:findByName('button2', true)

function onValueChanged(key)
  if key == 'x' and self.values.x == 1 then
    -- Get the value of 'button2' and make 
    -- a decision based on the value. 
    if myButton2.values.x == 0 then
      print('The value of button2 is 0 (not pressed)')
    elseif myButton2.values.x == 1 then
      print('The value of button2 is 1 (pressed)')
    end
  end
end
```

## Set the value of a control from another control

In much the same way we can get(read) the value of another control, we can also set(write) that value. The tricky part about setting a control's value is that it will also trigger the control's value change messages and functions. We'll discuss how to handle that as well.

For the next example, we want 'button2' to be set to the opposite state of 'button1' whenever 'button1' is pressed.

```lua
-- Code for 'button1'

-- First, get our referemce outside of any function
-- since we only need to do this once, not on every
-- callback. 
myButton2 = root:findByName('button2', true)

function onValueChanged(key)
  -- Set the value of 'button2' opposite of 'button1'
  if key == 'x' then
    if self.values.x == 1 then
      myButton2.values.x = 0
    elseif self.values.x == 0 then
      myButton2.values.x = 1
    end
  end
end
```

This seems like a good place to introduce the 'init' function. 'init' is a special callback function that is called once each time a template is played. An 'init' function can be defined at the document root level and also for each control, so the script code of each control can handle its own initialization.

For the example above, we'll use the 'init' function to make sure 'button2' is set to the opposite of 'button1' every time the template is played.

```lua
-- Code for 'button1'

-- First, get our referemce outside of any function
-- since we only need to do this once, not on every
-- callback. 
myButton2 = root:findByName('button2', true)

function init()
  -- Are we starting out set to 1?
  if self.values.x == 1 then
    myButton2.values.x = 0
  -- or are we starting out set to 0?
  elseif self.values.x == 0 then
    myButton2.values.x = 1
  end
end

function onValueChanged(key)
  -- Set the value of 'button2' opposite of 'button1'
  if key == 'x' then
    if self.values.x == 1 then
      myButton2.values.x = 0
    elseif self.values.x == 0 then
      myButton2.values.x = 1
    end
  end
end
```

Finally, let's talk about how to keep a control from triggering a value change when we set its value. For this example, we need to add a bit of code to 'button2'. When a value change occurs, we can check to see if 'button2' was touched to determine if it was pressed or released or if its value was changed by another control.

```lua
-- Code for 'button2'

function onValueChanged(key)
  if key == 'x' then
    -- Check to see if button was touched
    if self.values.touch == true then
      print('button2 was pressed or released')
    else
      print('button2 value changed by another control')
    end
  end
end
```

## Set a label's text and color from another control

Labels are great for communicating useful information to the user. Sometimes we want to have a label reflect the state of another control.

In this example, we'll set the text and background color of a label based on the state of a button. 'text' is a value of a label and 'color' is a property of a label.

Refer to the following link for all of the [control properties and values available to scripting](https://hexler.net/touchosc/manual/script-properties-and-values).

Just like we've done in the previous examples, we need to get a reference to the label we want to change.

```lua
myLabel = root:findByName('label1', true)
```

Then it's simply a matter of setting the text value and background color property whenever the value of a button changes. Just like in the previous example, we'll use init to match the label with the initial button state so they start out in sync.

```lua
--Code for 'button1'

-- Just for convenience, define our
-- colors in one place and use variables
white = 'FFFFFFFF'
black = '00000000'
red = 'FF0000FF'
green = '00FF00FF'

-- get a reference to our label
myLabel = root:findByName('label1', true)

function init()
    -- Are we starting out set to 1?
  if self.values.x == 1 then
    myLabel.values.text = 'Pressed'
    myLabel.color = green
  -- or are we starting out set to 0?
  elseif self.values.x == 0 then
    myLabel.values.text = 'Unpressed'
    myLabel.color = red 
  end
end

function onValueChanged(key)
  if key == 'x' then
    if self.values.x == 1 then
      myLabel.values.text = 'Pressed'
      myLabel.color = green
    elseif self.values.x == 0 then
      myLabel.values.text = 'Unpressed'
      myLabel.color = red 
    end
  end
end
```

To close out this section, let's also change the color of the label text. To do this we change the 'textColor' property of the label.

```lua
myLabel.textColor = white
```

While we're at it, let's change the 'button1' color as well.

```lua
self.color = green
```

Now, for the completed script.

```lua
--Code for 'button1'

-- Just for convenience, define our
-- colors in one place and use variables
white = 'FFFFFFFF'
black = '00000000'
red = 'FF0000FF'
green = '00FF00FF'

-- get a reference to our label
myLabel = root:findByName('label1', true)

function init()
    -- Are we starting out set to 1?
  if self.values.x == 1 then
    myLabel.values.text = 'Pressed'
    myLabel.color = green
    myLabel.textColor = white
    self.color = green
  -- or are we starting out set to 0?
  elseif self.values.x == 0 then
    myLabel.values.text = 'Unpressed'
    myLabel.color = red
    myLabel.textColor = black
    self.color = red
  end
end

function onValueChanged(key)
  if key == 'x' then
    if self.values.x == 1 then
      myLabel.values.text = 'Pressed'
      myLabel.color = green
      myLabel.textColor = white
      self.color = green
    elseif self.values.x == 0 then
      myLabel.values.text = 'Unpressed'
      myLabel.color = red
      myLabel.textColor = black
      self.color = red
    end
  end
end
```

In the next section we'll look at how to set values and properties of controls based on data received from MIDI and OSC messages. We'll also use values of controls in messages.
