# تنفيذ الوظائف الأساسية لتطبيق قارئ المانجا

سنبدأ بتنفيذ الوظائف الأساسية للتطبيق بناءً على التصميم الذي قمنا بإعداده. سنركز على إنشاء هيكل المشروع وتنفيذ نظام المصادر الذي يدعم المواقع العربية ويتجاوز تقنيات الأمان.

## هيكل المشروع

سنقوم بإنشاء مشروع Android باستخدام Kotlin وهيكل MVVM مع Clean Architecture:

```
app/
├── src/
│   ├── main/
│   │   ├── java/com/mangareader/
│   │   │   ├── data/
│   │   │   │   ├── repository/
│   │   │   │   ├── source/
│   │   │   │   │   ├── online/
│   │   │   │   │   │   ├── ar/
│   │   │   │   │   │   │   ├── MangaSwatSource.kt
│   │   │   │   │   │   │   ├── MangaLionsSource.kt
│   │   │   │   │   │   │   ├── TeamXSource.kt
│   │   │   │   │   │   ├── en/
│   │   │   │   │   ├── local/
│   │   │   │   ├── model/
│   │   │   │   ├── database/
│   │   │   ├── domain/
│   │   │   │   ├── model/
│   │   │   │   ├── repository/
│   │   │   │   ├── usecase/
│   │   │   ├── presentation/
│   │   │   │   ├── ui/
│   │   │   │   │   ├── library/
│   │   │   │   │   ├── browse/
│   │   │   │   │   ├── reader/
│   │   │   │   │   ├── downloads/
│   │   │   │   │   ├── settings/
│   │   │   │   ├── viewmodel/
│   │   │   ├── core/
│   │   │   │   ├── network/
│   │   │   │   │   ├── security/
│   │   │   │   ├── storage/
│   │   │   │   ├── extension/
│   │   │   │   ├── util/
│   │   ├── res/
│   │   ├── AndroidManifest.xml
```

## الملفات الأساسية

### 1. نماذج البيانات (Data Models)

#### `Manga.kt`
```kotlin
data class Manga(
    val id: Long,
    val sourceId: Long,
    val url: String,
    val title: String,
    val thumbnailUrl: String?,
    val description: String?,
    val author: String?,
    val artist: String?,
    val status: Int,
    val genres: List<String>?,
    val initialized: Boolean
)
```

#### `Chapter.kt`
```kotlin
data class Chapter(
    val id: Long,
    val mangaId: Long,
    val url: String,
    val name: String,
    val dateUpload: Long,
    val chapterNumber: Float,
    val scanlator: String?,
    val read: Boolean,
    val bookmark: Boolean,
    val lastPageRead: Int,
    val downloaded: Boolean
)
```

#### `Page.kt`
```kotlin
data class Page(
    val index: Int,
    val url: String?,
    val imageUrl: String?,
    val status: Int
)
```

### 2. واجهة المصدر (Source Interface)

#### `Source.kt`
```kotlin
interface Source {
    val id: Long
    val name: String
    val lang: String
    val supportsLatest: Boolean
    
    suspend fun getPopularManga(page: Int): MangaPageResult
    suspend fun searchManga(query: String, page: Int): MangaPageResult
    suspend fun getMangaDetails(manga: Manga): Manga
    suspend fun getChapterList(manga: Manga): List<Chapter>
    suspend fun getPageList(chapter: Chapter): List<Page>
}
```

#### `HttpSource.kt`
```kotlin
abstract class HttpSource : Source {
    protected abstract val baseUrl: String
    protected abstract val client: OkHttpClient
    
    protected open fun headersBuilder() = Headers.Builder()
        .add("User-Agent", DEFAULT_USER_AGENT)
    
    protected open val headers: Headers by lazy { headersBuilder().build() }
    
    companion object {
        const val DEFAULT_USER_AGENT = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36"
    }
}
```

### 3. تنفيذ المصادر العربية

#### `MangaSwatSource.kt`
```kotlin
class MangaSwatSource(
    private val networkHelper: NetworkHelper,
    private val securityHelper: SecurityHelper
) : HttpSource() {
    override val id: Long = 1001
    override val name: String = "مانجا سوات"
    override val lang: String = "ar"
    override val supportsLatest: Boolean = true
    override val baseUrl: String = "https://mangaswat.com"
    
    override val client: OkHttpClient = networkHelper.client.newBuilder()
        .addInterceptor(securityHelper.cloudflareInterceptor)
        .addInterceptor(securityHelper.rateLimitInterceptor)
        .build()
    
    override fun headersBuilder(): Headers.Builder = super.headersBuilder()
        .add("Referer", baseUrl)
    
    override suspend fun getPopularManga(page: Int): MangaPageResult {
        val url = "$baseUrl/manga/page/$page/?order=popular"
        val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
        
        if (!response.isSuccessful) {
            return MangaPageResult.Error("HTTP error ${response.code}")
        }
        
        val document = Jsoup.parse(response.body?.string() ?: "")
        
        val mangas = document.select("div.manga-card").map { element ->
            Manga(
                id = element.attr("data-id").toLongOrDefault(-1),
                sourceId = id,
                url = element.selectFirst("a")?.attr("href") ?: "",
                title = element.selectFirst("h3.manga-title")?.text() ?: "",
                thumbnailUrl = element.selectFirst("img")?.attr("src"),
                description = null,
                author = null,
                artist = null,
                status = 0,
                genres = null,
                initialized = false
            )
        }
        
        val hasNextPage = document.selectFirst("a.next-page") != null
        
        return MangaPageResult.Success(mangas, hasNextPage)
    }
    
    // تنفيذ باقي الوظائف...
}
```

#### `MangaLionsSource.kt`
```kotlin
class MangaLionsSource(
    private val networkHelper: NetworkHelper,
    private val securityHelper: SecurityHelper
) : HttpSource() {
    override val id: Long = 1002
    override val name: String = "مانجا ليونز"
    override val lang: String = "ar"
    override val supportsLatest: Boolean = true
    override val baseUrl: String = "https://mangalions.com"
    
    override val client: OkHttpClient = networkHelper.client.newBuilder()
        .addInterceptor(securityHelper.cloudflareInterceptor)
        .addInterceptor(securityHelper.rateLimitInterceptor)
        .build()
    
    override fun headersBuilder(): Headers.Builder = super.headersBuilder()
        .add("Referer", baseUrl)
    
    // تنفيذ الوظائف...
}
```

#### `TeamXSource.kt`
```kotlin
class TeamXSource(
    private val networkHelper: NetworkHelper,
    private val securityHelper: SecurityHelper
) : HttpSource() {
    override val id: Long = 1003
    override val name: String = "تيم إكس"
    override val lang: String = "ar"
    override val supportsLatest: Boolean = true
    override val baseUrl: String = "https://teamx.site"
    
    override val client: OkHttpClient = networkHelper.client.newBuilder()
        .addInterceptor(securityHelper.cloudflareInterceptor)
        .addInterceptor(securityHelper.rateLimitInterceptor)
        .build()
    
    override fun headersBuilder(): Headers.Builder = super.headersBuilder()
        .add("Referer", baseUrl)
    
    // تنفيذ الوظائف...
}
```

### 4. تنفيذ تقنيات تجاوز الأمان

#### `SecurityHelper.kt`
```kotlin
class SecurityHelper(private val context: Context) {
    
    val cloudflareInterceptor: CloudflareInterceptor by lazy {
        CloudflareInterceptor(context)
    }
    
    val rateLimitInterceptor: RateLimitInterceptor by lazy {
        RateLimitInterceptor()
    }
    
    val captchaSolver: CaptchaSolver by lazy {
        CaptchaSolver(context)
    }
    
    val antiScrapingBypass: AntiScrapingBypass by lazy {
        AntiScrapingBypass()
    }
}
```

#### `CloudflareInterceptor.kt`
```kotlin
class CloudflareInterceptor(private val context: Context) : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        val response = chain.proceed(request)
        
        // التحقق من وجود حماية CloudFlare
        if (response.code == 503 && response.headers["Server"]?.contains("cloudflare") == true) {
            return handleCloudflare(response, chain)
        }
        
        return response
    }
    
    private fun handleCloudflare(response: Response, chain: Interceptor.Chain): Response {
        // استخراج التحدي من الاستجابة
        val responseBody = response.body?.string() ?: ""
        val challenge = extractChallenge(responseBody)
        
        if (challenge.isEmpty()) {
            return response
        }
        
        // حل التحدي
        val solution = solveChallenge(challenge, response.request.url.host)
        
        // إنشاء طلب جديد مع الحل
        val newRequest = response.request.newBuilder()
            .header("Cookie", "cf_clearance=$solution")
            .build()
        
        // إعادة المحاولة مع الحل
        return chain.proceed(newRequest)
    }
    
    private fun extractChallenge(html: String): String {
        // استخراج تحدي CloudFlare من HTML
        val regex = "setTimeout\\(function\\(\\)\\{\\s+(var s,t,o,p,b,r,e,a,k,i,n,g,f.+?\\r?\\n[\\s\\S]+?a\\.value =.+?)\\r?\\n".toRegex()
        val match = regex.find(html) ?: return ""
        return match.groupValues[1]
    }
    
    private fun solveChallenge(challenge: String, domain: String): String {
        // حل تحدي JavaScript
        // تنفيذ محرك JavaScript لحل التحدي
        return ""
    }
}
```

#### `RateLimitInterceptor.kt`
```kotlin
class RateLimitInterceptor : Interceptor {
    private val requestTimestamps = ConcurrentHashMap<String, Long>()
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        val host = request.url.host
        
        // التحقق من قيود معدل الطلبات
        if (shouldDelayRequest(host)) {
            Thread.sleep(calculateDelay(host))
        }
        
        val response = chain.proceed(request)
        
        // تحديث سجل الطلبات
        requestTimestamps[host] = System.currentTimeMillis()
        
        return response
    }
    
    private fun shouldDelayRequest(host: String): Boolean {
        val lastRequest = requestTimestamps[host] ?: return false
        val elapsed = System.currentTimeMillis() - lastRequest
        return elapsed < MIN_REQUEST_INTERVAL
    }
    
    private fun calculateDelay(host: String): Long {
        val lastRequest = requestTimestamps[host] ?: return 0
        val elapsed = System.currentTimeMillis() - lastRequest
        return if (elapsed < MIN_REQUEST_INTERVAL) {
            MIN_REQUEST_INTERVAL - elapsed
        } else {
            0
        }
    }
    
    companion object {
        private const val MIN_REQUEST_INTERVAL = 2000L // 2 ثانية
    }
}
```

#### `AntiScrapingBypass.kt`
```kotlin
class AntiScrapingBypass {
    private val userAgents = listOf(
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.0.3 Safari/605.1.15",
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:85.0) Gecko/20100101 Firefox/85.0",
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36"
    )
    
    fun createBrowserLikeRequest(url: String): Request {
        return Request.Builder()
            .url(url)
            .header("User-Agent", getRandomUserAgent())
            .header("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8")
            .header("Accept-Language", "ar,en-US;q=0.9,en;q=0.8")
            .header("Referer", getRefererForUrl(url))
            .header("DNT", "1")
            .header("Connection", "keep-alive")
            .header("Upgrade-Insecure-Requests", "1")
            .header("Cache-Control", "max-age=0")
            .build()
    }
    
    private fun getRandomUserAgent(): String {
        return userAgents.random()
    }
    
    private fun getRefererForUrl(url: String): String {
        val uri = Uri.parse(url)
        return "${uri.scheme}://${uri.host}/"
    }
}
```

### 5. مدير المصادر

#### `SourcesManager.kt`
```kotlin
class SourcesManager @Inject constructor(
    private val context: Context,
    private val networkHelper: NetworkHelper,
    private val securityHelper: SecurityHelper,
    private val externalSourcesManager: ExternalSourcesManager,
    private val sourcePreferences: SourcePreferences
) {
    private val _internalSources = listOf(
        MangaSwatSource(networkHelper, securityHelper),
        MangaLionsSource(networkHelper, securityHelper),
        TeamXSource(networkHelper, securityHelper)
    )
    
    private val _sourcesFlow = MutableStateFlow<List<Source>>(_internalSources)
    
    val sources: Flow<List<Source>> = _sourcesFlow
        .combine(sourcePreferences.enabledSourceIds) { sources, enabledIds ->
            sources.filter { it.id in enabledIds }
        }
    
    fun getSource(sourceId: Long): Source? {
        return _sourcesFlow.value.find { it.id == sourceId }
    }
    
    fun getSourcesByLang(lang: String): Flow<List<Source>> {
        return sources.map { sources ->
            sources.filter { it.lang == lang }
        }
    }
    
    suspend fun updateSources() {
        val externalSources = externalSourcesManager.getExternalSources()
        _sourcesFlow.value = _internalSources + externalSources
    }
    
    fun toggleSource(sourceId: Long, enable: Boolean) {
        sourcePreferences.toggleSource(sourceId, enable)
    }
}
```

### 6. نظام المصادر الخارجية

#### `ExternalSourcesManager.kt`
```kotlin
class ExternalSourcesManager @Inject constructor(
    private val context: Context,
    private val networkHelper: NetworkHelper,
    private val securityHelper: SecurityHelper,
    private val sourceInstaller: SourceInstaller,
    private val repositoryPreferences: RepositoryPreferences
) {
    private val _externalSources = MutableStateFlow<List<ExternalSource>>(emptyList())
    val externalSources: Flow<List<ExternalSource>> = _externalSources
    
    private val _repositories = MutableStateFlow<List<SourceRepository>>(emptyList())
    val repositories: Flow<List<SourceRepository>> = _repositories
    
    init {
        loadRepositories()
    }
    
    private fun loadRepositories() {
        viewModelScope.launch {
            val repos = repositoryPreferences.getRepositories()
            _repositories.value = repos
        }
    }
    
    suspend fun addSourceRepository(repoUrl: String): Result<SourceRepository> {
        return try {
            val repo = fetchRepositoryInfo(repoUrl)
            repositoryPreferences.addRepository(repo)
            _repositories.value = _repositories.value + repo
            Result.success(repo)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    private suspend fun fetchRepositoryInfo(repoUrl: String): SourceRepository {
        val response = networkHelper.client.newCall(
            Request.Builder().url("$repoUrl/index.json").build()
        ).await()
        
        if (!response.isSuccessful) {
            throw IOException("Failed to fetch repository info: ${response.code}")
        }
        
        val json = response.body?.string() ?: throw IOException("Empty response")
        return Json.decodeFromString<SourceRepository>(json)
    }
    
    suspend fun getExternalSources(): List<ExternalSource> {
        return _externalSources.value
    }
    
    suspend fun updateExternalSources(): Result<List<ExternalSource>> {
        return try {
            val sources = mutableListOf<ExternalSource>()
            
            _repositories.value.forEach { repo ->
                val repoSources = fetchRepositorySources(repo)
                sources.addAll(repoSources)
            }
            
            _externalSources.value = sources
            Result.success(sources)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    private suspend fun fetchRepositorySources(repo: SourceRepository): List<ExternalSource> {
        val response = networkHelper.client.newCall(
            Request.Builder().url("${repo.url}/sources.json").build()
        ).await()
        
        if (!response.isSuccessful) {
            throw IOException("Failed to fetch repository sources: ${response.code}")
        }
        
        val json = response.body?.string() ?: throw IOException("Empty response")
        return Json.decodeFromString<List<ExternalSource>>(json)
    }
    
    suspend fun installExternalSource(source: ExternalSource): Result<Boolean> {
        return sourceInstaller.installSourceFromRepo(source.repository.url, source.id)
    }
    
    suspend fun uninstallExternalSource(sourceId: String): Result<Boolean> {
        return sourceInstaller.uninstallSource(sourceId)
    }
}
```

### 7. تنفيذ واجهة المستخدم الأساسية

#### `MainActivity.kt`
```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    private val navController: NavController by lazy {
        findNavController(R.id.nav_host_fragment)
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        setupBottomNavigation()
    }
    
    private fun setupBottomNavigation() {
        binding.bottomNavigation.setupWithNavController(navController)
    }
}
```

#### `LibraryFragment.kt`
```kotlin
@AndroidEntryPoint
class LibraryFragment : Fragment() {
    
    private var _binding: FragmentLibraryBinding? = null
    private val binding get() = _binding!!
    
    private val viewModel: LibraryViewModel by viewModels()
    private val mangaAdapter = MangaAdapter()
    
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
        observeViewModel()
    }
    
    private fun setupRecyclerView() {
        binding.recyclerView.apply {
            adapter = mangaAdapter
            layoutManager = GridLayoutManager(requireContext(), 3)
        }
        
        mangaAdapter.setOnItemClickListener { manga ->
            navigateToMangaDetails(manga)
        }
    }
    
    private fun observeViewModel() {
        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.mangaList.collect { mangas ->
                    mangaAdapter.submitList(mangas)
                }
            }
        }
    }
    
    private fun navigateToMangaDetails(manga: Manga) {
        val action = LibraryFragmentDirections.actionLibraryToDetails(manga.id, manga.sourceId)
        findNavController().navigate(action)
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

#### `BrowseFragment.kt`
```kotlin
@AndroidEntryPoint
class BrowseFragment : Fragment() {
    
    private var _binding: FragmentBrowseBinding? = null
    private val binding get() = _binding!!
    
    private val viewModel: BrowseViewModel by viewModels()
    private val sourceAdapter = SourceAdapter()
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentBrowseBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        setupRecyclerView()
        observeViewModel()
        setupLanguageFilter()
    }
    
    private fun setupRecyclerView() {
        binding.recyclerView.apply {
            adapter = sourceAdapter
            layoutManager = LinearLayoutManager(requireContext())
        }
        
        sourceAdapter.setOnItemClickListener { source ->
            navigateToSourceBrowse(source)
        }
    }
    
    private fun observeViewModel() {
        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.sources.collect { sources ->
                    sourceAdapter.submitList(sources)
                }
            }
        }
    }
    
    private fun setupLanguageFilter() {
        binding.languageChipGroup.setOnCheckedChangeListener { _, checkedId ->
            when (checkedId) {
                R.id.chipArabic -> viewModel.filterByLanguage("ar")
                R.id.chipEnglish -> viewModel.filterByLanguage("en")
                R.id.chipAll -> viewModel.filterByLanguage(null)
            }
        }
    }
    
    private fun navigateToSourceBrowse(source: Source) {
        val action = BrowseFragmentDirections.actionBrowseToSource(source.id)
        findNavController().navigate(action)
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

## ملف GitHub Repository

سنقوم بإنشاء ملف README.md للمستودع على GitHub:

```markdown
# MangaReader - قارئ المانجا المتميز

تطبيق قارئ مانجا متطور يجمع بين أفضل ميزات تطبيقات مثل Tachiyomi مع إضافة ميزات فريدة وتحسينات للأداء ودعم متميز للمواقع العربية.

## الميزات الرئيسية

- **نظام مصادر متطور**: دعم للعديد من مواقع المانجا العربية والعالمية
- **تنزيل متقدم**: نظام تنزيل ذكي مع إدارة متقدمة للتخزين
- **تجاوز الحماية**: القدرة على تجاوز تقنيات الأمان الخاصة بمواقع المانجا
- **واجهة مستخدم حديثة**: تصميم عصري وسهل الاستخدام
- **أداء متفوق**: تطبيق خفيف وسريع بدون إعلانات
- **تخصيص متقدم**: خيارات متعددة لتخصيص تجربة القراءة
- **دعم للمصادر الخارجية**: إمكانية إضافة مصادر خارجية عبر روابط repositories

## المصادر المدعومة

### المصادر العربية
- مانجا سوات
- مانجا ليونز
- تيم إكس
- وغيرها الكثير...

### المصادر العالمية
- MangaDex
- MangaSee
- وغيرها...

## التثبيت

قم بتنزيل أحدث إصدار من صفحة [الإصدارات](https://github.com/yourusername/mangareader/releases).

## إضافة مصادر خارجية

يمكنك إضافة مصادر خارجية عبر الخطوات التالية:
1. انتقل إلى الإعدادات > المصادر الخارجية
2. اضغط على "إضافة مستودع"
3. أدخل رابط المستودع
4. اختر المصادر التي ترغب في تثبيتها

## المساهمة في التطوير

نرحب بمساهماتكم في تطوير التطبيق! يمكنكم:
1. عمل Fork للمشروع
2. إنشاء فرع جديد للميزة التي ترغبون في إضافتها
3. تقديم Pull Request

## الترخيص

هذا المشروع مرخص تحت [رخصة MIT](LICENSE).
```

هذا يمثل بداية تنفيذ الوظائف الأساسية للتطبيق، مع التركيز على نظام المصادر الذي يدعم المواقع العربية ويتجاوز تقنيات الأمان. في الخطوات القادمة، سنقوم بتنفيذ المزيد من الوظائف وتحسين التصميم وإنشاء مستودع GitHub للمشروع.
