# Masterpass QR SDK

This SDK provides parser for parsing Push Payment QR code.

You can use this SDK to generate Push Payment QR code string also by filling the data and using `PushPaymentData.generatePushPaymentString()`. See documentation for more information.

_This sdk only deals with actual QR code strings. So you need to use a separate QR scanning SDK to get QR code strings. You can use [Masterpass QR Scan SDK][1]_

*This SDK is developed in Swift and it works with Objective-C but it is recommended to use Swift for development with this SDK.*

### Requirements:
1. Xcode 8.2
2. iOS >= 9.0

### Features:
1. Capabilities to parse and validate Push Payment QR code string.
2. Generate Push Payment QR code string.

### Installation

#### Cocoapods
- In your *Podfile* write the following

  ```
  pod 'MasterpassQRCoreSDK'
  ```

- Do `pod install`
- Everything is setup now

#### Manual
##### Swift
- Download the latest release of [Masterpass QR SDK][2].
- Unzip the file.
- Go to your Xcode project’s “General” settings. Drag MasterpassQRCoreSDK.framework to the “Embedded Binaries” section. Make sure to select **Copy items if needed** and click Finish.
- Create a new **Run Script Phase** in your app’s target’s **Build Phases** and paste the following snippet in the script text field:

	`bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/MasterpassQRCoreSDK.framework/strip-frameworks.sh"`

  This step is required to work around an App Store submission bug when archiving universal binaries.


##### Objective-C
- Follow same instructions as Swift
- Go to your Xcode project's **Build Settings** and set **Always Embed Swift Standard Libraries** to **YES**

[1]: https://www.github.com/Mastercard/masterpass-qr-scan-sdk-ios
[2]: https://github.com/Mastercard/masterpass-qr-core-sdk-ios/releases/download/1.0.0/masterpassqrcoresdk-framework-ios.zip

### Usage

#### Parsing

__Swift__

```swift
import MasterpassQRCoreSDK

func parseQRCode(code: String) {
  do {
      // Parse qr code
      var pushData = try MPQRParser.parseWithoutTagValidation(string: code)
      // Validate parsed data
      try pushData!.validate()
      // Print data in string format
      print(pushData.dumpData())
  } catch MPQRError.invalidFormat(let message) {
      print("Invalid exception: \(message)")
  } catch MPQRError.invalidTagValue(let tag, let value) {
      print("Invalid tag \(tag) with value \(value)")
  } catch MPQRError.missingTag(let message, let tags) {
      print("Missing tags \(tags) with message: \(message ?? "")")
  } catch MPQRError.unknownTag(let tagString) {
      print("Unknown tag \(tagString)")
  } catch MPQRError.conflictingTag(let message, let tags) {
      print("Conflicting tags \(tags) with message: \(message ?? "")")
  } catch {
      print("Unknown error occurred \(error)")
  }
}
```

__Objective-C__

```objc
@import MasterpassQRCoreSDK;

- (void)parseQRCode:(NSString *)code {
  NSString* (^handleError)(NSError *) = ^NSString* (NSError* error) {
      if (error.domain == MPQRErrorDomain) {
          if (error.code == InvalidFormat) {
              return [NSString stringWithFormat:@"Invalid format: %@", error.userInfo[MPQRErrorMessageKey]];
          } else if (error.code == InvalidTagValue) {
              return [NSString stringWithFormat:@"Invalid tag %@ with value %@", error.userInfo[MPQRErrorTagInfoKey], error.userInfo[MPQRErrorTagValueKey]];
          } else if (error.code == MissingTag) {
              return [NSString stringWithFormat:@"Missing tags %@ with message %@", error.userInfo[MPQRErrorTagsKey], error.userInfo[MPQRErrorMessageKey]];
          } else if (error.code == UnknownTag) {
              return [NSString stringWithFormat:@"Unknown tag %@", error.userInfo[MPQRErrorTagValueKey]];
          } else if (error.code == ConflictingTag) {
              return [NSString stringWithFormat:@"Conflicting tags %@ with message %@", error.userInfo[MPQRErrorTagsKey], error.userInfo[MPQRErrorMessageKey]];
          }
      }

      return [NSString stringWithFormat:@"Unknown error occurred %@", error];
  };

  NSError *error;
  // Parse qr code
  PushPaymentData *pushPaymentData = [MPQRParser parseWithoutTagValidationWithString:code error:&error];
  if (!error) {
      // Validate parsed data
      [pushPaymentData validateAndReturnError:&error];
  }

  if (error) {
    NSLog(handleError(error));
  } else {
    // Print data in string format
    NSLog([pushPaymentData dumpData]);
  }
}
```

#### Generating

__Swift__

```swift
import CoreImage

func generateQRCode(from string: String) -> UIImage? {
    let data = string.data(using: String.Encoding.utf8)

    if let filter = CIFilter(name: "CIQRCodeGenerator") {
        filter.setValue(data, forKey: "inputMessage")
        // Set scale according to your device display. If the qr code is blurry then increase scale
        let transform = CGAffineTransform(scaleX: 3, y: 3)

        if let output = filter.outputImage?.applying(transform) {
            return UIImage(ciImage: output)
        }
    }

    return nil
}

func generatePushPaymentQR() {
  // Generate
  let pushPaymentData = PushPaymentData()
  // Set required properties on push payment data e.g pushPaymentData.payloadFormatIndicator = "01"
  do {
      // Validate generated data
      try pushPaymentData.validate()
  } catch {
      print("Error occurred during validation \(error)")
      return
  }

  // Generate image
  let image = generateQRCode(from: pushPaymentData.generatePushPaymentString())
}
```

__Objective-C__

```objc
@import CoreImage;

- (UIImage *)generateQRCode:(NSString *)string {
    NSData *data = [string dataUsingEncoding:NSUTF8StringEncoding];
    CIFilter *filter = [CIFilter filterWithName:@"CIQRCodeGenerator"];
    if (!filter || !data)
        return nil;

    [filter setValue:data forKey:@"inputMessage"];
    CGAffineTransform transform = CGAffineTransformMakeScale(3, 3);
    CIImage *image = [[filter outputImage] imageByApplyingTransform:transform];
    return [UIImage imageWithCIImage:image];
}

- (void)generatePushPaymentQR {
  // Generate
  PushPaymentData *pushPaymentData = [[PushPaymentData alloc] init];
  // Set required properties on push payment data e.g pushPaymentData.payloadFormatIndicator = @"01"

  NSError *error;
  // Validate parsed data
  [pushPaymentData validateAndReturnError:&error];

  if (error) {
    NSLog(@"Error occurred during validation %@", error);
  } else {
    // Generate image
    UIImage *image = [self generateQRCode: [pushPaymentData generatePushPaymentString]];
  }
}
```
