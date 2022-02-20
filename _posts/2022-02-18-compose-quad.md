---
layout: post
title: "Android: Compose Custom Layout - Quad Layout"
categories: [Dev, Android]
tags: [android, compose]

---

# QuadLayout?

자식 View개수에 따라 전체 영역을 4분할 하여 노출해주는 뷰를 구현해보자. 약간만 수정하면 2,3개인경우 빈공간을 `PlaceHolder`로 채워주는 형태로 수정가능하니 참고.

### 단일 아이템

![single](/assets/img/220218_1_1.png)

### 아이템 4개

![quad](/assets/img/220218_1_2.png)

# 구현

```kotlin
@Composable
fun Quad(
    modifier: Modifier,
    gap: Dp = 1.dp,
    content: @Composable () -> Unit,
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurable, constraints ->
        check(measurable.size < 5) { "must quad item size 4 or less" }
        layout(constraints.maxWidth, constraints.maxHeight) {
            val eachWidth =
                if (measurable.size < 2) constraints.maxWidth else constraints.maxWidth / 2

            val eachHeight =
                if (measurable.size < 2) constraints.maxHeight else constraints.maxHeight / 2

            val childConstraint = Constraints.fixed(eachWidth, eachHeight)
            val gapPx = gap.roundToPx()

            measurable.map { it.measure(childConstraint) }
                .forEachIndexed { index, placeable ->
                    val x = (index % 2) * (eachWidth + gapPx)
                    val y = (index / 2) * (eachHeight + gapPx)
                    placeable.placeRelative(x, y)
                }
        }
    }
}

@Preview
@Composable
private fun PreviewQuadWithSingle() {
    Quad(modifier = Modifier.size(140.dp)) {
        Box(modifier = Modifier.background(Color.Red))
    }
}

@Preview
@Composable
private fun PreviewQuadWithQuad() {
    Quad(modifier = Modifier.size(140.dp)) {
        Box(modifier = Modifier.background(Color.Red))
        Box(modifier = Modifier.background(Color.White))
        Box(modifier = Modifier.background(Color.Green))
        Box(modifier = Modifier.background(Color.Yellow))
    }
}
```
