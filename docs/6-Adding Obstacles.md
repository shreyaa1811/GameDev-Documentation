# 6. Add Obstacles

## 6.1 Add Obstacles

There are a couple of sprites that you can use for walls:

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/ab7af444-f199-494a-be1b-097cf57798b3_wall-sprites-1.png)

To create and add WallObjects to your game, follow these instructions:

**1.** In the **Hierarchy** window, create an empty GameObject and name it “Wall”.

Remember, you aren’t using sprites; the CellObject will modify the tilemap.

**2.** In the **Project** window, right click the **Scripts** folder and select **Create** > **Scripting** > **MonoBehaviour Script**, and name it “WallObject”.

```csharp
public class WallObject:CellObject{}
```

**WallObject**, just like **FoodObject**, inherits from CellObject.

**3.** Add the **WallObject** script as a component to the **Wall** GameObject, click and drag the **Wall** GameObject into your **Prefabs** folder, then delete the **Wall** GameObject from the scene.

**4.** Add a public variable of type **WallObject** called “WallPrefab” to your **BoardManager** and create a new **GenerateWall** function that will be very similar to your **GenerateFood** function.

The **GenerateWall** function will spawn a certain number of walls in the level.

```csharp
void GenerateWall()
{
    int wallCount = Random.Range(6, 10);

    for (int i = 0; i < wallCount; ++i)
    {
        int randomIndex = Random.Range(0, m_EmptyCellsList.Count);
        Vector2Int coord = m_EmptyCellsList[randomIndex];

        m_EmptyCellsList.RemoveAt(randomIndex);

        CellData data = m_BoardData[coord.x, coord.y];

        WallObject newWall = Instantiate(WallPrefab);
        newWall.transform.position = CellToWorld(coord);

        data.ContainedObject = newWall;
    }
}

```

**5.** Call the above **GenerateWall** method inside the **Init** method, before the call to the **GenerateFood** function.

```csharp
public void Init()
{
    ...
    GenerateWall();

    // new line
    GenerateFood();
}

```

This way all the cells that will generate walls will be removed from the empty cell list and food won’t be able to generate there.

In the Editor, assign your WallPrefab to the WallPrefab slot on your BoardManager and if you enter Play mode now, because the **WallObjects** class is empty. Even if the walls aren’t visible, by using the **Scene** view and **Hierarchy** window, you can check that the generating code works before going to the next step.`

The **Wall** GameObject will need to do two things that CellObject can’t do for now:

- Change the tile sprite it's placed on to a wall sprite.
- Stop the player character from entering a cell.

The goal is for the code to perform an action when the **CellObject** is placed over a cell. This seems like something that you might want to do with other CellObjects, not only walls. So it’ll be beneficial to add a new virtual function to **CellObject** that’s called after the object is placed. This function will also need to know the cell it’s placed on.

We can combine all these requirements into an **Init** virtual function that takes a **Vector2Int** storing the coordinate of the cell it’s placed on.

```csharp
public class CellObject : MonoBehaviour
{
    protected Vector2Int m_Cell;

    public virtual void Init(Vector2Int cell)
    {       
        m_Cell = cell;
    }

    public virtual void PlayerEntered()
    {
    }
}

```

**Note**: **m_Cell** is protected**,** not private. **Private** would make it inaccessible to the subclasses of **CellObject** like **FoodObject** or **WallObject**. Instead, **protected** will let other subclasses like **WallObject** access it, while still hiding it from other unrelated classes.

**6.** Update the **GenerateWall** function in the **BoardManager** script to the following, calling **Init** just after instantiating the new Wall:

```csharp
void GenerateWall()
{
    int wallCount = Random.Range(6, 10);

    for (int i = 0; i < wallCount; ++i)
    {
        int randomIndex = Random.Range(0, m_EmptyCellsList.Count);
        Vector2Int coord = m_EmptyCellsList[randomIndex];

        m_EmptyCellsList.RemoveAt(randomIndex);

        CellData data = m_BoardData[coord.x, coord.y];

        WallObject newWall = Instantiate(WallPrefab);

        // init the wall
        newWall.Init(coord);

        newWall.transform.position = CellToWorld(coord);
        data.ContainedObject = newWall;
    }
}

```

**7.** Add the following new method to **BoardManager**:

```csharp
public void SetCellTile(Vector2Int cellIndex, Tile tile)
{
    m_Tilemap.SetTile(new Vector3Int(cellIndex.x, cellIndex.y, 0), tile);
}

```

**8.** Add the following code to your script:

```csharp
using UnityEngine;
using UnityEngine.Tilemaps;

public class WallObject : CellObject
{
    public Tile ObstacleTile;

    public override void Init(Vector2Int cell)
    {
        base.Init(cell);
        GameManager.Instance.BoardManager.SetCellTile(cell, ObstacleTile);
    }
}

```

**9.** Save your scripts. Select the Wall prefab and, in the **Inspector** window, assign the tile you want to use to the **Obstacle Tile** slot.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/7823f650-9045-4a1f-b388-bfe40646d3ae_game-view-walls.png)

However, the player character can still move on top of the walls. This is because right now the walls just change the tilemap tile when initialized. To stop the player character from moving on top of the walls, you can change the **Passable** property of the **CellData** to **false** in the **Init** code of the **WallObject**.

A new virtual **PlayerWantsToEnter** method in **CellObject** can serve this purpose. The player character movement code calls this method on the object in a cell when the player character tries to move there, and this method either returns true if the player character can enter the cell (like for **FoodObject**) or cannot (like for **WallObject**):

```csharp
public virtual bool PlayerWantsToEnter()
{
    return true;
}

```

Then you can use that new method by modifying the code in the **if(haseMoved)** condition from the **Update** function of the **PlayerController**:

```csharp
if (cellData != null && cellData.Passable)
{
    GameManager.Instance.TurnManager.Tick();

    if (cellData.ContainedObject == null)
    {
        MoveTo(newCellTarget);
    }
    else if (cellData.ContainedObject.PlayerWantsToEnter())
    {
        MoveTo(newCellTarget);

        // Call PlayerEntered AFTER moving the player, otherwise not in the cell yet
        cellData.ContainedObject.PlayerEntered();
    }
}

```

Let’s take a closer look at the lines of code above:

- If a cell is passable the game ticks once (bumping into an object counts as a tick, but not being able to enter a cell like an outer wall or non destructible wall doesn’t).
- Then, a conditional block will run a couple of checks:
    - If the cell the player is trying to move to doesn’t have a cell object, the player character can move to that cell.
    - If the cell has a cell object, it will call **PlayerWantsToEnter** on that cell object and if that returns **true**, then the player character can enter.

```csharp
public class WallObject : CellObject
{
    public Tile ObstacleTile;

    public override void Init(Vector2Int cell)
    {
        base.Init(cell);
        GameManager.Instance.BoardManager.SetCellTile(cell, ObstacleTile);
    }

    public override bool PlayerWantsToEnter()
    {
        return false;
    }
}

```

Now if you enter Play mode, your newly placed wall should stop the player character from moving to the cell.

## 6.2 Refactoring

```csharp
void AddObject(CellObject obj, Vector2Int coord)
{
    CellData data = m_BoardData[coord.x, coord.y];

    obj.transform.position = CellToWorld(coord);
    data.ContainedObject = obj;

    obj.Init(coord);
}

```

```csharp
void GenerateWall()
{
    int wallCount = Random.Range(6, 10);

    for (int i = 0; i < wallCount; ++i)
    {
        int randomIndex = Random.Range(0, m_EmptyCellsList.Count);
        Vector2Int coord = m_EmptyCellsList[randomIndex];

        m_EmptyCellsList.RemoveAt(randomIndex);

        WallObject newWall = Instantiate(WallPrefab);
        AddObject(newWall, coord);
    }
}

```

## 6.3 **Damaging walls**

Currently when the player bumps into a wall cell, they won't be able to move, but the food counter will decrease as bumping is taken as a turn. So, let’s add the functionality of damaging walls.

- Create a new counter variable in **WallObject** that stores the hitpoint count of that wall.
- Save the original tile in that cell during **Init** before it’s replaced with the wall tile (so you can set it back after the wall is destroyed). This will require you to add a new function in **BoardManager** just like we did for **SetCellTile**!
- When the **PlayerWantToEnter** function is called in **WallObject**, reduce the hit point counter by 1, and when the hit points reach 0, the **WallObject** is destroyed, and the wall tile is removed and is set back to the original tile.

### **WallObject**

```csharp
public class WallObject : CellObject
{
    public Tile ObstacleTile;
    public int MaxHealth = 3;

    private int m_HealthPoint;
    private Tile m_OriginalTile;

    public override void Init(Vector2Int cell)
    {
        base.Init(cell);

        m_HealthPoint = MaxHealth;
        m_OriginalTile = GameManager.Instance.BoardManager.GetCellTile(cell);

        GameManager.Instance.BoardManager.SetCellTile(cell, ObstacleTile);
    }

    public override bool PlayerWantsToEnter()
    {
        m_HealthPoint -= 1;

        if (m_HealthPoint > 0)
        {
            return false;
        }

        GameManager.Instance.BoardManager.SetCellTile(m_Cell, m_OriginalTile);
        Destroy(gameObject);

        return true;
    }
}

```

### **BoardManager**

```csharp
public Tile GetCellTile(Vector2Int cellIndex)
{
    return m_Tilemap.GetTile<Tile>(new Vector3Int(cellIndex.x, cellIndex.y, 0));
}

```