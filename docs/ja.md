# Interactive Base — シーンの中でクリックできるもの

Interactive Base は、画像シーンにクリックできるオブジェクトを置きます。地面に落ちている物、扉、"墓の裏"の一点などです。オブジェクトには背景の上に重ねる独自の画像を持たせられ、マウスカーソルを重ねると光らせられ、クリックでシナリオを開始できます。シナリオは物語の進行に合わせてオブジェクトを追加・削除します。

Cthangover のプロローグがこれを使います。RPG World のタイルマップは、代わりに独自のイベントを使います。

## ゲームの中で

Cthangover の冒頭、路地では自転車のそばに赤い布が置かれています。これは背景の一部ではなく、独自の画像を持つインタラクティブオブジェクトです。

![Cthangover 冒頭の路地: 壁際の自転車と、それに掛かった赤い布。右上にはツールバー。](assets/alley.jpg)

マウスを布に重ねると、ポインターは手の形に変わり、布はかすかな金色を帯びます。

![マウスを重ねる前と後の布。右はわずかに暖かい色になります。](assets/cloth-hover.jpg)

クリックすると布のシナリオが始まります。Marao がそれを拾うと、シナリオはオブジェクトをシーンから取り除きます。

![クリック後: ダイアログボックスに Marao のポートレートと、布のシナリオの最初の行。](assets/cloth-clicked.jpg)

## インタラクティブオブジェクト

オブジェクトは、mod の `interactives/` フォルダーに入る JSON ファイルです。こちらは画像の布（Cthangover のもの）で、輪郭の頂点は短くしてあります。

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

- `texture` — 画面全体と同じ大きさの画像で、オブジェクト以外は透明なので、背景に自然と重なります。見えない地点にするなら省きます。
- `hitArea` — オブジェクトをクリックできる範囲を、画面に対する割合（0–1）で示します。`rect` は `x`、`y`、`width`、`height`、`circle` は `x`、`y`、`radius`、`polygon` は `vertices` を使います。
- `layer` — `background` か `foreground`。どちらも絵の一部で、ダイアログボックス・ツールバー・パネルの下に置かれます。`ui` は何よりも上に描かれます。
- `zIndex` — 同じレイヤーのオブジェクト同士の順序。数値が大きいほど上に描かれます。
- `cursor` — オブジェクト上でのマウスポインター。例は `PointingHand`。
- `highlight` — マウスが乗っている間の色づけ。`color`（透明度つき）、`scale`、フェードの `duration`（秒）です。
- `enabled`、`visible` — 反応するか、最初に描画されるか。
- `actions` — `onClick` は `scenario` ファイルかインラインの `commands` を実行し、`onHoverEnter` と `onHoverLeave` はシナリオコマンドを実行します。

## オブジェクトをシーンに置く

定義だけでは何も表示されません。追加するのはシーンのシナリオで、通常はクリック先のシナリオが取り除きます。

```scenario
interactive_add cloak_ground
interactive_remove cloak_ground
```

シーンが変わるとオブジェクトは消えるため、Cthangover は入るたびに、物語の状態を確かめるシーンのシナリオで改めて追加します。

```scenario
scene: prolog_alley
priority: 15
condition: AwakeningQuest.notHasTag("cloak_taken") && AwakeningQuest.notHasTag("book_taken")
---
interactive_add cloak_ground
interactive_add book_ground
end
```

すでに存在するオブジェクトを追加しても何も起きないので、繰り返しても安全です。

オブジェクトの配置とクリック領域の描画は、Editor Tools のインタラクティブエディターでも行えます。

## うまく動かないとき

1. **クリックしても何も起きない。** このシーンにオブジェクトが追加されていないか、`enabled` がオフか、クリックが `hitArea` の外だったかです。
2. **画像がずれている。** テクスチャーは背景と重なる画面全体の画像である必要があり、小さな切り抜きではいけません。
3. **取ったはずのオブジェクトが戻ってくる。** 追加する側のシナリオは、クリック先のシナリオが立てるのと同じタグやフラグを確認しなければなりません。
