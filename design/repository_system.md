# نظام المصادر (Repository System) لتطبيق قارئ المانجا

## نظرة عامة على نظام المصادر

نظام المصادر هو العمود الفقري لتطبيقنا، حيث يتيح للمستخدمين الوصول إلى المانجا من مواقع مختلفة بطريقة موحدة وسلسة. تم تصميم النظام ليكون:

- **قابل للتوسع**: يمكن إضافة مصادر جديدة بسهولة
- **مرن**: يدعم أنواع مختلفة من مواقع المانجا
- **قوي**: قادر على تجاوز تقنيات الأمان المختلفة
- **فعال**: يقلل من استهلاك البيانات والموارد

## هيكل نظام المصادر

### 1. واجهة المصدر (Source Interface)

الواجهة الأساسية التي يجب أن تنفذها جميع المصادر:

```kotlin
interface Source {
    val id: Long                // معرف فريد للمصدر
    val name: String            // اسم المصدر
    val lang: String            // لغة المصدر (ar, en, etc.)
    val supportsLatest: Boolean // هل يدعم المصدر عرض أحدث المانجا
    
    // الوظائف الأساسية
    suspend fun getPopularManga(page: Int): MangaPageResult
    suspend fun searchManga(query: String, page: Int): MangaPageResult
    suspend fun getMangaDetails(manga: SManga): SManga
    suspend fun getChapterList(manga: SManga): List<SChapter>
    suspend fun getPageList(chapter: SChapter): List<SPage>
}
```

### 2. نظام المصادر الخارجية (External Sources System)

يسمح بإضافة مصادر من مستودعات خارجية:

```kotlin
class ExternalSourcesManager(
    private val context: Context,
    private val preferences: PreferencesHelper
) {
    // إضافة مصدر خارجي من URL
    suspend fun addSourceRepository(repoUrl: String): Result<SourceRepository>
    
    // تحديث المصادر الخارجية
    suspend fun updateExternalSources(): Result<List<ExternalSource>>
    
    // تثبيت/إلغاء تثبيت مصدر خارجي
    suspend fun installExternalSource(source: ExternalSource): Result<Boolean>
    suspend fun uninstallExternalSource(sourceId: Long): Result<Boolean>
    
    // الحصول على قائمة المصادر المتاحة
    fun getAvailableSources(): Flow<List<ExternalSource>>
    
    // الحصول على قائمة المستودعات المثبتة
    fun getInstalledRepositories(): Flow<List<SourceRepository>>
}
```

### 3. مدير المصادر (Sources Manager)

يدير جميع المصادر المتاحة في التطبيق:

```kotlin
class SourcesManager(
    private val context: Context,
    private val externalSourcesManager: ExternalSourcesManager
) {
    // الحصول على جميع المصادر المتاحة
    fun getSources(): Flow<List<Source>>
    
    // الحصول على مصدر محدد بواسطة المعرف
    fun getSource(sourceId: Long): Source?
    
    // تمكين/تعطيل مصدر
    fun toggleSource(sourceId: Long, enable: Boolean)
    
    // تحديث المصادر
    suspend fun updateSources(): Result<Unit>
    
    // تصفية المصادر حسب اللغة
    fun getSourcesByLang(lang: String): Flow<List<Source>>
}
```

## المصادر العربية المدعومة

### 1. مانجا سوات (MangaSwat)

```kotlin
class MangaSwatSource : HttpSource() {
    override val name = "مانجا سوات"
    override val baseUrl = "https://mangaswat.com"
    override val lang = "ar"
    override val id = 1001L
    
    // تنفيذ وظائف المصدر
    override suspend fun getPopularManga(page: Int): MangaPageResult {
        // تنفيذ خاص بموقع مانجا سوات
    }
    
    // تنفيذ باقي الوظائف...
}
```

### 2. مانجا ليونز (MangaLions)

```kotlin
class MangaLionsSource : HttpSource() {
    override val name = "مانجا ليونز"
    override val baseUrl = "https://mangalions.com"
    override val lang = "ar"
    override val id = 1002L
    
    // تنفيذ وظائف المصدر
    override suspend fun getPopularManga(page: Int): MangaPageResult {
        // تنفيذ خاص بموقع مانجا ليونز
    }
    
    // تنفيذ باقي الوظائف...
}
```

### 3. تيم إكس (TeamX)

```kotlin
class TeamXSource : HttpSource() {
    override val name = "تيم إكس"
    override val baseUrl = "https://teamx.site"
    override val lang = "ar"
    override val id = 1003L
    
    // تنفيذ وظائف المصدر
    override suspend fun getPopularManga(page: Int): MangaPageResult {
        // تنفيذ خاص بموقع تيم إكس
    }
    
    // تنفيذ باقي الوظائف...
}
```

### 4. مصادر عربية إضافية

- **مانجا عرب (MangaArab)**
- **أنمي سانكشواري (AnimeSanctuary)**
- **مانجا العرب (MangaAlarab)**

## تقنيات تجاوز الأمان

### 1. تجاوز CloudFlare

```kotlin
class CloudflareInterceptor(private val context: Context) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        val response = chain.proceed(request)
        
        // التحقق من وجود حماية CloudFlare
        if (response.code == 503 && response.headers["Server"]?.contains("cloudflare") == true) {
            // تنفيذ تقنيات تجاوز CloudFlare
            return handleCloudflare(response, chain)
        }
        
        return response
    }
    
    private fun handleCloudflare(response: Response, chain: Interceptor.Chain): Response {
        // تنفيذ خوارزمية تجاوز CloudFlare
    }
}
```

### 2. تجاوز CAPTCHA

```kotlin
class CaptchaSolver(private val context: Context) {
    // حل CAPTCHA باستخدام تقنيات التعرف على الصور
    suspend fun solveCaptcha(imageUrl: String): String {
        // تنفيذ خوارزمية حل CAPTCHA
    }
}
```

### 3. تجاوز مكافحة الزحف (Anti-Scraping)

```kotlin
class AntiScrapingBypass {
    // تقليد سلوك المتصفح
    fun createBrowserLikeRequest(url: String): Request {
        return Request.Builder()
            .url(url)
            .header("User-Agent", getRandomUserAgent())
            .header("Accept", "text/html,application/xhtml+xml,application/xml")
            .header("Accept-Language", "ar,en-US;q=0.9,en;q=0.8")
            .header("Referer", getRefererForUrl(url))
            .build()
    }
    
    // توليد User-Agent عشوائي
    private fun getRandomUserAgent(): String {
        // قائمة من User-Agents المختلفة
    }
    
    // الحصول على Referer مناسب
    private fun getRefererForUrl(url: String): String {
        // توليد Referer مناسب للموقع
    }
}
```

### 4. تجاوز قيود معدل الطلبات (Rate Limiting)

```kotlin
class RateLimitBypass(private val preferences: PreferencesHelper) : Interceptor {
    private val requestTimestamps = ConcurrentHashMap<String, Long>()
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        val host = request.url.host
        
        // التحقق من قيود معدل الطلبات
        if (shouldDelayRequest(host)) {
            // تأخير الطلب لتجنب القيود
            Thread.sleep(calculateDelay(host))
        }
        
        val response = chain.proceed(request)
        
        // تحديث سجل الطلبات
        requestTimestamps[host] = System.currentTimeMillis()
        
        return response
    }
    
    private fun shouldDelayRequest(host: String): Boolean {
        // التحقق مما إذا كان يجب تأخير الطلب
    }
    
    private fun calculateDelay(host: String): Long {
        // حساب وقت التأخير المناسب
    }
}
```

## نظام إضافة المصادر الخارجية

### 1. هيكل ملف المصدر الخارجي

```json
{
  "name": "اسم المصدر",
  "pkg": "com.example.source",
  "version": "1.0.0",
  "lang": "ar",
  "classPath": "com.example.source.ExampleSource",
  "dependencies": [
    {
      "name": "dependency1",
      "version": "1.0.0"
    }
  ]
}
```

### 2. مدير تثبيت المصادر

```kotlin
class SourceInstaller(private val context: Context) {
    // تثبيت مصدر من ملف APK
    suspend fun installSourceFromApk(apkFile: File): Result<Source>
    
    // تثبيت مصدر من مستودع
    suspend fun installSourceFromRepo(repoUrl: String, sourceId: String): Result<Source>
    
    // إلغاء تثبيت مصدر
    suspend fun uninstallSource(sourceId: Long): Result<Boolean>
    
    // تحديث مصدر
    suspend fun updateSource(sourceId: Long): Result<Source>
}
```

### 3. واجهة إضافة المستودعات

```kotlin
class RepositoryAddDialog(
    private val context: Context,
    private val externalSourcesManager: ExternalSourcesManager
) {
    // عرض مربع حوار لإضافة مستودع
    fun show() {
        // عرض واجهة إضافة مستودع
    }
    
    // إضافة مستودع
    suspend fun addRepository(url: String): Result<SourceRepository> {
        return externalSourcesManager.addSourceRepository(url)
    }
}
```

## آلية تحديث المصادر

### 1. جدولة التحديثات

```kotlin
class SourceUpdateScheduler(
    private val context: Context,
    private val sourcesManager: SourcesManager
) {
    // جدولة تحديث المصادر
    fun scheduleSourcesUpdate(intervalHours: Int) {
        // إعداد جدولة التحديث
    }
    
    // تنفيذ التحديث
    suspend fun performUpdate() {
        sourcesManager.updateSources()
    }
}
```

### 2. تحديث تلقائي للمصادر

```kotlin
class AutoUpdateWorker(
    context: Context,
    params: WorkerParameters,
    private val sourcesManager: SourcesManager
) : CoroutineWorker(context, params) {
    override suspend fun doWork(): Result {
        return try {
            sourcesManager.updateSources()
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
}
```

## إدارة ذاكرة التخزين المؤقت للمصادر

```kotlin
class SourceCacheManager(
    private val context: Context,
    private val preferences: PreferencesHelper
) {
    // تنظيف ذاكرة التخزين المؤقت
    suspend fun clearCache(sourceId: Long? = null): Result<Unit>
    
    // الحصول على حجم ذاكرة التخزين المؤقت
    suspend fun getCacheSize(sourceId: Long? = null): Long
    
    // تعيين حد أقصى لحجم ذاكرة التخزين المؤقت
    fun setCacheLimit(maxSizeMb: Int)
}
```

## تكامل نظام المصادر مع باقي التطبيق

### 1. تكامل مع وحدة المتصفح

```kotlin
class BrowserPresenter(
    private val sourcesManager: SourcesManager,
    private val preferences: PreferencesHelper
) {
    // الحصول على المصادر المتاحة للتصفح
    fun getAvailableSources(): Flow<List<Source>>
    
    // البحث في مصدر محدد
    suspend fun searchInSource(sourceId: Long, query: String, page: Int): MangaPageResult
    
    // الحصول على المانجا الشائعة من مصدر
    suspend fun getPopularManga(sourceId: Long, page: Int): MangaPageResult
}
```

### 2. تكامل مع وحدة التنزيلات

```kotlin
class DownloadPresenter(
    private val sourcesManager: SourcesManager,
    private val downloadManager: DownloadManager
) {
    // تنزيل فصل من مصدر محدد
    suspend fun downloadChapter(sourceId: Long, mangaId: String, chapterId: String): Result<Unit>
    
    // إلغاء تنزيل
    suspend fun cancelDownload(downloadId: Long): Result<Unit>
}
```

## اعتبارات الأداء والأمان

1. **تقليل عدد الطلبات**: استخدام ذاكرة التخزين المؤقت بشكل فعال لتقليل عدد الطلبات إلى المواقع.
2. **توزيع الطلبات**: تأخير الطلبات وتوزيعها لتجنب الحظر من المواقع.
3. **تشفير البيانات**: تشفير البيانات الحساسة مثل ملفات تكوين المصادر.
4. **تحديثات الأمان**: تحديث تقنيات تجاوز الأمان بانتظام لمواكبة التغييرات في مواقع المانجا.
5. **استهلاك الموارد**: مراقبة وتحسين استهلاك الذاكرة والبطارية أثناء استخدام المصادر.

## خطة التنفيذ

1. **المرحلة 1**: تنفيذ الواجهات الأساسية ونظام المصادر الداخلية.
2. **المرحلة 2**: تنفيذ المصادر العربية الأساسية (مانجا سوات، مانجا ليونز، تيم إكس).
3. **المرحلة 3**: تنفيذ نظام المصادر الخارجية وآلية التثبيت.
4. **المرحلة 4**: تنفيذ تقنيات تجاوز الأمان.
5. **المرحلة 5**: تكامل نظام المصادر مع باقي وحدات التطبيق.
6. **المرحلة 6**: اختبار وتحسين أداء نظام المصادر.
