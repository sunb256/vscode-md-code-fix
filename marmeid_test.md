# Runner Mermaid

## 実行フローとモジュール構成

```mermaid
flowchart TD
  User["実行ユーザー"] --> CLI["run.ts<br/>CLIエントリ"]

  CLI --> Args["引数解析<br/>parseConfigPathOption<br/>parseRuntimeOptions"]
  CLI --> ConfigLoader["loadRunnerConfig<br/>config-loader.ts"]
  CLI --> TaskLoader["loadTasks<br/>task-loader.ts"]
  CLI --> Merge["defaults統合<br/>mergeTaskDefaults + resolveDefaultsCwd"]
  CLI --> RotateLog["setupRotatingLog<br/>logs/log.log"]

  ConfigLoader --> ConfigYml["config/config.yml"]
  TaskLoader --> ActionYml["tasks/projects/*/action.yml"]

  CLI --> Spawn["codex app-server を spawn"]
  Spawn --> Transport["JsonlTransport<br/>JSONL over stdio"]
  Transport --> Client["CodexAppServerClient"]

  subgraph AppLayer["アプリ層"]
    Client --> Notify["notification.ts<br/>通知処理"]
    Client --> Request["request.ts<br/>承認/入力要求処理"]
    Client --> Reply["continueConversationIfNeeded<br/>harfauto/fullauto"]
  end

  Transport <--> Codex["codex app-server"]

  CLI --> TaskLoop["taskごとに turn/start"]
  TaskLoop --> Client
  Client --> TurnWait["waitForTurnCompletion"]
  TurnWait --> Reply

  RotateLog --> LogFile["logs/log.log<br/>10MB x 5世代"]
```

## 1タスク実行時のシーケンス

```mermaid
sequenceDiagram
  actor User as ユーザー
  participant Runner as run.ts
  participant Loader as config/task loader
  participant Client as CodexAppServerClient
  participant Transport as JsonlTransport
  participant Server as codex app-server

  User->>Runner: runner を起動
  Runner->>Loader: config.yml / action.yml を読込
  Loader-->>Runner: config + tasks
  Runner->>Server: spawn codex app-server
  Runner->>Transport: JsonlTransport を初期化
  Runner->>Client: initialize()
  Client->>Transport: request initialize
  Transport->>Server: JSON-RPC initialize
  Server-->>Transport: result
  Client->>Transport: notify initialized

  Runner->>Client: startThread(...)
  Client->>Transport: request thread/start
  Transport->>Server: thread/start
  Server-->>Transport: thread.id

  Runner->>Client: startTurn(task action)
  Client->>Transport: request turn/start
  Transport->>Server: turn/start

  loop 実行中イベント
    Server-->>Transport: notification(item/agentMessage/delta など)
    Transport-->>Client: onNotification
    Client-->>Runner: 標準出力へ逐次表示
    Server-->>Transport: server request(requestApproval/requestUserInput)
    Transport-->>Client: onServerRequest
    Client->>User: 承認/入力を質問
    User-->>Client: 回答
    Client->>Transport: respond / respondError
    Transport->>Server: JSON-RPC response
  end

  Server-->>Transport: notification(turn/completed)
  Transport-->>Client: turn完了通知
  Client-->>Runner: waitForTurnCompletion を解放
  Runner->>Client: continueConversationIfNeeded()
  Runner-->>User: タスク完了を表示
```
