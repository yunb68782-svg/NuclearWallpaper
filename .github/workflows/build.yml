name: Build Nuclear Live Wallpaper APK

on:
  push:
    branches: [ "main", "master" ]
  workflow_dispatch:

jobs:
  build-apk:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Setup Gradle 8.7
        uses: gradle/actions/setup-gradle@v3
        with:
          gradle-version: '8.7'

      - name: Generate Full Android Project Architecture
        run: |
          mkdir -p app/src/main/java/com/i7ruhua/nuclearwallpaper
          mkdir -p app/src/main/res/xml
          mkdir -p app/src/main/res/values

          cat << 'EOF' > settings.gradle.kts
          pluginManagement {
              repositories {
                  google()
                  mavenCentral()
                  gradlePluginPortal()
              }
          }
          dependencyResolutionManagement {
              repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
              repositories {
                  google()
                  mavenCentral()
              }
          }
          rootProject.name = "NuclearWallpaper"
          include(":app")
          EOF

          cat << 'EOF' > build.gradle.kts
          plugins {
              id("com.android.application") version "8.4.2" apply false
              id("org.jetbrains.kotlin.android") version "1.9.24" apply false
          }
          EOF

          cat << 'EOF' > gradle.properties
          org.gradle.jvmargs=-Xmx4096m -Dfile.encoding=UTF-8
          android.useAndroidX=true
          android.nonTransitiveRClass=true
          EOF

          cat << 'EOF' > app/build.gradle.kts
          plugins {
              id("com.android.application")
              id("org.jetbrains.kotlin.android")
          }

          android {
              namespace = "com.i7ruhua.nuclearwallpaper"
              compileSdk = 34

              defaultConfig {
                  applicationId = "com.i7ruhua.nuclearwallpaper"
                  minSdk = 26
                  targetSdk = 34
                  versionCode = 1
                  versionName = "1.0-ARGON"
              }

              buildTypes {
                  release {
                      isMinifyEnabled = false
                  }
              }
              compileOptions {
                  sourceCompatibility = JavaVersion.VERSION_17
                  targetCompatibility = JavaVersion.VERSION_17
              }
              kotlinOptions {
                  jvmTarget = "17"
              }
          }

          dependencies {
              implementation("androidx.core:core-ktx:1.13.1")
              implementation("androidx.appcompat:appcompat:1.7.0")
              implementation("com.google.android.material:material:1.12.0")
          }
          EOF

          cat << 'EOF' > app/src/main/res/values/strings.xml
          <resources>
              <string name="app_name">核芯壁纸引擎</string>
              <string name="wallpaper_desc">4 Argon 驱动 · OpenGL ES 2.0 实时物理视差与聚变流体引擎</string>
          </resources>
          EOF

          cat << 'EOF' > app/src/main/res/xml/wallpaper.xml
          <?xml version="1.0" encoding="utf-8"?>
          <wallpaper xmlns:android="http://schemas.android.com/apk/res/android"
              android:description="@string/wallpaper_desc"
              android:settingsActivity="com.i7ruhua.nuclearwallpaper.MainActivity" />
          EOF

          cat << 'EOF' > app/src/main/AndroidManifest.xml
          <?xml version="1.0" encoding="utf-8"?>
          <manifest xmlns:android="http://schemas.android.com/apk/res/android">

              <uses-feature
                  android:name="android.software.live_wallpaper"
                  android:required="true" />
              <uses-feature
                  android:glEsVersion="0x00020000"
                  android:required="true" />

              <application
                  android:allowBackup="true"
                  android:label="@string/app_name"
                  android:supportsRtl="true"
                  android:theme="@style/Theme.AppCompat.DayNight.NoActionBar">

                  <activity
                      android:name=".MainActivity"
                      android:exported="true">
                      <intent-filter>
                          <action android:name="android.intent.action.MAIN" />
                          <category android:name="android.intent.category.LAUNCHER" />
                      </intent-filter>
                  </activity>

                  <service
                      android:name=".NuclearWallpaperService"
                      android:enabled="true"
                      android:exported="true"
                      android:label="@string/app_name"
                      android:permission="android.permission.BIND_WALLPAPER">
                      <intent-filter>
                          <action android:name="android.service.wallpaper.WallpaperService" />
                      </intent-filter>
                      <meta-data
                          android:name="android.service.wallpaper"
                          android:resource="@xml/wallpaper" />
                  </service>
              </application>
          </manifest>
          EOF

          cat << 'EOF' > app/src/main/java/com/i7ruhua/nuclearwallpaper/MainActivity.kt
          package com.i7ruhua.nuclearwallpaper

          import android.app.WallpaperManager
          import android.content.ComponentName
          import android.content.Context
          import android.content.Intent
          import android.graphics.Color
          import android.graphics.Typeface
          import android.graphics.drawable.GradientDrawable
          import android.net.Uri
          import android.os.Bundle
          import android.view.Gravity
          import android.view.View
          import android.widget.Button
          import android.widget.LinearLayout
          import android.widget.ScrollView
          import android.widget.SeekBar
          import android.widget.Switch
          import android.widget.TextView
          import android.widget.Toast
          import androidx.activity.result.contract.ActivityResultContracts
          import androidx.appcompat.app.AppCompatActivity
          import java.io.File
          import java.io.FileOutputStream

          class MainActivity : AppCompatActivity() {

              private val prefs by lazy { getSharedPreferences("nuclear_prefs", Context.MODE_PRIVATE) }
              private lateinit var statusLabel: TextView
              private val modeButtons = mutableListOf<Button>()

              private val pickImageLauncher = registerForActivityResult(ActivityResultContracts.GetContent()) { uri: Uri? ->
                  if (uri != null) {
                      if (copyUriToInternalFile(uri, "custom_image.jpg")) {
                          val ver = prefs.getInt("media_version", 0) + 1
                          prefs.edit()
                              .putInt("mode", 2)
                              .putInt("media_version", ver)
                              .apply()
                          updateModeButtonsUI(2)
                          statusLabel.text = "当前状态：已载入自定义 3D 视差图片壁纸"
                          Toast.makeText(this, "图片已注入引擎沙盒！", Toast.LENGTH_SHORT).show()
                      }
                  }
              }

              private val pickVideoLauncher = registerForActivityResult(ActivityResultContracts.GetContent()) { uri: Uri? ->
                  if (uri != null) {
                      if (copyUriToInternalFile(uri, "custom_video.mp4")) {
                          val ver = prefs.getInt("media_version", 0) + 1
                          prefs.edit()
                              .putInt("mode", 3)
                              .putInt("media_version", ver)
                              .apply()
                          updateModeButtonsUI(3)
                          statusLabel.text = "当前状态：已载入超清视频流壁纸"
                          Toast.makeText(this, "视频已注入引擎沙盒！", Toast.LENGTH_SHORT).show()
                      }
                  }
              }

              override fun onCreate(savedInstanceState: Bundle?) {
                  super.onCreate(savedInstanceState)
                  window.statusBarColor = Color.parseColor("#0B0F19")
                  window.navigationBarColor = Color.parseColor("#0B0F19")

                  val rootScroll = ScrollView(this).apply {
                      setBackgroundColor(Color.parseColor("#0B0F19"))
                      isFillViewport = true
                  }

                  val container = LinearLayout(this).apply {
                      orientation = LinearLayout.VERTICAL
                      setPadding(dp(20), dp(28), dp(20), dp(40))
                  }

                  val titleView = TextView(this).apply {
                      text = "☢️ 核芯壁纸引擎 · 4 ARGON"
                      textSize = 24f
                      setTextColor(Color.parseColor("#00F5D4"))
                      typeface = Typeface.DEFAULT_BOLD
                  }
                  val subTitleView = TextView(this).apply {
                      text = "OpenGL ES 2.0 硬件着色器 | 陀螺仪 3D 景深 | OES 视频流"
                      textSize = 13f
                      setTextColor(Color.parseColor("#8E9AAF"))
                      setPadding(0, dp(6), 0, dp(20))
                  }
                  container.addView(titleView)
                  container.addView(subTitleView)

                  // 激活壁纸主按钮
                  val applyBtn = Button(this).apply {
                      text = "🚀 一键激活 / 设为系统桌面壁纸"
                      textSize = 16f
                      setTextColor(Color.parseColor("#0B0F19"))
                      typeface = Typeface.DEFAULT_BOLD
                      background = roundedBg("#00F5D4", dp(14))
                      setPadding(dp(16), dp(14), dp(16), dp(14))
                      setOnClickListener {
                          try {
                              val intent = Intent(WallpaperManager.ACTION_CHANGE_LIVE_WALLPAPER).apply {
                                  putExtra(
                                      WallpaperManager.EXTRA_LIVE_WALLPAPER_COMPONENT,
                                      ComponentName(this@MainActivity, NuclearWallpaperService::class.java)
                                  )
                              }
                              startActivity(intent)
                          } catch (e: Exception) {
                              val fallback = Intent(WallpaperManager.ACTION_LIVE_WALLPAPER_CHOOSER)
                              startActivity(fallback)
                          }
                      }
                  }
                  container.addView(applyBtn)

                  statusLabel = TextView(this).apply {
                      text = getModeDescription(prefs.getInt("mode", 0))
                      textSize = 13f
                      setTextColor(Color.parseColor("#FEE440"))
                      setPadding(dp(4), dp(14), dp(4), dp(10))
                  }
                  container.addView(statusLabel)

                  // 核心模式选择卡片
                  val modeCard = createSectionCard("🎨 渲染核心模式切换（实时生效）")
                  val modes = listOf(
                      0 to "☢️ 模式 1：手搓核弹 · 聚变等离子核心 (GPU 实时演算)",
                      1 to "🕳️ 模式 2：赛博黑洞 · 引力视界星云 (GPU 实时演算)",
                      2 to "🖼️ 模式 3：全息 3D 陀螺仪视差图片 (点击选相册图片)",
                      3 to "🎬 模式 4：超清视频流 + 触摸水波纹 (点击选相册视频)"
                  )
                  val currentMode = prefs.getInt("mode", 0)
                  for ((mId, label) in modes) {
                      val btn = Button(this).apply {
                          text = label
                          textSize = 14f
                          isAllCaps = false
                          gravity = Gravity.START or Gravity.CENTER_VERTICAL
                          setPadding(dp(16), dp(12), dp(16), dp(12))
                          val lp = LinearLayout.LayoutParams(
                              LinearLayout.LayoutParams.MATCH_PARENT,
                              LinearLayout.LayoutParams.WRAP_CONTENT
                          ).apply { topMargin = dp(10) }
                          layoutParams = lp
                          setOnClickListener {
                              when (mId) {
                                  2 -> pickImageLauncher.launch("image/*")
                                  3 -> pickVideoLauncher.launch("video/*")
                                  else -> {
                                      prefs.edit().putInt("mode", mId).apply()
                                      updateModeButtonsUI(mId)
                                      statusLabel.text = getModeDescription(mId)
                                  }
                              }
                          }
                      }
                      modeButtons.add(btn)
                      modeCard.addView(btn)
                  }
                  updateModeButtonsUI(currentMode)
                  container.addView(modeCard)

                  // 物理与视差参数调节卡片
                  val paramCard = createSectionCard("⚙️ 陀螺仪 3D 视差 & 帧率引擎参数")

                  // FPS 档位
                  val currentFps = prefs.getInt("fps", 60)
                  val fpsLabel = TextView(this).apply {
                      text = "目标刷新率上限：$currentFps FPS"
                      setTextColor(Color.WHITE)
                      textSize = 14f
                      setPadding(0, dp(8), 0, dp(4))
                  }
                  paramCard.addView(fpsLabel)

                  val fpsRow = LinearLayout(this).apply {
                      orientation = LinearLayout.HORIZONTAL
                  }
                  listOf(30, 60, 90, 120).forEach { fpsVal ->
                      val fBtn = Button(this).apply {
                          text = "${fpsVal}Hz"
                          textSize = 13f
                          setTextColor(Color.WHITE)
                          background = roundedBg("#252F42", dp(8))
                          val lp = LinearLayout.LayoutParams(0, dp(40), 1f).apply {
                              marginEnd = dp(6)
                          }
                          layoutParams = lp
                          setOnClickListener {
                              prefs.edit().putInt("fps", fpsVal).apply()
                              fpsLabel.text = "目标刷新率上限：$fpsVal FPS"
                              Toast.makeText(this@MainActivity, "已切换至 $fpsVal FPS 档位", Toast.LENGTH_SHORT).show()
                          }
                      }
                      fpsRow.addView(fBtn)
                  }
                  paramCard.addView(fpsRow)

                  // 陀螺仪视差强度
                  val initParallax = prefs.getInt("parallax", 65)
                  val parallaxText = TextView(this).apply {
                      text = "陀螺仪 3D 景深偏移强度：$initParallax%"
                      setTextColor(Color.WHITE)
                      textSize = 14f
                      setPadding(0, dp(16), 0, dp(4))
                  }
                  val parallaxSeek = SeekBar(this).apply {
                      max = 100
                      progress = initParallax
                      setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
                          override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
                              parallaxText.text = "陀螺仪 3D 景深偏移强度：$progress%"
                              prefs.edit().putInt("parallax", progress).apply()
                          }
                          override fun onStartTrackingTouch(seekBar: SeekBar?) {}
                          override fun onStopTrackingTouch(seekBar: SeekBar?) {}
                      })
                  }
                  paramCard.addView(parallaxText)
                  paramCard.addView(parallaxSeek)

                  // 能量流速 / 动画速度
                  val initSpeed = prefs.getInt("speed", 50)
                  val speedText = TextView(this).apply {
                      text = "着色器能量演化速率：$initSpeed%"
                      setTextColor(Color.WHITE)
                      textSize = 14f
                      setPadding(0, dp(14), 0, dp(4))
                  }
                  val speedSeek = SeekBar(this).apply {
                      max = 100
                      progress = initSpeed
                      setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
                          override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
                              speedText.text = "着色器能量演化速率：$progress%"
                              prefs.edit().putInt("speed", progress).apply()
                          }
                          override fun onStartTrackingTouch(seekBar: SeekBar?) {}
                          override fun onStopTrackingTouch(seekBar: SeekBar?) {}
                      })
                  }
                  paramCard.addView(speedText)
                  paramCard.addView(speedSeek)
                  container.addView(paramCard)

                  // 交互与特效开关卡片
                  val switchCard = createSectionCard("✨ 交互水波纹与多维图层开关")

                  switchCard.addView(createSwitchRow("手指划过桌面触发折射水波纹", "ripple", true))
                  switchCard.addView(createSwitchRow("叠加 3D 悬浮赛博星尘粒子层", "particles", true))
                  switchCard.addView(createSwitchRow("视频壁纸模式开启原声播放", "video_audio", false))

                  container.addView(switchCard)

                  rootScroll.addView(container)
                  setContentView(rootScroll)
              }

              private fun updateModeButtonsUI(selectedMode: Int) {
                  modeButtons.forEachIndexed { index, btn ->
                      if (index == selectedMode) {
                          btn.background = roundedBg("#7B2CBF", dp(10))
                          btn.setTextColor(Color.parseColor("#00F5D4"))
                      } else {
                          btn.background = roundedBg("#1E2638", dp(10))
                          btn.setTextColor(Color.parseColor("#D9E2EC"))
                      }
                  }
              }

              private fun getModeDescription(mode: Int): String {
                  return when (mode) {
                      0 -> "当前状态：☢️ 手搓核弹 · 聚变等离子核心运行中"
                      1 -> "当前状态：🕳️ 赛博黑洞 · 引力透镜视界运行中"
                      2 -> "当前状态：🖼️ 全息 3D 视差自定义图片模式"
                      3 -> "当前状态：🎬 超清视频流 + 实时 Shader 后处理模式"
                      else -> "当前状态：就绪"
                  }
              }

              private fun createSectionCard(title: String): LinearLayout {
                  val card = LinearLayout(this).apply {
                      orientation = LinearLayout.VERTICAL
                      background = roundedBg("#151C2C", dp(14))
                      setPadding(dp(16), dp(16), dp(16), dp(16))
                      val lp = LinearLayout.LayoutParams(
                          LinearLayout.LayoutParams.MATCH_PARENT,
                          LinearLayout.LayoutParams.WRAP_CONTENT
                      ).apply { topMargin = dp(14) }
                      layoutParams = lp
                  }
                  val header = TextView(this).apply {
                      text = title
                      textSize = 15f
                      typeface = Typeface.DEFAULT_BOLD
                      setTextColor(Color.parseColor("#00BBF9"))
                      setPadding(0, 0, 0, dp(6))
                  }
                  card.addView(header)
                  return card
              }

              private fun createSwitchRow(label: String, key: String, defVal: Boolean): View {
                  val sw = Switch(this).apply {
                      text = label
                      textSize = 14f
                      setTextColor(Color.WHITE)
                      isChecked = prefs.getBoolean(key, defVal)
                      setPadding(0, dp(12), 0, dp(12))
                      setOnCheckedChangeListener { _, isChecked ->
                          prefs.edit().putBoolean(key, isChecked).apply()
                      }
                  }
                  return sw
              }

              private fun copyUriToInternalFile(uri: Uri, targetName: String): Boolean {
                  return try {
                      contentResolver.openInputStream(uri)?.use { input ->
                          val outFile = File(filesDir, targetName)
                          FileOutputStream(outFile).use { output ->
                              input.copyTo(output)
                          }
                      }
                      true
                  } catch (e: Exception) {
                      Toast.makeText(this, "读取文件失败: ${e.message}", Toast.LENGTH_LONG).show()
                      false
                  }
              }

              private fun roundedBg(hexColor: String, radiusPx: Int): GradientDrawable {
                  return GradientDrawable().apply {
                      shape = GradientDrawable.RECTANGLE
                      cornerRadius = radiusPx.toFloat()
                      setColor(Color.parseColor(hexColor))
                  }
              }

              private fun dp(value: Int): Int {
                  return (value * resources.displayMetrics.density).toInt()
              }
          }
          EOF

          cat << 'EOF' > app/src/main/java/com/i7ruhua/nuclearwallpaper/NuclearWallpaperService.kt
          package com.i7ruhua.nuclearwallpaper

          import android.content.Context
          import android.content.SharedPreferences
          import android.graphics.BitmapFactory
          import android.graphics.SurfaceTexture
          import android.hardware.Sensor
          import android.hardware.SensorEvent
          import android.hardware.SensorEventListener
          import android.hardware.SensorManager
          import android.media.MediaPlayer
          import android.opengl.EGL14
          import android.opengl.EGLConfig
          import android.opengl.EGLContext
          import android.opengl.EGLDisplay
          import android.opengl.EGLSurface
          import android.opengl.GLES11Ext
          import android.opengl.GLES20
          import android.opengl.GLUtils
          import android.service.wallpaper.WallpaperService
          import android.view.MotionEvent
          import android.view.Surface
          import android.view.SurfaceHolder
          import java.io.File
          import java.nio.ByteBuffer
          import java.nio.ByteOrder
          import java.nio.FloatBuffer
          import java.util.concurrent.atomic.AtomicBoolean

          class NuclearWallpaperService : WallpaperService() {

              override fun onCreateEngine(): Engine {
                  return NuclearEngine()
              }

              inner class NuclearEngine : Engine(), SensorEventListener,
                  SharedPreferences.OnSharedPreferenceChangeListener {

                  private val prefs by lazy {
                      applicationContext.getSharedPreferences("nuclear_prefs", Context.MODE_PRIVATE)
                  }
                  private val sensorManager by lazy {
                      applicationContext.getSystemService(Context.SENSOR_SERVICE) as SensorManager
                  }

                  private var renderThread: GLRenderThread? = null
                  private var isEngineVisible = false

                  @Volatile private var targetTiltX = 0f
                  @Volatile private var targetTiltY = 0f
                  @Volatile private var touchX = 0.5f
                  @Volatile private var touchY = 0.5f
                  @Volatile private var lastTouchTimestamp = -10f

                  override fun onCreate(surfaceHolder: SurfaceHolder?) {
                      super.onCreate(surfaceHolder)
                      setTouchEventsEnabled(true)
                      prefs.registerOnSharedPreferenceChangeListener(this)
                  }

                  override fun onDestroy() {
                      prefs.unregisterOnSharedPreferenceChangeListener(this)
                      unregisterSensors()
                      stopRenderThread()
                      super.onDestroy()
                  }

                  override fun onVisibilityChanged(visible: Boolean) {
                      super.onVisibilityChanged(visible)
                      isEngineVisible = visible
                      if (visible) {
                          registerSensors()
                          renderThread?.setPaused(false)
                      } else {
                          unregisterSensors()
                          renderThread?.setPaused(true)
                      }
                  }

                  override fun onSurfaceCreated(holder: SurfaceHolder) {
                      super.onSurfaceCreated(holder)
                      startRenderThread(holder)
                  }

                  override fun onSurfaceChanged(holder: SurfaceHolder, format: Int, width: Int, height: Int) {
                      super.onSurfaceChanged(holder, format, width, height)
                      renderThread?.updateSurfaceSize(width, height)
                  }

                  override fun onSurfaceDestroyed(holder: SurfaceHolder) {
                      stopRenderThread()
                      super.onSurfaceDestroyed(holder)
                  }

                  override fun onTouchEvent(event: MotionEvent) {
                      if (event.action == MotionEvent.ACTION_DOWN || event.action == MotionEvent.ACTION_MOVE) {
                          val w = renderThread?.screenWidth ?: 1080
                          val h = renderThread?.screenHeight ?: 2400
                          if (w > 0 && h > 0) {
                              touchX = (event.x / w.toFloat()).coerceIn(0f, 1f)
                              touchY = (1f - event.y / h.toFloat()).coerceIn(0f, 1f)
                              lastTouchTimestamp = renderThread?.currentElapsedSec ?: 0f
                          }
                      }
                      super.onTouchEvent(event)
                  }

                  override fun onSharedPreferenceChanged(sharedPreferences: SharedPreferences?, key: String?) {
                      renderThread?.notifyPrefsChanged()
                  }

                  private fun registerSensors() {
                      val accel = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)
                      if (accel != null) {
                          sensorManager.registerListener(this, accel, SensorManager.SENSOR_DELAY_GAME)
                      }
                  }

                  private fun unregisterSensors() {
                      sensorManager.unregisterListener(this)
                  }

                  override fun onSensorChanged(event: SensorEvent) {
                      if (event.sensor.type == Sensor.TYPE_ACCELEROMETER) {
                          // 归一化重力倾角至 -1.0 .. 1.0
                          targetTiltX = (-event.values[0] / 9.81f).coerceIn(-1.2f, 1.2f)
                          targetTiltY = (-event.values[1] / 9.81f).coerceIn(-1.2f, 1.2f)
                      }
                  }

                  override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {}

                  private fun startRenderThread(holder: SurfaceHolder) {
                      stopRenderThread()
                      renderThread = GLRenderThread(holder).apply {
                          start()
                          setPaused(!isEngineVisible)
                      }
                  }

                  private fun stopRenderThread() {
                      renderThread?.let { thread ->
                          thread.requestStop()
                          try {
                              thread.join(1200)
                          } catch (_: InterruptedException) {}
                      }
                      renderThread = null
                  }

                  private inner class GLRenderThread(private val holder: SurfaceHolder) : Thread("NuclearGLThread") {
                      @Volatile var screenWidth = 1080
                      @Volatile var screenHeight = 2400
                      @Volatile var currentElapsedSec = 0f

                      private val running = AtomicBoolean(true)
                      private val paused = AtomicBoolean(false)
                      private val prefsDirty = AtomicBoolean(true)
                      private val videoFrameAvailable = AtomicBoolean(false)

                      private var eglDisplay: EGLDisplay = EGL14.EGL_NO_DISPLAY
                      private var eglContext: EGLContext = EGL14.EGL_NO_CONTEXT
                      private var eglSurface: EGLSurface = EGL14.EGL_NO_SURFACE

                      private var progStandard = 0
                      private var progVideo = 0
                      private var imageTexId = 0
                      private var videoOesTexId = 0

                      private var surfaceTexture: SurfaceTexture? = null
                      private var videoSurface: Surface? = null
                      private var mediaPlayer: MediaPlayer? = null

                      private var activeMode = 0
                      private var targetFps = 60
                      private var parallaxStrength = 0.65f
                      private var speedFactor = 1.0f
                      private var rippleEnabled = true
                      private var particlesEnabled = true
                      private var videoAudioEnabled = false
                      private var loadedMediaVersion = -1
                      private var mediaAspect = 0.5625f
                      private var hasValidImage = false
                      private var hasValidVideo = false

                      private var smoothTiltX = 0f
                      private var smoothTiltY = 0f
                      private val stMatrix = FloatArray(16)

                      private val quadVertices = floatArrayOf(
                          -1f, -1f,
                           1f, -1f,
                          -1f,  1f,
                           1f,  1f
                      )
                      private val vertexBuffer: FloatBuffer = ByteBuffer
                          .allocateDirect(quadVertices.size * 4)
                          .order(ByteOrder.nativeOrder())
                          .asFloatBuffer()
                          .apply {
                              put(quadVertices)
                              position(0)
                          }

                      fun updateSurfaceSize(w: Int, h: Int) {
                          if (w > 0 && h > 0) {
                              screenWidth = w
                              screenHeight = h
                          }
                      }

                      fun setPaused(p: Boolean) {
                          paused.set(p)
                          synchronized(this) {
                              (this as Object).notifyAll()
                          }
                      }

                      fun notifyPrefsChanged() {
                          prefsDirty.set(true)
                          synchronized(this) {
                              (this as Object).notifyAll()
                          }
                      }

                      fun requestStop() {
                          running.set(false)
                          synchronized(this) {
                              (this as Object).notifyAll()
                          }
                      }

                      override fun run() {
                          if (!initEGL()) return
                          initGLResources()

                          val startTimeNs = System.nanoTime()

                          while (running.get()) {
                              synchronized(this) {
                                  while (paused.get() && running.get()) {
                                      pauseVideoSafe()
                                      try {
                                          (this as Object).wait()
                                      } catch (_: InterruptedException) {}
                                  }
                              }
                              if (!running.get()) break

                              if (prefsDirty.compareAndSet(true, false)) {
                                  syncPreferencesAndMedia()
                              }

                              if (activeMode == 3 && hasValidVideo) {
                                  resumeVideoSafe()
                              } else {
                                  pauseVideoSafe()
                              }

                              val frameStartMs = System.currentTimeMillis()
                              currentElapsedSec = (System.nanoTime() - startTimeNs) / 1_000_000_000f

                              smoothTiltX += (targetTiltX - smoothTiltX) * 0.08f
                              smoothTiltY += (targetTiltY - smoothTiltY) * 0.08f

                              drawFrame()
                              EGL14.eglSwapBuffers(eglDisplay, eglSurface)

                              val frameBudgetMs = (1000L / targetFps.coerceIn(24, 120))
                              val spentMs = System.currentTimeMillis() - frameStartMs
                              val sleepMs = frameBudgetMs - spentMs
                              if (sleepMs > 1L) {
                                  try {
                                      sleep(sleepMs)
                                  } catch (_: InterruptedException) {}
                              }
                          }

                          releaseAll()
                      }

                      private fun syncPreferencesAndMedia() {
                          val prevMode = activeMode
                          val newVersion = prefs.getInt("media_version", 0)
                          activeMode = prefs.getInt("mode", 0)
                          targetFps = prefs.getInt("fps", 60)
                          parallaxStrength = prefs.getInt("parallax", 65) / 100f
                          speedFactor = 0.25f + (prefs.getInt("speed", 50) / 100f) * 1.75f
                          rippleEnabled = prefs.getBoolean("ripple", true)
                          particlesEnabled = prefs.getBoolean("particles", true)
                          videoAudioEnabled = prefs.getBoolean("video_audio", false)

                          val needReloadMedia = (newVersion != loadedMediaVersion) || (prevMode != activeMode)
                          loadedMediaVersion = newVersion

                          if (activeMode == 2 && (needReloadMedia || !hasValidImage)) {
                              loadCustomImageTexture()
                          } else if (activeMode == 3 && (needReloadMedia || !hasValidVideo)) {
                              setupCustomVideoPlayer()
                          }

                          mediaPlayer?.let { mp ->
                              try {
                                  val vol = if (videoAudioEnabled) 1f else 0f
                                  mp.setVolume(vol, vol)
                              } catch (_: Exception) {}
                          }
                      }

                      private fun loadCustomImageTexture() {
                          val imgFile = File(applicationContext.filesDir, "custom_image.jpg")
                          if (!imgFile.exists()) {
                              hasValidImage = false
                              return
                          }
                          try {
                              val opts = BitmapFactory.Options().apply {
                                  inPreferredConfig = android.graphics.Bitmap.Config.ARGB_8888
                              }
                              val bmp = BitmapFactory.decodeFile(imgFile.absolutePath, opts)
                              if (bmp != null && bmp.width > 0 && bmp.height > 0) {
                                  mediaAspect = bmp.width.toFloat() / bmp.height.toFloat()
                                  GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, imageTexId)
                                  GLUtils.texImage2D(GLES20.GL_TEXTURE_2D, 0, bmp, 0)
                                  bmp.recycle()
                                  hasValidImage = true
                              } else {
                                  hasValidImage = false
                              }
                          } catch (_: Exception) {
                              hasValidImage = false
                          }
                      }

                      private fun setupCustomVideoPlayer() {
                          val vidFile = File(applicationContext.filesDir, "custom_video.mp4")
                          if (!vidFile.exists()) {
                              hasValidVideo = false
                              return
                          }
                          try {
                              releaseMediaPlayerOnly()
                              if (surfaceTexture == null) {
                                  surfaceTexture = SurfaceTexture(videoOesTexId).apply {
                                      setOnFrameAvailableListener {
                                          videoFrameAvailable.set(true)
                                      }
                                  }
                                  videoSurface = Surface(surfaceTexture)
                              }
                              mediaPlayer = MediaPlayer().apply {
                                  setDataSource(vidFile.absolutePath)
                                  setSurface(videoSurface)
                                  isLooping = true
                                  val vol = if (videoAudioEnabled) 1f else 0f
                                  setVolume(vol, vol)
                                  setOnVideoSizeChangedListener { _, w, h ->
                                      if (w > 0 && h > 0) {
                                          mediaAspect = w.toFloat() / h.toFloat()
                                      }
                                  }
                                  prepare()
                                  start()
                              }
                              hasValidVideo = true
                          } catch (_: Exception) {
                              hasValidVideo = false
                          }
                      }

                      private fun pauseVideoSafe() {
                          try {
                              mediaPlayer?.let {
                                  if (it.isPlaying) it.pause()
                              }
                          } catch (_: Exception) {}
                      }

                      private fun resumeVideoSafe() {
                          try {
                              mediaPlayer?.let {
                                  if (!it.isPlaying) it.start()
                              }
                          } catch (_: Exception) {}
                      }

                      private fun drawFrame() {
                          GLES20.glViewport(0, 0, screenWidth, screenHeight)
                          GLES20.glClearColor(0.03f, 0.04f, 0.08f, 1f)
                          GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT)

                          val useVideoShader = (activeMode == 3 && hasValidVideo)
                          if (useVideoShader && videoFrameAvailable.compareAndSet(true, false)) {
                              try {
                                  surfaceTexture?.updateTexImage()
                                  surfaceTexture?.getTransformMatrix(stMatrix)
                              } catch (_: Exception) {}
                          }

                          val prog = if (useVideoShader) progVideo else progStandard
                          GLES20.glUseProgram(prog)

                          val posLoc = GLES20.glGetAttribLocation(prog, "aPosition")
                          GLES20.glEnableVertexAttribArray(posLoc)
                          GLES20.glVertexAttribPointer(posLoc, 2, GLES20.GL_FLOAT, false, 0, vertexBuffer)

                          val screenAspect = screenWidth.toFloat() / screenHeight.coerceAtLeast(1).toFloat()
                          GLES20.glUniform1f(GLES20.glGetUniformLocation(prog, "uTime"), currentElapsedSec * speedFactor)
                          GLES20.glUniform1f(GLES20.glGetUniformLocation(prog, "uRawTime"), currentElapsedSec)
                          GLES20.glUniform2f(GLES20.glGetUniformLocation(prog, "uResolution"), screenWidth.toFloat(), screenHeight.toFloat())
                          GLES20.glUniform1f(GLES20.glGetUniformLocation(prog, "uScreenAspect"), screenAspect)
                          GLES20.glUniform1f(GLES20.glGetUniformLocation(prog, "uMediaAspect"), mediaAspect)
                          GLES20.glUniform2f(
                              GLES20.glGetUniformLocation(prog, "uTilt"),
                              smoothTiltX * parallaxStrength,
                              smoothTiltY * parallaxStrength
                          )
                          GLES20.glUniform2f(GLES20.glGetUniformLocation(prog, "uTouch"), touchX, touchY)
                          GLES20.glUniform1f(
                              GLES20.glGetUniformLocation(prog, "uTouchAge"),
                              if (rippleEnabled) (currentElapsedSec - lastTouchTimestamp) else 999f
                          )
                          GLES20.glUniform1f(
                              GLES20.glGetUniformLocation(prog, "uParticles"),
                              if (particlesEnabled) 1f else 0f
                          )

                          if (useVideoShader) {
                              GLES20.glActiveTexture(GLES20.GL_TEXTURE0)
                              GLES20.glBindTexture(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, videoOesTexId)
                              GLES20.glUniform1i(GLES20.glGetUniformLocation(prog, "uVideoTex"), 0)
                          } else {
                              val effectiveMode = if (activeMode == 2 && hasValidImage) 2 else if (activeMode == 1) 1 else 0
                              GLES20.glUniform1i(GLES20.glGetUniformLocation(prog, "uMode"), effectiveMode)
                              if (effectiveMode == 2) {
                                  GLES20.glActiveTexture(GLES20.GL_TEXTURE0)
                                  GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, imageTexId)
                                  GLES20.glUniform1i(GLES20.glGetUniformLocation(prog, "uImageTex"), 0)
                              }
                          }

                          GLES20.glDrawArrays(GLES20.GL_TRIANGLE_STRIP, 0, 4)
                          GLES20.glDisableVertexAttribArray(posLoc)
                      }

                      private fun initEGL(): Boolean {
                          eglDisplay = EGL14.eglGetDisplay(EGL14.EGL_DEFAULT_DISPLAY)
                          if (eglDisplay == EGL14.EGL_NO_DISPLAY) return false
                          val version = IntArray(2)
                          if (!EGL14.eglInitialize(eglDisplay, version, 0, version, 1)) return false

                          val attribList = intArrayOf(
                              EGL14.EGL_RENDERABLE_TYPE, EGL14.EGL_OPENGL_ES2_BIT,
                              EGL14.EGL_RED_SIZE, 8,
                              EGL14.EGL_GREEN_SIZE, 8,
                              EGL14.EGL_BLUE_SIZE, 8,
                              EGL14.EGL_ALPHA_SIZE, 8,
                              EGL14.EGL_NONE
                          )
                          val configs = arrayOfNulls<EGLConfig>(1)
                          val numConfigs = IntArray(1)
                          if (!EGL14.eglChooseConfig(eglDisplay, attribList, 0, configs, 0, 1, numConfigs, 0)) {
                              return false
                          }
                          val config = configs[0] ?: return false

                          val ctxAttribs = intArrayOf(EGL14.EGL_CONTEXT_CLIENT_VERSION, 2, EGL14.EGL_NONE)
                          eglContext = EGL14.eglCreateContext(eglDisplay, config, EGL14.EGL_NO_CONTEXT, ctxAttribs, 0)
                          if (eglContext == EGL14.EGL_NO_CONTEXT) return false

                          val surfAttribs = intArrayOf(EGL14.EGL_NONE)
                          eglSurface = EGL14.eglCreateWindowSurface(eglDisplay, config, holder, surfAttribs, 0)
                          if (eglSurface == EGL14.EGL_NO_SURFACE) return false

                          return EGL14.eglMakeCurrent(eglDisplay, eglSurface, eglSurface, eglContext)
                      }

                      private fun initGLResources() {
                          progStandard = buildProgram(VERTEX_SHADER, FRAGMENT_SHADER_STANDARD)
                          progVideo = buildProgram(VERTEX_SHADER, FRAGMENT_SHADER_VIDEO)

                          val texIds = IntArray(2)
                          GLES20.glGenTextures(2, texIds, 0)
                          imageTexId = texIds[0]
                          videoOesTexId = texIds[1]

                          GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, imageTexId)
                          GLES20.glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MIN_FILTER, GLES20.GL_LINEAR)
                          GLES20.glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MAG_FILTER, GLES20.GL_LINEAR)
                          GLES20.glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_S, GLES20.GL_CLAMP_TO_EDGE)
                          GLES20.glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_T, GLES20.GL_CLAMP_TO_EDGE)

                          GLES20.glBindTexture(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, videoOesTexId)
                          GLES20.glTexParameteri(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, GLES20.GL_TEXTURE_MIN_FILTER, GLES20.GL_LINEAR)
                          GLES20.glTexParameteri(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, GLES20.GL_TEXTURE_MAG_FILTER, GLES20.GL_LINEAR)
                          GLES20.glTexParameteri(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, GLES20.GL_TEXTURE_WRAP_S, GLES20.GL_CLAMP_TO_EDGE)
                          GLES20.glTexParameteri(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, GLES20.GL_TEXTURE_WRAP_T, GLES20.GL_CLAMP_TO_EDGE)
                      }

                      private fun buildProgram(vSrc: String, fSrc: String): Int {
                          val vs = compileShader(GLES20.GL_VERTEX_SHADER, vSrc)
                          val fs = compileShader(GLES20.GL_FRAGMENT_SHADER, fSrc)
                          val prog = GLES20.glCreateProgram()
                          GLES20.glAttachShader(prog, vs)
                          GLES20.glAttachShader(prog, fs)
                          GLES20.glLinkProgram(prog)
                          return prog
                      }

                      private fun compileShader(type: Int, src: String): Int {
                          val s = GLES20.glCreateShader(type)
                          GLES20.glShaderSource(s, src)
                          GLES20.glCompileShader(s)
                          return s
                      }

                      private fun releaseMediaPlayerOnly() {
                          try {
                              mediaPlayer?.stop()
                          } catch (_: Exception) {}
                          try {
                              mediaPlayer?.release()
                          } catch (_: Exception) {}
                          mediaPlayer = null
                      }

                      private fun releaseAll() {
                          releaseMediaPlayerOnly()
                          try {
                              videoSurface?.release()
                              surfaceTexture?.release()
                          } catch (_: Exception) {}
                          videoSurface = null
                          surfaceTexture = null

                          if (eglDisplay != EGL14.EGL_NO_DISPLAY) {
                              EGL14.eglMakeCurrent(
                                  eglDisplay,
                                  EGL14.EGL_NO_SURFACE,
                                  EGL14.EGL_NO_SURFACE,
                                  EGL14.EGL_NO_CONTEXT
                              )
                              if (eglSurface != EGL14.EGL_NO_SURFACE) {
                                  EGL14.eglDestroySurface(eglDisplay, eglSurface)
                              }
                              if (eglContext != EGL14.EGL_NO_CONTEXT) {
                                  EGL14.eglDestroyContext(eglDisplay, eglContext)
                              }
                              EGL14.eglTerminate(eglDisplay)
                          }
                      }
                  }
              }

              companion object {
                  private const val VERTEX_SHADER = """
                      attribute vec2 aPosition;
                      varying vec2 vUv;
                      void main() {
                          vUv = aPosition * 0.5 + 0.5;
                          gl_Position = vec4(aPosition, 0.0, 1.0);
                      }
                  """

                  // 涵盖：模式0(手搓核弹·聚变核心)、模式1(赛博黑洞视界)、模式2(自定义3D视差图片+色散+粒子+水波纹)
                  private const val FRAGMENT_SHADER_STANDARD = """
                      precision highp float;
                      varying vec2 vUv;

                      uniform float uTime;
                      uniform float uRawTime;
                      uniform vec2 uResolution;
                      uniform float uScreenAspect;
                      uniform float uMediaAspect;
                      uniform vec2 uTilt;
                      uniform vec2 uTouch;
                      uniform float uTouchAge;
                      uniform float uParticles;
                      uniform int uMode;
                      uniform sampler2D uImageTex;

                      float hash21(vec2 p) {
                          p = fract(p * vec2(234.34, 435.345));
                          p += dot(p, p + 34.23);
                          return fract(p.x * p.y);
                      }

                      vec3 starLayer(vec2 uv, float scale, float seed) {
                          vec2 gv = fract(uv * scale) - 0.5;
                          vec2 id = floor(uv * scale) + seed;
                          float n = hash21(id);
                          float size = fract(n * 345.32) * 0.08;
                          float d = length(gv - vec2(n - 0.5, fract(n * 12.7) - 0.5) * 0.6);
                          float spark = smoothstep(size, 0.0, d);
                          vec3 col = mix(vec3(0.0, 0.96, 0.83), vec3(0.98, 0.25, 0.65), fract(n * 19.3));
                          return col * spark * (0.5 + 0.5 * sin(uRawTime * 3.0 + n * 6.28));
                      }

                      void main() {
                          vec2 uv = vUv;
                          vec2 aspectVec = vec2(uScreenAspect, 1.0);

                          // 1. 计算触摸折射水波纹与能量光环
                          vec2 rippleOffset = vec2(0.0);
                          float rippleGlow = 0.0;
                          if (uTouchAge >= 0.0 && uTouchAge < 3.0) {
                              float dist = length((uv - uTouch) * aspectVec);
                              float wave = sin(dist * 45.0 - uTouchAge * 14.0);
                              float env = exp(-dist * 5.5) * exp(-uTouchAge * 1.8);
                              rippleOffset = normalize((uv - uTouch) + vec2(0.0001)) * wave * env * 0.028;
                              rippleGlow = smoothstep(0.04, 0.0, abs(dist - uTouchAge * 0.32)) * exp(-uTouchAge * 1.5);
                          }

                          vec3 finalColor = vec3(0.0);

                          if (uMode == 2) {
                              // 自定义图片 + 居中裁剪 + 陀螺仪3D视差 + 边缘色散
                              vec2 scale = vec2(1.0);
                              if (uScreenAspect > uMediaAspect) {
                                  scale.y = uMediaAspect / uScreenAspect;
                              } else {
                                  scale.x = uScreenAspect / uMediaAspect;
                              }
                              // 预留 10% 边缘余量给陀螺仪视差平移
                              vec2 imgUv = (uv - 0.5) * scale * 0.88 + 0.5 + uTilt * 0.055 + rippleOffset;
                              imgUv = clamp(imgUv, 0.001, 0.999);
                              vec2 flippedUv = vec2(imgUv.x, 1.0 - imgUv.y);

                              float chroma = length(uTilt) * 0.004 + rippleGlow * 0.006;
                              float r = texture2D(uImageTex, flippedUv + vec2(chroma, 0.0)).r;
                              float g = texture2D(uImageTex, flippedUv).g;
                              float b = texture2D(uImageTex, flippedUv - vec2(chroma, 0.0)).b;
                              finalColor = vec3(r, g, b);
                          } else if (uMode == 1) {
                              // 模式1：赛博黑洞·引力透镜视界
                              vec2 p = (uv - 0.5) * aspectVec + uTilt * 0.14 + rippleOffset;
                              float r = length(p);
                              float a = atan(p.y, p.x);
                              float swirl = a + 1.2 / (r + 0.15) - uTime * 0.9;
                              float disk = sin(swirl * 5.0) * 0.5 + 0.5;
                              float ring = smoothstep(0.55, 0.18, r) * smoothstep(0.09, 0.19, r);
                              float photonRing = 0.015 / (abs(r - 0.15) + 0.008);
                              vec3 colA = vec3(0.48, 0.17, 0.95);
                              vec3 colB = vec3(0.0, 0.96, 0.83);
                              finalColor = mix(colA, colB, disk) * ring * 1.6 + vec3(0.95, 0.85, 1.0) * photonRing;
                              finalColor *= smoothstep(0.11, 0.14, r); // 事件视界黑洞中心
                          } else {
                              // 模式0：手搓核弹·聚变等离子反应堆
                              vec2 p = (uv - 0.5) * aspectVec * 2.2 + uTilt * 0.25 + rippleOffset * 2.0;
                              vec2 z = p;
                              float acc = 0.0;
                              for (int i = 0; i < 5; i++) {
                                  float fi = float(i) + 1.0;
                                  z += vec2(
                                      sin(z.y * fi * 1.6 + uTime * 1.1),
                                      cos(z.x * fi * 1.6 - uTime * 0.9)
                                  ) / fi;
                                  acc += 0.18 / (length(z - p * 0.35) + 0.12);
                              }
                              float coreDist = length(p);
                              float corePulse = 0.08 / (abs(coreDist - 0.42 - 0.06 * sin(uTime * 3.0)) + 0.04);
                              vec3 plasmaCol = vec3(
                                  0.5 + 0.5 * sin(acc * 2.2 + uTime),
                                  0.5 + 0.5 * sin(acc * 2.2 + 2.1),
                                  0.5 + 0.5 * sin(acc * 2.2 + 4.2)
                              );
                              finalColor = plasmaCol * acc * 0.48 + vec3(0.0, 0.98, 0.85) * corePulse;
                          }

                          // 2. 叠加多景深陀螺仪 3D 悬浮粒子层
                          if (uParticles > 0.5) {
                              vec2 pUv1 = uv * aspectVec + uTilt * 0.12 + vec2(0.0, uRawTime * 0.02);
                              vec2 pUv2 = uv * aspectVec + uTilt * 0.24 + vec2(uRawTime * 0.015, -uRawTime * 0.03);
                              finalColor += starLayer(pUv1, 7.0, 1.3) * 0.7;
                              finalColor += starLayer(pUv2, 12.0, 5.7) * 0.9;
                          }

                          // 3. 叠加触摸冲击波霓虹光环
                          finalColor += vec3(0.0, 0.96, 0.83) * rippleGlow * 0.95;

                          gl_FragColor = vec4(finalColor, 1.0);
                      }
                  """

                  // 模式3专用：OES 视频硬件纹理 + 陀螺仪3D视差 + 触摸折射水波纹 + 悬浮粒子
                  private const val FRAGMENT_SHADER_VIDEO = """
                      #extension GL_OES_EGL_image_external : require
                      precision highp float;
                      varying vec2 vUv;

                      uniform float uTime;
                      uniform float uRawTime;
                      uniform vec2 uResolution;
                      uniform float uScreenAspect;
                      uniform float uMediaAspect;
                      uniform vec2 uTilt;
                      uniform vec2 uTouch;
                      uniform float uTouchAge;
                      uniform float uParticles;
                      uniform samplerExternalOES uVideoTex;

                      float hash21(vec2 p) {
                          p = fract(p * vec2(234.34, 435.345));
                          p += dot(p, p + 34.23);
                          return fract(p.x * p.y);
                      }

                      vec3 starLayer(vec2 uv, float scale, float seed) {
                          vec2 gv = fract(uv * scale) - 0.5;
                          vec2 id = floor(uv * scale) + seed;
                          float n = hash21(id);
                          float size = fract(n * 345.32) * 0.07;
                          float d = length(gv - vec2(n - 0.5, fract(n * 12.7) - 0.5) * 0.6);
                          float spark = smoothstep(size, 0.0, d);
                          vec3 col = mix(vec3(0.0, 0.96, 0.83), vec3(0.98, 0.85, 0.25), fract(n * 19.3));
                          return col * spark * (0.5 + 0.5 * sin(uRawTime * 3.0 + n * 6.28));
                      }

                      void main() {
                          vec2 uv = vUv;
                          vec2 aspectVec = vec2(uScreenAspect, 1.0);

                          vec2 rippleOffset = vec2(0.0);
                          float rippleGlow = 0.0;
                          if (uTouchAge >= 0.0 && uTouchAge < 3.0) {
                              float dist = length((uv - uTouch) * aspectVec);
                              float wave = sin(dist * 45.0 - uTouchAge * 14.0);
                              float env = exp(-dist * 5.5) * exp(-uTouchAge * 1.8);
                              rippleOffset = normalize((uv - uTouch) + vec2(0.0001)) * wave * env * 0.025;
                              rippleGlow = smoothstep(0.04, 0.0, abs(dist - uTouchAge * 0.32)) * exp(-uTouchAge * 1.5);
                          }

                          vec2 scale = vec2(1.0);
                          if (uScreenAspect > uMediaAspect) {
                              scale.y = uMediaAspect / uScreenAspect;
                          } else {
                              scale.x = uScreenAspect / uMediaAspect;
                          }

                          vec2 vidUv = (uv - 0.5) * scale * 0.90 + 0.5 + uTilt * 0.045 + rippleOffset;
                          vidUv = clamp(vidUv, 0.001, 0.999);
                          vec2 flippedUv = vec2(vidUv.x, 1.0 - vidUv.y);

                          vec3 finalColor = texture2D(uVideoTex, flippedUv).rgb;

                          if (uParticles > 0.5) {
                              vec2 pUv = uv * aspectVec + uTilt * 0.18 + vec2(0.0, uRawTime * 0.02);
                              finalColor += starLayer(pUv, 9.0, 3.1) * 0.65;
                          }

                          finalColor += vec3(0.0, 0.96, 0.83) * rippleGlow * 0.85;
                          gl_FragColor = vec4(finalColor, 1.0);
                      }
                  """
              }
          }
          EOF

      - name: Compile Debug APK via Gradle
        run: gradle assembleDebug --no-daemon

      - name: Upload Compiled APK Artifact
        uses: actions/upload-artifact@v4
        with:
          name: NuclearWallpaper-Argon-APK
          path: app/build/outputs/apk/debug/app-debug.apk
