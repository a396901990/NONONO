##简介
前面写了几篇NDK相关的文章：

- [NDK开发－简介&环境搭建(Eclipse，Android Studio)](http://blog.csdn.net/a396901990/article/details/51872849)
- [NDK开发－Android Studio+gradle-experimental开发NDK](http://blog.csdn.net/a396901990/article/details/51922182)，
- [NDK开发－零散知识点整理](http://blog.csdn.net/a396901990/article/details/52143847)
 

这个demo就是对这几篇文章中涉及内容的一个应用。其功能只是对数组进行排序。如果单纯使用Java来做十分简单，只需写个排序方法，或者将数组转化为集合，然后调用sort函数就ok了。  
但我们的目的是练习JNI所以需要使劲的折腾一下，其中主要涉及了：

- Java调用JNI函数
- JNI调用Java函数
- JNI中打印Android Log
- JNI中调用c++函数库
- JNI调用普通方法／静态方法
- JNI全局变量使用
- JNI数组的操作
- JNI异常处理

##演示图
####点击按钮对数组排序：
![这里写图片描述](http://img.blog.csdn.net/20160826181458629)

####排序成功：
![这里写图片描述](http://img.blog.csdn.net/20160826181546833)

##Java代码：

###Activity代码：
```
public class MainActivity extends AppCompatActivity {

    // 导入动态库
    static {
        System.loadLibrary("test");
    }

    // 本地排序方法
    public native boolean doSort(int [] array);

    private Button mOrderBtn;
    private TextView mResultText;

    // 需要排序的数组
    int[] orderArrays = {6,2,5,9,1,3,4,7,8,0,10};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 调用jni调用静态方法返回一个字符串(hello world?)
        Log.d("TestNDK", JNIHelper.getStringFromNative());

        mResultText = (TextView) this.findViewById(R.id.result);
        setResultText(orderArrays);

        mOrderBtn = (Button) this.findViewById(R.id.plus_btn);
        mOrderBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 调用本地排序方法, 根据是否有异常打印success/failed
                boolean isSuccess = doSort(orderArrays);
                mOrderBtn.setText(isSuccess?"success":"failed");
            }
        });

    }

    public void setResultText(String resultText) {
        mResultText.setText(resultText);
    }

    public void setResultText(int[] array) {
        mResultText.setText(Arrays.toString(array));
    }

}
```

###JNI工具类：

```
public class JNIHelper {

    @Keep
    private void updateStatus(String msg) {
        if (msg.toLowerCase().contains("error")) {
            Log.e("JniHandler", "Native Err: " + msg);
        } else {
            Log.i("JniHandler", "Native Msg: " + msg);
        }
    }

    public static native String getStringFromNative();
}
```

##JNI代码

###头文件
```
#include <jni.h>
/* Header for class com_dean_testndk_JNIHelper */

#ifndef _Included_com_dean_testndk_JNIHelper
#define _Included_com_dean_testndk_JNIHelper

#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     com_dean_testndk_JNIHelper
 * Method:    getStringFromNative
 * Signature: ()Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL
        Java_com_dean_testndk_JNIHelper_getStringFromNative(JNIEnv *, jclass);

JNIEXPORT jboolean JNICALL
        Java_com_dean_testndk_MainActivity_doSort(JNIEnv *, jobject, jintArray);
#ifdef __cplusplus
}
#endif
#endif
```

###c++代码

```
//
// Created by Dean Guo on 7/23/16.
//
#include "com_dean_testndk_JNIHelper.h"
#include "string.h"
#include "iostream"
#include "vector"
#include "algorithm"
#include <android/log.h>

using namespace std;
// Android log function wrappers
static const char* kTAG = "testNDK";
#define LOGI(...) \
  ((void)__android_log_print(ANDROID_LOG_INFO, kTAG, __VA_ARGS__))
#define LOGW(...) \
  ((void)__android_log_print(ANDROID_LOG_WARN, kTAG, __VA_ARGS__))
#define LOGE(...) \
  ((void)__android_log_print(ANDROID_LOG_ERROR, kTAG, __VA_ARGS__))

struct GLOBAL_CONTEXT {
    JavaVM* javaVM;
    jclass mainActivityClz;
    jobject mainActivityObj;

    jclass jniHelperClz;
    jobject jniHelperObj;
};

/**
 * 全局引用
 */
GLOBAL_CONTEXT mContext;

void sendJavaMsg(JNIEnv *env, jobject instance, jmethodID func, const char *msg) {

    jstring javaMsg = env->NewStringUTF(msg);
    env->CallVoidMethod(instance, func, javaMsg);
}

void callSetResult(JNIEnv *env, vector<int> vector_) {

    char buff[1000] = "";

    vector<int>::iterator it;

    for (it = vector_.begin(); it != vector_.end(); it++) {
        char numChar[100] = "";
        sprintf(numChar, "%d", *it);
        strcat(numChar, ",");
        strcat(buff, numChar);
    }

    jstring javaMsg = env->NewStringUTF(buff);
    jthrowable ex = env->ExceptionOccurred();
    if (0 != ex) {
        env->ExceptionClear();
        env->DeleteLocalRef(javaMsg);
        LOGE("Exception-sendJavaMsg!");
    } else {
        jmethodID methodID = env->GetMethodID(mContext.mainActivityClz, "setResultText", "(Ljava/lang/String;)V");
        env->CallVoidMethod(mContext.mainActivityObj, methodID, javaMsg);
        env->DeleteLocalRef(javaMsg);
    }
}

/**
 * 根据排序好的vector集合, 创建排序数组返回
 */
void callSetArrayResult(JNIEnv *env, vector<int> vector_) {

    int size = vector_.size();

    // 排序后需要返回的数组
    jintArray array = env->NewIntArray(size);

    // 为jni数组赋值
    jint *num = new jint[size];
    for (int i = 0; i < size; i++)
    {
        num[i] = vector_[i];
    }
    env->SetIntArrayRegion(array, 0, size, num);

    // 调用java方法传递排序好的数组
    jmethodID methodID = env->GetMethodID(mContext.mainActivityClz, "setResultText", "([I)V");
    env->CallVoidMethod(mContext.mainActivityObj, methodID, array);

    // 释放数组
    env->ReleaseIntArrayElements(array, num, 0);
}

/**
 * 系统函数, JNI初始化时调用。
 * 可在这个函数这里做一些初始化赋值
 */
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {

    LOGI("JNI_OnLoad");

    JNIEnv* env;

    mContext.javaVM = vm;

    if (vm->GetEnv((void**)&env, JNI_VERSION_1_6) != JNI_OK) {
        LOGE("JNI_ERR");
        return JNI_ERR; // 不支持该版本
    }

    // 对JNIHelper设置全局引用
    jclass clz = env->FindClass("com/dean/testndk/JNIHelper");
    mContext.jniHelperClz = (jclass) env->NewGlobalRef(clz);

    jmethodID jniHelperCtor = env->GetMethodID(mContext.jniHelperClz, "<init>", "()V");
    jobject handler = env->NewObject(mContext.jniHelperClz, jniHelperCtor);
    mContext.jniHelperObj = env->NewGlobalRef(handler);

    // 调用JNIHelper中updateStatus方法打印信息
    jmethodID statusId = env->GetMethodID(mContext.jniHelperClz, "updateStatus", "(Ljava/lang/String;)V");
    sendJavaMsg(env, mContext.jniHelperObj, statusId, "JNI: initializing...");

    return JNI_VERSION_1_6;
}

/**
 * 简单返回一个字符串
 */
JNIEXPORT jstring JNICALL Java_com_dean_testndk_JNIHelper_getStringFromNative
        (JNIEnv *env, jclass jclz) {
    return env->NewStringUTF("from Native");
}

/**
 * 排序方法, 成功返回true, 否则false
 */
JNIEXPORT jboolean JNICALL
Java_com_dean_testndk_MainActivity_doSort(JNIEnv *env, jobject instance, jintArray arr_) {
    // 设置全局引用
    mContext.mainActivityClz = env->GetObjectClass(instance);
    mContext.mainActivityObj = env->NewGlobalRef(instance);

    // 获取传递过来的数组指针
    jint *nums = env->GetIntArrayElements(arr_, NULL);
    // 获取数组长度
    jsize len = env->GetArrayLength(arr_);

    // 使用标准库中的vector排序
    vector<int> mVector;
    for (int i=0; i< len; i++) {
        mVector.push_back(nums[i]);
    }
    sort(mVector.begin(), mVector.end());

    // 调用返回结果函数
    callSetArrayResult(env, mVector);

    // 释放数组
    env->ReleaseIntArrayElements(arr_, nums, 0);

    // 判断是否有异常, 没有表示成功返回true, 否则false
    jthrowable ex = env->ExceptionOccurred();
    if (0 != ex) {
        env->ExceptionClear();
        LOGE("Exception!");
        return false;
    } else {
        return true;
    }
}
```


##最后

###下载
[https://github.com/a396901990/TestNDK](https://github.com/a396901990/TestNDK)
###总结
demo中所有详细的知识点都可以在[NDK开发－零散知识点整理](http://blog.csdn.net/a396901990/article/details/52143847)中找到。

如果想深入NDK还是推荐去看Google NDK的官方demo：
[https://github.com/googlesamples/android-ndk](https://github.com/googlesamples/android-ndk)