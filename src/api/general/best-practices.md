## Shared State
Lua uses a shared state across all scripts. Use `local` variables *and* `local` functions (not necessary for local tables) so as to not cause conflicts with other scripts.

### Example
```lua
-- explicit local variables
local foo = {}

-- implicitly local due to local foo
function foo.baz()
    return "baz"
end

-- explicitly local because not part of a table
local function bar()
    return "bar"
end
```

## Hooking performance considerations
Using `sdk.hook` is very useful. However, this comes with a few things to take note of.

When you are hooking something like an `update` function that runs every frame, for every entity in the game, this can cause performance problems if you are calling a lot of functions within the hook.

A way to work around this, if it's possible in your script, is to stagger the function calls across multiple ticks. Also cache the methods and fields. [A real world example can be found here](https://github.com/GreenComfyTea/MHR-Overlay/pull/19).

You can also create a plugin that will run the heavy code natively, and then call it from Lua.

## Method calls & field access performance considerations
Another very useful pair of functions is `object:call` and `object:get/set_field`. Whenever these functions are called, they perform a hashmap lookup. These can be slightly more efficent if you cache off the method and field definitions.

Example implementation
```lua
local method1 = sdk.find_type_definition("Foo"):get_method("Bar")
local field1 = sdk.find_type_definition("Foo"):get_field("Baz")

re.on_frame(function()
    local some_object = sdk.get_managed_singleton("Qux")
    local bar_result = method1:call(some_object, 1, 2, 3)
    local baz_result = field1:get_data(some_object)
end)
```

## Plugins
If there's something you find you can't do without native code, Lua can `require` native DLLs. Native plugins are also an option.

Plugins can also be used to run heavy code that would otherwise be very slow in Lua. Parts of your script can still run in Lua, but you can expose APIs from your plugin that would run the heavy parts natively.

## Modules
Lua can require modules. Modules are a way to organize your code.

### Module1.lua
This goes in a subfolder inside the autorun folder.

```lua
local test = {}

function test.foo()
    print("foo")
end

return test
```

### Main.lua
This goes in the autorun folder.

```lua
local module1 = require("subfolder/Module1")

-- Now you can call module1.foo()
module1.foo()
```