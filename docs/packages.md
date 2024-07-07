# 可用的 Flutter 套件

在進入實作 ASR、NLU、NLG、TTS 的套件之前，我們先來看看 Flutter 有哪些套件可以用來實作這些功能。基本上，我們使用 speech、TTS、等關鍵字，就可以在 [Dart Pub](https://pub.dev/) 上找到許多套件。

## ASR

- [speech_to_text](https://pub.dev/packages/speech_to_text)：支援 iOS、Android 以及 Web 的語音辨識套件。這個套件主要用平台原生的語音辨識功能。在使用這個方案時，開發者不用支付額外的服務費用，這也是目前在 Dart Pub 上評分最高、最流行的語音辨識套件。
- [google_speech](https://pub.dev/packages/google_speech)：使用 Google 雲端服務的語音辨識套件。
- [azure_speech_recognition_null_safety](https://pub.dev/packages/azure_speech_recognition_null_safety)：使用 Azure 雲端服務的語音辨識套件。
- [ifly_speech_recognition](https://pub.dev/packages/ifly_speech_recognition)：科大訊飛是一家對岸的語音服務公司，也提供了連接他們的服務的 Flutter 套件。

在使用這些套件時，往往還需要一些額外的權限與隱私設定。像是 iOS 上可能需要在 Info.plist 中設定 `NSMicrophoneUsageDescription` 與 `NSSpeechRecognitionUsageDescription`，告知用戶為什麼要使用麥克風與語音辨識權限。而在 Android 上，可能需要在 AndroidManifest.xml 中設定 `RECORD_AUDIO` 權限，而如果我們想要用藍芽裝置錄音，那可能還要額外設定 `BLUETOOTH` 權限。使用時請先看這些套件的說明。

## TTS

## NLU/NLG