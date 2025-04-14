# توثيق كود تطبيق قارئ المانجا

هذا الملف يوثق الأجزاء الرئيسية من كود تطبيق قارئ المانجا، ويشرح كيفية عمل المكونات المختلفة وتفاعلها مع بعضها البعض.

## 1. هيكل المشروع

يتبع المشروع نمط هندسة البرمجيات Clean Architecture مع تقسيم التطبيق إلى طبقات:

```
com.mangareader.app/
├── data/                 # طبقة البيانات
│   ├── db/               # قاعدة البيانات المحلية
│   ├── download/         # نظام التنزيل
│   ├── network/          # طلبات الشبكة
│   ├── preferences/      # تفضيلات التطبيق
│   └── repository/       # تنفيذ المستودعات
├── di/                   # حقن التبعية (Dependency Injection)
├── domain/               # طبقة المجال
│   ├── model/            # نماذج البيانات
│   ├── repository/       # واجهات المستودعات
│   └── usecase/          # حالات الاستخدام
├── source/               # مصادر المانجا
│   ├── ar/               # المصادر العربية
│   ├── en/               # المصادر الإنجليزية
│   ├── online/           # المصادر عبر الإنترنت
│   └── local/            # المصادر المحلية
├── ui/                   # واجهة المستخدم
│   ├── browse/           # شاشة التصفح
│   ├── details/          # شاشة تفاصيل المانجا
│   ├── downloads/        # شاشة التنزيلات
│   ├── library/          # شاشة المكتبة
│   ├── reader/           # شاشة القراءة
│   └── settings/         # شاشة الإعدادات
├── util/                 # أدوات مساعدة
│   ├── extension/        # ملحقات Kotlin
│   └── security/         # أدوات الأمان وتجاوز الحماية
├── App.kt                # فئة التطبيق الرئيسية
└── MainActivity.kt       # النشاط الرئيسي
```

## 2. طبقة البيانات (Data Layer)

### 2.1 قاعدة البيانات المحلية

استخدمنا Room لإدارة قاعدة البيانات المحلية:

```kotlin
// AppDatabase.kt
@Database(
    entities = [
        Manga::class,
        Chapter::class,
        Category::class,
        MangaCategory::class,
        Source::class,
        ReadStat::class,
        MangaStat::class,
        CacheEntry::class
    ],
    version = 1,
    exportSchema = true
)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
    abstract fun mangaDao(): MangaDao
    abstract fun chapterDao(): ChapterDao
    abstract fun categoryDao(): CategoryDao
    abstract fun sourceDao(): SourceDao
    abstract fun statisticsDao(): StatisticsDao
    abstract fun cacheDao(): CacheDao
    
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null
        
        fun getInstance(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "manga_reader.db"
                )
                .fallbackToDestructiveMigration()
                .build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

### 2.2 نظام التنزيل

نظام التنزيل يستخدم WorkManager لإدارة تنزيلات المانجا في الخلفية:

```kotlin
// DownloadManager.kt
class DownloadManager @Inject constructor(
    private val context: Context,
    private val mangaRepository: MangaRepository,
    private val chapterRepository: ChapterRepository,
    private val sourceManager: SourceManager,
    private val preferenceManager: PreferenceManager
) {
    
    fun downloadChapter(mangaId: Long, chapterId: Long) {
        val workRequest = OneTimeWorkRequestBuilder<ChapterDownloadWorker>()
            .setInputData(
                workDataOf(
                    ChapterDownloadWorker.KEY_MANGA_ID to mangaId,
                    ChapterDownloadWorker.KEY_CHAPTER_ID to chapterId
                )
            )
            .setConstraints(createConstraints())
            .build()
        
        WorkManager.getInstance(context).enqueue(workRequest)
    }
    
    fun downloadChapters(mangaId: Long, chapterIds: List<Long>) {
        val workRequests = chapterIds.map { chapterId ->
            OneTimeWorkRequestBuilder<ChapterDownloadWorker>()
                .setInputData(
                    workDataOf(
                        ChapterDownloadWorker.KEY_MANGA_ID to mangaId,
                        ChapterDownloadWorker.KEY_CHAPTER_ID to chapterId
                    )
                )
                .setConstraints(createConstraints())
                .build()
        }
        
        WorkManager.getInstance(context).enqueue(workRequests)
    }
    
    fun downloadManga(mangaId: Long) {
        val workRequest = OneTimeWorkRequestBuilder<MangaDownloadWorker>()
            .setInputData(
                workDataOf(
                    MangaDownloadWorker.KEY_MANGA_ID to mangaId
                )
            )
            .setConstraints(createConstraints())
            .build()
        
        WorkManager.getInstance(context).enqueue(workRequest)
    }
    
    fun cancelDownload(chapterId: Long) {
        WorkManager.getInstance(context)
            .cancelAllWorkByTag("download_chapter_$chapterId")
    }
    
    fun cancelAllDownloads() {
        WorkManager.getInstance(context)
            .cancelAllWorkByTag("download")
    }
    
    private fun createConstraints(): Constraints {
        return Constraints.Builder().apply {
            // تنزيل فقط عندما يكون الجهاز متصلاً بالإنترنت
            setRequiredNetworkType(NetworkType.CONNECTED)
            
            // تنزيل فقط عندما يكون الجهاز متصلاً بشبكة Wi-Fi إذا كان ذلك مطلوباً
            if (preferenceManager.isDownloadWifiOnly()) {
                setRequiredNetworkType(NetworkType.UNMETERED)
            }
            
            // تنزيل فقط عندما يكون الجهاز متصلاً بالشاحن إذا كان ذلك مطلوباً
            if (preferenceManager.isDownloadWhileChargingOnly()) {
                setRequiresCharging(true)
            }
        }.build()
    }
}
```

### 2.3 طلبات الشبكة

استخدمنا Retrofit و OkHttp لإدارة طلبات الشبكة:

```kotlin
// NetworkModule.kt
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideOkHttpClient(
        cloudflareInterceptor: CloudflareInterceptor,
        userAgentInterceptor: UserAgentInterceptor,
        dataSaver: DataSaver
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .writeTimeout(30, TimeUnit.SECONDS)
            .addInterceptor(cloudflareInterceptor)
            .addInterceptor(userAgentInterceptor)
            .addInterceptor { chain ->
                val request = chain.request()
                val newRequest = dataSaver.applyDataSaving(request)
                chain.proceed(newRequest)
            }
            .build()
    }
    
    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.mangareader.app/")
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}
```

## 3. طبقة المجال (Domain Layer)

### 3.1 نماذج البيانات

```kotlin
// Manga.kt
@Entity(tableName = "mangas")
data class Manga(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val title: String,
    val url: String,
    val thumbnailUrl: String?,
    val description: String?,
    val author: String?,
    val artist: String?,
    val genre: String?,
    val status: Int,
    val source: Long,
    val favorite: Boolean = false,
    val lastUpdate: Long = 0,
    val nextUpdate: Long = 0,
    val initialized: Boolean = false,
    val viewer: Int = 0,
    val flags: Int = 0
)

// Chapter.kt
@Entity(
    tableName = "chapters",
    foreignKeys = [
        ForeignKey(
            entity = Manga::class,
            parentColumns = ["id"],
            childColumns = ["mangaId"],
            onDelete = ForeignKey.CASCADE
        )
    ],
    indices = [Index(value = ["mangaId"])]
)
data class Chapter(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val mangaId: Long,
    val url: String,
    val name: String,
    val scanlator: String?,
    val read: Boolean = false,
    val bookmark: Boolean = false,
    val lastPageRead: Int = 0,
    val chapterNumber: Float = -1f,
    val sourceOrder: Int = 0,
    val dateFetch: Long = 0,
    val dateUpload: Long = 0,
    val lastRead: Long = 0
)
```

### 3.2 واجهات المستودعات

```kotlin
// MangaRepository.kt
interface MangaRepository {
    suspend fun getManga(id: Long): Manga?
    suspend fun getMangaByUrl(url: String): Manga?
    suspend fun insertManga(manga: Manga): Long
    suspend fun updateManga(manga: Manga): Int
    suspend fun deleteManga(id: Long): Int
    suspend fun getFavorites(): List<Manga>
    suspend fun getMangaCategoriesByCategory(categoryId: Long): List<Manga>
    suspend fun updateMangaCategories(mangaId: Long, categoryIds: List<Long>): Int
}

// ChapterRepository.kt
interface ChapterRepository {
    suspend fun getChapter(id: Long): Chapter?
    suspend fun getChapterByUrl(url: String): Chapter?
    suspend fun getChapters(mangaId: Long): List<Chapter>
    suspend fun insertChapter(chapter: Chapter): Long
    suspend fun insertChapters(chapters: List<Chapter>): List<Long>
    suspend fun updateChapter(chapter: Chapter): Int
    suspend fun deleteChapter(id: Long): Int
    suspend fun getChapterCount(mangaId: Long): Int
    suspend fun getReadChapterCount(mangaId: Long): Int
}
```

## 4. مصادر المانجا (Source Layer)

### 4.1 واجهة المصدر

```kotlin
// Source.kt
interface Source {
    val id: Long
    val name: String
    val lang: String
    
    suspend fun getPopularManga(page: Int): MangasPage
    suspend fun getLatestUpdates(page: Int): MangasPage
    suspend fun search(query: String, page: Int): MangasPage
    suspend fun getMangaDetails(manga: SManga): SManga
    suspend fun getChapterList(manga: SManga): List<SChapter>
    suspend fun getPageList(chapter: SChapter): List<Page>
}
```

### 4.2 مصدر عربي (مانجا سوات)

```kotlin
// MangaSwatSource.kt
class MangaSwatSource : HttpSource() {
    override val id: Long = 1
    override val name: String = "مانجا سوات"
    override val lang: String = "ar"
    override val baseUrl: String = "https://mangaswat.com"
    
    override fun popularMangaRequest(page: Int): Request {
        return GET("$baseUrl/manga/page/$page/?m_orderby=views", headers)
    }
    
    override fun popularMangaParse(response: Response): MangasPage {
        val document = response.asJsoup()
        
        val mangas = document.select("div.manga-card").map { element ->
            SManga.create().apply {
                title = element.select("h3.manga-title a").text()
                url = element.select("h3.manga-title a").attr("href").substringAfter(baseUrl)
                thumbnail_url = element.select("img").attr("src")
            }
        }
        
        val hasNextPage = document.select("div.pagination a.next").isNotEmpty()
        
        return MangasPage(mangas, hasNextPage)
    }
    
    override fun latestUpdatesRequest(page: Int): Request {
        return GET("$baseUrl/manga/page/$page/?m_orderby=latest", headers)
    }
    
    override fun latestUpdatesParse(response: Response): MangasPage {
        return popularMangaParse(response)
    }
    
    override fun searchMangaRequest(page: Int, query: String, filters: FilterList): Request {
        val url = "$baseUrl/?s=$query&post_type=wp-manga&page=$page"
        return GET(url, headers)
    }
    
    override fun searchMangaParse(response: Response): MangasPage {
        val document = response.asJsoup()
        
        val mangas = document.select("div.c-tabs-item__content").map { element ->
            SManga.create().apply {
                title = element.select("div.post-title h3.h4 a").text()
                url = element.select("div.post-title h3.h4 a").attr("href").substringAfter(baseUrl)
                thumbnail_url = element.select("img").attr("src")
            }
        }
        
        val hasNextPage = document.select("div.nav-previous").isNotEmpty()
        
        return MangasPage(mangas, hasNextPage)
    }
    
    override fun mangaDetailsParse(response: Response): SManga {
        val document = response.asJsoup()
        
        return SManga.create().apply {
            title = document.select("div.post-title h1").text()
            thumbnail_url = document.select("div.summary_image img").attr("src")
            description = document.select("div.description-summary div.summary__content").text()
            author = document.select("div.author-content a").text()
            artist = document.select("div.artist-content a").text()
            genre = document.select("div.genres-content a").joinToString { it.text() }
            status = parseStatus(document.select("div.post-status div.summary-content").text())
        }
    }
    
    private fun parseStatus(status: String): Int {
        return when {
            status.contains("مستمرة") -> SManga.ONGOING
            status.contains("مكتملة") -> SManga.COMPLETED
            else -> SManga.UNKNOWN
        }
    }
    
    override fun chapterListParse(response: Response): List<SChapter> {
        val document = response.asJsoup()
        
        return document.select("li.wp-manga-chapter").map { element ->
            SChapter.create().apply {
                name = element.select("a").text()
                url = element.select("a").attr("href").substringAfter(baseUrl)
                date_upload = parseChapterDate(element.select("span.chapter-release-date").text())
            }
        }
    }
    
    private fun parseChapterDate(date: String): Long {
        return try {
            if (date.contains("ago")) {
                val value = date.split(' ')[0].toInt()
                when {
                    date.contains("minutes") -> System.currentTimeMillis() - TimeUnit.MINUTES.toMillis(value.toLong())
                    date.contains("hours") -> System.currentTimeMillis() - TimeUnit.HOURS.toMillis(value.toLong())
                    date.contains("days") -> System.currentTimeMillis() - TimeUnit.DAYS.toMillis(value.toLong())
                    date.contains("weeks") -> System.currentTimeMillis() - TimeUnit.DAYS.toMillis(value.toLong() * 7)
                    date.contains("months") -> System.currentTimeMillis() - TimeUnit.DAYS.toMillis(value.toLong() * 30)
                    date.contains("years") -> System.currentTimeMillis() - TimeUnit.DAYS.toMillis(value.toLong() * 365)
                    else -> System.currentTimeMillis()
                }
            } else {
                try {
                    SimpleDateFormat("yyyy-MM-dd", Locale.US).parse(date)?.time ?: System.currentTimeMillis()
                } catch (e: Exception) {
                    System.currentTimeMillis()
                }
            }
        } catch (e: Exception) {
            System.currentTimeMillis()
        }
    }
    
    override fun pageListParse(response: Response): List<Page> {
        val document = response.asJsoup()
        
        return document.select("div.reading-content img").mapIndexed { i, element ->
            Page(i, "", element.attr("src"))
        }
    }
}
```

## 5. واجهة المستخدم (UI Layer)

### 5.1 النشاط الرئيسي

```kotlin
// MainActivity.kt
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    private lateinit var navController: NavController
    
    @Inject
    lateinit var arabicSupport: ArabicSupport
    
    @Inject
    lateinit var nightModeHelper: NightModeHelper
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // تطبيق الوضع الليلي
        nightModeHelper.applyNightMode()
        
        // تطبيق دعم اللغة العربية
        arabicSupport.setupArabicSupport(this)
        
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

### 5.2 شاشة المكتبة

```kotlin
// LibraryFragment.kt
@AndroidEntryPoint
class LibraryFragment : Fragment() {
    
    private var _binding: FragmentLibraryBinding? = null
    private val binding get() = _binding!!
    
    private val viewModel: LibraryViewModel by viewModels()
    
    private lateinit var mangaAdapter: MangaAdapter
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentLibraryBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        setupRecyclerView()
        setupTabLayout()
        setupFab()
        observeViewModel()
    }
    
    private fun setupRecyclerView() {
        mangaAdapter = MangaAdapter(requireContext()) { manga, view ->
            // التنقل إلى شاشة تفاصيل المانجا
            val extras = FragmentNavigatorExtras(view to "manga_cover_${manga.id}")
            findNavController().navigate(
                LibraryFragmentDirections.actionLibraryFragmentToMangaDetailsFragment(manga.id),
                extras
            )
        }
        
        binding.recyclerView.apply {
            adapter = mangaAdapter
            layoutManager = GridLayoutManager(requireContext(), 2)
            addItemDecoration(GridSpacingItemDecoration(2, 16, true))
        }
    }
    
    private fun setupTabLayout() {
        // مراقبة تغييرات الفئات
        viewModel.categories.observe(viewLifecycleOwner) { categories ->
            binding.tabLayout.removeAllTabs()
            
            // إضافة علامة تبويب "الكل"
            binding.tabLayout.addTab(
                binding.tabLayout.newTab().setText(R.string.all)
            )
            
            // إضافة علامة تبويب لكل فئة
            categories.forEach { category ->
                binding.tabLayout.addTab(
                    binding.tabLayout.newTab().setText(category.name)
                )
            }
        }
        
        // تعيين مستمع لتغيير علامة التبويب
        binding.tabLayout.addOnTabSelectedListener(object : TabLayout.OnTabSelectedListener {
            override fun onTabSelected(tab: TabLayout.Tab) {
                val position = tab.position
                if (position == 0) {
                    // عرض كل المانجا
                    viewModel.loadLibrary()
                } else {
                    // عرض المانجا في الفئة المحددة
                    viewModel.loadLibraryByCategory(position - 1)
                }
            }
            
            override fun onTabUnselected(tab: TabLayout.Tab) {}
            
            override fun onTabReselected(tab: TabLayout.Tab) {}
        })
    }
    
    private fun setupFab() {
        binding.fabSearch.setOnClickListener {
            findNavController().navigate(
                LibraryFragmentDirections.actionLibraryFragmentToSearchFragment()
            )
        }
    }
    
    private fun observeViewModel() {
        viewModel.library.observe(viewLifecycleOwner) { mangas ->
            mangaAdapter.updateMangas(mangas)
            
            // عرض رسالة إذا كانت المكتبة فارغة
            if (mangas.isEmpty()) {
                binding.emptyView.visibility = View.VISIBLE
            } else {
                binding.emptyView.visibility = View.GONE
            }
        }
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

### 5.3 شاشة القراءة

```kotlin
// ReaderActivity.kt
@AndroidEntryPoint
class ReaderActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityReaderBinding
    
    private val viewModel: ReaderViewModel by viewModels()
    
    private var isControlsVisible = false
    
    @Inject
    lateinit var nightModeHelper: NightModeHelper
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // إخفاء شريط الحالة وشريط التنقل
        window.decorView.systemUiVisibility = (View.SYSTEM_UI_FLAG_IMMERSIVE
                or View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                or View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                or View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                or View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                or View.SYSTEM_UI_FLAG_FULLSCREEN)
        
        binding = ActivityReaderBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // الحصول على معرف الفصل من المعلمات
        val chapterId = intent.getLongExtra(EXTRA_CHAPTER_ID, -1L)
        if (chapterId == -1L) {
            finish()
            return
        }
        
        // تحميل الفصل
        viewModel.loadChapter(chapterId)
        
        // إعداد ViewPager
        val adapter = ReaderPagerAdapter(this)
        binding.viewPager.adapter = adapter
        
        // مراقبة صفحات الفصل
        viewModel.pages.observe(this) { pages ->
            adapter.updatePages(pages)
            updatePageInfo(1, pages.size)
        }
        
        // مراقبة اتجاه القراءة
        viewModel.readingDirection.observe(this) { direction ->
            binding.viewPager.orientation = when (direction) {
                ReadingDirection.RIGHT_TO_LEFT -> ViewPager2.ORIENTATION_HORIZONTAL
                ReadingDirection.LEFT_TO_RIGHT -> ViewPager2.ORIENTATION_HORIZONTAL
                ReadingDirection.VERTICAL -> ViewPager2.ORIENTATION_VERTICAL
            }
            
            if (direction == ReadingDirection.RIGHT_TO_LEFT) {
                // عكس اتجاه التمرير للقراءة من اليمين إلى اليسار
                binding.viewPager.layoutDirection = View.LAYOUT_DIRECTION_RTL
            } else {
                binding.viewPager.layoutDirection = View.LAYOUT_DIRECTION_LTR
            }
        }
        
        // إعداد معالج تغيير الصفحة
        binding.viewPager.registerOnPageChangeCallback(object : ViewPager2.OnPageChangeCallback() {
            override fun onPageSelected(position: Int) {
                super.onPageSelected(position)
                updatePageInfo(position + 1, adapter.itemCount)
                viewModel.saveReadingProgress(position)
            }
        })
        
        // إعداد شريط التمرير
        binding.seekBar.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
            override fun onProgressChanged(seekBar: SeekBar, progress: Int, fromUser: Boolean) {
                if (fromUser && adapter.itemCount > 0) {
                    binding.viewPager.currentItem = progress
                }
            }
            
            override fun onStartTrackingTouch(seekBar: SeekBar) {}
            
            override fun onStopTrackingTouch(seekBar: SeekBar) {}
        })
        
        // إعداد معالج النقر لإظهار/إخفاء عناصر التحكم
        binding.viewPager.setOnClickListener {
            toggleControls()
        }
        
        // إعداد شريط الأدوات
        setSupportActionBar(binding.toolbar)
        supportActionBar?.setDisplayHomeAsUpEnabled(true)
        
        // مراقبة عنوان الفصل
        viewModel.chapter.observe(this) { chapter ->
            supportActionBar?.title = chapter.name
        }
    }
    
    private fun updatePageInfo(current: Int, total: Int) {
        binding.textPageInfo.text = getString(R.string.page_info, current, total)
        binding.seekBar.max = total - 1
        binding.seekBar.progress = current - 1
    }
    
    private fun toggleControls() {
        isControlsVisible = !isControlsVisible
        
        val visibility = if (isControlsVisible) View.VISIBLE else View.GONE
        binding.toolbar.visibility = visibility
        binding.bottomControls.visibility = visibility
        
        if (isControlsVisible) {
            // إعادة إظهار شريط الحالة وشريط التنقل
            window.decorView.systemUiVisibility = View.SYSTEM_UI_FLAG_LAYOUT_STABLE
        } else {
            // إخفاء شريط الحالة وشريط التنقل
            window.decorView.systemUiVisibility = (View.SYSTEM_UI_FLAG_IMMERSIVE
                    or View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                    or View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                    or View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                    or View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                    or View.SYSTEM_UI_FLAG_FULLSCREEN)
        }
    }
    
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        return when (item.itemId) {
            android.R.id.home -> {
                finish()
                true
            }
            else -> super.onOptionsItemSelected(item)
        }
    }
    
    companion object {
        const val EXTRA_CHAPTER_ID = "extra_chapter_id"
    }
}
```

## 6. أدوات مساعدة (Util Layer)

### 6.1 تجاوز تقنيات الأمان

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

### 6.2 دعم اللغة العربية

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

## 7. حقن التبعية (Dependency Injection)

استخدمنا Hilt لحقن التبعية:

```kotlin
// App.kt
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

// AppModule.kt
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    
    @Provides
    @Singleton
    fun provideContext(application: Application): Context {
        return application.applicationContext
    }
    
    @Provides
    @Singleton
    fun provideDatabase(context: Context): AppDatabase {
        return AppDatabase.getInstance(context)
    }
    
    @Provides
    @Singleton
    fun provideMangaDao(database: AppDatabase): MangaDao {
        return database.mangaDao()
    }
    
    @Provides
    @Singleton
    fun provideChapterDao(database: AppDatabase): ChapterDao {
        return database.chapterDao()
    }
    
    @Provides
    @Singleton
    fun provideCategoryDao(database: AppDatabase): CategoryDao {
        return database.categoryDao()
    }
    
    @Provides
    @Singleton
    fun provideSourceDao(database: AppDatabase): SourceDao {
        return database.sourceDao()
    }
    
    @Provides
    @Singleton
    fun provideStatisticsDao(database: AppDatabase): StatisticsDao {
        return database.statisticsDao()
    }
    
    @Provides
    @Singleton
    fun provideCacheDao(database: AppDatabase): CacheDao {
        return database.cacheDao()
    }
    
    @Provides
    @Singleton
    fun providePreferenceManager(context: Context): PreferenceManager {
        return PreferenceManager(context)
    }
    
    @Provides
    @Singleton
    fun provideSourceManager(
        context: Context,
        sourceDao: SourceDao
    ): SourceManager {
        return SourceManager(context, sourceDao)
    }
    
    @Provides
    @Singleton
    fun provideMangaRepository(
        mangaDao: MangaDao
    ): MangaRepository {
        return MangaRepositoryImpl(mangaDao)
    }
    
    @Provides
    @Singleton
    fun provideChapterRepository(
        chapterDao: ChapterDao
    ): ChapterRepository {
        return ChapterRepositoryImpl(chapterDao)
    }
    
    @Provides
    @Singleton
    fun provideCategoryRepository(
        categoryDao: CategoryDao
    ): CategoryRepository {
        return CategoryRepositoryImpl(categoryDao)
    }
    
    @Provides
    @Singleton
    fun provideDownloadManager(
        context: Context,
        mangaRepository: MangaRepository,
        chapterRepository: ChapterRepository,
        sourceManager: SourceManager,
        preferenceManager: PreferenceManager
    ): DownloadManager {
        return DownloadManager(
            context,
            mangaRepository,
            chapterRepository,
            sourceManager,
            preferenceManager
        )
    }
}
```

## 8. ملخص

هذا التوثيق يغطي الأجزاء الرئيسية من كود تطبيق قارئ المانجا، ويشرح كيفية عمل المكونات المختلفة وتفاعلها مع بعضها البعض. يتبع التطبيق نمط هندسة البرمجيات Clean Architecture مع تقسيم التطبيق إلى طبقات، ويستخدم تقنيات حديثة مثل Kotlin Coroutines و Hilt و Room و Retrofit و OkHttp و WorkManager و Glide.

يتميز التطبيق بدعم متميز للمصادر العربية، ونظام مصادر متطور، وتقنيات متقدمة لتجاوز الحماية، وتجربة قراءة متميزة، ونظام تنزيل متطور، وواجهة مستخدم حديثة واحترافية، بالإضافة إلى ميزات فريدة مثل مزامنة المكتبة والتقدم في القراءة، وإشعارات ذكية بالفصول الجديدة، وتصنيف وتنظيم متقدم للمانجا، وإحصائيات مفصلة عن عادات القراءة.
