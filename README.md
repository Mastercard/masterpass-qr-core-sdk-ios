# MP QR Core SDK

This SDK provides parser for parsing Push Payment QR code.

You can use this SDK to generate Push Payment QR code string also by filling the data and using `PushPaymentData.generatePushPaymentString()`. See documentation for more information.

_This sdk only deals with actual QR code strings. So you need to use a separate QR scanning SDK to get QR code strings. You can use [MP QR Scan SDK][1]_

*This SDK is developed in Objective-C and it works with Swift.*

### Requirements:
1. Xcode 9.0+
2. iOS 8.0+

### Features:
1. Capabilities to parse and validate Push Payment QR code string.
2. Generate Push Payment QR code string.

### Documentation
The code documentation can be found on Github link [here][3].

### Installation

#### Cocoapods
- In your *Podfile* write the following

  ```
  use_frameworks!
  pod 'MPQRCoreSDK'
  ```

- Do `pod install`
- Everything is setup now

#### Manual
##### Swift
- Download the latest release of [MP QR Core SDK][2].
- Unzip the file.
- Go to your Xcode project’s target “General” settings. Drag MPQRCoreSDK.framework to the “Embedded Binaries” section. Make sure to select **Copy items if needed** and click Finish.
- Create a new **Run Script Phase** in your app’s target’s **Build Phases** and paste the following snippet in the script text field:

	`bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/MPQRCoreSDK.framework/strip-frameworks.sh"`

  This step is required to work around an App Store submission bug when archiving universal binaries.


##### Objective-C
- Follow same instructions as Swift

[1]: https://www.github.com/Mastercard/s-qr-scan-sdk-ios
[2]: https://www.github.com/Mastercard/masterpass-qr-core-sdk-ios/releases/download/2.0.7/MPQRCoreCdk-framework-ios.zip
[3]: https://mastercard.github.io/masterpass-qr-core-sdk-ios

### Usage

#### Parsing

__Swift__

```swift
import MPQRCoreSDK

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
@import MPQRCoreSDK;

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

    // Payload format indicator
    pushPaymentData.payloadFormatIndicator = "01"

    // Point of initiation method
    pushPaymentData.pointOfInitiationMethod = "12"

    // Merchant identifier Visa with tag `02`
    pushPaymentData.merchantIdentifierVisa02 = "4600678934521435"

    // Merchant identifier Visa with tag `03`
    pushPaymentData.merchantIdentifierVisa03 = "4600678934521467"

    // Merchant identifier MasterCard with tag `04`
    pushPaymentData.merchantIdentifierMastercard04 = "555544443333111"

    // Merchant identifier MasterCard with tag `05`
    pushPaymentData.merchantIdentifierMastercard05 = "555544443333222"

    // Merchant identifier NPCI with tag `06`
    pushPaymentData.merchantIdentifierNPCI06 = "5600678934521435"

    // Merchant identifier NPCI with tag `07`
    pushPaymentData.merchantIdentifierNPCI07 = "5600678934521459"

    //Merchant identifier (EMVCo)IFSCCode
    pushPaymentData.merchantIdentifierIFSCCODE08 = "6800678934521565"

    //Merchant identifier (Discover)
    pushPaymentData.merchantIdentifierDISCOVER09 = "7800678934521675"

    //Merchant identifier (Discover)
    pushPaymentData.merchantIdentifierDISCOVER10 = "7900678934521655"

    //Merchant identifier (Amex)
    pushPaymentData.merchantIdentifierAMEX11 = "3200678934521434"

    //Merchant identifier (Amex)
    pushPaymentData.merchantIdentifierAMEX12 = "3400678934521469"

    //Merchant identifier (JCB)
    pushPaymentData.merchantIdentifierJCB13 = "5700678934521457"

    //Merchant identifier (JCB)
    pushPaymentData.merchantIdentifierJCB14 = "5800678934521435"

    //Merchant identifier (UnionPay)
    pushPaymentData.merchantIdentifierUNIONPAY15 = "5800678934524535"

    //Merchant identifier (UnionPay)
    pushPaymentData.merchantIdentifierUNIONPAY16 = "4850678934521598"

    //17 - 25 for merchant identifier EMVCO
    try! pushPaymentData.setMerchantIdentifierEMVCODataForTagString("17", data: "4657678934521449")


    //Merchant Identifier data 26-51
    let maiData = MAIData()
    var rootTag = "26"
    maiData.setRootTag(rootTag)
    maiData.AID = "AID0349509H"
    try! maiData.setPaymentNetworkSpecific("PNS93484jf", forTag:"01")
    try! maiData.setDynamicPaymentNetworkSpecific("PNSDyn8494738")
    var str01 = try!maiData.getPaymentNetworkSpecific(forTag: "01")
    var arr = maiData.getAllDynamicPaymentNetworkSpecificTags()!
    print("testSamplePushPaymentData = \(str01) \(arr)")
    try! pushPaymentData.setMAIDataForTagString(rootTag, data: maiData)

    // Merchant category code
    pushPaymentData.merchantCategoryCode = "1434"

    // Transaction currency code
    pushPaymentData.transactionCurrencyCode = "156"

    // Transaction amount
    pushPaymentData.transactionAmount = "83.80"

    let tipType = TipConvenienceIndicator.percentageConvenienceFee
    switch tipType {
    case .promptedToEnterTip:
        // Tip or convenience indicator
        pushPaymentData.tipOrConvenienceIndicator = "01"
    case .flatConvenienceFee:
        // Tip or convenience indicator
        pushPaymentData.tipOrConvenienceIndicator = "02"
        // Value of convenience fee fixed
        pushPaymentData.valueOfConvenienceFeeFixed = "10"
    case .percentageConvenienceFee:
        // Tip or convenience indicator
        pushPaymentData.tipOrConvenienceIndicator = "03"
        // Value of convenience fee percentage
        pushPaymentData.valueOfConvenienceFeePercentage = "5"
    case .unknownTipConvenienceIndicator:
        print("do nothing")
    }

    // Country code
    pushPaymentData.countryCode = "CN"

    // Merchant name
    pushPaymentData.merchantName = "BEST TRANSPORT"

    // Merchant city
    pushPaymentData.merchantCity = "BEIJING"

    // Postal code
    pushPaymentData.postalCode = "56748"


    // Additional data
    let addData = AdditionalData()
    addData.storeId = "A6008"
    addData.loyaltyNumber = "***"
    addData.terminalId = "A6008667"
    addData.additionalConsumerDataRequest = "ME"

    var rootSubTag = "50"
    var additionalUnrestrictedData = UnrestrictedData()
    additionalUnrestrictedData.setRootTag(rootSubTag)
    additionalUnrestrictedData.AID = "GUI123"
    try! additionalUnrestrictedData.setContextSpecific("CONT", forTag: "01")
    try! additionalUnrestrictedData.setDynamicContextSpecific("DYN6")
    str01 = try!additionalUnrestrictedData.getContextSpecificData(forTag: "01")
    arr = additionalUnrestrictedData.getAllDynamicContextSpecificDataTags()!
    try! addData.setUnreserved(additionalUnrestrictedData, forTag: rootSubTag)

    rootSubTag = "51"
    additionalUnrestrictedData = UnrestrictedData()
    additionalUnrestrictedData.AID = "GUI2"
    try! additionalUnrestrictedData.setContextSpecific("CON", forTag: "01")
    try! additionalUnrestrictedData.setDynamicContextSpecific("DYN22")
    str01 = try!additionalUnrestrictedData.getContextSpecificData(forTag: "01")
    arr = additionalUnrestrictedData.getAllDynamicContextSpecificDataTags()!
    try! addData.setDynamicTag(additionalUnrestrictedData)

    pushPaymentData.additionalData = addData

    // CRC
    pushPaymentData.crc = "6403"

    // Language Data
    let langData = LanguageData()
    langData.languagePreference = "ZH"
    langData.alternateMerchantCity = "北京"
    langData.alternateMerchantName = "最佳运输"
    pushPaymentData.languageData = langData

    //unrestricted data
    rootTag = "88"
    let unrestrictedData = UnrestrictedData()
    unrestrictedData.setRootTag(rootTag)
    unrestrictedData.AID = "GUI12319494"
    try! unrestrictedData.setContextSpecific("CONT7586F", forTag: "01")
    try! unrestrictedData.setDynamicContextSpecific("DYN647382")
    str01 = try!unrestrictedData.getContextSpecificData(forTag: "01")
    arr = unrestrictedData.getAllDynamicContextSpecificDataTags()!
    try! pushPaymentData.setUnreservedDataForTagString(rootTag, data: unrestrictedData)

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

    // Payload format indicator
    pushPaymentData.payloadFormatIndicator = @"01";

    // Point of initiation method
    pushPaymentData.pointOfInitiationMethod = @"12";

    // Merchant identifier Visa with tag `02`
    pushPaymentData.merchantIdentifierVisa02 = @"4600678934521435";

    // Merchant identifier Visa with tag `03`
    pushPaymentData.merchantIdentifierVisa03 = @"4600678934521467";

    // Merchant identifier MasterCard with tag `04`
    pushPaymentData.merchantIdentifierMastercard04 = @"555544443333111";

    // Merchant identifier MasterCard with tag `05`
    pushPaymentData.merchantIdentifierMastercard05 = @"555544443333222";

    // Merchant identifier NPCI with tag `06`
    pushPaymentData.merchantIdentifierNPCI06 = @"5600678934521435";

    // Merchant identifier NPCI with tag `07`
    pushPaymentData.merchantIdentifierNPCI07 = @"5600678934521459";

    //Merchant identifier (EMVCo)IFSCCode
    pushPaymentData.merchantIdentifierIFSCCODE08 = @"6800678934521565";

    //Merchant identifier (Discover)
    pushPaymentData.merchantIdentifierDISCOVER09 = @"7800678934521675";

    //Merchant identifier (Discover)
    pushPaymentData.merchantIdentifierDISCOVER10 = @"7900678934521655";

    //Merchant identifier (Amex)
    pushPaymentData.merchantIdentifierAMEX11 = @"3200678934521434";

    //Merchant identifier (Amex)
    pushPaymentData.merchantIdentifierAMEX12 = @"3400678934521469";

    //Merchant identifier (JCB)
    pushPaymentData.merchantIdentifierJCB13 = @"5700678934521457";

    //Merchant identifier (JCB)
    pushPaymentData.merchantIdentifierJCB14 = @"5800678934521435";

    //Merchant identifier (UnionPay)
    pushPaymentData.merchantIdentifierUNIONPAY15 = @"5800678934524535";

    //Merchant identifier (UnionPay)
    pushPaymentData.merchantIdentifierUNIONPAY16 = @"4850678934521598";

    //17 - 25 for merchant identifier EMVCO
    MPQRError* error;
    [pushPaymentData setMerchantIdentifierEMVCODataForTagString:@"17" data:@"4657678934521449" error:&error];

    //Merchant Identifier data 26-51
    MAIData* maiData = [MAIData new];
    NSString* rootTag = @"26";
    [maiData setRootTag:rootTag];
    maiData.AID = @"AID0349509H";
    [maiData setPaymentNetworkSpecific:@"PNS93484jf" forTag:@"01" error:&error];
    [maiData setDynamicPaymentNetworkSpecific:@"PNSDyn8494738" error:&error];
    NSString* strValue01 = [maiData getPaymentNetworkSpecificForTag:@"01" error:&error];
    NSArray* arrTags = [maiData getAllDynamicPaymentNetworkSpecificTags];
    [pushPaymentData setMAIDataForTagString:rootTag data:maiData error:&error];

    // Merchant category code
    pushPaymentData.merchantCategoryCode = @"1434";

    // Transaction currency code
    pushPaymentData.transactionCurrencyCode = @"156";

    // Transaction amount
    pushPaymentData.transactionAmount = @"83.80";

    TipConvenienceIndicator tipType = percentageConvenienceFee;
    switch (tipType) {
        case promptedToEnterTip:
            // Tip or convenience indicator
            pushPaymentData.tipOrConvenienceIndicator = @"01";
            break;
        case flatConvenienceFee:
            // Tip or convenience indicator
            pushPaymentData.tipOrConvenienceIndicator = @"02";
            // Value of convenience fee fixed
            pushPaymentData.valueOfConvenienceFeeFixed= @"10";
            break;
        case percentageConvenienceFee:
            // Tip or convenience indicator
            pushPaymentData.tipOrConvenienceIndicator = @"03";
            // Value of convenience fee percentage
            pushPaymentData.valueOfConvenienceFeePercentage = @"5";
        default:
            break;
    }

    // Country code
    pushPaymentData.countryCode = @"CN";

    // Merchant name
    pushPaymentData.merchantName = @"BEST TRANSPORT";

    // Merchant city
    pushPaymentData.merchantCity = @"BEIJING";

    // Postal code
    pushPaymentData.postalCode = @"56748";

    // Additional data
    AdditionalData* addData = [AdditionalData new];
    addData.storeId = @"A6008";
    addData.loyaltyNumber = @"***";
    addData.terminalId = @"A6008667";
    addData.additionalConsumerDataRequest = @"ME";

    NSString* rootSubTag = @"50";
    UnrestrictedData* additionalUnrestrictedData = [UnrestrictedData new];
    [additionalUnrestrictedData setRootTag:rootSubTag];
    additionalUnrestrictedData.AID = @"GUI123";
    [additionalUnrestrictedData setContextSpecificData:@"CONT" forTag:@"01" error:&error];
    [additionalUnrestrictedData setDynamicContextSpecificData:@"DYN6" error:&error];
    strValue01 = [additionalUnrestrictedData getContextSpecificDataForTag:@"01" error:&error];
    arrTags = [additionalUnrestrictedData getAllDynamicContextSpecificDataTags];
    [addData setUnreservedData:additionalUnrestrictedData forTag:rootSubTag error:&error];

    rootSubTag = @"51";
    additionalUnrestrictedData = [UnrestrictedData new];
    additionalUnrestrictedData.AID = @"GUI2";
    [additionalUnrestrictedData setContextSpecificData:@"CON" forTag:@"01" error:&error];
    [additionalUnrestrictedData setDynamicContextSpecificData:@"DYN22" error:&error];
    strValue01 = [additionalUnrestrictedData getContextSpecificDataForTag:@"01" error:&error];
    arrTags = [additionalUnrestrictedData getAllDynamicContextSpecificDataTags];
    [addData setDynamicTag:additionalUnrestrictedData error:&error];

    pushPaymentData.additionalData = addData;

    // CRC
    pushPaymentData.crc = @"6403";

    // Language Data
    LanguageData* langData = [LanguageData new];
    langData.languagePreference = @"ZH";
    langData.alternateMerchantCity = @"北京";
    langData.alternateMerchantName = @"最佳运输";
    pushPaymentData.languageData = langData;

    //unrestricted data
    rootTag = @"88";
    UnrestrictedData* unrestrictedData = [UnrestrictedData new];
    [unrestrictedData setRootTag:rootTag];
    unrestrictedData.AID = @"GUI12319494";
    [unrestrictedData setContextSpecificData:@"CONT7586F" forTag:@"01" error:&error];
    [unrestrictedData setDynamicContextSpecificData:@"DYN647382" error:&error];
    strValue01 = [unrestrictedData getContextSpecificDataForTag:@"01" error:&error];
    arrTags = [unrestrictedData getAllDynamicContextSpecificDataTags];
    [pushPaymentData setTagInfoValue:unrestrictedData forTagString:rootTag error:&error];

    NSString* strQRCode = [pushPaymentData generatePushPaymentString:&error];

    if (error) {
        NSLog(@"Error: %@", error);
    } else {
        // Generate image
        UIImage *image = [self generateQRCode: strQRCode];
    }
}
```
