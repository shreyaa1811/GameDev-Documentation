## 8. Animation

To add some life into the game, let’s add some animations to it! Animations can not only increase player experience by adding visual interest to your game, but they also help bring a higher level of polish and professionalism to applications.

## 8.1 Idle animation

**1.** In the **Hierarchy** window, select the **PlayerCharacter** GameObject and, in the **Inspector** window, add an **Animator** component to it.

**2.** In the **Project** window, right click the **Assets** folder, select **Create** > **Folder**, and name the new folder “Animation”.

**3.** Right click inside the **Animation** folder and select **Create** > **Animation** > **AnimatorController** to add an AnimatorController, then name it “PlayerAnimator”.

**4.** In the **Hierarchy** window, select the **PlayerCharater** GameObject. In the **Inspector** window, under the Animator component, select the **Controller** property picker (⊙) and select **PlayerAnimator**.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/01184024-7119-4097-901b-be21248a2e85_animator-controller.png)

**5.** From the main menu, select **Window** > **Animation** > **Animation** to open the **Animation** window.

Before you can continue, the **Animation** window will ask to create an Animation Clip.

**6.** Select the **Create** button and save the new clip as “Idle.anim” inside your **Animation** folder.

This tutorial won't go into detail about how everything works and will only cover how to create animations and handle them, but feel free to check the [2D Adventure Game](https://learn.unity.com/course/2d-beginner-adventure-game) again for more in depth information.

**7.** From the **Sprites** folder in the **Project** window (**Roguelike2D** > **TutorialAssets** > **Sprites**), drag and drop all the sprites you want to use for the idle animation one after the other into the timeline of the **Animation** window to add them as new keyframes.

Place the sprites at the same interval from each other to keep a constant speed.

![](https://connect-mediagw.unity.com/h1/20241010/learn/images/df64ef85-fb70-42cc-9f00-0ce8f287d87f_image23.png)

If you select the **Play** button in the **Animation** window now, you will see a preview in the **Scene** view of the animation on your **PlayerCharacter** GameObject. If you placed the sprites like in the example, the animation will probably look really fast.

The number at the top of the timeline is composed of the following parts:

- The number on the left of the colon (:) is the current time in seconds of the animation.
- The number on the right of the colon (:) is the frame number of that specific second.

So 0:15 means the 15th frame of second 0 of the animation, and 3:40 means the 40th frame of second 3 of the animation.

Right now the animation has 60 frames in 1 second, so you can see that the number goes up to 0:59 before changing to 1:00

To slow the animation down, you can move each sprite individually to a frame later in the animation, but it’s wasteful to have 60 possible frames in a second when this animation is only a couple of frames long. Instead, let’s change the number of frames per second.

For example, if your animation is six frames long, setting the sample rate to be three frames per second means the animation will last two seconds.

**8.** Open the **More** (**⋮**) menu next to the timeline and select **Show Sample Rate**, then change the **Samples** property on the left to roughly half of you number of frames (in our examples three).

As usual, feel free to experiment with different values!e timeline and select **Show Sample Rate**, then change the **Samples** property on the left to **3**.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/be8f6176-125d-4f27-a40c-949474fc374a_show-sample-rate.png)

You’ll then need to re-arrange the sprites because the way the system automatically moves them when you change the sample rate will probably result in a very slow animation. Zoom out of the timeline and move each frame until they are on consecutive frames.

## 8.2 Walking animation

Now you can add a new animation for when the player is walking between cells:

**1.** In the **Animation** window, with your **PlayerCharacter** GameObject still selected, open the **Idle** dropdown and select **Create New Clip…**.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/6db24733-0f8b-440f-b974-2038be8471c6_create-new-animation-clip.png)

**2.** Name that new clip “Walk.anim” and save it in the **Animation** folder.

**3.** From the **Sprites** folder in the **Project** window, drag and drop the walking animation sprites into the timeline of the **Animator** window and edit the frame position or frame rate to make it look pleasing to you.

Your walking animation timeline should look something like this:

![](https://connect-mediagw.unity.com/h1/20241010/learn/images/90f0cc86-c82e-4bb7-8418-d7eb28c4d741_adjusting-timestamps.png)

Use the same technique as above to change the sample rate to something that makes the walk of your character look good to you.

Once the animation is created, you now need a way to trigger it only when the player character is walking. To do this, you’ll use a new tool called an Animation State Machine.

**4.** From the main menu, select **Window** > **Animation** > **Animator** to open the **Animator** window.

It should contain a state for the two animations you created. If they don’t appear there because you created the animation differently, simply drag and drop your animation clip from the **Project** window to the **Animator** window to add them.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/f84b7471-534d-47d9-b1e1-739c32d1fee2_animation-desicion-tree.png)

**5.** Select the **Parameters** tab in the **Animator** window, select the **Add** (**+**) button, select **Bool**, and rename it “Moving”.

![](https://connect-mediagw.unity.com/h1/20240930/learn/images/5c98a2ec-7068-45df-87e6-7c4e6ca4347d_animator-parameters.png)

You’ll use this new parameter as a trigger to create a transition from the **Idle** to the **Walk** state when the **Moving** boolean is **true**.

**6.** Right click the **Idle** state and select **Make Transition**.

An arrow starting from the **Idle** state will appear.

**7.** Select the **Walk** state to create a transition between the **Idle** and **Walk** states.

**8.** Select the transition arrow to display its settings in the **Inspector** window, then disable the **Has Exit Time** checkbox.

If **Has Exit Time** is enabled, then the transition from **Idle** to **Walk** would happen as soon as this time has elapsed. This is not what you want; you want **Idle** to loop until the **Moving** boolean is true, so disabling **Has Exit Time** makes the animation loop.

**9.** Use the foldout (triangle) to expand the **Settings** section and set the **Transition Duration** property to **0**.

This time allows the gradual blending from one animation to another, but blending animations doesn’t make sense in 2D applications because you want to immediately switch from one state to the other.

**10.** At the bottom of the **Inspector** window, under the **Conditions** section, select the **Add** (**+**) button to add a new condition. Open the leftmost dropdown, select **Moving**, then open the rightmost dropdown and select **true**.

This tells the controller that when the **Moving** parameter is set to **true**, it needs to change the animation from **Idle** to **Walk**.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/0d302fff-ca27-4ae6-9689-a6532f7d825d_animation-transitions.png)

Repeat the above steps, but this time start with from the **Walk** to the **Idle** animation. This time the transition will be triggered when **Moving** is set back to **false**.

- Create a private variable of type **Animator** to store the Animator Controller component.
- Add an **Awake** function that will retrieve the **Animator** component using **GetComponent**.
- Everywhere inside code where you alter the **IsMoving** variable, you need to also call the function **SetBool** on the Animator component’s **Moving** bool.

```csharp
public class PlayerController:MonoBehaviour{...
private Animator m_Animator;
private void Awake()
{       
m_Animator=GetComponent<Animator>();
}...
public void MoveTo(Vector2Int cell,bool immediate)
{       
m_CellPosition= cell;
if(immediate)
{           
m_IsMoving=false;           
transform.position= m_Board.CellToWorld(m_CellPosition);
}
else
{           
m_IsMoving=true;           
m_MoveTarget= m_Board.CellToWorld(m_CellPosition);
}
m_Animator.SetBool("Moving", m_IsMoving);
}
private void Update()
{
...
if(m_IsMoving)
{           
transform.position= Vector3.MoveTowards(transform.position, m_MoveTarget, MoveSpeed* Time.deltaTime);
if(transform.position== m_MoveTarget)
{               
m_IsMoving=false;               
m_Animator.SetBool("Moving",false);
var cellData= m_Board.GetCellData(m_CellPosition);
if(cellData.ContainedObject!=null)                   
cellData.ContainedObject.PlayerEntered();
}
return;
}
```

**Note:** String base lookup on the Animator component is inefficient, and the efficient way is to store a reference to the Hash of **Moving** with the **Animator.StringToHash** function.