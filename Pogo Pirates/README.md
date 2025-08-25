# Pogo Pirates

[itch.io](https://yrgo-game-creator.itch.io/pogopirates)

*Worked as Gameplay Programmer*

14th November 2024 to 3rd January 2025.

I loved working with this game. It taught me a lot about programming design patterns, when to make critical decisions and my ability to work in a team.

## Gameplay
During the creation of Pogo Pirates, I worked with:

* The join lobby screen
* The core gameloop
* Character movement and actions

Among these I developed or used:

* Singletons and do not destroy in Unity
* Unity's new input system to develop a local multiplayer game
* State machines
* Observer patterns

## Teamwork
Our team used scrum to create a good flow. We used weekly sprints and set up goals to be completed and helped each other out whenever needed.

## Code snippet
The user class was used to keep track of a player's game info and instantiate their parts into the game.

```cs
public class User
{
    public int joinId;
    public int pawnId;

    public Pawn pawn;
    public InputDevice device;
    
    public GameObject menuPawnObject;
    
    public int victories;
    public bool isAlive = false;

    public User(int joinId, InputDevice device)
    {
        Debug.Log($"<color=blue>User {joinId}</color> has been created with <color=red>Device {device.name}</color>.");
        
        this.joinId = joinId;
        this.device = device;
    }

    public GameObject InstantiateCharacterPawn()
    {
        Debug.Log($"<color=blue>User {joinId}</color> has been instantiated as <color=green>character {pawn.Id}</color>");

        return Instantiate(pawn.characterPawnPrefab);
    }

    public GameObject InstantiateMenuPawn(GameObject prefab)
    {
        Debug.Log($"Menu pawn for <color=blue>User {joinId}</color> has been instantiated.");

        menuPawnObject = PlayerInput.Instantiate(
            prefab,
            pairWithDevice: device
        ).gameObject;
        
        menuPawnObject.GetComponent<MenuPawn>().user = this;
        return menuPawnObject;
    }

    public GameObject InstantiateKnockBar(Transform parent)
    {
        Debug.Log($"<color=red>Knock bar</color> for <color=blue>Character {pawn.Id}</color> has been instantiated for <color=green>User {joinId}</color>.");
        
        GameObject knockBarObject = Object.Instantiate(pawn.knockOutBarPrefab, parent);
        
        GameplayManager manager = Object.FindObjectOfType<GameplayManager>();

        if (manager is not SandboxManager _)
        {
            KnockOutBar knockOutBar = knockBarObject.GetComponent<KnockOutBar>();
            
            for (int i = 0; i < victories; i++)
            {
                Object.Instantiate(knockOutBar.victoryPrefab, knockBarObject.transform);
            }
        }
        
        return knockBarObject;
    }

    private GameObject Instantiate(GameObject prefab)
    {
        if (prefab is null)
            throw new ArgumentNullException($"The prefab variable is null for <color=blue>User {joinId}</color>.");
        
        Debug.Log($"Instantiated <color=blue>User {joinId}</color> with <color=red>Device {device.name}</color>.");
        
        return PlayerInput.Instantiate(
            prefab,
            pairWithDevice: device
        ).gameObject;
    }

    public void SetPawn(Pawn characterPawn)
    {
        pawn = characterPawn;
    }

    public void SetMenuPawn(GameObject pawnObject)
    {
        menuPawnObject = pawnObject;
    }
}
```