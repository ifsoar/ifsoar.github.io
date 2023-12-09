#### 1、添加依赖

```gradle
    //ExpPlayer播放器
    //https://developer.android.com/guide/topics/media/media3/exoplayer
    //https://exoplayer.dev/
    implementation 'com.google.android.exoplayer:exoplayer:2.18.1'
```

如果播放的网络地址需要使用地址，还需要添加如下依赖

```gradle
    //okhttp3 dataSource扩展      
    // https://github.com/google/ExoPlayer/tree/release-v2/extensions/okhttp
    implementation 'com.google.android.exoplayer:extension-okhttp:2.18.1'
```

#### 2、播放音频

```java
    private ExoPlayer exoPlayer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_play);
        exoPlayer = new ExoPlayer.Builder(this).build();

                MediaItem mediaItem = MediaItem.fromUri("https://video-weaver.tyo05.hls.ttvnw.net/v1/playlist/CpwEYG06sVQd28zt220scPgcXmgL1XgnFMdjJONNd2DH8NHEWAtUUrI38pM5k7Fozt2Os94v_UtJFSGtk0wguy4jeP4lq1sCFzc2bE2pi63-4MX4jgAD7keGt4_9aWW3LJLFzjGHEDzNxPHrhLcfoUmNHCB869E9RdDqXvd4JozAOFxQqMX6n0mWSKllnTPgV_Q7vCZOp8maU4do_AAG_nX1ubhvoTiaFEIfbA6JQWISUVxCtJW-wVLBA1NYnswOKvjzhOA7gND9AMWpsKuIc4JtzgV7nX3BTgTx4cyNxp511e39a6r6VH89a0kOm8FDu2Sf-u0pImkZIp1ckGGRK9xL2Sws8pcEcXh3kIyGdmeM5_YW-RAMNgbqUZyj4UG_M8sIqoC6GDUiskkTU1771S3jd6cOksEytx26XwMH4hMltuIcrfRl6oTMRcMQcHRxTRg6fOc9ctt8xMAtFgGVVxxlz5ufve8ydnn6FEKLeUIGLFkzhfdO-UsRSg_QBE8GWnSSvJvNk73tz-n78t5fIETIzXoTv5hdzD9pj5OR4EXIie94ApNP1KZJNwdNJHyOmbXIbETRf0oQV5V8G6_UtIMrxIsUW9Twsweu8ybOt1LKF77DWHJejsRB6uzoxyo-EaVTZwaXsaznh0D_IvOlF33mqPhpu8B-HTLPMEv9YzsUaB1MZ1W6jmRBARPMUwEKlP42E7jLqQguxXT-cSRJGgyVvUSPybEISWX16KwgASoJdXMtd2VzdC0yMKoE.m3u8");
       exoPlayer.setMediaItem(mediaItem);
        exoPlayer.prepare();
    }

    @Override
    protected void onResume() {
        super.onResume();
        exoPlayer.play();
    }

    @Override
    protected void onPause() {
        super.onPause();
        exoPlayer.pause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        exoPlayer.release();
    }
```

#### 3、播放视频

##### 3.1 在布局文件添加 StyledPlayerView 控件

```xml

<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.play.PlayActivity">

    <com.google.android.exoplayer2.ui.StyledPlayerView
        android:layout_margin="2dp"
        android:id="@+id/playerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:use_controller="false" />
androidx.constraintlayout.widget.ConstraintLayout>
```

use_controller 可以隐藏默认的控制按钮

##### 3.2 在java代码添加引用

```java
exoPlayer.setMediaItem(mediaItem);
playerView.setPlayer(exoPlayer);
exoPlayer.prepare();
```

#### 4、使用代理

替换setMediaItem方法，构造代理对象，使用http代理

```java
    MediaItem mediaItem = 
        MediaItem.fromUri("https://video-weaver.tyo05.hls.zdC0yMKoE.m3u8");

    Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress("10.10.10.130", 7890));
    Authenticator authenticator = new Authenticator() {
        @Nullable
        @Override
        public Request authenticate(
            @Nullable Route route, @NonNull Response response) throws IOException {
                //代理不检验身份
                return response.request();
            }
    };
    OkHttpClient client = new OkHttpClient.Builder()
                .proxy(proxy)
                .proxyAuthenticator(authenticator)
                .build();
    DataSource.Factory dataSourceFactory = 
        new OkHttpDataSource.Factory(new Call.Factory() {
            @NonNull
            @Override
            public Call newCall(@NonNull Request request) {
                return client.newCall(request);
            }
        });
    MediaSource mediaSource = new HlsMediaSource.Factory(dataSourceFactory)
        .createMediaSource(mediaItem);
    exoPlayer.setMediaSource(mediaSource);
```

​