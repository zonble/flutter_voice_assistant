# 確認與取消

會進入多輪對話的場合，除了想辦法蒐集完成一項任務所需要的所有 Slot 外（做法則向前面提到的，重複複製同一個 VuiFlow 子類別，循環使用），就是在要進行工作之前，與使用者再次確認。直到使用者說了「對」、「好」、「沒錯」等句子，才會實際呼叫 System Call。

## 確認

因為向使用者確認也是一段對話，我們也可以把這段對話包裝在 `VuiFlow` 中。

```dart

const _maxErrorCount = 5;

class ConfirmVuiFlow extends VuiFlow {
  final VuiFlow positiveFlow;
  final VuiFlow negativeFlow;
  var errorCount = 0;

  ConfirmVuiFlow({
    required this.positiveFlow,
    required this.negativeFlow,
  });

  @override
  Future<void> handle(NluIntent intent) async {
    if (['Confirm', 'Acknowledge', 'Agree'].contains(intent.intent)) {
      positiveFlow.delegate = delegate;
      positiveFlow.handle(NluIntent(intent: '', slots: {}));
    } else if (['Cancel', 'Reject', 'Disagree', 'Deny']
        .contains(intent.intent)) {
      negativeFlow.delegate = delegate;
      negativeFlow.handle(NluIntent(intent: '', slots: {}));
    } else {
      errorCount += 1;
      if (errorCount >= _maxErrorCount) {
        await delegate?.onPlayingPrompt('很抱歉，錯誤次數過多');
        await delegate?.onEndingConversation();
        return;
      }
      await delegate?.onPlayingPrompt('很抱歉，我不懂你的意思，你確定嗎？');
      final newFlow =
          ConfirmVuiFlow(positiveFlow: positiveFlow, negativeFlow: negativeFlow)
            ..errorCount = errorCount;
      await delegate?.onSettingCurrentVuiFlow(newFlow);
      await delegate?.onStartingAsr();
    }
  }

  @override
  String get intent => '';

  @override
  List<String> get slots => <String>[];
}


```

`ConfirmVuiFlow` 有兩個成員變數：`positiveFlow` 與 `negativeFlow`，分別代表使用者回答「對」與「不對」時，要進入的下一個 VuiFlow。在 `handle` 中，我們檢查使用者的回答，如果是「對」，就進入 `positiveFlow`，如果是「不對」，就進入 `negativeFlow`。如果使用者的回答不是我們預期的，我們會再次詢問使用者，直到錯誤次數過多，就結束對話。

## 取消

我們需要特別注意，使用者不一定在確認的流程中說出「取消」，而很有可能在各種其他的狀況下說出來。像是，在點餐系統中，使用者才剛說完「我想點餐」，我們用「您想要點什麼」回應之後，使用者突然反悔，可能是突然想去別家店，也可能是看不到喜歡的餐點，而說了「取消」，想要結束對話。

我們這時候可以有兩種選擇：一種就是在原本「你想要點什麼」的 VuiFlow 中，多判斷傳入的意圖是不是取消，如果是，那就進入結束對話的流程分支。這麼做在我們有很多的 VuiFlow，語音助理的功能很多時，就很容易產生大量重複的程式碼，因為等於每個詢問，都需要處理取消的意圖。我們也可以實作一個全域的判斷，在任何時刻，只要使用者說出「取消」，就結束對話。修改的地方就在對話引擎的 `handleInput` 這段：

```dart
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

    if (intent.intent == 'Cancel') {
        // 播放 TTS
        stop();
        return;
    }
    ...

```

兩種作法之間並沒有哪種比較好，而是根據實際的產品需求決定。
