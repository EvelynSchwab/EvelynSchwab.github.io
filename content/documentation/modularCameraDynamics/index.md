+++
title = "Modular Camera Dynamics - Documentation"
date = "2024-07-04"
description = "Modular Camera Dynamics (MCD) is an UnrealEngine plugin that reworks and extends the CameraModifier system to allow for more modular approaches to building advanced camera behaviours, with much faster iteration time."
[taxonomies]
tags = ["Unreal Engine", "Unreal Marketplace", "Documentation"]
+++

# Quick Start
## Installing and Activating Modular Camera Dynamics
Install Modular Camera Dynamics for your engine version from the Epic Games Launcher vault, or with an alternate Unreal Engine manager.

While in engine, navigate to Edit ⇾ Plugins and activate Modular Camera Dynamics. (Screenshot)

To see and access plugin content, ensure that plugin content is visible in your content browser. (Screenshot)

It is highly recommended that, should you wish to tweak the behaviour of the included camera data examples, you first duplicate the preset to your current project's `/Content` directory. This prevents any changes you make from influencing the plugin in all projects for that version, and will ensure that you do not overwrite your work if you update Modular Camera Dynamics.

## Setting up Your Player Camera Manager
### Creating your Player Camera Manager
Modular Camera Dynamics requires a player camera manager that inherits from `CDPlayerCameraManager` to manager camera behaviours.

To set up your Player Camera Manager, you will need to create a new Blueprint (or C++) class of type `CDPlayerCameramanager`. (Screenshot)

If you already have a Player Camera Manager, you can re-parent it by opening it and navigating to Class Settings → Parent Class.

### Implementing your Player Camera Manager
The Unreal Engine Gameplay Framework requires that you specify the intended Player Camera Manager through your Player Controller.

If you do not already have a player controller, you must set one up. (either add a link, or explain the process)

In your Player Controller, set the Camera Manager property to your Player Camera Manager.

## Adding Camera Data
In Modular Camera Dynamics, camera behaviours are dictated by camera stacks, within a Data Asset `CDCameraData`.

MCD includes a number of camera stack example instances, which you can use to test the implementation of your Player Camera Manager. If the player camera manager does not have any camera data specified, it will fall back to the default camera behaviour (such as a spring arm).

There are two options for adding new Camera Data:
1. Set the `DefaultCameraData` property within your Player Camera Manager (screenshot).
2. Call the `AddCameraData()` function, specifying the camera data to add. This is also how you will add camera data during gameplay (screenshot)

## Using the Modular Camera Dynamics Editor
(Add to this section once the editor has an engine button)

# Key Terms and Concepts
- **Camera behaviour:** A single action or reaction that a camera can take, broken down to the simplest form it can take as a standalone element. Some examples of this would be:
	- The base position of the camera being drawn from the player's location
	- The camera being offset behind the player (as in a third person camera)
- Camera mode: A collection of camera behaviours that come together to form a greater whole. An example would be:
	- A third person over-the-shoulder camera, combining these behaviours:
		- Camera base rotation tied to the player controller's control rotation
		- Camera base position tied to player's eye position
		- Camera offset 70 units in its local Y axis
		- Camera offset -700 units from its facing direction
		- Trace from to the player's head to check for collisions, and offset the camera if an obstruction is found
- **Modular camera modifier:** A camera modifier that exhibits a single potential camera behaviour in a way that would be combined with other modular modifiers in a camera stack to create a camera mode.
- **Camera stack:** A set of camera modifiers grouped together to create a camera mode, or to add a new element to an existing mode. An example of the latter would be:
	- An additive enemy lock-on mode:
		- Camera rotation overwritten to be facing a point on another actor
		- Camera offset 100 units in the global Z

The order of camera behaviours is important and does not need to be the same as the order they were in the stack, as will be shown below.

Putting together the above third person and lock-on examples, we might get the following. *This example is formatted as **Camera Stack** - Camera Modifier*:
1. Enemy Lock On - Camera rotation overwritten to face a point on the enemy mesh
2. Third Person - Camera base position sources from player eye height
3. Third Person - Camera offset 70 units on its Y axis
4. Third Person - Camera offset -700 units in its X axis\
5. Enemy Lock On - Camera position offset 100 units in the global Z
6. Third Person - Trace from the player's head to the camera and offset the camera's position to any obstruction.

It's important to note that the enemy lock on rotation behaviour has to come before the other behaviours, as it will change the rotation of the camera, which is used to calculate positions in later behaviours.

# Included Behaviours
The following behaviours are included in MCD. Each have a wide variety of exposed parameters for adjusting their behaviours.
#### Position Base
Sources the base position of a camera from one of the following:
- A global position
- A position local to the player
- A position of a socket on the player
- The player's eye height
#### Position Offset
Offsets the camera in global and/or local space by a specified amount.
#### Position Distance
Offsets the camera in its local X axis by a specific amount.
#### Position Lag
Smooths and interpolates the position of the camera in varying ways.
#### Position - Velocity Offset
Offsets the position of the camera based on the velocity of the player, with variable interpolation parameters.
#### Position - Dynamic Z
Adjusts the Z position of the camera based on a variety of factors, including the current distance from the previous position behaviour, if the player is grounded and the distance a player has travelled since leaving the ground.
#### Rotation Override
Overrides the rotation of the camera to a specified Euler rotation, or aims the camera at a specific location, actor, component or socket.
#### Rotation from Velocity
Gradually reorients the camera's rotation based on the velocity of the player and the time since the camera's rotation has been changed externally.
#### Sweep Basic
Traces from a specified position to the camera's position, offsetting the camera's position to that location as it finds an obstruction.
#### FOV Adjust
Adjusts the field of view by a given operation and magnitude.
#### FOV from Pitch
Adjusts the field of view based on the camera's pitch.

# Classes
There are three primary classes added by MCD:
- Dynamic Player Camera Manager - `UCDPlayerCameraManager`
	- Extends and adds logic to support managing instanced camera modifiers.
- Camera Stack - `UCDCameraData`
	- Data asset container for instanced camera modifiers. Used to group together a set of camera modifiers to make a more wholistic 'camera mode'
- Instanced Camera Modifier - `UCDCameraModifier_Instanced
	- Implementation of the Modular Camera Modifier concept, designed to be set up in a Camera Stack Data Asset.
	- Individual camera behaviours should be implemented in a single Instanced Camera Modifier.

# Creating New Camera Stacks
Start by creating a new data asset of type `CDCameraData`.

Within this class, you will find an array of Instanced Camera Modifiers, which you can add new modifier instances to. Selecting from the drop-down, you can choose any instanced camera modifier subclasses. 

Because these objects are instanced inline, you can customise their values here without needing to create subclasses for each variation of the camera behaviour. You can also adjust their values during play-in-editor sessions, and those changes will update in real-time (adding new modifiers or repositioning them will not update during PIE).

The priority of modifiers is dictated by the order the camera data is added, and the order of the modifiers in said camera data.
If you need to customise the priority of a modifier (for example, a trace/sweep modifier that should take place after *all* other modifiers) then you can check the `CustomPriority` Boolean and set a priority value.
Generally speaking, modifiers that will influence the rotation of the camera should be positioned earlier in the array.

# Debugging Cameras
Camera modifiers have a function `DisplayDebug()` that only runs when the console variable `ShowDebug Camera` is active. The same condition can be leveraged outside this function (and in blueprint) with the function `ShouldDrawDebug()`.
The included modifiers have built-in debug visualisers using this function, and any custom modifiers can also leverage this debugger in Blueprint or C++.

### Creating Custom Camera Behaviours in Modular Camera Modifiers
The Modular Camera Modifier parent class can be subclassed in Blueprint or C++. Overriding the below functions will allow you to set up custom location and FOV overrides. The blueprint override versions of these functions are prefixed with *Blueprint*.

| Function Name              | Description                                                |
| -------------------------- | ---------------------------------------------------------- |
| AddedToCamera              | Triggers when the modifier is added to the camera manager. |
| ModifyCameraBlended        | Modify the transform and FOV of a camera.                  |
| ProcessViewRotationBlended | Modify the delta rotation of a camera.                     |

Generally speaking, rotation modifiers should be handled in `ProcessViewRotationBlended` as it will modify the true control rotation, rather than just set the rotation of the camera (which would be overridden next frame by the control rotation). This doesn’t apply if your camera isn’t using the control rotation, though most camera setups will.

### Managing Camera Stacks and Camera Modifiers During Runtime
There is a property on the player camera manager `DefaultCameraData` that will activate the specified camera data when the camera manger is initialised. This does not need to be used, though, and data can be added manually.
The Camera Manager and Camera Dynamics Function Library include a number of functions for managing the active modifiers on your camera manager.

| Function Name                  | Function Description                                                                                          |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| AddCameraData                  | Adds the specified camera data, and all of its modifiers, to the camera manager                               |
| RemoveCameraData               | Removes any modifiers associated with the specified camera data.                                              |
| GetActiveCameraData            | Returns an array of all active camera data on the camera manager.                                             |
| GetActiveCameraModifierOfClass | Returns the first found active modifier of the specified class, optionally checking for matching tags.        |
| GetAllActiveModifiersOfClass   | Returns an array of all found active modifiers of the specified class, optionally checking for matching tags. |
| GetCameraDynamicsCameraManager | Gets an associated camera dynamics camera manager belonging to a specified player controller.                 |

Getting a reference/pointer to a camera modifier in runtime will allow you to read/write any exposed values on said camera modifier (This *will not* update the data asset, as the object you're modifying is a runtime copy of the instanced object in the data asset. This is by design.).

# Road map
- Disable and enable modifiers without fully removing them
- Set modifier alphas manually without influencing the blend in/out time
- Additional camera modifiers:
	- Whiskered sweep trace
	- Dynamic Y axis (dynamic offset - Horizon Zero Dawn style)
	- Auto-framing of target actors / actor bounds
- Debugging views for cameras, set up to override the camera to an orthographic view so that the camera’s position can be seen from another perspective. This will be especially useful for more advanced trace/sweep modifiers.

#  Known Issues
- Changing parameters via the details panel to their default value, or the default value of their data type, often won't update in-game until PIE is restart. This does not effect parameters set from code.