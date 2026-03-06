# 4. Adding a Turn System

## 4.1 Turn Manager

- If you go back to your list of tasks, the next step after having a moving player is implementing a turn system when the player character makes a move.
- To create a turn system, follow these instructions:
    
    **1.** Right-click the **Scripts** folder in the **Project** window and select **Create** > **Scripting** > **Empty C# Script**. Name this script “**TurnManager**”
    
    In this instance, you don’t need to use a MonoBehaviour script because, unlike the other scripts you’ve written so far, this one is a pure data class that just handles data and is not linked to a GameObject, so it doesn’t need to inherit from MonoBehaviour.
    

### **TurnManager:**

```csharp
using UnityEngine;
public class TurnManager
{

}
```

Because this is not a MonoBehaviour script and doesn’t exist in the scene, there is no **Start** or **Update** method, so you'll need to create/initialize it manually, like you would when you initialize a Vector3 or your CellData.

There are several ways you might do this:

- Inside your **BoardManager**’s **Start** method.
- Inside your **PlayerController**’s **Spawn** method.
- Inside a new system that takes care of initializing everything needed for a level at start.

That last solution seems to be the cleanest, as there will probably be other things you need to initialize when the game gets more complex.

Think about the startup of your game. Right now you rely only on the **Start** method of the **BoardManager** to initialize everything, but later on when you want to handle multiple levels, you might need to call the initialization code again.

Instead of this, let’s create a **GameManager** GameObject that will be the entry point for everything the game needs when starting. It will consist of a script whose **Start** method will do the following:

- Initialize the **TurnManager.**
- Trigger the **BoardGeneration**.
- Spawn the **Player**.

So you have some new code to write and some refactoring to do first!

**1.** In the **Main** scene, create a new empty GameObject and name it “GameManager”.

**2.** Right-click the **Scripts** folder in the **Project** window and select **Create** > **Scripting** > **MonoBehaviour Script**. Name this script “GameManager”.

**3.** Add the **GameManager** script as a component to the **GameManager** GameObject.

**4.** Inside the script create two public references:

- One for the **Board** of type **BoardManager**.
- One for the **Player** of type **PlayerController**.

**5.** Create a private variable of type **TurnManager**.

**6.** Inside **GameManager** script’s **Start** method do the following:

- Initialize the TurnManager (it’s just a normal class, so you can use “new”)
- Initialize the level (you’ll have to rename the **Start** method in the **BoardManager** to **Init** and make it **public** to be able to call it from this **Start** method)
- Spawn the player at (1,1), so you'll have to remove the call to spawn the player from the **BoardManager Init** method.

You should be able to do it all by yourself, and here is what the final scripts look like:

### **BoardManager**

```csharp
public class BoardManager:MonoBehaviour
{
public class CellData
{
public bool Passable;
}
private CellData[,] m_BoardData;
private Tilemap m_Tilemap;
private Grid m_Grid;
public int Width;
public int Height;
public Tile[] GroundTiles;
public Tile[] WallTiles;
public void Init()
{       
m_Tilemap=GetComponentInChildren<Tilemap>();       
m_Grid=GetComponentInChildren<Grid>();
       m_BoardData=newCellData[Width, Height];
for(int y=0; y< Height;++y)
{
for(int x=0; x< Width;++x)
{
Tile tile;               
m_BoardData[x, y]=newCellData();
if(x==0|| y==0|| x== Width-1|| y== Height-1)
{
tile= WallTiles[Random.Range(0, WallTiles.Length)];                   
m_BoardData[x, y].Passable=false;
}
else                  
{
tile= GroundTiles[Random.Range(0, GroundTiles.Length)];                   
m_BoardData[x, y].Passable=true;
}
m_Tilemap.SetTile(newVector3Int(x, y,0), tile);
}
}
}
public Vector3CellToWorld(Vector2Int cellIndex)
{
return m_Grid.GetCellCenterWorld((Vector3Int)cellIndex);
}
public CellData GetCellData(Vector2Int cellIndex)
{
if(cellIndex.x<0|| cellIndex.x>= Width|| cellIndex.y<0|| cellIndex.y>= Height)
{
return null;
}
return m_BoardData[cellIndex.x, cellIndex.y];
}
}
```

### GameManager

```csharp
public class GameManager:MonoBehaviour
{
public BoardManager BoardManager;
public PlayerController PlayerController;
private TurnManager m_TurnManager;
// Start is called once before the first execution of Update after the MonoBehaviour is created
 void Start()
 {       
 m_TurnManager=newTurnManager();
 BoardManager.Init();       
 PlayerController.Spawn(BoardManager,newVector2Int(1,1));
 }
 }
```

## 4.2 Ticking Turn

- To do this, you’ll need a private member variable that saves the current turn number, initialized at 1 in the constructor, and a **Tick** method that increases the count by 1 and writes the current turn count using the **Debug.Log** method, like the following:

```csharp
public class TurnManager
{
privateint m_TurnCount;
publicTurnManager()
{       
m_TurnCount=1;
}
public void Tick()
{       
m_TurnCount+=1;       
Debug.Log("Current turn count : "+ m_TurnCount);
}
}
```

- Then you’ll need to call **Tick** every time the player moves. But how to do this? The only place that has a reference to the **TurnManager** is inside the **GameManager**.
- You could pass the **GameManager** to our **Player**, like you did for the **BoardManager**, but as a lot of other things in your game may require access to the **GameManager**, it’s a better idea to explore another method: the singleton pattern.
- Using a singleton pattern means that there can only be one instance of a given class in your game/application and you can access it through a static member accessible from anywhere.
- It is a powerful tool that makes code faster to write and simpler to understand. In the case of the **TurnManager** script, the drawbacks of a singleton pattern are worth it in exchange for not having to pass the **GameManager** all around your application code to every object that needs it.

This is our new singleton **GameManager**:

```csharp
public class GameManager:MonoBehaviour
{
public static GameManager Instance
{
get
;
private set;
}
public BoardManager BoardManager;
public PlayerController PlayerController;
private TurnManager m_TurnManager;
private void Awake()
{
if(Instance!=null)
{
Destroy(gameObject);
return;
}
       Instance=this;
       }
voidStart()
{       
m_TurnManager=newTurnManager();
BoardManager.Init();       
PlayerController.Spawn(BoardManager,newVector2Int(1,1));
}
}
```

- There is a static **GameManager** property member called **Instance**. This will store a reference to your GameManager. There are two keywords: get and set. **set** is preceded by the **private** keyword. What this means is that accessing that member variable (get) is available for all parts of the code, but writing the value of that member variable (set) can only be done from inside the class.
- A new method called **Awake** was added. This is called before **Start** when the GameObject is created. In this method, the following happens:
    - The code tests if **Instance** is null. If it’s not, that means there’s already a **GameManager**. This shouldn’t happen because a singleton pattern needs to be unique, so we destroy this GameManager and exit the method (as this is a void type method, “return;” doesn’t return anything).
    - If there is no **GameManager** stored yet (**Instance** is null) we store that **GameManager** in **Instance**.
- Now you can use the following instance anywhere in your code to access the **GameManager**:
    
    GameManager.instance
    
- Now if you change the **TurnManager** variable of the **GameManager** to be **public**, other scripts will also be able to access it through **GameManager.Instance**. Define the **TurnManager** **set** property as **private** so only the **GameManager** script can change this variable, but leave the **get** property **public** so other scripts can access the **TurnManager**.

```csharp
public class GameManager:MonoBehaviour
{
public static GameManager Instance
{
get;
private set;
}
public BoardManager BoardManager;
public PlayerController PlayerController;
public TurnManager TurnManager
{
get;
private set;
}
private voidAwake()
{
if(Instance!=null)
{
Destroy(gameObject);
return;
}
       Instance=this;
       }
voidStart()
{       
TurnManager=newTurnManager();
BoardManager.Init();       
PlayerController.Spawn(BoardManager,newVector2Int(1,1));
}
}
```

Then in the **Update** method of your **PlayerController**, you can “tick” the **TurnManager** just before calling **MoveTo**:

```csharp
if(hasMoved)
{
//check if the new position is passable, then move there if it is.
BoardManager.CellData cellData= m_Board.GetCellData(newCellTarget);
if(cellData!=null&& cellData.Passable)
{       
GameManager.Instance.TurnManager.Tick();
MoveTo(newCellTarget);
}
}
```

Now you can try to play the game and check that a new turn count is printed in the console every time you move.

![image](images/image.png)