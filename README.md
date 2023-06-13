# FlutterTrafficManualIntercept
We generally use the reFlutter framework on the apk or on IPA to intercept the traffic in our proxy application. Some times we cant use the reFlutter framework as it throws error saying this engine is currently not supported.
Then devs need to manually add the burp certificate and include proxy configuration manually in the flutter app code.
I have created a step by step guide on a basic level on how to add proxy manually in flutter


Step 1: Obtain Burp Suite and Flutter Development Environment
- Download and install Burp Suite from the PortSwigger website.
- Set up the Flutter development environment on your machine by following the official Flutter documentation.

Step 2: Retrieve Burp Suite CA Certificate
- Open Burp Suite and go to the "Proxy" tab.
- Under the "Intercept" sub-tab, click on "Options".
- In the "Proxy Options" section, click on the "Import / Export CA Certificate" button.
- Choose to export the certificate in DER format and save it to a location on your computer.

Step 3: Create a Flutter Project
- Create a new Flutter project or open an existing one using your preferred code editor.

Step 4: Import Burp Suite CA Certificate
- Copy the exported Burp Suite CA certificate (in DER format) to your Flutter app project directory.
- Create a new folder in the root of your Flutter project called `assets` (if it doesn't already exist).
- Move the Burp Suite CA certificate file into the `assets` folder.
- Open the `pubspec.yaml` file in your Flutter app's root directory.
- Add the following lines under the `flutter` section:
  ```yaml
  assets:
    - assets/
  ```
- Save the `pubspec.yaml` file.

Step 5: Modify Flutter App Code
- Locate the file in your Flutter project responsible for handling network requests (e.g., `api_service.dart`, `http_client.dart`).
- Import the necessary dependencies at the top of the file:
  ```dart
  import 'dart:io';
  import 'package:flutter/services.dart' show rootBundle;
  ```
- Add the following code to load the Burp Suite CA certificate:
  ```dart
  static Future<void> loadCertificate() async {
    SecurityContext clientContext = SecurityContext();
    ByteData certificateData = await rootBundle.load('assets/burp_certificate.cer');
    List<int> certBytes = certificateData.buffer.asUint8List();
    clientContext.setTrustedCertificatesBytes(certBytes);
    HttpClient.context = clientContext;
  }
  ```
- Locate the method responsible for creating the `HttpClient` object.
- Modify the method as follows to include the proxy configuration:
  ```dart
  static HttpClient getHttpClient() {
    HttpClient client = HttpClient()
      ..findProxy = (uri) {
        return "PROXY 127.0.0.1:8080;"; // Replace with the IP address and port of your Burp Suite proxy
      }
      ..badCertificateCallback = ((X509Certificate cert, String host, int port) => true);
  
    return client;
  }
  ```
- In the `main()` function or the entry point of your Flutter app, call the `loadCertificate()` method before making any network requests:
  ```dart
  void main() async {
    WidgetsFlutterBinding.ensureInitialized();

    await loadCertificate();

    // Perform other app initialization tasks

    HttpClient client = getHttpClient();

    // Make network requests using the client

    // Run the Flutter app
  }
  ```

Step 6: Replace Proxy Details
- Replace `"PROXY 127.0.0.1:8080;"` in the `getHttpClient()` method with the actual IP address and port of your Burp Suite proxy.

Step 7: Run the Flutter App
- Ensure that Burp Suite is running and the proxy listener is configured to listen on the specified IP address and port.
- Run your Flutter app either through the command line (`flutter run`) or using your preferred IDE.
With these modifications, your Flutter app should route its network traffic through Burp Suite, allowing you to intercept and analyze the requests and responses in Burp Suite while using the app.

Please note that these modifications should be used for development and testing purposes only and should not be deployed in a production environment.

**Note**:
Sometime above will not work. For that we need to use this below:

We need to use both HttpClient _client, IOClient _ioClient. Also we need to make API call to _ioClient
```
Future<SecurityContext> get globalContext async {
    final sslCert1 = await
    rootBundle.load('resources/certificates/vapt.pem');
    SecurityContext sc = new SecurityContext(withTrustedRoots: true);
    sc.setTrustedCertificatesBytes(sslCert1.buffer.asInt8List());
    return sc;
  }
  Future<void> setSSLPinning(String uri) async {
           _client = HttpClient(context: await globalContext);
            this.proxy = await _secureStorage.getLocalIp();
       


            _client.findProxy = (uri) => "PROXY $proxy;";

          _client.badCertificateCallback = (X509Certificate cert, String host, int port) => true;
        _ioClient = new IOClient(_client);
    }
```
If you are using **DIO** then you can also use this:
![WhatsApp Image 2023-06-07 at 3 44 37 PM (1)](https://github.com/SecurityBong/FlutterTrafficManualIntercept/assets/52169190/5da47f9d-2204-4b06-a33c-cf0d2d8e91a9)
![WhatsApp Image 2023-06-07 at 3 44 37 PM](https://github.com/SecurityBong/FlutterTrafficManualIntercept/assets/52169190/4ba277f9-ceb9-4789-84a8-2c945f2415be)


