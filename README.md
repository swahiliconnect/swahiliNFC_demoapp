# SwahiliNFC Example App Documentation

## Overview

This example application demonstrates the capabilities of the SwahiliNFC Flutter package, which provides a comprehensive solution for NFC business card applications with a focus on secure contact exchange. The app showcases the core features of the SwahiliNFC package with a good UI/UX experience.

![SwahiliNFC Demo App](https://raw.githubusercontent.com/swahiliconnect/swahili_nfc/main/doc/images/swahilinfc_demo.gif)

## Features

- **Multi-type Content Writing**: Write contact information or links to NFC cards
- **Intelligent Reading**: Automatically detect content type and provide relevant actions
- **Dark/Light Mode**: Toggle between theme modes for better visibility
- **UX**: Bottom sheet animations, success/error dialogs, and smooth transitions
- **Interactive Card Display**: Take actions directly from scanned cards (call, email, open links)
- **Comprehensive Status**: NFC availability checking and troubleshooting information

## Setup Requirements

### Prerequisites

- Flutter SDK (2.17.0 or later)
- Android Studio with NDK installed
- Xcode (for iOS development)
- A physical device with NFC capabilities for testing

## Installation

1. Clone the repository
2. Run `flutter pub get` to install dependencies
3. Connect an NFC-enabled device
4. Run `flutter run` to launch the app

## Android Configuration

### AndroidManifest.xml

The app requires several permissions and configurations in the `AndroidManifest.xml`:

```xml
<!-- Required NFC permissions -->
<uses-permission android:name="android.permission.NFC" />

<!-- Indicates NFC capability - set to false to make app available to devices without NFC -->
<uses-feature android:name="android.hardware.nfc" android:required="true" />

<!-- For NFC tag discovery -->
<intent-filter>
    <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <data android:mimeType="application/vnd.swahilicard" />
</intent-filter>

<!-- General tech discovery as fallback -->
<intent-filter>
    <action android:name="android.nfc.action.TECH_DISCOVERED"/>
    <category android:name="android.intent.category.DEFAULT"/>
</intent-filter>

<!-- Tag discovery as final fallback -->
<intent-filter>
    <action android:name="android.nfc.action.TAG_DISCOVERED"/>
    <category android:name="android.intent.category.DEFAULT"/>
</intent-filter>
```

### build.gradle.kts

Make sure to include the following in your app's `build.gradle.kts`:

```kotlin
android {
    namespace = "com.example.swahili_nfc_demo"
    compileSdk = flutter.compileSdkVersion
    ndkVersion = "27.0.12077973"  // Required for NFC capabilities

    defaultConfig {
        // Ensure minSdk is at least 16 for NFC support
        minSdk = if (flutter.minSdkVersion < 16) 16 else flutter.minSdkVersion
    }

    // Add this to ensure the plugin's package name is properly included
    sourceSets {
        getByName("main") {
            java.srcDirs("src/main/kotlin", "src/main/java")
        }
    }
}

dependencies {
    // Add this line to explicitly include the plugin
    implementation(project(":swahili_nfc"))
}
```

### NDK Installation

1. Open Android Studio
2. Go to Tools > SDK Manager
3. Select the "SDK Tools" tab
4. Check "NDK (Side by side)" and click "Apply"
5. Accept the license agreement and complete the installation

## iOS Configuration

### Info.plist

Add the following to your `Info.plist`:

```xml
<key>NFCReaderUsageDescription</key>
<string>This app needs NFC to read and write business cards</string>

<key>com.apple.developer.nfc.readersession.formats</key>
<array>
    <string>NDEF</string>
    <string>TAG</string>
</array>
```

### Entitlements

You'll need to create NFC entitlements for your iOS app:

1. In Xcode, select your Runner target
2. Go to the "Signing & Capabilities" tab
3. Click "+ Capability" and select "Near Field Communication Tag Reading"
4. Ensure your Apple Developer account has NFC capabilities enabled

## Usage Examples

### Initializing SwahiliNFC

```dart
void main() {
  // Enable NFC debugging for development
  SwahiliNFC.enableDebugLogging(true);
  runApp(const SwahiliNFCApp());
}
```

### Checking NFC Availability

```dart
Future<void> _checkNfcAvailability() async {
  try {
    final isAvailable = await SwahiliNFC.isAvailable();
    setState(() {
      _isNfcAvailable = isAvailable;
      _statusMessage = isAvailable
          ? 'NFC is available. Ready to use.'
          : 'NFC is not available on this device.';
    });
  } catch (e) {
    // Handle error
  }
}
```

### Reading an NFC Card

```dart
Future<void> _readTag() async {
  try {
    // Raw data for debugging (optional)
    String rawData = await SwahiliNFC.dumpTagRawData();
    
    // Read the card data
    final cardData = await SwahiliNFC.readTag();
    
    // cardData contains properties like name, company, email, phone, etc.
    setState(() {
      _lastScannedCard = cardData;
    });
  } catch (e) {
    // Handle error
  }
}
```

### Writing Contact Information

```dart
Future<void> _writeContact() async {
  try {
    // Create business card data with contact information
    final cardData = BusinessCardData(
      name: _nameController.text.trim(),
      company: _companyController.text.trim(),
      position: _positionController.text.trim(),
      email: _emailController.text.trim(),
      phone: _phoneController.text.trim(),
      custom: {'type': 'contact'},
    );
    
    // Write to tag with verification
    final success = await SwahiliNFC.writeTag(
      data: cardData,
      verifyAfterWrite: true,
    );
  } catch (e) {
    // Handle error
  }
}
```

### Writing a Link

```dart
Future<void> _writeLink() async {
  try {
    // Create business card data with link information
    final cardData = BusinessCardData(
      name: _linkTitleController.text.trim(),
      custom: {
        'type': 'link', 
        'url': _linkUrlController.text.trim()
      },
    );
    
    // Write to tag with verification
    final success = await SwahiliNFC.writeTag(
      data: cardData,
      verifyAfterWrite: true,
    );
  } catch (e) {
    // Handle error
  }
}
```

## App Structure

The example app is structured as follows:

- `main.dart` - Contains the entire application in a single file
- Core Components:
  - `SwahiliNFCApp` - Main application with theme configuration
  - `NFCHomePage` - Main screen with tab controller
  - `NFCOperationSheet` - Bottom sheet for NFC operations
  - Custom UI components for different card types

## Tabs Explained

### Write Tab
This tab allows users to write two types of content to NFC cards:
- **Contact Information**: Name, company, position, email, and phone
- **Link**: Title and URL

### Read Tab
This tab reads data from NFC cards and displays:
- For contact cards: Personal info, contact details, and action buttons (call, email, save)
- For link cards: Title, URL, and an open button

### Status Tab
This tab displays:
- Current NFC status (available or not)
- Troubleshooting tips
- Information about SwahiliNFC features

## UI/UX Features

- **Theme Toggle**: Switch between light and dark mode
- **Bottom Sheet Animation**: Animated wave pattern during NFC operations
- **Card Design**: Elegant card-based UI with actionable components
- **Feedback System**: Dialogs for success and detailed error information
- **Troubleshooting Tips**: Context-aware help based on error type

## Troubleshooting

### Common Issues

1. **NFC Not Detected**
   - Ensure NFC is enabled in your device settings
   - Try moving the card around the back of your phone (NFC antenna locations vary)
   - Restart the app or your device

2. **Write Operation Fails**
   - Ensure the NFC card is writable (not locked)
   - Hold the card steady during the entire operation
   - Try with less data initially

3. **Android Build Errors**
   - Verify NDK is installed and properly configured
   - Check that minSdk is at least 16
   - Ensure all necessary permissions are in the AndroidManifest.xml

4. **iOS Build Errors**
   - Verify your provisioning profile supports NFC capabilities
   - Check that entitlements are properly configured
   - Ensure Info.plist has the required NFC entries

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- SwahiliCard team for developing the SwahiliNFC package
- Contributors to the example app

---

For more information, visit the [SwahiliNFC package](https://pub.dev/packages/swahili_nfc) on pub.dev.