# AutoService 
 组件化开发框架可以细化为不同的部分，包括 Android UI、网络请求、数据库持久化、图片处理、View、工具类、sdk、内部统一风格组件等；框架包括但不限于通用功能，如果是部门内部项目中通用的功能，也可以独立出来成为一个通用的库存在。

原理：AutoService会自动在META-INF文件夹下生成Processor配置信息文件，该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候， 就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。

① 将 AutoService 依赖到项目中

```kotlin
implementation 'com.google.code.gson:gson:2.8.6'
```
② 在 gradle 中配置插件并添加 Javapoet 常用 api

```groovy
annotationProcessor 'com.google.code.gson:gson:2.8.6'
//Kotlin 需要kapt支持
apply plugin: 'kotlin-android'apply 
plugin: 'kotlin-android-extensions'
//kapt插件存在些许问题、博客地址：https://www.jianshu.com/p/b58d733bc54eapply 
plugin: 'kotlin-kapt'
```
③ 完成测试使用 @AutoService 注解

```kotlin
/******** 第一步 创建下沉接口  ********/
interface IWebViewService {    
fun startWebActivity(context: Context, title: String, url: String)
    fun startWebFragment(url: String): Fragment  
  fun starLocalTestHtml(context: Context)
}

/******** 第二步 实现接口 ********/ 
@AutoService(IWebViewService::class)
class WebViewServiceImpl : IWebViewService { 
   override fun startWebActivity(context: Context, title: String, url: String) {  
      WebActivity.create(context, title, url)   
 }   
 override fun startWebFragment(url: String): Fragment {     
   return WebFragment.create(url) 
   }   
 override fun starLocalTestHtml(context: Context) {    
    WebActivity.createHtml(context)  
  }
}

/******** 第三步 查找实例、进行通信 ********/ 
binding.starWebActivity.setOnClickListener {  
// AutoService工具类找实现
 AutoService.load(IWebViewService::class.java)?.apply {   
     starLocalTestHtml(this@AccountActivity)  
  }
}
object AutoService {  
  fun <S> load(clazz: Class<S>): S? {    
    val service = ServiceLoader.load(clazz).iterator()    
    try {      
      if (service.hasNext()) {  
              return service.next()     
       }       
 } catch (e: Exception) {     
       e.printStackTrace()    
    }     
   return null  
  }
}
```
至此就完成 AutoService 路由方案的配置。
