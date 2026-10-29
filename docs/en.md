# Interactive Base — things to click in a scene

Interactive Base puts clickable objects into a picture scene: a thing lying on the ground, a door, a spot "behind the crypt". An object can have its own picture drawn over the background, it can glow when the mouse is over it, and a click starts a scenario. Scenarios add and remove objects as the story goes on.

It is used by Cthangover's prologue. Tile maps of RPG World use their own events instead.

## In the game

At the start of Cthangover a red cloth lies by the bicycle in the alley. It is not part of the background: it is an interactive object with its own picture.

![The alley at the start of Cthangover: a bicycle by the wall and a red cloth hanging from it, the toolbar at the top right.](assets/alley.jpg)

When the mouse is over the cloth, the pointer becomes a hand and the cloth takes on a faint gold tint:

![The cloth without and with the mouse over it; on the right it is slightly warmer.](assets/cloth-hover.jpg)

A click starts the cloth's scenario: Marao picks it up, and the scenario removes the object from the scene.

![After the click: Marao's portrait and the first line of the cloth's scenario in the dialog box.](assets/cloth-clicked.jpg)

## An interactive object

Objects are JSON files in a mod's `interactives/` folder. This is the cloth from the pictures (from Cthangover), with its outline shortened:

```json
{
  "Items": [
    {
      "id": "cloak_ground",
      "texture": "cloak_ground",
      "layer": "foreground",
      "zIndex": 5,
      "cursor": "PointingHand",
      "hitArea": {
        "type": "polygon",
        "vertices": [
          { "x": 0.2796, "y": 0.7318 }, { "x": 0.3207, "y": 0.7167 },
          { "x": 0.3456, "y": 0.8434 }, { "x": 0.3387, "y": 0.8872 }
        ]
      },
      "highlight": { "color": "#FFD70044", "scale": 1, "duration": 0.12 },
      "actions": {
        "onClick": { "scenario": "scenarios/prolog/morning_cloak.scenario" }
      }
    }
  ]
}
```

- `texture` — a picture the size of the whole screen, transparent except for the object, so it lines up with the background by itself. Leave it out for an invisible spot.
- `hitArea` — where the object can be clicked, in fractions of the screen (0–1): `rect` with `x`, `y`, `width`, `height`; `circle` with `x`, `y`, `radius`; or `polygon` with `vertices`.
- `layer` — `background` or `foreground`; both are part of the picture and stay under the dialog box, the toolbar and the panels. `ui` is drawn above everything.
- `zIndex` — order among objects of the same layer: a higher number is drawn on top.
- `cursor` — the mouse pointer over the object, for example `PointingHand`.
- `highlight` — the tint while the mouse is over it: `color` (with transparency), `scale`, `duration` of the fade in seconds.
- `enabled`, `visible` — whether it reacts and whether it is drawn at first.
- `actions` — `onClick` runs a `scenario` file or inline `commands`; `onHoverEnter` and `onHoverLeave` run scenario commands.

## Putting objects into a scene

A definition alone does not appear anywhere. A scenario of the scene adds it, and the click's scenario usually removes it:

```scenario
interactive_add cloak_ground
interactive_remove cloak_ground
```

Objects are cleared when the scene changes, so Cthangover adds them again on every entry with a scene scenario that checks the story:

```scenario
scene: prolog_alley
priority: 15
condition: AwakeningQuest.notHasTag("cloak_taken") && AwakeningQuest.notHasTag("book_taken")
---
interactive_add cloak_ground
interactive_add book_ground
end
```

Adding an object that is already there does nothing, so this is safe to repeat.

Objects can also be placed and their click areas drawn in the interactive editor of Editor Tools.

## If something goes wrong

1. **Nothing happens on click.** The object was never added in this scene, it is not `enabled`, or the click is outside its `hitArea`.
2. **The picture is in the wrong place.** The texture must be a full-screen picture matching the background, not a small cut-out.
3. **The object comes back after it was taken.** The scenario that adds it must check the same tag or flag that the click's scenario sets.
