# Snake in Unity

A **Snake** game created according to [this tutorial](https://www.youtube.com/watch?v=U8gUnpeaMbQ).
See the example [GitHub repository here](https://github.com/zigurous/unity-snake-tutorial/wiki).

## Snake Kochrezept (Video Transcript)

### Projekt anlegen und initialisieren
- Neues Projekt "Snake" als *2D Core* anlegen (dauert)
- GitHub Setup
  - Währenddessen in GitHub ein Repository Unity-Snake anlegen (mit .gitignore für Unity)
  - Das Repository Clonen ("auschecken") im Entwicklungsverzeichnis
  - Dann die Inhalte aus dem Verzeichnis Snake nach Unity-Snake verschieben
  - In TortoiseGit "Add" ausführen und die vorgeschlagenen Dateien hinzufügen -> Commit -> Push
  - Nun den Unity Editor wieder öffnen und das Projet Unity-Snake hinzufügen und öffnen
- Sample Scene zu `Snake` umbennen
- Hintergrundfarbe ändern:
  - Links in der Hierarchy *MainCamera* auswählen
  - Background Farbe ändern auf *schwarz*  
    (im Game Tab kann man sich die Vorschau anschauen)
- Sichtbare Fläche der Kamera ändern
  - Camera/Size = 15

Die benötigten GameObjects für das Spiel Snake werden sein:  
Eine Kamera, die Snake, Food, Wände

### Snake erstellen
- GameObject mit SpriteRenderer
  - In der Hierarchy: Rechtsklick -> 2D Object -> Sprite  
    das erstellt ein neues GameObject mit einem SpriteRenderer
  - Umbenennen zu Snake  
  - Position zurücksetzen (mit den drei Punkten rechts bei *Transform*)
  - Im Assets Verzeichnis: Neuen Ordner Sprites erstellen
  - Im Ordner Sprites: Rechtsklick -> Create -> Sprite -> Square
  - Snake Objekt in der Hierarchy auswählen
  - Square von Assets rechts in den im Inspector -> SpriteRenderer/Sprite ziehen
  - Color im SpriteRenderer ändern zu grün (#00FF00)
- Add Component: *Rigidbody2D* hinzufügen  
  (eigentlich wird das Objekt dadurch zu einen trägen physikalischen Körper, aber Rigidbody ist auch notwendig um Kollisionen zu erkennen)
  - BodyType = Kinematic ... dadurch wird die Physik ausgeschaltet
- Add Component: *BoxCollider2D* ... das legt die Form des Colliders fest
  - Trigger = True setzen  
    dadurch wird keine Physik-Kollision verarbeitet, aber wir können im Skript darauf reagieren  

  Der Collider hat jetzt die volle Größe des Quadrats. Um zu vermeiden, dass eine Kollision erkannt wird, wenn nur angrenzende Objekte berührt werden, soll die Größe des Quadrats etwas reduziert werden.
  - daher im BoxCollider2D: Size = 0.75 für X und Y setzen

### Food erstellen
- Wie zuvor GameObject mit SpriteRenderer erstellen (siehe oben),
  - umbenennen zu *Food*
  - jetzt existiert das Square bereits,
  - und wir setzen Color im SpriteRenderer = rot (#FF0000)
- Das Food GameObjekt braucht keinen Rigidbody (Nur die Snake verarbeitet Kollisionen)
- Add Component: *BoxCollider2D*
  - Is Trigger = True setzen
  - im BoxCollider2D: Size = 0.75 für X und Y setzen

### Wände hinzufügen
- Erstes Wandobjekt erstellen
  - 2D Object -> Sprite erstellen
  - zu *Wall* umbenennen
  - *Square* von Assets wieder in den SpriteRenderer/Sprite ziehen
  - Farbe auf dunkelgrau setzen
  - Position: X=-24 (nur ganze Zahlen verwenden!)
  - Size: Y = 25
- Wand duplizieren (rechte Maustaste -> Duplicate)
  - Position: X=24
- Wand duplizieren
  - Size: X=48, Y=1
  - Position: Y=12, X=0
- Wand duplizieren
  - Position: Y=-12
- Alle Wände gemeinsam markieren und zu *Wall* umbenennen

### Bewegung der Schlange
- Im Assets Verzeichnis neuen Ordner *Scripts* anlegen
- C# Script `Snake` erstellen
- Dem GameObject *Snake* das Skript `Snake` zuweisen (hineinziehen im Inspctor)
- Bearbeitung starten
- Variable `direction` erstellen
```c#
private Vector2 _direction = Vector2.right
```
- `Update()` Funktion erstellen ... zur Verarbeitung von Input  
  (wir wollen auf die Pfeiltasten und auf die Buchstaben ASDW reagieren)
```c#
private void Update() {
  if (Input.GetKeyDown(KeyCode.W) || Input.GetKeyDown(KeyCode.UpArrow))
  {
    _direction = Vector2.up;
  } else if (Input.GetKeyDown(KeyCode.S) || Input.GetKeyDown(KeyCode.DownArrow))
  {
    _direction = Vector2.down;
  } else if (Input.GetKeyDown(KeyCode.A) || Input.GetKeyDown(KeyCode.LeftArrow))
  {
    _direction = Vector2.left;
  } else if (Input.GetKeyDown(KeyCode.D) || Input.GetKeyDown(KeyCode.RightArrow))
  {
    _direction = Vector2.right;
  }
}
```

- `FixedUpdate()` Funktion erstellen ... zum Bewegen des Objekts  
  (Im Gegensatz zu *Update* (wird mit variabler Framerate aufgerufen) wird diese Funktion in fixen Zeitabständen aufgerufen, und alles was mit Physik zu tun hat soll man in *FixedUpdate machen*)
```c#
private void FixedUpdate() {
  this.transform.position = new Vector3(
    Math.Round(this.transform.position.x) + _direction.x, // runden auf ganze Koordinaten
    Math.Round(this.transform.position.y) + _direction.y, // just in case... 
    0.0f
  );
}
```

Wenn man das Spiel jetzt startet, kann man mit dem Kopf der Schlange herumfahren.

Allerdings bewegt sie sich zu schnell.
Man kann deshalb die Geschwindigkeit generell setzen:
- Im Menü Edit -> ProjectSettings -> Time -> Fixed Timestep = 0.04 oder 0.05 setzen  
  (in diesem Zeitintervall werden Physik-Berechnungen und FixedUpdate Events ausgeführt)


**Bugfix**: Damit später die Schlange nicht in sich selbst reinfährt, wenn man versehentlich umdreht, kann man `Update()` so erweitern:
```c#
private void Update() {
  if (Input.GetKeyDown(KeyCode.W) || Input.GetKeyDown(KeyCode.UpArrow))
  {
    if (!_direction.Equals(Vector2.down))
    {
      _direction = Vector2.up;
    }
  } else if (Input.GetKeyDown(KeyCode.S) || Input.GetKeyDown(KeyCode.DownArrow))
  {
    if (!_direction.Equals(Vector2.up))
    {
      _direction = Vector2.down;
    }
  } else if (Input.GetKeyDown(KeyCode.A) || Input.GetKeyDown(KeyCode.LeftArrow))
  {
    if (!_direction.Equals(Vector2.right))
    {
      _direction = Vector2.left;
    }
  } else if (Input.GetKeyDown(KeyCode.D) || Input.GetKeyDown(KeyCode.RightArrow))
  {
    if (!_direction.Equals(Vector2.left))
    {
      _direction = Vector2.right;
    }
  }
}
```

### Food Objekt Scripting
- Neues Script `Food` erstellen
- Script dem *Food* GameObject zuweisen
- Im Food Skript wollen wir das Food an einer zufälligen Position platzieren
- Zur Erkennung der erlaubten Position machen wir folgendes:
  - neues Empty GameObject erstellen
  - *BoxCollieder2D* hinzufügen und *IsTrigger = True* setzen
  - Umbenennen zu *GridArea*
  - Size ausrichten an den bestehenden Wänden, aber jeweils eine halbe Einheit innerhalb bleiben  
    (weil das Food mit Size=1 jeweils mittig ausgerichtet ist, geht es sich genau aus ...)  
    X = 46, Y = 22
- Bearbeitung `Food` Script starten
- Variable definieren  `public BoxCollider2D gridArea;`
- Im Editor die *GridArea* aus der Hierarchy zur Variablen Grid Area im Inspector / Food (Script) ziehen
- Funktion zur zufälligen Positionierung erstellen
  ```c#
  private void RandomizePosition()
  {
    Bounds bounds = this.gridArea.bounds;  // die erlaubte Fläche aus der GridArea auslesen
    float x = Random.Range(bounds.min.x, bounds.max.x); // Zufällige Position generieren
    float y = Random.Range(bounds.min.y, bounds.max.y);
    // Food Objekt positionieren
    this.transform.position = new Vector3(Mathf.Round(x), Mathf.Round(y), 0.0f); 
  }
  ```
- Funktion beim `Start()` aufrufen
  ```c#
  private void Start()
  {
    RandomizePosition();
  }
  ```
- Wenn die Schlange das *Food* isst (Kollision), dann neue zufällige Positionierung
  - Ob das Food von der Schlange (Player) gefressen wurde, überprüfen wir mit einem Tag  
    -> Snake GameObject auswählen und rechts im Inspector *Tag = Player* zuweisen
  ```c#
  private void OnTriggerEnter2D(Collider2D other)
  {
    if (other.tag.Equals("Player"))
    {
      RandomizePosition();
    }        
  }
  ```

### Schlange wachsen lassen
Dafür wird der Schlange ein Segment hinzugefügt, wenn sie frisst.

- `Snake` Script erweitern
- `using System.Collections.Generic;` hinzufügen, damit man `List<>` verwenden kann
- Variable erstellen: `private List<Transform> _segments;`
- Beim `Start()` die Liste initialisieren: 
  ```c#
  private void Start()
  {
    _segments = new List<Transform>();
    _segments.Add(this.transform);
  }
  ```
- `SnakeSegment` als Prefab erstellen
  - *Snake* GameObject in der Hierarchy duplizieren
  - umbenennen zu *SnakeSegment*
  - Rigidbody2D Komponente löschen
  - Script löschen
  - Ordner Prefabs in Assets erstellen
  - SnakeSegment GameObject von Hierarchie in Ordner Prefabs ziehen
  - SnakeSegment aus der Scene löschen
- Referenz zum `SnakeSegment` Prefab im Skript erstellen
  ```c#
  public Transform segmentPrefab;
  ```
  - im Editor steht jetzt im Inspector die Variable Segment Prefab zur Verfügung
  - Prefab vom Assets Ordner der Snake im Inspector zuweisen (in Snake(Script) Variable Segment Prefab reinziehen)
- Methode `Grow()` erstellen
  ```c#
  private void Grow()
  {
    Transform segment = Instantiate(this.segmentPrefab);
    // Neues Segment wird hinten angehängt, also an der Position des letzten Segments
    segment.position = _segments[_segments.Count - 1].position; 

    _segments.Add(segment);
  }
  ```
- Nun soll die Schlange wachsen, wenn sie eine Kollision mit einem Food hatte.  
  Daher Funktion `OnTriggerEnter2D(...)` von Food kopieren und ändern.
  ```c#
  private void OnTriggerEnter2D(Collider2D other)
  {
    if (other.tag.Equals("Food"))
    {
      Grow();
    }
  }
  ```
  - Allerdings muss das Tag *Food* zuvor erst erstellt werden  
    - Im Food Objekt im Inspector, oben bei *Tag* -> *Add Tag* wählen
    - Tag *Food* hinzufügen
    - Nun noch beim Food Objekt das Tag *Food* auswählen
  
  Wenn man jetzt testet, dann sieht man neue SnakeSegments entstehen, aber sie bewegen sich noch nicht mit.

  - `FixedUpdate` erweitern, damit sich die SnakeSegments mit bewegen  
    diesen *Code am Beginn einfügen*:
  ```c#
  for (int i = _segments.Count - 1; i > 0; i--)
  {
    _segments[i].position = _segments[i - 1].position;
  }
  ```
  - Zur besseren Unterscheidbarkeit der *SnakeSegments* die Größe leicht reduzieren
    - SnakeSegment Prefab in Assets auswählen
    - Scale: X = 0.9, Y = 0.9 setzen

### Kollisionserkennung
- Zusätzliches Tag *Obstacle* definieren wie oben
  - allen 4 Wänden das Tag *Obstacle* zuweisen
  - Dem SnakeSegment Prefab das Tag *Obstacle* zuweisen
- Skript `OnTriggerEnter2D` erweitern
  ```c#
  // an bestehendes if statement anhängen:
  else if (other.tag.Equals("Obstacle"))
  {
    ResetState();
  }
  ```

- Neue Methode *ResetState* erstellen
  ```c#
  private void ResetState()
  {
    _direction = Vector2.right;
    transform.position = Vector3.zero;

    for (int i = 1; i < _segments.Count; i++) // Beginn mit i=1, weil Index=1 ist der Kopf
    {
      Destroy(_segments[i].gameObject);
    }
    _segments.Clear(); // Liste leeren
    _segments.Add(this.transform); // Kopf wieder zurück in die Liste einfügen
  }
  ```

- Die Schlange soll beim Start bereits mit einer festgelegten Länge starten
  - Variable `initialSize` definieren
  ```c#
  public int initialSize = 4;
  ```
  - Funktion `ResetState` erweitern:
  ```c#
  // Beim Start die vorgegebene Anzahl Segmente generieren
  for (int i = 1; i<initialSize; i++)
  {
    _segments.Add(Instantiate(this.segmentPrefab));
  }
  ```

- Methode `Start()` anpassen und ebenfalls `ResetState()` aufrufen
  ```c#
  private void Start()
  {
    ResetState();
  }
  ```
- Damit das funktioniert, muss beim Start die Liste für die snakeSegments bereits vorhanden sein, daher Variablendefinition anpassen:
  ```c#
  private List<Transform> _segments = new List<Transform>();
  ```
- nun im Editor die Initial Size = 8 anpassen
