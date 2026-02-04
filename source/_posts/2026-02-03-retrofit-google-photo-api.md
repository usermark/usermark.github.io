---
title: '[Android] Retrofit應用 - Google Photos API'
date: 2026-02-03 10:37:40
tags:
- Android
- Retrofit
- Java
---
以Google Photos API的uploads為例。用Jetpack新提供的相片挑選工具，取得要上送的照片URI後，轉換成InputStream上送至服務。
<!--more-->
```java
public interface PhotoAPI {
    @Headers({
            "Content-type: application/octet-stream",
            "X-Goog-Upload-Protocol: raw"
    })
    @POST("uploads")
    Call<ResponseBody> uploadMediaItem(@Header("X-Goog-Upload-Content-Type") String mimeType, @Body RequestBody file);
}

public class MainActivity extends AppCompatActivity {
    private ActivityResultLauncher<PickVisualMediaRequest> pickMultipleMedia =
            registerForActivityResult(new ActivityResultContracts.PickMultipleVisualMedia(50), uris -> {
                if (uris.isEmpty()) {
                    Log.d("Mike", "No media selected");
                    return;
                }
                Log.d("Mike", "Number of items selected: " + uris.size());
                for (Uri uri : uris) {
                    ContentResolver contentResolver = getContentResolver();
                    try (Cursor cursor = contentResolver.query(uri, null, null, null, null)) {
                        if (cursor.moveToFirst()) {
                            String displayName = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.MediaColumns.DISPLAY_NAME));
                            uploadMediaItem(displayName, contentResolver.getType(uri), contentResolver.openInputStream(uri));
                        }
                    } catch (Exception e) {
                        Log.e("Mike", e.getMessage(), e);
                    }
                }
            });

    public void onUploadClick() {
        pickMultipleMedia.launch(new PickVisualMediaRequest.Builder()
                .setMediaType(ActivityResultContracts.PickVisualMedia.ImageAndVideo.INSTANCE)
                .build());
    }

    public void uploadMediaItem(String fileName, String mimeType, InputStream in) {
        OkHttpClient.Builder httpBuilder = new OkHttpClient.Builder()
                .addInterceptor(new TokenInterceptor());
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://photoslibrary.googleapis.com/v1/")
                .client(httpBuilder.build())
                .addConverterFactory(GsonConverterFactory.create())
                .build();

        RequestBody body = new RequestBody() {
            @Nullable
            @Override
            public MediaType contentType() {
                return MediaType.parse(mimeType);
            }

            @Override
            public void writeTo(BufferedSink sink) throws IOException {
                try (Source source = Okio.source(in)) {
                    sink.writeAll(source);
                }
            }
        };
        
        retrofit.create(PhotoAPI.class).uploadMediaItem(mimeType, body)
                .enqueue(new Callback<ResponseBody>() {
                    @Override
                    public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
                        
                    }

                    @Override
                    public void onFailure(Call<ResponseBody> call, Throwable t) {

                    }
                });
    }
}
```

**參考資料**
1. https://developers.google.com/photos/library/guides/upload-media?hl=en#uploading-bytes
2. https://developer.android.com/training/data-storage/shared/photopicker?hl=zh-tw