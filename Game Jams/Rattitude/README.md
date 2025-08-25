# Rattitude

[itch.io](https://lynche.itch.io/rattitude)

*Worked as Gameplay Programmer*

18th of October 2024 until 20th of October 2024.

![Rattitude enemy pathfinding](rattitude.gif)

This was one of my biggest challenges as a programmer up until that point. I had never worked with pathfinding or A* before. And to top it all off, it was ona a three day game jam!

## My part in the team
I spent 12 about how A* is implemented and making the code from scratch. We used the 2D tilemap grid to make the movement and then I used A* to make the enemy AI/pathfinding.

## Code snippet
The enemy movement that uses A* pathfinding.

<details>
<summary>Show me!</summary>

```cs
public class EnemyMovement : GridMovement
{
    [SerializeField] private PlayerMovement player;

    [SerializeField] private int maxDirectDistanceBeforeMove = 100;
    [SerializeField] private int maxDistanceScore = 300;
    [SerializeField] GameObject deathParticlePrefab;
    
    ...

    private void OnEnable() => player.onMove.AddListener(FindLowestScore);

    private void OnDisable() => player.onMove.RemoveListener(FindLowestScore);

    public void GetHit()
    {
        if (hasBeenHit) return;
        hasBeenHit = true;
        transform.GetChild(0).GetComponent<SpriteRenderer>().color = Color.grey;
        FindFirstObjectByType<GameManager>().enemies.Remove(this);
        Invoke(nameof(DeathParticles), 0.5f);
        Destroy(gameObject, 0.5f);
    }

    ...

    private void FindLowestScore()
    {
        if (hasBeenHit) return;
        int currentDistance = CalculateDistance(position, player.position);

        if (currentDistance > maxDirectDistanceBeforeMove)
            return;

        List<Cell> openCells = new();
        List<Cell> closedCells = new();

        Cell startCell = InitializeFirstCell();
        Cell cellToExplore = startCell;

        int ranTimes = 0;

        while (cellToExplore.position != player.position && ranTimes < 1000)
        {
            ranTimes++;

            int lowestDistanceScore = maxDistanceScore;
            Cell neighbourCell = AddNeighboursToOpenList(openCells, cellToExplore, lowestDistanceScore);

            if (neighbourCell is null)
            {
                foreach (Cell openCell in openCells)
                {
                    if (openCell.distanceScore >= lowestDistanceScore)
                        continue;

                    lowestDistanceScore = openCell.distanceScore;
                    cellToExplore = openCell;
                }
            }
            else
            {
                cellToExplore = neighbourCell;
            }

            closedCells.Add(cellToExplore);
            openCells.Remove(cellToExplore);

            if (openCells.Count == 0)
                break;
        }

        List<Cell> options = new();

        foreach (Cell closedCell in closedCells)
        {
            Vector3Int middlePoint = startCell.position - closedCell.position;

            if (middlePoint is { x: <= 1 and >= -1, y: 0 } or { x: 0, y: <= 1 and >= -1 })
                options.Add(closedCell);
        }

        if (options.Count == 0)
            return;

        Cell nextCell = options.Aggregate((cellWithLowestTarget, cell) => cell.distanceFromTarget <= cellWithLowestTarget.distanceFromTarget ? cell : cellWithLowestTarget);

        if (nextCell.distanceFromTarget <= 10)
            player.AttackedByEnemy(this);

        if (gameManager.enemies.Any(enemy => enemy.position == nextCell.position))
            return;

        if (nextCell.distanceFromTarget < 10)
            return;

        if (nextCell.distanceFromTarget > maxDirectDistanceBeforeMove)
            return;

        Vector3Int direction = nextCell.position - startCell.position;
        Move(direction);
    }

    ...
}
```
</details>