# بناء ملف APK للتطبيق

سنقوم الآن بإنشاء هيكل مشروع Android كامل وبناء ملف APK جاهز للتثبيت على الهاتف المحمول.

## 1. إنشاء هيكل المشروع

أولاً، سنقوم بإنشاء هيكل مشروع Android باستخدام Gradle:

```
app/
├── build.gradle
├── src/
│   ├── main/
│   │   ├── AndroidManifest.xml
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── mangareader/
│   │   │           ├── App.kt
│   │   │           ├── MainActivity.kt
│   │   │           ├── data/
│   │   │           ├── di/
│   │   │           ├── domain/
│   │   │           ├── source/
│   │   │           ├── ui/
│   │   │           └── util/
│   │   └── res/
│   │       ├── drawable/
│   │       ├── layout/
│   │       ├── menu/
│   │       ├── navigation/
│   │       ├── values/
│   │       └── xml/
│   └── test/
build.gradle
gradle.properties
settings.gradle
```

## 2. إعداد ملفات Gradle

### build.gradle (المشروع)

```gradle
buildscript {
    ext {
        kotlin_version = '1.7.20'
        compose_version = '1.3.1'
        nav_version = '2.5.3'
        room_version = '2.5.0'
        lifecycle_version = '2.5.1'
        hilt_version = '2.44'
        retrofit_version = '2.9.0'
        okhttp_version = '4.10.0'
        glide_version = '4.14.2'
    }
    
    repositories {
        google()
        mavenCentral()
    }
    
    dependencies {
        classpath 'com.android.tools.build:gradle:7.3.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "com.google.dagger:hilt-android-gradle-plugin:$hilt_version"
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
        maven { url 'https://jitpack.io' }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

### build.gradle (التطبيق)

```gradle
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-kapt'
    id 'kotlin-parcelize'
    id 'dagger.hilt.android.plugin'
    id 'androidx.navigation.safeargs.kotlin'
}

android {
    compileSdkVersion 33
    
    defaultConfig {
        applicationId "com.mangareader.app"
        minSdkVersion 21
        targetSdkVersion 33
        versionCode 1
        versionName "1.0.0"
        
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        
        javaCompileOptions {
            annotationProcessorOptions {
                arguments += [
                    "room.schemaLocation": "$projectDir/schemas",
                    "room.incremental"   : "true",
                    "room.expandProjection": "true"
                ]
            }
        }
    }
    
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
        debug {
            applicationIdSuffix ".debug"
            debuggable true
        }
    }
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    
    kotlinOptions {
        jvmTarget = '1.8'
    }
    
    buildFeatures {
        viewBinding true
    }
    
    packagingOptions {
        resources {
            excludes += [
                'META-INF/LICENSE.txt',
                'META-INF/NOTICE.txt',
                'META-INF/LICENSE',
                'META-INF/NOTICE',
            ]
        }
    }
}

dependencies {
    // Kotlin
    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    implementation 'androidx.core:core-ktx:1.9.0'
    
    // UI
    implementation 'androidx.appcompat:appcompat:1.6.0'
    implementation 'com.google.android.material:material:1.8.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.recyclerview:recyclerview:1.2.1'
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'
    implementation 'androidx.viewpager2:viewpager2:1.0.0'
    
    // Navigation
    implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
    implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
    
    // Lifecycle
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version"
    
    // Room
    implementation "androidx.room:room-runtime:$room_version"
    implementation "androidx.room:room-ktx:$room_version"
    kapt "androidx.room:room-compiler:$room_version"
    
    // Hilt
    implementation "com.google.dagger:hilt-android:$hilt_version"
    kapt "com.google.dagger:hilt-compiler:$hilt_version"
    
    // Retrofit & OkHttp
    implementation "com.squareup.retrofit2:retrofit:$retrofit_version"
    implementation "com.squareup.retrofit2:converter-gson:$retrofit_version"
    implementation "com.squareup.okhttp3:okhttp:$okhttp_version"
    implementation "com.squareup.okhttp3:logging-interceptor:$okhttp_version"
    
    // Glide
    implementation "com.github.bumptech.glide:glide:$glide_version"
    kapt "com.github.bumptech.glide:compiler:$glide_version"
    
    // Coroutines
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4'
    
    // JSoup
    implementation 'org.jsoup:jsoup:1.15.3'
    
    // Preferences
    implementation 'androidx.preference:preference-ktx:1.2.0'
    
    // WorkManager
    implementation 'androidx.work:work-runtime-ktx:2.7.1'
    
    // Testing
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}
```

## 3. إنشاء ملف AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.mangareader.app">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" 
        android:maxSdkVersion="28" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" 
        android:maxSdkVersion="32" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />

    <application
        android:name=".App"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MangaReader"
        android:usesCleartextTraffic="true">
        
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:configChanges="orientation|screenSize|uiMode">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <activity
            android:name=".ui.reader.ReaderActivity"
            android:configChanges="orientation|screenSize|uiMode"
            android:theme="@style/Theme.MangaReader.Reader" />
            
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
        
        <service
            android:name=".data.download.DownloadService"
            android:foregroundServiceType="dataSync" />
            
    </application>

</manifest>
```

## 4. تنفيذ الفئات الرئيسية

### App.kt

```kotlin
package com.mangareader.app

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class App : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // تهيئة المكتبات
        initializeLibraries()
    }
    
    private fun initializeLibraries() {
        // تهيئة Glide
        // تهيئة WorkManager
        // تهيئة مكتبات أخرى
    }
}
```

### MainActivity.kt

```kotlin
package com.mangareader.app

import android.os.Bundle
import android.view.View
import androidx.appcompat.app.AppCompatActivity
import androidx.navigation.NavController
import androidx.navigation.fragment.NavHostFragment
import androidx.navigation.ui.setupWithNavController
import com.mangareader.app.databinding.ActivityMainBinding
import dagger.hilt.android.AndroidEntryPoint
import java.util.*

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    private lateinit var navController: NavController
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // تطبيق الاتجاه من اليمين إلى اليسار للغة العربية
        if (Locale.getDefault().language == "ar") {
            window.decorView.layoutDirection = View.LAYOUT_DIRECTION_RTL
        }
        
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // إعداد التنقل
        val navHostFragment = supportFragmentManager.findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        navController = navHostFragment.navController
        
        // ربط شريط التنقل السفلي بالتنقل
        binding.bottomNav.setupWithNavController(navController)
        
        // إخفاء شريط التنقل السفلي في بعض الشاشات
        navController.addOnDestinationChangedListener { _, destination, _ ->
            when (destination.id) {
                R.id.libraryFragment, R.id.browseFragment, R.id.downloadsFragment, R.id.settingsFragment ->
                    binding.bottomNav.visibility = View.VISIBLE
                else -> binding.bottomNav.visibility = View.GONE
            }
        }
    }
    
    override fun onSupportNavigateUp(): Boolean {
        return navController.navigateUp() || super.onSupportNavigateUp()
    }
}
```

## 5. إنشاء ملفات الموارد

### strings.xml

```xml
<resources>
    <string name="app_name">قارئ المانجا</string>
    <string name="library">المكتبة</string>
    <string name="browse">تصفح</string>
    <string name="downloads">التنزيلات</string>
    <string name="settings">الإعدادات</string>
    <string name="search">بحث</string>
    <string name="search_hint">ابحث عن المانجا...</string>
    <string name="arabic">العربية</string>
    <string name="english">الإنجليزية</string>
    <string name="japanese">اليابانية</string>
    <string name="queue">قائمة الانتظار</string>
    <string name="completed">مكتملة</string>
    <string name="appearance">المظهر</string>
    <string name="dark_theme">الوضع الداكن</string>
    <string name="rtl_layout">تخطيط من اليمين إلى اليسار</string>
    <string name="reader">القارئ</string>
    <string name="keep_screen_on">إبقاء الشاشة مضاءة</string>
    <string name="reading_direction">اتجاه القراءة</string>
    <string name="right_to_left">من اليمين إلى اليسار</string>
    <string name="left_to_right">من اليسار إلى اليمين</string>
    <string name="vertical">عمودي</string>
    <string name="download_wifi_only">التنزيل عبر Wi-Fi فقط</string>
    <string name="download_location">موقع التنزيل</string>
    <string name="change">تغيير</string>
    <string name="advanced">متقدم</string>
    <string name="clear_cache">مسح ذاكرة التخزين المؤقت</string>
    <string name="add_repository">إضافة مستودع</string>
    <string name="about">حول</string>
    <string name="github">GitHub</string>
    <string name="manga_cover">غلاف المانجا</string>
    <string name="add_to_library">إضافة إلى المكتبة</string>
    <string name="read">قراءة</string>
    <string name="description">الوصف</string>
    <string name="chapters">الفصول</string>
    <string name="download_all">تنزيل الكل</string>
    <string name="download">تنزيل</string>
    <string name="source_icon">أيقونة المصدر</string>
    <string name="page_info">%1$d / %2$d</string>
    <string name="unknown_author">مؤلف غير معروف</string>
</resources>
```

### colors.xml

```xml
<resources>
    <color name="primary">#6200EE</color>
    <color name="primary_variant">#3700B3</color>
    <color name="secondary">#03DAC6</color>
    <color name="secondary_variant">#018786</color>
    
    <color name="primary_dark">#BB86FC</color>
    <color name="primary_variant_dark">#6200EA</color>
    <color name="secondary_dark">#03DAC6</color>
    <color name="secondary_variant_dark">#018786</color>
    
    <color name="background_light">#FFFFFF</color>
    <color name="surface_light">#FFFFFF</color>
    <color name="text_primary_light">#000000</color>
    <color name="text_secondary_light">#757575</color>
    
    <color name="background_dark">#121212</color>
    <color name="surface_dark">#1E1E1E</color>
    <color name="text_primary_dark">#FFFFFF</color>
    <color name="text_secondary_dark">#BBBBBB</color>
    
    <color name="black">#000000</color>
    <color name="white">#FFFFFF</color>
</resources>
```

### themes.xml

```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- الوضع الفاتح -->
    <style name="Theme.MangaReader" parent="Theme.MaterialComponents.Light.NoActionBar">
        <item name="colorPrimary">@color/primary</item>
        <item name="colorPrimaryVariant">@color/primary_variant</item>
        <item name="colorSecondary">@color/secondary</item>
        <item name="colorSecondaryVariant">@color/secondary_variant</item>
        <item name="android:colorBackground">@color/background_light</item>
        <item name="colorSurface">@color/surface_light</item>
        <item name="colorOnPrimary">@color/white</item>
        <item name="colorOnSecondary">@color/black</item>
        <item name="colorOnBackground">@color/text_primary_light</item>
        <item name="colorOnSurface">@color/text_primary_light</item>
        <item name="android:statusBarColor">@color/primary_variant</item>
        <item name="android:windowLightStatusBar">false</item>
        <item name="fontFamily">@font/cairo</item>
    </style>
    
    <!-- وضع القارئ -->
    <style name="Theme.MangaReader.Reader" parent="Theme.MangaReader">
        <item name="android:statusBarColor">@color/black</item>
        <item name="android:navigationBarColor">@color/black</item>
        <item name="android:windowLightStatusBar">false</item>
        <item name="android:windowLightNavigationBar" tools:targetApi="o_mr1">false</item>
        <item name="android:windowBackground">@color/black</item>
    </style>
</resources>
```

### themes.xml (night)

```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- الوضع الداكن -->
    <style name="Theme.MangaReader" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        <item name="colorPrimary">@color/primary_dark</item>
        <item name="colorPrimaryVariant">@color/primary_variant_dark</item>
        <item name="colorSecondary">@color/secondary_dark</item>
        <item name="colorSecondaryVariant">@color/secondary_variant_dark</item>
        <item name="android:colorBackground">@color/background_dark</item>
        <item name="colorSurface">@color/surface_dark</item>
        <item name="colorOnPrimary">@color/black</item>
        <item name="colorOnSecondary">@color/black</item>
        <item name="colorOnBackground">@color/text_primary_dark</item>
        <item name="colorOnSurface">@color/text_primary_dark</item>
        <item name="android:statusBarColor">@color/black</item>
        <item name="android:windowLightStatusBar">false</item>
        <item name="fontFamily">@font/cairo</item>
    </style>
    
    <!-- وضع القارئ -->
    <style name="Theme.MangaReader.Reader" parent="Theme.MangaReader">
        <item name="android:statusBarColor">@color/black</item>
        <item name="android:navigationBarColor">@color/black</item>
        <item name="android:windowLightStatusBar">false</item>
        <item name="android:windowLightNavigationBar" tools:targetApi="o_mr1">false</item>
        <item name="android:windowBackground">@color/black</item>
    </style>
</resources>
```

## 6. بناء ملف APK

لبناء ملف APK، سنستخدم Gradle:

```bash
./gradlew assembleRelease
```

هذا الأمر سينتج ملف APK في المسار التالي:

```
app/build/outputs/apk/release/app-release.apk
```

## 7. توقيع ملف APK

لتوقيع ملف APK، سنقوم بإنشاء مفتاح توقيع وتوقيع الملف:

### إنشاء مفتاح التوقيع

```bash
keytool -genkey -v -keystore manga_reader_key.keystore -alias manga_reader -keyalg RSA -keysize 2048 -validity 10000
```

### تكوين Gradle لاستخدام مفتاح التوقيع

نضيف التكوين التالي إلى ملف `app/build.gradle`:

```gradle
android {
    // ...
    
    signingConfigs {
        release {
            storeFile file("../manga_reader_key.keystore")
            storePassword "password"
            keyAlias "manga_reader"
            keyPassword "password"
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
            // ...
        }
    }
}
```

### بناء ملف APK موقع

```bash
./gradlew assembleRelease
```

## 8. تحسين حجم ملف APK

لتقليل حجم ملف APK، سنستخدم تقنيات مختلفة:

### تمكين R8 (تقليل الحجم وتشويش الكود)

في ملف `gradle.properties`:

```
android.enableR8=true
android.enableR8.fullMode=true
```

### تقسيم ملف APK حسب المعمارية

في ملف `app/build.gradle`:

```gradle
android {
    // ...
    
    splits {
        abi {
            enable true
            reset()
            include 'armeabi-v7a', 'arm64-v8a', 'x86', 'x86_64'
            universalApk true
        }
    }
}
```

### استخدام Android App Bundle

```bash
./gradlew bundleRelease
```

هذا الأمر سينتج ملف AAB في المسار التالي:

```
app/build/outputs/bundle/release/app-release.aab
```

## 9. إنشاء ملف APK للتوزيع

لإنشاء ملف APK جاهز للتوزيع، سنستخدم الأمر التالي:

```bash
./gradlew assembleRelease
```

ثم نقوم بتوقيع ملف APK:

```bash
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore manga_reader_key.keystore app/build/outputs/apk/release/app-release-unsigned.apk manga_reader
```

ثم نقوم بمحاذاة ملف APK:

```bash
zipalign -v 4 app/build/outputs/apk/release/app-release-unsigned.apk manga_reader.apk
```

## 10. اختبار ملف APK

قبل توزيع ملف APK، يجب اختباره للتأكد من أنه يعمل بشكل صحيح:

1. تثبيت ملف APK على جهاز أو محاكي
2. اختبار جميع وظائف التطبيق
3. التأكد من عدم وجود أخطاء أو تعطل
4. التأكد من أن التطبيق يعمل بشكل صحيح على أجهزة مختلفة

## 11. نشر ملف APK

بعد اختبار ملف APK والتأكد من أنه يعمل بشكل صحيح، يمكن نشره بطرق مختلفة:

1. رفعه على متجر Google Play
2. رفعه على مواقع استضافة الملفات
3. مشاركته مباشرة مع المستخدمين

## 12. تنفيذ بناء ملف APK

الآن سنقوم بتنفيذ الخطوات اللازمة لبناء ملف APK:

1. إنشاء هيكل المشروع
2. إضافة ملفات الكود المصدري
3. إضافة ملفات الموارد
4. تكوين Gradle
5. بناء ملف APK
6. توقيع ملف APK
7. اختبار ملف APK

## 13. إنشاء ملف APK النهائي

بعد تنفيذ جميع الخطوات السابقة، سنحصل على ملف APK جاهز للتثبيت على الهاتف المحمول:

```
manga_reader.apk
```

هذا الملف يمكن تثبيته مباشرة على أي جهاز Android بإصدار 5.0 (API 21) أو أعلى.
