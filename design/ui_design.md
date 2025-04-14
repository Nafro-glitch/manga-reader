# تصميم واجهة المستخدم

سنقوم بتصميم واجهة مستخدم حديثة واحترافية لتطبيق قارئ المانجا، مع التركيز على سهولة الاستخدام والجاذبية البصرية ودعم كامل للغة العربية.

## 1. الألوان والسمات

سنستخدم نظام ألوان متناسق وحديث مع دعم للوضع الداكن والوضع الفاتح:

### الألوان الأساسية
- **اللون الأساسي**: #6200EE (أرجواني)
- **اللون الثانوي**: #03DAC6 (فيروزي)
- **لون الخلفية (الوضع الفاتح)**: #FFFFFF (أبيض)
- **لون الخلفية (الوضع الداكن)**: #121212 (أسود)
- **لون النص الأساسي (الوضع الفاتح)**: #000000 (أسود)
- **لون النص الأساسي (الوضع الداكن)**: #FFFFFF (أبيض)
- **لون النص الثانوي (الوضع الفاتح)**: #757575 (رمادي)
- **لون النص الثانوي (الوضع الداكن)**: #BBBBBB (رمادي فاتح)

### تنفيذ السمات

```xml
<!-- themes.xml (الوضع الفاتح) -->
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

<!-- themes.xml (الوضع الداكن) -->
<style name="Theme.MangaReader.Dark" parent="Theme.MaterialComponents.DayNight.NoActionBar">
    <item name="colorPrimary">@color/primary_dark</item>
    <item name="colorPrimaryVariant">@color/primary_variant_dark</item>
    <item name="colorSecondary">@color/secondary_dark</item>
    <item name="colorSecondaryVariant">@color/secondary_variant_dark</item>
    <item name="android:colorBackground">@color/background_dark</item>
    <item name="colorSurface">@color/surface_dark</item>
    <item name="colorOnPrimary">@color/white</item>
    <item name="colorOnSecondary">@color/black</item>
    <item name="colorOnBackground">@color/text_primary_dark</item>
    <item name="colorOnSurface">@color/text_primary_dark</item>
    <item name="android:statusBarColor">@color/black</item>
    <item name="android:windowLightStatusBar">false</item>
    <item name="fontFamily">@font/cairo</item>
</style>
```

## 2. الخطوط

سنستخدم خطوط مناسبة للغة العربية مع دعم للغات الأخرى:

```xml
<!-- fonts.xml -->
<font-family xmlns:app="http://schemas.android.com/apk/res-auto"
    app:fontProviderAuthority="com.google.android.gms.fonts"
    app:fontProviderPackage="com.google.android.gms"
    app:fontProviderQuery="name=Cairo"
    app:fontProviderCerts="@array/com_google_android_gms_fonts_certs">
</font-family>
```

## 3. هيكل التطبيق

سنستخدم تصميم Material Design مع Bottom Navigation للتنقل بين الأقسام الرئيسية:

```xml
<!-- activity_main.xml -->
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:defaultNavHost="true"
        app:layout_constraintBottom_toTopOf="@id/bottom_nav"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:navGraph="@navigation/nav_graph" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_nav"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="0dp"
        android:layout_marginEnd="0dp"
        android:background="?attr/colorSurface"
        app:itemIconTint="@color/bottom_nav_item_color"
        app:itemTextColor="@color/bottom_nav_item_color"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:menu="@menu/bottom_nav_menu" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## 4. الشاشات الرئيسية

### 4.1 الشاشة الرئيسية (المكتبة)

تعرض المانجا المحفوظة في مكتبة المستخدم مع إمكانية التصفية والبحث:

```xml
<!-- fragment_library.xml -->
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/colorSurface"
        app:elevation="0dp">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:title="@string/library"
            app:titleTextColor="?attr/colorOnSurface" />

        <com.google.android.material.tabs.TabLayout
            android:id="@+id/tab_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:tabGravity="fill"
            app:tabMode="scrollable"
            app:tabTextAppearance="@style/TabTextAppearance" />

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fab_search"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="16dp"
        android:contentDescription="@string/search"
        app:srcCompat="@drawable/ic_search" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

### 4.2 شاشة التصفح

تعرض مصادر المانجا المتاحة مع إمكانية البحث والتصفية:

```xml
<!-- fragment_browse.xml -->
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/colorSurface"
        app:elevation="0dp">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:title="@string/browse"
            app:titleTextColor="?attr/colorOnSurface">

            <androidx.appcompat.widget.SearchView
                android:id="@+id/search_view"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginEnd="16dp"
                app:iconifiedByDefault="false"
                app:queryHint="@string/search_hint" />

        </androidx.appcompat.widget.Toolbar>

        <com.google.android.material.chip.ChipGroup
            android:id="@+id/chip_group"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dp"
            android:layout_marginEnd="16dp"
            android:layout_marginBottom="8dp"
            app:singleLine="true">

            <com.google.android.material.chip.Chip
                android:id="@+id/chip_arabic"
                style="@style/Widget.MaterialComponents.Chip.Filter"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/arabic" />

            <com.google.android.material.chip.Chip
                android:id="@+id/chip_english"
                style="@style/Widget.MaterialComponents.Chip.Filter"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/english" />

            <com.google.android.material.chip.Chip
                android:id="@+id/chip_japanese"
                style="@style/Widget.MaterialComponents.Chip.Filter"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/japanese" />

        </com.google.android.material.chip.ChipGroup>

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:clipToPadding="false"
        android:padding="8dp"
        app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        app:spanCount="2" />

    <com.google.android.material.progressindicator.CircularProgressIndicator
        android:id="@+id/progress_indicator"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:indeterminate="true"
        android:visibility="gone" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

### 4.3 شاشة التنزيلات

تعرض المانجا التي تم تنزيلها أو قيد التنزيل:

```xml
<!-- fragment_downloads.xml -->
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/colorSurface"
        app:elevation="0dp">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:title="@string/downloads"
            app:titleTextColor="?attr/colorOnSurface" />

        <com.google.android.material.tabs.TabLayout
            android:id="@+id/tab_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:tabGravity="fill"
            app:tabMode="fixed">

            <com.google.android.material.tabs.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/queue" />

            <com.google.android.material.tabs.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/completed" />

        </com.google.android.material.tabs.TabLayout>

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

### 4.4 شاشة الإعدادات

تعرض إعدادات التطبيق المختلفة:

```xml
<!-- fragment_settings.xml -->
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/colorSurface"
        app:elevation="0dp">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:title="@string/settings"
            app:titleTextColor="?attr/colorOnSurface" />

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="8dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="2dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/appearance"
                        android:textAppearance="?attr/textAppearanceHeadline6" />

                    <com.google.android.material.switchmaterial.SwitchMaterial
                        android:id="@+id/switch_dark_theme"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:text="@string/dark_theme" />

                    <com.google.android.material.switchmaterial.SwitchMaterial
                        android:id="@+id/switch_rtl_layout"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:text="@string/rtl_layout" />

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="8dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="2dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/reader"
                        android:textAppearance="?attr/textAppearanceHeadline6" />

                    <com.google.android.material.switchmaterial.SwitchMaterial
                        android:id="@+id/switch_keep_screen_on"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:text="@string/keep_screen_on" />

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:text="@string/reading_direction"
                        android:textAppearance="?attr/textAppearanceSubtitle1" />

                    <RadioGroup
                        android:id="@+id/radio_group_reading_direction"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp">

                        <RadioButton
                            android:id="@+id/radio_rtl"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:text="@string/right_to_left" />

                        <RadioButton
                            android:id="@+id/radio_ltr"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:text="@string/left_to_right" />

                        <RadioButton
                            android:id="@+id/radio_vertical"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:text="@string/vertical" />

                    </RadioGroup>

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="8dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="2dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/downloads"
                        android:textAppearance="?attr/textAppearanceHeadline6" />

                    <com.google.android.material.switchmaterial.SwitchMaterial
                        android:id="@+id/switch_download_wifi_only"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:text="@string/download_wifi_only" />

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:text="@string/download_location"
                        android:textAppearance="?attr/textAppearanceSubtitle1" />

                    <TextView
                        android:id="@+id/text_download_location"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="4dp"
                        android:textAppearance="?attr/textAppearanceBody2" />

                    <com.google.android.material.button.MaterialButton
                        android:id="@+id/button_change_download_location"
                        style="@style/Widget.MaterialComponents.Button.TextButton"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="@string/change" />

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="8dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="2dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/advanced"
                        android:textAppearance="?attr/textAppearanceHeadline6" />

                    <com.google.android.material.button.MaterialButton
                        android:id="@+id/button_clear_cache"
                        style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:text="@string/clear_cache" />

                    <com.google.android.material.button.MaterialButton
                        android:id="@+id/button_add_repository"
                        style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/add_repository" />

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="8dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="2dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/about"
                        android:textAppearance="?attr/textAppearanceHeadline6" />

                    <TextView
                        android:id="@+id/text_version"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:textAppearance="?attr/textAppearanceBody2" />

                    <com.google.android.material.button.MaterialButton
                        android:id="@+id/button_github"
                        style="@style/Widget.MaterialComponents.Button.TextButton"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="@string/github" />

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

        </LinearLayout>

    </androidx.core.widget.NestedScrollView>

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

## 5. شاشات تفاصيل المانجا والقراءة

### 5.1 شاشة تفاصيل المانجا

تعرض تفاصيل المانجا وقائمة الفصول:

```xml
<!-- fragment_manga_details.xml -->
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:fitsSystemWindows="true"
        android:theme="@style/ThemeOverlay.MaterialComponents.Dark.ActionBar">

        <com.google.android.material.appbar.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="250dp"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:id="@+id/manga_cover"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:contentDescription="@string/manga_cover"
                android:fitsSystemWindows="true"
                android:scaleType="centerCrop"
                app:layout_collapseMode="parallax" />

            <View
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="@drawable/gradient_scrim"
                android:fitsSystemWindows="true" />

            <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/ThemeOverlay.MaterialComponents.Light" />

        </com.google.android.material.appbar.CollapsingToolbarLayout>

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:padding="16dp">

                <ImageView
                    android:id="@+id/manga_thumbnail"
                    android:layout_width="100dp"
                    android:layout_height="150dp"
                    android:contentDescription="@string/manga_cover"
                    android:scaleType="centerCrop" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="16dp"
                    android:orientation="vertical">

                    <TextView
                        android:id="@+id/manga_title"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:textAppearance="?attr/textAppearanceHeadline6"
                        tools:text="عنوان المانجا" />

                    <TextView
                        android:id="@+id/manga_author"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="4dp"
                        android:textAppearance="?attr/textAppearanceBody2"
                        tools:text="المؤلف" />

                    <TextView
                        android:id="@+id/manga_status"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="4dp"
                        android:textAppearance="?attr/textAppearanceBody2"
                        tools:text="الحالة: مستمرة" />

                    <com.google.android.material.chip.ChipGroup
                        android:id="@+id/genre_chip_group"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp" />

                </LinearLayout>

            </LinearLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:padding="16dp">

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/button_add_to_library"
                    style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_marginEnd="8dp"
                    android:layout_weight="1"
                    android:text="@string/add_to_library" />

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/button_read"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="8dp"
                    android:layout_weight="1"
                    android:text="@string/read" />

            </LinearLayout>

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:padding="16dp"
                android:text="@string/description"
                android:textAppearance="?attr/textAppearanceHeadline6" />

            <TextView
                android:id="@+id/manga_description"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:paddingStart="16dp"
                android:paddingEnd="16dp"
                android:paddingBottom="16dp"
                android:textAppearance="?attr/textAppearanceBody2"
                tools:text="وصف المانجا..." />

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:padding="16dp">

                <TextView
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:text="@string/chapters"
                    android:textAppearance="?attr/textAppearanceHeadline6" />

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/button_download_all"
                    style="@style/Widget.MaterialComponents.Button.TextButton"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="@string/download_all" />

            </LinearLayout>

            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/chapters_recycler_view"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:nestedScrollingEnabled="false"
                app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager" />

        </LinearLayout>

    </androidx.core.widget.NestedScrollView>

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fab_download"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:contentDescription="@string/download"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end"
        app:srcCompat="@drawable/ic_download" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

### 5.2 شاشة القراءة

توفر تجربة قراءة سلسة مع دعم للاتجاهات المختلفة:

```xml
<!-- activity_reader.xml -->
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/black">

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@drawable/gradient_toolbar"
        android:visibility="gone"
        app:layout_constraintTop_toTopOf="parent"
        app:popupTheme="@style/ThemeOverlay.MaterialComponents.Light"
        app:titleTextColor="@color/white" />

    <LinearLayout
        android:id="@+id/bottom_controls"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/gradient_toolbar_bottom"
        android:orientation="vertical"
        android:padding="16dp"
        android:visibility="gone"
        app:layout_constraintBottom_toBottomOf="parent">

        <TextView
            android:id="@+id/text_page_info"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:textColor="@color/white"
            android:textSize="14sp" />

        <SeekBar
            android:id="@+id/seek_bar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp" />

    </LinearLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

## 6. عناصر واجهة المستخدم المخصصة

### 6.1 عنصر بطاقة المانجا

يستخدم لعرض المانجا في القوائم والشبكات:

```xml
<!-- item_manga.xml -->
<com.google.android.material.card.MaterialCardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="4dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="2dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <ImageView
            android:id="@+id/manga_cover"
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:contentDescription="@string/manga_cover"
            android:scaleType="centerCrop" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="8dp">

            <TextView
                android:id="@+id/manga_title"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:ellipsize="end"
                android:maxLines="2"
                android:textAppearance="?attr/textAppearanceSubtitle1"
                tools:text="عنوان المانجا" />

            <TextView
                android:id="@+id/manga_subtitle"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:ellipsize="end"
                android:maxLines="1"
                android:textAppearance="?attr/textAppearanceCaption"
                tools:text="المؤلف | الحالة" />

        </LinearLayout>

    </LinearLayout>

</com.google.android.material.card.MaterialCardView>
```

### 6.2 عنصر الفصل

يستخدم لعرض فصول المانجا في قائمة الفصول:

```xml
<!-- item_chapter.xml -->
<com.google.android.material.card.MaterialCardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginStart="16dp"
    android:layout_marginEnd="16dp"
    android:layout_marginBottom="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="1dp">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp">

        <TextView
            android:id="@+id/chapter_title"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textAppearance="?attr/textAppearanceSubtitle1"
            app:layout_constraintEnd_toStartOf="@id/chapter_download_button"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            tools:text="الفصل 1: البداية" />

        <TextView
            android:id="@+id/chapter_date"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="4dp"
            android:textAppearance="?attr/textAppearanceCaption"
            app:layout_constraintEnd_toStartOf="@id/chapter_download_button"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/chapter_title"
            tools:text="2023-01-01" />

        <ImageButton
            android:id="@+id/chapter_download_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:contentDescription="@string/download"
            android:padding="8dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:srcCompat="@drawable/ic_download" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</com.google.android.material.card.MaterialCardView>
```

### 6.3 عنصر المصدر

يستخدم لعرض مصادر المانجا في شاشة التصفح:

```xml
<!-- item_source.xml -->
<com.google.android.material.card.MaterialCardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="2dp">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp">

        <ImageView
            android:id="@+id/source_icon"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:contentDescription="@string/source_icon"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/source_name"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dp"
            android:textAppearance="?attr/textAppearanceSubtitle1"
            app:layout_constraintEnd_toStartOf="@id/source_switch"
            app:layout_constraintStart_toEndOf="@id/source_icon"
            app:layout_constraintTop_toTopOf="parent"
            tools:text="مانجا سوات" />

        <TextView
            android:id="@+id/source_language"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dp"
            android:layout_marginTop="4dp"
            android:textAppearance="?attr/textAppearanceCaption"
            app:layout_constraintEnd_toStartOf="@id/source_switch"
            app:layout_constraintStart_toEndOf="@id/source_icon"
            app:layout_constraintTop_toBottomOf="@id/source_name"
            tools:text="العربية" />

        <com.google.android.material.switchmaterial.SwitchMaterial
            android:id="@+id/source_switch"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</com.google.android.material.card.MaterialCardView>
```

## 7. الرسوم المتحركة والانتقالات

لتحسين تجربة المستخدم، سنضيف رسوم متحركة وانتقالات سلسة:

```kotlin
// تنفيذ الانتقال المشترك بين شاشة المكتبة وشاشة تفاصيل المانجا
class SharedElementTransition {
    
    fun setupTransition(activity: AppCompatActivity) {
        val transition = TransitionInflater.from(activity)
            .inflateTransition(R.transition.shared_element_transition)
        
        activity.window.sharedElementEnterTransition = transition
        activity.window.sharedElementReturnTransition = transition
    }
    
    fun startTransition(
        fragment: Fragment,
        view: View,
        mangaId: Long,
        transitionName: String
    ) {
        val extras = FragmentNavigatorExtras(view to transitionName)
        
        fragment.findNavController().navigate(
            R.id.action_libraryFragment_to_mangaDetailsFragment,
            bundleOf("manga_id" to mangaId),
            null,
            extras
        )
    }
}
```

```xml
<!-- transition/shared_element_transition.xml -->
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:interpolator="@android:interpolator/fast_out_slow_in">
    <changeBounds />
    <changeTransform />
    <changeClipBounds />
    <changeImageTransform />
</transitionSet>
```

## 8. دعم اللغة العربية

لضمان دعم كامل للغة العربية، سنضيف الإعدادات التالية:

```xml
<!-- AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.mangareader">
    
    <application
        android:supportsRtl="true"
        ...>
        ...
    </application>
</manifest>
```

```xml
<!-- strings.xml (ar) -->
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
</resources>
```

## 9. تنفيذ الواجهة البرمجية

### 9.1 نشاط القارئ الرئيسي

```kotlin
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

### 9.2 مكيف المانجا

```kotlin
class MangaAdapter(
    private val context: Context,
    private val onMangaClick: (Manga, View) -> Unit
) : RecyclerView.Adapter<MangaAdapter.MangaViewHolder>() {
    
    private val mangas = mutableListOf<Manga>()
    
    fun updateMangas(newMangas: List<Manga>) {
        mangas.clear()
        mangas.addAll(newMangas)
        notifyDataSetChanged()
    }
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MangaViewHolder {
        val binding = ItemMangaBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return MangaViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: MangaViewHolder, position: Int) {
        holder.bind(mangas[position])
    }
    
    override fun getItemCount(): Int = mangas.size
    
    inner class MangaViewHolder(private val binding: ItemMangaBinding) : RecyclerView.ViewHolder(binding.root) {
        
        fun bind(manga: Manga) {
            binding.mangaTitle.text = manga.title
            binding.mangaSubtitle.text = manga.author ?: context.getString(R.string.unknown_author)
            
            // تحميل صورة الغلاف
            Glide.with(context)
                .load(manga.thumbnailUrl)
                .placeholder(R.drawable.placeholder_cover)
                .error(R.drawable.error_cover)
                .centerCrop()
                .into(binding.mangaCover)
            
            // تعيين اسم الانتقال المشترك
            binding.mangaCover.transitionName = "manga_cover_${manga.id}"
            
            // تعيين معالج النقر
            binding.root.setOnClickListener {
                onMangaClick(manga, binding.mangaCover)
            }
        }
    }
}
```

### 9.3 مكيف الفصول

```kotlin
class ChapterAdapter(
    private val context: Context,
    private val onChapterClick: (Chapter) -> Unit,
    private val onDownloadClick: (Chapter) -> Unit
) : RecyclerView.Adapter<ChapterAdapter.ChapterViewHolder>() {
    
    private val chapters = mutableListOf<Chapter>()
    
    fun updateChapters(newChapters: List<Chapter>) {
        chapters.clear()
        chapters.addAll(newChapters)
        notifyDataSetChanged()
    }
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ChapterViewHolder {
        val binding = ItemChapterBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return ChapterViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: ChapterViewHolder, position: Int) {
        holder.bind(chapters[position])
    }
    
    override fun getItemCount(): Int = chapters.size
    
    inner class ChapterViewHolder(private val binding: ItemChapterBinding) : RecyclerView.ViewHolder(binding.root) {
        
        fun bind(chapter: Chapter) {
            binding.chapterTitle.text = chapter.name
            
            // تنسيق التاريخ
            val dateFormat = SimpleDateFormat("yyyy-MM-dd", Locale.getDefault())
            binding.chapterDate.text = dateFormat.format(Date(chapter.dateUpload))
            
            // تعيين حالة التنزيل
            if (chapter.downloaded) {
                binding.chapterDownloadButton.setImageResource(R.drawable.ic_downloaded)
            } else {
                binding.chapterDownloadButton.setImageResource(R.drawable.ic_download)
            }
            
            // تعيين معالجات النقر
            binding.root.setOnClickListener {
                onChapterClick(chapter)
            }
            
            binding.chapterDownloadButton.setOnClickListener {
                onDownloadClick(chapter)
            }
        }
    }
}
```

### 9.4 شاشة القراءة

```kotlin
class ReaderActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityReaderBinding
    private lateinit var viewModel: ReaderViewModel
    
    private var isControlsVisible = false
    
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
        
        // إعداد ViewModel
        viewModel = ViewModelProvider(this).get(ReaderViewModel::class.java)
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

## 10. تحسينات إضافية

### 10.1 دعم الوضع الداكن التلقائي

```kotlin
class ThemeHelper(private val context: Context) {
    
    fun applyTheme(themeMode: ThemeMode) {
        when (themeMode) {
            ThemeMode.LIGHT -> AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO)
            ThemeMode.DARK -> AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES)
            ThemeMode.SYSTEM -> AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_FOLLOW_SYSTEM)
            ThemeMode.AUTO -> applyAutoTheme()
        }
    }
    
    private fun applyAutoTheme() {
        // تطبيق الوضع الداكن تلقائياً بناءً على الوقت
        val currentHour = Calendar.getInstance().get(Calendar.HOUR_OF_DAY)
        
        if (currentHour in 6..17) {
            // النهار (6 صباحاً - 6 مساءً): الوضع الفاتح
            AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO)
        } else {
            // الليل: الوضع الداكن
            AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES)
        }
    }
    
    enum class ThemeMode {
        LIGHT, DARK, SYSTEM, AUTO
    }
}
```

### 10.2 تحسين أداء تحميل الصور

```kotlin
class GlideModule : AppGlideModule() {
    
    override fun applyOptions(context: Context, builder: GlideBuilder) {
        // تعيين حجم ذاكرة التخزين المؤقت
        val memoryCacheSizeBytes = 1024 * 1024 * 50 // 50 ميجابايت
        builder.setMemoryCache(LruResourceCache(memoryCacheSizeBytes.toLong()))
        
        // تعيين حجم ذاكرة التخزين المؤقت للصور
        val bitmapPoolSizeBytes = 1024 * 1024 * 30 // 30 ميجابايت
        val arrayPoolSizeBytes = 1024 * 1024 * 10 // 10 ميجابايت
        builder.setBitmapPool(LruBitmapPool(bitmapPoolSizeBytes.toLong()))
        builder.setArrayPool(LruArrayPool(arrayPoolSizeBytes))
        
        // تعيين حجم ذاكرة التخزين المؤقت على القرص
        val discCacheSizeBytes = 1024 * 1024 * 200 // 200 ميجابايت
        builder.setDiskCache(InternalCacheDiskCacheFactory(context, discCacheSizeBytes.toLong()))
    }
    
    override fun registerComponents(context: Context, glide: Glide, registry: Registry) {
        // تسجيل معالجات مخصصة للصور
        registry.append(
            String::class.java,
            InputStream::class.java,
            OkHttpUrlLoader.Factory(createOkHttpClient())
        )
    }
    
    private fun createOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .connectTimeout(15, TimeUnit.SECONDS)
            .readTimeout(15, TimeUnit.SECONDS)
            .build()
    }
}
```

### 10.3 تحسين تجربة القراءة للمانجا العربية

```kotlin
class ArabicReadingHelper {
    
    // تحديد اتجاه القراءة الافتراضي للمصادر العربية
    fun getDefaultReadingDirection(source: Source): ReadingDirection {
        return if (source.lang == "ar") {
            ReadingDirection.RIGHT_TO_LEFT
        } else {
            ReadingDirection.LEFT_TO_RIGHT
        }
    }
    
    // تحسين عرض النص العربي في صفحات المانجا
    fun optimizeArabicTextRendering(textView: TextView) {
        textView.textDirection = View.TEXT_DIRECTION_RTL
        textView.textAlignment = View.TEXT_ALIGNMENT_VIEW_START
        
        // استخدام خط مناسب للغة العربية
        val typeface = ResourcesCompat.getFont(textView.context, R.font.cairo)
        textView.typeface = typeface
    }
    
    // تحسين البحث باللغة العربية
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
}
```

هذا التصميم المفصل لواجهة المستخدم يوفر تجربة مستخدم حديثة واحترافية مع دعم كامل للغة العربية، ويتضمن جميع الميزات المطلوبة مثل تصفح المانجا وتنزيلها وقراءتها بطرق مختلفة.
