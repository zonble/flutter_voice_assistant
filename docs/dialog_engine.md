# 實作完整的對話引擎

我們已經逐一實作了 ASR、NLU、NLG、TTS 引擎以及對話流程，現在，就可以根據前章[〈語音助理的基本組成〉](components.md)所述，將各個元件串接起來，形成一個完整的對話引擎。

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
