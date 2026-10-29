# Interactive Base — 장면 안에서 클릭할 수 있는 것들

Interactive Base는 그림 장면에 클릭할 수 있는 오브젝트를 넣습니다. 땅에 놓인 물건, 문, "납골당 뒤"의 한 지점 같은 것입니다. 오브젝트는 배경 위에 그려지는 자체 이미지를 가질 수 있고, 마우스를 올리면 빛나며, 클릭하면 시나리오가 시작됩니다. 시나리오는 이야기가 진행됨에 따라 오브젝트를 추가하거나 제거합니다.

Cthangover의 프롤로그가 이를 사용합니다. RPG World의 타일 맵은 대신 자체 이벤트를 사용합니다.

## 게임 속에서

Cthangover 시작 부분에서 골목의 자전거 옆에는 붉은 천이 놓여 있습니다. 배경의 일부가 아니라 자체 이미지를 가진 인터랙티브 오브젝트입니다.

![Cthangover 시작의 골목: 벽에 기댄 자전거와 그에 걸린 붉은 천, 오른쪽 위의 툴바.](assets/alley.jpg)

마우스를 천 위에 올리면 포인터가 손 모양으로 바뀌고, 천은 은은한 금빛으로 물듭니다.

![마우스를 올리기 전과 후의 천; 오른쪽은 색조가 약간 따뜻해집니다.](assets/cloth-hover.jpg)

클릭하면 천의 시나리오가 시작됩니다. Marao가 그것을 주우면, 시나리오는 오브젝트를 장면에서 제거합니다.

![클릭 후: 대화 상자에 Marao의 초상화와 천의 시나리오 첫 줄.](assets/cloth-clicked.jpg)

## 인터랙티브 오브젝트

오브젝트는 mod의 `interactives/` 폴더에 들어 있는 JSON 파일입니다. 사진 속 천(Cthangover 것)이며, 외곽선 정점은 축약했습니다.

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

- `texture` — 화면 전체 크기의 그림으로, 오브젝트 외에는 투명하므로 배경과 저절로 맞춰집니다. 보이지 않는 지점으로 만들려면 생략합니다.
- `hitArea` — 오브젝트를 클릭할 수 있는 위치로, 화면 비율(0–1)로 나타냅니다. `rect`는 `x`, `y`, `width`, `height`, `circle`은 `x`, `y`, `radius`, `polygon`은 `vertices`를 씁니다.
- `layer` — `background` 또는 `foreground`. 둘 다 그림의 일부로 대화 상자, 툴바, 패널 아래에 머뭅니다. `ui`는 모든 것 위에 그려집니다.
- `zIndex` — 같은 레이어 오브젝트 사이의 순서: 숫자가 클수록 위에 그려집니다.
- `cursor` — 오브젝트 위의 마우스 포인터, 예를 들어 `PointingHand`.
- `highlight` — 마우스가 올라와 있는 동안의 색조: `color`(투명도 포함), `scale`, 페이드 시간 `duration`(초).
- `enabled`, `visible` — 반응하는지, 처음에 그려지는지.
- `actions` — `onClick`은 `scenario` 파일이나 인라인 `commands`를 실행하고, `onHoverEnter`와 `onHoverLeave`는 시나리오 커맨드를 실행합니다.

## 오브젝트를 장면에 넣기

정의만으로는 어디에도 나타나지 않습니다. 장면의 시나리오가 오브젝트를 추가하고, 보통 클릭의 시나리오가 제거합니다.

```scenario
interactive_add cloak_ground
interactive_remove cloak_ground
```

장면이 바뀌면 오브젝트는 지워지므로, Cthangover는 들어올 때마다 이야기를 확인하는 장면 시나리오로 다시 추가합니다.

```scenario
scene: prolog_alley
priority: 15
condition: AwakeningQuest.notHasTag("cloak_taken") && AwakeningQuest.notHasTag("book_taken")
---
interactive_add cloak_ground
interactive_add book_ground
end
```

이미 있는 오브젝트를 추가해도 아무 일도 일어나지 않으므로, 반복해도 안전합니다.

오브젝트 배치와 클릭 영역 그리기는 Editor Tools의 인터랙티브 편집기에서도 할 수 있습니다.

## 문제가 생겼을 때

1. **클릭해도 아무 일도 없습니다.** 오브젝트가 이 장면에 추가된 적이 없거나, `enabled`가 꺼져 있거나, 클릭이 `hitArea` 밖에 있었던 경우입니다.
2. **그림이 엉뚱한 곳에 있습니다.** 텍스처는 배경과 맞는 화면 전체 크기의 그림이어야 하며, 잘라낸 작은 조각이면 안 됩니다.
3. **가져간 오브젝트가 다시 나타납니다.** 추가하는 시나리오는 클릭의 시나리오가 설정하는 것과 같은 태그나 플래그를 확인해야 합니다.
