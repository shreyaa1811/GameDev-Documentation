# 2. Creating the game

Now you’ll start working on the game itself. The first thing you’ll need to do is generate a game board through code.

## 2.1 Creating the game board

1. The provided project should already contain a **Scenes** folder, but if it doesn’t, in the **Project** window, right-click the **Assets** folder and select **Create** > **Folder** and name it “Scenes”.
2. Right-click the **Scenes** folder and select **Create** > **Scene** > **Scene** to create a new empty scene, and name it “Main”.
3. Double click **Main** scene to open it. In the **Hierarchy** window, select the **Main** **Camera** GameObject, then in the **Inspector** window, under the **Camera** section, ensure that the **Projection** property is set to **Orthographic** and the **Size** property is set to **5**.

![Image](images/25f6895b-e1f9-4b30-b305-f1122aacf250_camera-orthograpic.png)

1. Under the **Environment** section of the **Camera** settings, ensure the **Background Type** property is set to **Solid Color** and set the **Background Color** property to black (**000000**).

![Image](images/de7de6a8-8858-4876-8852-ebe253c70e28_camera-background-black.png)

1. Right-click in the **Hierarchy** window and select **2D Object** > **Tilemap** > **Rectangular**.
2. In the **Project** window, right-click the **Assets** folder and select **Create** > **Folder** to create a new folder, then name it “Tiles”. Right-click the **Tiles** folder and select **Create** > **2D** > **Tile Palette** > **Rectangular** and name the new asset “GamePalette”.
3. From the main menu, select **Window** > **2D** > **Tile Palette**.
    
    This opens the **Tile Palette** window.
    

![Image](images/d0affab1-15b8-4efa-8834-1f4296a1a0b3_tile-palette-window.png)

1. In the **Project** window, select **Assets** > **Roguelike2D** > **TutorialAssets** > **Sprites**. Choose whichever Sprite Sheet you want, use the foldout (triangle) to expand it, and select and drop all the ground tiles into the center of the **Tile Palette** window.

The Editor will ask you to select a folder where you want the new tiles to be saved; select the **Tiles** folder you just created.

## 2.2 Scripting

Now that you have your ground tiles, let’s write the code that creates the board where the game will happen on each level:

**1.** In the **Hierarchy** window, rename the **Grid** GameObject “BoardManager”.

**2.** In the **Project** window, right-click the **Assets** folder and select **Create** > **Folder** and name it “Scripts”.

**3.** Right-click the **Scripts** folder and select **Create** > **Scripting** > **MonoBehaviour Script**, name it “BoardManager”, and add it as a component of the **BoardManager** GameObject.

**4.** Double-click the new **BoardManager** script to open it in your code editor.

You’ll need to create the following three variables that you can set in the Editor:

* The width (in number of tiles) of our level.
* The height (in number of tiles) of our level.
* An array of tiles that are going to be used for the board.

First, in the **Start** method, you need to get the Tilemap component from the child GameObjects of the GameObject the script is on (the **Tilemap** GameObejct is a child GameObject of the **BoardManager** GameObject) and store it in a private member variable for easy access later. Then the code needs to go over all the tiles in the Tilemap component and randomly select a ground tile.

The method to set a tile from a tilemap is **SetTile**, which takes as first parameter a Vector3 Int with the coordinates of the tile (so (0, 0, 0) is the first cell, (1, 0, 0) the one on its right, (0, -1, 0) the one below etc. z is always 0 in this case as you work in 2D) and the second parameter is the actual tile to be set at that position.

**Note**: Don’t forget to add the **using UnityEngine. Tilemaps** namespace at the beginning of your file so you can have access to the Tilemap and Tile classes.

Here is the solution:

```csharp
using UnityEngine;
using UnityEngine.Tilemaps;

public class BoardManager : MonoBehaviour
{
	private Tilemap m_Tilemap;
	
	public int Width;
	public into Height;
	public Tile[] GroundTiles;
	
	//Start is called before first frame update
	void Start()
	{
		m_Tilemap = GetComponentInChildren<Tilemap>();
		
		for (int y = 0; y < Height; ++y)
		{
			for (int x = 0; x < Width; ++x)
			{
				int tileNumber = Random.Range(0, GroundTiles.Length);
				m_Tilemap.SetTitle(new Vector3Int(x,y,0),GroundTiles[titleNumber]);
			}
		}
	}
}
```

5.  After completing your script, make sure to save it to compile the changes so they are visible in the Editor.

**6.**  Select the **BoardManager** GameObject. In the **Inspector** window, set the **width** and **height** variables of the Board Manager script component to **8**.

**7.**  Use the foldout (triangle) to expand the Ground Tiles section, then select the **Add** (**+**) button. Use the picker (**⊙**) and select a tile for the ground. Try to add a couple of different tiles.

**8.**  In the **Game** view toolbar, select the **Play** button to enter Play mode.

You can now see the game board in both the **Scene** view and the **Game** view!

Note that the **Game** view will only show part of the board, but you will fix the camera later!

```csharp
void Start()
{
    m_Tilemap = GetComponentInChildren<Tilemap>();

    for (int y = 0; y < Height; ++y)
    {
        for (int x = 0; x < Width; ++x)
        {
            Tile tile;

            if (x == 0 || y == 0 || x == Width - 1 || y == Height - 1)
            {
                tile = WallTiles[Random.Range(0, WallTiles.Length)];
            }
            else
            {
                tile = GroundTiles[Random.Range(0, GroundTiles.Length)];
            }

            m_Tilemap.SetTile(new Vector3Int(x, y, 0), tile);
        }
    }
}
```

## 2.3 Camera

As your tilemap is at **0**, **0**, **0**, and it is **8** by **8**, its center is at (**4**, **4**, **0**) so set the **Main Camera Position** to (**4**, **4**, **-10**) so it aims at the center of your board.

Now, when you try to enter Play mode, your board will look something like this:

![Image](images/93e360bd-ecbc-4fca-a5b8-ff1399953f01_game-view-board.png)

## 2.4 Board Data

* As of now, your board is only a collection of tiles, which are just visual. For the game to really function, you'll need to keep more data for each cell than just its sprites.
* To do this, you'll use a 2D array, which is an array that has two indices (in this case, the x and y of the cell). This array type will be a custom C# class, which will allow you to store any kind of data per cell.
* At the top of your **BoardManager** class, add a new class called “CellData” and make its first member variable a boolean that defines if the cell is passable or not. And then let’s declare a private property that is our 2D array board data that we will name **m_BoardData**.

```csharp
public class BoardManager : MonoBehaviour
{
    public class CellData
    {
        public bool Passable;
    }

    private CellData[,] m_BoardData;
}
```

* **CellData[, ]** is the way to declare a 2D array containing objects of type **CellData**. The “, ” denotes there will be two indices to access it. So **m_BoardData[0, 0]** is the first item of the first line,  **m_BoardData[1, 3]** the fourth item of the second line (indices start at 0!), etc.
* In C# you can create an array of any dimension, so **CellData[, , ]** would be a 3D array (three indices)
* Finally you can update your **Start** method. To do this, you’ll need to do the following:
    - Initialize the m_BoardData array with your board width/height so it’s size is enough to store the data of all the cells.
    - Create each CellData inside your loops and set its Passable value to **true** if the cell is not a border and **false** if it is a border.

```csharp
void Start()
{
    m_Tilemap = GetComponentInChildren<Tilemap>();
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
}

```
