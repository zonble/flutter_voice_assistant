# 實作 ASR 與 TTS 引擎

## ASR 引擎

我們選擇用 [speech_to_text](https://pub.dev/packages/speech_to_text) 實作我們的 ASR 引擎。可以看到，其實我們就只是簡單地把 speech_to_text 包裝一層。而使用 speech_to_text 時要注意一點，就是 `cancel()` 與 `stop()` 的差異，兩者雖然都會停止辨識，但是 `stop()` 會把目前辨識中的結果強制送到 `onResult` 這個 callback 中。

```dart
import 'package:speech_to_text/speech_to_text.dart' as stt;
import '../interface/asr_engine.dart';

const _pauseForSeconds = 3;

class PlatformAsrEngine extends AsrEngine {
  /// The default locale ID.
  var _localeId = 'en_US';
  var _isInitialized = false;

  @override
  Future<bool> init() async {
    if (_isInitialized) {
      return _isInitialized;
    }

    final speech = stt.SpeechToText();

    _isInitialized = await speech.initialize(
        onStatus: (status) {
          final map = {
            stt.SpeechToText.listeningStatus: AsrEngineState.listening,
            stt.SpeechToText.notListeningStatus: AsrEngineState.notListening,
            stt.SpeechToText.doneStatus: AsrEngineState.done,
          };
          var state = map[status] ?? AsrEngineState.notListening;
          onStatusChange?.call(state);
        },
        onError: (error) => onError?.call(error));
    if (!_isInitialized) {
      print("The user has denied the use of speech recognition.");
    }
    return _isInitialized;
  }

  @override
  Future<bool> startRecognition() async {
    if (!_isInitialized) {
      return false;
    }
    final speech = stt.SpeechToText();
    speech.listen(
        pauseFor: const Duration(seconds: _pauseForSeconds),
        localeId: _localeId,
        listenOptions: stt.SpeechListenOptions(
          listenMode: stt.ListenMode.search,
          partialResults: true,
          onDevice: false,
          cancelOnError: true,
        ),
        onResult: (result) {
          final words = result.recognizedWords;
          final isFinal = result.finalResult;
          onResult?.call(words, isFinal);
        });
    return true;
  }

  @override
  Future<bool> stopRecognition() async {
    if (!_isInitialized) {
      return false;
    }
    final speech = stt.SpeechToText();
    await speech.cancel();
    return true;
  }

  @override
  Future<void> setLanguage(String language) async {
    _localeId = language;
  }

  @override
  bool get isInitialized => _isInitialized;
}

```

## TTS 引擎

至於 TTS，我們也只是把 [flutter_tts](https://pub.dev/packages/flutter_tts) 簡單包裝一層。在這邊，我們希望 `await playPrompt(prompt) ` 的時候，可以等整句話都播放完畢再執行下一行，但 flutter_tts 的行為並不是這樣，所以額外增加了一個 Completer 等待。

```dart
import 'dart:async';
import 'package:flutter_tts/flutter_tts.dart';
import '../interface/tts_engine.dart';

/// Please refer to https://pub.dev/packages/flutter_tts to update your Android
/// manifest and iOS configuration.
class PlatformTtsEngine extends TtsEngine {
  final flutterTTs = FlutterTts();
  Completer? _ttsCompleter;

  PlatformTtsEngine() {
    flutterTTs.setStartHandler(() {
      onStart?.call();
    });
    flutterTTs.setCompletionHandler(() {
      _ttsCompleter?.complete();
      _ttsCompleter = null;
      onComplete?.call();
    });
    flutterTTs.setProgressHandler((text, startOffset, endOffset, word) {
      onProgress?.call(text, startOffset, endOffset, word);
    });
    flutterTTs.setErrorHandler((msg) {
      _ttsCompleter?.complete();
      _ttsCompleter = null;
      onError?.call(msg);
    });
    flutterTTs.setCancelHandler(() {
      _ttsCompleter?.complete();
      _ttsCompleter = null;
      onCancel?.call();
    });
    flutterTTs.setPauseHandler(() {
      print('TTS setPauseHandler');
      onPause?.call();
    });
    flutterTTs.setContinueHandler(() {
      print('TTS setContinueHandler');
      onContinue?.call();
    });
  }

  @override
  Future<void> playPrompt(String prompt) async {
    await flutterTTs.speak(prompt);
    var ttsCompleter = Completer();
    _ttsCompleter = ttsCompleter;
    await ttsCompleter.future;
  }

  @override
  Future<void> stopPlaying() async {
    await flutterTTs.stop();
    _ttsCompleter?.complete();
    _ttsCompleter = null;
  }

  @override
  Future<void> setLanguage(String language) async {
    await flutterTTs.setLanguage(language);
  }

  @override
  Future<void> setPitch(double pitch) async {
    await flutterTTs.setPitch(pitch);
  }

  @override
  Future<void> setSpeechRate(double rate) async {
    await flutterTTs.setSpeechRate(rate);
  }

  @override
  Future<void> setVolume(double volume) async {
    await flutterTTs.setVolume(volume);
  }

  @override
  Future<void> setVoice(Map<String, String> voice) async {
    flutterTTs.setVoice(voice);
  }
}

```

## Mock

上面的 ASR 與 NLU 引擎，由於用到了平台的功能，所以沒辦法在單元測試使用。為了測試方面，我們可以實作 Mock 的 ASR 與 NLU 引擎。基本上就是基於 `AsrEngine`與 `TtsEngine` 的介面，但是不做任何的實作。

```dart
import '../interface/asr_engine.dart';

/// A mock ASR (Automatic Speech Recognition) engine for testing.
class MockAsrEngine extends AsrEngine {
  @override
  Future<bool> init() async {
    return true;
  }

  @override
  bool get isInitialized => true;

  @override
  Future<bool> startRecognition() async {
    return true;
  }

  @override
  Future<bool> stopRecognition() async {
    return true;
  }

  @override
  Future<void> setLanguage(String language) async {}
}
```

```dart
import '../interface/tts_engine.dart';

/// A mock TTS (Text to Speech) engine for testing.
class MockTtsEngine extends TtsEngine {
  @override
  Future<void> playPrompt(String prompt) async {
    print('MockTtsEngine play $prompt');
    await Future.delayed(const Duration(milliseconds: 200));
    onComplete?.call();
  }

  @override
  Future<void> setLanguage(String language) async {}

  @override
  Future<void> setPitch(double pitch) async {}

  @override
  Future<void> setSpeechRate(double rate) async {}

  @override
  Future<void> setVoice(Map<String, String> voice) async {}

  @override
  Future<void> setVolume(double volume) async {}

  @override
  Future<void> stopPlaying() async {
    onCancel?.call();
  }
}

```

在後續章節[〈實作完整的對話引擎〉](dialog_engine.md)中，會說明如何使用這些 Mock 的 ASR 與 TTS 引擎。
