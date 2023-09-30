# Forge
Powerful class-based systems in Luau

# Acquiring
To begin using Forge, you must first acquire the 3 base components:

```lua
local forge, const, sojourn = require(forgeModule):get_comp()
```

# Overview

To create a class, use the `forge` keyword followed by the name of the class and its components, like this:

```lua
local MyClass = forge "MyClass" {
  Hello = "Hello, World!"
}
```

To handle a newly constructed class member, call method `init` (while passing the construction function) of the respective class. Do note that `init` will return `self`, so you can use it while initializing the class. There can only be one `init` method per parent-class, by the way.

```lua
--// method 1

local MyClass = forge "MyClass" {
  Hello = "Hello, World!"
}: init(function(self)
  print ("New member of MyClass was created! :", self)
end)

--// method 2

local MyClass = forge "MyClass" {
  Hello = "Hello, World!"
}

MyClass:init(function(self)
  print ("New member of MyClass was created! :", self)
end)

-- .. either one works
```

To add methods to a class, you'd follow the same prodecure as normally adding a method to a table:

```lua
function MyClass:AddFive(v)
  return v + 5
end
```

To create a new member, you can either directly call the class, or you can call the function `new` of the class.

```lua
local Member1 = MyClass()
local Member2 = MyClass.new()
-- .. either one works
```

Members have two starting methods — `is_a` and `of`, and the former returns whether or not the member is a member of a class with the given name, while the latter returns whether or not the member's parent-class descends from the class with the given name.


# Inheritance

Inheritance is simple, because it is completely handled by Forge. By the way, inheriting a class will copy all of its fallbacks (I'll get into that later), metamethods, and init-methods (constructors). However, the inheriting-class will not inherit any properties (not methods!) made outside the base table given on the initialization of the class. To briefly go over how inheriting init_methods work, essentially, a new member of the inherited-class will first go through all the init-methods of the inherited class, then it will go through its parent-class's init method (if there is any). 

Heres an example of how to inherit:
```lua
local SomeClass = forge "SomeClass" {
  Time = 50
}

SomeClass.SomeProp = 80; function SomeClass:Do()
  print "Hello, World!"
end

local NewClass = forge (SomeClass) "NewClass" {
  OtherProp = 100
} -- Inherits from SomeClass

--/ ...

local Member = NewClass.new()

print(Member.Time) -- "50"
print(Member.SomeProp) -- "nil"
print(Member.OtherProp) -- "100"
Member:Do() -- "Hello, World!"
```

Heres an example of how init-methods are inherited:
```lua
local a = forge "a" {}: init(function()
  print "a"
end)

local b = forge (a) "b" {}: init(function()
  print "b"
end)

local c = forge (b) "c" {}: init(function()
  print "c"
end)

local member_of_c = c.new() -- This should print a, b, and c in that given order
```

# Utilities

Along with the `forge` keyword, there is also `const` and `sojourn` — const creates a new constant value and sojourn creates a new dynamic value. To create a constant, simply call const with the given value. to create a designation (sojourn), create it through sojourn, passing a function that returns the value. Both of these values will only work if it is a DIRECT member of the base class table that is passed!

Here's an example of both of their usages:
```lua
local Since = os.time()

local Clock = forge "Clock" {
  Time = sojourn (function()
    return math.round( Since - os.time() ) -- In this case, the time will change in accordance to the actual time.. 
  end)
  
  Owner = const "Mlgisbetter"
}

Clock.OtherProperty = Const "Hi" -- This will not work! Instead of the value given in the constant, it will return the constant's metadata.

-- / ...

local MyClock = Clock.new()

print(MyClock.Time) -- About 0
task.wait(3)
print(MyClock.Time) -- About 3

print(MyClock.Owner) -- "Mlgisbetter"
print(MyClock.OtherProperty) -- Not "Hi"; instead, the metadata of that constant, so a weird looking table

MyClock.Owner = "Mlgsucks" -- Will throw an error!
```

Classes, alongside `init`, have the methods `metamethod` and `depend`. `metamethod` is used to add metamethods to the class. These metamethods, however, aren't applied to the class, but rather new members of the class. There are a few metamethods that cannot be modified through `metamethod` — `__index`, `__newindex`, and `__tostring`. the `depend` method creates a new fallback layer for the class's __index method; in a nutshell, it adds a bundle of properties to a class (but note it is a bit more complex than that).

Here's an example of both methods:
```lua
local Class = forge "Class" {}

Class:depend {
  Bee = "b"
}

Class:metamethod("__len", function()
  return 100
end)

--/ ...

local Member = Class.new()
print(#Member) -- 100
print(Member.Bee) -- "b"
```
