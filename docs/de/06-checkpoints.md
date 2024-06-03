# Checkpoints und Level Manager

In diesem Kapitel werden wir *Checkpoints* (Kontrollpunkte) in unser Spiel einbauen, sodass das Spiel an bestimmten Stellen neu startet, wenn der *Player* runtergefallen ist. Du solltest hier mit einem funktionsfähigen Stand aus dem [letzten Kapitel](/docs/05-cleanup.md) starten. Im Notfall kannst du das Musterprojekt [hier](https://github.com/FrankFlamme/UnityKidsWorkshop/releases/tag/0.5) nutzen.

## Checkpoint in die Szene einfügen

Im letzten Kapitel haben wir eine *KillZone* unter den Plattformen erstellt und den *Player* deaktiviert, sobald er auf diese gefallen ist. Wir wollen das Spiel nun so umbauen, dass er an einer bestimmten Stelle im Spiel wieder erscheint. Dafür bauen wir uns einen Kontrollpunkt in Form einer Flagge. 

Öffne den Ordner Textures in dem Project Fenster und klappe dort die Items aus. Ziehe eine der geschlossenen Fahnen (zum Beispiel Items_0) in die Szene und nenne das neue Objekt "CheckpointFlag". Gehe nun in den Ordner Scripts und erstelle ein neues Script (Rechtsklick > Create > C# Script), das du "Checkpoint" nennst. Hänge das neue Script nun an das Objekt *CheckpointFlag* an. Wähle dafür *CheckpointFlag* in der Hierarchy aus, klicke im Inspector auf "Add Component" und suche nach dem *Checkpoint* Script oder ziehe das Script direkt auf das Objekt in der Hierarchy.

### Animation

Als nächstes animieren wir die Fahne so wie wir im ersten Kapitel die Bewegung des *Player* animiert haben. Die Fahne soll solange geschlossen bleiben, bis der *Checkpoint* erreicht wurde. Danach soll sie im Wind wehen.

Wähle in der Hierarchy *CheckpointFlag* aus und klicke im Animation Fenster auf "Create". Falls du das Animation Fenster noch nicht geöffnet hast, findest du es im Menü unter Window > Animation > Animation. Nenne deine neue Animation "FlagClosed" und achte darauf, dass du sie im Ordner Animations speicherst. Füge in der Timeline ein Mal die geschlossene Fahne (zum Beispiel Items_0) bei 0 Sekunden ein.

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/124025345-9f203a80-d9f0-11eb-8517-bccab368e50f.png" width="800">
</p>

Erstelle nun über "Create New Clip..." im Dropdown-Menü des Animation Fensters eine weitere Animation für die wehende Fahne. Nenne die neue Animation "FlagOpen" und achte auch hier darauf, dass du sie in den Animations Ordner ablegst. Füge hier in die Timeline fünf mal in einem Abstand von 10 Sekunden die Sprites der offenen Fahnen abwechselnd ein. Nutzt du die gelbe Fahne wäre die Reihenfolge also zum Beipsiel: Items_17, Items_20, Items_17, Items_20, Items_17. Mit dem Play-Button kannst du dir in der Szene ansehen, wie die Animation aussieht.

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/124026406-f07cf980-d9f1-11eb-9f05-23b7ec9074b6.png" width="800">
</p>

Öffne im Unity Editor nun das Animator Fenster. Setze FlagClosed als Default State indem du darauf rechtsklickst und "Set as Layer Default State" wählst. Erstelle mit Rechtsklick und "Make Transition" sowohl eine Transition (Übergang) von FlagClosed nach FlagOpen, als auch in die andere Richtung.

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/124027177-dc85c780-d9f2-11eb-8017-1e224edbbb5e.png" width="400">
</p>

Erstelle nun im Animator Fenster über den Plus-Button einen neuen Parameter des Typen Bool und nenne ihn "FlagOpen". Bearbeite nun die Transitionen indem du auf die Pfeile zwischen den States klickst. Öffne bei beiden Transitionen im Inspector die Settings, entferne die Haken bei "Has Exit Time" und "Fixed Duration" und setze "Transition Duration" auf 0. Füge bei beiden Transitionen eine Condition hinzu. Bei der Transition von "FlagClosed" zu "FlagOpen" ist die Condition FlagOpen = true und bei der Transition "FlagOpen" zu "FlagClosed" ist die Condition FlagOpen = false. 

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/124029025-fcb68600-d9f4-11eb-895e-699df42f572f.png" width="250"> <img src="https://user-images.githubusercontent.com/75975986/124029028-fde7b300-d9f4-11eb-964d-f05b59af716a.png" width="250">
</p>

Als nächstes müssen wir einen Collider mit Trigger zum Objekt hinzufügen. In diesem Fall verwenden wir keinen BoxCollider2D, da dieser nicht zum Objekt passt. Stattdessen verwenden wir einen CircleCollider2D. Wähle das Objekt *CheckpointFlag* in der Hierarchy aus und füge im Inspector eine Komponente über "Add Component" hinzu. Wähle den CircleCollider2D aus und setze einen Haken bei "Is Trigger". Positioniere den Collider zentral um die Flagge indem du auf "Edit Collider" klickst und den Collider in der Szene anpasst.

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/124030257-4fdd0880-d9f6-11eb-90d4-35db59ba53d0.png" width="400">
</p>

Nun ergänzen wir unser Script. Öffne dafür das *Checkpoint* Script und erstelle darin fünf Variablen. Wir benötigen zwei Variablen vom Typ Sprite, um die beiden Bilder für die geöffnete und geschlossene Fahne festzulegen. Dann brauchen wir noch eine Variable vom Typ bool, um eine Verknüpfung mit dem Parameter unserer Animation herzustellen. Die anderen beiden Variablen sind aus unseren letzten Scripten bekannt. 

```csharp
public Sprite flagOpen;
public Sprite flagClosed;
public bool checkpointActive = false;

private SpriteRenderer _spriteRenderer;
private Animator _animator;
```

In der `Start()`-Methode initialisieren wir den SpriteRenderer für die Bilder der Flagge und den Animator.

```csharp
_spriteRenderer = GetComponent<SpriteRenderer>();
_animator = GetComponent<Animator>();
```

In der `Update()`-Methode setzen wir den Parameter „FlagOpen“ aus dem Animator auf den Wert des „checkpointActive“ Bools.

```csharp
_animator.SetBool("FlagOpen", checkpointActive);
```

Außerdem erstellen wir eine neue Methode `OnTriggerEnter2D` für das Öffnen der Fahne, wenn der *Player* auf die Fahne trifft.

```csharp
void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Player"))
        {
            _spriteRenderer.sprite = flagOpen;
            checkpointActive = true;
        }
    }
```

Damit das Ganze funktioniert, muss der *Player* noch den richtigen Tag zugewiesen bekommen. Wechsle dafür zurück in den Unity Editor und wähle den *Player* in der Hierarchy aus. Weise ihm im Inspector den Tag "Player" zu (nicht zu verwechseln mit dem Layer). Wähle nun in der Hierarchy *CheckpointFlag* aus und erstelle im Inspector unter Tag > Add Tag... und nenne den neuen Tag "Checkpoint". Weise *CheckpointFlag* den Tag "Checkpoint" zu. 

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/124184498-f0483100-dab9-11eb-9efd-ed5da91d2592.png" width="200">
</p>

Wenn du das Spiel nun startest, fängt die Fahne an zu wehen, sobald der *Player* hindurch läuft.

### Respawning

Im letzen Kapitel haben wir den *Player* einfach deaktiviert, wenn er heruntergefallen ist. Nun wollen wir ihn nach dem Herunterfallen am *Checkpoint* wiederherstellen, wenn er diesen bereits aktiviert hat.

Dafür bearbeiten wir das *Player* Script. Öffne das Script und erstelle darin eine neue Variable, in der die Position der Fahne gespeichert wird.

```csharp
public Vector3 respawnPos;
```

In der `OnTriggerEnter2D()`-Methode fügen wir am Ende der Methode folgenden Code ein, sodass die Position an welcher der *Player* in die Fahne läuft in derr neuen Variable `respawnPos` gespeichert wird. 

```csharp
if (collision.CompareTag("Checkpoint"))
   {
       respawnPos = collision.transform.position;
   }
```

Die erste if-Condition in der `OnTriggerEnter2D()`-Methode passen wir auch an, sodass der *Player* nicht mehr deaktiviert wird, sondern seine Position auf den Wert unserer Variable `respawnPos` gesetzt wird. Die if-Condition sollte nun folgendermaßen aussehen: 

```csharp
if (collision.CompareTag("KillZone"))
   {
       transform.position = respawnPos;
   }
```

Erweitere die `Start()`-Methode um folgenden Code, sodass der *Player* an der Ausgangsposition wieder erscheint, wenn er den Checkpoint noch nicht aktiviert hat.

```csharp
respawnPos = transform.position;
```

Nun wird der *Player* nach dem Herunterfallen am Anfang des Spieles oder am *Checkpoint* wiederhergestellt, falls er diesen aktiviert hat. Du kannst aus dem *Checkpoint* einen Prefab machen, sodass du das Element mehrmals wiederverwenden kannst. Ziehe dafür einfach die *CheckpointFlag* aus der Hierarchy in den Ordner Prefabs.

## Level Manager

Damit wir uns später schnell und einfach eigene Level bauen können, implementieren wir nun einen *LevelManager*.
Erstelle in der Hierarchy ein neues leeres Element mit einem Rechtsklick > "Create Empty". Erstelle außerdem ein neues Script in dem Ordner Scripts und nenne beides "LevelManager". Hänge das neue Script an das neue Objekt indem du den *LevelManager* in der Hierarchy auswählst, im Inspector "Add Component" klickst und das *LevelManager* Script suchst.

Zuvor haben wir eine Respawn-Funktion eingebaut, sodass der *Player* wieder neu in der Szene erscheint, nachdem er runtergefallen ist. Diese Funktion holen wir nun in den *LevelManager*. Öffne das *LevelManager* Script und erstelle zwei Variablen.

```csharp
    public float timeToRespawn;
    public Player player;
```

Initialisiere den *Player* in der `Start()`-Methode. Diesmal nutzen wir hierfür nicht `GetComponent`, sondern `FindObjectOfType<Player>`, da wir mit dem *LevelManager* nicht innerhalb eines Objekts bleiben wollen.

```csharp
    player = FindObjectOfType<Player>();
```

Baue außerdem eine `Respawn()`-Methode, die dann aus dem *LevelManager* heraus aufgerufen werden kann. 

```csharp
    public void Respawn()
    {
        player.gameObject.SetActive(false);
        player.transform.position = player.respawnPos;
        player.gameObject.SetActive(true);
    }
```

Gehe nun wieder in das *Player* Script  und erstelle eine Variable für den *LevelManager*.

```csharp
    public LevelManager levelManager;
```

Füge in der `Start()`-Methode eine Zeile hinzu, mit der du den *LevelManager* initialisierst.

```csharp
    levelManager = FindObjectOfType<LevelManager>();
```

Schreibe die `OnTriggerEnter2D()`-Methode so um, dass die Position zur Wiederherstellung nicht mehr manuell gesetzt wird, sondern über den *LevelManager*. Folgendermaßen sollte die Methode nun aussehen:

```csharp
    void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("KillZone"))
        {
            levelManager.Respawn();
        }

        if (collision.CompareTag("Checkpoint"))
        {
            respawnPos = collision.transform.position;
        }
    }
```

Das Respawning sollte nun wie zuvor funktionieren. Allerdings geht die Wiederherstellung des *Player* aktuell ziemlich schnell, wenn er irgendwo herunter fällt. Die Kamera reagiert dann sehr "hektisch", daher wollen wir das ganze ein bisschen verzögern. Dazu verwenden wir eine "CoRoutine".

Die CoRoutine implementieren wir im *LevelManager* Script. Füge dort eine "CoRoutine" ein und nenne sie `RespawnCo()`. Verschiebe anschließend den Inhalt aus der Funktion `Respawn()` in die "CoRoutine". Erweitere den Code nach der Deaktivierung des *Player* um eine weitere Zeile, in der die Verzögerung des Respawning implementiert wird. So sieht die "CoRoutine" dann aus: 

```csharp
    public IEnumerator RespawnCo()
    {
        player.gameObject.SetActive(false);
        yield return new WaitForSeconds(timeToRespawn);
        player.transform.position = player.respawnPos;
        player.gameObject.SetActive(true);
    }
```

Rufe die "CoRoutine" in der Methode `Respawn()`auf. 

```csharp
    public void Respawn()
    {
        //...
        StartCoroutine("RespawnCo");
    }
```

Gehe zurück in den Unity Editor, wähle den *LevelManager* in der Hierarchy aus und setze im Inspector den Wert von "Time To Respawn" auf 2. Die Kamerabewegung funktioniert nun etwas flüssiger beim Wiederherstellen des *Player*. 

## Spikes

Nun erstellen wir noch *Spikes*, die als Hindernis für den *Player* dienen. Öffne im Unity Editor den Ordner "Textures" und öffne Tiles. Ziehe das Sprite für die *Spikes* (Tiles_69) in die Szene und benenne das Objekt in der Hierarchy zu "Spikes" um. Erstelle nun im Ordner Scripts ein neues C#-Script und nenne es "PlayerHurt". Wähle das Objekt *Spikes* in der Hierarchy aus und füge ihm das neue Script im Inspector über "Add component" hinzu.

Öffne das neue Script und erstelle eine neue Variable für den *LevelManager* über folgenden Code:

```csharp
    private LevelManager _levelManager;
```

Initialisiere den *LevelManager* in der `Start()`-Methode: 

```csharp
    _levelManager = FindObjectOfType<LevelManager>();
```

Wenn der *Player* auf die *Spikes* springt oder hinein läuft, soll er sterben bzw. später ein Leben verlieren. Dafür implementieren wir die Methode `OnTriggerEnter2D()`, welche aktiv wird, wenn ein als Trigger gekennzeichneter BoxCollider2D berührt wird. In dieser Methode soll das Respawning aus dem *LevelManager* aufgerufen werden, wenn der *Player* den BoxCollider2D berührt. 

```csharp
    void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Player"))
        {
            _levelManager.Respawn();
        }
    }
```

Nun müssen wir noch den BoxCollider2D um die *Spikes* setzen. Wechsel in den Unity Editor und wähle das Objekt *Spikes* in der Hierarchy aus. Füge im Inspector über "Add Component" die neue Komponente "Box Collider 2D" hinzu und setze einen Haken bei "Is Trigger". Bearbeite nun die Größe des Colliders indem du auf den Button neben "Edit Collider" klickst. Setze den Collider etwas nach innen bei den Spikes, sodass der *Player* nicht direkt stirbt, wenn er in der Nähe des Objektes ist. 

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/126035040-5cfa788b-c80d-4e5e-9b4f-db38b872db48.png" width="400">
</p>

Der *Player* stirbt nun, wenn er in die Spikes reinläuft - das Kapitel ist hiermit abgeschlossen. 

[Hier](https://github.com/FrankFlamme/UnityKidsWorkshop/releases/tag/0.6) findest du die Musterlösung zum Herunterladen und [hier](/docs/07-level_elements.md) geht es weiter zum nächsten Kapitel.
