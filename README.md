# BelZSpeedScan: A Cross-Platform QR Code and Barcode Scanning Library

BelZSpeedScan is a lightweight and easy-to-use library for scanning QR codes and barcodes. It supports both Kotlin Multiplatform (KMP) and native Android development, providing a consistent API across platforms.  This allows you to use the same scanning logic in your shared KMP code and seamlessly integrate it into your Android application.

## Installation

### Kotlin Multiplatform (KMP)

Add the BelZSpeedScan dependency to your `commonMain` source set in your project's `build.gradle` file:

```gradle
dependencies {
    commonMain {
        implementation("io.github.ismoy.BelZSpeedScan:library:1.0.0") // Replace with the actual version
    }
    // ... other dependencies
}
```
## Android Native
For native Android development, include the dependency in your module's build.gradle file:
```gradle
dependencies {
    implementation("io.github.ismoy.BelZSpeedScan:library:1.0.0") // Replace with the actual version
    // ... other dependencies
}
```
## How to Use
The core functionality of BelZSpeedScan is accessed through the App function (or similar entry point in your KMP project).  This function requires a context parameter, which in Android would typically be your MainActivity's context.
```kotlin
import io.github.ismoy.belzspeedscan.domain.CodeScanner // Import CodeScanner
import androidx.compose.runtime.*
import androidx.compose.ui.platform.LocalLifecycleOwner
// ... other imports

fun App(context: Any? = null) {
    val lifecycleOwner = LocalLifecycleOwner.current // Get the LifecycleOwner
    var scannedCode by remember { mutableStateOf("") } // Store the scanned code
    var hasCameraPermission by remember { mutableStateOf(false) } // Track camera permission
    val scanner: CodeScanner? by remember { mutableStateOf(null) } // Store the CodeScanner instance

    // Request Camera Permission (Default Dialog)
    RequestCameraPermission { granted ->
        hasCameraPermission = granted
    }
      ###OR use your custom Dialog
    // Request Camera Permission (Custom Dialog)
    RequestCameraPermission(
        customDeniedDialog = { onRetry ->
            // Custom design for the denied permission dialog.
        },
        customSettingsDialog = { onOpenSettings ->
            // Custom design for the settings dialog.
        },
        onResult = { granted ->
            hasCameraPermission = granted
        }
    )
    

    if (hasCameraPermission) {
        var currentScanner by remember { mutableStateOf<CodeScanner?>(null) }
        CameraPreview(
            onPreviewViewReady = { preview ->
                currentScanner = createBelSpeedScanCodeScanner(
                    context = context, // Pass the context
                    lifecycleOwner = lifecycleOwner, // Pass the LifecycleOwner
                    previewView = preview, // Pass the PreviewView
                    isQRScanning = false, // Set to true for QR only, false for Barcode only. No default value.
                    playSound = true // Optional sound feedback
                ) { scannedCode ->
                    println("App: Código escaneado: $scannedCode")
                }.also {
                    it.startScanning() // Start scanning
                }
            },
            scanner = currentScanner,
            modifier = Modifier.fillMaxSize() // Optional
        )
    } else {
        // UI shown when permission is not granted 
           Box(
              modifier = Modifier
                    .fillMaxSize()
                    .background(color = Color.Black.copy(alpha = 0.5f)),
                    contentAlignment = Alignment.Center
            ) {}
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


