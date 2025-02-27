`Animatable`  是一个能够提供初始值的基础`API`， `animate*AsState` 系列 API 其内部均使用 `Animatable` 定制完成的。当你希望对自定义的数据类型进行动画计算时，可以选择使用 `Animatable` 来完成

## 1. 简单使用

接下来，我们仍然使用介绍 `animate*AsState` 时所使用的小红心作为示例。与之不同的是，这次我们使用其内部所包含的 `Animatable` 来直接完成。

值得注意的是，当我们在 `Composable` 使用 `Animatable` 时，其必须包裹在 `rememebr ` 中，如果你没有这么做，编译器会贴心的提示你添加 `rememeber` 。

> Creating an Animatable during composition without using remember

观察 `Animatable` 参数列表，可以发现传入了名为 `Dp.Companion.VectorConverter` 的参数。这实际上是一个类型转化器 `TwoWayConverter`。对于 `Animatable` 而言只认识 Float 类型，当你使用像 Dp 这种常用类型时，Google 已经在对应类型的伴生对象中通过拓展属性方式进行了提供。当你如果像为自定义类型适配 `Animatable` 时就需要自定义  `TwoWayConverter` 了。

还有一点值得注意，对于 `Animatable` 而言，动画数值更新需要在协程中完成，也就是调用 `animateTo` 方法。此时我们需要确保 `Animatable` 的初始状态与 `LaunchedEffect` 代码块首次执行时状态保持一致。在我们的示例中，我们使用 `Animatable(24.dp, Dp.Companion.VectorConverter)` 声明 `Animatable` 的初始状态为 `24.dp` ，由于 `LaunchedEffect` 代码块会跟随 `Composable` 首先执行一次，所以此时 `change` 所对应的状态与初始状态为 `24.dp`，否则页面被渲染后没有人为点击会自动进行 buttonSize `24.dp -> 32.dp` 的动画效果。

```kotlin
@Preview
@Composable
fun Demo() {
    var change by remember{ mutableStateOf(false) }
    var flag by remember{ mutableStateOf(false) }

    val buttonSizeVariable = remember {
        Animatable(24.dp, Dp.Companion.VectorConverter)
    }

    LaunchedEffect(change) {
        buttonSizeVariable.animateTo(if(change) 32.dp else 24.dp)
    }
    if(buttonSizeVariable.value == 32.dp) {
        change = false
    }
    IconButton(
        onClick = {
            change = true
            flag = !flag
        }
    ) {
        Icon(
            Icons.Rounded.Favorite,
            contentDescription = null,
            modifier = Modifier.size(buttonSizeVariable.value),
            tint = if(flag) Color.Red else Color.Gray
        )
    }
}
```

![]({{config.assets}}/design/animation/animatable/demo1.gif)

趁热打铁，让我们看看另一个实例如何使用 `Animatable` 来完成。

```kotlin
@Preview
@Composable
fun Demo() {
    var text by remember{ mutableStateOf("") }
    var focusState by remember { mutableStateOf(false)}
    var sizeVariable = remember {
        Animatable(0f)
    }

    LaunchedEffect(focusState) {
        sizeVariable.animateTo(if(focusState) 1f else 0.5f)
    }
    Column(
        modifier = Modifier.fillMaxWidth()
    ) {
        TextField(
            value = text,
            onValueChange = {
                text = it
            },
            modifier = Modifier
                .align(Alignment.CenterHorizontally)
                .onFocusChanged {
                    focusState = it.isFocused
                }
                .fillMaxWidth(sizeVariable.value)
        )
    }
}
```

![]({{config.assets}}/design/animation/animatable/demo2.gif)

与  `animate*AsState` 一样，你可以通过提供一个 `AnimationSpec` 来定制动画规格，设置 `animateTo` 的第二个参数即可。请参阅 [AnimationSpec]() 以了解更多信息。

```kotlin
suspend fun animateTo(
 	targetValue: T,
 	animationSpec: AnimationSpec<T> = defaultSpringSpec,
 	initialVelocity: T = velocity,
 	block: (Animatable<T, V>.() -> Unit)? = null
)
```

