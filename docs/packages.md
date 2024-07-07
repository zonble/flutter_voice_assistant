# 可用的 Flutter 套件

在進入實作 ASR、NLU、NLG、TTS 的套件之前，我們先來看看 Flutter 有哪些套件可以用來實作這些功能。基本上，我們使用 speech、TTS、等關鍵字，就可以在 [Dart Pub](https://pub.dev/) 上找到許多套件。

## ASR 相關套件

- [speech_to_text](https://pub.dev/packages/speech_to_text)：支援 iOS、Android 以及 Web 的語音辨識套件。這個套件主要用平台原生的語音辨識功能。在使用這個方案時，開發者不用支付額外的服務費用，這也是目前在 Dart Pub 上評分最高、最流行的語音辨識套件。
- [google_speech](https://pub.dev/packages/google_speech)：使用 Google 雲端服務的語音辨識套件。
- [azure_speech_recognition_null_safety](https://pub.dev/packages/azure_speech_recognition_null_safety)：使用 Azure 雲端服務的語音辨識套件。
- [openai_api](https://pub.dev/packages/openai_api)：當中包含了 OpenAI 的語音辨識服務。
- [ifly_speech_recognition](https://pub.dev/packages/ifly_speech_recognition)：科大訊飛是一家對岸的語音服務公司，也提供了連接他們的服務的 Flutter 套件。

在使用這些套件時，往往還需要一些額外的權限與隱私設定。像是 iOS 上可能需要在 Info.plist 中設定 `NSMicrophoneUsageDescription` 與 `NSSpeechRecognitionUsageDescription`，告知用戶為什麼要使用麥克風與語音辨識權限。而在 Android 上，可能需要在 AndroidManifest.xml 中設定 `RECORD_AUDIO` 權限，而如果我們想要用藍芽裝置錄音，那可能還要額外設定 `BLUETOOTH` 權限。使用時請先看這些套件的說明。

## TTS 相關套件

- [flutter_tts](https://pub.dev/packages/flutter_tts)：支援 iOS、Android、Web、macOS 與 Windows 文字轉語音套件。一樣使用平台原生的語音合成功能，因此開發者不用支付額外的服務費用，這也是目前在 Dart Pub 上評分最高、最流行的文字轉語音套件。
- [text_to_speech](https://pub.dev/packages/text_to_speech)：支援 iOS、Android、Web 與 macOS。
- [galli_text_to_speech](https://pub.dev/packages/galli_text_to_speech)：另外一套支援 iOS、Android、Web 與 macOS 的文字轉語音套件。
- [cloud_text_to_speech](https://pub.dev/packages/cloud_text_to_speech)：介接了 Google、微軟以及 Amazon 的文字轉語音雲端服務的套件。
- [flutter_azure_tts](https://pub.dev/packages/flutter_azure_tts)：使用 Azure 雲端服務的文字轉語音套件。
- [openai_api](https://pub.dev/packages/openai_api)：當中包含了 OpenAI 的文字轉聲音服務。

在 Dart Pub 上，我們還可以看到有人介接了騰訊、百度、科大訊飛等公司的文字轉語音服務，不過看起來一陣子沒有維護。

## NLU/NLG 相關套件

如前所述，我們計畫使用目前流行的 LLM 來實作 NLU 與 NLG。

### Gemini

- [google_generative_ai](https://pub.dev/packages/google_generative_ai)：Google 的官方 Gemini 套件，支援各種平台。
- [flutter_gemini](https://pub.dev/packages/flutter_gemini)：另外一套 Gemini 實作
- [gemini_flutter](https://pub.dev/packages/gemini_flutter)：另外一套 Gemini 實作

### Gemma

- [flutter_gemma](https://pub.dev/packages/flutter_gemma)：可以讓 Flutter App 使用 Gemma 本地模型的套件。

### ChatGPT

當然也有不少與 ChatGPT 有關的套件。

- [flutter_chatgpt_api](https://pub.dev/packages/flutter_chatgpt_api)
- [chat_gpt_api](https://pub.dev/packages/chat_gpt_api)
- [chat_gpt_flutter](https://pub.dev/packages/chat_gpt_flutter)
