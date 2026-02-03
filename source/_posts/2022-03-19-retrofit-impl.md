---
layout: article
title: '[Android] Retrofit應用'
date: 2022-03-19 16:24
tags:
- Android
- Retrofit
---
紀錄開發上常遇到的問題，避免重複踩坑。
<!--more-->
# 紀錄API請求和回應

可使用寫好的套件 HttpLoggingInterceptor，參考<https://github.com/square/okhttp/tree/master/okhttp-logging-interceptor>

或自行實作。
```java
public class LogInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        String bodyString = bodyToString(request);
        Log.d("Mike", String.format("[%X] Request to %s\n%s",
                chain.hashCode(), request.url(), bodyString));

        Response response;
        try {
            response = chain.proceed(request);
        } catch (SocketTimeoutException e) {
            Log.d("Mike", String.format("[%X] Response timeout for %s",
                    chain.hashCode(), request.url()));
            throw e;
        }
        ResponseBody responseBody = response.body();
        String responseBodyString = responseBody.string();
        Log.d("Mike", String.format("[%X] Response to %s(%d)\n%s",
                chain.hashCode(), response.request().url(), response.code(), responseBodyString));

        return response.newBuilder().body(
                ResponseBody.create(responseBody.contentType(), responseBodyString.getBytes())).build();
    }

    private String bodyToString(Request request) {
        try {
            Request copy = request.newBuilder().build();
            Buffer buffer = new Buffer();
            if (copy.body() != null) {
                copy.body().writeTo(buffer);
                return buffer.readUtf8();
            }
            return "";
        } catch (IOException e) {
            return "";
        }
    }
}
```

若要印出headers，可加上這段。

注意：不要使用response.headers().names()跑for each處理，例如回傳的Set-Cookie有兩個，但該寫法只會印出一個。
```java
Headers headers = response.headers();
for (int i = 0; i < headers.size(); i++) {
    Log.d("Mike", headers.name(i) + ": " + headers.value(i));
}
```

# 請求加上token認證

例如使用Google API需要獲得token後，才有權限使用。

```java
public class TokenInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request().newBuilder()
                .header("Authorization", "Bearer " + token)
                .build();
        return chain.proceed(request);
    }
}
```

# 保持連線和Cookie

參考<https://stackoverflow.com/questions/36706795/how-to-keep-session-using-retrofit-okhttpclient>
```groovy
implementation 'com.squareup.okhttp3:okhttp-urlconnection:4.9.3'
```

```java
OkHttpClient.Builder httpBuilder = new OkHttpClient.Builder()
        .cookieJar(new JavaNetCookieJar(new CookieManager()))
        .build();
```

# 修改FieldMap

使用情境是有多個API都要傳同個欄位和值，例如UUID或時間戳記。

使用下列方式可對所有API都加上該值；但若是遇到只有部分需要，則不適用。建議改用Builder類別包住Map操作，然後將常用的欄位寫成public的method。

參考<https://stackoverflow.com/questions/56515769/okhttp3-interceptor-add-fields-to-request-body>
```java
public class FieldMapInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        if ("POST".equals(request.method()) && request.body() instanceof FormBody) {
            FormBody.Builder bodyBuilder = new FormBody.Builder();
            FormBody formBody = (FormBody) request.body();
            for (int i = 0; i < formBody.size(); i++) {
                bodyBuilder.addEncoded(formBody.encodedName(i), formBody.encodedValue(i));
            }
            bodyBuilder.add("key", "value");
            request = request.newBuilder().post(bodyBuilder.build()).build();
        }
        return chain.proceed(request);
    }
}
```

# 上送檔案(body為raw)

以Google Photos API的uploads為例，參考 [\[Android\] Retrofit應用 - Google Photos API](/2026/02/03/retrofit-google-photo-api)

# 上送檔案(body為form-data)

以工作上遇到的日誌上送為例。

```java
public interface CustomAPI {
    @POST("Upload")
    Call<ResponseBody> uploadLog(@Body RequestBody body);
}
```

```java
public void uploadLog(String date, File zipFile) {
    Retrofit retrofit = new Retrofit.Builder()
            .baseUrl("https://xxx/")
            .addConverterFactory(GsonConverterFactory.create())
            .build();

    final RequestBody body = new MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart("date", date)
            .addFormDataPart("log", zipFile.getName(), RequestBody.create(null, zipFile))
            .build();

    retrofit.create(CustomAPI.class).uploadLog(body)
            .enqueue(new Callback<ResponseBody>() {
                @Override
                public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
                    
                }

                @Override
                public void onFailure(Call<ResponseBody> call, Throwable t) {

                }
            });
}
```

# 串接Google API遇到java.lang.IllegalArgumentException: Malformed URL

以Google Photos API的mediaItems:batchCreate為例。因其使用gRPC轉碼語法，若直接使用會有問題，endpoint開頭加上./即可。

```java
public interface PhotoAPI {
    @POST("./mediaItems:batchCreate")
    Call<BatchCreateMediaItemsResponse> batchCreateMediaItems(@Body BatchCreateMediaItemsRequest request);
}
```

**參考資料**
1. https://developers.google.com/photos/library/reference/rest/v1/mediaItems/batchCreate?hl=zh-tw
2. https://stackoverflow.com/questions/54406788/retrofit-how-to-use-colon-in-url