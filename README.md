# Proguard-master —— Android 代码混淆规则

## Proguard介绍 ##
Android SDK自带了混淆工具Proguard。它位于SDK根目录\tools\proguard下面。
ProGuard是一个免费的Java类文件收缩，优化，混淆和预校验器。它可以检测并删除未使用的类，字段，方法和属性。它可以优化字节码，并删除未使用的指令。它可以将类、字段和方法使用短无意义的名称进行重命名。最后，预校验的Java6或针对Java MicroEdition的所述处理后的码。
如果开启了混淆，Proguard默认情况下会对所有代码，包括第三方包都进行混淆，可是有些代码或者第三方包是不能混淆的，这就需要我们手动编写混淆规则来保持不能被混淆的部分。

## Proguard作用 ##
**压缩（Shrinking）**：默认开启，用以减小应用体积，移除未被使用的类和成员，并且会在优化动作执行之后再次执行（因为优化后可能会再次暴露一些未被使用的类和成员）。
```
    -dontshrink 关闭压缩
```
**优化（Optimization）**：默认开启，在字节码级别执行优化，让应用运行的更快。
```
    -dontoptimize  关闭优化
    -optimizationpasses n 表示proguard对代码进行迭代优化的次数，Android一般为5
```
**混淆（Obfuscation）**：默认开启，增大反编译难度，类、函数、变量名会被随机命名成无意义的代号形如：a,b,c...之类的，除非用keep保护。
```
    -dontobfuscate 关闭混淆
```
上面这几个功能都是默认打开的，要关闭他们只需配置对应的规则即可。
混淆后默认会在工程目录app/build/outputs/mapping/release下生成一个mapping.txt文件，这就是混淆规则，我们可以根据这个文件把混淆后的代码反推回源本的代码，所以这个文件很重要，注意保护好。原则上，代码混淆后越乱越无规律越好，但有些地方我们是要避免混淆的，否则程序运行就会出错。

## Proguard规则 ##


## Proguard使用 ##
#### 开启混淆 ####
在项目的可执行工程Module中打开build.gradle文件进行编辑：
```gradle
android {
    ......
    defaultConfig {
        ......
    }
    buildTypes {
        release {
            minifyEnabled true      // 开启代码混淆
            zipAlignEnabled true    // 开启Zip压缩优化
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    ......
}
```
 - minifyEnabled：是否进行代码混淆
 - zipAlignEnabled：是否进行Zip压缩优化
 - proguardFiles：混淆规则配置文件
     - proguard-android.txt：AndroidStudio默认自动导入的规则，这个文件位于Android SDK根目录\tools\proguard\proguard-android.txt。这里面是一些比较常规的不能被混淆的代码规则。
     - proguard-rules.pro：针对自己的项目需要特别定义的混淆规则，它位于项目每个Module的根目录下面，里面的内容需要我们自己编写。

#### 编写混淆规则 ####
```pro
# --------------------------------------------基本指令区--------------------------------------------#
-ignorewarning                                      # 是否忽略警告
-optimizationpasses 5                               # 指定代码的压缩级别(在0~7之间，默认为5)
-dontusemixedcaseclassnames                         # 是否使用大小写混合(windows大小写不敏感，建议加入)
-dontskipnonpubliclibraryclasses                    # 是否混淆非公共的库的类
-dontskipnonpubliclibraryclassmembers               # 是否混淆非公共的库的类的成员
-dontpreverify                                      # 混淆时是否做预校验(Android不需要预校验，去掉可以加快混淆速度)
-verbose                                            # 混淆时是否记录日志(混淆后会生成映射文件)

#指定外部模糊字典
-obfuscationdictionary dictionary1.txt
#指定class模糊字典
-classobfuscationdictionary dictionary1.txt
#指定package模糊字典
-packageobfuscationdictionary dictionary2.txt

# 混淆时所采用的算法(谷歌推荐算法)
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*,!code/allocation/variable

# 添加支持的jar(引入libs下的所有jar包)
-libraryjars libs(*.jar;)

# 将文件来源重命名为“SourceFile”字符串
-renamesourcefileattribute SourceFile

# 保持注解不被混淆
-keepattributes *Annotation*
-keep class * extends java.lang.annotation.Annotation {*;}

# 保持泛型不被混淆
-keepattributes Signature
# 保持反射不被混淆
-keepattributes EnclosingMethod
# 保持异常不被混淆
-keepattributes Exceptions
# 保持内部类不被混淆
-keepattributes Exceptions,InnerClasses
# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable

# --------------------------------------------默认保留区--------------------------------------------#
# 保持基本组件不被混淆
-keep public class * extends android.app.Fragment
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference

# 保持 Google 原生服务需要的类不被混淆
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# Support包规则
-dontwarn android.support.**
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**

# 保持 native 方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留自定义控件(继承自View)不被混淆
-keep public class * extends android.view.View {
    *** get*();
    void set*(***);
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 保留指定格式的构造方法不被混淆
-keepclasseswithmembers class * {
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 保留在Activity中的方法参数是view的方法(避免布局文件里面onClick被影响)
-keepclassmembers class * extends android.app.Activity {
    public void *(android.view.View);
}

# 保持枚举 enum 类不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保持R(资源)下的所有类及其方法不能被混淆
-keep class **.R$* { *; }

# 保持 Parcelable 序列化的类不被混淆(注：aidl文件不能去混淆)
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# 需要序列化和反序列化的类不能被混淆(注：Java反射用到的类也不能被混淆)
-keepnames class * implements java.io.Serializable

# 保持 Serializable 序列化的类成员不被混淆
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# 保持 BaseAdapter 类不被混淆
-keep public class * extends android.widget.BaseAdapter { *; }
# 保持 CusorAdapter 类不被混淆
-keep public class * extends android.widget.CusorAdapter{ *; }

# --------------------------------------------webView区--------------------------------------------#
# WebView处理，项目中没有使用到webView忽略即可
# 保持Android与JavaScript进行交互的类不被混淆
-keep class **.AndroidJavaScript { *; }
-keepclassmembers class * extends android.webkit.WebViewClient {
     public void *(android.webkit.WebView,java.lang.String,android.graphics.Bitmap);
     public boolean *(android.webkit.WebView,java.lang.String);
}
-keepclassmembers class * extends android.webkit.WebChromeClient {
     public void *(android.webkit.WebView,java.lang.String);
}

# 网络请求相关
-keep public class android.net.http.SslError

# --------------------------------------------删除代码区--------------------------------------------#
# 删除代码中Log相关的代码
-assumenosideeffects class android.util.Log {
    public static boolean isLoggable(java.lang.String, int);
    public static int v(...);
    public static int i(...);
    public static int w(...);
    public static int d(...);
    public static int e(...);
}


# --------------------------------------------可定制化区--------------------------------------------#
#---------------------------------1.实体类---------------------------------



#--------------------------------------------------------------------------

#---------------------------------2.与JS交互的类-----------------------------



#--------------------------------------------------------------------------

#---------------------------------3.反射相关的类和方法-----------------------



#--------------------------------------------------------------------------

#---------------------------------2.第三方依赖--------------------------------



#--------------------------------------------------------------------------


```





