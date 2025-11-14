## Procedural-Generation

La génération procédurale est une méthode qui crée automatiquement du contenu à l'aide d'algorithmes. Elle permet de générer dynamiquement des mondes, des niveaux, des textures et des objets. Au lieu de tout créer manuellement, des règles et des paramètres sont définis. L'ordinateur utilise ensuite ces règles pour générer un résultat unique. Cette technique est largement utilisée dans les jeux vidéo, tels que Minecraft et No Man's Sky. Elle permet de créer des environnements vastes et variés avec peu de ressources humaines. Les algorithmes peuvent intégrer une part d'aléatoire contrôlé pour diversifier le contenu. Malgré cela, il est possible de maintenir une certaine cohérence en définissant des contraintes. La génération procédurale améliore également la rejouabilité, car chaque partie peut être différente. Aujourd'hui, c'est un outil essentiel pour créer des expériences riches et infiniment renouvelables.

Il existe différent algorythme en voici l'explication de quelques uns.

## Cellular Automata

Le cellular reprend le principe du Jeu de la vie de John Horton Conway. En fonction des regles établis une cellule "survie" "nait" ou "meurt". 
- Une cellule survie si elle est entouré de minimum 3 autres cellule vivante. 
- une cellule nait quand elle a au minimum 3 voisines vivantent.
- une cellule meurt si elle possede moins de 3 voisines vivantent.

On creer dabord un bruit blanc (une grille de cellules blanches (mortes) et noires (vivantes)) et on y applique les regles ci dessus.

```C#
    private bool IsDeadOrAlive(int ix, int iy)
    {
        int aliveSisters = 0;

        for (int x = -1; x <= 1; x++)
        {
            for (int y = -1; y <= 1; y++)
            {

                if (x == 0 && y == 0)
                    continue;

                if (ix + x < 0 || ix + x > Grid.Width - 1 || iy + y < 0 || iy + y > Grid.Lenght - 1)
                    continue;

                if (currentBuffer[ix + x, iy + y])
                    aliveSisters++;

                if (aliveSisters >= AliveSisterNeeded)
                  breaks;
            }
        }

        return aliveSisters >= AliveSisterNeeded;
    }
```

On definit deux buffer : 
- un buffer actuel 
- un buffer ayant subit les modifications 

On verifie chaque cellules voisines sauf la cellule actuelle (0,0) et on incremmente le nombre de cellule vivante 

On renvoie la valeur booleene correspondant a la logique : est ce que je possede assez de cellule vivante ?

Et voici le resultat : 

![cellularAutomataGIF](https://i.imgur.com/ibB0hHK.gif)

---

## Noise Generation 

La méthode commence par créer une instance de **FastNoiseLite** en utilisant une seed random. Le bruit utilisé est du **OpenSimplex2**, idéal pour de la génération de terrain et simillaire au Prelin Noise.

L’algorithme parcourt chaque coordonnée X et Y de la grille, et pour chaque cellule 

En fonction de parametre pre-definis entre -1 et 1 on attribut des données a chaque cellule.
- Entre -1 et X on attribut de l'eau 
- entre X et Y on attribut du sable 
- entre Y et Z on attribut de l'herbe
- et au dessus de Z on attribut de la roche

Et voici le resultat : 

![openSimplex2GIF](https://i.imgur.com/2BbgxZj.gif)

---

## BSP (Binary Space Partitioning)

Le BSP a pour but de découpé une carte en plusieurs case de maniere recursive. La case principale sera découpé en deux puis chacune des cases encore séparer en deux de maniere jusqu'à ce que les plus petites cases respecte une certaine taille.
On obtient alors une grilles qu'on peut representer sous forme d'arbre.

![](https://www.tutorialspoint.com/computer_graphics/images/what_are_bsp_trees.jpg)

---

## Simple Room Placement

Le simple room placement est un moyen de generer de maniere procédurale des donjons. Il va creer dans un premier temps une liste de X nombre de salle non supperposée et les relier par des coulours.

---
### Création des salles :
On prend un point random on verifie qu'il est disponible et on y ajoute la salle. On repete l'opperation `_maxSteps` fois.

```C#
List<RectInt> roomList = new();
for (int i = 0; i < _maxSteps; i++)
{
  int x = RandomService.Range(0, Grid.Width);
  int y = RandomService.Range(0, Grid.Lenght);

  RectInt room = new RectInt(x, y, 10, 10);

  if (!CellAvailable(room))
    continue;

  PlaceRoom(room);
  roomList.Add(room);
}
```

--- 
### Creation des couloirs
On récupere le centre de chaque Room avec `GetCenter()` et on connecte les salles 

```C#
for (int i = 0; i < roomList.Count - 1; i++)
{
  Vector2Int a = roomList[i].GetCenter();
  Vector2Int b = roomList[i + 1].GetCenter();

  ConnectRooms(a, b);
}

```
```C#
private void ConnectRooms(Vector2Int a, Vector2Int b)
{
  for (int x = Mathf.Min(a.x, b.x); x <= Mathf.Max(a.x, b.x); x++)
  {
    if (!Grid.TryGetCellByCoordinates(x, a.y, out var cell);
      continue;
      AddTileToCell(cell, CORRIDOR_TILE_NAME, true);
    }

    for (int y = Mathf.Min(a.y, b.y); y <= Mathf.Max(a.y, b.y); y++)
    {
      if (!Grid.TryGetCellByCoordinates(b.x, y, out var cell))
        continue;

      AddTileToCell(cell, CORRIDOR_TILE_NAME, true);
    }
}
```
Et voici le resultat : 

![SimpleRoomPlacementGIF](https://i.imgur.com/qbyXUih.gif)
