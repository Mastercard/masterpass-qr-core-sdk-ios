# Masterpass QR Core SDK

This SDK provides parser for parsing Push Payment QR code.

You can use this SDK to generate Push Payment QR code string also by filling the data and using `PushPaymentData.generatePushPaymentString()`. See documentation for more information.

_This sdk only deals with actual QR code strings. So you need to use a separate QR scanning SDK to get QR code strings. You can use [Masterpass QR Scan SDK][1]_

*This SDK is developed in Objective-C and it works with Swift.*

### Requirements:
1. Xcode 9.0+
2. iOS 8.0+

### Features:
1. Capabilities to parse and validate Push Payment QR code string.
2. Generate Push Payment QR code string.

### Installation

#### Cocoapods
- In your *Podfile* write the following

  ```
  use_frameworks!
  pod 'MasterpassQRCoreSDK'
  ```

- Do `pod install`
- Everything is setup now

#### Manual
##### Swift
- Download the latest release of [Masterpass QR Core SDK][2].
- Unzip the file.
- Go to your Xcode project’s target “General” settings. Drag MasterpassQRCoreSDK.framework to the “Embedded Binaries” section. Make sure to select **Copy items if needed** and click Finish.
- Create a new **Run Script Phase** in your app’s target’s **Build Phases** and paste the following snippet in the script text field:

	`bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/MasterpassQRCoreSDK.framework/strip-frameworks.sh"`

  This step is required to work around an App Store submission bug when archiving universal binaries.


##### Objective-C
- Follow same instructions as Swift

[1]: https://www.github.com/Mastercard/masterpass-qr-scan-sdk-ios
[2]: https://www.github.com/Mastercard/masterpass-qr-core-sdk-ios/releases/download/2.0.2/masterpassqrcoresdk-framework-ios.zip

### Usage

#### Parsing

__Swift__

```swift
import MasterpassQRCoreSDK

func parseQRCode(code: String) {
  do {
      // Parse qr code
      let pushData = try MPQRParser.parse(string:code)
      // Print data in string format
      print(pushData.dumpData())
  } catch let error as MPQRError{
      if let str = error.getString(){
           print("Error: \(str)")
      }else
      {
          print("Unknown error occurred \(error)")
      }
  } catch {
      print("Unknown error occurred \(error)")
  }
}
```

__Objective-C__

```objc
@import MasterpassQRCoreSDK;

- (void)parseQRCode:(NSString *)code {
    PushPaymentData *pushPaymentData;
    NSString* (^handleError)(MPQRError *) = ^NSString* (MPQRError* error) {
        if (error.domain == MPQRErrorDomain) {
            return [NSString stringWithFormat:@"Error: %@", [error getString]];
        }
        return [NSString stringWithFormat:@"Unkown error occured %@", error];
    };

    MPQRError *error;
    pushPaymentData = [MPQRParser parse:code error:&error];

    if (error) {
        NSLog(handleError(error));
    }else
    {
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

        if let output = filter.outputImage?.transformed(by: transform) {
            return UIImage(ciImage: output)
        }
    }
    return nil
}

func generatePushPaymentQR() {
    // Generate
    let pushPaymentData = PushPaymentData()
    // Set required properties on push payment data e.g pushPaymentData.payloadFormatIndicator = "01"
    var ppdString:String?
    do {
        // Validate generated data
        ppdString = try pushPaymentData.generatePushPaymentString()
    } catch {
        print("Error occurred during validation \(error)")
        return
    }

    // Generate image
    let image = generateQRCode(from: ppdString!)
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

    MPQRError *error;
    NSString* strQRCode = [pushPaymentData generatePushPaymentString:&error];

    if (error) {
        NSLog(@"Error: %@", error);
    } else {
        // Generate image
        UIImage *image = [self generateQRCode: strQRCode];
    }
}
```
