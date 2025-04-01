## Why fork?

解决AndroidPdfViewerV2 与 React Native 0.72.6(相近版本也存在该问题) 分别引用不同的 libc++_shared.so 导致冲突问题

## Describe question

```
AndroidPdfViewerV2   
   |_ pdfium-android
        |_ libc++_shared.so
```

详见 https://github.com/barteksc/AndroidPdfViewerV2/blob/master/android-pdf-viewer/build.gradle

pdfium-android库内包含libc++_shared.so，目录：PdfiumAndroid/src/main/jni/lib/arm64-v8a/libc++_
shared.so

该so与rn react-android下的libc++_shared.so重复：

```
2 files found with path 'lib/arm64-v8a/libc++_shared.so' from inputs:
-/Users/mac/.gradle/caches/transforms-3/b0270a9132f5d54e1cbc34f796ba0bfa/transformed/jetified-pdfium-android-1.9.0/jni/arm64-v8a/libc++_
shared.so
-/Users/mac/.gradle/caches/transforms-3/decbf84f3bab8ddf06d654d61140e56a/transformed/jetified-react-android-0.72.6-debug/jni/arm64-v8a/libc++_
shared.so
```

因配置 `pickFirst 'lib/arm64-v8a/libc++_shared.so'` 导致选择了pdfium-android下的so，rn下的libc++_
shared.so与pdfium-android的非同一版本，rn引用的libc++_shared.so含有symbol "__emutls_get_address"

故报异常：

```
FATAL EXCEPTION: main
             Process: test.cyberflow.zapry, PID: 1493
             java.lang.UnsatisfiedLinkError: dlopen failed: cannot locate symbol "__emutls_get_address" referenced by "/data/app/~~R0LG2F8LNGU8AwpmfQ0zVw==/test.cyberflow.zapry-wshrNdhM7abVe2b7qlqHAA==/base.apk!/lib/arm64-v8a/libfolly_runtime.so"...
             at java.lang.Runtime.loadLibrary0(Runtime.java:1081)
             at java.lang.Runtime.loadLibrary0(Runtime.java:1003)
             at java.lang.System.loadLibrary(System.java:1765)
             at com.facebook.soloader.nativeloader.SystemDelegate.loadLibrary(SystemDelegate.java:24)
             at com.facebook.soloader.nativeloader.NativeLoader.loadLibrary(NativeLoader.java:52)
             at com.facebook.soloader.nativeloader.NativeLoader.loadLibrary(NativeLoader.java:30)
             at com.facebook.soloader.SoLoader.loadLibrary(SoLoader.java:869)
             at com.facebook.hermes.reactexecutor.HermesExecutor.loadLibrary(HermesExecutor.java:26)
             at com.facebook.hermes.reactexecutor.HermesExecutor.<clinit>(HermesExecutor.java:20)
             at com.facebook.react.ReactInstanceManagerBuilder.getDefaultJSExecutorFactory(ReactInstanceManagerBuilder.java:379)
             at com.facebook.react.ReactInstanceManagerBuilder.build(ReactInstanceManagerBuilder.java:325)
             at com.facebook.react.ReactNativeHost.createReactInstanceManager(ReactNativeHost.java:96)
             at com.facebook.react.ReactNativeHost.getReactInstanceManager(ReactNativeHost.java:42)
             at com.***.lib.react.RNFakeViewManager.lazyReactRootViewThenCheckAdd(RNFakeViewManager.kt:41)
             at com.***.message.ui.home.HomeActivity.onCreate(HomeActivity.kt:151)
```

## 解决方案：

将AndroidPdfViewerV2内的libc++_shared.so删除，重新打包

这两个库都是7 8年前的工程，gradle版本较低，使用相应版本的gradle进行编译打包./gradlew build

## 相似问题

https://github.com/facebook/react-native/issues/43126

https://github.com/flyskywhy/playtorch/commit/8bcbb0c2c5679c70e0416f0a0a5e3aace529a833#diff-5ca845e716ffa59f02f9605a084e1844a55d6f63d7a16848a89333e70e1465c0

## 源库链接

AndroidPdfViewerV2: https://github.com/barteksc/AndroidPdfViewerV2/tree/master

pdfium-android: https://github.com/barteksc/PdfiumAndroid

AOSP pdfium: https://android.googlesource.com/platform/external/pdfium/
     