# ポモドーロタイマー Webアプリケーション アーキテクチャ設計書

## 1. 概要

このドキュメントは、FlaskとHTML/CSS/JavaScriptを使用したポモドーロタイマーWebアプリケーションのアーキテクチャ設計をまとめたものです。テスタビリティ、保守性、拡張性を重視した設計となっています。

## 2. プロジェクト構造

```
1.pomodoro/
├── app.py                      # Flaskアプリケーションエントリーポイント
├── pomodoro/
│   ├── __init__.py
│   ├── routes.py              # ルート定義（薄いレイヤー）
│   ├── services.py            # ビジネスロジック
│   ├── models.py              # データモデル
│   └── utils.py               # ユーティリティ関数
├── static/
│   ├── css/
│   │   └── style.css          # スタイルシート
│   ├── js/
│   │   ├── timer.js           # タイマークラス（純粋なロジック）
│   │   ├── ui.js              # UI操作（DOM依存）
│   │   └── app.js             # アプリケーション統合
│   └── audio/
│       └── notification.mp3   # 通知音（オプション）
├── templates/
│   └── index.html             # メインUIテンプレート
├── tests/
│   ├── __init__.py
│   ├── test_services.py       # バックエンドロジックのテスト
│   ├── test_routes.py         # APIエンドポイントのテスト
│   ├── test_models.py         # モデルのテスト
│   └── frontend/
│       ├── test_timer.spec.js # Jest/Mochaでのフロントエンドテスト
│       └── setup.js
├── requirements.txt           # 本番依存関係
├── requirements-dev.txt       # 開発・テスト用依存関係
├── pytest.ini                 # pytest設定
├── package.json               # フロントエンドテスト用
└── architecture.md            # 本ドキュメント
```

## 3. アーキテクチャパターン

### 3.1 全体アーキテクチャ

**フロントエンド重視アプローチ**を採用します。

```
┌─────────────────────────────────────────────────┐
│                  ブラウザ                         │
├─────────────────────────────────────────────────┤
│  UI Layer (ui.js)                               │
│    ↕                                            │
│  Business Logic (timer.js)                      │
│    ↕                                            │
│  Storage (localStorage)                         │
│    ↕                                            │
│  API Client (fetch)                             │
└─────────────────────────────────────────────────┘
           ↕ HTTP/JSON
┌─────────────────────────────────────────────────┐
│            Flask Backend                         │
├─────────────────────────────────────────────────┤
│  Routes Layer (routes.py)                       │
│    ↕                                            │
│  Service Layer (services.py)                    │
│    ↕                                            │
│  Data Layer (models.py)                         │
│    ↕                                            │
│  Database (SQLite/PostgreSQL)                   │
└─────────────────────────────────────────────────┘
```

### 3.2 責務の分離

#### **フロントエンド**
- **タイマーロジック**: JavaScriptで実装（クライアントサイド）
- **状態管理**: ブラウザのlocalStorageでセッション履歴を保存
- **リアルタイム性**: setInterval/setTimeoutを使用
- **UI更新**: バニラJavaScript（DOM操作）

**メリット**:
- サーバー負荷が少ない
- オフラインでも動作可能
- レスポンスが早い
- テストが容易

#### **バックエンド（Flask）**
- 静的ファイル配信
- テンプレートレンダリング
- RESTful API提供
- セッション履歴の永続化（オプション）
- 統計データの集計・取得（オプション）

## 4. 主要コンポーネント設計

### 4.1 フロントエンド

#### **timer.js - タイマークラス（ピュアロジック）**

```javascript
export class PomodoroTimer {
    constructor(duration, onTick, onComplete) {
        this.duration = duration;        // 初期時間（秒）
        this.remaining = duration;       // 残り時間（秒）
        this.onTick = onTick;           // コールバック: 毎秒呼ばれる
        this.onComplete = onComplete;   // コールバック: 完了時
        this.intervalId = null;
    }
    
    start() { /* タイマー開始 */ }
    stop() { /* タイマー停止 */ }
    reset() { /* タイマーリセット */ }
    getProgress() { /* 進捗率を返す（0-1） */ }
}
```

**特徴**:
- DOM操作を含まない純粋なロジック
- コールバックパターンで疎結合
- 時間をモック可能（テスト容易）

#### **ui.js - UI操作クラス**

```javascript
export class TimerUI {
    constructor(elements) {
        this.elements = elements;  // DOM要素の参照
    }
    
    updateDisplay(seconds) { /* 表示更新 */ }
    formatTime(seconds) { /* 時間フォーマット */ }
    showNotification(message) { /* 通知表示 */ }
    updateProgressBar(progress) { /* 進捗バー更新 */ }
}
```

**特徴**:
- DOM操作に特化
- ビジネスロジックを持たない
- 再利用可能なUI部品

#### **app.js - アプリケーション統合**

```javascript
import { PomodoroTimer } from './timer.js';
import { TimerUI } from './ui.js';

// 初期化とイベントバインディング
const ui = new TimerUI({ /* DOM要素 */ });
const timer = new PomodoroTimer(
    25 * 60,
    (remaining) => ui.updateDisplay(remaining),
    () => ui.showNotification('完了！')
);

// イベントリスナー設定
document.getElementById('start-btn').addEventListener('click', () => {
    timer.start();
});
```

### 4.2 バックエンド

#### **services.py - ビジネスロジック層**

```python
from datetime import datetime

class PomodoroService:
    """ポモドーロセッション管理サービス"""
    
    def __init__(self, db_adapter):
        self.db = db_adapter
    
    def create_session(self, duration, session_type, timestamp=None):
        """セッションを作成（依存性注入可能）"""
        if timestamp is None:
            timestamp = datetime.now()
        
        session = {
            'duration': duration,
            'type': session_type,
            'timestamp': timestamp,
            'completed': False
        }
        return self.db.insert_session(session)
    
    def complete_session(self, session_id):
        """セッションを完了としてマーク"""
        return self.db.update_session(session_id, {'completed': True})
    
    def get_daily_stats(self, date):
        """日次統計を取得"""
        sessions = self.db.get_sessions_by_date(date)
        return {
            'total_sessions': len(sessions),
            'completed_sessions': sum(1 for s in sessions if s['completed']),
            'total_minutes': sum(s['duration'] for s in sessions) / 60
        }
```

**特徴**:
- ピュアなビジネスロジック
- フレームワークに依存しない
- 依存性注入でテスト容易

#### **routes.py - ルーティング層**

```python
from flask import Blueprint, jsonify, request, current_app

bp = Blueprint('pomodoro', __name__)

@bp.route('/api/sessions', methods=['POST'])
def create_session():
    """セッション作成API"""
    data = request.json
    service = current_app.config['pomodoro_service']
    
    session = service.create_session(
        duration=data['duration'],
        session_type=data['type']
    )
    return jsonify(session), 201

@bp.route('/api/sessions/<int:session_id>', methods=['PATCH'])
def update_session(session_id):
    """セッション更新API"""
    service = current_app.config['pomodoro_service']
    session = service.complete_session(session_id)
    return jsonify(session)

@bp.route('/api/stats/daily', methods=['GET'])
def get_daily_stats():
    """日次統計取得API"""
    date = request.args.get('date', datetime.now().date())
    service = current_app.config['pomodoro_service']
    stats = service.get_daily_stats(date)
    return jsonify(stats)
```

**特徴**:
- 薄いレイヤー（リクエスト/レスポンスの変換のみ）
- ビジネスロジックをサービス層に委譲
- Flask Test Clientでテスト可能

#### **app.py - アプリケーションファクトリ**

```python
from flask import Flask, render_template
from pomodoro import routes
from pomodoro.services import PomodoroService
from pomodoro.models import SQLiteAdapter

def create_app(config=None, db_adapter=None):
    """アプリケーションファクトリパターン"""
    app = Flask(__name__)
    
    if config:
        app.config.update(config)
    
    # 依存性注入
    if db_adapter is None:
        db_adapter = SQLiteAdapter('pomodoro.db')
    
    app.config['db_adapter'] = db_adapter
    app.config['pomodoro_service'] = PomodoroService(db_adapter)
    
    # ブループリント登録
    app.register_blueprint(routes.bp)
    
    @app.route('/')
    def index():
        return render_template('index.html')
    
    return app

if __name__ == '__main__':
    app = create_app()
    app.run(debug=True)
```

**特徴**:
- ファクトリパターンでテスト時に設定変更可能
- 依存性注入でモック差し込み可能

## 5. データフロー

### 5.1 タイマー実行フロー

```
ユーザーがスタートボタンクリック
    ↓
app.js: イベントハンドラ起動
    ↓
timer.js: PomodoroTimer.start()
    ↓
setInterval でカウントダウン開始
    ↓
1秒ごとに onTick コールバック実行
    ↓
ui.js: TimerUI.updateDisplay()
    ↓
DOM更新（画面に残り時間表示）
    ↓
タイマー完了時 onComplete コールバック
    ↓
ui.js: 通知表示
    ↓
localStorage: セッション履歴保存
    ↓
（オプション）Flask API: セッション記録
```

### 5.2 データ永続化フロー

```
localStorage (クライアント)
    ↓
ユーザーが統計を表示
    ↓
fetch API でサーバーに履歴同期
    ↓
Flask API: POST /api/sessions
    ↓
services.py: create_session()
    ↓
models.py: データベースに挿入
    ↓
レスポンス返却
```

## 6. テスタビリティ設計

### 6.1 設計原則

1. **依存性注入**: テスト時にモック・スタブを差し込める
2. **関心の分離**: ビジネスロジック、UI、インフラを分離
3. **純粋関数優先**: 副作用を持たない関数を多用
4. **時間の抽象化**: タイマー機能を注入可能に
5. **DOM分離**: ロジックとDOM操作を分離

### 6.2 テスト戦略

#### **テストピラミッド**

```
        /\
       /E2E\          <- 少数（Selenium/Playwright）
      /------\
     /統合Test \       <- 中程度（Flask Test Client）
    /----------\
   /ユニットTest \     <- 多数（個別関数・クラス）
  /--------------\
```

#### **カバレッジ目標**
- ユニットテスト: 80%以上
- 統合テスト: 主要APIパス全て
- E2Eテスト: クリティカルパスのみ

### 6.3 テスト例

#### **バックエンドテスト**

```python
# tests/test_services.py
import pytest
from datetime import datetime
from pomodoro.services import PomodoroService

class MockDBAdapter:
    def __init__(self):
        self.sessions = []
    
    def insert_session(self, session):
        session['id'] = len(self.sessions) + 1
        self.sessions.append(session)
        return session

def test_create_session():
    """セッション作成のテスト"""
    mock_db = MockDBAdapter()
    service = PomodoroService(mock_db)
    
    fixed_time = datetime(2026, 2, 24, 10, 0, 0)
    session = service.create_session(
        duration=25,
        session_type='work',
        timestamp=fixed_time
    )
    
    assert session['duration'] == 25
    assert session['type'] == 'work'
    assert session['timestamp'] == fixed_time
    assert len(mock_db.sessions) == 1

def test_daily_stats():
    """日次統計のテスト"""
    mock_db = MockDBAdapter()
    service = PomodoroService(mock_db)
    
    # セッション作成
    service.create_session(25, 'work')
    service.create_session(5, 'short_break')
    
    stats = service.get_daily_stats(datetime.now().date())
    
    assert stats['total_sessions'] == 2
    assert stats['total_minutes'] == 30 / 60
```

#### **フロントエンドテスト**

```javascript
// tests/frontend/test_timer.spec.js
import { PomodoroTimer } from '../../static/js/timer.js';

describe('PomodoroTimer', () => {
    beforeEach(() => {
        jest.useFakeTimers();  // モックタイマー
    });
    
    afterEach(() => {
        jest.useRealTimers();
    });
    
    test('タイマーが正しくカウントダウンする', () => {
        const onTick = jest.fn();
        const onComplete = jest.fn();
        const timer = new PomodoroTimer(5, onTick, onComplete);
        
        timer.start();
        
        // 1秒進める
        jest.advanceTimersByTime(1000);
        expect(onTick).toHaveBeenCalledWith(4);
        
        // 残り4秒進める
        jest.advanceTimersByTime(4000);
        expect(onComplete).toHaveBeenCalled();
    });
    
    test('進捗率を正しく計算する', () => {
        const timer = new PomodoroTimer(100, () => {}, () => {});
        timer.remaining = 75;
        
        expect(timer.getProgress()).toBe(0.25);
    });
    
    test('停止とリセットが正しく動作する', () => {
        const timer = new PomodoroTimer(60, () => {}, () => {});
        
        timer.start();
        jest.advanceTimersByTime(10000);
        
        timer.stop();
        expect(timer.remaining).toBe(50);
        
        timer.reset();
        expect(timer.remaining).toBe(60);
    });
});
```

#### **統合テスト**

```python
# tests/test_routes.py
import pytest
from app import create_app
from pomodoro.models import MockDBAdapter

@pytest.fixture
def client():
    """テスト用クライアント"""
    mock_db = MockDBAdapter()
    app = create_app(
        config={'TESTING': True},
        db_adapter=mock_db
    )
    with app.test_client() as client:
        yield client

def test_create_session_api(client):
    """セッション作成APIのテスト"""
    response = client.post('/api/sessions', json={
        'duration': 25,
        'type': 'work'
    })
    
    assert response.status_code == 201
    data = response.get_json()
    assert data['duration'] == 25
    assert data['type'] == 'work'

def test_get_daily_stats_api(client):
    """統計取得APIのテスト"""
    # セッション作成
    client.post('/api/sessions', json={'duration': 25, 'type': 'work'})
    client.post('/api/sessions', json={'duration': 5, 'type': 'short_break'})
    
    # 統計取得
    response = client.get('/api/stats/daily')
    
    assert response.status_code == 200
    data = response.get_json()
    assert data['total_sessions'] == 2
```

## 7. 主要機能仕様

### 7.1 タイマー機能

- **作業時間**: デフォルト25分（カスタマイズ可能）
- **短い休憩**: デフォルト5分
- **長い休憩**: デフォルト15分（4セッション後）
- **操作**: 開始/一時停止/リセット
- **状態管理**: 現在のモード、残り時間、セッション数

### 7.2 UI コンポーネント

- タイマー表示（分:秒）
- 開始/一時停止/リセットボタン
- モード切替（作業/休憩）
- 進捗表示（セッション数、プログレスバー）
- 設定パネル（時間カスタマイズ）
- 統計表示（今日の完了セッション数）

### 7.3 通知機能

- ブラウザのNotification API
- 音声アラート
- タブタイトルでの通知

### 7.4 データ永続化

- **ローカル**: localStorage（セッション履歴、設定）
- **サーバー**: SQLite/PostgreSQL（オプション）
- **同期**: ローカルとサーバーの双方向同期

## 8. 技術スタック

### 8.1 バックエンド
- **フレームワーク**: Flask 3.x
- **データベース**: SQLite（開発）/ PostgreSQL（本番）
- **テスト**: pytest, pytest-flask, pytest-cov
- **その他**: python-dotenv（環境変数管理）

### 8.2 フロントエンド
- **JavaScript**: ES6+ モジュール
- **スタイリング**: CSS3（Flexbox/Grid）、カスタムCSS変数
- **アイコン**: Font Awesome または SVG
- **テスト**: Jest, @testing-library/dom
- **ビルド**: 不要（バニラJS）

### 8.3 開発ツール
- **バージョン管理**: Git
- **CI/CD**: GitHub Actions
- **コードフォーマット**: Black（Python）、Prettier（JavaScript）
- **リンター**: flake8（Python）、ESLint（JavaScript）

## 9. 開発フェーズ

### フェーズ1: MVP（最小機能）
- [x] アーキテクチャ設計
- [ ] 基本的なタイマー機能（クライアントサイド）
- [ ] シンプルなUI
- [ ] 開始/停止/リセット
- [ ] 音声通知

### フェーズ2: 機能拡張
- [ ] セッション履歴（localStorage）
- [ ] カスタマイズ可能な時間設定
- [ ] 統計表示（今日の完了セッション数）
- [ ] プログレスバー
- [ ] レスポンシブデザイン

### フェーズ3: バックエンド統合
- [ ] Flask APIエンドポイント実装
- [ ] データベーススキーマ設計
- [ ] サーバー側セッション管理
- [ ] ローカル・サーバー同期機能

### フェーズ4: 高度な機能
- [ ] ユーザー認証
- [ ] 生産性レポート（週次・月次）
- [ ] タスク管理機能との統合
- [ ] カスタムテーマ

## 10. セキュリティ考慮事項

- **CSRF保護**: Flask-WTFでトークン生成
- **入力検証**: サーバーサイドで厳密に検証
- **SQLインジェクション**: ORMまたはパラメータ化クエリ使用
- **認証**: Flask-Login（フェーズ4）
- **HTTPS**: 本番環境では必須

## 11. パフォーマンス考慮事項

- **クライアント側処理**: タイマーロジックでサーバー負荷軽減
- **キャッシング**: 静的ファイルのブラウザキャッシュ
- **API最適化**: 必要な場合のみサーバーにアクセス
- **データベース**: 適切なインデックス設計

## 12. 拡張性

このアーキテクチャは以下の拡張を容易にします：

- **マイクロサービス化**: サービス層が独立しているため分離容易
- **他のフロントエンド**: React/Vueへの移行が容易
- **モバイルアプリ**: API経由で同じバックエンドを利用可能
- **リアルタイム機能**: WebSocketの追加が容易
- **分析機能**: セッションデータを活用した高度な分析

## 13. まとめ

このアーキテクチャは以下の特徴を持ちます：

✅ **テスタビリティ**: 依存性注入、関心の分離により高いテストカバレッジを実現
✅ **保守性**: レイヤー分離により変更が局所化
✅ **拡張性**: 段階的な機能追加が容易
✅ **パフォーマンス**: クライアント側処理でサーバー負荷軽減
✅ **開発効率**: 明確な責務分離とモジュール設計

---

**ドキュメント更新日**: 2026年2月24日
**バージョン**: 1.0
**ステータス**: 承認済み
