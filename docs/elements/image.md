```kotlin
@Composable
fun Image(
    painter: Painter,
    contentDescription: String?,
    modifier: Modifier = Modifier,
    alignment: Alignment = Alignment.Center,
    contentScale: ContentScale = ContentScale.Fit,
    alpha: Float = DefaultAlpha,
    colorFilter: ColorFilter? = null
)
```

!!! 注意
    目前在 `Compose` 中 `Image` 有三种，详情可先在[底部](http://localhost:8000/elements/image/#4)找到

`Image` 可以帮我们加载一张图片。

``` kotlin
@Composable
fun ImageDemo() {

    Image(
        painter = painterResource(id = R.drawable.wallpaper),
        contentDescription = null
    )

}
```
![]({{config.assets}}/elements/image/image.png)

## 1. 图片大小

我们可以使用 `Modifier.size()` 来设置图片大小。

``` kotlin
@Composable
fun ImageDemo() {

    Image(
        painter = painterResource(id = R.drawable.wallpaper),
        contentDescription = null,
        modifier = Modifier.size(350.dp)
    )

}
```

![]({{config.assets}}/elements/image/image2.png)

## 2. 图片形状

我们可以使用 `Surface` 来帮助我们设置形状。

``` kotlin
@Composable
fun ImageDemo() {

    Surface(
        shape = CircleShape
    ) {
        Image(
            painter = painterResource(id = R.drawable.wallpaper),
            contentDescription = null,
            modifier = Modifier.size(350.dp)
        )
    }

}
```

![]({{config.assets}}/elements/image/image3.png)

是不是有一点小问题？似乎只有左右两边变成了圆形，而上下并没有。

这是因为 `Image` 中源码的 `contentScale` 参数默认是 `ContentScale.Fit`，

也就是保持图片的宽高比，缩小到可以完整显示整张图片。

而 `ContentScale.Crop` 也是保持宽高比，但是尽量让宽度**或者**高度完整的占满。

所以我们将 `contentScale` 设置成 `ContentScale.Crop`。

``` kotlin
@Composable
fun ImageDemo() {

    Surface(
        shape = CircleShape
    ) {
        Image(
            painter = painterResource(id = R.drawable.wallpaper),
            contentDescription = null,
            modifier = Modifier.size(350.dp),
            contentScale = ContentScale.Crop
        )
    }

}
```

![]({{config.assets}}/elements/image/image4.png)

## 3. 图像边框

你可以利用 `Surface` 中的 `border` 参数来设置边框。

``` kotlin
@Composable
fun ImageDemo() {

    Surface(
        shape = CircleShape,
        border = BorderStroke(5.dp, Color.Gray)
    ) {
        Image(
            painter = painterResource(id = R.drawable.wallpaper),
            contentDescription = null,
            modifier = Modifier.size(350.dp),
            contentScale = ContentScale.Crop
        )
    }

}
```

![]({{config.assets}}/elements/image/image5.png)

## 4. 使用 Coil 来动态加载图片

Compose 自带的 `Image` 只能加载资源管理器中的图片文件，如果想加载网络图片或者是其他本地路径下的文件，则需要考虑其他的库，比如 [Coil](https://coil-kt.github.io/coil/compose/)

简单使用 Coil 加载网络图片:

记得打开网络权限测试

    <uses-permission android:name="android.permission.INTERNET" />


``` kotlin
Image(
    painter = rememberImagePainter(data = "https://picsum.photos/300/300"),
    contentDescription = null
)
```

### 加载 Svg 图像

`Coil` 可以加载 Svg 图像



添加依赖 

    implementation "io.coil-kt:coil-svg:1.3.2" // Gradle
    implementation("io.coil-kt:coil-svg:1.3.2") // KTS


``` kotlin
val context = LocalContext.current
val imageLoader = ImageLoader.Builder(context)
    .componentRegistry {
        add(SvgDecoder(context))
    }
    .build()

Image(
    painter = rememberImagePainter(
        data = "https://coil-kt.github.io/coil/images/coil_logo_black.svg",
        imageLoader = imageLoader
    ),
    contentDescription = null
)
```

### 放大缩小 Svg 图像文件

虽然 Coil 可以显示 Svg 图像，但是如果在我们的 app 中，需要动态的放大 Svg 图像，那么你大概率会得到强行拉升 Svg 像素后的图像，而不是无损放大

导致的原因可能是 Coil 中的 ImageLoader 会把 Svg 转换成位图，而不是安卓的矢量图 vector drawable, 而位图则不能无损放大

``` kotlin
val context = LocalContext.current
val imageLoader = ImageLoader.Builder(context)
    .componentRegistry {
        add(SvgDecoder(context))
    }
    .build()

var flag by remember { mutableStateOf(false) }
val size by animateDpAsState(targetValue = if(flag) 450.dp else 50.dp)

Box(
    modifier = Modifier.fillMaxSize(),
    contentAlignment = Alignment.Center
) {
    Column {
        Image(
            painter = rememberImagePainter(
                data = "https://coil-kt.github.io/coil/images/coil_logo_black.svg",
                imageLoader = imageLoader
            ),
            contentDescription = null,
            modifier = Modifier
                .size(size)
                .clickable(
                    onClick = {
                        flag = !flag
                    },
                    indication = null,
                    interactionSource = MutableInteractionSource()
                )
        )
    }
}
```

![]({{config.assets}}/elements/image/demo.gif)

那么要解决这个问题，就是尝试实现 svg 转换为 vector drawable, 在 github 上可以搜索到这个库 [Svg to Compose](https://github.com/DevSrSouza/svg-to-compose)，但是它并不能支持动态加载图像，例如从网络上加载，或者是从 app 的内部存储或者外部存储加载。

但是在我搜寻中，发现了 [Landscapist](https://github.com/skydoves/Landscapist)

依赖：

    implementation "com.github.skydoves:landscapist-coil:1.3.2"

``` kotlin
CoilImage(
    imageModel = "https://coil-kt.github.io/coil/images/coil_logo_black.svg",
    contentDescription = null,
    modifier = Modifier
        .size(size)
        .clickable(
            onClick = {
                flag = !flag
            },
            indication = null,
            interactionSource = MutableInteractionSource()
        ),
    imageLoader = imageLoader
)
```

![]({{config.assets}}/elements/image/demo2.gif)


## 5. 更多

[Image 参数详情](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary#image)

[Ucrop 一个图片裁剪库](https://github.com/Yalantis/uCrop)

[Surface 详情](surface.md)

[Coil](https://coil-kt.github.io/coil/)