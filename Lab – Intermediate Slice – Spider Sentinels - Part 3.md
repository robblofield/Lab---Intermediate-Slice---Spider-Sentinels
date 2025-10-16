# Lab – Intermediate Slice – Spider Sentinels (Part 3: Alternating Opposite Legs)

*In this lab, you’ll coordinate leg movement so diagonally opposite pairs do not step at the same time. We’ll build directly on Part 1–2 and refactor the `BoneConstraintController` to respect “opposite leg is moving” checks.*

![Spider Sentinels Render](<Images/spider sentinels render.png>)
---

### Quick Recap (from Parts 1–2)
- **Part 1:** IK targets locked; Leg Aims created; step trigger snaps feet to aims based on distance.  
- **Part 2:** Leg Aims made **terrain-aware** using raycasts; fixed the raycast origin bug.

---

### Prerequisites
- Working spider rig from Part 1–2.  
- Four **Bone Constraint** targets (**`Bone Constraint 1/2/3/4`**) each with a **`BoneConstraintController`**.  
- Four **Leg Aim** objects (**`Leg Aim 1/2/3/4`**) grounded by your **`LegGrounding`** script.  

> **Note:** In this session, we only add **alternating logic**. Smooth step **arcs** and timing curves are scheduled for **Part 4**.

---

### Section 1 – Add Opposite-Leg Coordination

We’ll add:
- a reference to the **opposite leg**’s controller,
- a **`legIsMoving`** state flag,
- and a **`CheckIsMoving()`** helper.

**Goal:** A leg may only step if:
1) its distance to its Leg Aim exceeds `maxStepDistance`, **and**  
2) its **opposite leg is not moving**.

1. Open **`BoneConstraintController`**.  
2. Add fields for position/distance, step attributes, opposite leg reference, and a moving flag.  
3. Implement **`CheckIsMoving()`** to expose whether a leg is currently moving.  
4. Update **`SetIKPosition()`** to include an **opposite leg** check.

<details>
<summary>💡 Example: Refactored BoneConstraintController (click to expand)</summary>

```csharp
using UnityEngine;

public class BoneConstraintController : MonoBehaviour
{
    // Position/Distance variables
    public GameObject legAimPosition;
    private float currentStepDistance;
    private Vector3 currentIKPosition;

    // Step Attributes
    public float maxStepDistance = 2f;

    // Step sequencing variables
    public BoneConstraintController oppositeLeg;
    private bool legIsMoving;

    void Start()
    {
        currentIKPosition = transform.position;
    }

    void Update()
    {
        SetIKPosition();
    }

    public bool CheckIsMoving()
    {
        return legIsMoving;
    }

    private void SetIKPosition()
    {
        // Measure distance to the aim
        currentStepDistance = Vector3.Distance(currentIKPosition, legAimPosition.transform.position);

        // Can we move? Only if we're beyond the step threshold AND the opposite leg is not moving
        bool canStep = currentStepDistance > maxStepDistance && (oppositeLeg == null || !oppositeLeg.CheckIsMoving());

        if (canStep)
        {
            // Start step (instant snap for now; smooth arcs come in Part 4)
            legIsMoving = true;

            transform.position = legAimPosition.transform.position;
            currentIKPosition = transform.position;

            // End step (since we're snapping this frame)
            legIsMoving = false;
        }
        else
        {
            // Hold position
            legIsMoving = false;
            transform.position = currentIKPosition;
        }
    }
}
```
</details>

> **Tip:** If you used different variable names previously (e.g., `currentIKPos`), keep them consistent or refactor cleanly across all legs.

---

### Section 2 – Wire Up Opposite Pairs in the Inspector

Set **opposite leg** references so legs alternate in **pairs**.

1. Decide your opposing pairs (example):  
   - Front Left (**1**) ↔ Front Right (**2**)  
   - Rear Left (**3**) ↔ Rear Right (**4**)  
2. In the Inspector for each **`BoneConstraintController`**:  
   - Assign its **`oppositeLeg`** to the corresponding opposite controller.  
   - Confirm each controller has its own **`legAimPosition`** set.  
   - Ensure **`maxStepDistance`** is set (e.g., `2`).

> **Note:** Your exact numbering may differ based on how you set up the rig; just ensure each leg’s opposite is correct.

---

### Section 3 – Test Alternating Movement

Create a simple test plan to verify alternating behavior:

1. **Flat Ground Test**  
   - Move the spider; confirm only **one leg of a pair** steps at a time.  
2. **Uneven Ground Test**  
   - On steps/slopes, ensure **opposite legs never step together**, even when both are past the threshold.  
3. **Edge Case: Rapid Movement**  
   - Move the body quickly; confirm the opposite check still suppresses simultaneous steps.  
4. **Tuning**  
   - Adjust **`maxStepDistance`** to control how frequently legs step.

> If both legs appear to stick, verify that each leg’s **`oppositeLeg`** is assigned and that **`maxStepDistance`** isn’t too large for your scale.

---

# Take a Break

Think about the next challenge:  
- **How should a foot move from A → B?**  
  Next time, we’ll replace the instant snap with a **smooth arc**, and introduce timing so steps look deliberate (lift → swing → drop).

---

### Lab Summary

In this session, you coordinated your spider’s legs so **diagonally opposite pairs never move simultaneously**.

- Added **opposite-leg references** between paired legs.  
- Introduced a **`legIsMoving`** state and **`CheckIsMoving()`** helper.  
- Updated step logic to **respect opposite-leg movement**, reducing jitter and improving stability.

This gives your rig a more believable gait foundation, ready for **timed, curved steps** in the next session.

---

### Next Steps in Future Lab Sessions

**Lab Part 4**  
- Replace snap moves with **smooth step arcs** (interpolation + curves).  
- Add timing profiles (e.g., slow lift, fast drop) for more natural motion.

**Lab Part 5**  
- Adjust body pose (height/tilt) based on current leg placements to prevent clipping on steeper terrain.

**Lab Part 6+**  
- Integrate navigation or AI control and polish the full movement loop.
