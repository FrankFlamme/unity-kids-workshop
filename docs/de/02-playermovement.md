# Bewegung des Spielers
In diesem Kapitel werden wir die Bewegung des *Player* in der Szene programmieren. Du solltest das [Kapitel 1](/docs/01-start.md) bereits abgeschlossen haben.

## Laufen
Öffne zuerst das Script, das für den *Player* angelegt wurde - Player.cs. Hier müssen zwei Variablen erstellt werden: `speed` ist die Geschwindigkeit, mit der unser *Player* sich durch die Szene bewegen wird und `_rb` ist der "Rigidbody2D", mit dessen Hilfe der *Player* auf physikalische Einwirkungen reagieren kann. 

```csharp
public float speed;    
private Rigidbody2D _rb;
```

In der `Start()`-Methode wird der "Rigidbody2D" initialisiert. Wir weisen `_rb` also die Komponente "Rigidbody2D" zu, die im Vorfeld an den *Player* gehängt wurde.

```csharp
_rb = GetComponent<Rigidbody2D>();
```

Die Bewegung implementieren wir in der `Update()`-Methode. Diese Methode wird mit jeder Aktualisierung des Bildschirminhalts ein Mal aufgerufen. 
Für die Bewegung bauen wir eine bedingte Anweisung (if-Anweisung), in der wir die Bewegung des "Rigidbody2D", die durch die Unity bereitgestellt wird, ansprechen.

```csharp
if (Input.GetAxisRaw("Horizontal") > 0f)
{
    _rb.velocity = new Vector3(speed, _rb.velocity.y, 0f);
}
else if (Input.GetAxisRaw("Horizontal") < 0f)
{
    _rb.velocity = new Vector3(-speed, _rb.velocity.y, 0f);
}
else
{
    _rb.velocity = new Vector3(0f, _rb.velocity.y, 0f);
}
```

Wenn du nun im Unity Editor deinen *Player* auswählst, kannst du rechts im Inspector unter deinem *Player* Script die gewünschte Geschwindigkeit "speed" einstellen. Setze sie zum Beispiel auf 5. Wenn du das Spiel nun startest, kann sich der *Player* mit Hilfe der Keyboard-Pfeiltasten nach links und rechts bewegen. 

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/122826402-5fb46880-d2e3-11eb-83d0-96a0bf2aa350.png" width="300">
</p>

## Springen
Für das Springen brauchen wir eine dritte Variable im *Player* Script, die Sprungkraft jump.

```csharp
public float jump;
```

In der Update()-Methode fügen wir eine weitere if-Anweisung hinzu, in der wir die vertikale Bewegung des *Player* ansprechen, wenn die Leertaste gedrückt wird.

```csharp
if (Input.GetButtonDown("Jump"))
{
    _rb.velocity = new Vector3(_rb.velocity.x, jump, 0f);
}
```

Wenn du nun im Unity Editor die Sprungkraft einstellst, kann der *Player* auch springen. Du kannst die Sprungkraft genau wie die Laufgeschwindigkeit einstellen, indem du deinen *Player* auswählst und im Inspector rechts unter deinem *Player* Script die Variable "jump" zum Beispiel auf 15 setzt. 

Der *Player* kann jetzt allerdings beliebig oft in der Luft springen. Wir wollen als nächstes dafür sorgen, dass er nur abspringen kann, wenn er sich auf dem Boden befindet. Dafür erstellen wir zuerst zwei neue Layer im Unity Editor. Wähle dazu das Dropdown "Layers" über dem Inspector aus und klicke auf "Edit Layers...". Erstelle sowohl in der Liste "Sorting Layers" als auch in der Liste "Layers" jeweils einen Layer für den *Ground* und einen für den *Player*. Achte dabei darauf, dass der *Player* unter dem *Ground* Layer einsortiert ist.

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/122826414-6511b300-d2e3-11eb-9ddc-e68373c2c766.png" width="300"> <img src="https://user-images.githubusercontent.com/75975986/122827711-00efee80-d2e5-11eb-8e23-ec234a2111e9.png" width="300">
</p>

Ordne dem *Player* den Player-Layer zu, indem du den *Player* in der Hierarchy auswählst und den "Layer" im Inspector, wie auf dem nächsten Screenshot abgebildet, auswählst. Tue das selbe für die *Grounds*. Für die *Grounds* haben wir Prefabs erstellt, denen der Ground-Layer zugeordnet werden muss. Wähle dafür im Unity Editor im Project-Fenster den Ordner Prefabs aus. Dort findest du zwei Grounds. Markiere den ersten Ground und setze den Layer auf "Ground". Möglicherweise wirst du gefragt, ob die Änderungen für alle child-Objekte übernommen werden sollen. Wähle dann "Ja" aus.

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/122829276-182fdb80-d2e7-11eb-8b96-a3fddf0d8c69.png" width="300">
</p>

Öffne nun wieder das *Player* Script. Hier legen wir nun drei Variablen an. `checkGround` ist eine leere Form, die sich an den Füßen des *Player* befinden wird. Mit `checckGroundRadius` prüfen wir durch ein kleines Kreisobjekt, ob der *Player* den *Ground* berührt. `isGround` ist dafür da, um den *Ground* zu identifizieren.

```csharp
public Transform checkGround;
public float checkGroundRadius;
public LayerMask isGround; 
```

Gehe zurück in den Unity Editor und erstelle ein leeres Objekt unter dem *Player*, indem du in der Hierarchy auf den *Player* rechtsklickst und "Create Empty" auswählst. Nenne das leere Element "GroundCheck" und bewege es mit dem Move-Tool zu den Füßen des *Player*.

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/122996587-5393de00-d3ab-11eb-94ea-cbfb2ce93db1.png" height="300"><img src="https://user-images.githubusercontent.com/75975986/122997372-3d3a5200-d3ac-11eb-876d-619e81f5058a.png" height="300">
</p>

Nun befüllen wir unsere neuen Variablen. Wähle in der Hierarchy den *Player* aus. Setze im Inspector beim *Player* Script die Variable "Is Ground" auf *Ground*, "Check Ground Radius" auf 0.25 und ziehe das neue Element *GroundCheck* in die Variable "Check Ground". So sollten die Variablen nun bei dir aussehen:

<p align="center">
<img src="https://user-images.githubusercontent.com/75975986/123169435-991edc80-d479-11eb-8e05-6098e3f7c90f.png" width="300">
</p>

Nun noch einmal zurück in das *Player* Script. Setze eine neue Variable `grounded`. 

```csharp
public bool grounded;
```

Erweitere die `Update()`-Methode zum Setzen der Variable. Hier prüfen wir, ob unser zuvor angelegtes Objekt (an den Füßen des *Player*) ein Objekt berührt, das den Layer Ground gesetzt hat. Füge folgenden Code am Anfang der `Update()`-Methode ein:

```
grounded = Physics2D.OverlapCircle(checkGround.position, checkGroundRadius, isGround);
```

Ändere außerdem die if-Condition für den Sprung ("jump"), sodass beim Abspringen auch geprüft wird, ob `grounded` zutrifft. Der Code sollte dann so aussehen: 

```csharp
if (Input.GetButtonDown("Jump") && grounded)
{
    _rb.velocity = new Vector3(_rb.velocity.x, jump, 0f);
}
```

Der *Player* kann nun laufen und springen. Du findest die Musterlösung [hier](https://github.com/FrankFlamme/UnityKidsWorkshop/releases/tag/0.2) zum Herunterladen. Ansonsten kannst du mit dem [nächsten Kapitel](/docs/03-animations.md) weitermachen. :)
