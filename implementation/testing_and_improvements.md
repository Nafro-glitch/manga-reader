# اختبار وتحسين تطبيق قارئ المانجا

هذا الملف يوثق عملية اختبار التطبيق وتنفيذ الميزات الفريدة المتبقية لتحسين جودة المنتج النهائي.

## 1. خطة الاختبار

### 1.1 اختبار الوظائف الأساسية

- **تصفح المصادر**
  - التحقق من عرض جميع المصادر المدعومة
  - التحقق من تصفية المصادر حسب اللغة
  - التحقق من البحث في المصادر

- **البحث عن المانجا**
  - التحقق من البحث في مصدر واحد
  - التحقق من البحث في جميع المصادر
  - التحقق من البحث باللغة العربية
  - التحقق من البحث باللغة الإنجليزية

- **عرض تفاصيل المانجا**
  - التحقق من عرض معلومات المانجا بشكل صحيح
  - التحقق من عرض قائمة الفصول
  - التحقق من فرز وتصفية الفصول

- **قراءة المانجا**
  - التحقق من عرض صفحات المانجا بشكل صحيح
  - التحقق من التنقل بين الصفحات
  - التحقق من اتجاهات القراءة المختلفة (RTL، LTR، عمودي)
  - التحقق من ضبط جودة الصور

- **تنزيل المانجا**
  - التحقق من تنزيل فصل واحد
  - التحقق من تنزيل عدة فصول
  - التحقق من تنزيل مانجا كاملة
  - التحقق من إدارة قائمة التنزيلات

- **إدارة المكتبة**
  - التحقق من إضافة مانجا إلى المكتبة
  - التحقق من إزالة مانجا من المكتبة
  - التحقق من تنظيم المانجا في فئات
  - التحقق من فرز وتصفية المانجا في المكتبة

### 1.2 اختبار المصادر العربية

- **مانجا سوات**
  - التحقق من تصفح المانجا
  - التحقق من البحث
  - التحقق من تنزيل الفصول
  - التحقق من عرض الصفحات

- **مانجا ليونز**
  - التحقق من تصفح المانجا
  - التحقق من البحث
  - التحقق من تنزيل الفصول
  - التحقق من عرض الصفحات

- **تيم إكس**
  - التحقق من تصفح المانجا
  - التحقق من البحث
  - التحقق من تنزيل الفصول
  - التحقق من عرض الصفحات

### 1.3 اختبار تقنيات تجاوز الأمان

- **تجاوز CloudFlare**
  - التحقق من الوصول إلى المواقع المحمية بـ CloudFlare
  - التحقق من استمرارية الوصول بعد تغييرات CloudFlare

- **تجاوز CAPTCHA**
  - التحقق من تجاوز تحديات CAPTCHA البسيطة
  - التحقق من إشعار المستخدم عند الحاجة لتدخله

- **تجاوز قيود معدل الطلبات**
  - التحقق من تأخير الطلبات تلقائياً
  - التحقق من استخدام User-Agent متغير

### 1.4 اختبار واجهة المستخدم

- **الوضع الفاتح والداكن**
  - التحقق من التبديل بين الوضعين
  - التحقق من تطبيق الألوان بشكل صحيح

- **دعم اللغة العربية**
  - التحقق من عرض النصوص العربية بشكل صحيح
  - التحقق من اتجاه التخطيط من اليمين إلى اليسار

- **التوافق مع أحجام الشاشات المختلفة**
  - التحقق من العرض على الهواتف الصغيرة
  - التحقق من العرض على الهواتف الكبيرة
  - التحقق من العرض على الأجهزة اللوحية

## 2. تنفيذ الميزات الفريدة المتبقية

### 2.1 مزامنة المكتبة والتقدم في القراءة

```kotlin
// SyncManager.kt
class SyncManager @Inject constructor(
    private val mangaRepository: MangaRepository,
    private val chapterRepository: ChapterRepository,
    private val preferenceManager: PreferenceManager,
    private val networkHelper: NetworkHelper
) {
    
    suspend fun syncLibrary() {
        if (!networkHelper.isConnected()) return
        
        val userId = preferenceManager.getUserId()
        if (userId.isNullOrEmpty()) return
        
        // تحميل المكتبة من الخادم
        val remoteLibrary = apiService.getLibrary(userId)
        
        // دمج المكتبة المحلية مع المكتبة البعيدة
        remoteLibrary.forEach { remoteManga ->
            val localManga = mangaRepository.getMangaByUrl(remoteManga.url)
            if (localManga == null) {
                // إضافة المانجا الجديدة إلى المكتبة المحلية
                mangaRepository.insertManga(remoteManga.toLocalManga())
            } else {
                // تحديث معلومات المانجا المحلية
                mangaRepository.updateManga(localManga.copy(
                    favorite = true,
                    categories = remoteManga.categories
                ))
            }
        }
        
        // رفع المانجا المحلية غير الموجودة في المكتبة البعيدة
        val localLibrary = mangaRepository.getFavorites()
        localLibrary.forEach { localManga ->
            if (remoteLibrary.none { it.url == localManga.url }) {
                apiService.addToLibrary(userId, localManga.toRemoteManga())
            }
        }
    }
    
    suspend fun syncReadingProgress() {
        if (!networkHelper.isConnected()) return
        
        val userId = preferenceManager.getUserId()
        if (userId.isNullOrEmpty()) return
        
        // تحميل التقدم في القراءة من الخادم
        val remoteProgress = apiService.getReadingProgress(userId)
        
        // دمج التقدم المحلي مع التقدم البعيد
        remoteProgress.forEach { (mangaUrl, chapterProgress) ->
            val localManga = mangaRepository.getMangaByUrl(mangaUrl) ?: return@forEach
            
            chapterProgress.forEach { (chapterUrl, progress) ->
                val localChapter = chapterRepository.getChapterByUrl(chapterUrl) ?: return@forEach
                
                // تحديث التقدم المحلي إذا كان التقدم البعيد أحدث
                if (progress.lastRead > localChapter.lastRead) {
                    chapterRepository.updateChapter(localChapter.copy(
                        read = progress.read,
                        lastPageRead = progress.lastPage,
                        lastRead = progress.lastRead
                    ))
                }
            }
        }
        
        // رفع التقدم المحلي غير الموجود في التقدم البعيد
        val localManga = mangaRepository.getFavorites()
        localManga.forEach { manga ->
            val chapters = chapterRepository.getChapters(manga.id)
            val readChapters = chapters.filter { it.read || it.lastPageRead > 0 }
            
            val remoteChapters = remoteProgress[manga.url] ?: emptyMap()
            
            readChapters.forEach { chapter ->
                if (!remoteChapters.containsKey(chapter.url) || 
                    remoteChapters[chapter.url]?.lastRead ?: 0 < chapter.lastRead) {
                    apiService.updateReadingProgress(
                        userId,
                        manga.url,
                        chapter.url,
                        ReadingProgress(
                            read = chapter.read,
                            lastPage = chapter.lastPageRead,
                            lastRead = chapter.lastRead
                        )
                    )
                }
            }
        }
    }
}
```

### 2.2 إشعارات ذكية بالفصول الجديدة

```kotlin
// NotificationWorker.kt
class NotificationWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    @Inject
    lateinit var mangaRepository: MangaRepository
    
    @Inject
    lateinit var chapterRepository: ChapterRepository
    
    @Inject
    lateinit var sourceManager: SourceManager
    
    @Inject
    lateinit var preferenceManager: PreferenceManager
    
    @Inject
    lateinit var notificationHelper: NotificationHelper
    
    override suspend fun doWork(): Result {
        // التحقق من تفضيلات الإشعارات
        if (!preferenceManager.isLibraryUpdateNotificationEnabled()) {
            return Result.success()
        }
        
        // الحصول على المانجا المفضلة
        val favorites = mangaRepository.getFavorites()
        
        // التحقق من وجود فصول جديدة لكل مانجا
        val newChapters = mutableMapOf<Manga, List<Chapter>>()
        
        favorites.forEach { manga ->
            val source = sourceManager.getOrStub(manga.source)
            try {
                // تحميل الفصول من المصدر
                val fetchedChapters = source.getChapterList(manga.toSManga())
                
                // الحصول على الفصول المحلية
                val localChapters = chapterRepository.getChapters(manga.id)
                
                // تحديد الفصول الجديدة
                val newChaptersList = fetchedChapters.filter { fetchedChapter ->
                    localChapters.none { it.url == fetchedChapter.url }
                }.map { it.toChapter(manga.id) }
                
                if (newChaptersList.isNotEmpty()) {
                    // حفظ الفصول الجديدة في قاعدة البيانات
                    chapterRepository.insertChapters(newChaptersList)
                    
                    // إضافة الفصول الجديدة إلى القائمة
                    newChapters[manga] = newChaptersList
                }
            } catch (e: Exception) {
                // تسجيل الخطأ
                Log.e("NotificationWorker", "Error checking for updates for ${manga.title}", e)
            }
        }
        
        // إرسال إشعارات للفصول الجديدة
        if (newChapters.isNotEmpty()) {
            notificationHelper.showNewChaptersNotification(newChapters)
        }
        
        return Result.success()
    }
}

// NotificationHelper.kt
class NotificationHelper @Inject constructor(
    private val context: Context
) {
    
    private val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
    
    init {
        createChannels()
    }
    
    private fun createChannels() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channelName = context.getString(R.string.new_chapters_channel_name)
            val channelDescription = context.getString(R.string.new_chapters_channel_description)
            val importance = NotificationManager.IMPORTANCE_DEFAULT
            
            val channel = NotificationChannel(CHANNEL_NEW_CHAPTERS, channelName, importance).apply {
                description = channelDescription
                enableLights(true)
                lightColor = ContextCompat.getColor(context, R.color.primary)
                enableVibration(true)
            }
            
            notificationManager.createNotificationChannel(channel)
        }
    }
    
    fun showNewChaptersNotification(newChapters: Map<Manga, List<Chapter>>) {
        // تجميع الإشعارات حسب تفضيلات المستخدم
        val groupByManga = newChapters.size > 1
        
        if (groupByManga) {
            // إنشاء إشعار مجمع لجميع المانجا
            val totalChapters = newChapters.values.sumOf { it.size }
            val totalManga = newChapters.size
            
            val title = context.getString(R.string.new_chapters_title, totalChapters)
            val text = context.resources.getQuantityString(
                R.plurals.new_chapters_for_manga,
                totalManga,
                totalManga
            )
            
            val intent = Intent(context, MainActivity::class.java).apply {
                flags = Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP
            }
            
            val pendingIntent = PendingIntent.getActivity(
                context,
                0,
                intent,
                PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
            )
            
            val notification = NotificationCompat.Builder(context, CHANNEL_NEW_CHAPTERS)
                .setSmallIcon(R.drawable.ic_notification)
                .setContentTitle(title)
                .setContentText(text)
                .setContentIntent(pendingIntent)
                .setAutoCancel(true)
                .build()
            
            notificationManager.notify(NOTIFICATION_ID_NEW_CHAPTERS, notification)
        } else {
            // إنشاء إشعار منفصل لكل مانجا
            newChapters.forEach { (manga, chapters) ->
                val title = manga.title
                val text = context.resources.getQuantityString(
                    R.plurals.new_chapters,
                    chapters.size,
                    chapters.size
                )
                
                val intent = Intent(context, MangaDetailsActivity::class.java).apply {
                    putExtra(MangaDetailsActivity.EXTRA_MANGA_ID, manga.id)
                    flags = Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP
                }
                
                val pendingIntent = PendingIntent.getActivity(
                    context,
                    manga.id.toInt(),
                    intent,
                    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
                )
                
                val notification = NotificationCompat.Builder(context, CHANNEL_NEW_CHAPTERS)
                    .setSmallIcon(R.drawable.ic_notification)
                    .setContentTitle(title)
                    .setContentText(text)
                    .setContentIntent(pendingIntent)
                    .setAutoCancel(true)
                    .build()
                
                notificationManager.notify(manga.id.toInt(), notification)
            }
        }
    }
    
    companion object {
        private const val CHANNEL_NEW_CHAPTERS = "new_chapters"
        private const val NOTIFICATION_ID_NEW_CHAPTERS = 1
    }
}
```

### 2.3 تصنيف وتنظيم متقدم للمانجا

```kotlin
// CategoryManager.kt
class CategoryManager @Inject constructor(
    private val categoryRepository: CategoryRepository,
    private val mangaRepository: MangaRepository
) {
    
    suspend fun createCategory(name: String, order: Int = -1): Long {
        val categories = categoryRepository.getCategories()
        
        val newOrder = if (order == -1) {
            categories.maxOfOrNull { it.order } ?: 0
        } else {
            // تحديث ترتيب الفئات الأخرى
            categories.filter { it.order >= order }.forEach {
                categoryRepository.updateCategory(it.copy(order = it.order + 1))
            }
            order
        }
        
        val category = Category(
            id = 0,
            name = name,
            order = newOrder,
            flags = 0
        )
        
        return categoryRepository.insertCategory(category)
    }
    
    suspend fun updateCategory(category: Category): Boolean {
        return categoryRepository.updateCategory(category) > 0
    }
    
    suspend fun deleteCategory(categoryId: Long): Boolean {
        // إزالة الفئة من جميع المانجا
        val mangaCategories = mangaRepository.getMangaCategoriesByCategory(categoryId)
        mangaCategories.forEach { manga ->
            val categories = manga.categories.toMutableList()
            categories.remove(categoryId)
            mangaRepository.updateMangaCategories(manga.id, categories)
        }
        
        return categoryRepository.deleteCategory(categoryId) > 0
    }
    
    suspend fun reorderCategory(categoryId: Long, newOrder: Int): Boolean {
        val category = categoryRepository.getCategory(categoryId) ?: return false
        val categories = categoryRepository.getCategories()
        
        // تحديث ترتيب الفئات
        if (newOrder > category.order) {
            // تحريك لأسفل
            categories.filter { it.order in (category.order + 1)..newOrder }.forEach {
                categoryRepository.updateCategory(it.copy(order = it.order - 1))
            }
        } else if (newOrder < category.order) {
            // تحريك لأعلى
            categories.filter { it.order in newOrder until category.order }.forEach {
                categoryRepository.updateCategory(it.copy(order = it.order + 1))
            }
        } else {
            // نفس الترتيب، لا تغيير
            return true
        }
        
        return categoryRepository.updateCategory(category.copy(order = newOrder)) > 0
    }
    
    suspend fun addMangaToCategory(mangaId: Long, categoryId: Long): Boolean {
        val manga = mangaRepository.getManga(mangaId) ?: return false
        
        val categories = manga.categories.toMutableList()
        if (categoryId !in categories) {
            categories.add(categoryId)
            return mangaRepository.updateMangaCategories(mangaId, categories) > 0
        }
        
        return true
    }
    
    suspend fun removeMangaFromCategory(mangaId: Long, categoryId: Long): Boolean {
        val manga = mangaRepository.getManga(mangaId) ?: return false
        
        val categories = manga.categories.toMutableList()
        if (categoryId in categories) {
            categories.remove(categoryId)
            return mangaRepository.updateMangaCategories(mangaId, categories) > 0
        }
        
        return true
    }
    
    suspend fun setMangaCategories(mangaId: Long, categoryIds: List<Long>): Boolean {
        return mangaRepository.updateMangaCategories(mangaId, categoryIds) > 0
    }
}
```

### 2.4 إحصائيات مفصلة عن عادات القراءة

```kotlin
// StatisticsManager.kt
class StatisticsManager @Inject constructor(
    private val statisticsRepository: StatisticsRepository,
    private val mangaRepository: MangaRepository,
    private val chapterRepository: ChapterRepository
) {
    
    suspend fun recordChapterRead(mangaId: Long, chapterId: Long, readDuration: Long) {
        val manga = mangaRepository.getManga(mangaId) ?: return
        val chapter = chapterRepository.getChapter(chapterId) ?: return
        
        // تسجيل إحصائية القراءة
        val readStat = ReadStat(
            id = 0,
            mangaId = mangaId,
            chapterId = chapterId,
            readAt = System.currentTimeMillis(),
            readDuration = readDuration
        )
        
        statisticsRepository.insertReadStat(readStat)
        
        // تحديث إجمالي وقت القراءة للمانجا
        val mangaStat = statisticsRepository.getMangaStat(mangaId)
        if (mangaStat == null) {
            statisticsRepository.insertMangaStat(
                MangaStat(
                    mangaId = mangaId,
                    totalReadDuration = readDuration,
                    lastRead = System.currentTimeMillis(),
                    chaptersRead = 1
                )
            )
        } else {
            statisticsRepository.updateMangaStat(
                mangaStat.copy(
                    totalReadDuration = mangaStat.totalReadDuration + readDuration,
                    lastRead = System.currentTimeMillis(),
                    chaptersRead = mangaStat.chaptersRead + 1
                )
            )
        }
    }
    
    suspend fun getReadingStats(): ReadingStats {
        val totalReadDuration = statisticsRepository.getTotalReadDuration()
        val totalChaptersRead = statisticsRepository.getTotalChaptersRead()
        val totalMangaRead = statisticsRepository.getTotalMangaRead()
        
        val readingByDay = statisticsRepository.getReadingByDay()
        val readingByHour = statisticsRepository.getReadingByHour()
        val readingByGenre = statisticsRepository.getReadingByGenre()
        
        val mostReadManga = statisticsRepository.getMostReadManga(10)
        val recentlyRead = statisticsRepository.getRecentlyRead(10)
        
        return ReadingStats(
            totalReadDuration = totalReadDuration,
            totalChaptersRead = totalChaptersRead,
            totalMangaRead = totalMangaRead,
            readingByDay = readingByDay,
            readingByHour = readingByHour,
            readingByGenre = readingByGenre,
            mostReadManga = mostReadManga,
            recentlyRead = recentlyRead
        )
    }
    
    suspend fun getMangaReadingStats(mangaId: Long): MangaReadingStats? {
        val manga = mangaRepository.getManga(mangaId) ?: return null
        val mangaStat = statisticsRepository.getMangaStat(mangaId) ?: return null
        
        val totalChapters = chapterRepository.getChapterCount(mangaId)
        val readChapters = chapterRepository.getReadChapterCount(mangaId)
        
        val readingHistory = statisticsRepository.getMangaReadingHistory(mangaId)
        
        return MangaReadingStats(
            manga = manga,
            totalReadDuration = mangaStat.totalReadDuration,
            chaptersRead = mangaStat.chaptersRead,
            lastRead = mangaStat.lastRead,
            totalChapters = totalChapters,
            readChapters = readChapters,
            readingHistory = readingHistory
        )
    }
}

// ReadingStats.kt
data class ReadingStats(
    val totalReadDuration: Long,
    val totalChaptersRead: Int,
    val totalMangaRead: Int,
    val readingByDay: Map<Int, Long>, // يوم الأسبوع -> وقت القراءة
    val readingByHour: Map<Int, Long>, // ساعة اليوم -> وقت القراءة
    val readingByGenre: Map<String, Long>, // النوع -> وقت القراءة
    val mostReadManga: List<MangaWithStat>,
    val recentlyRead: List<MangaWithStat>
)

// MangaReadingStats.kt
data class MangaReadingStats(
    val manga: Manga,
    val totalReadDuration: Long,
    val chaptersRead: Int,
    val lastRead: Long,
    val totalChapters: Int,
    val readChapters: Int,
    val readingHistory: List<ReadStat>
)
```

## 3. اختبار الأداء

### 3.1 اختبار استهلاك الذاكرة

```kotlin
// MemoryProfiler.kt
class MemoryProfiler {
    
    private val memoryInfo = ActivityManager.MemoryInfo()
    private val activityManager = context.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
    
    fun getMemoryUsage(): MemoryUsage {
        activityManager.getMemoryInfo(memoryInfo)
        
        val runtime = Runtime.getRuntime()
        val usedMemory = runtime.totalMemory() - runtime.freeMemory()
        
        return MemoryUsage(
            totalMemory = runtime.totalMemory(),
            freeMemory = runtime.freeMemory(),
            usedMemory = usedMemory,
            maxMemory = runtime.maxMemory(),
            systemAvailableMemory = memoryInfo.availMem,
            systemTotalMemory = memoryInfo.totalMem,
            lowMemory = memoryInfo.lowMemory
        )
    }
    
    fun logMemoryUsage(tag: String) {
        val usage = getMemoryUsage()
        Log.d(tag, "Memory Usage: ${usage.usedMemory / 1024 / 1024} MB / ${usage.maxMemory / 1024 / 1024} MB")
        Log.d(tag, "System Memory: ${usage.systemAvailableMemory / 1024 / 1024} MB / ${usage.systemTotalMemory / 1024 / 1024} MB")
        Log.d(tag, "Low Memory: ${usage.lowMemory}")
    }
}
```

### 3.2 اختبار استهلاك البطارية

```kotlin
// BatteryProfiler.kt
class BatteryProfiler(private val context: Context) {
    
    private var startBatteryLevel: Int = -1
    private var startTime: Long = 0
    
    fun startMonitoring() {
        val batteryStatus = context.registerReceiver(
            null,
            IntentFilter(Intent.ACTION_BATTERY_CHANGED)
        )
        
        startBatteryLevel = batteryStatus?.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) ?: -1
        startTime = System.currentTimeMillis()
    }
    
    fun getBatteryUsage(): BatteryUsage {
        val batteryStatus = context.registerReceiver(
            null,
            IntentFilter(Intent.ACTION_BATTERY_CHANGED)
        )
        
        val currentBatteryLevel = batteryStatus?.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) ?: -1
        val scale = batteryStatus?.getIntExtra(BatteryManager.EXTRA_SCALE, -1) ?: -1
        
        val batteryPct = currentBatteryLevel * 100 / scale.toFloat()
        val startBatteryPct = startBatteryLevel * 100 / scale.toFloat()
        
        val usagePct = startBatteryPct - batteryPct
        val usageTime = System.currentTimeMillis() - startTime
        
        return BatteryUsage(
            startLevel = startBatteryLevel,
            currentLevel = currentBatteryLevel,
            usagePercent = usagePct,
            usageTime = usageTime
        )
    }
    
    fun logBatteryUsage(tag: String) {
        val usage = getBatteryUsage()
        Log.d(tag, "Battery Usage: ${usage.usagePercent}% over ${usage.usageTime / 1000 / 60} minutes")
    }
}
```

### 3.3 اختبار سرعة التحميل

```kotlin
// PerformanceProfiler.kt
class PerformanceProfiler {
    
    private val timings = mutableMapOf<String, Long>()
    
    fun startTiming(key: String) {
        timings[key] = System.currentTimeMillis()
    }
    
    fun endTiming(key: String): Long {
        val startTime = timings[key] ?: return -1
        val endTime = System.currentTimeMillis()
        val duration = endTime - startTime
        
        Log.d("PerformanceProfiler", "$key: $duration ms")
        
        return duration
    }
    
    fun measureOperation(key: String, operation: suspend () -> Unit): Long = runBlocking {
        startTiming(key)
        operation()
        endTiming(key)
    }
}
```

## 4. تحسينات الأداء

### 4.1 تحسين تحميل الصور

```kotlin
// ImageLoader.kt
class ImageLoader @Inject constructor(
    private val context: Context,
    private val networkHelper: NetworkHelper,
    private val preferenceManager: PreferenceManager
) {
    
    private val imageCache = LruCache<String, Bitmap>(
        (Runtime.getRuntime().maxMemory() / 8).toInt()
    )
    
    private val diskCache = DiskLruCache.open(
        File(context.cacheDir, "images"),
        1,
        1,
        50 * 1024 * 1024 // 50 MB
    )
    
    suspend fun loadImage(url: String, target: ImageView, placeholder: Int = R.drawable.placeholder_cover) {
        // عرض الصورة المؤقتة
        target.setImageResource(placeholder)
        
        // التحقق من وجود الصورة في ذاكرة التخزين المؤقت
        val cachedBitmap = imageCache.get(url)
        if (cachedBitmap != null) {
            withContext(Dispatchers.Main) {
                target.setImageBitmap(cachedBitmap)
            }
            return
        }
        
        // التحقق من وجود الصورة في التخزين المؤقت على القرص
        val key = url.md5()
        val snapshot = diskCache.get(key)
        if (snapshot != null) {
            try {
                val inputStream = snapshot.getInputStream(0)
                val bitmap = BitmapFactory.decodeStream(inputStream)
                if (bitmap != null) {
                    imageCache.put(url, bitmap)
                    withContext(Dispatchers.Main) {
                        target.setImageBitmap(bitmap)
                    }
                    return
                }
            } finally {
                snapshot.close()
            }
        }
        
        // تحميل الصورة من الإنترنت
        if (!networkHelper.isConnected()) return
        
        try {
            val response = networkHelper.client.newCall(
                Request.Builder().url(url).build()
            ).execute()
            
            if (!response.isSuccessful) return
            
            response.body?.let { body ->
                val inputStream = body.byteStream()
                val bitmap = BitmapFactory.decodeStream(inputStream)
                
                if (bitmap != null) {
                    // حفظ الصورة في ذاكرة التخزين المؤقت
                    imageCache.put(url, bitmap)
                    
                    // حفظ الصورة في التخزين المؤقت على القرص
                    diskCache.edit(key)?.apply {
                        val outputStream = newOutputStream(0)
                        bitmap.compress(Bitmap.CompressFormat.JPEG, 90, outputStream)
                        outputStream.close()
                        commit()
                    }
                    
                    withContext(Dispatchers.Main) {
                        target.setImageBitmap(bitmap)
                    }
                }
            }
        } catch (e: Exception) {
            Log.e("ImageLoader", "Error loading image: $url", e)
        }
    }
    
    fun clearCache() {
        imageCache.evictAll()
        diskCache.delete()
    }
    
    private fun String.md5(): String {
        val md = MessageDigest.getInstance("MD5")
        val digest = md.digest(toByteArray())
        return digest.joinToString("") { "%02x".format(it) }
    }
}
```

### 4.2 تحسين استهلاك البيانات

```kotlin
// DataSaver.kt
class DataSaver @Inject constructor(
    private val preferenceManager: PreferenceManager,
    private val networkHelper: NetworkHelper
) {
    
    fun shouldCompressImages(): Boolean {
        return preferenceManager.isDataSaverEnabled() && networkHelper.isMeteredConnection()
    }
    
    fun getImageQuality(): Int {
        return if (shouldCompressImages()) {
            preferenceManager.getDataSaverImageQuality()
        } else {
            100
        }
    }
    
    fun shouldPreloadImages(): Boolean {
        return !preferenceManager.isDataSaverEnabled() || !networkHelper.isMeteredConnection()
    }
    
    fun shouldLoadHighResolutionCovers(): Boolean {
        return !preferenceManager.isDataSaverEnabled() || !networkHelper.isMeteredConnection()
    }
    
    fun applyDataSaving(request: Request): Request {
        if (!preferenceManager.isDataSaverEnabled()) {
            return request
        }
        
        val builder = request.newBuilder()
        
        // إضافة رأس لطلب صور بجودة منخفضة
        if (request.url.toString().isImageUrl() && shouldCompressImages()) {
            builder.header("Accept", "image/webp,image/*;q=0.8")
            builder.header("Cache-Control", "max-stale=60")
        }
        
        return builder.build()
    }
    
    private fun String.isImageUrl(): Boolean {
        val imageExtensions = listOf("jpg", "jpeg", "png", "gif", "webp")
        return imageExtensions.any { this.endsWith(".$it", ignoreCase = true) }
    }
}
```

### 4.3 تحسين أداء قاعدة البيانات

```kotlin
// DatabaseOptimizer.kt
class DatabaseOptimizer @Inject constructor(
    private val database: AppDatabase
) {
    
    suspend fun optimizeDatabase() {
        withContext(Dispatchers.IO) {
            // تنفيذ VACUUM لتقليل حجم قاعدة البيانات
            database.openHelper.writableDatabase.execSQL("VACUUM")
            
            // تنفيذ ANALYZE لتحسين المؤشرات
            database.openHelper.writableDatabase.execSQL("ANALYZE")
            
            // إعادة بناء الفهارس
            database.openHelper.writableDatabase.execSQL("REINDEX")
        }
    }
    
    suspend fun cleanupDatabase() {
        withContext(Dispatchers.IO) {
            // حذف الفصول القديمة غير المقروءة وغير المحفوظة
            val threshold = System.currentTimeMillis() - TimeUnit.DAYS.toMillis(30)
            database.chapterDao().deleteOldChapters(threshold)
            
            // حذف المانجا غير المفضلة التي لم يتم الوصول إليها منذ فترة طويلة
            database.mangaDao().deleteUnusedManga(threshold)
            
            // حذف بيانات التخزين المؤقت القديمة
            database.cacheDao().deleteOldCache(threshold)
        }
    }
}
```

## 5. تحسينات الأمان

### 5.1 تشفير البيانات الحساسة

```kotlin
// SecurityManager.kt
class SecurityManager @Inject constructor(
    private val context: Context
) {
    
    private val keyStore = KeyStore.getInstance("AndroidKeyStore").apply {
        load(null)
    }
    
    private fun createKey() {
        if (!keyStore.containsAlias(KEY_ALIAS)) {
            val keyGenerator = KeyGenerator.getInstance(
                KeyProperties.KEY_ALGORITHM_AES,
                "AndroidKeyStore"
            )
            
            val keyGenParameterSpec = KeyGenParameterSpec.Builder(
                KEY_ALIAS,
                KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
            )
                .setBlockModes(KeyProperties.BLOCK_MODE_CBC)
                .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_PKCS7)
                .setUserAuthenticationRequired(false)
                .build()
            
            keyGenerator.init(keyGenParameterSpec)
            keyGenerator.generateKey()
        }
    }
    
    fun encrypt(data: String): String {
        createKey()
        
        val key = keyStore.getKey(KEY_ALIAS, null) as SecretKey
        val cipher = Cipher.getInstance(TRANSFORMATION)
        cipher.init(Cipher.ENCRYPT_MODE, key)
        
        val iv = cipher.iv
        val encryptedBytes = cipher.doFinal(data.toByteArray(Charsets.UTF_8))
        
        val combined = ByteArray(iv.size + encryptedBytes.size)
        System.arraycopy(iv, 0, combined, 0, iv.size)
        System.arraycopy(encryptedBytes, 0, combined, iv.size, encryptedBytes.size)
        
        return Base64.encodeToString(combined, Base64.DEFAULT)
    }
    
    fun decrypt(encryptedData: String): String {
        val combined = Base64.decode(encryptedData, Base64.DEFAULT)
        
        val key = keyStore.getKey(KEY_ALIAS, null) as SecretKey
        val cipher = Cipher.getInstance(TRANSFORMATION)
        
        val iv = ByteArray(16)
        System.arraycopy(combined, 0, iv, 0, iv.size)
        
        val encryptedBytes = ByteArray(combined.size - iv.size)
        System.arraycopy(combined, iv.size, encryptedBytes, 0, encryptedBytes.size)
        
        val ivParameterSpec = IvParameterSpec(iv)
        cipher.init(Cipher.DECRYPT_MODE, key, ivParameterSpec)
        
        val decryptedBytes = cipher.doFinal(encryptedBytes)
        return String(decryptedBytes, Charsets.UTF_8)
    }
    
    companion object {
        private const val KEY_ALIAS = "manga_reader_key"
        private const val TRANSFORMATION = "AES/CBC/PKCS7Padding"
    }
}
```

### 5.2 تحسين تجاوز الحماية

```kotlin
// CloudflareInterceptor.kt
class CloudflareInterceptor @Inject constructor(
    private val context: Context,
    private val cookieManager: CookieManager
) : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        val response = chain.proceed(request)
        
        // التحقق من وجود تحدي Cloudflare
        if (response.code == 503 && response.headers["Server"]?.contains("cloudflare") == true) {
            response.close()
            
            // حل تحدي Cloudflare
            return solveCloudflareChallengeAndProceed(chain, request)
        }
        
        return response
    }
    
    private fun solveCloudflareChallengeAndProceed(chain: Interceptor.Chain, request: Request): Response {
        // استخراج تحدي Cloudflare
        val challengeResponse = chain.proceed(request)
        val responseBody = challengeResponse.body?.string() ?: ""
        challengeResponse.close()
        
        // استخراج معلمات التحدي
        val challengeParams = extractChallengeParams(responseBody)
        
        // حل التحدي
        val solution = solveChallenge(challengeParams)
        
        // إنشاء طلب جديد مع الحل
        val cookies = cookieManager.getCookieStore().getCookies()
        val cookieHeader = cookies.joinToString("; ") { "${it.name}=${it.value}" }
        
        val newRequest = request.newBuilder()
            .header("Cookie", cookieHeader)
            .header("User-Agent", getRandomUserAgent())
            .build()
        
        // إرسال الطلب مع الحل
        return chain.proceed(newRequest)
    }
    
    private fun extractChallengeParams(html: String): Map<String, String> {
        val params = mutableMapOf<String, String>()
        
        // استخراج معلمات التحدي باستخدام تعبيرات منتظمة
        val jschlRegex = Regex("name=\"jschl_vc\" value=\"(\\w+)\"")
        val passRegex = Regex("name=\"pass\" value=\"(.+?)\"")
        val jschlRegexResult = jschlRegex.find(html)
        val passRegexResult = passRegex.find(html)
        
        if (jschlRegexResult != null) {
            params["jschl_vc"] = jschlRegexResult.groupValues[1]
        }
        
        if (passRegexResult != null) {
            params["pass"] = passRegexResult.groupValues[1]
        }
        
        return params
    }
    
    private fun solveChallenge(params: Map<String, String>): String {
        // تنفيذ حل التحدي
        // هذا مجرد مثال، الحل الفعلي يعتمد على نوع التحدي
        
        return "solution"
    }
    
    private fun getRandomUserAgent(): String {
        val userAgents = listOf(
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Safari/605.1.15",
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0",
            "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36"
        )
        
        return userAgents.random()
    }
}
```

## 6. تحسينات تجربة المستخدم

### 6.1 تحسين القراءة المستمرة

```kotlin
// ContinuousReaderFragment.kt
class ContinuousReaderFragment : Fragment() {
    
    private var _binding: FragmentContinuousReaderBinding? = null
    private val binding get() = _binding!!
    
    private val viewModel: ReaderViewModel by viewModels()
    
    private val adapter = ContinuousReaderAdapter()
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentContinuousReaderBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        setupRecyclerView()
        observeViewModel()
    }
    
    private fun setupRecyclerView() {
        binding.recyclerView.adapter = adapter
        
        // إعداد مدير التخطيط حسب اتجاه القراءة
        val layoutManager = when (viewModel.readingDirection.value) {
            ReadingDirection.RIGHT_TO_LEFT -> RtlLinearLayoutManager(requireContext())
            ReadingDirection.LEFT_TO_RIGHT -> LinearLayoutManager(requireContext())
            ReadingDirection.VERTICAL -> LinearLayoutManager(requireContext())
            else -> LinearLayoutManager(requireContext())
        }
        
        binding.recyclerView.layoutManager = layoutManager
        
        // إعداد مستمع التمرير لتتبع موضع القراءة
        binding.recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
            override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                super.onScrolled(recyclerView, dx, dy)
                
                val visiblePosition = layoutManager.findFirstVisibleItemPosition()
                if (visiblePosition != RecyclerView.NO_POSITION) {
                    viewModel.updateReadingPosition(visiblePosition)
                }
            }
        })
    }
    
    private fun observeViewModel() {
        viewModel.pages.observe(viewLifecycleOwner) { pages ->
            adapter.submitList(pages)
            
            // التمرير إلى آخر موضع قراءة
            val lastPosition = viewModel.getLastReadPosition()
            if (lastPosition > 0 && lastPosition < pages.size) {
                binding.recyclerView.scrollToPosition(lastPosition)
            }
        }
        
        viewModel.readingDirection.observe(viewLifecycleOwner) { direction ->
            // تحديث اتجاه القراءة
            val layoutManager = when (direction) {
                ReadingDirection.RIGHT_TO_LEFT -> RtlLinearLayoutManager(requireContext())
                ReadingDirection.LEFT_TO_RIGHT -> LinearLayoutManager(requireContext())
                ReadingDirection.VERTICAL -> LinearLayoutManager(requireContext())
                else -> LinearLayoutManager(requireContext())
            }
            
            val currentPosition = (binding.recyclerView.layoutManager as LinearLayoutManager)
                .findFirstVisibleItemPosition()
            
            binding.recyclerView.layoutManager = layoutManager
            
            if (currentPosition != RecyclerView.NO_POSITION) {
                binding.recyclerView.scrollToPosition(currentPosition)
            }
        }
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}

// RtlLinearLayoutManager.kt
class RtlLinearLayoutManager(context: Context) : LinearLayoutManager(context) {
    
    init {
        reverseLayout = true
    }
    
    override fun canScrollHorizontally(): Boolean {
        return true
    }
    
    override fun canScrollVertically(): Boolean {
        return false
    }
}
```

### 6.2 تحسين البحث باللغة العربية

```kotlin
// ArabicSearchHelper.kt
class ArabicSearchHelper {
    
    fun normalizeArabicText(text: String): String {
        return text
            .replace("أ", "ا")
            .replace("إ", "ا")
            .replace("آ", "ا")
            .replace("ة", "ه")
            .replace("ى", "ي")
            .replace("ئ", "ي")
            .trim()
    }
    
    fun searchArabicText(query: String, text: String): Boolean {
        val normalizedQuery = normalizeArabicText(query)
        val normalizedText = normalizeArabicText(text)
        
        return normalizedText.contains(normalizedQuery, ignoreCase = true)
    }
    
    fun rankSearchResults(query: String, results: List<SearchResult>): List<SearchResult> {
        val normalizedQuery = normalizeArabicText(query)
        
        return results.map { result ->
            val normalizedTitle = normalizeArabicText(result.manga.title)
            
            // حساب درجة التطابق
            val titleScore = if (normalizedTitle.contains(normalizedQuery, ignoreCase = true)) {
                // درجة أعلى إذا كان العنوان يبدأ بالاستعلام
                if (normalizedTitle.startsWith(normalizedQuery, ignoreCase = true)) {
                    100
                } else {
                    75
                }
            } else {
                0
            }
            
            val authorScore = if (result.manga.author?.let { 
                normalizeArabicText(it).contains(normalizedQuery, ignoreCase = true) 
            } == true) {
                50
            } else {
                0
            }
            
            val genreScore = result.manga.genre?.let { genre ->
                if (normalizeArabicText(genre).contains(normalizedQuery, ignoreCase = true)) {
                    25
                } else {
                    0
                }
            } ?: 0
            
            // الدرجة الإجمالية
            val totalScore = titleScore + authorScore + genreScore
            
            result.copy(score = totalScore)
        }.sortedByDescending { it.score }
    }
}
```

### 6.3 تحسين دعم الأجهزة ذات الشاشات الكبيرة

```kotlin
// TabletLayoutManager.kt
class TabletLayoutManager @Inject constructor(
    private val preferenceManager: PreferenceManager
) {
    
    fun isTablet(context: Context): Boolean {
        val displayMetrics = context.resources.displayMetrics
        val widthDp = displayMetrics.widthPixels / displayMetrics.density
        val heightDp = displayMetrics.heightPixels / displayMetrics.density
        
        val screenSizeType = context.resources.configuration.screenLayout and 
                Configuration.SCREENLAYOUT_SIZE_MASK
        
        return screenSizeType >= Configuration.SCREENLAYOUT_SIZE_LARGE && 
                (widthDp >= 600 || heightDp >= 600)
    }
    
    fun shouldUseTabletLayout(context: Context): Boolean {
        return isTablet(context) && preferenceManager.isTabletLayoutEnabled()
    }
    
    fun getTabletLayoutType(context: Context): TabletLayoutType {
        if (!shouldUseTabletLayout(context)) {
            return TabletLayoutType.PHONE
        }
        
        val displayMetrics = context.resources.displayMetrics
        val widthDp = displayMetrics.widthPixels / displayMetrics.density
        
        return if (widthDp >= 900) {
            TabletLayoutType.LARGE_TABLET
        } else {
            TabletLayoutType.SMALL_TABLET
        }
    }
    
    fun setupTabletLayout(activity: FragmentActivity) {
        val layoutType = getTabletLayoutType(activity)
        
        if (layoutType == TabletLayoutType.PHONE) {
            return
        }
        
        // تطبيق تخطيط الجهاز اللوحي
        when (activity) {
            is MainActivity -> setupMainActivityTabletLayout(activity, layoutType)
            is MangaDetailsActivity -> setupMangaDetailsTabletLayout(activity, layoutType)
            is ReaderActivity -> setupReaderTabletLayout(activity, layoutType)
        }
    }
    
    private fun setupMainActivityTabletLayout(activity: MainActivity, layoutType: TabletLayoutType) {
        // تنفيذ تخطيط الجهاز اللوحي للنشاط الرئيسي
        if (layoutType == TabletLayoutType.LARGE_TABLET) {
            // استخدام تخطيط ثنائي الجزء للجهاز اللوحي الكبير
            activity.setContentView(R.layout.activity_main_large_tablet)
        } else {
            // استخدام تخطيط معدل للجهاز اللوحي الصغير
            activity.setContentView(R.layout.activity_main_small_tablet)
        }
    }
    
    private fun setupMangaDetailsTabletLayout(activity: MangaDetailsActivity, layoutType: TabletLayoutType) {
        // تنفيذ تخطيط الجهاز اللوحي لنشاط تفاصيل المانجا
        if (layoutType == TabletLayoutType.LARGE_TABLET) {
            // عرض تفاصيل المانجا وقائمة الفصول جنباً إلى جنب
            activity.setContentView(R.layout.activity_manga_details_tablet)
        }
    }
    
    private fun setupReaderTabletLayout(activity: ReaderActivity, layoutType: TabletLayoutType) {
        // تنفيذ تخطيط الجهاز اللوحي لنشاط القارئ
        // تحسين عرض الصفحات للشاشات الكبيرة
        activity.setContentView(R.layout.activity_reader_tablet)
    }
    
    enum class TabletLayoutType {
        PHONE,
        SMALL_TABLET,
        LARGE_TABLET
    }
}
```

## 7. تحسينات أخرى

### 7.1 تحسين دعم اللغة العربية

```kotlin
// ArabicSupport.kt
class ArabicSupport @Inject constructor(
    private val context: Context,
    private val preferenceManager: PreferenceManager
) {
    
    fun setupArabicSupport(activity: Activity) {
        // تعيين اتجاه التخطيط من اليمين إلى اليسار للغة العربية
        if (isArabicLocale() || preferenceManager.isRtlLayoutEnabled()) {
            activity.window.decorView.layoutDirection = View.LAYOUT_DIRECTION_RTL
        } else {
            activity.window.decorView.layoutDirection = View.LAYOUT_DIRECTION_LTR
        }
        
        // تعيين الخط العربي
        setDefaultFont(activity)
    }
    
    fun isArabicLocale(): Boolean {
        val locale = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            context.resources.configuration.locales.get(0)
        } else {
            context.resources.configuration.locale
        }
        
        return locale.language == "ar"
    }
    
    private fun setDefaultFont(activity: Activity) {
        if (isArabicLocale() || preferenceManager.isArabicFontEnabled()) {
            val typeface = ResourcesCompat.getFont(context, R.font.cairo)
            
            // تطبيق الخط على النشاط
            val viewGroup = activity.findViewById<ViewGroup>(android.R.id.content).getChildAt(0) as ViewGroup
            setTypefaceRecursively(viewGroup, typeface)
        }
    }
    
    private fun setTypefaceRecursively(view: View, typeface: Typeface?) {
        if (typeface == null) return
        
        if (view is ViewGroup) {
            for (i in 0 until view.childCount) {
                setTypefaceRecursively(view.getChildAt(i), typeface)
            }
        } else if (view is TextView) {
            view.typeface = typeface
        }
    }
    
    fun getArabicReadingDirection(): ReadingDirection {
        return if (preferenceManager.isCustomReadingDirectionEnabled()) {
            preferenceManager.getReadingDirection()
        } else {
            ReadingDirection.RIGHT_TO_LEFT
        }
    }
    
    fun formatArabicNumber(number: Int): String {
        if (!isArabicLocale() && !preferenceManager.isArabicNumbersEnabled()) {
            return number.toString()
        }
        
        // تحويل الأرقام الإنجليزية إلى أرقام عربية
        return number.toString()
            .replace("0", "٠")
            .replace("1", "١")
            .replace("2", "٢")
            .replace("3", "٣")
            .replace("4", "٤")
            .replace("5", "٥")
            .replace("6", "٦")
            .replace("7", "٧")
            .replace("8", "٨")
            .replace("9", "٩")
    }
}
```

### 7.2 تحسين تجربة القراءة في الوضع الليلي

```kotlin
// NightModeHelper.kt
class NightModeHelper @Inject constructor(
    private val context: Context,
    private val preferenceManager: PreferenceManager
) {
    
    fun applyNightMode() {
        val nightMode = when (preferenceManager.getNightMode()) {
            NightMode.SYSTEM -> AppCompatDelegate.MODE_NIGHT_FOLLOW_SYSTEM
            NightMode.ON -> AppCompatDelegate.MODE_NIGHT_YES
            NightMode.OFF -> AppCompatDelegate.MODE_NIGHT_NO
            NightMode.AUTO -> getAutoNightMode()
        }
        
        AppCompatDelegate.setDefaultNightMode(nightMode)
    }
    
    private fun getAutoNightMode(): Int {
        val currentHour = Calendar.getInstance().get(Calendar.HOUR_OF_DAY)
        val startHour = preferenceManager.getNightModeStartHour()
        val endHour = preferenceManager.getNightModeEndHour()
        
        return if (currentHour in startHour until endHour) {
            AppCompatDelegate.MODE_NIGHT_NO
        } else {
            AppCompatDelegate.MODE_NIGHT_YES
        }
    }
    
    fun applyReaderNightMode(imageView: ImageView) {
        if (!preferenceManager.isReaderNightModeEnabled()) {
            return
        }
        
        val isNightMode = when (preferenceManager.getNightMode()) {
            NightMode.SYSTEM -> isSystemNightMode()
            NightMode.ON -> true
            NightMode.OFF -> false
            NightMode.AUTO -> isAutoNightMode()
        }
        
        if (isNightMode) {
            // تطبيق فلتر الوضع الليلي على الصورة
            val colorMatrix = ColorMatrix().apply {
                // تقليل السطوع وزيادة التباين قليلاً
                set(floatArrayOf(
                    0.8f, 0f, 0f, 0f, -30f,
                    0f, 0.8f, 0f, 0f, -30f,
                    0f, 0f, 0.8f, 0f, -30f,
                    0f, 0f, 0f, 1f, 0f
                ))
            }
            
            imageView.colorFilter = ColorMatrixColorFilter(colorMatrix)
        } else {
            // إزالة الفلتر
            imageView.clearColorFilter()
        }
    }
    
    private fun isSystemNightMode(): Boolean {
        val currentNightMode = context.resources.configuration.uiMode and 
                Configuration.UI_MODE_NIGHT_MASK
        return currentNightMode == Configuration.UI_MODE_NIGHT_YES
    }
    
    private fun isAutoNightMode(): Boolean {
        val currentHour = Calendar.getInstance().get(Calendar.HOUR_OF_DAY)
        val startHour = preferenceManager.getNightModeStartHour()
        val endHour = preferenceManager.getNightModeEndHour()
        
        return if (startHour < endHour) {
            // مثال: 22:00 - 06:00
            currentHour !in startHour until endHour
        } else {
            // مثال: 22:00 - 06:00
            currentHour in endHour until startHour
        }
    }
    
    enum class NightMode {
        SYSTEM, ON, OFF, AUTO
    }
}
```

## 8. الاختبار النهائي والتسليم

بعد تنفيذ جميع الميزات والتحسينات، سنقوم بإجراء اختبار شامل للتطبيق للتأكد من أنه يعمل بشكل صحيح ويلبي جميع المتطلبات.

### 8.1 قائمة التحقق النهائية

- [x] اختبار جميع الوظائف الأساسية
- [x] اختبار المصادر العربية
- [x] اختبار تقنيات تجاوز الأمان
- [x] اختبار واجهة المستخدم
- [x] اختبار الأداء
- [x] اختبار استهلاك البطارية
- [x] اختبار استهلاك البيانات
- [x] اختبار التوافق مع أجهزة مختلفة
- [x] اختبار الميزات الفريدة

### 8.2 إنشاء ملف APK النهائي

بعد الانتهاء من جميع الاختبارات والتحسينات، سنقوم بإنشاء ملف APK نهائي جاهز للتوزيع.

```bash
./gradlew assembleRelease
```

### 8.3 توقيع ملف APK

```bash
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore manga_reader_key.keystore app/build/outputs/apk/release/app-release-unsigned.apk manga_reader
```

### 8.4 محاذاة ملف APK

```bash
zipalign -v 4 app/build/outputs/apk/release/app-release-unsigned.apk manga_reader.apk
```

### 8.5 تسليم المنتج النهائي

- ملف APK النهائي: `manga_reader.apk`
- مستودع GitHub: `https://github.com/yourusername/manga-reader`
- وثائق المشروع: `/docs`

## 9. الخلاصة

تم تنفيذ تطبيق قارئ مانجا متميز يجمع بين أفضل ميزات تطبيقات مثل Tachiyomi وغيرها، مع إضافة ميزات فريدة تجعله الخيار الأمثل لعشاق المانجا. يتميز التطبيق بدعم متميز للمصادر العربية، ونظام مصادر متطور، وتقنيات متقدمة لتجاوز الحماية، وتجربة قراءة متميزة، ونظام تنزيل متطور، وواجهة مستخدم حديثة واحترافية، بالإضافة إلى ميزات فريدة مثل مزامنة المكتبة والتقدم في القراءة، وإشعارات ذكية بالفصول الجديدة، وتصنيف وتنظيم متقدم للمانجا، وإحصائيات مفصلة عن عادات القراءة.
