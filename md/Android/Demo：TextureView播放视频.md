```java
package com.soar.videoview;
import android.animation.ObjectAnimator;
import android.app.Activity;
import android.graphics.SurfaceTexture;
import android.media.AudioManager;
import android.media.MediaPlayer;
import android.net.Uri;
import android.os.Bundle;
import android.os.Environment;
import android.util.Log;
import android.view.Surface;
import android.view.TextureView;
import android.view.View;
import android.widget.Button;
import java.io.File;
public class MainActivity extends Activity implements View.OnClickListener {
	private TextureView textureView_main_video;
	private MySurfaceTextureListener mySurfaceTextureListener;
	private Surface surface;
	private MediaPlayer mediaPlayer;
	private Button start, pause, stop, zoomUp, zoomDown;
	private Boolean isPlay = false;
	@Override
	    protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		initView();
		addListener();
		initData();
	}
	private void initView() {
		textureView_main_video = (TextureView) findViewById(R.id.textureView_main_video);
		mySurfaceTextureListener = new MySurfaceTextureListener();
		textureView_main_video.setSurfaceTextureListener(mySurfaceTextureListener);
		start = (Button) findViewById(R.id.btn_main_play);
		pause = (Button) findViewById(R.id.btn_main_pause);
		stop = (Button) findViewById(R.id.btn_main_stop);
		zoomUp = (Button) findViewById(R.id.btn_main_zoomUp);
		zoomDown = (Button) findViewById(R.id.btn_main_zoomDown);
	}
	private void addListener() {
		start.setOnClickListener(this);
		pause.setOnClickListener(this);
		stop.setOnClickListener(this);
		zoomUp.setOnClickListener(this);
		zoomDown.setOnClickListener(this);
	}
	private void initData() {
	}
	@Override
	    public void onClick(View view) {
		switch (view.getId()) {
			case R.id.btn_main_play: {
				new Thread(new PlayThread()).start();
			}
			break;
			case R.id.btn_main_pause: {
				if (isPlay) {
					mediaPlayer.pause();
				} else {
					mediaPlayer.start();
				}
				isPlay = !isPlay;
			}
			break;
			case R.id.btn_main_stop: {
				mediaPlayer.stop();
			}
			break;
			case R.id.btn_main_zoomUp: {
				ObjectAnimator animator = ObjectAnimator.ofint(textureView_main_video,"width",320);
				animator.setDuration(500).start();
				//                ObjectAnimator.ofFloat(textureView_main_video, "scaleY", 0.9F);
			}
			break;
			case R.id.btn_main_zoomDown: {
			}
			break;
		}
	}
	private class MySurfaceTextureListener implements TextureView.SurfaceTextureListener {
		@Override
		        public void onSurfaceTextureAvailable(SurfaceTexture surfaceTexture, int i, int i1) {
			surface = new Surface(surfaceTexture);
			//            new Thread(new PlayThread()).start();
		}
		@Override
		        public void onSurfaceTextureSizeChanged(SurfaceTexture surfaceTexture, int i, int i1) {
		}
		@Override
		        public Boolean onSurfaceTextureDestroyed(SurfaceTexture surfaceTexture) {
			surfaceTexture = null;
			surface = null;
			return true;
		}
		@Override
		        public void onSurfaceTextureUpdated(SurfaceTexture surfaceTexture) {
		}
	}
	private class PlayThread implements Runnable {
		@Override
		        public void run() {
			try {
				File file = new File(Environment.getExternalStorageDirectory().getAbsolutePath() + File.separator + "video.mp4");
				Log.d("soar_", ">>>" + file.getAbsolutePath());
				if (!file.exists() || !file.isFile()) {
					return;
				}
				Uri uri = Uri.parse(file.getAbsolutePath());
				mediaPlayer = MediaPlayer.create(MainActivity.this, uri);
				//                mediaPlayer.setDataSource(file.getAbsolutePath());
				mediaPlayer.setSurface(surface);
				mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
				mediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
					@Override
					                    public void onPrepared(MediaPlayer mediaPlayer) {
						mediaPlayer.start();
						isPlay = true;
					}
				}
				);
				//                mediaPlayer.prepare();
			}
			catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
}
```