##JavaVM
标准Java平台下，每一个Process可以产生很多JavaVM对象，但在Android平台上，每一个Process只能产生一个Dalvik VM对象，也就是说在Android进程中是通过一个虚拟器对象来服务所有Java和c/c++代码。

###JavaVM使用
- 在加载动态链接库的时候，JVM会调用`JNI_OnLoad(JavaVM* jvm, void* reserved)`（如果定义了该函数）。第一个参数会传入JavaVM指针。

- 在native code中调用`JNI_CreateJavaVM(&jvm, (void**)&env, &vm_args)`可以得到JavaVM指针。

> 两种情况下，都可以用全局变量，比如`JavaVM* g_jvm`来保存获得的指针以便在任意上下文中使用。

##JNIEnv
JNIEnv是一个指针, 指向一个线程相关的结构, 线程相关结构指向JNI函数指针数组, 这个数组中存放了大量的JNI函数指针, 这些指针指向了具体的JNI函数。
![](http://img.my.csdn.net/uploads/201607/31/1469905986_5347.png)

###JNIEnv作用：
- 调用Java函数：JNIEnv代表Java运行环境, 可以使用JNIEnv调用Java中的代码;

- 操作Java对象：Java对象传入JNI层就是Jobject对象, 需要使用JNIEnv来操作这个Java对象;


###JNIEnv使用：
当本地c/c++代码想获得当前线程的JNIEnv时，可以使用两种方法：

- `vm->AttachCurrentThread(&env, 0)`
- `vm->GetEnv((void**)&env, JNI_VERSION_1_6)`

>需要强调的是JNIEnv是跟线程相关的，最好还是不要缓存这个JNIEnv* 。
>
>当创建的线程需要获取JNIEnv* 的时候，最好在刚创建的时候调用一次AttachCurrentThread，并且不要忘记线程结束的时候执行DettachCurrentThread。


##实例方法&静态方法
###实例方法:
`public native String getStringFromNative();`  
原生实例方法通过第二个参数获取实例引用，是**jobject**类型：

```
JNIEXPORT jstring JNICALL Java_com_dean_testndk_JNIHelper_getStringFromNative
        (JNIEnv *, jobject);
```

###静态方法:
`public static native String getStringFromNative();`  
静态方法没有与实例绑定，因此第二个参数是**jclass**类型：

```
JNIEXPORT jstring JNICALL Java_com_dean_testndk_JNIHelper_getStringFromNative
        (JNIEnv *, jclass);
```

##C/C++
原生代码中，c与c++调用JNI函数的语法不同：
###C：
`return (*env)->NewStringUTF(env, "Hello from JNI !");`
> C代码中，JNIEnv是指向JNIVativeInterface结构的指针，为了访问任何一个JNI函数，该指针需要首先被解引用。因为不了解JNI环境，所以需要将JNIEnv传递给调用者

###C++:
`return env->NewStringUTF("Hello from JNI !");`
> C++代码中，JNIEnv是C++类实例，JNI函数以成员函数的形式存在。因此不需要给调用着传递参数即可使用。

##数据类型
Java中有两种数据类型：

###基本数据类型：
boolean, byte, char, short, int, long, float, double

> Java基本数据类型，可以直接与C/C++的相应基本数据类型映射

###引用类型：
字符串类，数组类，以及其他类
> 与基本类型不同，引用类型对原生方法不透明，因此引用类型不能直接使用和修改，需要通过JNIEnv接口指针来调用JNI提供的API。

![](http://img.my.csdn.net/uploads/201607/31/1469905849_2543.png)

##类&对象

###获取类
- FindClass：`jclass FindClass(JNIEnv *env, const char *name);`  
通过传入类的完全限定名（注意JNI这边类的完全限定名通过"/"分隔，而不是Java那边的"."），即可得到一个对应的jclass对象。
```
jclass clz = (*env)->FindClass(env, "com/example/hello_jnicallback/JniHandler");
```


- GetObjectClass: `jclass GetObjectClass(JNIEnv *env, jobject obj);`  
根据一个jobject对象得到这个对象的jclass对象。这事获取jclass的另一种方式。
```
jclass clz = (*env)->GetObjectClass(env, instance);
```

###创建对象
- `jobject NewObject(JNIEnv *env, jclass clazz,
jmethodID methodID, ...);`

通过类，方法ID，对应参数来创建一个对象的实例：

```
jclass clz = (*env)->FindClass(env, "com/example/hello_jnicallback/JniHandler");

jmethodID jniHelperCtor = (*env)->GetMethodID(env, g_ctx.jniHelperClz, "<init>", "()V");

jobject handler = (*env)->NewObject(env, g_ctx.jniHelperClz, jniHelperCtor);
```
##域&方法
在JNI开发中，经常会在Native中调用Java的域和方法。  

Java的域和方法都有两类：

- 实例域，静态域；
- 实例方法，静态方法;

如下：

```
public class JavaClass {

    // 实例域
    private String instanceFiled = "Instance Field";
    // 静态域
    private static String staticFiled = "Static Field";

    // 实例方法
    private String instanceMethod() {
        return "Instance Method";
    }
    // 静态方法
    private static String staticMethod() {
        return "Static Method";
    }
}
```
下面例子是获取实例域，方法的代码。静态域，静态方法使用基本相同，都是先获取描述它的ID，然后在通过ID调用相应方法。

```
void nativeCallJavaClass(JNIEnv *env, jobject instance) {

    jclass clazz = env->GetObjectClass(instance);

    // 获取实例域的FieldID
    jfieldID instanceFieldId = env->GetFieldID(clazz, "instanceField", "Ljava/lang/String;");
    // 获取实例域
    jstring instanceField = (jstring) env->GetObjectField(instance, instanceFieldId);

    // 获取实例方法的MethodID
    jmethodID instanceMethodId = env->GetMethodID(clazz, "staticMethod", "Ljava/lang/String;");
    // 调用方法获得返回值
    jstring  instanceMethod = (jstring) env->CallObjectMethod(clazz, instanceMethodId);

};
```
> 为了提升应用程序的性能，对于频繁使用的域和方法，可以缓存它们的ID方便下次使用。


##描述符

![](http://img.my.csdn.net/uploads/201607/31/1469905850_6249.png)

###域描述符
- 基本类型的描述符
- 引用类型的描述符
	- 引用类型：L + 该类型类描述符 + ; 
		- String类型的域描述符为 `Ljava/lang/String`; 
	- 数组：[ + 其类型的域描述符 + ;
     	- int[] `[I`
		- float[] `[F`
		- String[] `[Ljava/lang/String;`
       - Object[] `[Ljava/lang/Object;`
	- 多维数组：n个[ +该类型的域描述符 , N代表的是几维数组。
		- int[][] `[[I`
       - float[][] `[[F`

###类描述符
- 类描述符：将完整的包名+类名中的.分隔符换成/分隔符
   - java.lang.String类的类描述符为：`java/lang/String`或者使用域描述符`[Ljava/lang/String;`

- 数组类型的描述符：同域描述符

###方法描述符
- String test() `Ljava/lang/String;`
- int f(int i, Object object) `(ILjava/lang/Object;)I`
- void set(byte[ ] bytes) `([B)V`

###Javap
java在bin中提供了javap命令，用于查看一个类方法的签名

cd到.class对应路径，然后执行`javap -s classname`
例如：

```
cd /Users/DeanGuo/TestNDK/app/build/intermediates/classes/debug/com/dean/testndk/;

javap -s JNIHelper

Compiled from "JNIHelper.java"
public class com.dean.testndk.JNIHelper {
  public com.dean.testndk.JNIHelper();
    descriptor: ()V

  public static native java.lang.String getStringFromNative();
    descriptor: ()Ljava/lang/String;
}
bogon:testndk DeanGuo$ javap -s JNIHelper
Warning: Binary file JNIHelper contains com.dean.testndk.JNIHelper
Compiled from "JNIHelper.java"
public class com.dean.testndk.JNIHelper {
  public com.dean.testndk.JNIHelper();
    descriptor: ()V

  public static native java.lang.String getStringFromNative();
    descriptor: ()Ljava/lang/String;
}

```
##局部&全局引用
###局部引用
局部引用不能在后续的调用中呗缓存及重用，主要因为它们的使用期限仅限于原生方法，一旦原生函数返回，局部引用就被释放。也可以用`void DeleteLocalRef(JNIEnv *env, jobject localRef);`函数显示的释放。

###全局引用
全局引用在原生方法的后续调用过程中依然有效，除非它们被原生代码显式释放。

- 创建全局引用：

```
jclass globalClazz;
jclass localClazz = env->FindClass("java/lang/String");
globalClazz = (jclass) env->NewGlobalRef(localClazz);
```
- 删除全局引用：

```
env->DeleteGlobalRef(globalClazz);
```
###弱全局引用
与全局应用一样，弱全局引用在原生方法的后续调用过程中依然有效。不同的是，弱全局引用并不阻止潜在对象被垃圾回事。

- 创建弱全局引用：

```
jclass weakGlobalClazz;
jclass localClazz = env->FindClass("java/lang/String");
weakGlobalClazz = (jclass) env->NewWeakGlobalRef(localClazz);
```

- 有效性检验：

```
if (JNI_FALSE == env->IsSameObject(weakGlobalClazz, nullptr)) {
    // 有效
} else {
    // 对象被垃圾回收器回收, 不可使用
}
```

- 删除弱全局引用：

```
env->DeleteGlobalRef(weakGlobalClazz);
```
##异常处理
JNI中的一场行为与Java不同，在Java中，当抛出一个异常时，虚拟机将停止执行代码块并进入异常处理。而JNI要求在异常发生后显示地实现异常处理。

```
jthrowable ex = env->ExceptionOccurred();
if (0 != ex) {
    env->ExceptionClear();
    // exception handler
}
```
###异常的捕获

##LOG
NDK开发中JNI时，可以使用`__android_log_print`打印log信息。
###使用步骤：

- 引入`#include <android/log.h>`头文件
- 加入Lib
	- **gralde**：在ndk标签下加入`ldLibs "log"`
	- **Android.mk**：加入`LOCAL_LDLIBS+= -L$(SYSROOT)/usr/lib -llog`

- 使用`__android_log_print(ANDROID_LOG_INFO, "JNITag","string From Java To C : %s", str); `

###封装：

```
#include <android/log.h>

// Android log function wrappers
static const char* kTAG = "testNDK";
#define LOGI(...) \
  ((void)__android_log_print(ANDROID_LOG_INFO, kTAG, __VA_ARGS__))
#define LOGW(...) \
  ((void)__android_log_print(ANDROID_LOG_WARN, kTAG, __VA_ARGS__))
#define LOGE(...) \
  ((void)__android_log_print(ANDROID_LOG_ERROR, kTAG, __VA_ARGS__))

// 使用如下：
LOGI("JNI_LOG");
```
##标准库
如果想在NDK中使用c++STL库：

- **Application.mk**：加入`APP_STL := gnustl_static`
- **gralde**：在ndk标签下加入`stl "gnustl_static"`

> - system：使用默认最小的C++运行库，这样生成的应用体积小，内存占用小，但部分功能将无法支持
- stlport_static：使用STLport作为静态库，这项是Android开发网极力推荐的
- stlport_shared：STLport 作为动态库，这个可能产生兼容性和部分低版本的Android固件，目前不推荐使用。
- gnustl_static：使用 GNU libstdc++ 作为静态库

###文章链接：

[JNI官方文档](http://docs.oracle.com/javase/6/docs/technotes/guides/jni/spec/jniTOC.html)


