# eco-fps-walker
Artificial intelligence for characters moving in a 3D terrain in Godot.

As long as there is a map with collision feature (physics body), the bot implementing the AI will be able to move in this map.

## Features
### State machine
The bots implement a state machine that allows to configure easily complex behaviours with a simple declarative code.

### Travel with dynamic waypoints
The AI can use a navigation node to calculate an optimal path to a destination point. But rather than following each point of the calculated route, it will take the first waypoint to reach and move to this destination. When the waypoint is reached, a new route is calculated. And so on until the real destination point is reached. When the AI reaches the last waypoint, it will directly go to the final destination. That means it will chase the target even if it's moving.

This implementation allows the bot to follow a moving target, like a human player. It also take in account difficult paths, like stairs and U-turn around a wall, which can cause problems when the bot has a large body. It is also tolerant to strange paths of the navigation mesh (won't be needed if the route calculation is improved in the future).

### No need for a navigation mesh
The bot can improve its path by using a navigation node but it's not mandatory. In the absence of a navmesh the target goes directly to the target. Of course it won't be able to avoid dead ends. But if there's no obstacle that requires path finding, this mode is very functionnal and usually it's enough.

### Avoid walls and holes
Wether the bot uses a navigation node or not, it will avoid holes and walls. Thanks to a concept of watching his steps the bot can detect if the path is clear. It uses a stereo detector which tells him if the hindrance is at its left or right, or both. The bot will then turn around and try another direction.

## Types of bots
There are 3 kinds of AI:
* walker_core : It's the core of the bots. This script allows a bot to go from point A to point B, with or without the help of a Navigation node. If there is a navigation node, the bot will follow the path with waypoints and try to avoid being stuck in funny situations generated by the navigation route calculation, such as extra waypoints in opposite direction. If there's no navigation node, the bot will move directly to the target.
* simple_bot : Extended version of the walker_core with the ability to attack a target. If the target dies, the bot will sleep until it gets a new target (it won't search by itself).
* basic_guard : Extended version of the simple_bot with the feature to scan an area in order to acquire a new target. When it has no current target, it will randomly rotate but not move, until a new target is in sight. This bot has a parameter 'target_group' which tells which group of nodes to search when looking for a new target. 

## Demos

[![Demos part 1](http://img.youtube.com/vi/FJbStSlSJgY/0.jpg)](https://www.youtube.com/watch?v=FJbStSlSJgY)
[![Demos part 2](http://img.youtube.com/vi/pRPh9VJlANY/0.jpg)](https://www.youtube.com/watch?v=pRPh9VJlANY)
[![Demos part 3](http://img.youtube.com/vi/iClqDfMWYWk/0.jpg)](https://www.youtube.com/watch?v=iClqDfMWYWk)

older demo

[![Demos part 3](http://img.youtube.com/vi/Kj0Vu-ShVWQ/0.jpg)](https://www.youtube.com/watch?v=Kj0Vu-ShVWQ)

## Installation
Either install from the asset store or copy the content of the addons folder in your project root folder.
The samples folder can be deleted.

## Usage
The usual way to use the bot is to add a node in the scene from one of the available bot nodes in /addons/eco.fps.walker/. You must select the .tscn node file.
Then you add as child of the node a visual mesh or node. The mesh will automatically move and rotate with the bot.

### Animation
It is possible to animated the mesh using information from the bot, like current speed or current state, by adding signal listeners.
The walker_core has 2 signals:
* walk_speed_changed : gives the current speed of the bot. The parameter is the speed of the bot.
* action_changed : sent when the bot's state changed. The parameter is the new state.

### Communication between bot and children
When the bot needs an event from the child, the recommended way is to implement a signal in the child node and extend the bot script to handle the signal event. For instance, a simple bot that attacks a target and it has an attack animation that must be able to send a signal to the bot for when the animation reaches the moment where the attack reaches the target. Also, when the attack animation ends, the bot must know that the attack ended and change its state to the next one.

## Add features
Those scripts are usually too basic to be used as it is in a game. They are meant to be extended, like in demo 8 where a simple patrolling behaviour is implemented while still attacking any target at sight. You can either use the simple_bot or basic_guard as a base for your own code, or start with walker_core to implement from almost scratch your own bot.

### State machine
Thanks to the state machine implemented in the core, the proper way to implement a bot is to use states rather than variables with conditions if-then-else. But it's not mandatory if your need is too specific to be implemented in the state machine. And you can add another state machines for your own usage if you feel the need to.

#### Declaration
The standard states are declared in one field called action_states, defined like this example:

const action_states=["sleep","move","turn","wait","my_state1","my_state2","some_state"]

It's a required step, unless you are happy with the default states.

#### Configuration
After the states are declared, it is required to configure them. The scripts have methods you can override to make your own configuration.

If you use the walker_core as base, you have only the method _init_fsm to override.

If you use basic_bot or basic_guard, you don't need to override _init_fsm but rather 3 sub-methods:
* _init_fsm_states : It creates the states and organize the groups.
* _init_fsm_move : It creates the links between states when the bot is moving or idling.
* _init_fsm_attack : It creates the links between states for when the bot is attacking a target.

Check the documentation of the state machine and the source code of the scripts for further information.

#### Change the state programmatically
The state machine usually does the update of the current state automatically with the given rules. But sometimes it's much easier to set a specific state from the script rather than the state machine rules.
In that case, you call the method:
   fsm_action.set_state(new_state_name)
where the fsm_action is the instance of the state machine, as defined in walker_core.

The signal action_changed will be triggered.

Beware that a wrong state name will cause an error and make the bot unstable.

## API
### walker_core
*Extends RigidBody*
Field | Type | Default | Description
:---: | :---: | :---: | ---
body_radius | float | 0.8 | Radius of the capsule being the collision body of the bot.
body_height | float | 1 | Height of the capsule being the collision body of the bot.
leg_length | float | 0.3 | Length of the leg of the bot. Since the leg is a sphere, it's its radius.
sight_height | float | 2 | Height of the rays detecting the holes and collisions. Setting it higher means it can see the holes further, but it would detect false collisions with the ceiling if set too high. Too low and it won't see holes in time.
walk_speed | float | 3 | Maximum speed
dynamic_speed | bool | false | Activate the system to reduce speed when the bot is avoiding collision.
max_speed_accel | float | 1.01 | d 
turn_speed_deccel | float | 1 | d
max_accel | float | 0.02 | d
air_accel | float | 0.05 | d
debug_mode | bool | false | d
debug_path | NodePath | null | d
debug_wpt | NodePath | null | d
target | NodePath | null | d
navigation | NodePath | null | d



### basic_bot

### guard_bot

## Tips and pitfalls

### Debug mode
The script contains a debug mode that display the calculated route and the current waypoint. See the samples for further information.
This can help the level designer to optimize the level where the bot can have trouble finding its way.

### Stairs and slopes
The bot is technically unable to climb real stairs. You must make the collision shape of the stairs like slopes, otherwise it won't work.

The hole detection of the bot is made for a certain angle. If the slope is too steep, it will be considered as a hole.
Also, since the bot is a rigid body, it might bounce if it walks down a long steep slope.

### U-turn in stairs
This case is definitely the worst scenario case for the bot. Without path finding it's almost impossible that the bot reaches the target. And even with a navigation node, it will struggle to turn around a corner in U-turn. If there's no wall, there is even a risk, though very small, that the bot falls of the stairs.

### Speed
The speed factor, which is a parameter, has a great impact in the collision detection quality. If you set a high speed to the bot, it won't be able to avoid collisions and holes as well as when it's walking at lower speed. Especially near a cliff or of a bridge, the risk to fall is related to the speed. It has also to do with the inertia of the rigid body of the bot, where higher speed means longer time to brake.
One way to avoid that would be to extend the script to dynamically change the speed when the bot is near a hole. Usually the problem arises when the bot goes in a U-turn at high speed.

### Strange behaviors
Sometimes the bot can suddenly backtrack or turn in circles for a little moment. Those are the limitations of the AI implementation and the path finding of Godot. It's optimized for performance but at the cost of some glitches in the path finding. And in rare special cases the bot can fall down a cliff. To reduce the risk that happens you can either adapt the level design or extend the script to fix this particular case.

In other words you can be 'almost' certain that the bot won't get eternally stuck somewhere, or fall down a hole, but it's hard to say how much time it will take. However its path is very predictable, as the bot will always take the same route. That helps a lot for test cases.
