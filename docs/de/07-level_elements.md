# Zusätzliche Level Elemente

In diesem Kapitel erstellen wir weitere Level-Elemente in unserem Spiel. Du solltest hier mit einem funktionsfähigen Stand aus dem [letzten Kapitel](/docs/06-checkpoints.md) starten. Im Notfall kannst du das Musterprojekt [hier](https://github.com/FrankFlamme/UnityKidsWorkshop/releases/tag/0.6) nutzen.

## Bewegliche Plattform

Als erstes erstellen wir eine kleine Plattform, die sich horizontal von links nach rechts und umgekehrt bewegt.
Öffne den Unity Editor und erstelle in der Hierarchy ein neues Objekt mit dem Rechtsklick > Create Empty. Nenne das neue Element "MovingPlatform". Nimm dann das bestehende Prefab „SmallGround_1x5“ aus dem Ordner Prefabs und ziehe es als Objekt unter „MovingPlatform“. Erstelle unter dem neuen Objekt zwei weitere leere Objekte und nenne sie "Start" und "Stop". Setze die Position der neuen Objekte auf der X, Y und Z Achse im Inspector auf 0. 

<p align="center">
<img src="https://user-images.githubusercontent.com/13068729/126306166-c663dac2-143a-4f85-8d7f-8d9dcfd73ec7.png" width="400">
</p>

Nutze das Move-Tool, um das leere Objekt an eine geeignete Stelle in der Szene zu verschieben, am besten hinter die letzte Plattform. Damit wir den Start- und Endpunkt in der Szene später auch sehen können, markieren wir die Punkte farblich. Das geht, wenn du das entsprechende Objekt in der Hierarchy gewählt hast und in dem Inspector auf den Würfel oben klickst:

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/126035665-2da2917c-b20e-48f6-926b-b61fa326f03d.png" width="400">
</p>

Wähle für Start einen grünen und für Stop einen roten Punkt aus und ziehe den Stop-Punkt direkt ein wenig nach rechts, so dass wir eine Strecke zwischen beiden Punkten haben. In der Szene sieht das dann so aus:

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/126035848-dac96de6-1459-4dd0-96a7-a8353c92334f.png" width="800">
</p>

Erstelle nun im Ordner Scripts ein neues C# Script und nenne es "MovingObjects". Dieses Script können wir später auch für andere bewegliche Plattformen nutzen. Hänge das Script an das Objekt *MovingPlatform*, indem du es in der Hierarchy auswählst und das Script im Inspector unter "Add Component" auswählst. 

Öffne das Script und erstelle folgende Variablen: 

```csharp
    public GameObject movingObject;
    public Transform startPoint;
    public Transform stopPoint;
    public float speed;


    private Vector3 _curTarget;
```

Initialisiere in der `Start()`-Methode die Anfangsposition des Objektes: 

```csharp
    _curTarget = stopPoint.position;
```

Implementiere die Bewegung des Objektes in der `Update()`-Methode:

```csharp
    movingObject.transform.position = Vector3.MoveTowards(movingObject.transform.position, _curTarget, speed * Time.deltaTime);

    if (movingObject.transform.position == stopPoint.position)
    {
        _curTarget = startPoint.position;
    }

    if (movingObject.transform.position == startPoint.position)
    {
        _curTarget = stopPoint.position;
    }
```


Im Unity Editor weisen wir nun noch die Variablen zu. Wähle das Objekt *MovingPlatform* in der Hierarchy aus und ziehe die darunterliegende Plattform im Inspector in das *MovingObject*. Start wird der Start Point und Stop wird der Stop Point. Setze außerdem "Speed" auf 3.

<p align="center">
<img src="https://user-images.githubusercontent.com/13068729/126306019-e35514f4-1d94-43f3-9ef1-1cb17d76484b.png" width="400">
</p>

Wenn wir das Spiel nun starten, bewegt sich die neue Plattform, allerdings bleibt der *Player* nicht auf der beweglichen Plattform stehen, sondern rutscht herunter. Um das zu beheben müssen wir einige Anpassungen im *Player* Script vornehmen.

Erstelle als erstes einen neuen Tag für die bewegliche Plattform, indem du das Objekt *SmallGround_1x5* (Achtung: Nicht *MovingPlatform*) in der Hierarchy unterhalb von *MovingPlatform* auswählst und im Inspector unter Tag > Add Tag... wählst. Nenne den neuen Tag "MovingPlatform" und wähle ihn für das Objekt *SmallGround_1x5* aus.

Gehe nun in das *Player* Script und erstelle zwei neue Methoden. Wir verwenden hier die Methoden `OnCollisionEnter()` und `OnCollisionExit()`, weil die *MovingPlatform* keinen Trigger hat. Solange der *Player* auf der Plattform steht, übernimmt er die Position der Plattform. Verlässt er die Plattform, z.B. durch Springen, setzen wir die "Transform" (Position) wieder zurück. Folgendermaßen sieht der Code aus.

```csharp
    void OnCollisionEnter2D(Collision2D collision)
    {
        if (collision.gameObject.CompareTag("MovingPlatform"))
        {
            transform.parent = collision.transform;
        }
    }

    void OnCollisionExit2D(Collision2D collision)
    {
        if (collision.gameObject.CompareTag("MovingPlatform"))
        {
            transform.parent = null;
        }
    }
```

## Sorting Layers

Damit die Objekte in der Szene korrekt angeordnet werden, legen wir nun über das Menü im Inspector die "Sorting Layer" fest. Wähle über dem Inspector Layers > Edit Layers aus und klappt die "Sorting Layers" aus. Die Reihenfolge der Layers geht von oben nach unten. Was oben in der Liste steht, ist weiter im Hintergrund. Was am weitesten unten in der Liste steht, ist in der Szene im Vordergrund zu sehen.

Lege nun zwei neue Sorting Layer an. Den ersten nennst du *World Items*, den zweiten nennst du *Player*. Verschiebe *World Items* an die erste Stelle, *Default* verbleibt an der zweiten Stelle und *Player* kommt an die dritte Stelle.

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/126036858-010a4d79-8ecb-4743-a9b8-cf228cd3abf3.png" width="400">
</p>

Weise den Objekten und Prefabs (*Snail*, *Spikes*, ...) anschließend die richtigen Sorting Layer zu, indem du sie in der Hierarchy wählst und die entsprechenden Sorting Layer im Sprite Renderer auswählst. 

## Münzen einsammeln

Als nächstes wollen wir die bereits vorbereiteten *Coins* (Münzen) einsammeln und zusammenrechnen. Markiere dafür alle 5 *Coins* in der Hierarchy und setzen den Haken „Is Trigger“ im Inspector beim CircleCollider2D. Erstelle ein neues Script im Ordner Scripts und nenne es "Coin".

Öffne das neue Script und erstelle eine neue Variable für den *LevelManager*, damit das Einsammeln der Münzen zentral funktioniert. Erstelle eine zweite Variable für den Wert der Münze (falls mehrere Münzen mit unterschiedlichen Werten im Spiel vorhanden sind).

```csharp
    private LevelManager _levelManager;
    public int coinValue;
```

Instanziiere den Level Manager in der `Start()`-Methode.

```csharp
    _levelManager = FindObjectOfType<LevelManager>();
```

Füge außerdem eine `OnTriggerEnter2D()`-Methode hinzu, in der die Münzen deaktiviert werden, sobald der *Player* sie berührt.

```csharp
    void OnTriggerEnter2D(Collider2D collider)
    {
        if (collider.tag == "Player")
        {
            _levelManager.AddCoins(coinValue);
            gameObject.SetActive(false);
        }
    }
```

Die `AddCoins`-Funktion muss nun noch im *LevelManager* implementiert werden. Öffne dafür das *LevelManager* Script und lege als erstes eine Variable für die Anzahl der gesammelten Münzen an. 

```csharp
    public int coinCount;
```

Schreibe außerdem die neue Funktion `AddCoin()`, in welcher der Wert der gerade aufgesammelten Münze auf die schon gesammelten Münzen addiert wird. 

```csharp
    public void AddCoins(int addedCoins)
    {
        coinCount += addedCoins;
    }
```

Gehe nun wieder zurück in den Unity Editor, markiere alle *Coins* in der Hierarchy und füge ihnen das *Coin* Script im Editor über "Add Component" hinzu. Setze den "Coin Value" auf 1. 

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/126137542-da758d2b-417a-4b8d-a40f-74e91ee0cc83.png" width="400">
</p>

Wenn du das Spiel nun startest, verschwinden die Münzen nach dem Einsammeln und du kannst beobachten, wie sich der "Coin Count" im *LevelManager* erhöht. Kapitel 7 ist damit abgeschlossen.

[Hier](https://github.com/FrankFlamme/UnityKidsWorkshop/releases/tag/0.7) findest du die Musterlösung zum Herunterladen und [hier](/docs/08-ui_elements.md) geht es weiter zum nächsten Kapitel.
