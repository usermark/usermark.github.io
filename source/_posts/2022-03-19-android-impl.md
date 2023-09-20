---
layout: article
title: '[Android] 應用'
date: 2022-03-19 17:00
tags: Android
---
紀錄開發上常遇到的問題，避免重複踩坑。
<!--more-->
# 各種進制轉換成10進制

```java
// 2轉10
assertEquals(102, Integer.parseInt("1100110", 2));
// 8轉10
assertEquals(56, Integer.parseInt("70", 8));
// 16轉10
assertEquals(-255, Integer.parseInt("-FF", 16));
```

# 10進制轉換成各種進制

```java
// 10轉2
assertEquals("10011", Integer.toBinaryString(19));
// 10轉8
assertEquals("23", Integer.toOctalString(19));
// 10轉16
assertEquals("c8", Integer.toHexString(200));
```

# 對齊或補0

```java
// 左對齊
assertEquals("15   ", String.format("%-5d", 15));
// 右對齊
assertEquals("   15", String.format("% 5d", 15));
// 前面補0
assertEquals("00015", String.format("%05d", 15));
```

# 加密

```java
private static final char[] HEX_ARRAY = "0123456789ABCDEF".toCharArray();

/**
 * 參考https://stackoverflow.com/questions/9655181/how-to-convert-a-byte-array-to-a-hex-string-in-java
 */
public static String bytesToHex(byte[] bytes) {
    char[] hexChars = new char[bytes.length * 2];
    for (int j = 0; j < bytes.length; j++) {
        int v = bytes[j] & 0xFF;
        hexChars[j * 2] = HEX_ARRAY[v >>> 4];
        hexChars[j * 2 + 1] = HEX_ARRAY[v & 0x0F];
    }
    return new String(hexChars);
}

public static String getSHA256(String message) {
    try {
        MessageDigest md = MessageDigest.getInstance("SHA-256");
        md.update(message.getBytes(StandardCharsets.UTF_8));
        return bytesToHex(md.digest());
    } catch (NoSuchAlgorithmException e) {
        return "";
    }
}

/**
 * 用 HMAC-SHA256 演算法加密並轉成 Base64
 */
public static String getHmac256WithBase64(String message, String key) throws Exception {
    final String algorithm = "HmacSHA256";
    Mac mac = Mac.getInstance(algorithm);
    mac.init(new SecretKeySpec(key.getBytes(), algorithm));
    return Base64.encodeToString(mac.doFinal(message.getBytes()), Base64.NO_WRAP);
}
```

# 隱藏鍵盤

```java
public static void hideKeyboard(View view) {
    InputMethodManager imm = (InputMethodManager) view.getContext()
            .getSystemService(Context.INPUT_METHOD_SERVICE);
    imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
}
```

# 將keystore資料保存在build.gradle外

當有重要資料，例如keystore的帳密不想直接寫在build.gradle內時，可改用properties保存。

首先在根目錄新增keystore.properties
```
storePassword=my.keystore
keyPassword=key_password
keyAlias=my_key_alias
storeFile=store_file
appID=my_app_id
```

修改build.gralde，新增以下內容
```groovy
// Load keystore
def keystorePropertiesFile = rootProject.file("keystore.properties")
def keystoreProperties = new Properties()
keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

android {
    defaultConfig {
        buildConfigField "String", "APP_ID", "\"$keystoreProperties.appID\""
    }
    signingConfigs {
        release {
            storeFile file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
        }
    }
}
```

這樣做還有個好處是可將properties排除於git版控，只保留重要或敏感的資料於本地端。

參考<https://stackoverflow.com/questions/20562189/sign-apk-without-putting-keystore-info-in-build-gradle>

# 使用AsyncTask注意事項

SecondAsyncTask並不會和FirstAsyncTask並行執行，而是等FirstAsyncTask先跑完，才跑SecondAsyncTask。

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    for (int i = 0; i < 10; i++) {
        new FirstAsyncTask().execute(i);
    }
    new SecondAsyncTask().execute();
}

static class FirstAsyncTask extends AsyncTask<Integer, Void, Void> {

    @Override
    protected Void doInBackground(Integer... params) {
        Log.d("Mike", "first " + params[0]);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return null;
    }
}

static class SecondAsyncTask extends AsyncTask<Void, Void, Void> {

    @Override
    protected Void doInBackground(Void... params) {
        Log.d("Mike", "second");
        return null;
    }
}
```

原因是AsyncTask，預設是跑SERIAL_EXECUTOR，採排隊的方式。

若要並行執行，原本呼叫execute()部分要改成executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR)即可。
```java
public static final Executor THREAD_POOL_EXECUTOR;

static {
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
            CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>(), sThreadFactory);
    threadPoolExecutor.setRejectedExecutionHandler(sRunOnSerialPolicy);
    THREAD_POOL_EXECUTOR = threadPoolExecutor;
}

private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
```

最後，官方已不建議再用AsyncTask，應自行實作主執行緒切換背景執行緒的方案。

# 使用Java 8 API

因開發需求要用到Java 8的java.time.Period來計算日期間距，才因此接觸到Android Gradle提供的脫糖引擎，可參考https://developer.android.com/studio/write/java8-support?hl=zh-tw#library-desugaring

首先使用的Android Gradle外掛程式版本需要4.0.0以上。接著修改build.gradle如下便可。

```groovy
android {
    defaultConfig {
        // Required when setting minSdkVersion to 20 or lower
        multiDexEnabled true
    }

    compileOptions {
        // Flag to enable support for the new language APIs
        coreLibraryDesugaringEnabled true
        // Sets Java compatibility to Java 8
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:2.0.3'
}
```

# 編譯遇到Execution failed for task ':app:checkDebugDuplicateClasses'

完整錯誤如下
```
* Exception is:
org.gradle.api.tasks.TaskExecutionException: Execution failed for task ':app:checkDebugDuplicateClasses'.
Caused by: org.gradle.workers.internal.DefaultWorkerExecutor$WorkExecutionException: A failure occurred while executing com.android.build.gradle.internal.tasks.CheckDuplicatesRunnable
    at java.base/java.util.Optional.orElseGet(Optional.java:364)
Caused by: java.lang.RuntimeException: Duplicate class kotlin.collections.jdk8.CollectionsJDK8Kt found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.internal.jdk7.JDK7PlatformImplementations found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk7-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.6.21)
Duplicate class kotlin.internal.jdk7.JDK7PlatformImplementations$ReflectSdkVersion found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk7-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.6.21)
Duplicate class kotlin.internal.jdk8.JDK8PlatformImplementations found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.internal.jdk8.JDK8PlatformImplementations$ReflectSdkVersion found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.io.path.ExperimentalPathApi found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk7-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.6.21)
Duplicate class kotlin.io.path.PathRelativizer found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk7-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.6.21)
Duplicate class kotlin.io.path.PathsKt found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk7-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.6.21)
Duplicate class kotlin.io.path.PathsKt__PathReadWriteKt found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk7-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.6.21)
Duplicate class kotlin.io.path.PathsKt__PathUtilsKt found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk7-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.6.21)
Duplicate class kotlin.jdk7.AutoCloseableKt found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk7-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.6.21)
Duplicate class kotlin.jvm.jdk8.JvmRepeatableKt found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.random.jdk8.PlatformThreadLocalRandom found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.streams.jdk8.StreamsKt found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.streams.jdk8.StreamsKt$asSequence$$inlined$Sequence$1 found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.streams.jdk8.StreamsKt$asSequence$$inlined$Sequence$2 found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.streams.jdk8.StreamsKt$asSequence$$inlined$Sequence$3 found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.streams.jdk8.StreamsKt$asSequence$$inlined$Sequence$4 found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.text.jdk8.RegexExtensionsJDK8Kt found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)
Duplicate class kotlin.time.jdk8.DurationConversionsJDK8Kt found in modules kotlin-stdlib-1.8.22 (org.jetbrains.kotlin:kotlin-stdlib:1.8.22) and kotlin-stdlib-jdk8-1.6.21 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.21)

Go to the documentation to learn how to <a href="d.android.com/r/tools/classpath-sync-errors">Fix dependency resolution errors</a>.
	at com.android.build.gradle.internal.tasks.CheckDuplicateClassesDelegate.run(CheckDuplicateClassesDelegate.kt:65)
	at com.android.build.gradle.internal.tasks.CheckDuplicatesRunnable.execute(CheckDuplicateClassesDelegate.kt:91)
	... 5 more
```

於build.gradle加上該設定
```groovy
configurations.implementation {
    exclude group: 'org.jetbrains.kotlin', module: 'kotlin-stdlib-jdk8'
}
```