# 語音助理的基本組成

在這邊需要介紹一些專有名詞。一套語音助理系統的組成，通常會包括—ASR、NLU、NLG、TTS 等，在一個完整的語音互動中，扮演各自的角色。

- ASR：Automatic Speech Recognition，自動語音識別，又稱為 STR （Speech-to-Text），負責將語音訊號轉換為文字。對於行動開發者來說，應該清楚蘋果提供了 [Sppech](https://developer.apple.com/documentation/speech) 框架，在 Android 系統中也有 [SpeechRecognizer](https://developer.android.com/reference/android/speech/SpeechRecognizer) 可用，在瀏覽器中也一樣有 [SpeechRecognition](https://developer.mozilla.org/en-US/docs/Web/API/SpeechRecognition)。除此之外，許多雲端服務，像是 Google Cloud 與 Azure，也具備將上傳的語音資料轉換成文字的服務。
- NLU：Natural Language Understanding，自然語言理解，負責從文字中抽取出意圖，像是知道使用者想要做什麼事情，想要問什麼問題，也可以分析用戶所說的話當中的情緒（悲傷、高興、憤怒），或是口吻是正式還是輕鬆等。在行動開發的框架中，通常沒有這部份的框架，但是所有的交談型 LLM 都一定包含 NLU 的能力。此外，比較有名的 NLU 引擎包括 [Google Dialogflow](https://cloud.google.com/dialogflow/docs/)、[Microsoft LUIS](https://learn.microsoft.com/en-us/azure/ai-services/luis/)、[IBM Watson](https://www.ibm.com/products/natural-language-understanding)、[Nuance Recognizer](https://www.nuance.com/omni-channel-customer-engagement/contact-center-ai/nuance-recognizer.html) 等等。
- NLG：Natural Language Generation，自然語言生成，負責產生對特定文字的回覆。其實各種 LLM，就是在擔任 NLG 引擎的工作。
- TTS：Text-to-Speech。負責將文字轉換為語音。在行動開發者的框架中，蘋果提供了 [AVSpeechSynthesizer](https://developer.apple.com/documentation/avfoundation/avspeechsynthesizer)、Google 提供了 [TextToSpeech](https://developer.android.com/reference/android/speech/tts/TextToSpeech)。在瀏覽器中，也有 [SpeechSynthesis](https://developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesis) 可用。雲端服務中，像是 Google Cloud 與 Azure 也提供了將文字轉換成語音的服務。

## 各種組件的分工

