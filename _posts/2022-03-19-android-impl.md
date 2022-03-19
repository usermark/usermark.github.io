---
layout: post
title: '[Android] 應用'
date: 2022-03-19 17:00
categories: 
---
紀錄開發上常遇到的問題，避免重複踩坑。

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