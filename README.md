# ocrsdriver
OpenComputers + Refined Storage = :heart:

Allows OpenComputers to access Refined Storage grid nodes via Adapters.
This is for Minecraft 1.10.2. Refined Storage for 1.11.2 has this functionality included by default.

## Known Issues

- This is currently unrestricted, i.e. you can export items and fluids using just a cable - and fast.

## Available Commands

| Method                                      | Description                                                      |
| ------------------------------------------- | ---------------------------------------------------------------- |
| isConnected()                               | Returns whether the grid node is connected to a network          |
| getEnergyUsage()                            | Returns the energy usage of the whole network in RS/tick         |
| getItems()                                  | Returns a list of all itemstacks stored in the network           |
| getItem(itemstack, cmpMeta, cmpNbt, cmpOre) | Returns a stack in the network matching the given itemstack      |
| extractItem(itemstack, amount, side)        | Extracts the given amount of itemstack to the specified side and returns the number of transfered items. |
| getFluids()                                 | Returns a list of all fluid stacks stored in the network         |
| getFluid(fluidstack)                        | Returns a stack in the network matching the given fluidstack     |
| extractFluid(fluidstack, amount, side)      | Extracts the given amount of fluid to the specified side and returns the amount of millibuckets transfered. |
| getTasks()                                  | Returns a list of crafting tasks the system has currently queued |
| getPatterns()                               | Returns a list of all crafting patterns in the network           |
| hasPattern(itemstack)                       | Returns true if the system has a pattern for the given itemstack |
| getMissingItems(itemstack, amount)          | Returns a list of all items missing to craft the given itemstack in the given amount |
| craftItem(itemstack, amount)                | Requests crafting of the specified item in the specified quantity |
| cancelCrafting(itemstack)                   | Cancels all crafting operations for the given itemstack and returns the number of cancelled tasks |

## Examples

### Drop all items you got more than 8192 of

Place a Trash Can below an interface connected to an OpenComputers Adapter.
```lua
local component = require("component")
local sides = require("sides")

local rs = component.block_refinedstorage_interface

local limit = 8192
local side  = sides.down

for i,stack in ipairs(rs.getItems()) do
	while(stack.size > limit) do
		local dropped = rs.extractItem(stack, stack.size - limit, side)
		stack.size = stack.size - dropped
		if(dropped < 1) then
			break
		end
	end
end
```

### Keep 64 torches and levers in your crafting system

Don't forget to teach a crafter the recipe for torches and levers
```lua
local component = require("component")
local sides = require("sides")

local rs = component.block_refinedstorage_interface

local targetAmount = 64
local items = {
    {name = "minecraft:torch"},
    {name = "minecraft:lever"}
}

while(true) do
    for i,stack in ipairs(items) do
        if(rs.hasPattern(stack)) then
            local rsStack = rs.getItem(stack)

            local toCraft = targetAmount;
            if(rsStack ~= nil) then
                toCraft = toCraft - rsStack.size
            end

            if(toCraft > 0) then
                rs.craftItem(stack, toCraft)
            end
        else
            print("Missing pattern for: " .. stack.name)
        end
    end

    os.sleep(5)
end
```
