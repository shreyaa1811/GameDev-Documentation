# 7.  Add Win and Loose conditions

## 7.1 Finishing A level

Let’s start by writing a new type of **CellObject**, **ExitCellObject**, that sets the tile to an exit tile (so the player knows where the exit is) and reacts to the player character entering that cell by generating a new level.

To do this, you’ll need to do the following:

- Create a new **ExitCellObject** subclass that inherits from **CellObject**.
- Create a new exit tile (there’s an exit sign sprite you can use for this).
- Give the exit tile as a public reference to your new **ExitCellObject**.
- Create a prefab of this new type of **CellObject**.
- Give that prefab as a new public reference to your **BoardManager** so it can add the object to the exit cell (Use the upper-right empty cell as the exit position).

```csharp
using UnityEngine;
using UnityEngine.Tilemaps;
public class ExitCellObject:CellObject
{
public Tile EndTile;
public override void Init(Vector2Int coord)
{
base.Init(coord);       
GameManager.Instance.BoardManager.SetCellTile(coord, EndTile);
}
publicoverridevoidPlayerEntered()
{       
Debug.Log("Reached the exit cell");
}
}
```

**1.** Add a new empty GameObject to your scene, rename it “ExitCell”, and add the above script to it.

**2.** In the **Inspector** window, assign the tile asset you want to use for your exit to the **EndTile** property, then create a prefab of this **ExitCell** GameObject and delete it from the scene.

**3.** Add a reference to the ExitCell prefab in your **BoardManager** and spawn that GameObject in the upper-right corner (at the opposite end of the player character’s starting position) during the level creation phase.

**4.** Inside the **Init** function, after the **for** loops that generate the board, add the following code:

```csharp
public class BoardManager:MonoBehaviour
{
public Exit CellObject ExitCellPrefab;
...
publicvoidInit()
{
...for(int y=0; y< Height;++y)
{...
}
   m_EmptyCellsList.Remove(newVector2Int(1,1));
Vector2Int endCoord=newVector2Int(Width-2, Height-2);
AddObject(Instantiate(ExitCellPrefab), endCoord);   
m_EmptyCellsList.Remove(endCoord);
GateWall();GenerateFood();
 }
```

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/9d46d683-a30d-4fc9-88f6-9827dbe97bc2_game-view-exit.png)

### **7.2 Generating a new level**

- **BoardManager** will need a new function that cleans the current level. This means going over every cell, removing their tile from the tilemap, deleting the data saved inside them and destroying their CellObject.
- **GameManager** will need a **NewLevel** function that will clean the current board, create a new one, and spawn the player character at the start of this new board.

### **BoardManager**

```csharp
public void Clean()
{
//no board data, so exit early, nothing to clean
if(m_BoardData==null)
return;

for(int y=0; y< Height;++y)
{
for(int x=0; x< Width;++x)
{
var cellData= m_BoardData[x, y];
if(cellData.ContainedObject!=null)
{
//CAREFUL! Destroy the GameObject NOT just cellData.ContainedObject
//Otherwise what you are destroying is the JUST CellObject COMPONENT
//and not the whole gameobject with sprite
Destroy(cellData.ContainedObject.gameObject);
}
SetCellTile(newVector2Int(x,y),null);
}
}
}
```

### **GameManager**

```csharp
private int m_CurrentLevel=1;
...voidStart()
{   
TurnManager=newTurnManager();   
TurnManager.OnTick+= OnTurnHappen;
NewLevel();
   m_FoodLabel= UIDoc.rootVisualElement.Q<Label>("FoodLabel");   
   m_FoodLabel.text="Food : "+ m_FoodAmount;
   }
public void NewLevel()
{   
BoardManager.Clean();   
BoardManager.Init();   
PlayerController.Spawn(BoardManager,newVector2Int(1,1));
   m_CurrentLevel++;
   }
```

### **ExitCellObject**

```csharp
public override void PlayerEntered()
{
   GameManager.Instance.NewLevel();
   }
```

## 7.3 Game Over

For your game to have an ending, let’s add a **Game Over** state to the game. This will occur when the food count reaches 0. After that, this is what needs to happen:

1. The UI text display will need to change to a GameOver message that shows how many levels the player survived before running out of food.
2. The game will need to be set to a **Game** **Over** state that disables reading input so the user can’t move the player character anymore.
3. The whole game will need to restart at level 1 when a certain key is pressed to create a game loop.

Let’s start with the UI game over message, which will consist of a semi transparent background that darkens the screen and a text in the middle that shows the number of completed levels:

**1.** In the **UI** folder of the **Project** window, double click the uxml document **GameUI** to open the **UI builder** window.

**2.** In the **Library** window, in the **Containers** section, click and drag a **VisualElement** into the **Hierarchy** window and rename it “GameOverPanel”.

Make sure it’s placed below after the **FoodLabel** in the **Hierarchy** window, as objects are rendered from top to bottom, so being below after **FoodLabel** means it will be rendered above the **FoodLabel**.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/dd0ecf0f-0271-474d-9747-1ae230015db0_game-over-panel.png)

**3.** With the **GameOverPanel** Element selected, use the foldout (triangle) to expand the **Position** section in the **Inspector** window, then open the **Position Mode** dropdown and select **Absolute** and set the **Top**, **Right**, **Bottom** and **Left** offsets to **0**, to make sure it covers the entire screen.

You don’t want the **GameOverPanel** Element to be part of the layout, you want it to be over everything.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/e4f7e751-b6c7-4567-bf97-6fbfb5bb734e_game-over-panel-anchors.png)

**4.** Use the foldout (triangle) to expand the **Background** section and set the **Color** to black (**000000**) but with an **alpha** (**A**) value of ~**200**.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/6167f056-af5d-4558-88b8-de6ff2329871_background-color-wheel.png)

**5.** In the **Library** window, in the **Controls** section, click and drag a **Label** into the **Hierarchy** directly on top of the **GameOverPanel** Element , then rename that **Label** Element “GameOverMessage”.

![](https://connect-mediagw.unity.com/h1/20250820/learn/images/33953bc5-6eb9-45cd-82d2-9d3e03245eff_Untitled_44.png)

**6.** With **GameOverMessage** selected, use the foldout (triangle) to expand the **Attributes** section and set the **Text** property to “Game Over”. Then use the foldout (triangle) to expand the **Inlined** **Styles** section, then the **Text** section and change the **Font**, **Size**, and **Color** of the text to make it visible on the dark background.

The Game Over **Text** property is a placeholder to help you design its look; the real message will be written through code when a game over event happens.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/c7899b96-2558-4101-8599-1b4c1362d3df_text-game-over-panel.png)

**7.** Select the **GameOverPanel** GameObject and use the foldout (triangle) to expand the **Inlined** **Styles** section, then the **Align** section in the **Inspector** window. Set the **Align Items** property to **center** and the **Justify Content** property to **space-around**.

![](https://connect-mediagw.unity.com/h1/20240927/learn/images/7ff4259d-d499-463f-a496-c88768465664_align-styles.png)

- One of type **VisualElement**, to store the reference to the **GameOverPanel** Element.
- One of type **Label** to store a reference to the **GameOverMessage** Element so you can update it when the **GameOver** condition happens.

```csharp
private VisualElement 
m_GameOverPanel;
private Label m_GameOverMessage;
```

**8.** In the **Start** function of the **GameManager,** you need to retrieve The **GameOverPanel** VisualElement and the **GameOverMessage** Label from the **UI Document** root **VisualElement** and set the visibility of the panel to **Hidden**:

```csharp
m_GameOverPanel= UIDoc.rootVisualElement.Q<VisualElement>("GameOverPanel");
m_GameOverMessage= m_GameOverPanel.Q<Label>("GameOverMessage");
m_GameOverPanel.style.visibility= Visibility.Hidden;
```

**9.** In the **ChangeFood** function of the **GameManager**, you need to test if the food Amount is 0 or less. If it is, this is the Game Over condition, which means you need to set the visibility of the **GameOverPanel** GameObject to **Visible** and display the message with the number of completed levels.

```csharp
public void ChangeFood(int amount) 
{  
 m_FoodAmount+= amount;   
 m_FoodLabel.text="Food : "+ m_FoodAmount;
if(m_FoodAmount<=0)
{       
m_GameOverPanel.style.visibility= Visibility.Visible;      
m_GameOverMessage.text="Game Over!\n\nYou traveled through "+ m_CurrentLevel+" levels";
}
}
```

### **Game Over state**

1. Create a bool variable in the **PlayerController** that stores whether the player character is in a **Game** **Over** state or not.
2. Early exit the **Update** function to disable input handling. Make the **GameManager** have a reference to the **Player Controller** script so it can easily call functions on it.

### **PlayerController**

```csharp
using UnityEngine;
using UnityEngine.InputSystem;
public class PlayerController:MonoBehaviour
{
privatebool m_IsGameOver;
...
publicvoidGameOver()
{   
m_IsGameOver=true;
}
...
private void Update()
{
if(m_IsGameOver)
{
return;
}...}
```

### **GameManager**

```csharp
public void ChangeFood(int amount) 
{     
m_FoodAmount+= amount;     
m_FoodLabel.text="Food : "+ m_FoodAmount;
if(m_FoodAmount<=0)
{         
PlayerController.GameOver();         
m_GameOverPanel.style.visibility= Visibility.Visible;         
m_GameOverMessage.text="Game Over!\n\nSurvived "+ m_CurrentLevel+" days";
}
}
```

1. Move the part of the **Start** function in the **GameManager** that handles initializing the game into a new function called **StartNewGame**.

Things like getting references to UI elements or creating new objects like the **TurnManager** only needs to happen once when the game is launched, not every new game, and can be left in the **Start** function.

1. The **PlayerController** needs to check if the **Enter** key is pressed only during game over, and if it's pressed it starts a new game.

### **GameManager**

```csharp
void Start()
{   
TurnManager=newTurnManager();   
TurnManager.OnTick+= OnTurnHappen;
   m_FoodLabel= UIDoc.rootVisualElement.Q<Label>("FoodLabel");
   m_GameOverPanel= UIDoc.rootVisualElement.Q<VisualElement>("GameOverPanel");   
   m_GameOverMessage= m_GameOverPanel.Q<Label>("GameOverMessage");
Start NewGame();
}
public void Start NewGame()
{   
m_GameOverPanel.style.visibility= Visibility.Hidden;
   m_CurrentLevel=1;   
   m_FoodAmount=20;   
   m_FoodLabel.text="Food : "+ m_FoodAmount;
  BoardManager.Clean();   
  BoardManager.Init();
   PlayerController.Init();   
   PlayerController.Spawn(BoardManager,newVector2Int(1,1));
   } 
```

- Hiding the GameOver message.
- Setting the level back to 1 at the start.
- Setting the starting food value (before it was only set through the default value, which would not be reset when just calling that function).
- Initializing the board.
- Initializing the player character and spawning it.

### **PlayerController**

```csharp
public void Init()
{   
m_IsGameOver=false;
}
private void Update()
{
if(m_IsGameOver)
{
if(Keyboard.current.enterKey.wasPressedThisFrame)
{           
GameManager.Instance.StartNewGame();
}
return;
}
...}
```