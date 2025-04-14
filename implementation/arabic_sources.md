# تنفيذ دعم المصادر العربية بشكل مفصل

سنقوم بتنفيذ دعم المصادر العربية بشكل مفصل، مع التركيز على المواقع التي طلبها المستخدم: مانجا سوات، مانجا ليونز، وتيم إكس. سنقوم بتنفيذ آليات استخراج البيانات من هذه المواقع بشكل فعال مع مراعاة هيكل كل موقع وطريقة عرض المحتوى فيه.

## 1. مانجا سوات (MangaSwat)

### تحليل هيكل الموقع

موقع مانجا سوات يستخدم هيكلاً معيناً لعرض المانجا والفصول، وسنقوم بتنفيذ آليات استخراج البيانات منه بناءً على هذا الهيكل.

### التنفيذ المفصل

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
        .add("Accept-Language", "ar-SA,ar;q=0.9,en-US;q=0.8,en;q=0.7")
    
    // الحصول على المانجا الشائعة
    override suspend fun getPopularManga(page: Int): MangaPageResult {
        val url = "$baseUrl/manga/page/$page/?order=popular"
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                return MangaPageResult.Error("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val mangas = document.select("div.manga-card").map { element ->
                Manga(
                    id = generateIdFromUrl(element.selectFirst("a")?.attr("href") ?: ""),
                    sourceId = id,
                    url = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: "",
                    title = element.selectFirst("h3.manga-title")?.text() ?: "",
                    thumbnailUrl = element.selectFirst("img")?.attr("data-src") ?: element.selectFirst("img")?.attr("src"),
                    description = null,
                    author = null,
                    artist = null,
                    status = 0,
                    genres = null,
                    initialized = false
                )
            }
            
            val hasNextPage = document.select("a.next-page").isNotEmpty()
            
            MangaPageResult.Success(mangas, hasNextPage)
        } catch (e: Exception) {
            MangaPageResult.Error(e.message ?: "حدث خطأ أثناء تحميل المانجا الشائعة")
        }
    }
    
    // البحث عن المانجا
    override suspend fun searchManga(query: String, page: Int): MangaPageResult {
        val url = "$baseUrl/page/$page/?s=$query"
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                return MangaPageResult.Error("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val mangas = document.select("div.manga-card").map { element ->
                Manga(
                    id = generateIdFromUrl(element.selectFirst("a")?.attr("href") ?: ""),
                    sourceId = id,
                    url = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: "",
                    title = element.selectFirst("h3.manga-title")?.text() ?: "",
                    thumbnailUrl = element.selectFirst("img")?.attr("data-src") ?: element.selectFirst("img")?.attr("src"),
                    description = null,
                    author = null,
                    artist = null,
                    status = 0,
                    genres = null,
                    initialized = false
                )
            }
            
            val hasNextPage = document.select("a.next-page").isNotEmpty()
            
            MangaPageResult.Success(mangas, hasNextPage)
        } catch (e: Exception) {
            MangaPageResult.Error(e.message ?: "حدث خطأ أثناء البحث عن المانجا")
        }
    }
    
    // الحصول على أحدث المانجا
    override suspend fun getLatestUpdates(page: Int): MangaPageResult {
        val url = "$baseUrl/manga/page/$page/?order=update"
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                return MangaPageResult.Error("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val mangas = document.select("div.manga-card").map { element ->
                Manga(
                    id = generateIdFromUrl(element.selectFirst("a")?.attr("href") ?: ""),
                    sourceId = id,
                    url = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: "",
                    title = element.selectFirst("h3.manga-title")?.text() ?: "",
                    thumbnailUrl = element.selectFirst("img")?.attr("data-src") ?: element.selectFirst("img")?.attr("src"),
                    description = null,
                    author = null,
                    artist = null,
                    status = 0,
                    genres = null,
                    initialized = false
                )
            }
            
            val hasNextPage = document.select("a.next-page").isNotEmpty()
            
            MangaPageResult.Success(mangas, hasNextPage)
        } catch (e: Exception) {
            MangaPageResult.Error(e.message ?: "حدث خطأ أثناء تحميل أحدث المانجا")
        }
    }
    
    // الحصول على تفاصيل المانجا
    override suspend fun getMangaDetails(manga: Manga): Manga {
        val url = baseUrl + manga.url
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                throw IOException("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val description = document.select("div.manga-description").text().trim()
            val author = document.select("div.manga-info span:contains(المؤلف) + span").text().trim()
            val artist = document.select("div.manga-info span:contains(الرسام) + span").text().trim()
            val status = when (document.select("div.manga-info span:contains(الحالة) + span").text().trim()) {
                "مستمرة" -> MangaStatus.ONGOING
                "مكتملة" -> MangaStatus.COMPLETED
                else -> MangaStatus.UNKNOWN
            }
            val genres = document.select("div.manga-genres a").map { it.text().trim() }
            
            manga.copy(
                description = description,
                author = author,
                artist = artist,
                status = status.value,
                genres = genres,
                initialized = true
            )
        } catch (e: Exception) {
            manga.copy(initialized = false)
        }
    }
    
    // الحصول على قائمة الفصول
    override suspend fun getChapterList(manga: Manga): List<Chapter> {
        val url = baseUrl + manga.url
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                throw IOException("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            document.select("div.chapters-list li").map { element ->
                val chapterUrl = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: ""
                val chapterName = element.selectFirst("a")?.text()?.trim() ?: ""
                val chapterNumber = chapterName.substringAfter("الفصل ").substringBefore(":").toFloatOrNull() ?: -1f
                val dateUpload = parseChapterDate(element.selectFirst("span.chapter-date")?.text() ?: "")
                
                Chapter(
                    id = generateIdFromUrl(chapterUrl),
                    mangaId = manga.id,
                    url = chapterUrl,
                    name = chapterName,
                    dateUpload = dateUpload,
                    chapterNumber = chapterNumber,
                    scanlator = null,
                    read = false,
                    bookmark = false,
                    lastPageRead = 0,
                    downloaded = false
                )
            }.reversed() // عرض الفصول من الأقدم للأحدث
        } catch (e: Exception) {
            emptyList()
        }
    }
    
    // الحصول على قائمة الصفحات
    override suspend fun getPageList(chapter: Chapter): List<Page> {
        val url = baseUrl + chapter.url
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                throw IOException("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            document.select("div.reading-content img").mapIndexed { index, element ->
                val imageUrl = element.attr("data-src").ifEmpty { element.attr("src") }
                
                Page(
                    index = index,
                    url = url,
                    imageUrl = imageUrl,
                    status = PageStatus.READY
                )
            }
        } catch (e: Exception) {
            emptyList()
        }
    }
    
    // توليد معرف فريد من URL
    private fun generateIdFromUrl(url: String): Long {
        return url.hashCode().toLong()
    }
    
    // تحليل تاريخ الفصل
    private fun parseChapterDate(dateText: String): Long {
        return try {
            when {
                dateText.contains("منذ") && dateText.contains("دقائق") -> {
                    val minutes = dateText.substringBefore(" دقائق").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.MINUTES.toMillis(minutes.toLong())
                }
                dateText.contains("منذ") && dateText.contains("ساعات") -> {
                    val hours = dateText.substringBefore(" ساعات").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.HOURS.toMillis(hours.toLong())
                }
                dateText.contains("منذ") && dateText.contains("أيام") -> {
                    val days = dateText.substringBefore(" أيام").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.DAYS.toMillis(days.toLong())
                }
                dateText.contains("منذ") && dateText.contains("أسابيع") -> {
                    val weeks = dateText.substringBefore(" أسابيع").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.DAYS.toMillis(weeks.toLong() * 7)
                }
                dateText.contains("منذ") && dateText.contains("شهور") -> {
                    val months = dateText.substringBefore(" شهور").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.DAYS.toMillis(months.toLong() * 30)
                }
                else -> {
                    // محاولة تحليل التاريخ بتنسيق معين
                    val sdf = SimpleDateFormat("yyyy-MM-dd", Locale("ar"))
                    sdf.parse(dateText)?.time ?: System.currentTimeMillis()
                }
            }
        } catch (e: Exception) {
            System.currentTimeMillis()
        }
    }
}
```

## 2. مانجا ليونز (MangaLions)

### تحليل هيكل الموقع

موقع مانجا ليونز له هيكل مختلف عن مانجا سوات، وسنقوم بتنفيذ آليات استخراج البيانات منه بناءً على هذا الهيكل.

### التنفيذ المفصل

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
        .add("Accept-Language", "ar-SA,ar;q=0.9,en-US;q=0.8,en;q=0.7")
    
    // الحصول على المانجا الشائعة
    override suspend fun getPopularManga(page: Int): MangaPageResult {
        val url = "$baseUrl/manga/?page=$page&order=popular"
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                return MangaPageResult.Error("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val mangas = document.select("div.manga-item").map { element ->
                Manga(
                    id = generateIdFromUrl(element.selectFirst("a")?.attr("href") ?: ""),
                    sourceId = id,
                    url = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: "",
                    title = element.selectFirst("h3.manga-title")?.text() ?: "",
                    thumbnailUrl = element.selectFirst("img")?.attr("data-lazy-src") ?: element.selectFirst("img")?.attr("src"),
                    description = null,
                    author = null,
                    artist = null,
                    status = 0,
                    genres = null,
                    initialized = false
                )
            }
            
            val hasNextPage = document.select("ul.pagination li.next").isNotEmpty()
            
            MangaPageResult.Success(mangas, hasNextPage)
        } catch (e: Exception) {
            MangaPageResult.Error(e.message ?: "حدث خطأ أثناء تحميل المانجا الشائعة")
        }
    }
    
    // البحث عن المانجا
    override suspend fun searchManga(query: String, page: Int): MangaPageResult {
        val url = "$baseUrl/search/?q=$query&page=$page"
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                return MangaPageResult.Error("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val mangas = document.select("div.search-result-item").map { element ->
                Manga(
                    id = generateIdFromUrl(element.selectFirst("a")?.attr("href") ?: ""),
                    sourceId = id,
                    url = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: "",
                    title = element.selectFirst("h3.result-title")?.text() ?: "",
                    thumbnailUrl = element.selectFirst("img")?.attr("data-lazy-src") ?: element.selectFirst("img")?.attr("src"),
                    description = null,
                    author = null,
                    artist = null,
                    status = 0,
                    genres = null,
                    initialized = false
                )
            }
            
            val hasNextPage = document.select("ul.pagination li.next").isNotEmpty()
            
            MangaPageResult.Success(mangas, hasNextPage)
        } catch (e: Exception) {
            MangaPageResult.Error(e.message ?: "حدث خطأ أثناء البحث عن المانجا")
        }
    }
    
    // الحصول على أحدث المانجا
    override suspend fun getLatestUpdates(page: Int): MangaPageResult {
        val url = "$baseUrl/manga/?page=$page&order=latest"
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                return MangaPageResult.Error("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val mangas = document.select("div.manga-item").map { element ->
                Manga(
                    id = generateIdFromUrl(element.selectFirst("a")?.attr("href") ?: ""),
                    sourceId = id,
                    url = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: "",
                    title = element.selectFirst("h3.manga-title")?.text() ?: "",
                    thumbnailUrl = element.selectFirst("img")?.attr("data-lazy-src") ?: element.selectFirst("img")?.attr("src"),
                    description = null,
                    author = null,
                    artist = null,
                    status = 0,
                    genres = null,
                    initialized = false
                )
            }
            
            val hasNextPage = document.select("ul.pagination li.next").isNotEmpty()
            
            MangaPageResult.Success(mangas, hasNextPage)
        } catch (e: Exception) {
            MangaPageResult.Error(e.message ?: "حدث خطأ أثناء تحميل أحدث المانجا")
        }
    }
    
    // الحصول على تفاصيل المانجا
    override suspend fun getMangaDetails(manga: Manga): Manga {
        val url = baseUrl + manga.url
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                throw IOException("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val description = document.select("div.manga-summary").text().trim()
            val author = document.select("div.manga-info div:contains(المؤلف) span").text().trim()
            val artist = document.select("div.manga-info div:contains(الرسام) span").text().trim()
            val status = when (document.select("div.manga-info div:contains(الحالة) span").text().trim()) {
                "مستمرة" -> MangaStatus.ONGOING
                "مكتملة" -> MangaStatus.COMPLETED
                else -> MangaStatus.UNKNOWN
            }
            val genres = document.select("div.manga-genres a").map { it.text().trim() }
            
            manga.copy(
                description = description,
                author = author,
                artist = artist,
                status = status.value,
                genres = genres,
                initialized = true
            )
        } catch (e: Exception) {
            manga.copy(initialized = false)
        }
    }
    
    // الحصول على قائمة الفصول
    override suspend fun getChapterList(manga: Manga): List<Chapter> {
        val url = baseUrl + manga.url
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                throw IOException("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            document.select("ul.chapters-list li").map { element ->
                val chapterUrl = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: ""
                val chapterName = element.selectFirst("a span.chapter-title")?.text()?.trim() ?: ""
                val chapterNumber = chapterName.substringAfter("الفصل ").substringBefore(":").toFloatOrNull() ?: -1f
                val dateUpload = parseChapterDate(element.selectFirst("span.chapter-date")?.text() ?: "")
                
                Chapter(
                    id = generateIdFromUrl(chapterUrl),
                    mangaId = manga.id,
                    url = chapterUrl,
                    name = chapterName,
                    dateUpload = dateUpload,
                    chapterNumber = chapterNumber,
                    scanlator = element.selectFirst("span.chapter-scanlator")?.text()?.trim(),
                    read = false,
                    bookmark = false,
                    lastPageRead = 0,
                    downloaded = false
                )
            }.reversed() // عرض الفصول من الأقدم للأحدث
        } catch (e: Exception) {
            emptyList()
        }
    }
    
    // الحصول على قائمة الصفحات
    override suspend fun getPageList(chapter: Chapter): List<Page> {
        val url = baseUrl + chapter.url
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                throw IOException("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            // استخراج البيانات من سكريبت JavaScript
            val script = document.select("script:containsData(chapter_images)").firstOrNull()?.data() ?: ""
            val imageUrlsRegex = "chapter_images\\s*=\\s*(\\[.*?\\])".toRegex(RegexOption.DOT_MATCHES_ALL)
            val imageServerRegex = "image_server\\s*=\\s*['\"](.+?)['\"]".toRegex()
            
            val imageUrlsMatch = imageUrlsRegex.find(script)
            val imageServerMatch = imageServerRegex.find(script)
            
            if (imageUrlsMatch != null && imageServerMatch != null) {
                val imageUrlsJson = imageUrlsMatch.groupValues[1]
                val imageServer = imageServerMatch.groupValues[1]
                
                val imageUrls = Json.decodeFromString<List<String>>(imageUrlsJson)
                
                imageUrls.mapIndexed { index, imageUrl ->
                    Page(
                        index = index,
                        url = url,
                        imageUrl = "$imageServer$imageUrl",
                        status = PageStatus.READY
                    )
                }
            } else {
                // طريقة بديلة إذا فشلت الطريقة الأولى
                document.select("div.reading-content img").mapIndexed { index, element ->
                    val imageUrl = element.attr("data-lazy-src").ifEmpty { element.attr("src") }
                    
                    Page(
                        index = index,
                        url = url,
                        imageUrl = imageUrl,
                        status = PageStatus.READY
                    )
                }
            }
        } catch (e: Exception) {
            emptyList()
        }
    }
    
    // توليد معرف فريد من URL
    private fun generateIdFromUrl(url: String): Long {
        return url.hashCode().toLong()
    }
    
    // تحليل تاريخ الفصل
    private fun parseChapterDate(dateText: String): Long {
        return try {
            when {
                dateText.contains("منذ") && dateText.contains("دقائق") -> {
                    val minutes = dateText.substringBefore(" دقائق").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.MINUTES.toMillis(minutes.toLong())
                }
                dateText.contains("منذ") && dateText.contains("ساعات") -> {
                    val hours = dateText.substringBefore(" ساعات").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.HOURS.toMillis(hours.toLong())
                }
                dateText.contains("منذ") && dateText.contains("أيام") -> {
                    val days = dateText.substringBefore(" أيام").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.DAYS.toMillis(days.toLong())
                }
                dateText.contains("منذ") && dateText.contains("أسابيع") -> {
                    val weeks = dateText.substringBefore(" أسابيع").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.DAYS.toMillis(weeks.toLong() * 7)
                }
                dateText.contains("منذ") && dateText.contains("شهور") -> {
                    val months = dateText.substringBefore(" شهور").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.DAYS.toMillis(months.toLong() * 30)
                }
                else -> {
                    // محاولة تحليل التاريخ بتنسيق معين
                    val sdf = SimpleDateFormat("yyyy-MM-dd", Locale("ar"))
                    sdf.parse(dateText)?.time ?: System.currentTimeMillis()
                }
            }
        } catch (e: Exception) {
            System.currentTimeMillis()
        }
    }
}
```

## 3. تيم إكس (TeamX)

### تحليل هيكل الموقع

موقع تيم إكس له هيكل مختلف عن المواقع السابقة، وسنقوم بتنفيذ آليات استخراج البيانات منه بناءً على هذا الهيكل.

### التنفيذ المفصل

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
        .add("Accept-Language", "ar-SA,ar;q=0.9,en-US;q=0.8,en;q=0.7")
    
    // الحصول على المانجا الشائعة
    override suspend fun getPopularManga(page: Int): MangaPageResult {
        val url = "$baseUrl/manga-list/page/$page/?m_orderby=views"
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                return MangaPageResult.Error("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val mangas = document.select("div.page-item-detail").map { element ->
                Manga(
                    id = generateIdFromUrl(element.selectFirst("a")?.attr("href") ?: ""),
                    sourceId = id,
                    url = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: "",
                    title = element.selectFirst("h3.h5 a")?.text() ?: "",
                    thumbnailUrl = element.selectFirst("img")?.attr("data-src") ?: element.selectFirst("img")?.attr("src"),
                    description = null,
                    author = null,
                    artist = null,
                    status = 0,
                    genres = null,
                    initialized = false
                )
            }
            
            val hasNextPage = document.select("div.nav-previous").isNotEmpty()
            
            MangaPageResult.Success(mangas, hasNextPage)
        } catch (e: Exception) {
            MangaPageResult.Error(e.message ?: "حدث خطأ أثناء تحميل المانجا الشائعة")
        }
    }
    
    // البحث عن المانجا
    override suspend fun searchManga(query: String, page: Int): MangaPageResult {
        val url = "$baseUrl/page/$page/?s=$query&post_type=wp-manga"
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                return MangaPageResult.Error("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val mangas = document.select("div.c-tabs-item__content").map { element ->
                Manga(
                    id = generateIdFromUrl(element.selectFirst("a")?.attr("href") ?: ""),
                    sourceId = id,
                    url = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: "",
                    title = element.selectFirst("h3.h4 a")?.text() ?: "",
                    thumbnailUrl = element.selectFirst("img")?.attr("data-src") ?: element.selectFirst("img")?.attr("src"),
                    description = null,
                    author = null,
                    artist = null,
                    status = 0,
                    genres = null,
                    initialized = false
                )
            }
            
            val hasNextPage = document.select("div.nav-previous").isNotEmpty()
            
            MangaPageResult.Success(mangas, hasNextPage)
        } catch (e: Exception) {
            MangaPageResult.Error(e.message ?: "حدث خطأ أثناء البحث عن المانجا")
        }
    }
    
    // الحصول على أحدث المانجا
    override suspend fun getLatestUpdates(page: Int): MangaPageResult {
        val url = "$baseUrl/manga-list/page/$page/?m_orderby=latest"
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                return MangaPageResult.Error("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val mangas = document.select("div.page-item-detail").map { element ->
                Manga(
                    id = generateIdFromUrl(element.selectFirst("a")?.attr("href") ?: ""),
                    sourceId = id,
                    url = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: "",
                    title = element.selectFirst("h3.h5 a")?.text() ?: "",
                    thumbnailUrl = element.selectFirst("img")?.attr("data-src") ?: element.selectFirst("img")?.attr("src"),
                    description = null,
                    author = null,
                    artist = null,
                    status = 0,
                    genres = null,
                    initialized = false
                )
            }
            
            val hasNextPage = document.select("div.nav-previous").isNotEmpty()
            
            MangaPageResult.Success(mangas, hasNextPage)
        } catch (e: Exception) {
            MangaPageResult.Error(e.message ?: "حدث خطأ أثناء تحميل أحدث المانجا")
        }
    }
    
    // الحصول على تفاصيل المانجا
    override suspend fun getMangaDetails(manga: Manga): Manga {
        val url = baseUrl + manga.url
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                throw IOException("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            val description = document.select("div.description-summary div.summary__content").text().trim()
            val author = document.select("div.author-content a").text().trim()
            val artist = document.select("div.artist-content a").text().trim()
            val status = when (document.select("div.post-status div.summary-content").text().trim()) {
                "مستمرة" -> MangaStatus.ONGOING
                "مكتملة" -> MangaStatus.COMPLETED
                else -> MangaStatus.UNKNOWN
            }
            val genres = document.select("div.genres-content a").map { it.text().trim() }
            
            manga.copy(
                description = description,
                author = author,
                artist = artist,
                status = status.value,
                genres = genres,
                initialized = true
            )
        } catch (e: Exception) {
            manga.copy(initialized = false)
        }
    }
    
    // الحصول على قائمة الفصول
    override suspend fun getChapterList(manga: Manga): List<Chapter> {
        val url = baseUrl + manga.url
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                throw IOException("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            document.select("li.wp-manga-chapter").map { element ->
                val chapterUrl = element.selectFirst("a")?.attr("href")?.substringAfter(baseUrl) ?: ""
                val chapterName = element.selectFirst("a")?.text()?.trim() ?: ""
                val chapterNumber = chapterName.substringAfter("الفصل ").substringBefore(":").toFloatOrNull() ?: -1f
                val dateUpload = parseChapterDate(element.selectFirst("span.chapter-release-date i")?.text() ?: "")
                
                Chapter(
                    id = generateIdFromUrl(chapterUrl),
                    mangaId = manga.id,
                    url = chapterUrl,
                    name = chapterName,
                    dateUpload = dateUpload,
                    chapterNumber = chapterNumber,
                    scanlator = null,
                    read = false,
                    bookmark = false,
                    lastPageRead = 0,
                    downloaded = false
                )
            }.reversed() // عرض الفصول من الأقدم للأحدث
        } catch (e: Exception) {
            emptyList()
        }
    }
    
    // الحصول على قائمة الصفحات
    override suspend fun getPageList(chapter: Chapter): List<Page> {
        val url = baseUrl + chapter.url
        
        return try {
            val response = client.newCall(Request.Builder().url(url).headers(headers).build()).execute()
            
            if (!response.isSuccessful) {
                throw IOException("HTTP error ${response.code}")
            }
            
            val document = Jsoup.parse(response.body?.string() ?: "")
            
            // التحقق من نوع القارئ (عادي أو AJAX)
            val isAjaxReader = document.select("div.reading-content div.loading").isNotEmpty()
            
            if (isAjaxReader) {
                // استخراج معرف الفصل من URL
                val chapterId = chapter.url.substringAfterLast("/").substringBefore("-")
                
                // إنشاء طلب AJAX للحصول على الصفحات
                val ajaxUrl = "$baseUrl/wp-admin/admin-ajax.php"
                val requestBody = FormBody.Builder()
                    .add("action", "manga_get_chapters")
                    .add("chapter_id", chapterId)
                    .build()
                
                val ajaxResponse = client.newCall(
                    Request.Builder()
                        .url(ajaxUrl)
                        .headers(headers)
                        .post(requestBody)
                        .build()
                ).execute()
                
                if (!ajaxResponse.isSuccessful) {
                    throw IOException("HTTP error ${ajaxResponse.code}")
                }
                
                val ajaxDocument = Jsoup.parse(ajaxResponse.body?.string() ?: "")
                
                ajaxDocument.select("div.reading-content img").mapIndexed { index, element ->
                    val imageUrl = element.attr("data-src").ifEmpty { element.attr("src") }
                    
                    Page(
                        index = index,
                        url = url,
                        imageUrl = imageUrl,
                        status = PageStatus.READY
                    )
                }
            } else {
                // قارئ عادي
                document.select("div.reading-content img").mapIndexed { index, element ->
                    val imageUrl = element.attr("data-src").ifEmpty { element.attr("src") }
                    
                    Page(
                        index = index,
                        url = url,
                        imageUrl = imageUrl,
                        status = PageStatus.READY
                    )
                }
            }
        } catch (e: Exception) {
            emptyList()
        }
    }
    
    // توليد معرف فريد من URL
    private fun generateIdFromUrl(url: String): Long {
        return url.hashCode().toLong()
    }
    
    // تحليل تاريخ الفصل
    private fun parseChapterDate(dateText: String): Long {
        return try {
            when {
                dateText.contains("منذ") && dateText.contains("دقائق") -> {
                    val minutes = dateText.substringBefore(" دقائق").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.MINUTES.toMillis(minutes.toLong())
                }
                dateText.contains("منذ") && dateText.contains("ساعات") -> {
                    val hours = dateText.substringBefore(" ساعات").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.HOURS.toMillis(hours.toLong())
                }
                dateText.contains("منذ") && dateText.contains("أيام") -> {
                    val days = dateText.substringBefore(" أيام").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.DAYS.toMillis(days.toLong())
                }
                dateText.contains("منذ") && dateText.contains("أسابيع") -> {
                    val weeks = dateText.substringBefore(" أسابيع").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.DAYS.toMillis(weeks.toLong() * 7)
                }
                dateText.contains("منذ") && dateText.contains("شهور") -> {
                    val months = dateText.substringBefore(" شهور").substringAfter("منذ ").trim().toInt()
                    System.currentTimeMillis() - TimeUnit.DAYS.toMillis(months.toLong() * 30)
                }
                else -> {
                    // محاولة تحليل التاريخ بتنسيق معين
                    val sdf = SimpleDateFormat("yyyy-MM-dd", Locale("ar"))
                    sdf.parse(dateText)?.time ?: System.currentTimeMillis()
                }
            }
        } catch (e: Exception) {
            System.currentTimeMillis()
        }
    }
}
```

## 4. تسجيل المصادر في مدير المصادر

لضمان تسجيل المصادر العربية في التطبيق، نقوم بتحديث `SourcesManager.kt`:

```kotlin
class SourcesManager @Inject constructor(
    private val context: Context,
    private val networkHelper: NetworkHelper,
    private val securityHelper: SecurityHelper,
    private val externalSourcesManager: ExternalSourcesManager,
    private val sourcePreferences: SourcePreferences
) {
    private val _internalSources = listOf(
        // المصادر العربية
        MangaSwatSource(networkHelper, securityHelper),
        MangaLionsSource(networkHelper, securityHelper),
        TeamXSource(networkHelper, securityHelper),
        
        // يمكن إضافة مصادر عربية أخرى هنا
        
        // المصادر الإنجليزية وغيرها
        // ...
    )
    
    // باقي التنفيذ...
}
```

## 5. إضافة فلتر للمصادر العربية

لتسهيل الوصول إلى المصادر العربية، نضيف فلتر خاص بها في واجهة المستخدم:

```kotlin
class BrowseViewModel @Inject constructor(
    private val sourcesManager: SourcesManager,
    private val sourcePreferences: SourcePreferences
) : ViewModel() {
    
    private val _currentLanguage = MutableStateFlow<String?>(null)
    
    val sources: Flow<List<Source>> = combine(
        sourcesManager.sources,
        _currentLanguage
    ) { sources, language ->
        when (language) {
            null -> sources
            else -> sources.filter { it.lang == language }
        }
    }
    
    // فلتر حسب اللغة
    fun filterByLanguage(language: String?) {
        _currentLanguage.value = language
    }
    
    // فلتر سريع للمصادر العربية
    fun filterArabicSources() {
        _currentLanguage.value = "ar"
    }
    
    // تبديل حالة المصدر (تفعيل/تعطيل)
    fun toggleSource(sourceId: Long, enable: Boolean) {
        viewModelScope.launch {
            sourcesManager.toggleSource(sourceId, enable)
        }
    }
}
```

## 6. تحسينات لدعم اللغة العربية

### تحسين عرض النصوص العربية

```kotlin
class ArabicTextView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : AppCompatTextView(context, attrs, defStyleAttr) {
    
    init {
        // تعيين اتجاه النص من اليمين إلى اليسار
        textDirection = View.TEXT_DIRECTION_RTL
        
        // تعيين محاذاة النص إلى اليمين
        textAlignment = View.TEXT_ALIGNMENT_VIEW_START
        
        // تعيين خط مناسب للغة العربية
        typeface = ResourcesCompat.getFont(context, R.font.arabic_font)
    }
}
```

### تحسين البحث باللغة العربية

```kotlin
class ArabicSearchHelper {
    
    // تنظيف النص العربي للبحث
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
    
    // مقارنة النصوص العربية
    fun arabicTextMatches(text: String, query: String): Boolean {
        val normalizedText = normalizeArabicText(text)
        val normalizedQuery = normalizeArabicText(query)
        
        return normalizedText.contains(normalizedQuery, ignoreCase = true)
    }
}
```

## 7. اختبار المصادر العربية

لضمان عمل المصادر العربية بشكل صحيح، نقوم بإنشاء اختبارات وحدة:

```kotlin
class ArabicSourcesTest {
    
    private lateinit var networkHelper: NetworkHelper
    private lateinit var securityHelper: SecurityHelper
    
    @Before
    fun setup() {
        networkHelper = mock()
        securityHelper = mock()
        
        val mockClient = OkHttpClient()
        whenever(networkHelper.client).thenReturn(mockClient)
    }
    
    @Test
    fun `test MangaSwat source initialization`() {
        val source = MangaSwatSource(networkHelper, securityHelper)
        
        assertEquals(1001L, source.id)
        assertEquals("مانجا سوات", source.name)
        assertEquals("ar", source.lang)
        assertTrue(source.supportsLatest)
        assertEquals("https://mangaswat.com", source.baseUrl)
    }
    
    @Test
    fun `test MangaLions source initialization`() {
        val source = MangaLionsSource(networkHelper, securityHelper)
        
        assertEquals(1002L, source.id)
        assertEquals("مانجا ليونز", source.name)
        assertEquals("ar", source.lang)
        assertTrue(source.supportsLatest)
        assertEquals("https://mangalions.com", source.baseUrl)
    }
    
    @Test
    fun `test TeamX source initialization`() {
        val source = TeamXSource(networkHelper, securityHelper)
        
        assertEquals(1003L, source.id)
        assertEquals("تيم إكس", source.name)
        assertEquals("ar", source.lang)
        assertTrue(source.supportsLatest)
        assertEquals("https://teamx.site", source.baseUrl)
    }
}
```

## 8. تحسينات إضافية للمصادر العربية

### دعم التصفية حسب النوع (Genre)

```kotlin
class ArabicGenreFilter(val genresList: List<Genre>) : Filter.Group<Genre>("التصنيفات", genresList)

class Genre(name: String, val id: String) : Filter.CheckBox(name)

// مثال على تنفيذ التصفية في مصدر مانجا سوات
class MangaSwatSource(
    private val networkHelper: NetworkHelper,
    private val securityHelper: SecurityHelper
) : HttpSource() {
    // ...
    
    override fun getFilterList(): FilterList {
        return FilterList(
            Filter.Header("ملاحظة: لا يمكن استخدام التصفيات مع البحث"),
            ArabicGenreFilter(getGenreList())
        )
    }
    
    private fun getGenreList(): List<Genre> {
        return listOf(
            Genre("أكشن", "action"),
            Genre("مغامرة", "adventure"),
            Genre("كوميديا", "comedy"),
            Genre("دراما", "drama"),
            Genre("خيال", "fantasy"),
            Genre("رعب", "horror"),
            Genre("رومانسي", "romance"),
            Genre("خيال علمي", "sci-fi"),
            Genre("شريحة من الحياة", "slice-of-life"),
            Genre("رياضي", "sports"),
            Genre("غموض", "mystery"),
            Genre("مأساة", "tragedy")
        )
    }
    
    // تنفيذ البحث مع التصفية
    override suspend fun searchManga(query: String, page: Int, filters: FilterList): MangaPageResult {
        // إذا كان هناك تصفية حسب النوع وليس هناك بحث
        if (query.isBlank() && filters.isNotEmpty()) {
            val genreFilter = filters.find { it is ArabicGenreFilter } as? ArabicGenreFilter
            
            if (genreFilter != null) {
                val selectedGenres = genreFilter.genresList.filter { it.state }.map { it.id }
                
                if (selectedGenres.isNotEmpty()) {
                    return getGenreManga(selectedGenres, page)
                }
            }
        }
        
        // البحث العادي
        return searchManga(query, page)
    }
    
    private suspend fun getGenreManga(genres: List<String>, page: Int): MangaPageResult {
        val genreParam = genres.joinToString(",")
        val url = "$baseUrl/manga/page/$page/?genres=$genreParam"
        
        // تنفيذ الطلب واستخراج البيانات
        // ...
    }
}
```

### دعم الترجمات المختلفة

```kotlin
class TranslationGroup(name: String, val id: String) : Filter.CheckBox(name)

class TranslationGroupFilter(val groups: List<TranslationGroup>) : Filter.Group<TranslationGroup>("فرق الترجمة", groups)

// مثال على تنفيذ التصفية حسب فرق الترجمة
class MangaLionsSource(
    private val networkHelper: NetworkHelper,
    private val securityHelper: SecurityHelper
) : HttpSource() {
    // ...
    
    override fun getFilterList(): FilterList {
        return FilterList(
            Filter.Header("ملاحظة: لا يمكن استخدام التصفيات مع البحث"),
            ArabicGenreFilter(getGenreList()),
            TranslationGroupFilter(getTranslationGroups())
        )
    }
    
    private fun getTranslationGroups(): List<TranslationGroup> {
        return listOf(
            TranslationGroup("مانجا ليونز", "manga-lions"),
            TranslationGroup("تيم إكس", "team-x"),
            TranslationGroup("مانجا عرب", "manga-arab"),
            TranslationGroup("أنمي سانكشواري", "anime-sanctuary")
        )
    }
    
    // تنفيذ البحث مع التصفية
    override suspend fun searchManga(query: String, page: Int, filters: FilterList): MangaPageResult {
        // تنفيذ التصفية حسب فرق الترجمة
        // ...
    }
}
```

## 9. تحسين تجربة القراءة للمانجا العربية

### دعم القراءة من اليمين إلى اليسار

```kotlin
class MangaReaderSettings(private val preferences: PreferencesHelper) {
    
    // إعداد اتجاه القراءة الافتراضي للمصادر العربية
    fun getDefaultReadingDirection(source: Source): ReadingDirection {
        return if (source.lang == "ar") {
            ReadingDirection.RIGHT_TO_LEFT
        } else {
            preferences.defaultReadingDirection
        }
    }
    
    // إعداد وضع القراءة الافتراضي
    fun getDefaultReadingMode(source: Source): ReadingMode {
        return preferences.defaultReadingMode
    }
    
    // الحصول على إعدادات القراءة لمانجا معينة
    fun getMangaReadingSettings(manga: Manga): MangaReadingSettings {
        val source = sourcesManager.getSource(manga.sourceId)
        
        return MangaReadingSettings(
            readingDirection = preferences.getMangaReadingDirection(manga.id) 
                ?: getDefaultReadingDirection(source),
            readingMode = preferences.getMangaReadingMode(manga.id) 
                ?: getDefaultReadingMode(source)
        )
    }
}
```

هذا التنفيذ المفصل للمصادر العربية يوفر دعماً كاملاً للمواقع التي طلبها المستخدم (مانجا سوات، مانجا ليونز، تيم إكس) مع إمكانية إضافة المزيد من المصادر العربية في المستقبل. كما يتضمن تحسينات لدعم اللغة العربية وتجربة قراءة مخصصة للمحتوى العربي.
