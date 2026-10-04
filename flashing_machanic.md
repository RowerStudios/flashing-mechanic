# Flashing Mechanic Implementation explained

## contains 6 parts consisting of 2 mechanics explained briefly

# 1. hitbox mechanic

For BillboardGui entities like `craven`, flashing can detect the mouse position using:

```lua
local mouseX = UIS:GetMouseLocation().X
local mouseY = UIS:GetMouseLocation().Y
```

when the player clicks/parries, a client-sided `parry` remoteEvent fires.
the entity checks if the mouse position is inside its hitbox.

if it matches:

```lua
-- successful parry
```

otherwise, do nothing or optionally fire `parry_fail`.

---

# 2. required measures for hitbox mechanic

make a new remote event called parry (can be anywhere)

Example:

```lua
local hitbox = xx
local parry_remote = game.ReplicatedStorage.parry < - can be anywhere

parry_remote.OnServerEvent:Connect(function(plr, target)

    if target == entity and is_inside_hitbox(plr) then
        -- successful parry
    end

end)
```

the important part is that the client only sends the target.
and then the entity/server decides whether the parry is actually valid.

---

# 3. convex cone detection on both billboard and 3d (my recommended choice)

Instead of requiring the player to click the exact center of an entity, cast a cone from the mouse/camera direction.

The cone uses:

* origin
* direction
* maximum distance
* cone angle
* entity hitbox

For 3D entities:

```lua
local to_entity = entity_position - camera_position
local direction_to_entity = to_entity.Unit
local dot = ray.Direction:Dot(direction_to_entity)
local threshold = math.cos(math.rad(12)) < - cone radius

if dot >= threshold then
    --TODO: parry success
end
```

for billboard entities, check whether the mouse position is inside the projected BillboardGui hitbox instead.

both methods should return the same result:

```lua
local target = detect_parry_target()

if target then
    parry:FireServer(target)
end
```

---

# 4. final parry validation (optional but safer)

detection and validation should be separate.

The client says:

```text
"yo ima try to parry this entity"
```

The server/entity checks:

```"text
does the entity exists? < - part 5.1
u sure you are hitting the entity? < - part 5.2
did the remote event gets sent unkowingly? < - part 5.3
did u parry when the window was inactive?" < - part 5.4
```

ultimately avoiding exploiters

Then:

```lua
if valid_target
and parry_active
and inside_hitbox then

    --TODO: successful parry

else

    --TODO: success / fail

end
```

the flash itself controls the parry window :

```lua
entity.parry_active = true

task.delay(parry_window, function()
    entity.parry_active = false
end)
```

so its gonna be like :

```text
player clicks
     ↓
detection
     ↓
the cone
     ↓
the event
     ↓
validation
     ↓
success/fail
```

The main idea is to keep **detection**, **parry event**, and **validation** separate so new entities can use the same system without rewriting the whole mechanic.
##

## part 5 is still wip. it shows how the validation can be done in code
in future..

##
the pyramid goes to school and windows broke.
so it cant code :( 