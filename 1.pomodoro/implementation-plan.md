# ポモドーロタイマー 段階的実装計画

このドキュメントは、ポモドーロタイマーアプリケーションの段階的な実装計画を定義します。各イテレーションで具体的な成果物を作成し、動作確認しながら段階的に機能を追加していきます。

---

## 実装方針

### 基本原則

1. **動く最小限の機能から始める** - 各段階で動作するアプリケーションを維持
2. **画面を見ながら開発** - UIデザインに忠実に実装し、視覚的フィードバックを重視
3. **テスタビリティを確保** - 各コンポーネントを独立してテスト可能に
4. **段階的なコミット** - 各イテレーション完了時にコミット

### 開発環境

- Python 3.9+
- Flask 3.x
- モダンブラウザ（Chrome/Firefox/Safari）
- テスト: pytest（バックエンド）、Jest（フロントエンド・後で追加）

---

## イテレーション1: 最小限の動作確認（0.5-1日）

### 目標
「Hello World」レベルのアプリケーションを動かし、開発環境を確立する

### 実装内容

#### 1.1 プロジェクト構造の構築
```
1.pomodoro/
├── app.py
├── requirements.txt
├── static/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── app.js
└── templates/
    └── index.html
```

#### 1.2 最小限のFlaskアプリ
- `app.py`: 基本的なFlaskアプリケーション
- ルート `/` でHTMLを表示
- 静的ファイルの配信設定

#### 1.3 超シンプルなHTML
- 基本的なHTML構造のみ
- 「ポモドーロタイマー」のタイトル
- プレースホルダーで「25:00」を表示

#### 1.4 基本的なCSS
- Google Fontsの読み込み
- 背景色の設定（紫グラデーション）
- 基本的なレイアウト（中央配置）

### 成果物
- ブラウザで http://localhost:5000 にアクセスすると、タイトルと「25:00」が表示される
- 背景がデザインと同じ紫色

### 動作確認
```bash
cd 1.pomodoro
pip install -r requirements.txt
python app.py
# ブラウザで http://localhost:5000 を開く
```

### 完了条件
- [ ] Flask開発サーバーが起動する
- [ ] ブラウザにタイトルと時間が表示される
- [ ] 背景色がデザイン通りに表示される

---

## イテレーション2: 静的なUI完成（1-1.5日）

### 目標
画像と同じ見た目の静的なUIを完成させる（動作はしない）

### 実装内容

#### 2.1 HTMLの完全実装
- メインカードコンテナ
- タイマー表示エリア（25:00）
- 状態表示（「作業中」）
- 円形プログレスバー（SVG）
- ボタンエリア（開始、リセット）
- 統計表示エリア（「今日の進捗」、完了数、集中時間）

#### 2.2 CSSの完全実装
- カードのスタイル（白背景、角丸、シャドウ）
- タイポグラフィ（フォントサイズ、ウェイト）
- 円形プログレスバーのスタイル
  - 背景円（グレー）
  - 進捗円（青紫、dasharray使用）
- ボタンのスタイル
  - 「開始」ボタン（塗りつぶし、青紫）
  - 「リセット」ボタン（アウトライン、青紫枠）
  - ホバー効果
- 統計表示エリアのスタイル（薄い青紫背景）
- レスポンシブ対応（スマホでも見やすく）

#### 2.3 CSS変数の定義
```css
:root {
    --primary-color: #6c63ff;
    --background-gradient: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    --text-dark: #2d3748;
    --text-light: #718096;
    --card-background: #ffffff;
    --card-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}
```

### 成果物
- 画像と同じ見た目のUI
- ボタンをクリックしても何も起こらない（まだ実装していない）

### 動作確認
- デザインと見比べて視覚的に確認
- 各要素のサイズ、配置、色を確認
- スマホ表示でも崩れないことを確認

### 完了条件
- [ ] デザイン画像と80%以上一致している
- [ ] すべての要素が正しく配置されている
- [ ] ホバー効果が動作する
- [ ] レスポンシブ表示が正しく動作する

---

## イテレーション3: タイマーコアロジック（1-1.5日）

### 目標
タイマーの基本的なカウントダウン機能を実装する（UIとの連携はまだ）

### 実装内容

#### 3.1 `static/js/timer.js` の実装

```javascript
export class PomodoroTimer {
    constructor(duration, onTick, onComplete) {
        this.duration = duration;
        this.remaining = duration;
        this.onTick = onTick;
        this.onComplete = onComplete;
        this.intervalId = null;
        this.isActive = false;
    }
    
    start() {
        if (this.isActive) return;
        
        this.isActive = true;
        this.intervalId = setInterval(() => {
            this.remaining--;
            
            if (this.onTick) {
                this.onTick(this.remaining);
            }
            
            if (this.remaining <= 0) {
                this.stop();
                if (this.onComplete) {
                    this.onComplete();
                }
            }
        }, 1000);
    }
    
    stop() {
        if (this.intervalId) {
            clearInterval(this.intervalId);
            this.intervalId = null;
        }
        this.isActive = false;
    }
    
    reset() {
        this.stop();
        this.remaining = this.duration;
    }
    
    isRunning() {
        return this.isActive;
    }
    
    getProgress() {
        return 1 - (this.remaining / this.duration);
    }
}
```

#### 3.2 ブラウザコンソールでの動作確認
- `index.html` に `<script type="module">` で読み込み
- コンソールで手動でインスタンスを作成してテスト

### 成果物
- タイマークラスが正しくカウントダウンする
- コールバックが正しく呼ばれる

### 動作確認（ブラウザコンソール）
```javascript
const timer = new PomodoroTimer(
    5, 
    (remaining) => console.log('残り:', remaining),
    () => console.log('完了！')
);
timer.start();
// 5秒後に「完了！」が表示されることを確認
```

### 完了条件
- [ ] タイマーが正しくカウントダウンする
- [ ] `start()`, `stop()`, `reset()` が正しく動作する
- [ ] `getProgress()` が正しい値を返す
- [ ] コールバックが適切なタイミングで呼ばれる

---

## イテレーション4: UI操作クラスとタイマー連携（1-1.5日）

### 目標
タイマーとUIを連携させ、カウントダウンが画面に表示されるようにする

### 実装内容

#### 4.1 `static/js/ui.js` の実装

```javascript
export class TimerUI {
    constructor(elements) {
        this.elements = elements;
    }
    
    updateDisplay(seconds) {
        const formatted = this.formatTime(seconds);
        this.elements.timerDisplay.textContent = formatted;
    }
    
    formatTime(seconds) {
        const mins = Math.floor(seconds / 60);
        const secs = seconds % 60;
        return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    }
    
    updateProgressBar(progress) {
        // SVG円形プログレスバーの更新
        const circumference = 2 * Math.PI * 120; // 半径120の円周
        const offset = circumference * (1 - progress);
        this.elements.progressCircle.style.strokeDashoffset = offset;
    }
    
    setMode(mode) {
        const modeText = {
            'work': '作業中',
            'short_break': '休憩中',
            'long_break': '長い休憩'
        };
        this.elements.modeDisplay.textContent = modeText[mode] || '作業中';
    }
    
    updateButtonState(isRunning) {
        if (isRunning) {
            this.elements.startButton.textContent = '一時停止';
        } else {
            this.elements.startButton.textContent = '開始';
        }
    }
}
```

#### 4.2 `static/js/app.js` の実装

```javascript
import { PomodoroTimer } from './timer.js';
import { TimerUI } from './ui.js';

document.addEventListener('DOMContentLoaded', () => {
    // DOM要素の取得
    const elements = {
        timerDisplay: document.getElementById('timer-display'),
        progressCircle: document.getElementById('progress-circle'),
        modeDisplay: document.getElementById('mode-display'),
        startButton: document.getElementById('start-button'),
        resetButton: document.getElementById('reset-button')
    };
    
    // UI初期化
    const ui = new TimerUI(elements);
    
    // タイマー初期化（25分 = 1500秒）
    const timer = new PomodoroTimer(
        25 * 60,
        (remaining) => {
            ui.updateDisplay(remaining);
            ui.updateProgressBar(timer.getProgress());
        },
        () => {
            console.log('タイマー完了！');
            ui.updateButtonState(false);
        }
    );
    
    // 初期表示
    ui.updateDisplay(timer.remaining);
    ui.updateProgressBar(0);
    
    // イベントリスナー
    elements.startButton.addEventListener('click', () => {
        if (timer.isRunning()) {
            timer.stop();
        } else {
            timer.start();
        }
        ui.updateButtonState(timer.isRunning());
    });
    
    elements.resetButton.addEventListener('click', () => {
        timer.reset();
        ui.updateDisplay(timer.remaining);
        ui.updateProgressBar(0);
        ui.updateButtonState(false);
    });
});
```

#### 4.3 HTMLの更新
- 各要素にIDを追加（`id="timer-display"` など）
- `<script type="module" src="/static/js/app.js"></script>` を追加

### 成果物
- 「開始」ボタンをクリックするとカウントダウンが始まる
- タイマー表示が毎秒更新される
- 円形プログレスバーが進む
- 「一時停止」で停止、再度クリックで再開
- 「リセット」で初期状態に戻る

### 動作確認
1. 開始ボタンをクリック → カウントダウン開始
2. タイマー表示が「24:59, 24:58...」と減っていく
3. プログレスバーが時計回りに進む
4. 一時停止ボタンでストップ
5. リセットボタンで25:00に戻る

### 完了条件
- [ ] すべてのボタンが期待通りに動作する
- [ ] タイマー表示が正しく更新される
- [ ] プログレスバーがスムーズに動く
- [ ] 一時停止・再開が正しく動作する

---

## イテレーション5: 通知機能（0.5-1日）

### 目標
タイマー完了時に通知を表示する

### 実装内容

#### 5.1 TimerUIに通知メソッドを追加

```javascript
showNotification(message) {
    // ブラウザ通知
    if ('Notification' in window && Notification.permission === 'granted') {
        new Notification('ポモドーロタイマー', {
            body: message,
            icon: '/static/icon.png' // オプション
        });
    }
    
    // 視覚的な通知（モーダル風）
    const notification = document.createElement('div');
    notification.className = 'notification';
    notification.textContent = message;
    document.body.appendChild(notification);
    
    setTimeout(() => {
        notification.classList.add('show');
    }, 10);
    
    setTimeout(() => {
        notification.classList.remove('show');
        setTimeout(() => notification.remove(), 300);
    }, 3000);
}

requestNotificationPermission() {
    if ('Notification' in window && Notification.permission === 'default') {
        Notification.requestPermission();
    }
}
```

#### 5.2 CSSに通知スタイルを追加

```css
.notification {
    position: fixed;
    top: 20px;
    right: 20px;
    background: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
    transform: translateX(400px);
    transition: transform 0.3s ease;
    z-index: 1000;
}

.notification.show {
    transform: translateX(0);
}
```

#### 5.3 app.jsで通知を呼び出す

```javascript
// ページ読み込み時に権限をリクエスト
ui.requestNotificationPermission();

// タイマー完了時
const timer = new PomodoroTimer(
    25 * 60,
    (remaining) => {
        ui.updateDisplay(remaining);
        ui.updateProgressBar(timer.getProgress());
    },
    () => {
        ui.showNotification('お疲れ様でした！休憩しましょう。');
        ui.updateButtonState(false);
    }
);
```

### 成果物
- タイマー完了時に通知が表示される
- ブラウザ通知（許可されている場合）
- 画面上の視覚的な通知

### 動作確認
1. ページを開いたときに通知権限のリクエストが表示される（初回のみ）
2. タイマーを5秒などの短い時間に設定してテスト
3. 完了時に通知が表示されることを確認

### 完了条件
- [ ] 通知権限のリクエストが動作する
- [ ] ブラウザ通知が表示される（許可時）
- [ ] 画面上の視覚的通知が表示される
- [ ] 通知が自動的に消える

---

## イテレーション6: ローカルストレージと統計表示（1-1.5日）

### 目標
セッション履歴を保存し、統計を表示する

### 実装内容

#### 6.1 StorageManager クラスの作成

```javascript
// static/js/storage.js
export class StorageManager {
    constructor() {
        this.STORAGE_KEY = 'pomodoro_sessions';
    }
    
    saveSession(session) {
        const sessions = this.getSessions();
        sessions.push({
            ...session,
            timestamp: new Date().toISOString()
        });
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(sessions));
    }
    
    getSessions() {
        const data = localStorage.getItem(this.STORAGE_KEY);
        return data ? JSON.parse(data) : [];
    }
    
    getTodayStats() {
        const sessions = this.getSessions();
        const today = new Date().toDateString();
        
        const todaySessions = sessions.filter(s => {
            const sessionDate = new Date(s.timestamp).toDateString();
            return sessionDate === today;
        });
        
        const workSessions = todaySessions.filter(s => s.type === 'work');
        const totalMinutes = workSessions.reduce((sum, s) => sum + s.duration / 60, 0);
        
        return {
            completed: workSessions.length,
            totalMinutes: Math.round(totalMinutes)
        };
    }
}
```

#### 6.2 UIに統計更新メソッドを追加

```javascript
// ui.js に追加
updateStats(stats) {
    this.elements.completedSessions.textContent = stats.completed;
    this.elements.totalTime.textContent = `${Math.floor(stats.totalMinutes / 60)}時間${stats.totalMinutes % 60}分`;
}
```

#### 6.3 app.jsでストレージを統合

```javascript
import { StorageManager } from './storage.js';

const storage = new StorageManager();

// 初期表示時に統計を更新
const stats = storage.getTodayStats();
ui.updateStats(stats);

// タイマー完了時にセッションを保存
const timer = new PomodoroTimer(
    25 * 60,
    (remaining) => {
        ui.updateDisplay(remaining);
        ui.updateProgressBar(timer.getProgress());
    },
    () => {
        // セッションを保存
        storage.saveSession({
            type: 'work',
            duration: 25 * 60
        });
        
        // 統計を更新
        const stats = storage.getTodayStats();
        ui.updateStats(stats);
        
        ui.showNotification('お疲れ様でした！休憩しましょう。');
        ui.updateButtonState(false);
    }
);
```

### 成果物
- タイマー完了後、セッション履歴がlocalStorageに保存される
- 統計表示が自動的に更新される
- ページをリロードしても統計が保持される

### 動作確認
1. タイマーを完了させる
2. 統計表示が更新される（4 → 5）
3. ブラウザの開発者ツールでlocalStorageを確認
4. ページをリロードしても統計が表示される

### 完了条件
- [ ] セッション履歴が正しく保存される
- [ ] 統計が正しく計算される
- [ ] 統計表示がリアルタイムで更新される
- [ ] ページリロード後も統計が保持される

---

## イテレーション7: モード切替とサイクル管理（1-1.5日）

### 目標
作業 → 短い休憩 → 作業 → ... → 長い休憩のサイクルを実装

### 実装内容

#### 7.1 PomodoroManagerクラスの作成

```javascript
// static/js/pomodoro-manager.js
export class PomodoroManager {
    constructor(settings = {}) {
        this.settings = {
            workDuration: settings.workDuration || 25 * 60,
            shortBreakDuration: settings.shortBreakDuration || 5 * 60,
            longBreakDuration: settings.longBreakDuration || 15 * 60,
            longBreakInterval: settings.longBreakInterval || 4
        };
        
        this.currentMode = 'work';
        this.sessionsCompleted = 0;
    }
    
    getCurrentDuration() {
        switch (this.currentMode) {
            case 'work':
                return this.settings.workDuration;
            case 'short_break':
                return this.settings.shortBreakDuration;
            case 'long_break':
                return this.settings.longBreakDuration;
        }
    }
    
    completeSession() {
        if (this.currentMode === 'work') {
            this.sessionsCompleted++;
            
            // 4セッションごとに長い休憩
            if (this.sessionsCompleted % this.settings.longBreakInterval === 0) {
                this.currentMode = 'long_break';
            } else {
                this.currentMode = 'short_break';
            }
        } else {
            // 休憩後は作業に戻る
            this.currentMode = 'work';
        }
    }
    
    getNextMode() {
        if (this.currentMode === 'work') {
            const nextSession = this.sessionsCompleted + 1;
            if (nextSession % this.settings.longBreakInterval === 0) {
                return 'long_break';
            }
            return 'short_break';
        }
        return 'work';
    }
    
    reset() {
        this.currentMode = 'work';
        this.sessionsCompleted = 0;
    }
}
```

#### 7.2 app.jsでサイクル管理を統合

```javascript
import { PomodoroManager } from './pomodoro-manager.js';

const pomodoroManager = new PomodoroManager();

function createTimer() {
    const duration = pomodoroManager.getCurrentDuration();
    
    return new PomodoroTimer(
        duration,
        (remaining) => {
            ui.updateDisplay(remaining);
            ui.updateProgressBar(timer.getProgress());
        },
        () => {
            // セッション完了
            if (pomodoroManager.currentMode === 'work') {
                storage.saveSession({
                    type: 'work',
                    duration: duration
                });
                const stats = storage.getTodayStats();
                ui.updateStats(stats);
            }
            
            pomodoroManager.completeSession();
            
            // 次のモードのメッセージ
            const messages = {
                'work': '休憩終了！作業を始めましょう。',
                'short_break': 'お疲れ様！短い休憩です。',
                'long_break': 'お疲れ様！長い休憩です。'
            };
            
            ui.showNotification(messages[pomodoroManager.currentMode]);
            ui.setMode(pomodoroManager.currentMode);
            
            // 新しいタイマーを作成（自動開始はしない）
            timer = createTimer();
            ui.updateDisplay(timer.remaining);
            ui.updateProgressBar(0);
            ui.updateButtonState(false);
        }
    );
}

let timer = createTimer();
ui.setMode(pomodoroManager.currentMode);
```

### 成果物
- 作業完了後、自動的に休憩モードに切り替わる
- 4セッション後は長い休憩
- 各モードで適切な時間が設定される

### 動作確認
1. 作業タイマーを完了させる → 「短い休憩」モードに
2. 休憩タイマーを完了させる → 「作業中」モードに
3. 4回作業を完了させる → 「長い休憩」モードに

### 完了条件
- [ ] モードが自動的に切り替わる
- [ ] 各モードで正しい時間が設定される
- [ ] 4セッション後に長い休憩になる
- [ ] モード表示が更新される

---

## イテレーション8: 設定機能（1-1.5日）

### 目標
時間のカスタマイズを可能にする

### 実装内容

#### 8.1 設定モーダルのHTML追加

```html
<div id="settings-modal" class="modal">
    <div class="modal-content">
        <h2>設定</h2>
        <div class="setting-item">
            <label>作業時間（分）</label>
            <input type="number" id="work-duration" min="1" max="60" value="25">
        </div>
        <div class="setting-item">
            <label>短い休憩（分）</label>
            <input type="number" id="short-break-duration" min="1" max="30" value="5">
        </div>
        <div class="setting-item">
            <label>長い休憩（分）</label>
            <input type="number" id="long-break-duration" min="1" max="60" value="15">
        </div>
        <div class="modal-actions">
            <button id="save-settings">保存</button>
            <button id="cancel-settings">キャンセル</button>
        </div>
    </div>
</div>
<button id="settings-button">⚙️</button>
```

#### 8.2 設定の保存・読み込み

```javascript
// storage.js に追加
saveSettings(settings) {
    localStorage.setItem('pomodoro_settings', JSON.stringify(settings));
}

getSettings() {
    const data = localStorage.getItem('pomodoro_settings');
    return data ? JSON.parse(data) : null;
}
```

#### 8.3 app.jsで設定機能を統合

```javascript
// 設定を読み込んでPomodoroManagerを初期化
const savedSettings = storage.getSettings();
const pomodoroManager = new PomodoroManager(savedSettings);

// 設定モーダルの表示/非表示
document.getElementById('settings-button').addEventListener('click', () => {
    const modal = document.getElementById('settings-modal');
    modal.classList.add('show');
    
    // 現在の設定を表示
    document.getElementById('work-duration').value = pomodoroManager.settings.workDuration / 60;
    document.getElementById('short-break-duration').value = pomodoroManager.settings.shortBreakDuration / 60;
    document.getElementById('long-break-duration').value = pomodoroManager.settings.longBreakDuration / 60;
});

// 設定の保存
document.getElementById('save-settings').addEventListener('click', () => {
    const newSettings = {
        workDuration: parseInt(document.getElementById('work-duration').value) * 60,
        shortBreakDuration: parseInt(document.getElementById('short-break-duration').value) * 60,
        longBreakDuration: parseInt(document.getElementById('long-break-duration').value) * 60,
        longBreakInterval: 4
    };
    
    storage.saveSettings(newSettings);
    pomodoroManager.settings = newSettings;
    
    // モーダルを閉じる
    document.getElementById('settings-modal').classList.remove('show');
    
    // タイマーをリセット
    timer.reset();
    timer = createTimer();
    ui.updateDisplay(timer.remaining);
    ui.updateProgressBar(0);
});
```

### 成果物
- 設定ボタンから時間をカスタマイズできる
- 設定が保存され、次回起動時に反映される

### 動作確認
1. 設定ボタン（⚙️）をクリック
2. 作業時間を30分に変更して保存
3. タイマーが30:00になることを確認
4. ページをリロードしても30分が維持される

### 完了条件
- [ ] 設定モーダルが正しく表示される
- [ ] 設定が保存される
- [ ] 設定がタイマーに反映される
- [ ] ページリロード後も設定が維持される

---

## イテレーション9: 音声アラート（オプション、0.5日）

### 目標
タイマー完了時に音声を再生する

### 実装内容

#### 9.1 音声ファイルの追加
- `static/audio/notification.mp3` を配置
- または Web Audio API で簡単な音を生成

#### 9.2 ui.jsに音声再生メソッドを追加

```javascript
playSound() {
    const audio = new Audio('/static/audio/notification.mp3');
    audio.volume = 0.5;
    audio.play().catch(err => {
        console.log('音声再生エラー:', err);
    });
}
```

#### 9.3 タイマー完了時に音声を再生

```javascript
// app.js のタイマー完了時
() => {
    ui.playSound();
    ui.showNotification(messages[pomodoroManager.currentMode]);
    // ... その他の処理
}
```

### 成果物
- タイマー完了時に音が鳴る

### 動作確認
1. タイマーを完了させる
2. 音が鳴ることを確認
3. ブラウザの音声設定を確認

### 完了条件
- [ ] 音声ファイルが正しく再生される
- [ ] 音量が適切
- [ ] エラーハンドリングが適切

---

## イテレーション10: バックエンド基盤（オプション、2-3日）

### 目標
FlaskのRESTful APIを実装し、サーバーサイドでデータを管理

### 実装内容

#### 10.1 プロジェクト構造の拡張
```
1.pomodoro/
├── app.py
├── pomodoro/
│   ├── __init__.py
│   ├── routes.py
│   ├── services.py
│   └── models.py
└── pomodoro.db
```

#### 10.2 models.py の実装
- SQLiteAdapterクラス
- データベーススキーマの作成
- CRUD操作

#### 10.3 services.py の実装
- PomodoroServiceクラス
- ビジネスロジック

#### 10.4 routes.py の実装
- RESTful APIエンドポイント
- `/api/sessions` (POST, GET)
- `/api/stats/daily` (GET)

#### 10.5 app.py の更新
- アプリケーションファクトリパターン
- Blueprintの登録

#### 10.6 フロントエンドからAPIを呼び出す
```javascript
// storage.js に API通信を追加
async syncToServer(session) {
    try {
        const response = await fetch('/api/sessions', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(session)
        });
        return await response.json();
    } catch (err) {
        console.error('同期エラー:', err);
    }
}
```

### 成果物
- サーバーサイドでセッション履歴を管理
- オフライン時はローカルストレージ、オンライン時はサーバーに同期

### 動作確認
1. タイマーを完了させる
2. ブラウザの開発者ツールでネットワークタブを確認
3. POSTリクエストが正しく送信されている
4. データベースにデータが保存されている

### 完了条件
- [ ] APIエンドポイントが正しく動作する
- [ ] データがデータベースに保存される
- [ ] フロントエンドから正しく呼び出せる
- [ ] エラーハンドリングが適切

---

## テスト実装（並行して実施）

### バックエンドテスト
- イテレーション10と並行して実施
- pytest + pytest-flask
- カバレッジ目標: 80%以上

### フロントエンドテスト
- イテレーション5-7の後に実施
- Jest + @testing-library/dom
- タイマーロジックのテスト
- UI操作のテスト

---

## 完成イメージのタイムライン

```
週1: イテレーション1-2（環境構築 + 静的UI）
週2: イテレーション3-5（タイマー機能 + 通知）
週3: イテレーション6-8（ストレージ + モード切替 + 設定）
週4: イテレーション9-10（音声 + バックエンド）+ テスト
```

### 最小限のMVP（2週間）
- イテレーション1-7
- 基本的なポモドーロタイマーとして完全に機能

### フル機能版（4週間）
- すべてのイテレーション
- バックエンド統合、テスト完備

---

## リスクと対策

### リスク1: ブラウザ互換性
- **対策**: モダンブラウザ（Chrome, Firefox, Safari最新版）をターゲットにする
- **対策**: 必要に応じてBabelでトランスパイル

### リスク2: 通知権限
- **対策**: 権限が拒否された場合も、画面上の視覚的通知で対応

### リスク3: localStorage容量制限
- **対策**: 古いデータを定期的に削除する機能を追加
- **対策**: サーバー同期で重要なデータをバックアップ

### リスク4: タイマーの精度
- **対策**: setIntervalの誤差を考慮し、実際の経過時間を計測する方法も検討

---

## 次のステップ

### 推奨アプローチ
1. **イテレーション1から順番に実装**
2. **各イテレーション完了時にGitコミット**
3. **動作確認を必ず実施**
4. **問題があれば次に進まない**

### 最初の一歩
```bash
cd 1.pomodoro
touch requirements.txt
echo "Flask==3.0.0" > requirements.txt
mkdir -p static/css static/js templates
touch app.py static/css/style.css static/js/app.js templates/index.html
```

---

**ドキュメント作成日**: 2026年2月24日
**バージョン**: 1.0
**ステータス**: 作成完了

**推奨**: まずイテレーション1から始めましょう！
