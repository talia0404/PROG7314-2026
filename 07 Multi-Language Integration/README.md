# 🌍 Multi-Language Support in Android

In this activity, you will add multi-language support to an Android application.

The application should support:

```text
English
Afrikaans
isiZulu
```

The user should be able to:

- Open the application.
- View all text in the currently selected language.
- Change the application's language.
- Switch between English, Afrikaans and isiZulu.
- Return to the device/system language.
- Close and reopen the application without unnecessarily losing the selected app language.
- Use Android's per-app language functionality where supported.

The intended flow is:

```text
Application starts
-> Android determines the application locale
-> Correct string resources are loaded
-> User chooses another language
-> Application locale changes
-> Android reloads the relevant resources
-> Interface appears in the selected language
```

---

# 🧠 1. Understand Internationalisation and Localisation

Before implementing anything, understand the two main concepts.

## Internationalisation

Internationalisation means designing an application so that it **can support different languages and regional formats**.

Examples include:

```text
Moving visible text into resource files

Avoiding hardcoded user-facing Strings

Supporting different date formats

Supporting different number formats

Supporting right-to-left layouts
```

The application is being prepared so that localisation is possible.

---

## Localisation

Localisation means adapting the application for a particular language or region.

For example:

```text
English:
Welcome to our application

Afrikaans:
Welkom by ons toepassing

isiZulu:
Siyakwamukela kuhlelo lwethu
```

Android provides a resource system specifically for this purpose.

### 📚 Helpful Resource

**Android — Localise Your App:**  
https://developer.android.com/guide/topics/resources/localization

---

# 🧩 2. Understand the Approach Used in This Activity

There are several ways an Android application can support multiple languages.

## Option 1 — Follow the Device Language

The application can simply provide translated resources.

Android then loads the language that best matches the device language.

For example:

```text
Device language:
Afrikaans

->

Android loads:
values-af
```

This is the simplest localisation approach.

---

## Option 2 — Android Per-App Language Settings

Android 13 and later support language preferences for individual applications.

This means a user's phone could use:

```text
English
```

while one specific application uses:

```text
isiZulu
```

Android provides system-level per-app language preferences for applications that declare their supported locales. :contentReference[oaicite:0]{index=0}

---

## Option 3 — In-App Language Selector

An application can also provide its own language screen.

For example:

```text
Choose Language

English

Afrikaans

isiZulu

Use Device Language
```

Changing the language through the application updates its locale.

---

# ⭐ 3. Approach Used for This Demo

You will use:

```text
Android String Resources
+
AppCompat Per-App Locale Support
+
In-App Language Selector
```

This approach is useful because it demonstrates:

- How translated resources are stored.
- How Android selects resources.
- How the user can change only the application's language.
- How Android 13+ integrates app language preferences with system settings.
- How older Android versions can still support application-specific language selection.

AndroidX AppCompat provides equivalent per-app locale APIs for backward compatibility with versions before Android 13. :contentReference[oaicite:1]{index=1}

---

# 🏗️ 4. Create the Android Project

Create a new Android Studio project.

Use:

```text
New Project
-> Empty Activity
```

Suggested settings:

```text
Name:
MultiLanguageDemo

Language:
Kotlin

UI:
Jetpack Compose

Minimum SDK:
API 24 or later
```

Choose an appropriate package name.

For example:

```text
com.yourname.multilanguagedemo
```

Before continuing:

- Allow Gradle Sync to finish.
- Run the project.
- Confirm that the application builds successfully.
- Confirm that the default activity opens.

---

# 📦 5. Add AppCompat

The application will use AndroidX AppCompat's per-app locale functionality.

Open:

```text
app/build.gradle.kts
```

Add AppCompat if it is not already available:

```kotlin
implementation("androidx.appcompat:appcompat:1.7.1")
```

Then run:

```text
File
-> Sync Project with Gradle Files
```

Do not continue until the Gradle Sync succeeds.

`AppCompatDelegate.setApplicationLocales()` has been available since AppCompat 1.6.0. :contentReference[oaicite:2]{index=2}

### 📚 Helpful Resource

**Android — AppCompatDelegate:**  
https://developer.android.com/reference/kotlin/androidx/appcompat/app/AppCompatDelegate

Focus on:

```text
setApplicationLocales()

getApplicationLocales()
```

---

# 📝 6. Move User-Facing Text into `strings.xml`

One of the most important localisation rules is:

> User-facing text should not normally be hardcoded directly inside Compose.

Avoid designs such as:

```text
Text displaying:
"Welcome to the application"
```

directly from a Kotlin String.

Instead, visible text should be stored inside Android String resources.

The default resource file is:

```text
app/src/main/res/values/strings.xml
```

---

# 🇬🇧 7. Create the Default English Resources

Open:

```text
app/src/main/res/values/strings.xml
```

Add all user-facing Strings required by your screen.

For this demonstration, include Strings for:

```text
Application name

Screen title

Welcome message

Description

Choose language heading

English

Afrikaans

isiZulu

Current language

Use device language
```

For example:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <string name="app_name">Multi-Language Demo</string>

    <string name="screen_title">Language Settings</string>

    <string name="welcome_message">
        Welcome to our application!
    </string>

    <string name="description">
        Choose the language you would like to use.
    </string>

    <string name="choose_language">
        Choose a language
    </string>

    <string name="language_english">
        English
    </string>

    <string name="language_afrikaans">
        Afrikaans
    </string>

    <string name="language_zulu">
        isiZulu
    </string>

    <string name="current_language">
        Current language
    </string>

    <string name="use_system_language">
        Use device language
    </string>

</resources>
```

The default:

```text
values/
```

resource directory should contain a complete set of resources required by the application.

Android uses these resources as the default/fallback set when a more specific localisation is unavailable. ([developer.android.com](https://developer.android.com/guide/topics/resources/localization))

---

# 🇿🇦 8. Create the Afrikaans Resource Directory

In Android Studio:

```text
res
-> Right-click
-> New
-> Android Resource Directory
```

Choose:

```text
Resource Type:
values
```

Add the:

```text
Locale
```

qualifier.

Select:

```text
Afrikaans
```

Android should create:

```text
values-af/
```

Inside this directory, create:

```text
strings.xml
```

Use the **same String resource names** as the default English file.

For example:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <string name="app_name">
        Meertalige Demo
    </string>

    <string name="screen_title">
        Taalinstellings
    </string>

    <string name="welcome_message">
        Welkom by ons toepassing!
    </string>

    <string name="description">
        Kies die taal wat jy graag wil gebruik.
    </string>

    <string name="choose_language">
        Kies 'n taal
    </string>

    <string name="language_english">
        Engels
    </string>

    <string name="language_afrikaans">
        Afrikaans
    </string>

    <string name="language_zulu">
        isiZulu
    </string>

    <string name="current_language">
        Huidige taal
    </string>

    <string name="use_system_language">
        Gebruik toestel se taal
    </string>

</resources>
```

---

# 🇿🇦 9. Create the isiZulu Resource Directory

Create another locale-specific `values` resource directory.

The language code for isiZulu is:

```text
zu
```

The directory should therefore be:

```text
values-zu/
```

Inside it, create:

```text
strings.xml
```

Use the same resource names again.

For example:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <string name="app_name">
        Uhlelo Lwezilimi Eziningi
    </string>

    <string name="screen_title">
        Izilungiselelo Zolimi
    </string>

    <string name="welcome_message">
        Siyakwamukela kuhlelo lwethu!
    </string>

    <string name="description">
        Khetha ulimi ongathanda ukulusebenzisa.
    </string>

    <string name="choose_language">
        Khetha ulimi
    </string>

    <string name="language_english">
        IsiNgisi
    </string>

    <string name="language_afrikaans">
        IsiBhunu
    </string>

    <string name="language_zulu">
        isiZulu
    </string>

    <string name="current_language">
        Ulimi lwamanje
    </string>

    <string name="use_system_language">
        Sebenzisa ulimi lwedivayisi
    </string>

</resources>
```

> ⚠️ In a production application, translations should ideally be checked by fluent speakers rather than relying entirely on machine translation.

---

# 📁 10. Check the Resource Structure

Your resources should now resemble:

```text
res/
|
|-> values/
|   |-> strings.xml
|
|-> values-af/
|   |-> strings.xml
|
|-> values-zu/
|   |-> strings.xml
```

Each `strings.xml` should use matching resource names.

For example:

```text
values/strings.xml
-> welcome_message

values-af/strings.xml
-> welcome_message

values-zu/strings.xml
-> welcome_message
```

The String **name remains the same**.

The String **value changes according to language**.

---

# 🧠 11. Understand How Android Selects the Translation

Your Kotlin UI will request a String resource by its resource ID.

Android then determines which resource directory best matches the current locale.

Conceptually:

```text
UI requests welcome_message
-> Android checks current locale
-> Matching resource directory selected
-> Correct translated value returned
```

For example:

```text
Current locale:
English

-> values/
```

```text
Current locale:
Afrikaans

-> values-af/
```

```text
Current locale:
isiZulu

-> values-zu/
```

The Compose screen does not require separate versions for every language.

---

# 🌍 12. Understand Language Tags

Each language is represented by a standard language tag.

For this demo:

```text
English
-> en

Afrikaans
-> af

isiZulu
-> zu
```

You may also encounter regional tags.

Examples:

```text
English - South Africa
-> en-ZA

English - United Kingdom
-> en-GB

English - United States
-> en-US
```

For this activity, you only need:

```text
en
af
zu
```

---

# 🌍 13. Declare the Supported Application Locales

Android should know which languages the application officially supports.

Inside:

```text
app/src/main/res/xml/
```

create:

```text
locales_config.xml
```

If the `xml` directory does not exist:

```text
res
-> New
-> Android Resource Directory
-> Resource Type: xml
```

Add:

```xml
<?xml version="1.0" encoding="utf-8"?>

<locale-config xmlns:android="http://schemas.android.com/apk/res/android">

    <locale android:name="en" />

    <locale android:name="af" />

    <locale android:name="zu" />

</locale-config>
```

This declares:

```text
English
Afrikaans
isiZulu
```

as supported application locales.

Android uses BCP 47 language tags in this configuration. :contentReference[oaicite:3]{index=3}

---

# 📜 14. Reference the Locale Configuration in the Manifest

Open:

```text
app/src/main/AndroidManifest.xml
```

Inside the:

```xml
<application>
```

element, reference the locale configuration.

Add:

```xml
android:localeConfig="@xml/locales_config"
```

Your `<application>` element should therefore include the locale configuration alongside the application's existing attributes.

Android's `android:localeConfig` manifest attribute references an XML resource containing the application's supported locales. :contentReference[oaicite:4]{index=4}

---

# 🔄 15. Configure Locale Storage for Older Android Versions

Android 13 and later automatically persist application-level locale preferences when using the per-app locale APIs.

For earlier Android versions, AppCompat can automatically store the selected application locale.

Inside the:

```xml
<application>
```

element, add:

```xml
<service
    android:name="androidx.appcompat.app.AppLocalesMetadataHolderService"
    android:enabled="false"
    android:exported="false">

    <meta-data
        android:name="autoStoreLocales"
        android:value="true" />

</service>
```

You are **not creating this service yourself**.

It is supplied by AppCompat.

Its purpose is to allow AppCompat to persist app-level locale selections on older Android versions.

On Android 13 and later, AppCompat handles application-locale storage through the platform automatically. :contentReference[oaicite:5]{index=5}

---

# 📱 16. Use an AppCompat-Compatible Activity

The demo uses:

```text
AppCompatDelegate
```

for per-app locale support.

Your Activity should therefore use an AppCompat-compatible Activity setup.

Change the Activity superclass from:

```text
ComponentActivity
```

to:

```text
AppCompatActivity
```

You can still use Jetpack Compose normally inside an `AppCompatActivity`.

Make sure the required AppCompat import is added.

---

# 🎨 17. Create the Language Selection Interface

Create a Compose screen for selecting the application's language.

The screen should include:

```text
Title

Welcome message

Short description

English option

Afrikaans option

isiZulu option

Use device language option
```

You may use:

- Buttons.
- Radio buttons.
- A dropdown.
- Cards.
- A Settings-style list.

The exact visual design is your choice.

---

# 📝 18. Load All Visible Text from String Resources

Every localisable piece of text on the screen must be loaded from the appropriate Android String resource.

Do not hardcode:

```text
Welcome

Settings

English

Afrikaans

isiZulu
```

directly inside the Compose UI.

Use Android's Compose String resource functionality instead.

### 📚 Helpful Resource

**Android — Resources in Compose:**  
https://developer.android.com/develop/ui/compose/resources

Focus on:

```text
stringResource()
```

---

# 🧠 19. Create the Language Selection Logic

You now need to implement the behaviour that changes the application locale.

Your language logic must support:

```text
English
-> en

Afrikaans
-> af

isiZulu
-> zu
```

When the user selects a language:

1. Determine the language tag.
2. Create an application locale list.
3. Apply the locale through AppCompat's application-locale functionality.
4. Allow Android to process the resulting configuration change.
5. Reload the translated String resources.

You will need to research:

```text
LocaleListCompat

AppCompatDelegate.setApplicationLocales()
```

### 📚 Helpful Resources

**AppCompatDelegate:**  
https://developer.android.com/reference/kotlin/androidx/appcompat/app/AppCompatDelegate

**LocaleListCompat:**  
https://developer.android.com/reference/androidx/core/os/LocaleListCompat

Focus on:

```text
forLanguageTags()

setApplicationLocales()
```

Do not manually replace every Text value when the user selects a language.

---

# 🔁 20. Understand What Happens When the Language Changes

When the application locale changes, Android performs a configuration change.

This may recreate the Activity.

This behaviour is normal.

Conceptually:

```text
User selects Afrikaans
-> App locale becomes "af"
-> Android configuration changes
-> Activity may be recreated
-> Resources load again
-> values-af selected
-> Afrikaans UI displayed
```

You should not attempt to manually translate every existing composable after the selection.

Android's resource system performs that work.

Changing application locales may cause attached components to receive a configuration change and potentially be recreated. :contentReference[oaicite:6]{index=6}

---

# 📱 21. Add a "Use Device Language" Option

The user should be able to remove their application-specific language choice.

When this option is selected:

```text
ParkSmart/app locale preference removed
-> Application follows device language
```

Research how an:

```text
empty LocaleListCompat
```

can be passed to AppCompat's application-locale API to return the application to the system locale.

### 📚 Resource

https://developer.android.com/reference/kotlin/androidx/appcompat/app/AppCompatDelegate

---

# 🧭 22. Consider Separating Language Logic from the UI

Do not place every localisation operation directly inside each language button.

A cleaner design is to create a dedicated language-management class/object responsible for:

```text
Changing app language

Returning to system language

Reading current app language
```

The Compose screen should mainly be responsible for:

```text
Displaying language choices
-> Reporting user selection
```

while the language-management functionality handles the locale operation.

You are responsible for deciding the exact class name and structure.

---

# ▶️ 23. Test the Default English Resources

Run the application.

Set or confirm the language as:

```text
English
```

Verify that the application displays the English resources.

For example:

```text
Language Settings

Welcome to our application!

Choose the language you would like to use.
```

---

# 🇿🇦 24. Test Afrikaans

Choose:

```text
Afrikaans
```

Expected flow:

```text
Select Afrikaans
-> Application locale becomes "af"
-> Android reloads resources
-> values-af selected
```

Confirm that all translated text changes.

Look carefully for text that remains English.

Any remaining English text may indicate:

- Hardcoded text.
- A missing translation.
- An incorrectly named String resource.

---

# 🇿🇦 25. Test isiZulu

Choose:

```text
isiZulu
```

Expected:

```text
Locale:
zu

->

Resources:
values-zu
```

Confirm that the screen uses the isiZulu translations.

---

# 📱 26. Test Returning to the Device Language

Select:

```text
Use device language
```

The application should stop forcing one of its own language preferences.

It should then follow the device language where an appropriate translated resource exists.

---

# 🔁 27. Test Language Persistence

Perform:

```text
Select Afrikaans
-> Close application
-> Reopen application
```

Confirm that the application language behaves correctly according to the per-app language configuration.

Then repeat with isiZulu.

Per-app locale choices are persisted by the platform on Android 13+, while AppCompat can provide storage for earlier versions when configured. :contentReference[oaicite:7]{index=7}

---

# 📱 28. Test Android 13+ App Language Settings

On Android 13 or later, open the application's system settings.

Depending on the Android version/device manufacturer, look for:

```text
Settings
-> Apps
-> Multi-Language Demo
-> Language
```

or a similar App Languages section.

Confirm that the supported languages include:

```text
English

Afrikaans

isiZulu
```

Android 13 introduced a system location where users can select preferred languages for individual applications. :contentReference[oaicite:8]{index=8}

---

# 🔄 29. Test Changing the Language Outside the App

On Android 13+:

1. Close or minimise the application.
2. Open Android's App Language settings.
3. Change the app language.
4. Return to the application.

The UI should use the newly selected language.

Your in-app selector and Android's per-app language settings should represent the same underlying app language preference.

---

# 📝 30. Handle Strings Containing Dynamic Information

Not every String is static.

Consider:

```text
Welcome, Talia!
```

The user's name is dynamic.

Avoid building localised sentences using several hardcoded pieces.

For example, avoid:

```text
"Welcome " + name
```

Instead, create a formatted String resource.

The default resource might contain:

```xml
<string name="welcome_user">
    Welcome, %1$s!
</string>
```

Each language can then define its own translation while keeping the placeholder.

The Compose UI supplies the user's name when retrieving the resource.

This allows each language to control its own sentence structure.

### 📚 Helpful Resource

**Android — String Resources:**  
https://developer.android.com/guide/topics/resources/string-resource

---

# 🔢 31. Understand Plural Resources

Consider:

```text
1 parking bay available
```

and:

```text
5 parking bays available
```

Different languages may have different pluralisation rules.

Do not assume that adding:

```text
s
```

is sufficient.

Android supports plural resources using:

```text
<plurals>
```

For example:

```xml
<plurals name="parking_bays_available">

    <item quantity="one">
        %d parking bay available
    </item>

    <item quantity="other">
        %d parking bays available
    </item>

</plurals>
```

For applications with quantities, create translated plural resources for each supported language.

### 📚 Helpful Resource

**Android — Quantity Strings:**  
https://developer.android.com/guide/topics/resources/string-resource#Plurals

---

# 📅 32. Remember That Localisation Includes Dates and Times

Localisation is not only:

```text
English -> Afrikaans
```

Regional preferences can also affect:

```text
Dates

Times

Numbers

Currency
```

For example:

```text
25/08/2026
```

and:

```text
08/25/2026
```

can represent different regional formatting conventions.

Avoid manually hardcoding regional display formats without considering the user's locale.

---

# 💰 33. Localisation Also Affects Numbers and Currency

Different locales may use different conventions for:

```text
Decimal separators

Thousands separators

Currency symbols

Currency positioning
```

For example, a value may be displayed differently depending on locale.

When your applications later work with currency, measurements or numbers, use locale-aware formatting rather than manually inserting symbols and separators.

---

# ↔️ 34. Consider Right-to-Left Languages

English, Afrikaans and isiZulu are written left-to-right.

Other languages, such as Arabic and Hebrew, are commonly written right-to-left.

Android supports RTL layout behaviour.

Your manifest should normally support:

```xml
android:supportsRtl="true"
```

When creating layouts, prefer directional terms such as:

```text
start

end
```

rather than assuming:

```text
left

right
```

This makes the application easier to adapt to RTL languages later.

---

# 🧪 35. Test Missing Translations

Temporarily remove one non-essential translated String from one language resource file.

Run the application using that language.

Observe what Android does.

Android can fall back to default resources when a more specific translation is unavailable.

However, your final submission should aim to provide complete translations for important user-facing text.

---

# 🚫 36. Do Not Implement Language Switching Manually

Avoid code structures conceptually similar to:

```text
if Afrikaans
-> manually change every Text

else if isiZulu
-> manually change every Text

else
-> manually use English
```

This does not scale.

Imagine an app containing:

```text
30 screens

100 Strings

5 languages
```

Android's resource system exists to avoid this problem.

Use:

```text
Resource IDs
+
Locale-specific resources
```

instead.

---

# 🐛 Common Problems

## Some Text Does Not Change

Check whether the text is hardcoded.

Search your Compose files for visible text that does not come from a String resource.

---

## Afrikaans Does Not Load

Check the directory name.

Use:

```text
values-af
```

not:

```text
values-afrikaans
```

---

## isiZulu Does Not Load

Use:

```text
values-zu
```

not:

```text
values-zulu
```

---

## App Language Option Does Not Appear in Android Settings

Check:

- Device uses Android 13 or later.
- `locales_config.xml` exists.
- Locale tags are valid.
- Manifest references `android:localeConfig`.
- Application has been rebuilt/reinstalled where necessary.

---

## Application Does Not Change Language

Check:

- AppCompat dependency is installed.
- App uses an AppCompat-compatible Activity.
- Correct language tag is passed.
- Application locale API is being called.
- Translated resources exist.

---

## Language Changes but Resets After Restart

Check the AppCompat automatic locale storage configuration for versions below Android 13.

---

## App Recreates When Language Changes

This is expected.

Locale changes are configuration changes.

Android may recreate the Activity so that resources can be loaded for the new locale. :contentReference[oaicite:9]{index=9}

---

## Some Strings Fall Back to English

Check whether the resource exists inside the translated `strings.xml`.

If it does not, Android may use the default resource.

---

# 🧪 Student Extension Activity

Once English, Afrikaans and isiZulu work, add one additional language.

Choose one of:

```text
French

German

Spanish

Portuguese
```

You must determine:

- The correct language tag.
- The correct `values-...` resource directory.
- The translated String resources.
- The new entry required in `locales_config.xml`.
- The additional language option required in the UI.
- How your language selector should apply the new locale.

Test that all existing languages still work after the new language is added.

---

# 📚 Helpful Resources

## 🌍 Android Localisation

**Android — Localise Your App:**  
https://developer.android.com/guide/topics/resources/localization

Use this to understand:

```text
values/

values-af/

values-zu/

resource fallback

locale-specific resources
```

---

## 📱 Per-App Languages

**Android — Per-App Language Preferences:**  
https://developer.android.com/guide/topics/resources/app-languages

Use this for the overall Android per-app language approach.

**Android 13 — App Languages:**  
https://developer.android.com/about/versions/13/features#app-languages

Android 13 provides system support for app-specific language preferences. :contentReference[oaicite:10]{index=10}

---

## 🧩 AppCompat Locale Support

**AppCompatDelegate:**  
https://developer.android.com/reference/kotlin/androidx/appcompat/app/AppCompatDelegate

Focus on:

```text
setApplicationLocales()

getApplicationLocales()
```

`setApplicationLocales()` accepts a `LocaleListCompat`; passing an empty list resets the application to the system locale. :contentReference[oaicite:11]{index=11}

---

## 🌐 Locale Lists

**LocaleListCompat:**  
https://developer.android.com/reference/androidx/core/os/LocaleListCompat

Focus on:

```text
forLanguageTags()

getEmptyLocaleList()
```

---

## 📜 Supported Locale Configuration

**LocaleConfig:**  
https://developer.android.com/reference/android/app/LocaleConfig

Use this to understand:

```text
locales_config.xml

BCP 47 language tags
```

The locale configuration can be supplied through an XML `<locale-config>` resource referenced by `android:localeConfig`. :contentReference[oaicite:12]{index=12}

---

## 📝 String Resources

**Android — String Resources:**  
https://developer.android.com/guide/topics/resources/string-resource

Use this for:

```text
<string>

formatted Strings

placeholders

plurals
```

---

## 🎨 Compose Resources

**Android — Resources in Compose:**  
https://developer.android.com/develop/ui/compose/resources

Focus on using Android resources from Jetpack Compose rather than hardcoded UI Strings.

---


The key principle is:

```text
Compose requests a String resource
-> Android decides which translation to provide
```

Your UI should not be responsible for manually translating itself.

That separation makes the application easier to maintain and allows additional languages to be added without rewriting every screen.
