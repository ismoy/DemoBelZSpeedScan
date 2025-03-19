# BelZSpeedScan
### A Cross-Platform QR Code and Barcode Scanning Library, using MLKIT for decoding.

BelZSpeedScan is a lightweight and easy-to-use library for scanning QR codes and barcodes. It supports both Kotlin Multiplatform (KMP) and native Android development, providing a consistent API across platforms.  This allows you to use the same scanning logic in your shared KMP code and seamlessly integrate it into your Android application.


[![Buy Me a Coffee](https://img.buymeacoffee.com/button-api/?text=Buy%20Me%20a%20Coffee&emoji=☕&slug=ismoy&button_colour=FFDD00&font_colour=000000&font_family=Arial&outline_colour=000000&coffee_colour=ffffff)](https://www.buymeacoffee.com/Ismoy)

[![Maven Central](https://img.shields.io/maven-central/v/io.github.ismoy/kmpswipe.svg)](https://search.maven.org/artifact/io.github.ismoy/kmpswipe)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![API](https://img.shields.io/badge/API-21%2B-brightgreen.svg)](https://android-arsenal.com/api?level=21)

[![Kotlin](https://img.shields.io/badge/Kotlin-1.9.0-blue.svg?logo=kotlin)](https://kotlinlang.org)
[![Compose](https://img.shields.io/badge/Compose-1.5.0-green.svg?logo=jetpackcompose)](https://developer.android.com/jetpack/compose)
[![Platform](https://img.shields.io/badge/Platform-Android%20|%20iOS-orange.svg)](https://kotlinlang.org/docs/multiplatform.html)

[![KMP](https://img.shields.io/badge/KMP-Kotlin%20Multiplatform-7F52FF.svg)](https://kotlinlang.org/docs/multiplatform.html)
[![Swipe](https://img.shields.io/badge/UI-Swipe%20Gestures-red.svg)](https://github.com/ismoy/kmpswipe)
[![UX](https://img.shields.io/badge/UX-Haptic%20Feedback-blueviolet.svg)](https://github.com/ismoy/kmpswipe)
# Demonstrations
| Android 
|---------|
| ![Scanner](https://github.com/ismoy/DemoBelzSpeedScan/blob/main/demoCamera_compressed.gif)

## Installation

### Kotlin Multiplatform (KMP)

Add the BelZSpeedScan dependency to your `commonMain` source set in your project's `build.gradle` file:

```gradle
repositories {
    mavenCentral()
}

dependencies {
    commonMain {
        implementation("io.github.ismoy:belzspeedscan:1.0.10") // Replace with the actual version
    }
    // ... other dependencies
}
//Your Info.plist add this permission
<key>NSCameraUsageDescription</key>
    <string>Necesitamos acceso a la cámara para escanear códigos QR y códigos de barras</string>
    <key>UIBackgroundModes</key>
    <array>
        <string>audio</string>
    </array>
//go to src/iosMain create a package /resources
//src/iosMain/resources/resourename.mp3
```
## Android Native
For native Android development, include the dependency in your module's build.gradle file:
```gradle
repositories {
    mavenCentral()
}

dependencies {
    implementation("io.github.ismoy:belzspeedscan:1.0.10") // Replace with the actual version
    // ... other dependencies
}
```
## How to Use
The core functionality of BelZSpeedScan is accessed through the App function (or similar entry point in your KMP project).  This function requires a context parameter, which in Android would typically be your MainActivity's context.
# Use in your KMP and Android project
```kotlin
import io.github.ismoy.belzspeedscan.domain.CodeScanner // Import CodeScanner
import androidx.compose.runtime.*
import androidx.compose.ui.platform.LocalLifecycleOwner
// ... other imports
fun App(context: Any? = null) {
CameraScreen(context)
}
// composeApp/androidMain/MainActivity
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            App(this.applicationContext)
        }
    }
}
// create CameraManagerUtil
fun CameraManagerUtils(
    context: Any?,
    onCodeScanned: (String) -> Unit
) {
    val lifecycleOwner = androidx.lifecycle.compose.LocalLifecycleOwner.current
    var hasCameraPermission by remember { mutableStateOf(false) }
    val scanner: CodeScanner? by remember { mutableStateOf(null) }

    DisposableEffect(Unit) {
        onDispose {
            scanner?.stopScanning()
        }
    }
    // Default request camera dialog
    RequestCameraPermission { granted ->
        hasCameraPermission = granted
    }
       OR
// Custom Dialog Permission
 RequestCameraPermission(
        titleDialogConfig = "Your app needs camera permission",
        descriptionDialogConfig = "Your app needs camera permission to scan QR codes",
        btnDialogConfig = "Open Settings",
        titleDialogDenied = "Camera permission denied",
        descriptionDialogDenied = "You need to grant camera permission to scan QR codes",
        btnDialogDenied = "Grant Permission",
        customDeniedDialog = {
            //Your custom dialog
        },
        customSettingsDialog = {
            //Your custom dialog
        }
    ) {granted-> 
        println("CameraManagerUtils: $granted")
    }

    Scaffold(
        content = { innerPadding ->
            if (hasCameraPermission) {
                Box(
                    modifier = Modifier
                        .padding(innerPadding)
                        .fillMaxSize()
                ) {
                    Box {
                        var currentScanner by remember { mutableStateOf<CodeScanner?>(null) }
                        CameraPreview(
                            onPreviewViewReady = { preview ->
                                currentScanner = createBelSpeedScanCodeScanner(
                                    context = context,
                                    lifecycleOwner = lifecycleOwner,
                                    previewView = preview,
                                    playSound = true,
                                    resourceName = "resourename"//your resource name only for emit a scan beep in IOS,
                                    resourceExtension = "mp3"//your extension recommended mp3,
                                    delayToNextScan = 3000,
                                    onCodeScanned = { scannedText ->
                                        onCodeScanned(scannedText)
                                    }
                                ).also {
                                    it.startScanning()
                                }
                            },
                            scanner = currentScanner,
                            modifier = Modifier.fillMaxHeight(1F),
                            tooFarText = "Acerca el código a la cámara\nDistancia demasiado lejana",//Add your custom message o user default message just remove the tooFarText
                            tooOptimalText = "¡Distancia perfecta!\nMantén el código dentro del marco",//Add your custom message o user default message just remove the tooOptimalText
                        )
                    }
                }
            } else {
                Box(
                    modifier = Modifier
                        .fillMaxSize()
                        .fillMaxHeight(1F)
                        .padding(innerPadding)
                        .background(Color.Black.copy(alpha = 0.8f))
                )
            }
        }
    )
}
// create a CameraScreen
fun CameraScreen(context: Any?) {
CameraManagerUtils(context) { codeScanned ->
                if (codeScanned.isNotEmpty()) {
                    if (scannedCodes.contains(codeScanned)) {
                        alertManager.showAlert("Código duplicado: $codeScanned")
                    } else {
                        scannedCodes.add(codeScanned)
                        alertManager.showAlert("Código escaneado: $codeScanned")
                        coroutineScope.launch {
                            bottomSheetState.expand()
                        }
                    }
                }
            }
}
```
## Camera Permissions

BelZSpeedScan handles camera permissions automatically on Android. However, on iOS, you need to add the camera usage description to your `Info.plist` file.

### iOS Permission
If you don't have an `Info.plist` file, you need to create it. Then, go to `composeApp/iosMain/iosApp/iosApp/Info.plist` and add the following:
```xml
<key>NSCameraUsageDescription</key>
<string>We need access to the camera to scan QR codes and barcodes.</string>
```
### Add your Scanner Sound
If you need emit a sound when the code scan., go to `composeApp/iosMain/resources/beep.mp3` remember respect this exactly name resources/beep.mp3

## Explanation:
1. **Imports:** Import necessary classes, including CodeScanner from the library.
2. **Function:** This function serves as the entry point for using the scanner. The context parameter is crucial for Android integration.
3. ## State Variables:
• **scannedCode**: Stores the result of the scan.  
• **hasCameraPermission**: Tracks whether the user has granted camera permission.  
• **scanner**: Holds an instance of the CodeScanner.

4. **RequestCameraPermission:** This function handles requesting camera permissions. The default implementation provides a standard dialog. You can customize the dialogs by using the customDeniedDialog and customSettingsDialog parameters. The onResult lambda provides a boolean indicating whether permission was granted.
5. **Conditional Rendering:** The if (hasCameraPermission) block ensures that the camera preview and scanner are only initialized if permission has been granted.
6. **CameraPreview:** This composable displays the camera preview. The onPreviewViewReady lambda is called when the PreviewView is ready, allowing you to initialize the CodeScanner.
7. **createBelSpeedScanCodeScanner:** This function creates an instance of the CodeScanner. It takes the context, LifecycleOwner, PreviewView, isQRScanning (true for QR code scanning only, false for barcode scanning only. No default value), playSound, and a lambda for handling the scanned code.
8. **startScanning():** Starts the scanning process.
9. **Scanned Code Handling:** The lambda passed to createBelSpeedScanCodeScanner is called when a code is scanned. The scannedCode parameter contains the scanned data.

## Native iOS Support

Good news! Native iOS support is on the way. We're working hard to bring the functionality of BelZSpeedScan to the iOS platform, allowing you to use the same scanning logic in your iOS applications. Stay tuned for future updates and announcements regarding the availability of iOS support.
## This detailed explanation should help you integrate BelZSpeedScan into your KMP and Android projects effectively. Remember to replace placeholder version numbers and adapt the code to your specific UI and application logic.
