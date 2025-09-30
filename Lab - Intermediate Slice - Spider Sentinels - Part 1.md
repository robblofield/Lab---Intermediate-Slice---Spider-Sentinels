# Lab - Intermediate Slice - Spider Sentinels Part 1 - Game Dev
*In this lab we will begin to create a procedural animation system for insect/spider-like creatures that can be used to drive an existing 3D armature via code*

![Spider Sentinels Render](<spider sentinels render.png>)

### Additional Notes & Documentation
### SpiderMechEnemy.fbx
Your lab team will be creating a custom spider enemy for the lab, to implement the procedural animation on the finished design, ensure it has a standard armature (no ik or constraints) from Blender as an .fbx into Unity and closely look at the bones in the armature to make sure you are working with the correct selection. For the initial stages of this lab this you should use the provided SpiderMechEnemy.fbx uploaded to canvas as a template.

### Refactoring/Temp Code
This lab features some instances of code which will be refactored or completely overwritten in the following sessions, in some cases it is not demonstrating best practices, but instead focusing on fast implementation for testing so that we can progress forward. Feel free to refactor as you go, but make sure you remember what you chnaged so that the subsequent lab notes are mostly in line with your code base.

### Documentation
[Unity Documentation - Animation Rigging](https://docs.unity3d.com/Packages/com.unity.animation.rigging@1.3/manual/index.html)


### Section 1 - Setting up IK in Unity
* Install the Unity `Animation Rigging` package to your project
* Select the .fbx in your scene and go to `Top Menu > Animation Rigging > Rig Setup`
> This will add a new Empty to the scene that will hold the Unity rig, and it will pre-populate `animator` and `rig builder` components onto the parent object.
* Rename the Unity rig to something suitable, such as `LegIK`
* `Add a child to LegIK` and name it `Bone Constraint` (or similar)
* Add a `Two Bone IK Constraint` component to `Bone Constraint` and set it up with the following parameters (The names used are from the sample model created by Rob, but your custom models may differ)
  ```
  Weight = 1
  Root = Select the top bone in your leg assembly (FrontShoulder.L)
  Mid = Select the highest bone in the leg assembly you want to control (FrontLegUpper.L)
  Tip = Select the very last bone in the assembly (FrontLegLower.L_end)
  Source objects > Target = Bone Constraint
  ```
![Two Bone IK Params](image-3.png)

* In the scene view, expand the `Animation Rigging floating window` and set the shape to a `sphere` and `size to 1` - this will give us a visual indicator of the IK target
  
![Creating an IK Indicator](image-4.png)

* Align the "Bone Constraint" game object with the tip of the leg via the Animation Rigging menu
```
Hierarchy > Select "Bone Constraint"
Hierarchy > Ctrl+Click the "FrontLegLower.L_end" bone
Top Menu > Animation Rigging > Align Transform
```
> The IK target has now inherited the local transformation of the end of the bone. Keeping objects in their relative 3D space is very important in this workflow.

* Run the game to test the IK on the leg. You should be able to move the IK target around in the scene view and see the leg follow
> If the IK is not behaving as expected, take a close look at your `Two Bone IK Constraint component`, to ensure it matches the example, additionally make sure you used the `Align Transform` option inside of `Animation Rigging`.
* Duplicate the Bone Constraint until you have 4 (or however many legs you are creating)
* Rename the Bone Constraints to `Bone Constraint 1/2/3/4`
* Test the IK on all 4 legs
* ![Testing the IK rig](image-5.png)

We have just added IK constraints to an existing armature created in Blender. Wherever we place those IK targets (Bone Constraints 1/2/3/4) in 3D space, the leg bones will attempt to go there (limited by the max length of the bones/legs)

# Take a Break
Take a break and think about how you would work on the next challenge:
* Forcing the IK targets to stay perfectly still whilst the body moves around, this might be harder than you think as the IK constraints are children of the spider character
> Why? - Ultimately we want the legs to stay grounded until they MUST move, the first step is to lock those feet while the body moves, then we can work on setting a new position for the feet to move to and trigger the movement.

### Section 2 - Locking the IK Targets in space
We'll need to write a simple script to keep on each IK target that sets its own `transform.position` in `Update()`, that way the IK targets stay in place, even when the parent moves.
* Make an new script named `BoneConstraintController`
* Have a go at writing it without looking at the example
* Put the script on every `Bone Constraint object` and test by moving the character around
* Check your method against the example to compare

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class BoneConstraintController : MonoBehaviour
{
    private Vector3 currentIKPos;
    // Start is called before the first frame update
    void Start()
    {
        currentIKPos = transform.position;
    }

    // Update is called once per frame
    void Update()
    {
        transform.position = currentIKPos;
    }
}

```

# Take a Break
Take a break and think about how you would work on the next challenge:
* Setting an Ideal point resting point that the each leg should want to rest at so that as the character moves we automatically create a target point in 3D space where we should move the leg to. Additionally this new target point should work dynamically in the scene so that we can independently aim each foot higher or lower as required when the spider walks over obstacles.
> Why? - If we don't factor in the terrain in our procedural animation code, then the legs will clip through geometry as we climb over stuff. 


### Section 3 - Adding Leg Aiming

The next challenge is to create a leg aiming and grounding system. In this first part we will make cubes to represent where each leg should move to as its ideal resting location.


* Create 4 new cubes named `Leg Aim 1/2/3/4` these will correspond to each of the `Bone Constraint 1/2/3/4` IK targets.
* Use the `Animation Rigging` tools to align the `position` (not rotation) of each `Leg Aim` to each `Bone Constraint`
```
Hierarchy > Select "Leg Aim 1"
Hierarchy > Ctrl+Click "Bone Constraint 1"
Top Menu > Animation Rigging > Align Position

...Repeat for all 4 legs
```
* Set the `Leg Aim objects` as children of the `LegIK game object` and tidy up your hierarchy as desired.

![Leg Aim Cubes in position and tidied up hierarchy](image-7.png)

> I have created two extra empty parents to hold the Leg Aim and Bone Constraint objects, additionally i have renamed my `Rig` and `Spider` trees to show that they are the original armature/mesh from Blender, and make it clearer that `LegIK` contains all of the Unity-based procedural animation. This is entirely optional for the sake of clarity.

* Run the game to test that the Leg Aim objects move with the main body of the character, but the feet are still stuck to the ground. These white cubes indicate where the character will be moving to.

![Initial test of Leg Aims](image-8.png)

# Take a Break
Take a break and think about how you would work on the next challenge:
* Updating the leg positions when the leg aim object gets far enough away

### Section 4 - Updating the Leg Positions
In this next part we will update the `BoneConstraintController` to measure the distance between the `currentIKPosition` and the `legAimPosition` - we'll call this variable `currentStepDistance`. Once `currentStepDistance` becomes greater than our `maxStepDistance`, we will set the `currentIKPosition` equal to the `legAimPosition` - snapping it to its new location

* Update the `BoneConstraintController` to include new variables
```csharp
 public GameObject legAimPosition;
 public float maxStepDistance;
 private float currentStepDistance;
 private Vector3 currentIKPosition;
```

* Create a new method named `SetIKPosition()` within `BoneConstraintController`
* Use `Vector3.Distance()` to calculate `currentStepDistance`
* Use an `if/else` to manage logic to move to the new `legAimPosition` when the `currentStepDistance` is larger than `maxStepDistance`, else stay stationary.

```csharp
private void SetIKPosition()
{
    // Find current step distance
    currentStepDistance = Vector3.Distance(currentIKPosition, legAimPosition.transform.position);
    if (currentStepDistance > maxStepDistance )
    {
        currentIKPosition = legAimPosition.transform.position;
    }
    else
    {
        transform.position = currentIKPosition;
    }
}

```

* Call `SetIKPosition()` inside of `Update()`

```csharp
void Update()
{
  SetIKPosition();
}
```
* Drag the `Leg Aim 1/2/3/4` objects into the `Leg Aim Position` fields within the `BoneConstraintsController` component on each corresponding `Bone Constraint 1/2/3/4` object
* Set the `Max Step Distance` to `2`

![Referencing the Leg Aims in the Inspector](image-9.png)

* Test the game now to see if these behaviors are working as expected. These additions should now allow you to move the spider character around, and the IK legs should jump to the new position after the distance becomes large enough.

> If you are not seeing these behaviors, go through and confirm all of your Leg Aims have been added referenced in the Inspector of the Bone Constraint objects. Make sure you have set a Max Step Distance

# Take a Break
Take a break and think about how you would work on the next challenge:
* Making sure that the legs are always detecting the ground and following the profile of the ground


### Section 5 - Adding Leg Grounding

In this next section we will check for grounding. We want to make sure that the `Leg Aim Objects` are `detecting the ground` and are always `sitting above it`. If a Leg Aim detects ground below its current position, it should update the position to move down to it. We will also use an offset in the y axis in checking for the ground so that find the correct higher level when walking up slopes.

* Create a script named `LegGrounding`
* Create variables for the new behaviours
```csharp
private int layerMask;
public Vector3 groundingOffset = new Vector3(0f,0.75f,0f);
```
* In `Start()` find the `LayerMask` for `Ground`

```csharp
void Start()
    {
        layerMask = LayerMask.GetMask("Ground"); // Get the ground mask
    }
```
* In `Update()` cast a Ray from the Leg Aim Object downwards (including a buffer of `groundingOffset`), checking for collisions on the `layerMask`, if a collision is detected, move the `Leg Aim object` to the `hit.point`
```csharp 
void Update()
    {
        // Shoot a raycast out from below the Leg Aim cube to find the ground
        RaycastHit hit;
        if (Physics.Raycast(transform.position + groundingOffset, -transform.up, out hit, Mathf.Infinity, layerMask))
        {
            // if we have found a ground layer, move the cube to the hit point (with the offset added)
            transform.position = hit.point + groundingOffset;
        }
    }


```

Next we need to address an edge case bug. If there is a groundlayer object below the ground we are currently on and we walk off to it, we will not be able to walk back up from it.

> Why? The ray is cast directly from the Leg Aim Object (plus a small vertical offset), when it finds the "new" ground below itself, it moves the Leg Aim Object down into position, so now the raycast is coming from this new grounded position, and can never detect things higher than itself.

To resolve this we want to cast our ray from a parent empty that is at the max height of the legs, not from the leg aim object itself. First try to do this without using the example below. Thing about the things we need to change:
* We need to reference a new object that the Raycast should originate from for each leg
* We need a parent/child relationship so that the Raycast origin stays with the spider, not the Leg Aim object
* We need to update the LegGrounding script to use the new Raycast origin

```csharp
// Changes to Variables
public GameObject raycastOrigin;

//Changes to Start()
raycastOrigin = transform.parent.gameObject;

//Changes to Update()
if (Physics.Raycast(raycastOrigin.transform.position + groundingOffset, -transform.up, out hit, Mathf.Infinity, layerMask))
```
> You will then need to give each leg aim an new `Empty Parent`, align position to the corresponding leg using the `Animation Rigging Menu` and set it's transform to have a suitable Y value. Using the `Animation Rigging floating window` to add some visual indicators for these is a good idea. In the example below i have created green spheres to indicate the raycast origins

![Raycast Origins](image-10.png)

Build out some basic obstacles to test the rig, we should now be able to handle low steps and slopes of uneven terrain, and be able to get back up if we drop down to a lower step. 

![Small Steps and Slopes example](image-11.png)

However, Our entire IK system still relies on the position and rotation of the main body as it's parent, so if we try to ascend to terrain that is higher than our body, we will begin to clip, through, and eventually our raycastOrigin objects will be below the slope and will snap to objects below.

![Large Slope Example](image-12.png)

As we build out further functionality to the rig we will need a way to normalize our body position and rotation from the current position of each leg. But for this lab we will focus on getting a good base level of movement on uneven terrain. lets get each leg moving to the new position in sequence where each leg waits for it's opposite to stop moving.

# Take a Break
Take a break and think about how you would work on the next challenge:
* Moving the legs only if the opposite leg is not moving.

### Section 6 - Checking if the Opposite Leg is Moving

* In `BoneConstraintController` we will tidy up the existing variables and add a new reference to the opposite leg's `BoneConstraintController` and a bool for `legIsMoving`

```csharp
// Position/Distance variables
public GameObject legAimPosition;
private float currentStepDistance;
private Vector3 currentIKPosition;

// Step Attributes
public float maxStepDistance;

// Step sequencing variables
public BoneConstraintController oppositeLeg;
private bool legIsMoving;
```

* Next we will create a new `CheckIsMoving()` method which we will use to `return the legIsMoving bool` from the oppositeLeg

```csharp
public bool CheckIsMoving()
{
  return legIsMoving;
}
```

* Add an additional condition to the `if statement` within `SetIKPosition()` method to call `CheckIsMoving()` on the `oppositeLeg` and return when `oppositeLEg.legIsMoving` is false.

```csharp
private void SetIKPosition()
{
    // Find current step distance
    currentStepDistance = Vector3.Distance(currentIKPosition, legAimPosition.transform.position);
    // Check the current distance against the max distance AND check if the opposite leg is  moving
    if (currentStepDistance > maxStepDistance && !oppositeLeg.CheckIsMoving())
    {
      // Set legIsMoving to true, transform.position to the new leg aim position, update currentIK position to the new position
        legIsMoving = true;
        transform.position = legAimPosition.transform.position;
        currentIKPosition = transform.position;
        
    }
    else
    {
      // if we dont need to or arent able to move yet, set legIsMoving to false, set transform.position to currentIKposition
        legIsMoving = false;
        transform.position = currentIKPosition;
        
    }
}
```
* Set the references to the `Opposite leg (BoneConstraintController)` for each `Bone Constraint 1/2/3/4` object in the inspector.
> In my implementation the front legs are 1/2 and rear legs 3/4, so i have the following references
> * 1 --Opposite Leg-- 2
> * 2 --Opposite Leg-- 1
> * 3 --Opposite Leg-- 4
> * 4 --Opposite Leg-- 3
> Your implementation may differ depending on how you set up your legs

![Example Implementation within Inspector](image-13.png)

* Run the program and test the rig, each leg should now move in sequence separately to it's opposite counterpart.

### Lab Summary
We have implemented the fundamentals for this type of procedural animation controller, and assuming a flat terrain or flat terrain with small uneven details, this controller, and a wide camera perspective such as an isometric view to mostly obscure the lets snapping to new positions rather than animating smoothly across an arc; this may be functional enough to implement into gameplay. For this lab however, we will be improving this controller with a lot more functionality, an overview of the next problems to solve are below:

**Lab Part 2**
* AI Navmesh to auto-follow the player
* Legs moving to new positions over time
* Legs lifting up and over to new positions over time within constraints
* Legs moving with dynamic control (e.g a Slow rise and a fast fall)

**Lab Part 3**
* Body adjusting rotation and position based on the leg positions
* Finding and compensating the body position and rotation transitioning to vertical or near vertical walls





























