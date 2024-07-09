# 實作完整的對話引擎

我們已經逐一實作了 ASR、NLU、NLG、TTS 引擎以及對話流程，現在，就可以根據前章[〈語音助理的基本組成〉](components.md)所述，將各個元件串接起來，形成一個完整的對話引擎。

## 初始化

在可以使用這個對話引擎之前，需要先初始化 ASR 引擎。我們於是在對話引擎中加入了一個 `init` method，另外，寫了一個 `_emit` method，用來發送對話引擎的狀態。

```dart
class DialogEngine implements VuiFlowDelegate {
  void _emit(DialogEngineState state) {
    _state = state;
    _stateStream.add(state);
  }

  Future<bool> init() async => await asrEngine.init();
}
```

## 建立意圖與對話流程的對應表格

對話引擎內部會維護一份叫做 \_vuiFlowMap 的表格，之後，當對話引擎收到某個意圖時，就可以去這個表格中尋找是否有對應的對話流程。而我們也設計了一個叫做 `registerFlows` 的 method，可以將新的對話流程加到這個表格中。

```dart
class DialogEngine implements VuiFlowDelegate {
...

  Map<String, VuiFlow> _vuiFlowMap = {};
  VuiFlow? _currentVuiFlow;
...

  /// Resets the list of [VuiFlow].
  void resetFlows() {
    _vuiFlowMap = {};
    _updateIntents();
  }

  /// Register a list of [VuiFlow].
  void registerFlows(List<VuiFlow> flows) {
    for (final flow in flows) {
      _vuiFlowMap[flow.intent] = flow;
      flow.delegate = this;
    }
    _updateIntents();
  }

  void _updateIntents() {
    var intents = <String>{
      'Confirm',
      'Acknowledge',
      'Agree',
      'Cancel',
      'Reject',
      'Disagree',
      'Deny',
    };
    var slots = <String>{};

    for (final key in _vuiFlowMap.keys) {
      intents.add(key);
      final flow = _vuiFlowMap[key];
      if (flow == null) {
        continue;
      }
      slots.addAll(flow.slots);
    }
    nluEngine.availableIntents = intents;
    nluEngine.availableSlots = slots;
  }

...
}
```

當有新的對話流程被加入到對話引擎時，我們會呼叫 `_updateIntents` method，這個 method 會將所有的意圖與 Slot 都更新到 NLU 引擎中，這樣，NLU 引擎就可以知道目前有哪些意圖與 Slot 可以被辨識。

## 實作 VUI flow Delegate

每個 VUI flow 的 delegate，其實就是我們的對話引擎。所以我們需要實作 `VuiFlowDelegate` 這個 interface。

```dart

class DialogEngine implements VuiFlowDelegate {
...
  @override
  Future<void> onEndingConversation() async {
    await stop();
  }

  @override
  Future<String?> onGeneratingResponse(
    String utterance, {
    bool useDefaultPrompt = true,
  }) async {
    return await nlgEngine.generateResponse(
      utterance,
      useDefaultPrompt: useDefaultPrompt,
    );
  }

  @override
  Future<void> onPlayingPrompt(String prompt) async {
    await ttsEngine.stopPlaying();
    _emit(DialogEnginePlayingTts(prompt: prompt));
    await ttsEngine.playPrompt(prompt);
  }

  @override
  Future<void> onSettingCurrentVuiFlow(VuiFlow? vuiFlow) async {
    vuiFlow?.delegate = this;
    _currentVuiFlow = vuiFlow;
  }

  @override
  Future<void> onStartingAsr() async {
    await start(clearCurrentVuiFlow: false);
  }
}

```

然後 `start` 與 `stop` 的實作如下。大概就是開始以及停止 ASR/NLU 引擎的相關工作。可以想像如果 NLU 或 NLG 引擎正在運作中，也應該停止，不過這邊就先偷懶不寫。

```dart
  Future<bool> start({
    clearCurrentVuiFlow = true,
  }) async {
    if (!asrEngine.isInitialized) {
      return false;
    }

    if (clearCurrentVuiFlow) {
      _currentVuiFlow?.cancel();
      _currentVuiFlow = null;
    }
    await ttsEngine.stopPlaying();
    await asrEngine.stopRecognition();
    await asrEngine.startRecognition();
    _emit(DialogEngineListening(asrResult: ''));
    return true;
  }

  Future<bool> stop() async {
    if (!asrEngine.isInitialized) {
      return false;
    }

    _currentVuiFlow?.cancel();
    _currentVuiFlow = null;
    await ttsEngine.stopPlaying();
    await asrEngine.stopRecognition();
    _emit(DialogEngineIdling());
    return true;
  }
```

## 連接 ASR 與 NLU 引擎

在開始建立這個對話引擎的時候，我們就開始監聽 ASR 引擎的狀態。如果 ASR 引擎還在識別當中，我們就透過更新狀態，反應目前的辨識結果，如果辨識完成，就開始讓 NLU 引擎分析意圖（也就是 `handleInput` 這段）。這邊有一小段邏輯，在於處理使用者完全不說話的狀態—如果完全沒有識別結果，而且不在對話流程中，就會直接進入閒置狀態。

```dart
  DialogEngine({
    required this.asrEngine,
    required this.ttsEngine,
    required this.nluEngine,
    required this.nlgEngine,
  }) {
    asrEngine.onResult = (result, isFinal) async {
      if (!isFinal) {
        _emit(DialogEngineListening(asrResult: result));
      } else {
        await handleInput(result);
      }
    };
    asrEngine.onError = (error) async {
      await ttsEngine.stopPlaying();
      await asrEngine.stopRecognition();
      _emit(DialogEngineIdling());
    };
    asrEngine.onStatusChange = (state) async {
      if (state == AsrEngineState.listening) {
        return;
      }
      final current = this.state;
      if (current is DialogEngineCompleteListening) {
        return;
      }
      if (current is DialogEngineListening) {
        if (current.asrResult != '') {
          return;
        }
      }

      if (_currentVuiFlow != null) {
        final intent = NluIntent(intent: '', slots: {});
        await _currentVuiFlow
            ?.handle(intent)
            .timeout(const Duration(seconds: 60));
        return;
      }

      if (current is DialogEngineListening) {
        _emit(DialogEngineIdling());
      }
    };
  }
```

至於 `handleInput` 的內容如下。我們平常會暴露 `handleInput`，因此，外部可以不用真的透過 ASR 語音錄音，而是直接對這個 method 傳入一段文字，就可以啟動 NLU 引擎以及對話流程，進行整合測試。

```dart
  String fallbackErrorMessage = 'Sorry, I do not understand for now.';

  Future handleInput(String input) async {
    _emit(DialogEngineCompleteListening(asrResult: input));
    await ttsEngine.stopPlaying();
    await asrEngine.stopRecognition();

    try {
      final additionalPrompt = _collectionAdditionalNluPrompt();
      final intent = await nluEngine.extractIntent(
        input,
        currentIntent: _currentVuiFlow?.intent,
        additionalRequirement: additionalPrompt,
      );
      if (_currentVuiFlow != null) {
        await _currentVuiFlow
            ?.handle(intent)
            .timeout(const Duration(seconds: 60));
        return;
      }
      final flow = _vuiFlowMap[intent.intent];
      if (flow != null) {
        await flow.handle(intent).timeout(const Duration(seconds: 60));
        return;
      }
      final prompt =
          await nlgEngine.generateResponse(input) ?? fallbackErrorMessage;
      await onPlayingPrompt(prompt);
      await stop();
    } catch (e) {
      _currentVuiFlow = null;
      await onPlayingPrompt(fallbackErrorMessage);
      await stop();
    }

```

到這裡，我們已經完成了我們的對話引擎。

## 整合測試

在我們急著把這個對話引擎整合到我們的應用程式之前，我們可以先寫一些整合測試，來確保這個對話引擎的功能是正確的。這邊我們可以使用 `flutter_test` 這個套件，來寫一些測試。測試的內容是，我們先用「我想請假」，啟動 `LeaveApplication` 意圖，但由於欠缺日期與事由，所以會進入多輪對話，然後我們再輸入「明天下午我想出去玩」，這樣就可以完成這個對話流程，最後驗證是否正確到了 System Call。為了避免因為網路斷線等問題，造成測試卡住，我們預期二十秒之內要完成測試。

```dart
  test('Test Engine with Leave Application', () async {
    final engine = DialogEngine(
      asrEngine: MockAsrEngine(),
      ttsEngine: MockTtsEngine(),
      nluEngine: GeminiNluEngine(apiKey: key),
      nlgEngine: GeminiNlgEngine(apiKey: key),
    );

    final completer = Completer();
    var systemCallCalled = true;
    engine.registerFlows([
      LeaveApplicationVuiFlow(
          onMakingLeaveApplication: (reason, date, text) async {
        expect(date, '明天下午');
        expect(reason, '出去玩');
        completer.complete();
        return true;
      })
    ]);
    await engine.init();
    await engine.handleInput('我想請假');
    await Future.delayed(const Duration(seconds: 5));
    await engine.handleInput('明天下午我想出去玩');
    await completer.future.timeout(const Duration(seconds: 20));
    expect(systemCallCalled, isTrue);
  });
```

在這個整合測試當中，我們也示範了怎樣從外部建立我們的對話引擎的作法。

- 首先建立對話引擎的 instance，當中選擇了我們要使用的 ASR、NLU、NLG、TTS 引擎
- 對引擎註冊我們要使用的 VUI flow。
- 初始化對話引擎

之後，我們透過 `handleInput` 測試，在實際的 app 中，我們則用 `start` method 來啟動對話引擎。
