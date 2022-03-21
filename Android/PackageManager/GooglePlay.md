
# GooglePlayPackage
 获取 Android 系统内是否安装了 Google Play 的状态 ，在安装了 Google 服务框架的情况下，传统的方式获取系统所有应用包名列表然后进行查找已然不合适。在 Android 11 版本以后、增加了运用包名访问权限。

>在创建应用时，请务必考虑您的应用打算访问的一组软件包，这些软件包代表设备上的其他已安装应用。如果您的应用以 Android 11（API 级别 30）或更高版本为目标平台，在默认情况下，系统会[自动让部分应用对您的应用可见](https://developer.android.com/training/basics/intents/package-visibility?hl=zh-cn#automatic)，但会隐藏其他应用。通过让部分应用在默认情况下不可见，系统可以了解应向您的应用显示哪些其他应用，这样有助于鼓励最小权限原则，还可帮助 Google Play 等应用商店评估应用为用户提供的隐私权和安全性。

 在 Android 11 SDK 版本以上访问运用是否安装、首先在 AndroidManifest.xml 中将其软件包名添加到 <queries> 元素内的一组 <package> 元素中：

```xml
<manifest package="com.example.game">
    <queries>
        <package android:name="com.example.store" />
        <package android:name="com.example.services" />
    </queries>
    ...
</manifest>
```
问题：在安装 Google 服务框架后，若没有安装 Google Play 或安装了 Google Play 然后卸载了，利用 PackageManager 去获取 Google Play 的包信息时会返回有效数据，但是手机系统并未安装 Google Play 运用，从而造成判断逻辑错误。

解决方案：首先利用 PackageManger 获取 Google Play 的包信息，若未返回 Google Play 包有效信息则证明系统内无 Google Play 运用；若返回包有效信息则用 PackageManager 的 resolveActivity() 函数，尝试确定对给定 Intent 执行的最佳操作，返回包含确定为最佳操作的最终活动意图的 ResolveInfo 对象。如果未找到匹配的活动，则返回 null。如果找到多个匹配的活动并且没有默认设置，则返回一个包含其他内容的 ResolveInfo 对象。

```kotlin
fun isAvailable(context: Context, packageName: String): Boolean {
    return try {
        val packageManager: PackageManager = context.packageManager
        val packageInfo = packageManager.getPackageInfo(packageName, PackageManager.GET_CONFIGURATIONS)
        if (packageName != googleMarkPackage) return packageInfo != null
        val intent = Intent(Intent.ACTION_VIEW).apply {
            data = Uri.parse("https://play.google.com/store/apps/details?id=com.example.android")
            setPackage("com.android.vending")
        }
        val resolveInfo = packageManager.resolveActivity(intent, PackageManager.MATCH_DEFAULT_ONLY)
        return packageInfo != null && resolveInfo != null
    } catch (e: Exception) {
        e.printStackTrace()
        false
    }
}
```
