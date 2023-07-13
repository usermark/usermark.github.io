---
layout: article
title: '[Android] AES加解密在Android和Java結果不一致'
date: 2023-07-10 13:46:28
tags:
- Android
- AES
---
與server通訊要求用AES加密，但Android加密後，送到server卻報錯。
<!--more-->

先提供加密的實作方式
```java
public class AESUtil {
    private static final String KEY_ALGORITHM = "AES";
    private static final String DEFAULT_CIPHER_ALGORITHM = "AES/ECB/PKCS5Padding";

    public static String encrypt(String data, String key) {
        try {
            Cipher cipher = Cipher.getInstance(DEFAULT_CIPHER_ALGORITHM);
            byte[] byteContent = data.getBytes(StandardCharsets.UTF_8);
            cipher.init(Cipher.ENCRYPT_MODE, getSecretKey(key));
            byte[] result = cipher.doFinal(byteContent);
            return new String(Base64.encodeBase64(result), StandardCharsets.UTF_8);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    private static SecretKeySpec getSecretKey(String key) {
        try {
            KeyGenerator kg = KeyGenerator.getInstance(KEY_ALGORITHM);
            SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
            secureRandom.setSeed(key.getBytes(StandardCharsets.UTF_8));
            System.out.println("secureRandom.getProvider()=" + secureRandom.getProvider());
            kg.init(128, secureRandom);
            SecretKey secretKey = kg.generateKey();
            return new SecretKeySpec(secretKey.getEncoded(), KEY_ALGORITHM);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
```

同樣是對Hello world字串作加密，每次執行，在Java會得到同樣的結果StB5b2z9qPuulJ7rt2V1hQ==，但在Android跑結果都不同。
```java
AESUtil.encrypt("Hello world", "C6DFE59F58353AEF73549370CD98F0A9");
```

原因是SecureRandom不一致，Java的Provider是使用SUN version 17，Android則是AndroidOpenSSL version 1.0。

彙整資訊為下表：

|           | Java         | Android                  |
| --------- | ------------ | ------------------------ |
|AES加密結果| 固定          |每次跑都不同              |
|Provider   |SUN version 17|AndroidOpenSSL version 1.0|

解法是將Java實作的[SecureRandom]((https://github.com/frohoff/jdk8u-jdk/blob/master/src/share/classes/java/security/SecureRandom.java))搬進Android，首先找到source code

複製SecureRandom進專案後，重新命名為[SunSecureRandom](https://github.com/usermark/AesSample/blob/master/app/src/main/java/com/github/usermark/aessample/sun/SunSecureRandom.java)，將原先彈性取用SHA演算法的寫法改為自行實作的SHA。
```java
    private void init(byte[] seed) {
        digest = new SHA();
//        try {
//            /*
//             * Use the local SUN implementation to avoid native
//             * performance overhead.
//             */
//            digest = MessageDigest.getInstance("SHA", "SUN");
//        } catch (NoSuchProviderException | NoSuchAlgorithmException e) {
//            // Fallback to any available.
//            try {
//                digest = MessageDigest.getInstance("SHA");
//            } catch (NoSuchAlgorithmException exc) {
//                throw new InternalError(
//                    "internal error: SHA-1 not available.", exc);
//            }
//        }

        if (seed != null) {
           engineSetSeed(seed);
        }
    }
```

[SHA](https://github.com/usermark/AesSample/blob/master/app/src/main/java/com/github/usermark/aessample/sun/SHA.java)新增3個method，讓SunSecureRandom可以使用。
```java
void update(byte[] input) {
    engineUpdate(input, 0, input.length);
}

byte[] digest() {
    return engineDigest();
}

byte[] digest(byte[] input) {
    update(input);
    return digest();
}
```

原本[Unsafe](https://github.com/usermark/AesSample/blob/master/app/src/main/java/com/github/usermark/aessample/sun/Unsafe.java)都是呼叫底層native，改成自行實作的。
```java
public final class Unsafe {

    private Unsafe() {}

    private static final Unsafe theUnsafe = new Unsafe();

    public static Unsafe getUnsafe() {
        return theUnsafe;
    }

    public int getInt(byte[] o, long offset) {
        ByteBuffer bf = ByteBuffer.wrap(o, (int) offset, 4);
        return bf.getInt();
    }

    public void putInt(byte[] o, long offset, int x) {
        ByteBuffer bf = ByteBuffer.wrap(o);
        bf.putInt((int) offset, x);
    }
}
```

改寫取得SecretKey的方式
```java
private static SecretKeySpec getSecretKey(String key) {
    try {
        KeyGenerator kg = KeyGenerator.getInstance(KEY_ALGORITHM);
        kg.init(128);
        SecretKey secretKey = kg.generateKey();
        byte[] keyBytes = secretKey.getEncoded();
        SunSecureRandom sunRandom = new SunSecureRandom();
        sunRandom.engineSetSeed(key.getBytes(StandardCharsets.UTF_8));
        sunRandom.engineNextBytes(keyBytes);
        return new SecretKeySpec(keyBytes, KEY_ALGORITHM);
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
}
```

最終不管是在Java或Android上跑都可以得到同樣的加密結果，完整實作範例於此 <https://github.com/usermark/AesSample>