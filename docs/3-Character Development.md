# 3. Character Development

## 3.1 Adding Player Character

To add a **PlayerCharacter**

**1.** Choose one of the player character sprites in the Sprite Sheet in the **Assets** > **Roguelike2D** > **TutorialAssets** > **Sprites** folder and drag it into the scene to automatically create a new GameObject with a Sprite Renderer component using that sprite. Rename that GameObject “PlayerCharacter”.

**2.** Inside the **Scripts** folder, create a new MonoBehaviour script called “PlayerController” and add it to the **PlayerCharacter** GameObject you just created.

- At the start, the board sets the player on a specific cell after it finishes generating the level.
- When the user presses a direction button (up, down, left, or right arrow keys), the script checks if the cell in that direction is passable.
- If the cell is passable, the script moves the character to that new cell.

This outline should help you realize the following about what your code needs to do to achieve this functionality:

- When the board places the player character, the player character will need a **Spawn** method that will move it to the right spot.
- The script needs to know where the player character currently is in order to search for the next cells the player character can move to, so it will need to store its current cell index.
- As the script needs to know if the cell the player character is trying to move to is passable, and that information is stored in the **BoardManager**, the script will need to store a reference to the **BoardManager** to query it about the state of a given cell.

**3.** Inside your new **PlayerController** script, add the following methods and variables:

- A private variable of type **BoardManager**.
- A private variable of type **Vector2Int** that saves the current cell the player is on.
- A public method called “Spawn” that saves the BoardManager that the player is placed in and the index where it is currently.

```csharp
usingUnityEngine;

public class PlayerController:MonoBehaviour
{
private BoardManager m_Board; 
private Grid m_Grid;
private Vector2Int m_CellPosition;
public Vector3 CellToWorld(Vector2Int cellIndex)
    {
        return m_Grid.GetCellCenterWorld((Vector3Int)cellIndex);
    }
public void Spawn(BoardManager boardManager,Vector2Int cell)
{       
m_Board= boardManager;       
m_CellPosition= cell;
public void Spawn(BoardManager boardManager, Vector2Int cell)
{
   m_Board = boardManager;
   m_CellPosition = cell;

   //let's move to the right position
   transform.position = m_Board.CellToWorld(cell);
}
```

1. Add the following code into your BoardManager Script:

```csharp
public PlayerController Player;
// Start is called before the first frame update 
voidStart()
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
Player.Spawn(this,newVector2Int(1,1));
}
```

The above code does the following things:

- Adds a public member variable (**PlayerController)** to the **BoardManager** so you can assign your **PlayerCharacter** GameObject to it in the **Inspector** window.
- At the end of the **Start** method, after the board is generated, it calls the **Spawn** method on that **PlayerController** script with a given cell (for example (1,1) the first lower-left cell that is not a wall).
1. In the Editor, drag the **PlayerCharacter** GameObject from the scene into the **Player** slot on the **BoardManager** script, then enter Play mode.

![](https://connect-mediagw.unity.com/h1/20241010/learn/images/acd57f69-31e1-48cb-bb66-65a536b0518a_game-view-board.png)

**7.** Select the **PlayerCharacter** GameObject in the **Hierarchy** window and look at the **Scene** view.

You'll see that your player character is in the right place, but the board is rendered above it:

![image](images/image.png)

**8.** In the **Inspector** window, use the foldout (triangle) to expand the Sprite Renderer component and set the **Order in Layer** property **10**.

Setting the player character to level 10 gives you space in the future to place objects that should be above the board but under the player character.

![](https://unity-connect-prd.storage.googleapis.com/20250825/learn/images/e9369e0f-0661-4fee-9319-8dca3f951f16_Untitled_53.png)

## 3.2 Moving Player Character

The PlayerController script will need to do the following:

- Inside the **Update** method of the **PlayerController** script, check if an arrow key is pressed.
- Check if the cell in that direction is passable.
- If the cell is passable, move the player to the new cell.

The code below shows one possible way of doing this.

 Note that you’ll need to add the **UnityEngine.InputSystem** library at the top of your file:

```csharp
private void Update()
{
Vector2Int newCellTarget= m_CellPosition;
bool hasMoved=false;
if(Keyboard.current.upArrowKey.wasPressedThisFrame)
{       
newCellTarget.y+=1;       
hasMoved=true;
}
else if(Keyboard.current.downArrowKey.wasPressedThisFrame)
{       
newCellTarget.y-=1;       
hasMoved=true;
}
else if(Keyboard.current.rightArrowKey.wasPressedThisFrame)
{       
newCellTarget.x+=1;       
hasMoved=true;
}
else if(Keyboard.current.leftArrowKey.wasPressedThisFrame)
{       
newCellTarget.x-=1;       
hasMoved=true;
}
if(hasMoved)
{
//check if the new position is passable, then move there if it is.}}
```

Just like with the **BoardManager** GameObject, you have no way of getting the information about whether a cell is passable. As before, retrieving the cell data of a specific cell is something you'll need to do a lot, so add the following method for this to your **BoardManager** script:

```csharp
public CellData GetCellData(Vector2Int cellIndex) 
{
if(cellIndex.x<0|| cellIndex.x>= Width|| cellIndex.y<0|| cellIndex.y>= Height)
{
return null;
}
return m_BoardData[cellIndex.x, cellIndex.y];
}
```

This method returns the information saved on the array **m_BoardData** of that cell. Restricting the search on the array to only the actual cells available in that level so you don’t generate an exception trying to index a cell that doesn’t exist. In that case, the method returns null.

You can now update your **Update** method in your **PlayerController** script so it uses the following method:

```csharp
private void Update()
{
Vector2Int new CellTarget= m_CellPosition;
bool hasMoved=false;
if(Keyboard.current.upArrowKey.wasPressedThisFrame)
{        
newCellTarget.y+=1;        
hasMoved=true;
}
else if(Keyboard.current.downArrowKey.wasPressedThisFrame)
{        
newCellTarget.y-=1;        
hasMoved=true;
}
else if(Keyboard.current.rightArrowKey.wasPressedThisFrame)
{        
newCellTarget.x+=1;        
hasMoved=true;
}
else if(Keyboard.current.leftArrowKey.wasPressedThisFrame)
{        
newCellTarget.x-=1;        
hasMoved=true;
}
if(hasMoved)
{
//check if the new position is passable, then move there if it 
is.BoardManager.CellData cellData= m_Board.GetCellData(newCellTarget);
if(cellData!=null&& cellData.Passable)
{
m_CellPosition= newCellTarget;            
transform.position= m_Board.CellToWorld(m_CellPosition);
}
}
}
```

## 3.3 Refactoring

- In a prototyping environment like this one, where you keep finding solutions to problems that appear as you progress, it’s good to take a second to look at your code and check if you have areas that you can clean up.
- In this case, you may have noticed some code duplication: you move the player in the **Spawn** method and also later in the **Update** method.
- Code duplication is not always bad, but in this case it could create errors: when the character moves, both the **CellPosition** and the **Transform** position need to be updated. It would be easy to forget one of the two if you need to move the character elsewhere in the code. Plus, there might be things you want to add to your code later; for example, adding a sound every time the character moves. To do this, you would have to track every place you move the character to add that sound.
- It would be safer and more efficient to wrap all of that functionality into a **MoveTo** method that takes a new cell the player will move to as a parameter and takes care of everything related to moving the **PlayerCharacter** GameObject.
- Copy and paste the **MoveTo** method code below into your **PlayerController** script to add that method and replace both movements in the **Start** and **Update** methods with it:

```csharp
using UnityEngine;
using UnityEngine.InputSystem;
public class PlayerController:MonoBehaviour
{
private BoardManager m_Board;
private Vector2Int m_CellPosition;
public void Spawn(BoardManager boardManager,Vector2Int cell)
{       
m_Board= boardManager;MoveTo(cell);
}
public void MoveTo(Vector2Int cell)
{       
m_CellPosition= cell;       
transform.position= m_Board.CellToWorld(m_CellPosition);
}

private voidUpdate()
{
Vector2Int newCellTarget= m_CellPosition;
bool hasMoved=false;
if(Keyboard.current.upArrowKey.wasPressedThisFrame)
{           
newCellTarget.y+=1;           
hasMoved=true;
}
else if(Keyboard.current.downArrowKey.wasPressedThisFrame)
{           
newCellTarget.y-=1;           
hasMoved=true;
}
else if(Keyboard.current.rightArrowKey.wasPressedThisFrame)
{           
newCellTarget.x+=1;           
hasMoved=true;
}
else if(Keyboard.current.leftArrowKey.wasPressedThisFrame)
{           
newCellTarget.x-=1;           
hasMoved=true;
}
if(hasMoved)
{
//check if the new position is passable, then move there if it 
is.BoardManager.CellData cellData= m_Board.GetCellData(newCellTarget);
if(cellData!=null&& cellData.Passable){MoveTo(newCellTarget);
}
}
}
}
```