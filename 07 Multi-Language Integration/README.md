# 🌍 Multi-Language Support in Android Using ML Kit

In this activity, you will add **multi-language support** to an Android application.

The application should support:

```text
English
Afrikaans
isiZulu
```

However, you will **not use the same translation method for every language**.

You will combine Android's normal localisation system with **Google ML Kit Translation**:

```text
English
-> Default Android string resources

Afrikaans
-> Google ML Kit automatic translation

isiZulu
-> Android string resource fallback
```

This hybrid approach is necessary because ML Kit Translation currently supports **Afrikaans**, but does **not currently support isiZulu**.

The user should be able to:

* Open the application.
* View all text in the currently selected language.
* Choose between English, Afrikaans and isiZulu.
* Automatically translate the English interface into Afrikaans using ML Kit.
* Use pre-translated Android resources when isiZulu is selected.
* Switch between the three supported languages.
* Return to English when required.
* Close and reopen the application without unnecessarily losing the selected app language.
* Receive appropriate feedback while a translation model is downloading or text is being translated.
* Continue using the application if automatic translation fails.

The overall intended flow is:

```text
Application starts
-> Current language is determined
-> Correct text is loaded
-> User chooses another language

-> English selected
   -> Load default English resources

-> Afrikaans selected
   -> Retrieve English source strings
   -> Prepare ML Kit Afrikaans translation model
   -> Automatically translate English text
   -> Update interface with translated text

-> isiZulu selected
   -> Change Android application locale to zu
   -> Load values-zu/strings.xml
   -> Interface appears in isiZulu
```

---

# 🧠 1. Understand Internationalisation and Localisation

Before implementing anything, understand the two main concepts involved.

## 🌐 Internationalisation

**Internationalisation** means designing an application so that it **can support different languages and regional formats**.

It is often abbreviated as:

```text
i18n
```

Examples of internationalisation include:

```text
Moving visible text into resource files

Avoiding hardcoded user-facing Strings

Supporting different date formats

Supporting different number formats

Supporting different currencies

Supporting right-to-left layouts

Designing layouts that can accommodate longer translated text
```

For example, this is not ideal:

```kotlin
Text(
    text = "Welcome to our application"
)
```

The English sentence has been hardcoded directly into the Kotlin file.

A better approach is to store user-facing text in Android resources:

```xml
<string name="welcome_message">
    Welcome to our application
</string>
```

The application is then being designed in a way that makes localisation possible.

---

## 🗣️ Localisation

**Localisation** means adapting the application for a particular language or region.

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

[Android — Localise Your App](https://developer.android.com/guide/topics/resources/localization?)

---

# 🧩 2. Understand the Approaches to Multi-Language Support

There are several ways an Android application can support multiple languages.

Understanding these approaches is important because this activity will use **more than one**.

## 📱 Option 1 — Follow the Device Language

The application can provide translated Android resources and allow Android to automatically choose the appropriate resource set.

For example:

```text
Device language:
isiZulu

->

Android searches for:
values-zu/

->

Correct isiZulu resources are loaded
```

This is the traditional Android localisation approach.

---

## ⚙️ Option 2 — Android Per-App Language Settings

Modern Android versions support language preferences for individual applications.

For example, the user's phone could use:

```text
Device:
English
```

while one application uses:

```text
Application:
isiZulu
```

Android 13 and later provide system-level support for per-app language preferences when applications correctly declare their supported locales.

### 📚 Helpful Resource

[Android — Per-App Language Preferences](https://developer.android.com/guide/topics/resources/app-languages?)

---

## 👆 Option 3 — In-App Language Selector

An application can provide its own language selection interface.

For example:

```text
Choose Language

[ English ]

[ Afrikaans ]

[ isiZulu ]
```

The application then responds to the user's selection.

This is the approach you will implement in this activity.

---

## 🤖 Option 4 — Automatic Translation with ML Kit

Instead of manually creating a translated resource file for every supported language, Google ML Kit can automatically translate supported languages.

For example:

```text
English source:

Welcome to our multi-language app.

        |

        v

Google ML Kit Translation

        |

        v

Afrikaans result
```

ML Kit uses downloadable translation models.

Once the required model is available on the device, translation can happen **on-device**.

### 📚 Helpful Resource

[Google ML Kit — Translate Text on Android](https://developers.google.com/ml-kit/language/translation/android?)

---

# 🔀 3. Understand the Hybrid Approach Used in This Activity

This activity deliberately combines the approaches above.

You will implement:

```text
                    LANGUAGE SELECTED
                           |
          -------------------------------------
          |                 |                 |
          v                 v                 v
       ENGLISH          AFRIKAANS          ISIZULU
          |                 |                 |
          v                 v                 v
    strings.xml        ML Kit             values-zu
                         |                strings.xml
                         v                   |
                  Automatic Translation      |
                         |                   |
                         ---------   ----------
                                  | |
                                  v v
                              COMPOSE UI
```

### English

English is the application's **default/source language**.

```text
English
-> res/values/strings.xml
```

### Afrikaans

Afrikaans will demonstrate **automatic runtime translation**.

```text
Afrikaans
-> English source strings
-> ML Kit
-> Afrikaans translation
-> Compose UI
```

You will **not manually create**:

```text
values-af/strings.xml
```

for this activity.

### isiZulu

ML Kit Translation does not currently support isiZulu.

Therefore:

```text
isiZulu
-> Android localisation
-> values-zu/strings.xml
```

This is the application's **fallback translation strategy**.

### 📚 Helpful Resource

[ML Kit — Supported Translation Languages](https://developers.google.com/ml-kit/language/translation/translation-language-support?)

---

# 🏗️ 4. Create or Open Your Android Project

You may continue using the Android project created during the previous localisation demonstration.

Your application should use:

```text
Kotlin

Jetpack Compose

Material 3

Minimum SDK 23 or higher
```

ML Kit Translation requires API 23 or higher.

Check your module-level:

```text
app/build.gradle.kts
```

and ensure your project's minimum SDK is compatible.

For example:

```kotlin
android {

    defaultConfig {

        /*
         * ML Kit Translation requires API 23 or higher.
         */
        minSdk = 23
    }
}
```

If your project already uses a higher minimum SDK, **do not lower it**.

---

# 📦 5. Add the Required Dependencies

Open:

```text
app/build.gradle.kts
```

Locate:

```kotlin
dependencies {

}
```

## 🤖 Add ML Kit Translation

Add:

```kotlin
/*
 * Google ML Kit Translation
 *
 * This dependency allows the application to automatically
 * translate text between supported languages.
 *
 * In this activity it will be used for:
 *
 * English -> Afrikaans
 */
implementation("com.google.mlkit:translate:17.0.3")
```

---

## 🌐 Add AppCompat

Also add:

```kotlin
/*
 * AppCompat
 *
 * This will be used for Android's application locale
 * functionality.
 *
 * It is particularly important for our isiZulu fallback,
 * where Android will load values-zu/strings.xml.
 */
implementation("androidx.appcompat:appcompat:1.7.1")
```

Your dependencies should therefore contain:

```kotlin
dependencies {

    /*
     * Existing Compose and Android dependencies will
     * remain here.
     */


    /*
     * Used for automatic English -> Afrikaans translation.
     */
    implementation(
        "com.google.mlkit:translate:17.0.3"
    )


    /*
     * Used for Android application locale management.
     */
    implementation(
        "androidx.appcompat:appcompat:1.7.1"
    )
}
```

Do **not** remove your existing Compose dependencies.

After adding the dependencies:

```text
File
-> Sync Project with Gradle Files
```

Wait for Gradle to finish successfully before continuing.

---

# 🌐 6. Add Internet Permission

ML Kit performs translation on the device, but the required language model may first need to be downloaded.

Open:

```text
app/
└── src/
    └── main/
        └── AndroidManifest.xml
```

Above `<application>`, add:

```xml
<!--
    ML Kit may need Internet access to download
    the Afrikaans translation model.

    The model does not necessarily already exist
    on the user's device.
-->
<uses-permission
    android:name="android.permission.INTERNET" />
```

The basic structure should resemble:

```xml
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android">

    <!--
        Allows ML Kit to download translation models.
    -->
    <uses-permission
        android:name="android.permission.INTERNET" />


    <application

        ... >

    </application>

</manifest>
```

You do **not** need to request Internet permission from the user at runtime.

---

# 🇬🇧 7. Create the Default English `strings.xml`

English will be the application's default language and the **source language used by ML Kit**.

Open:

```text
app/
└── src/
    └── main/
        └── res/
            └── values/
                └── strings.xml
```

Use:

```xml
<?xml version="1.0" encoding="utf-8"?>

<resources>

    <!--
        DEFAULT LANGUAGE

        The values folder contains the application's
        default resources.

        English is our default/source language.

        These Strings will also be used as the source text
        when ML Kit translates the interface into Afrikaans.
    -->

    <string name="app_name">
        Language Demo
    </string>


    <!-- Screen content -->

    <string name="screen_title">
        Language Settings
    </string>

    <string name="welcome_message">
        Welcome to our multi-language app.
    </string>

    <string name="description">
        Choose your preferred language.
    </string>

    <string name="choose_language">
        Choose language:
    </string>


    <!-- Language button labels -->

    <string name="english">
        English
    </string>

    <string name="afrikaans">
        Afrikaans
    </string>

    <string name="zulu">
        isiZulu
    </string>


    <!-- Current language -->

    <string name="current_language">
        Current Language
    </string>


    <!-- Translation status -->

    <string name="preparing_translation">
        Preparing translation...
    </string>

    <string name="translation_failed">
        Automatic translation failed.
    </string>


    <!-- Device/system language option -->

    <string name="use_system_language">
        Use device language
    </string>

</resources>
```

---

# 🚫 8. Do Not Create an Afrikaans `strings.xml`

Normally Android localisation would use:

```text
values/
    strings.xml

values-af/
    strings.xml

values-zu/
    strings.xml
```

However, that would defeat the purpose of demonstrating ML Kit automatic translation.

For this activity, **do not create**:

```text
values-af/
```

Afrikaans should instead follow:

```text
values/strings.xml
        |
        v
English source text
        |
        v
ML Kit Translation
        |
        v
Afrikaans text
        |
        v
Compose UI
```

This allows you to demonstrate automatic translation rather than manually storing every Afrikaans translation.

---

# 🇿🇦 9. Create the isiZulu Fallback

ML Kit Translation does not currently provide an isiZulu translation model.

Therefore, isiZulu will use Android's normal localisation system.

Create:

```text
res/
└── values-zu/
    └── strings.xml
```

In Android Studio, you can create the directory using:

```text
res
-> New
-> Android Resource Directory
```

Choose:

```text
Resource type:
values

Locale:
zu
```

Your resources should now resemble:

```text
res/
│
├── values/
│   └── strings.xml
│
└── values-zu/
    └── strings.xml
```

---

# 📝 10. Add the isiZulu Strings

The **resource names must remain identical** to those in the English file.

For example, English uses:

```xml
<string name="screen_title">
    Language Settings
</string>
```

Therefore, isiZulu must also use:

```xml
<string name="screen_title">
    ...
</string>
```

Do **not** create:

```xml
<string name="screen_title_zulu">
```

Android knows which translation to load from the directory qualifier.

Use:

```xml
<?xml version="1.0" encoding="utf-8"?>

<resources>

    <!--
        ISIZULU FALLBACK

        ML Kit Translation does not currently support isiZulu.

        Therefore Android's normal resource localisation
        system is used for this language.
    -->

    <string name="app_name">
        Uhlelo Lwezilimi
    </string>

    <string name="screen_title">
        Izilungiselelo Zolimi
    </string>

    <string name="welcome_message">
        Siyakwamukela kuhlelo lwethu lwezilimi eziningi.
    </string>

    <string name="description">
        Khetha ulimi oluthandayo.
    </string>

    <string name="choose_language">
        Khetha ulimi:
    </string>


    <!-- Language buttons -->

    <string name="english">
        IsiNgisi
    </string>

    <string name="afrikaans">
        IsiBhunu
    </string>

    <string name="zulu">
        isiZulu
    </string>


    <!-- Current language -->

    <string name="current_language">
        Ulimi Lwamanje
    </string>


    <!-- Translation messages -->

    <string name="preparing_translation">
        Kulungiselelwa ukuhumusha...
    </string>

    <string name="translation_failed">
        Ukuhumusha akuphumelelanga.
    </string>


    <!-- Device language -->

    <string name="use_system_language">
        Sebenzisa ulimi lwedivayisi
    </string>

</resources>
```

For a production application, translated content should be reviewed by a fluent speaker.

---

# 🔎 11. Understand How Android Finds the Correct Resources

At this stage:

```text
res/
│
├── values/
│   └── strings.xml
│
└── values-zu/
    └── strings.xml
```

The directory names have meaning.

## `values`

```text
values/
```

contains the default resources.

In this application:

```text
values
-> English
```

## `values-zu`

The:

```text
-zu
```

qualifier represents the isiZulu language code.

Therefore:

```text
values-zu
-> isiZulu
```

If Android's application locale becomes:

```text
zu
```

Android automatically attempts to load resources from:

```text
values-zu/
```

You do not manually open the XML file from Kotlin.

---

# 🌍 12. Declare the Supported Locales

Create:

```text
res/
└── xml/
    └── locales_config.xml
```

If `xml` does not exist:

```text
res
-> New
-> Android Resource Directory
-> Resource type: xml
```

Then create:

```text
locales_config.xml
```

Add:

```xml
<?xml version="1.0" encoding="utf-8"?>

<locale-config
    xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- English -->
    <locale android:name="en" />

    <!--
        Afrikaans is supported by the application.

        In this activity the actual text translation
        is performed dynamically using ML Kit.
    -->
    <locale android:name="af" />

    <!--
        isiZulu uses Android resource localisation.
    -->
    <locale android:name="zu" />

</locale-config>
```

The language tags are:

| Language  |  Tag |
| --------- | ---: |
| English   | `en` |
| Afrikaans | `af` |
| isiZulu   | `zu` |

---

# 📜 13. Register the Locale Configuration

Open:

```text
AndroidManifest.xml
```

Inside `<application>`, add:

```xml
android:localeConfig="@xml/locales_config"
```

For example:

```xml
<application

    android:localeConfig="@xml/locales_config"

    ... >

</application>
```

This tells Android which locales your application supports.

---

# 💾 14. Configure AppCompat Locale Storage

Inside `<application>`, add:

```xml
<!--
    Allows AppCompat to store application locale
    information where required on older Android versions.
-->
<service
    android:name="androidx.appcompat.app.AppLocalesMetadataHolderService"
    android:enabled="false"
    android:exported="false">

    <meta-data
        android:name="autoStoreLocales"
        android:value="true" />

</service>
```

Do not replace your complete Manifest.

Add these elements to your **existing** Manifest.

---

# 📱 15. Make `MainActivity` AppCompat-Compatible

Your activity may currently extend:

```kotlin
class MainActivity : ComponentActivity()
```

For this activity, use:

```kotlin
class MainActivity : AppCompatActivity()
```

Add:

```kotlin
import androidx.appcompat.app.AppCompatActivity
```

You can still use Compose normally:

```kotlin
setContent {

    // Your Compose UI

}
```

AppCompat is being introduced because you will work with Android's application locale functionality.

---

# 🗂️ 16. Organise the Project

Do not place the entire implementation inside `MainActivity.kt`.

Your project should eventually resemble:

```text
java/com/example/languagedemo/
│
├── MainActivity.kt
├── LanguageScreen.kt
├── LanguageUiState.kt
├── MlKitTranslationManager.kt
└── AppLanguageManager.kt
```

Each class/file should have a specific responsibility.

### `MainActivity.kt`

Coordinates the application.

It should respond to:

```text
English selected
Afrikaans selected
isiZulu selected
Device language selected
```

### `LanguageScreen.kt`

Contains the Jetpack Compose interface.

### `LanguageUiState.kt`

Stores the text currently displayed by the interface.

### `MlKitTranslationManager.kt`

Handles communication with ML Kit Translation.

### `AppLanguageManager.kt`

Handles Android application locale changes.

---

# 🧠 17. Create the UI State

Create:

```text
LanguageUiState.kt
```

The UI needs somewhere to store the text currently being displayed.

You may begin with:

```kotlin
data class LanguageUiState(

    /*
     * Text displayed as the main screen heading.
     */
    val screenTitle: String,


    /*
     * Main welcome message.
     */
    val welcomeMessage: String,


    /*
     * Description displayed below the welcome message.
     */
    val description: String,


    /*
     * Continue adding the remaining Strings
     * required by your interface.
     */

)
```

Students must complete the remaining properties.

Consider:

```text
screen title

welcome message

description

choose-language label

English button label

Afrikaans button label

isiZulu button label

current-language label

device-language label
```

---

# 🤔 18. Why Do We Need UI State for Afrikaans?

With normal Android localisation, Compose can use:

```kotlin
stringResource(
    R.string.screen_title
)
```

Android then selects the correct resource.

But Afrikaans is different in this activity.

There is no:

```text
values-af/strings.xml
```

Instead:

```text
"Language Settings"
        |
        v
ML Kit
        |
        v
Afrikaans result
        |
        v
Store result in Compose state
        |
        v
Compose recomposes
```

Therefore, the UI needs somewhere to store the dynamically generated translations.

---

# 🤖 19. Create the ML Kit Translation Manager

Create:

```text
MlKitTranslationManager.kt
```

This class should be responsible specifically for **automatic translation**.

You will need classes including:

```kotlin
import com.google.mlkit.common.model.DownloadConditions

import com.google.mlkit.nl.translate.TranslateLanguage

import com.google.mlkit.nl.translate.Translation

import com.google.mlkit.nl.translate.Translator

import com.google.mlkit.nl.translate.TranslatorOptions
```

Do not place all ML Kit operations directly inside your Compose screen.

---

# ⚙️ 20. Configure the Translator

Research:

```text
TranslatorOptions.Builder()
```

Your translator must be configured as:

```text
SOURCE
English

TARGET
Afrikaans
```

ML Kit provides language constants such as:

```kotlin
TranslateLanguage.ENGLISH
```

and:

```kotlin
TranslateLanguage.AFRIKAANS
```

Your general configuration should therefore follow:

```text
TranslatorOptions
-> source = English
-> target = Afrikaans
-> build configuration
-> create Translator
```

### 📚 Helpful Resource

[ML Kit — Translation Implementation Guide](https://developers.google.com/ml-kit/language/translation/android?)

---

# 📥 21. Prepare the Afrikaans Translation Model

Creating a `Translator` does not necessarily mean that the Afrikaans model already exists on the device.

Your application must prepare it.

Research:

```text
downloadModelIfNeeded()
```

Your implementation should follow:

```text
User selects Afrikaans
        |
        v
Prepare Translator
        |
        v
Check Afrikaans model
        |
        v
     Available?
      /     \
    YES      NO
     |        |
     |     Download
     |        |
      \      /
        v   v
      Model ready
          |
          v
       Translate
```

---

# 📶 22. Configure Download Conditions

Research:

```kotlin
DownloadConditions.Builder()
```

For this classroom activity, the model may download using the available Internet connection.

In a production application, you may investigate:

```text
requireWifi()
```

This can be useful because translation models take up storage and downloading them uses data.

The first time Afrikaans is selected may therefore take longer than subsequent translations.

---

# 🔤 23. Implement the Translation Operation

Your translation manager should accept:

```text
English text
```

and eventually provide:

```text
Afrikaans text
```

Research:

```text
translator.translate(...)
```

ML Kit translation is **asynchronous**.

That means this assumption is incorrect:

```text
Call translate()
-> immediately receive String
```

The actual flow is:

```text
Request translation
        |
        v
ML Kit processes text
        |
        v
   ----------------
   |              |
Success         Failure
   |              |
   v              v
Use result    Handle error
```

Your implementation must handle **both success and failure**.

---

# 🔒 24. Release the Translator

ML Kit's `Translator` uses resources that should be released when they are no longer required.

Research:

```text
translator.close()
```

Consider the Android Activity lifecycle.

For example:

```text
Activity created
-> Translator used
-> Activity destroyed
-> Translator no longer required
-> Release resources
```

---

# 🌐 25. Create the Application Language Manager

Create:

```text
AppLanguageManager.kt
```

This component should handle Android's **resource-based locale system**.

Research:

```kotlin
LocaleListCompat.forLanguageTags(...)
```

and:

```kotlin
AppCompatDelegate.setApplicationLocales(...)
```

### Language tags

You will work with:

```text
English:
en

isiZulu:
zu
```

Afrikaans is intentionally handled differently because it is being translated dynamically with ML Kit.

### 📚 Helpful Resources

[AppCompatDelegate Reference](https://developer.android.com/reference/androidx/appcompat/app/AppCompatDelegate?)

[LocaleListCompat Reference](https://developer.android.com/reference/androidx/core/os/LocaleListCompat?)

---

# 📱 26. Build the Compose Screen

Create:

```text
LanguageScreen.kt
```

Your interface should contain at least:

```text
Language Settings

Welcome to our multi-language app.

Choose your preferred language.

Choose language:

[ English ]

[ Afrikaans ]

[ isiZulu ]

[ Use Device Language ]

Current Language: English
```

You may design the interface yourself.

The exact:

```text
colours
spacing
typography
button arrangement
icons
```

are not prescribed.

However, the user must clearly be able to select each supported language.

---

# 🔗 27. Connect `LanguageUiState` to the Screen

Your Compose screen should receive a:

```text
LanguageUiState
```

rather than hardcoding all text.

Conceptually:

```text
LanguageUiState
        |
        v
LanguageScreen
        |
        v
Compose Text / Buttons
```

When ML Kit completes translation:

```text
Old English UI State
        |
        v
New Afrikaans UI State
        |
        v
Compose detects state change
        |
        v
Recomposition
        |
        v
Afrikaans interface appears
```

---

# 🇬🇧 28. Implement English Selection

When:

```text
English
```

is selected, the application should return to the English resource set.

The flow should be:

```text
User selects English
-> application locale becomes en
-> Android loads default resources
-> values/strings.xml
-> English UI displayed
```

Do not create another copy of all the English text inside Kotlin.

Your source already exists in:

```text
values/strings.xml
```

---

# 🤖 29. Implement Afrikaans Selection

This is the main ML Kit part of the activity.

When the user selects:

```text
Afrikaans
```

the intended process is:

```text
User selects Afrikaans
        |
        v
Obtain ORIGINAL English Strings
        |
        v
Check Afrikaans ML Kit model
        |
        v
Download if required
        |
        v
Translate English Strings
        |
        v
Collect translation results
        |
        v
Update LanguageUiState
        |
        v
Compose recomposes
        |
        v
Afrikaans displayed
```

The source text should come from resources such as:

```text
R.string.screen_title

R.string.welcome_message

R.string.description

R.string.choose_language

R.string.english

R.string.afrikaans

R.string.zulu

R.string.current_language
```

---

# 📚 30. Translate a Collection of User-Facing Strings

ML Kit does **not automatically scan your Compose screen**.

It does not do this:

```text
Find every Text()
-> translate it automatically
```

Your application must provide the strings to be translated.

You may start by creating a collection:

```kotlin
val stringsToTranslate = listOf(

    /*
     * Retrieve the original English Strings
     * from Android resources.
     */

    getString(R.string.screen_title),

    getString(R.string.welcome_message),

    getString(R.string.description)

    // Continue with the remaining Strings.
)
```

Students must determine which remaining user-facing strings need translation.

---

# ⏳ 31. Wait for All Translations

Suppose your interface contains eight strings.

ML Kit translation operations are asynchronous.

You should therefore not assume:

```text
Translation 1
-> finishes first

Translation 2
-> finishes second

Translation 3
-> finishes third
```

Your implementation should preserve the relationship between each source string and its translation.

Ideally:

```text
Start translation requests
        |
        v
Collect translated results
        |
        v
Have all required translations completed?
        |
     ---------
     |       |
    NO      YES
     |       |
    Wait     v
        Create new UI state
             |
             v
        Update screen
```

---

# 🎨 32. Avoid a Half-Translated Interface

Do not immediately update the screen every time one translation finishes.

Otherwise, users may briefly see:

```text
Afrikaans heading

English description

Afrikaans button

English label

Afrikaans message
```

Instead:

```text
Translate required Strings
-> collect results
-> wait for completion
-> update LanguageUiState once
-> complete Afrikaans interface appears
```

---

# ⏱️ 33. Add a Loading State

The first Afrikaans translation may require a language model download.

The interface should tell the user that something is happening.

For example:

```text
Preparing translation...

        ⟳
```

Jetpack Compose provides:

```kotlin
CircularProgressIndicator()
```

Consider a state such as:

```text
isLoading
```

Your application should conceptually follow:

```text
Afrikaans selected
-> isLoading = true

Translation/model preparation
-> loading indicator displayed

Translation succeeds or fails
-> isLoading = false
```

---

# 🚫 34. Prevent Repeated Requests

While translation is occurring, users should not be able to repeatedly request another translation.

For example:

```text
Afrikaans
Afrikaans
Afrikaans
Afrikaans
```

should not result in four unnecessary translation processes.

Compose `Button` provides:

```kotlin
enabled = ...
```

Use your loading/application state to determine when controls should temporarily be disabled.

---

# 🇿🇦 35. Implement the isiZulu Fallback

When the user selects:

```text
isiZulu
```

do **not** send the English strings to ML Kit.

Instead:

```text
User selects isiZulu
        |
        v
Set application locale to zu
        |
        v
Android detects locale change
        |
        v
Android searches resources
        |
        v
values-zu/strings.xml
        |
        v
isiZulu interface displayed
```

This demonstrates your **fallback mechanism**.

---

# 🔄 36. Understand Activity Recreation

Changing the application locale can result in the Activity being recreated.

This is expected Android behaviour.

For example:

```text
Current locale:
en

        |

User selects isiZulu

        |

Locale:
zu

        |

Configuration changes

        |

Activity recreated

        |

values-zu resources loaded
```

Your application must therefore be able to reconstruct its interface correctly.

Do not treat Activity recreation during a locale change as an application failure.

---

# ⚠️ 37. Handle isiZulu -> Afrikaans Carefully

This is one of the most important problems in the activity.

Imagine:

```text
Current language:
isiZulu
```

Android is currently loading:

```text
values-zu/strings.xml
```

The user now selects:

```text
Afrikaans
```

Your ML Kit Translator is configured as:

```text
English
-> Afrikaans
```

Therefore, you should **not accidentally send isiZulu text into an English-source translator**.

Your solution needs to ensure:

```text
isiZulu currently active
        |
        v
User selects Afrikaans
        |
        v
Return to English source
        |
        v
Retrieve English Strings
        |
        v
ML Kit English -> Afrikaans
        |
        v
Display Afrikaans
```

Students must determine an appropriate way to manage this transition.

Possible concepts worth investigating include:

```text
ViewModel

Saved state

DataStore

pending language selection

application preferences
```

---

# 📱 38. Implement "Use Device Language"

Your language screen should also contain:

```text
Use Device Language
```

This option means:

> Stop forcing a specific application locale and allow Android to determine the appropriate language from the user's device/system preferences.

Research how an **empty locale list** can be used with:

```text
AppCompatDelegate.setApplicationLocales()
```

to return control to the system locale.

Consider what should happen if the device language is:

```text
English
```

versus:

```text
isiZulu
```

Also consider what should happen if the device uses a language for which your application does not provide translated resources.

Android should be able to fall back to the default resources where necessary.

---

# 💾 39. Preserve the Selected Language

Users should not unnecessarily lose their chosen application language every time they close the application.

Test:

```text
Choose isiZulu
-> close application
-> reopen application
```

and:

```text
Choose Afrikaans
-> close application
-> reopen application
```

There is an important difference here.

Resource-based application locales can be managed by Android/AppCompat.

However, your Afrikaans implementation is **dynamic ML Kit state**, not a `values-af` resource set.

Therefore, consider how your application should remember:

```text
Last selected language:
Afrikaans
```

and restore the appropriate behaviour when the application starts again.

An appropriate persistence mechanism may include:

```text
DataStore
```

Students should investigate how to persist a small language preference.

---

# ❌ 40. Handle Translation Errors

Automatic translation can fail.

Possible causes include:

```text
No Internet connection when model is required

Language model download failure

Insufficient device storage

Translation failure

Unexpected lifecycle changes
```

The application should **not crash**.

Instead, provide appropriate feedback.

You already have:

```xml
<string name="translation_failed">
    Automatic translation failed.
</string>
```

You should also log the underlying error during development.

---

# 🪵 41. Use Logcat to Follow the Process

Add meaningful Logcat messages while implementing the activity.

Your logs could show:

```text
LanguageDemo: Afrikaans selected

LanguageDemo: Checking Afrikaans model

LanguageDemo: Downloading model

LanguageDemo: Afrikaans model ready

LanguageDemo: Starting translation

LanguageDemo: Translation completed

LanguageDemo: isiZulu selected

LanguageDemo: Changing application locale to zu

LanguageDemo: Translation failed
```

This will help you understand the order in which asynchronous operations occur.

---

# 🧪 42. Test English

Start the application.

The default interface should display something similar to:

```text
Language Settings

Welcome to our multi-language app.

Choose your preferred language.

Choose language:

[ English ]

[ Afrikaans ]

[ isiZulu ]

[ Use Device Language ]

Current Language: English
```

Verify that the text comes from:

```text
res/values/strings.xml
```

---

# 🧪 43. Test Afrikaans Automatic Translation

Select:

```text
Afrikaans
```

Verify:

```text
Afrikaans selected
-> model checked
-> model downloaded if necessary
-> English Strings translated
-> translated results collected
-> UI state updated
-> Compose recomposes
-> Afrikaans displayed
```

There should deliberately be **no**:

```text
values-af/strings.xml
```

for this activity.

That is how you can demonstrate that the Afrikaans text is being generated dynamically.

---

# 📴 44. Test Afrikaans Offline

Once the Afrikaans model has successfully downloaded:

1. Close the application.
2. Disable Internet access on the emulator/device.
3. Reopen the application.
4. Select Afrikaans again.
5. Observe what happens.

This demonstrates the difference between:

```text
Cloud translation
```

and:

```text
On-device translation model
```

---

# 🧪 45. Test isiZulu

Select:

```text
isiZulu
```

Verify:

```text
Locale changes to zu
-> resources reload
-> values-zu/strings.xml selected
-> isiZulu displayed
```

ML Kit should **not** be required for this process.

---

# 🔄 46. Test Every Language Transition

Do not test only:

```text
English -> Afrikaans
```

Test all important combinations:

```text
English -> Afrikaans

Afrikaans -> English

English -> isiZulu

isiZulu -> English

Afrikaans -> isiZulu

isiZulu -> Afrikaans

English -> Device Language

isiZulu -> Device Language

Afrikaans -> Device Language
```

Pay particular attention to:

```text
isiZulu -> Afrikaans
```

because ML Kit expects the source text to be English.

---

# 🔁 47. Test Application Restart

For each language:

```text
Select language
-> close application
-> reopen application
-> check displayed language
```

Test:

```text
English

Afrikaans

isiZulu

Device language
```

Your application should behave consistently and should not unexpectedly display a different language.

---

# 📂 48. Expected Project Structure

Your completed project should resemble:

```text
app/
│
├── build.gradle.kts
│
└── src/
    └── main/
        │
        ├── AndroidManifest.xml
        │
        ├── java/com/example/languagedemo/
        │   │
        │   ├── MainActivity.kt
        │   ├── LanguageScreen.kt
        │   ├── LanguageUiState.kt
        │   ├── MlKitTranslationManager.kt
        │   └── AppLanguageManager.kt
        │
        └── res/
            │
            ├── values/
            │   └── strings.xml
            │
            ├── values-zu/
            │   └── strings.xml
            │
            └── xml/
                └── locales_config.xml
```

Notice that there is intentionally **no**:

```text
values-af/
```

Afrikaans is being used to demonstrate ML Kit automatic translation.

---

# 🧠 49. Architecture

By the end of the activity, you should understand this architecture:

```text
                       USER SELECTS LANGUAGE
                                |
        ------------------------------------------------
        |                  |                 |          |
        v                  v                 v          v
     ENGLISH           AFRIKAANS          ISIZULU    SYSTEM
        |                  |                 |          |
        v                  v                 v          v
  values/strings      English source     Locale zu   Clear app
        |                  |                 |        locale
        |                  v                 v          |
        |               ML KIT          values-zu      |
        |                  |                 |          |
        |                  v                 |          |
        |          Automatic Afrikaans      |          |
        |              Translation           |          |
        |                  |                 |          |
        --------------------                 |          |
                 |                           |          |
                 ---------------------------------------
                                |
                                v
                           COMPOSE UI
```

The most important lesson from this activity is that **multi-language support does not have to use one single technique**.

You are combining:

```text
Android localisation
+
Android per-app languages
+
ML Kit automatic translation
+
Fallback resources
+
Compose state management
```

This gives you an opportunity to compare **traditional localisation** with **automatic runtime translation** and understand the advantages and limitations of both.

# 📚 Helpful Resources

### 🤖 ML Kit Translation

[Google ML Kit — Translate Text on Android](https://developers.google.com/ml-kit/language/translation/android?)

### 🌍 ML Kit Supported Languages

[Google ML Kit — Translation Language Support](https://developers.google.com/ml-kit/language/translation/translation-language-support?)

### 📱 Android Localisation

[Android — Localise Your App](https://developer.android.com/guide/topics/resources/localization?)

### ⚙️ Per-App Languages

[Android — Per-App Language Preferences](https://developer.android.com/guide/topics/resources/app-languages?)

### 🌐 AppCompat Locale Management

[Android — AppCompatDelegate Reference](https://developer.android.com/reference/androidx/appcompat/app/AppCompatDelegate?)

### 🏷️ Locale Lists

[Android — LocaleListCompat Reference](https://developer.android.com/reference/androidx/core/os/LocaleListCompat?)

### 🎨 Compose String Resources

[Android — Resources in Jetpack Compose](https://developer.android.com/develop/ui/compose/resources?)
