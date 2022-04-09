---
layout: post
title: '[Android] 應用'
date: 2022-03-19 17:00
categories: 
---
紀錄開發上常遇到的問題，避免重複踩坑。

- toc
{: toc }

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

# 客製化View的屬性

需覆寫含兩個參數的建構子，才可應用在xml上。

其中字型大小需特別注意，在Pixel 1080x1920的density為2.625，用getDimension()取得12sp的輸出會是52.5px，所以setTextSize()時要用TypedValue.COMPLEX_UNIT_PX轉換回去，才可正常顯示大小。

參考[Android中getDimension,getDimensionPixelOffset和getDimensionPixelSize 区别](https://blog.csdn.net/weixin_42814000/article/details/107069608?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_paycolumn_v3&utm_relevant_index=1)
```java
public class CustomView extends LinearLayout {

    private LayoutCustomBinding binding;

    public CustomView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0, 0);
    }

    public CustomView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
        binding = LayoutCustomBinding.inflate(LayoutInflater.from(context), this, true);

        TypedArray a = context.getTheme().obtainStyledAttributes(attrs, R.styleable.CustomView, defStyleAttr, defStyleRes);
        
        // 設定內容
        text = a.getString(R.styleable.CustomView_label);
        // 需判斷getString()是否為空，如果沒做判斷，若未設置該屬性會有問題
        if (!TextUtils.isEmpty(text)) {
            binding.tv.setText(text);
        }
        // 設定大小
        float size = a.getDimension(R.styleable.CustomView_labelSize, 0);
        if (size > 0) {
            binding.tv.setTextSize(TypedValue.COMPLEX_UNIT_PX, size);
        }
        // 設定顏色
        int color = a.getColor(R.styleable.CustomView_labelColor, Color.BLACK);
        binding.tv.setTextColor(color);

        a.recycle();
    }
}
```

layout_custom.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</LinearLayout>
```

values/attrs.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CustomView">
        <attr name="label" format="string" />
        <attr name="labelSize" format="dimension" />
        <attr name="labelColor" format="color" />
    </declare-styleable>
</resources>
```

# 底線背景

不額外新建View來分隔項目，而是用drawable畫底線實現。

參考<https://stackoverflow.com/questions/9915793/shape-drawable-as-background-a-line-at-the-bottom>
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:left="-1dp"
        android:right="-1dp"
        android:top="-1dp">
        <shape>
            <stroke
                android:width="1dp"
                android:color="@color/black" />
        </shape>
    </item>
</layer-list>
```

# 隱藏鍵盤

```java
public static void hideKeyboard(View view) {
    InputMethodManager imm = (InputMethodManager) view.getContext()
            .getSystemService(Context.INPUT_METHOD_SERVICE);
    imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
}
```

# ProgressDialog改用ProgressBar

設定clickable為true，避免點擊到底下的物件；設定elevation為2dp，置於所有物件的上層。

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/mask"
    android:clickable="true"
    android:elevation="2dp"
    android:focusable="true">

    <ProgressBar
        android:id="@+id/progress_bar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/tv_message"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/loading"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/progress_bar" />

</androidx.constraintlayout.widget.ConstraintLayout>
```
參考
1. <https://stackoverflow.com/questions/36918219/how-to-disable-user-interaction-while-progressbar-is-visible-in-android>
2. <https://stackoverflow.com/questions/44351354/android-constraintlayout-put-one-view-on-top-of-another-view>

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