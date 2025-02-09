# تطبيق خلفيات صور باستخدام Android Kotlin و Jetpack Compose (MVVM)

هذا المشروع يوضح كيفية إنشاء تطبيق خلفيات صور باستخدام Android Kotlin و Jetpack Compose بنمط MVVM. يتم استدعاء الصور من API خارجي وعرضها في واجهة المستخدم.

## خريطة العمل (Workflow)

### 1. **UI Layer (واجهة المستخدم)**
   - استخدام Jetpack Compose لبناء واجهة المستخدم.
   - عرض قائمة من الصور (Grid أو List).
   - تفاصيل الصورة عند النقر عليها.

### 2. **ViewModel Layer**
   - استخدام ViewModel للفصل بين واجهة المستخدم والبيانات.
   - إدارة حالة البيانات (مثل تحميل الصور، الأخطاء، إلخ).

### 3. **Repository Layer**
   - التواصل مع API لاسترداد البيانات.
   - إدارة البيانات المحلية (إذا لزم الأمر).

### 4. **Data Layer**
   - استخدام Retrofit للتواصل مع API.
   - تحويل JSON إلى كائنات Kotlin باستخدام مكتبة مثل Gson أو Moshi.

### 5. **API**
   - استدعاء الصور من السيرفر عبر API.

---

## خريطة الملفات (File Structure)
```maps
  /app
├── /src
│ ├── /main
│ │ ├── /java/com/example/wallpaperapp
│ │ │ ├── /data
│ │ │ │ ├── /api
│ │ │ │ │ ├── ApiService.kt
│ │ │ │ │ ├── ApiClient.kt
│ │ │ │ ├── /repository
│ │ │ │ │ ├── WallpaperRepository.kt
│ │ │ ├── /di
│ │ │ │ ├── AppModule.kt
│ │ │ ├── /ui
│ │ │ │ ├── /theme
│ │ │ │ │ ├── Theme.kt
│ │ │ │ ├── /screens
│ │ │ │ │ ├── HomeScreen.kt
│ │ │ │ │ ├── DetailScreen.kt
│ │ │ │ ├── /components
│ │ │ │ │ ├── WallpaperItem.kt
│ │ │ │ ├── MainActivity.kt
│ │ │ ├── /viewmodel
│ │ │ │ ├── WallpaperViewModel.kt
│ │ │ ├── /model
│ │ │ │ ├── Wallpaper.kt
│ │ ├── /res
│ │ │ ├── /drawable
│ │ │ ├── /layout
│ │ │ ├── /values
│ │ │ │ ├── strings.xml
│ │ │ │ ├── colors.xml
│ │ │ │ ├── themes.xml
```

---

## تفاصيل الملفات

### 1. **ApiService.kt**
```kotlin
interface ApiService {
    @GET("wallpapers")
    suspend fun getWallpapers(): List<Wallpaper>
}
```
### 2. **ApiClient.kt**
```kotlin
object ApiClient {
    private const val BASE_URL = "https://your-api-url.com/"

    val apiService: ApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}
```
### 3. **WallpaperRepository.kt**
```kotlin
class WallpaperRepository {
    private val apiService = ApiClient.apiService

    suspend fun getWallpapers(): List<Wallpaper> {
        return apiService.getWallpapers()
    }
}
```
### 4. **WallpaperViewModel.kt**
```kotlin
class WallpaperViewModel : ViewModel() {
    private val repository = WallpaperRepository()
    val wallpapers = mutableStateOf<List<Wallpaper>>(emptyList())
    val isLoading = mutableStateOf(false)
    val error = mutableStateOf<String?>(null)

    fun loadWallpapers() {
        viewModelScope.launch {
            isLoading.value = true
            try {
                wallpapers.value = repository.getWallpapers()
            } catch (e: Exception) {
                error.value = e.message
            } finally {
                isLoading.value = false
            }
        }
    }
}
```
### 5. **Wallpaper.kt**
```kotlin
data class Wallpaper(
    val id: String,
    val url: String,
    val title: String
)
```
### 6. **HomeScreen.kt**
```kotlin
@Composable
fun HomeScreen(viewModel: WallpaperViewModel) {
    val wallpapers by viewModel.wallpapers
    val isLoading by viewModel.isLoading
    val error by viewModel.error

    if (isLoading) {
        CircularProgressIndicator()
    } else if (error != null) {
        Text(text = "Error: $error")
    } else {
        LazyVerticalGrid(
            columns = GridCells.Fixed(2)
        ) {
            items(wallpapers) { wallpaper ->
                WallpaperItem(wallpaper = wallpaper)
            }
        }
    }
}
```
### 7. **WallpaperItem.kt**
```kotlin
@Composable
fun WallpaperItem(wallpaper: Wallpaper) {
    Card(
        modifier = Modifier
            .padding(8.dp)
            .fillMaxWidth()
    ) {
        Image(
            painter = rememberImagePainter(data = wallpaper.url),
            contentDescription = wallpaper.title,
            modifier = Modifier.fillMaxSize(),
            contentScale = ContentScale.Crop
        )
    }
}
```
### 8. **MainActivity.kt**
```kotlin
class MainActivity : ComponentActivity() {
    private val viewModel: WallpaperViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            WallpaperAppTheme {
                HomeScreen(viewModel = viewModel)
            }
        }
        viewModel.loadWallpapers()
    }
}
```

### 8. **MainActivity.kt**
 ***الاعتماديات (Dependencies)***
 - أضف الاعتماديات التالية إلى ملف build.gradle (Module: app):
```groovy
dependencies {
    implementation "androidx.compose.ui:ui:1.0.0"
    implementation "androidx.compose.material:material:1.0.0"
    implementation "androidx.compose.ui:ui-tooling-preview:1.0.0"
    implementation "androidx.lifecycle:lifecycle-viewmodel-compose:1.0.0-alpha07"
    implementation "androidx.activity:activity-compose:1.3.0"
    implementation "com.squareup.retrofit2:retrofit:2.9.0"
    implementation "com.squareup.retrofit2:converter-gson:2.9.0"
    implementation "io.coil-kt:coil-compose:1.3.2"
}
```
### ملاحظات إضافية
- تأكد من وجود اتصال بالإنترنت في التطبيق.
- يمكنك إضافة تحسينات مثل التخزين المؤقت للصور باستخدام Coil أو Glide.
- يمكنك إضافة Pagination إذا كان API يدعمه لتحميل المزيد من الصور عند التمرير.
