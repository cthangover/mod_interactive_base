# Interactive Base — objetos clicables dentro de la escena

Interactive Base coloca objetos clicables en las escenas de imagen: un objeto tirado en el suelo, una puerta, un punto «detrás de la cripta». Un objeto puede llevar su propia imagen dibujada sobre el fondo, puede brillar cuando el ratón pasa por encima, y un clic lanza un escenario. Los escenarios añaden y quitan objetos a medida que avanza la historia.

Lo usa el prólogo de Cthangover. Los mapas de tiles de RPG World funcionan en cambio con sus propios eventos.

## En el juego

Al principio de Cthangover, un paño rojo descansa junto a la bicicleta, en el callejón. No es parte del fondo: es un objeto interactivo con su propia imagen.

![El callejón al principio de Cthangover: una bicicleta junto a la pared y un paño rojo colgado de ella; la barra de herramientas arriba a la derecha.](assets/alley.jpg)

Cuando el ratón pasa sobre el paño, el puntero se convierte en una mano y el paño toma un leve tinte dorado:

![El paño sin el ratón encima y con él; a la derecha está ligeramente más cálido.](assets/cloth-hover.jpg)

Un clic inicia el escenario del paño: Marao lo recoge, y el escenario retira el objeto de la escena.

![Después del clic: el retrato de Marao y la primera línea del escenario del paño en la caja de diálogo.](assets/cloth-clicked.jpg)

## Un objeto interactivo

Los objetos son archivos JSON en la carpeta `interactives/` de un mod. Este es el paño de las imágenes (de Cthangover), con su contorno acortado:

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

- `texture` — una imagen del tamaño de toda la pantalla, transparente salvo por el objeto, de modo que encaja sola con el fondo. Omítela para un punto invisible.
- `hitArea` — dónde se puede hacer clic en el objeto, en fracciones de la pantalla (0–1): `rect` con `x`, `y`, `width`, `height`; `circle` con `x`, `y`, `radius`; o `polygon` con `vertices`.
- `layer` — `background` o `foreground`; ambos son parte de la imagen y quedan bajo la caja de diálogo, la barra de herramientas y los paneles. `ui` se dibuja sobre todo lo demás.
- `zIndex` — el orden entre los objetos de una misma capa: el número mayor se dibuja encima.
- `cursor` — el puntero del ratón sobre el objeto, por ejemplo `PointingHand`.
- `highlight` — el tinte mientras el ratón está encima: `color` (con transparencia), `scale` y `duration` del fundido en segundos.
- `enabled`, `visible` — si reacciona y si se dibuja al principio.
- `actions` — `onClick` ejecuta un archivo `scenario` o `commands` en línea; `onHoverEnter` y `onHoverLeave` ejecutan órdenes de escenario.

## Cómo se colocan los objetos en una escena

Una definición por sí sola no aparece en ninguna parte. La añade un escenario de la escena, y normalmente la retira el escenario del clic:

```scenario
interactive_add cloak_ground
interactive_remove cloak_ground
```

Los objetos se limpian al cambiar de escena, así que Cthangover los vuelve a añadir en cada entrada con un escenario de escena que consulta la historia:

```scenario
scene: prolog_alley
priority: 15
condition: AwakeningQuest.notHasTag("cloak_taken") && AwakeningQuest.notHasTag("book_taken")
---
interactive_add cloak_ground
interactive_add book_ground
end
```

Añadir un objeto que ya está no hace nada, así que se puede repetir sin miedo.

También se pueden colocar los objetos y dibujar sus áreas de clic en el editor de interactivos de Editor Tools.

## Si algo falla

1. **El clic no hace nada.** El objeto nunca se añadió en esta escena, no está `enabled`, o el clic cayó fuera de su `hitArea`.
2. **La imagen está en un lugar equivocado.** La textura debe ser una imagen a pantalla completa que encaje con el fondo, no un pequeño recorte.
3. **El objeto reaparece después de recogerlo.** El escenario que lo añade debe comprobar la misma etiqueta o marcador que activa el escenario del clic.
