title: Niri 模糊特效設定
author: Kawa
tags:
  - Niri
  - blur
  - 模糊
  - Linux
categories:
  - Linux
date: 2026-06-05 20:04:53
---
[更新：2026-06-05 20:04:53]

Niri 26.04 版後支援[模糊背景](https://niri-wm.github.io/niri/Window-Effects.html)。
雖然我不需要這天功能，但還是跟風玩了一下。

讓背景視窗(非作用中/un-focused)套上模糊效果確實比較不會破壞專注力，
但...相對的，通訊軟件像 Discord, Line 的字就不容易看清了，一定要 focus 在上面才能看清楚，
如果不習慣，可以考慮調整模糊程度，或是用 window rule 把不想要模糊的視窗例如 Discord 排除在外。

編輯 `/.config/niri/config.kdl` 加入以下規則，就能開啟背景視窗模糊的效果：
```kdl
window-rule {
    match is-focused=false
    exclude app-id=r#"^discord$"#
    geometry-corner-radius 16
    
    opacity 0.7
    background-effect {
        blur true
        xray false
    }
}
```

模糊的特效是 background-effect 這個設定，
另外使用 geometry-corner-radius 作為模糊程度的參數。
加上 exclude 後可以排除指定的視窗在模糊規則之外。（搭配`niri msg windows`找出 APP ID）

實際的效果可以看這個 [Youtube](https://www.youtube.com/watch?v=n2cqVQCj-LU)。
