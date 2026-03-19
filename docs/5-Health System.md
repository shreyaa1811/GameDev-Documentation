# 5. Adding a Health System

## 5.1 Add a Food resource

- The problem is that right now there’s no way to know when a turn happens inside the **GameManager**. Getting notified when a turn happens is going to be an important function you will need many times in the game.
- To address this problem, you can implement a callback system. A **callback system** allows any part of the code to give a method that can be called when an event happens. In this case, when a turn happens, all registered components will get notified and the given methods will be called.
- Because this callback won’t have any parameters (it just needs to know if something happens or not), you can use **System.Action** as the type of the callback (it’s a built-in type in the C# standard library for callbacks).
- Add the following to your **TurnManager** script after the first “{“:
    
    `public event System.Action OnTick;`
    
- **Note**: **event** is a special C# keyword for callback. This means only the class in which **OnTick** is declared (in this case TurnManager) can trigger the event, nothing else.
- Add the following inside the **Tick** method:
    
    `OnTick?.Invoke();`
    
- **Invoke** is the method in **System.Action** that will call (“invoke”) all the callback methods that were registered to the **OnTick** event.
- The **?** is a special C# syntax used to test if **OnTick** is null. If it’s null, ? does nothing, but if it’s not null, ? will call the method on the right of the ?. It’s a shorthand version that does the same thing as:

```csharp
if (OnTick != null)
{
    OnTick.Invoke();
}

```

- You need to test if **OnTick** is null because if no other part of the code registered a callback function to **OnTick**, then it will be null, and trying to call **Invoke** on a null object will generate an error and break the game!
- Now other scripts can register to that callback to get notified when a tick happens. In your **GameManager**, do the following:
    - Add a private integer member that stores how much food you currently have (initialized at 100).
    - Add a method called “OnTurnHappen” that will be called when a turn happens.
    - Register the **OnTurnHappen** method to your **TurnManager.OnTick** callback.

```csharp
private int m_FoodAmount = 100;

private void Start()
{
    TurnManager = new TurnManager();
    TurnManager.OnTick += OnTurnHappen;

    BoardManager.Init();
    PlayerController.Spawn(BoardManager, new Vector2Int(1, 1));
}

private void OnTurnHappen()
{
    m_FoodAmount -= 1;
    Debug.Log("Current amount of food: " + m_FoodAmount);
}

```

**TurnManager.OnTick += OnTurnHappen** is how you register a method callback to the **OnTick** event. When **OnTick** has its **Invoke** method called, it calls the **OnTurnHappen** method. **+=** is used because we add the method to all the other ones already registered (right now there is a single one, but later you’ll have more methods called when **OnTick** happens).

## 5.2 UI Designing

To add UI elements to your game, follow these instructions:

**1.** Right-click in the **Hierarchy** window and select **UI Toolkit** > **UI Document**.

This will create a new GameObject with a **UI Document** component attached to it. It will also create all the default settings for the UI Toolkit in your project.

The UI toolkit works by decoupling the structure, look, and behavior of the UI in the following ways:

- The structure is given by a UXML file (that is similar to an HTML file) that defines which elements your UI is made of and their hierarchical relationship.
- The look is defined by a USS file (similar to a CSS file) that defines visual properties (colors, borders, spacing, alignment, etc.) of your UI element.
- The behavior is coded in C# by retrieving elements by their name or id and applying listeners or other behavior on them.

Right now, the UI document component expects a Visual Tree Asset as its source. A **Visual Tree Asset** is a UXML file that defines your UI’s structure.

**2.** In the **Project** window, right-click the **Assets** folder and select **Create** > **Folder** and name it “UI”.

**3.** Right-click the **UI** folder and select **Create** > **UI Toolkit** > **UI Document** and name it “GameUI”.

**4.** In the **Hierarchy** window, select the **UIDocument** GameObject, then in the **Inspector** window select the **Source Asset** picker (⊙) and select the **GameUI** document.

**5.** Double click the **GameUI** document.

- This will open the **UI Builder** window. Double click the window name to expand the window.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/048578fd-d7e6-4dc8-b4c9-fdb795ea5847_ui-builder-window.png)

- The **UI Builder** window is a visual tool that lets you edit UXML files (like your **GameUI** document) visually instead of through text.
- The upper-left panel contains the **Hierarchy** window of your UI, which at the moment only contains the GameUI.uxml file. Below the **Hierarchy** window there is the **Library** window that contains all kinds of UI controls you can add to your UI.
- The central **Viewpoint** window consists of the view of your UI, which is currently empty.
- The rightmost window is the **Inspector** window, which displays all the properties of a selected UI element.

**6.** In the **Library** window, under the **Controls** section, drag and drop the **Label** element into the **Viewport** window.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/cb13e86d-57a0-42c4-9bfb-c97caef020aa_drag-label-to-panel.png)

**7.** With **Label** selected in the **Hierarchy** window, select the **Label** box in the **Inspector** window and rename it “FoodLabel”.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/bfec1791-875f-415e-8369-6637cb16ae74_inspetor-label-window.png)

**8.** With the **Label** selected in the **Hierarchy** window, in the **Inspector** window, use the foldout (triangle) to expand the **Text** section.

**9.** Select the **Font** property picker (⊙) and assign either the provided PressStart2P-Regular font or your own.

**10.** Select the **Color** property box and set the color to white (**FFFFFF**) so the **Label** will stand out on the black background.

![](https://unity-connect-prd.storage.googleapis.com/20250825/learn/images/a5a2290e-884a-48a7-a2a3-ea4e86d9482c_Untitled_54.png)

The **Label** font should have changed in the **Viewport** window.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/1d1119ae-b67f-4286-b34d-6d52b16e5efb_label-top-left.png)

**11.** Change the **Size** property and make it bigger or smaller according to your own preferences (experiment with different sizes until you find one you like), and finally center the text using the **Align** property.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/f37b70da-9e9b-44e2-886d-1953964a4291_size-allign-properties.png)

Finally, let’s put the label at the bottom of the screen. Right now the UI toolkit uses automatic layout. The default layout organizes consecutive elements in the **Hierarchy** window from top to bottom, stretching across the width of the UI document.

Instead, use manual positioning to place the **Label** exactly where you want.

**12.** Use the foldout (triangle) to expand the **Position** section in the **Inspector** window.

**13.** Open the **Position Mode** property dropdown and change the **Position Mode** from **Relative** to **Absolute**.

In **Absolute** mode, the four offsets (Top/Right/Bottom/Left) are the distance to each border of the screen.

**14.** Set the **Bottom** offset to **0**.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/bb76cdd3-82ee-44fe-9a03-cf83fd333520_label-botton-left.png)

**15.** Set both the **Left** and **Right** offsets to **0**.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/c8a4a583-899c-4fcd-a6a5-9546c6a3030d_label-justified.png)

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/833af6ad-198f-4a85-a821-831fe26bc972_save-button.png)

**16.** In the Editor, enter Play mode to check how your changes look in game.

![](https://unity-connect-prd.storage.googleapis.com/20250825/learn/images/5ffe6a20-0a9f-4bc1-9acc-754c2107cb34_Untitled_55.png)

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/7f75086d-79a1-41d4-ad20-fc8532ebf5a6_maximize-wndow-button.png)

## 5.3 Updating the Label Using Code

To do this, you’ll use the **GameManager**:

**1.** Add a **public** variable of type **UIDocument** to your **GameManager** script. You also need to add **using** **UnityEngine.UIElements** at the top of the file to be able to use the classes related to UI Toolkit like UIDocument.

**2.** Create a **private** variable of type **Label** to store a reference to the Label so you can access and modify it.

You can now assign your UI Document (the component on the **UIDocument** GameObject, not the Game UI file) to your GameManager in the **Inspector** window.

You can then add the code that finds the Label inside your GameUI and update its text to the Food count in the **GameManager** **Start** method:

```csharp
using UnityEngine;
using UnityEngine.UIElements;

public class GameManager : MonoBehaviour
{
    public UIDocument UIDoc;

    private Label m_FoodLabel;
    private int m_FoodAmount = 100;

    private void Start()
    {
        m_FoodLabel = UIDoc.rootVisualElement.Q<Label>("FoodLabel");
        m_FoodLabel.text = "Food : " + m_FoodAmount;
    }
}

```

- **UIDocument** has a .**rootVisualElement** property that is the root (the first element) of the **Hierarchy** window you saw in the **UIBuilder** window earlier. All elements in the **Hierarchy** window (like the label) are child elements of that root element..
- This root (and any element of the UIToolkit) has a special method, **Q**, short for **Query**. This **Q** method looks for an element of the given type (for example, **Label**) with the given name (for example, **FoodLabel**) in the child elements of the element on which you call Q.
- **m_FoodLabel** is an additional private variable of type Label to add to your **GameManager** so you can store the **Label** and not have to look for it everytime with the **Q** method.
- The **.text** property is used to set the text of the label to display the current amount of food (for example, Food:100).

You should now have your food count at the bottom of your screen.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/16014662-233b-4075-a796-b3aa51afa0b6_game-view-food-counter.png)

However, you’ll notice that the counter doesn’t update when the player character moves. For this, you have to replace the **Debug.Log** you wrote earlier in the **OnTurnHappen** method with the following:

```csharp
private void OnTurnHappen()
{
    m_FoodAmount -= 1;
    m_FoodLabel.text = "Food : " + m_FoodAmount;
}


```

Now, if you enter Play mode and move around, the food counter will decrease

## 5.4 Objects in cell and food refill

Collider and Rigidbody components are used to detect collisions and overlapping of objects. But in this case, because the game is turn based and happens on a grid, you don’t need to use physics.

You already have **CellData** for each cell in the game, and you use it to store whether the cell is passable or not. Now, you’ll add a new value to that **CellData** that stores whether the cell contains an object or not:

**1.** Open your **BoardManager** script and inside the **CellData** class add the following new variable:

```csharp
public class CellData
{
    public bool Passable;
    public GameObject ContainedObject;
}

```

**2.** Drag your preferred food sprite into the scene to create a new GameObject with that sprite, name it “SmallFood”, set its **order in layer** to something between **1** and **9**.

**3.** In the **Project** window, right-click the **Assets** folder and select **Create** > **Folder,** new it “Prefabs”**,** then drag and drop your **SmallFood**GameObject from the **Hierarchy** window into the new **Prefabs** folder.

This will create a prefab from that GameObject that you can reuse multiple times and between scenes.

**4.** Delete the **SmallFood**GameObject from the scene.

**5.** In **BoardManager**, add a public variable **FoodPrefab** of type **GameObject**, to store the prefab of the food you just created.

```csharp
public GameObject FoodPrefab;
```

**6.** Write a new void method named “GenerateFood”, then copy and paste the following code into that method:

```csharp
private void GenerateFood()
{
    int foodCount = 5;

    for (int i = 0; i < foodCount; ++i)
    {
        int randomX = Random.Range(1, Width - 1);
        int randomY = Random.Range(1, Height - 1);

        CellData data = m_BoardData[randomX, randomY];

        if (data.Passable && data.ContainedObject == null)
        {
            GameObject newFood = Instantiate(FoodPrefab);
            newFood.transform.position = CellToWorld(new Vector2Int(randomX, randomY));
            data.ContainedObject = newFood;
        }
    }
}

```

- Creates an **int** variable that stores how many food prefabs will be created (making a variable allows you to easily change it to test different values)
- Loops this number of times:
    - On each loop, a random x and y position are picked. **Random.Range** returns a number contained between the first parameter (included) and the second (excluded). Note that the range is 1 to size - 2 (as size -1 is excluded) because the code shouldn’t pick a cell that is an outside wall!
    - It then retrieves the **CellData** for that cell position.
    - If this cell is passable and there is no object there, it instantiates a new food prefab in that position and stores that food as the **ContainedObject** in that cell in its **CellData**.

**7.** Call this new method at the end of the **Init** method of the **BoardManager**, once the level has been created.

```csharp
public void Init()
{
    m_Tilemap = GetComponentInChildren<Tilemap>();
    m_Grid = GetComponentInChildren<Grid>();

    m_BoardData = new CellData[Width, Height];

    for (int y = 0; y < Height; ++y)
    {
        for (int x = 0; x < Width; ++x)
        {
            Tile tile;
            m_BoardData[x, y] = new CellData();

            if (x == 0 || y == 0 || x == Width - 1 || y == Height - 1)
            {
                tile = WallTiles[Random.Range(0, WallTiles.Length)];
                m_BoardData[x, y].Passable = false;
            }
            else
            {
                tile = GroundTiles[Random.Range(0, GroundTiles.Length)];
                m_BoardData[x, y].Passable = true;
            }

            m_Tilemap.SetTile(new Vector3Int(x, y, 0), tile);
        }
    }

    GenerateFood();
}

```

**8.** Save your changes to the scripts, assign your SmallFood prefab to the **FoodPrefab** slot on the **BoardManager** in the **Inspector** window, and then enter Play mode to check if food appears on your map.

**9.** Add a new variable called “m_EmptyCellsList” of type **List** containing **Vector2Int** in the **BoardManager** script. You’ll need to add **using** **System.Collections.Generic** at the top of your file to be able to use **List:**

```csharp
public void Init()
{
    m_Tilemap = GetComponentInChildren<Tilemap>();
    m_Grid = GetComponentInChildren<Grid>();

    // Initialize the list
    m_EmptyCellsList = new List<Vector2Int>();

    m_BoardData = new CellData[Width, Height];

    for (int y = 0; y < Height; ++y)
    {
        for (int x = 0; x < Width; ++x)
        {
            Tile tile;
            m_BoardData[x, y] = new CellData();

            if (x == 0 || y == 0 || x == Width - 1 || y == Height - 1)
            {
                tile = WallTiles[Random.Range(0, WallTiles.Length)];
                m_BoardData[x, y].Passable = false;
            }
            else
```

```csharp
...m_EmptyCellsList.Remove(newVector2Int(1,1));GenerateFood();
```

You can replace picking a random coordinate in the **GenerateFood** method with an entry from the **m_EmtpyCellsList** list, and then remove that entry from the **m_EmtpyCellsList** list (because the cell is no longer empty).

This is the final version of GenerateFood using the empty cell list :

```csharp
private void GenerateFood()
{
    int foodCount = 5;

    for (int i = 0; i < foodCount; ++i)
    {
        int randomIndex = Random.Range(0, m_EmptyCellsList.Count);
        Vector2Int coord = m_EmptyCellsList[randomIndex];

        m_EmptyCellsList.RemoveAt(randomIndex);

        CellData data = m_BoardData[coord.x, coord.y];

        GameObject newFood = Instantiate(FoodPrefab);
        newFood.transform.position = CellToWorld(coord);

        data.ContainedObject = newFood;
    }
}

```

## 5.5 **Collect food**

A very basic way to do this would be to follow this structure:

- When entering a new cell, check if there is a **ContainedObject** in that cell’s **CellData**.
- If yes, increase the food counter of the player character by a certain amount.

This works, and you can even test doing it this way if you feel up for the challenge. However, this approach is very limited. For example, what if you want different food objects to give different amounts of food points, or you want to make poisoned food that makes you lose food points?

To do this, you need a way to make handling objects in a cell more generic, like a generic **CellObject** type that your CellData can contain and then act on that object when the player character interacts with a cell. That way you can easily write a new CellObject and the player code doesn’t have to be modified to handle every different type of object, even some you might not have thought about when designing this system.

This will be solved through **C# inheritance**: a base class CellObject that different objects (**FoodObject**, for example) can inherit from.

This base type will have a virtual method called **PlayerEntered** that will be called by the player code when the player character enters its cell. That way every subclass can handle what happens when the player character enters a cell (for example, food objects will destroy themselves and increase the food count by a certain amount).

To add this functionality, follow these instructions:

**1.** In the **Project** window, under the **Assets** folder, right -lick the **Scripts** folder and select **Create** > **Scripting** > **MonoBehaviour Script**. Name it “CellObject” and add a virtual method called “PlayerEntered”.

The **PlayerEntered** method will be empty in that base class because by default, cell objects do nothing when the player character enters their cell.

```csharp
using UnityEngine;

public class CellObject : MonoBehaviour
{
    // Called when the player enters the cell containing this object
    public virtual void PlayerEntered()
    {
    }
}

```

**2.** In the **Project** window, under the **Assets** folder, right-click the **Scripts** folder and select **Create** > **Scripting** > **MonoBehaviour Script**, and name it “FoodObject”.

**3.** Change the class the **FoodObject** script inherits from **MonoBehaviour** to **CellObject**. You then need to redefine the **PlayerEntered** method so it does a different thing than the base class one. This is called **overriding** the method, and uses the override keyword.

```csharp
using UnityEngine;

public class FoodObject : CellObject
{
    public override void PlayerEntered()
    { 
        Destroy(gameObject);

        // increase food
        Debug.Log("Food increased");
    }
}

```

For now, this will just destroy the **FoodObject** GameObject and write something in the console to show it happened. This is enough to test if the code works.

**4.** Replace the GameObject in **CellData** with a **CellObject** in the **BoardManager** script**.**

```csharp
public class CellData
{
    public bool Passable;
    public CellObject ContainedObject;
}

```

This will generate a couple of errors in your console you should be able to fix.The fixes are below, but see if you can figure them out for yourself before you look at the solutions!

Here are the bug fixes:

- Change the **FoodPrefab** type from **GameObject** to **FoodObject** in the **BoardManager script**.
- In the **GenerateFood** method, because you now instantiate a FoodObject, change the **newFood** type from **GameObject** to **FoodObject.**

**5.** In your **PlayerController** script, when the player enters a new cell, if it contains an object, call **PlayerEntered** on it:

```csharp
if (hasMoved)
{
    // check if the new position is passable, then move there if it is.
    BoardManager.CellData cellData = m_Board.GetCellData(newCellTarget);

    if (cellData != null && cellData.Passable)
    {
        GameManager.Instance.TurnManager.Tick();
        MoveTo(newCellTarget);

        if (cellData.ContainedObject != null)
        {
            cellData.ContainedObject.PlayerEntered();
        }
    }
}

```

Virtual methods allow you to change the behavior of a method in a subclass. In this case, **CellData** keeps a reference to a **CellObject**, but this reference can point to an object of any subclass of **CellObject** (in this case, **FoodObject**). When you call **PlayerEntered**, the correct overridden method will be automatically called, based on the **actual** type of the stored object. In this case, because **ContainedObject** will actually point to a **FoodObject**, it is its **PlayerEntered** method that will be called, not the empty **CellObject method**.

# 5.6 **Increasing the food count**

So far, the console is only outputting when the player character collects a **Food** GameObject. However, you also need to actually modify the amount of food displayed on the label. This is stored in the **GameManager**, and you have access to it through the **Singleton** instance.

- Create a method in the **GameManager** script that changes the amount of food by a given parameter and name it something like “ChangeFood”. Don’t forget to also update the label when you do so!
- Add a public property on your **FoodObject** so you can define how many food points are given by the **FoodObject** when collected.
- Call the new **GameManager** method in the **PlayerEntered** method on your **FoodObject**.

### **In GameManager**

```csharp
void OnTurnHappen() 
{
    ChangeFood(-1);
}

public void ChangeFood(int amount)
{
    m_FoodAmount += amount;
    m_FoodLabel.text = "Food : " + m_FoodAmount;
}

```

### **FoodObject**

```csharp
public class FoodObject : CellObject
{
    public int AmountGranted = 10;

    public override void PlayerEntered()
    {
        Destroy(gameObject);

        // increase food
        GameManager.Instance.ChangeFood(AmountGranted);
    }
}

```