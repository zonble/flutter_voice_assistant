# 前言

2024 and onwards © Weizhong Yang a.k.a zonble

![封面](images/cover.jpg)

這幾年語音助理應用以各種方式走入了每個人的生活。蘋果在 iPhone 4 之後加入了 Siri 功能，也應用在 Apple TV、HomePod 等產品上，Amazon 打造了 Alexa 智慧喇叭，Google 則在 Android 手機與 Google Nest 等裝置上加入了 Google Assistant 功能，之後又有更多的智慧喇叭以及車載語音產品。這些語音助理的功能不僅僅是回答問題，還能夠控制家電、播放音樂、設定鬧鐘等等。

我們可以注意到，這些語音助理有一個共同的特色，就是他們都有自己的生態系統。這些語音助理首先串接的是自家的服務，而幾乎每家都有自己的音樂、地圖導航等。蘋果就有自家的 Apple Music、Amazon 也有自己的 Prime Music、Google 則有 YouTube Music；地圖方面，蘋果一開始與 Google 合作使用 Google Maps，後來也開發了自家的 Apple Maps。或著，就是與一些最大的音樂、氣象、新聞等服務合作，像是 Spotify、Weather.com 等等。

為了擴大生態圈，第三方開發者也可以為這些語音助理開發自己的服務，像是 iOS App 的開發者可以透過[SiriKit](https://developer.apple.com/documentation/sirikit/)，為 Siri 增加功能，讓 Siri 可以串接到 App 的功能。Amazon 則提供了 Amazon 則使用 [Alexa Skills](https://www.amazon.com/-/alexa-skills/b?node=13727921011)，Google 則使用 Actions on Google 來串接各種第三方服務。而由於這些語音助理都是大眾導向的產品，大多建立在這些平台上的服務，也往往是大眾導向的服務，我們還是較少能看到在一些內部、或是專屬的系統上，使用語音助理技術或功能。

這幾年也陸續看到一些店家開始在專屬的服務上提供語音助理功能，像是餐飲店也開始提供語音點餐等，客戶在電家門口的 Kiosk 機台，就可以用語音完成點餐等。但即使如此，我們還是可以期待，語音助理可以應用在更多行業中、更多系統上。

語音助理最適合的場合大概是，當使用者不方便挪出雙手操作，然後又有明確目標，用語音比使用手指或鍵盤滑鼠慢慢選擇有效率時，而且通常會用在比較私密、個人的場合，像我們大概就不會在公共場合大聲叫 Siri 幫我們查詢一些私人資訊。所以我們往往會在駕車時請語音助理協助導航，在家中使用智慧喇叭，以及在運動、慢跑時使用耳機上的語音助理；另一方面，又對周圍環境有一定的要求，周圍如果太過吵雜，往往影響錄音以及語音辨識的效果。

我們或許可以想像，在醫院、病房當中，病床床頭可能擺了台平板，醫護人員或是病人可以快速使用語音命令，完成一些工作或是查詢資料，而不用放開手邊的工作，或是離開病床。剛生完寶寶的母親可以用語音命令，快速查看寶寶的影像。但前提是，語音助理要能夠連接醫院的系統，但目前都還相當欠缺這類型的應用。

以下就是一個使用 Flutter Web 技術開發，運作在 Chrome 瀏覽器當中的語音掛號的功能展示：

<video controls width="500">
  <source src="flutter_voice_assistant.mov" type="video/mp4" />
  在 PDF 檔案中，無法呈現影片，請下載 <a href="https://zonble.github.io/flutter_voice_assistant/flutter_voice_assistant.mov">MP4</a> 影片。
</video>

隨著這幾年 Flutter 應用程式的開發框架愈來愈成熟，加上大型語言模型（LLM）也快速普及，結合這兩者，其實就可以只寫大約幾百行的 Dart 程式，就能夠快速打造跨平台的語音助理。在這本小冊子中，會提到:

- 打造語音助理需要哪些元件
- 這些元件有哪些可用的 Flutter 套件
- 如何串接這些元件，成為完整的語音助理
- 如何設計語音互動流程

我預期您在閱讀這本小冊子之前，具備一定的 Flutter 應用程式的開發基礎與經驗。這本小冊子放在 Github 上，如果您發現錯漏，可以直接在 Github 上發 PR 修正，或是在 Github 上開 Issue 討論。因為內容也是以 Git 管理，因此不額外提供版本歷史。

這本小冊子當中的程式碼，也可以從 [flutter_dialog](https://github.com/zonble/flutter_dialog) 處取得。

本書內容以 CC BY-SA 4.0 授權條款釋出。

<img src="https://mirrors.creativecommons.org/presskit/buttons/88x31/png/by-sa.png" width="88">
