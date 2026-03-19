# 9. Adding an enemy

## 9.1 Create A Basic enemy

**1.** They will act on each turn of the game.

**2.** They are essentially a **CellObject** that moves, so they act like walls in the sense that they stop the player character from entering the cell where they are, and receiving damage can destroy them when their hit points reach **0**.

**3.** They can damage the player character (reducing their food amount by **1**) if the player character is adjacent to them when the turn happens.

**4.** If the player character is not next to them when the turn happens, the enemies will move one cell closer to them.

**5.** Your **Enemy** script inherits from **CellObject**.

**6.** Enemies will need a notification when a turn happens. Check how you did this when reducing the food on every turn. Don’t forget to add an **OnDestroy** function to the enemies to remove the callback! Otherwise the callback will try to still notify the destroyed enemy that a turn happened.
**7.** When a turn happens, enemies check where the player character is. If they are next to them, they damage the player character (remove some food points), but if they are not next to the player character, they move one cell closer.

**8.** When enemies move, because they are a **CellObject**, they need to remove themselves from the cell they move from and add themselves to the cell they move to.

**9.** Start with creating the GameObject with just the idle animation of the enemy. This will be the base of all your work.

**10.** Create your **Enemy** script as a subclass of **CellObject**, but leave all its functions empty for now.

**11.** Create a prefab from that GameObject and try to spawn that Enemy prefab into the **BoardManager** (remember they are **CellObjects**!)

**12.** Now you can test that your enemy appears in the game when you play! You might already have a bug to fix as your enemy might not appear. Check your **Hierarchy** window and, if it’s in the **Hierarchy** window but not visible in the **Scene** view, try to remember what you had to make the player character and food sprites appear on top of the tilemap.

**13.** Once you confirm your enemy appears in the game, it’s time to start coding it! Modify your **Enemy** script so that it stops the player character from entering its cell, and instead takes damage (you need to add health tracking to the **Enemy** script) and gets destroyed once its hit points reach **0**. In short, make it exactly like a **WallObject**.

**14.** Once this is confirmed to work, make the enemy move towards the player character each turn! You need to find which cell the enemy needs to move to get closer to the player. This is probably the hardest part. Try different codes to do this.

**15.** Now test to see if the enemy is trying to get closer to the player character. At this step, you might encounter a bug where the enemy gets under the player. Don’t forget to stop the enemy trying to get to the player character if it’s adjacent from it: the enemy should never try to enter the player character’s cell!

### **Enemy**

```csharp
public class Enemy : CellObject
{
    public int Health = 3;
    private int m_CurrentHealth;

    private void Awake()
    {      
        GameManager.Instance.TurnManager.OnTick += TurnHappened;
    }

    private void OnDestroy()
    {       
        GameManager.Instance.TurnManager.OnTick -= TurnHappened;
    }

    public override void Init(Vector2Int coord)
    {
        base.Init(coord);
        m_CurrentHealth = Health;
    }

    public override bool PlayerWantsToEnter()
    {       
        m_CurrentHealth -= 1;

        if (m_CurrentHealth <= 0)
        {
            Destroy(gameObject);
        }

        return false;
    }

    bool MoveTo(Vector2Int coord)
    {
        var board = GameManager.Instance.BoardManager;
        var targetCell = board.GetCellData(coord);

        if (targetCell == null || !targetCell.Passable || targetCell.ContainedObject != null)
        {
            return false;
        }

        // remove enemy from current cell
        var currentCell = board.GetCellData(m_Cell);
        currentCell.ContainedObject = null;

        // add it to the next cell
        targetCell.ContainedObject = this;
        m_Cell = coord;

        transform.position = board.CellToWorld(coord);
        return true;
    }

    void TurnHappened()
    {
        // Player current cell
        var playerCell = GameManager.Instance.PlayerController.Cell;

        int xDist = playerCell.x - m_Cell.x;
        int yDist = playerCell.y - m_Cell.y;

        int absXDist = Mathf.Abs(xDist);
        int absYDist = Mathf.Abs(yDist);

        // Adjacent to player → attack
        if ((xDist == 0 && absYDist == 1) || (yDist == 0 && absXDist == 1))
        {
            GameManager.Instance.ChangeFood(3);
        }
        else if (absXDist > absYDist)
        {
            // Try moving in X first
            if (!TryMoveInX(xDist))
            {
                // If X move fails, try Y
                TryMoveInY(yDist);
            }
        }
        else
        {
            // Try moving in Y first
            if (!TryMoveInY(yDist))
            {
                TryMoveInX(xDist);
            }
        }
    }

    bool TryMoveInX(int xDist)
    {
        // Player to the right
        if (xDist > 0)
        {
            return MoveTo(m_Cell + Vector2Int.right);
        }

        // Player to the left
        return MoveTo(m_Cell + Vector2Int.left);
    }

    bool TryMoveInY(int yDist)
    {
        // Player above
        if (yDist > 0)
        {
            return MoveTo(m_Cell + Vector2Int.up);
        }

        // Player below
        return MoveTo(m_Cell + Vector2Int.down);
    }
}

```