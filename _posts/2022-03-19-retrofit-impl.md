---
layout: post
title: '[Android] Retrofit應用'
date: 2022-03-19 16:24
categories: 
---
紀錄開發上常遇到的問題，避免重複踩坑。

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