# Proguard-master —— Android 代码混淆规则

## 1. Proguard介绍 ##
`Android SDK`自带了混淆工具`Proguard`。它位于SDK根目录`\tools\proguard`下面。
`ProGuard`是一个免费的Java类文件收缩，优化，混淆和预校验器。它可以检测并删除未使用的类，字段，方法和属性。它可以优化字节码，并删除未使用的指令。它可以将类、字段和方法使用短无意义的名称进行重命名。最后，预校验的Java6或针对Java MicroEdition的所述处理后的码。
如果开启了混淆，`Proguard`默认情况下会对所有代码，包括第三方包都进行混淆，可是有些代码或者第三方包是不能混淆的，这就需要我们手动编写混淆规则来保持不能被混淆的部分。

## 2. Proguard作用 ##
Android中的“混淆”可以分为两部分，一部分是 Java 代码的优化与混淆，依靠 `proguard` 混淆器来实现；另一部分是资源压缩，将移除项目及依赖的库中未被使用的资源(资源压缩严格意义上跟混淆没啥关系，但一般我们都会放一起讲)。
#### 2.1 代码混淆 ####
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
混淆后默认会在工程目录`app/build/outputs/mapping/release`下生成一个`mapping.txt`文件，这就是混淆规则，我们可以根据这个文件把混淆后的代码反推回源本的代码，所以这个文件很重要，注意保护好。原则上，代码混淆后越乱越无规律越好，但有些地方我们是要避免混淆的，否则程序运行就会出错。

#### 2.2 资源压缩 ####
资源压缩将移除项目及依赖的库中未被使用的资源，这在减少 apk 包体积上会有不错的效果，一般建议开启。具体做法是在 `build.grade` 文件中，将 `shrinkResources` 属性设置为 `true`。需要注意的是，只有在用`minifyEnabled true`开启了代码压缩后，资源压缩才会生效。
资源压缩包含了“合并资源”和“移除资源”两个流程。
“合并资源”流程中，名称相同的资源被视为重复资源会被合并。需要注意的是，这一流程不受`shrinkResources`属性控制，也无法被禁止， gradle 必然会做这项工作，因为假如不同项目中存在相同名称的资源将导致错误。gradle 在四处地方寻找重复资源：
 - `src/main/res/` 路径
 - 不同的构建类型（debug、release等等）
 - 不同的构建渠道
 - 项目依赖的第三方库
合并资源时按照如下优先级顺序：
```
    依赖 -> main -> 渠道 -> 构建类型
```
举个例子，假如重复资源同时存在于`main`文件夹和不同渠道中，gradle 会选择保留渠道中的资源。
同时，如果重复资源在同一层次出现，比如`src/main/res/` 和 `src/main/res2/`，则 `gradle` 无法完成资源合并，这时会报资源合并错误。
“移除资源”流程则见名知意，需要注意的是，类似代码，混淆资源移除也可以定义哪些资源需要被保留，这点在下文给出。

## 3. Proguard规则 ##
#### 3.1 基本指令 ####
 - **-ignorewarning**：是否忽略警告
 - **-optimizationpasses 5**：指定代码的压缩级别(在0~7之间，默认为5)
 - **-dontusemixedcaseclassnames**：是否使用大小写混合(windows大小写不敏感，建议加入)
 - **-dontskipnonpubliclibraryclasses**：是否混淆非公共的库的类
 - **-dontskipnonpubliclibraryclassmembers**：是否混淆非公共的库的类的成员
 - **-dontpreverify**：混淆时是否做预校验(Android不需要预校验，去掉可以加快混淆速度)
 - **-verbose**：混淆时是否记录日志(混淆后会生成映射文件)
 - **-obfuscationdictionary dictionary1.txt**：指定外部模糊字典
 - **-classobfuscationdictionary dictionary1.txt**：指定class模糊字典
 - **-packageobfuscationdictionary dictionary2.txt**：指定package模糊字典
 - **-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*,!code/allocation/variable**：混淆时所采用的算法(谷歌推荐算法)
 - **-libraryjars libs(*.jar;)**:添加支持的jar(引入libs下的所有jar包)
 - **-renamesourcefileattribute SourceFile**：将文件来源重命名为“SourceFile”字符串
 - **-keepattributes *Annotation***：保持注解不被混淆
 - **-keep class * extends java.lang.annotation.Annotation {*;}**：保持注解不被混淆
 - **-keep interface * extends java.lang.annotation.Annotation { *; }**：保持注解不被混淆
 - **-keepattributes Signature**：保持泛型不被混淆
 - **-keepattributes EnclosingMethod**：保持反射不被混淆
 - **-keepattributes Exceptions**：保持异常不被混淆
 - **-keepattributes InnerClasses**：保持内部类不被混淆
 - **-keepattributes SourceFile,LineNumberTable**：抛出异常时保留代码行号
#### 3.2 保留选项 ####
 - **-keep [,modifier，...] class_specification**：指定需要保留的类和类成员（作为公共类库，应该保留所有可公开访问的public方法）
 - **-keepclassmembers [,modifier，...] class_specification**：指定需要保留的类成员:变量或者方法
 - **-keepclasseswithmembers [,modifier，...] class_specification**：指定保留的类和类成员，条件是所指定的类成员都存在（既在压缩阶段没有被删除的成员，效果和keep差不多）
 - **-keepnames class_specification**:指定要保留名称的类和类成员，前提是在压缩阶段未被删除，仅用于模糊处理。[-keep allowshrinking class_specification 的简写]
 - **-keepclassmembernames class_specification**：指定要保留名称的类成员，前提是在压缩阶段未被删除，仅用于模糊处理。[-keepclassmembers allowshrinking class_specification 的简写]
 - **-keepclasseswithmembernames class_specification**：指定要保留名称的类成员，前提是在压缩阶段后所指定的类成员都存在，仅用于模糊处理。[-keepclasseswithmembers allowshrinking class_specification 的简写]
 - **-printseeds [filename]**：指定详尽列出由各种-keep选项匹配的类和类成员。列表打印到标准输出或给定文件。 该列表可用于验证是否真的找到了预期的类成员，特别是如果您使用通配符。

## 4. Keep命令说明 ##
命令 | 作用
| ------ | ------ |
-keep | 保持类和类成员，防止被移除或者被重命名
-keepnames | 保持类和类成员，防止被重命名
-keepclassmembers | 保持类成员，防止被移除或者被重命名
-keepclassmembernames | 保持类成员，防止被重命名
-keepclasseswithmembers | 保持拥有该成员的类和成员，防止被移除或者被重命名
-keepclasseswithmembernames | 保持拥有该成员的类和成员，防止被重命名

保持元素不参与混淆的规则的命令格式：
```
[保持命令] [类] {
    [成员]
}
```
“类”代表类相关的限定条件，它将最终定位到某些符合该限定条件的类。它的内容可以使用：
 - 具体的类
 - 访问修饰符（`public、protected、private`）
 - 通配符`*`，匹配任意长度字符，但不含包名分隔符(.)
 - 通配符`**`，匹配任意长度字符，并且包含包名分隔符(.)
 - `extends`，即可以指定类的基类
 - `implement`，匹配实现了某接口的类
 - `$`，内部类
“成员”代表类成员相关的限定条件，它将最终定位到某些符合该限定条件的类成员。它的内容可以使用：
 - `<init>` 匹配所有构造器
 - `<fields>` 匹配所有域
 - `<methods>` 匹配所有方法
 - 通配符`*`，匹配任意长度字符，但不含包名分隔符(.)
 - 通配符`**`，匹配任意长度字符，并且包含包名分隔符(.)
 - 通配符`***`，匹配任意参数类型
 - `…`，匹配任意长度的任意类型参数。比如void test(…)就能匹配任意 void test(String a) 或者是 void test(int a, String b) 这些方法。
 -  访问修饰符（`public、protected、private`）

#### 4.1 不混淆某个类 ####
```
    -keep public class com.android.proguard.example.Test { *; }
```
#### 4.2 不混淆某个包所有的类 ####
```
    -keep class com.android.proguard.example.** { *; }
```
#### 4.3 不混淆某个类的子类 ####
```
    -keep public class * extends com.android.proguard.example.Test { *; }
```
#### 4.4 不混淆所有类名中包含了“model”的类及其成员 ####
```
    -keep public class **.*model*.** {*;}
```
#### 4.5 不混淆某个接口的实现 ####
```
    -keep class * implements com.android.proguard.example.TestInterface { *; }
```
#### 4.6 不混淆某个类的构造方法 ####
```
    -keepclassmembers class com.android.proguard.example.Test {
        public <init>();
    }
```
#### 4.7 不混淆某个类的特定的方法 ####
```
    -keepclassmembers class com.android.proguard.example.Test {
        public void test(java.lang.String);
    }
```
#### 4.8 不混淆某个类的内部类 ####
```
    -keep class com.android.proguard.example.Test$* {
            *;
     }
```

## 5. Proguard注意事项 ##
#### 5.1 保持基本组件不被混淆 ####
```
    -keep public class * extends android.app.Fragment
    -keep public class * extends android.app.Activity
    -keep public class * extends android.app.Application
    -keep public class * extends android.app.Service
    -keep public class * extends android.content.BroadcastReceiver
    -keep public class * extends android.content.ContentProvider
    -keep public class * extends android.app.backup.BackupAgentHelper
    -keep public class * extends android.preference.Preference
```
#### 5.2 保持 Google 原生服务需要的类不被混淆 ####
```
    -keep public class com.google.vending.licensing.ILicensingService
    -keep public class com.android.vending.licensing.ILicensingService
```
#### 5.3 Support包规则 ####
```
    -dontwarn android.support.**
    -keep public class * extends android.support.v4.**
    -keep public class * extends android.support.v7.**
    -keep public class * extends android.support.annotation.**
```
#### 5.4 保持 native 方法不被混淆 ####
```
    -keepclasseswithmembernames class * { ####
        native <methods>;
    }
```
#### 5.5 保留自定义控件(继承自View)不被混淆 ####
```
    -keep public class * extends android.view.View { ####
        *** get*();
        void set*(***);
        public <init>(android.content.Context);
        public <init>(android.content.Context, android.util.AttributeSet);
        public <init>(android.content.Context, android.util.AttributeSet, int);
    }
```
#### 5.6 保留指定格式的构造方法不被混淆 ####
```
    -keepclasseswithmembers class * {
        public <init>(android.content.Context, android.util.AttributeSet);
        public <init>(android.content.Context, android.util.AttributeSet, int);
    }
```
#### 5.7 保留在Activity中的方法参数是view的方法(避免布局文件里面onClick被影响) ####
```
    -keepclassmembers class * extends android.app.Activity {
        public void *(android.view.View);
    }
```
#### 5.8 保持枚举 enum 类不被混淆 ####
```
    -keepclassmembers enum * {
        public static **[] values();
        public static ** valueOf(java.lang.String);
    }
```
#### 5.9 保持R(资源)下的所有类及其方法不能被混淆 ####
```
    -keep class **.R$* { *; }
```
#### 5.10 保持 Parcelable 序列化的类不被混淆(注：aidl文件不能去混淆) ####
```
    -keep class * implements android.os.Parcelable {
        public static final android.os.Parcelable$Creator *;
    }
```
#### 5.11 需要序列化和反序列化的类不能被混淆(注：Java反射用到的类也不能被混淆) ####
```
    -keepnames class * implements java.io.Serializable
```
#### 5.12 保持 Serializable 序列化的类成员不被混淆 ####
```
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
```
#### 5.13 保持 BaseAdapter 类不被混淆 ####
```
    -keep public class * extends android.widget.BaseAdapter { *; }
```
#### 5.14 保持 CusorAdapter 类不被混淆 ####
```
    -keep public class * extends android.widget.CusorAdapter{ *; }
```
#### 5.15 保持反射用到的类和与JavaScript进行交互的类不被混淆 ####

## 6. 自定义资源保持规则 ##
#### 6.1 keep.xml ####
用`shrinkResources true`开启资源压缩后，所有未被使用的资源默认被移除。假如你需要定义哪些资源必须被保留，在`res/raw/`路径下创建一个xml文件，例如`keep.xml`。
通过一些属性的设置可以实现定义资源保持的需求，可配置的属性有：
 - `tools:keep` 定义哪些资源需要被保留（资源之间用“,”隔开）
 - `tools:discard` 定义哪些资源需要被移除（资源之间用“,”隔开）
 - `tools:shrinkMode` 开启严格模式
当代码中通过 `Resources.getIdentifier()` 用动态的字符串来获取并使用资源时，普通的资源引用检查就可能会有问题。例如，如下代码会导致所有以“img_”开头的资源都被标记为已使用。
```java
    String name = String.format("img_%1d", angle + 1);
    res = getResources().getIdentifier(name, "drawable", getPackageName());
```
我们可以设置 `tools:shrinkMode` 为 `strict` 来开启严格模式，使只有确实被使用的资源被保留。
以上就是自定义资源保持规则相关的配置，举个例子：
```xml
    <?xml version="1.0" encoding="utf-8"?>
    <resources xmlns:tools="http://schemas.android.com/tools"
        tools:keep="@drawable/img_*,@drawable/ic_launcher,@layout/layout_used*"
        tools:discard="@layout/layout_unused"
        tools:shrinkMode="strict"/>
```

#### 6.2 移除替代资源 ####
一些替代资源，例如多语言支持的 `strings.xml`，多分辨率支持的 `layout.xml` 等，在我们不需要使用又不想删除掉时，可以使用资源压缩将它们移除。
我们使用 `resConfig` 属性来指定需要支持的属性，例如
```gradle
    android {
        defaultConfig {
            ...
            resConfigs "en", "zh"
        }
    }
```
其他未显式声明的语言资源将被移除。

## 7. Proguard使用 ##
####  7.1 开启混淆 ####
在项目的可执行工程Module中打开`build.gradle`文件进行编辑：
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
            shrinkResources true    // 移除未被使用的资源
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    ......
}
```
 - minifyEnabled：是否进行代码混淆
 - zipAlignEnabled：是否进行Zip压缩优化
 - shrinkResources：是否移除未被使用的资源
 - proguardFiles：混淆规则配置文件
     - proguard-android.txt：AndroidStudio默认自动导入的规则，这个文件位于Android SDK根目录\tools\proguard\proguard-android.txt。这里面是一些比较常规的不能被混淆的代码规则。
     - proguard-rules.pro：针对自己的项目需要特别定义的混淆规则，它位于项目每个Module的根目录下面，里面的内容需要我们自己编写。

####  7.2 编写混淆规则 ####
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
