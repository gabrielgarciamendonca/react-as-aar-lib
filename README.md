# Generate .AAR of React Native

Crie seu projeto utilizando o react-native cli, pelo npx:

```jsx
npx react-native init nome-do-projeto
```

antes de tudo dentro de build.gradle(module), altere o nome `apply plugin: "com.android.application"` para `apply plugin: "com.android.library"` e remova a propriedade `applicationId` de `defaultConfig`.

e também mude o nome de `applicationVariants` para `libraryVariants`.

Agora, edite as dependências, deixe apenas as 3 dependências abaixo.

```kotlin
dependencies {
	//noinspection GradleDynamicVersion
  implementation("com.facebook.react:react-android:+")
  implementation("androidx.swiperefreshlayout:swiperefreshlayout:1.1.0")
  //noinspection GradleDynamicVersion
  implementation("com.facebook.react:hermes-android:+")
}
```

após alterar a implementação, a próxima alteração será no mesmo arquivo, você irá adicionar dentro de android, as seguintes opções de pacotes:

```kotlin
android {
	compileSdkVersion getExtOrIntegerDefault("compileSdkVersion")

	packagingOptions {
	    pickFirst 'lib/x86/libc++_shared.so'
	    pickFirst 'lib/x86_64/libc++_shared.so'
	    pickFirst 'lib/armeabi-v7a/libc++_shared.so'
	    pickFirst 'lib/arm64-v8a/libc++_shared.so'
	    pickFirst 'lib/arm64-v8a/libfbjni.so'
	    pickFirst 'lib/armeabi-v7a/libfbjni.so'
	    pickFirst 'lib/x86/libfbjni.so'
	    pickFirst 'lib/x86_64/libfbjni.so'
	}
	...
}
```

muito bem, agora, feito isso, você poderá ir para a próxima etapa, que será no build.gradle(projeto), abaixo de buildScript, você irá adicionar em allProjects o código abaixo:

```kotlin
allprojects {
  repositories {
    google()
    mavenCentral()
    maven { url "https://jitpack.io" }
  }
}
```

Agora, no Manifest da aplicação, remova tudo e deixe apenas a tag manifest:

```kotlin
<manifest xmlns:android="http://schemas.android.com/apk/res/android"></manifest>
```

Remova o AndroidManifest de debug.

Agora na pasta res(resources) do projeto, remova tudo dentro dela. Afinal, você não irá utilizar nada dela.

Agora remova tudo de dentro do com.[nome-do-projeto](debug), seu projeto ficará mais ou menos parecido com esse:

 

![Capturar.PNG](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f0aa16a-f8b9-48d0-bb5e-dc858bd84e98/Capturar.png)

Obs.: Algumas dependencias da MainApplication irá quebrar, corrigindo elas, seu arquivo ficará parecido com esse:

```kotlin
private final ReactNativeHost mReactNativeHost =
      new DefaultReactNativeHost(this) {
        @Override
        public boolean getUseDeveloperSupport() {
          return false;
        }

        @Override
        protected List<ReactPackage> getPackages() {
          @SuppressWarnings("UnnecessaryLocalVariable")
          List<ReactPackage> packages = new PackageList(this).getPackages();
          // Packages that cannot be autolinked yet can be added manually here, for example:
          // packages.add(new MyReactNativePackage());
          return packages;
        }

        @Override
        protected String getJSMainModuleName() {
          return "index";
        }

        @Override
        protected boolean isNewArchEnabled() {
          return false;
        }

        @Override
        protected Boolean isHermesEnabled() {
          return true;
        }
      };

  @Override
  public ReactNativeHost getReactNativeHost() {
    return mReactNativeHost;
  }

  @Override
  public void onCreate() {
    super.onCreate();
    SoLoader.init(this, /* native exopackage */ false);
  }
```

pronto, agora, será necessário gerar um bundle do seu projeto RN, fora da pasta android do projeto, digite os seguintes comandos:

```kotlin
mkdir android/app/src/main/assets
npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res
```

pronto, agora você poderá utilizar os seguintes comandos para gerar seu tão aguardado .aar:

```kotlin
./gradlew clean
./gradlew assembleRelease
```

ele estará em: [nome-do-projeto]\android\build\outputs\aar.




# Using .AAR of React Native in a Android Project

Crie seu projeto Android, pelo android studio. E dentro do build.gradle(module) na propriedade `dependencies` inclua a seguinte linha:

```kotlin
implementation files('[localização-do-seu-aar]/[nome-do-seu-aar].aar')
```

Feito isso, sincronize o gradle e pronto, sua lib está instalada no projeto Android.

Mas como não é 1000 maravilhas, irá ocorrer alguns erros, para corrigi-los, vá nas dependencias do projeto build.gradle(module) e adicione as seguintes dependencias:

```kotlin
dependencies {
		...
    //noinspection GradleDependency
    implementation 'androidx.appcompat:appcompat:1.1.0-rc01'
    //noinspection GradleDependency
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0-alpha02'
    //noinspection GradleDynamicVersion
    implementation 'com.facebook.react:react-android:+'
    //noinspection GradleDynamicVersion
    implementation 'org.webkit:android-jsc:+'
    //noinspection GradleDynamicVersion
    implementation 'com.facebook.react:hermes-android:+'
    //noinspection GradleDependency,GradleDynamicVersion
    implementation 'com.facebook.soloader:soloader:0.9.0+'
}
```

Crie uma ReactApplication para extender sua MainApplication incluida pela lib, ficará parecido com isso:

```kotlin
package [package-do-seu-projeto]

class ReactApplication: com.[nome-da-lib].MainApplication() {
}
```

Antes de prosseguir, na MainActivity, substitua o código por isso:

```kotlin
package [package-do-seu-projeto]

class MainActivity : com.[nome-da-lib].MainActivity() {

}
```

E por hora, eu ainda não descobri como utilizar duas applications no mesmo projeto, então eu deletei a application gerada automaticamente e inclui uma nova application no arquivo Manifest.xml:

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.asgard.munnin"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:name=".ReactApplication"
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:allowBackup="false"
        android:theme="@style/Platform.MaterialComponents.Light"
        tools:ignore="DataExtractionRules">
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:configChanges="keyboard|keyboardHidden|orientation|screenLayout|screenSize|smallestScreenSize|uiMode"
            android:launchMode="singleTask"
            android:windowSoftInputMode="adjustResize"
            android:exported="true"
            tools:ignore="WrongManifestParent">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

Inicie o projeto e seja feliz 😊!
