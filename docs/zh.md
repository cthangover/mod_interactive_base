# Interactive Base — 场景中可点击的对象

Interactive Base 为图像场景加入可点击的对象：地上的东西、一扇门、"墓穴后面"的某个位置。对象可以有一张叠在背景之上的图片，鼠标悬停时可以发光，点击则启动一段剧本。剧本会随着故事推进添加和移除对象。

Cthangover 的序章使用了它。RPG World 的图块地图则使用自己的事件。

## 在游戏中

Cthangover 开场时，小巷的自行车旁放着一块红布。它不是背景的一部分，而是一个带有独立图片的交互物。

![Cthangover 开场的小巷：靠墙的自行车和挂在车上的红布，右上角是工具栏。](assets/alley.jpg)

当鼠标移到红布上时，指针变成手形，红布泛起一层淡淡的金色：

![红布在鼠标悬停前后的对比；右图色调略微偏暖。](assets/cloth-hover.jpg)

点击后启动红布的剧本：Marao 把它捡起来，剧本随即将这个对象从场景中移除。

![点击之后：对话框中出现 Marao 的肖像和红布剧本的第一行。](assets/cloth-clicked.jpg)

## 交互对象

对象是 mod 的 `interactives/` 文件夹里的 JSON 文件。下面就是截图中的红布（来自 Cthangover），轮廓顶点做了精简：

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

- `texture` — 一张覆盖整个屏幕的图片，除对象本身外全部透明，因此能自动与背景对齐。想要看不见的点击区就省略它。
- `hitArea` — 对象可以被点击的范围，以屏幕比例（0–1）表示：`rect` 用 `x`、`y`、`width`、`height`；`circle` 用 `x`、`y`、`radius`；或 `polygon` 用 `vertices`。
- `layer` — `background` 或 `foreground`；两者都属于画面本身，位于对话框、工具栏和各面板之下。`ui` 则画在一切之上。
- `zIndex` — 同一图层内对象之间的先后顺序：数值越大越靠上绘制。
- `cursor` — 鼠标悬停在对象上时的指针样式，例如 `PointingHand`。
- `highlight` — 鼠标悬停时的染色效果：`color`（带透明度）、`scale`，以及淡入时长 `duration`（秒）。
- `enabled`、`visible` — 对象是否响应点击，以及最初是否绘制。
- `actions` — `onClick` 运行一个 `scenario` 文件或内联的 `commands`；`onHoverEnter` 和 `onHoverLeave` 执行剧本命令。

## 把对象放进场景

仅有定义不会出现在任何地方。场景的剧本负责添加它，而点击触发的剧本通常会移除它：

```scenario
interactive_add cloak_ground
interactive_remove cloak_ground
```

切换场景时对象会被清空，因此 Cthangover 每次进入都用一段会检查剧情的场景剧本来重新添加它们：

```scenario
scene: prolog_alley
priority: 15
condition: AwakeningQuest.notHasTag("cloak_taken") && AwakeningQuest.notHasTag("book_taken")
---
interactive_add cloak_ground
interactive_add book_ground
end
```

添加已存在的对象不会有任何效果，所以重复执行是安全的。

也可以在 Editor Tools 的交互编辑器里摆放对象并绘制点击区域。

## 出问题时

1. **点击没有反应。** 对象从未在这个场景里添加过、`enabled` 未开启，或点击落在 `hitArea` 之外。
2. **图片位置不对。** 贴图必须是与背景对齐的全屏图片，而不是一小块裁剪图。
3. **对象被拿走后又出现了。** 添加它的剧本必须检查与点击剧本所设置的相同标签或标记。
