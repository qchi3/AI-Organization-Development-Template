# Smart Advertising System 最終要件定義書 v4.0

## 目次
1. [サービス概要](#1-サービス概要)
2. [ステークホルダー要件](#2-ステークホルダー要件)
3. [システムアーキテクチャ](#3-システムアーキテクチャ)
4. [機能要件](#4-機能要件)
5. [非機能要件](#5-非機能要件)
6. [データ設計](#6-データ設計)
7. [ハードウェア仕様](#7-ハードウェア仕様)
8. [セキュリティ要件](#8-セキュリティ要件)
9. [運用要件](#9-運用要件)
10. [開発計画](#10-開発計画)

---

## 1. サービス概要

### 1.1 ビジョン
店舗の会計台や待合スペースに設置したディスプレイを活用し、視聴者数に基づいた広告収益をディスプレイ所有者と分配する、次世代のデジタルサイネージプラットフォーム。

### 1.2 コアバリュー
- **即時性**: 視線検知から0.05秒以内に最適広告を表示（タブレット版）
- **簡便性**: 5分で設置完了、即日広告配信開始
- **透明性**: リアルタイムで視聴者数と収益を可視化
- **拡張性**: タブレットから大型ディスプレイまで対応

### 1.3 ビジネスモデル
```
収益源:
- 広告収入の50%をプラットフォーム手数料
- 店舗支援機能（月額¥1,000-2,000）
- データ分析サービス（月額¥1,500）
- プレミアムサポート（月額¥500）

コスト構造:
Phase1（タブレット）:
- ハードウェア: ¥15,000/台
- 保護ケース・スタンド: ¥3,000/台
- 月間運用費: ¥200/台

Phase2（MiniPC）:
- EdgeKit製造: ¥25,000/台
- SIM通信費: ¥110/月/台
- CDN費用: ¥20/月/台
```

### 1.4 展開戦略
```yaml
Phase1（0-6ヶ月）:
  - Androidタブレット展開
  - 会計台・小規模店舗向け
  - 目標: 1,000台

Phase2（6-12ヶ月）:
  - MiniPC版開発・展開
  - 大型ディスプレイ対応
  - 目標: 追加500台

Phase3（12ヶ月以降）:
  - 統合プラットフォーム
  - 全国展開
  - 目標: 10,000台
```

---

## 2. ステークホルダー要件

### 2.1 広告主要件

#### 2.1.1 基本要件
- 最低出稿額: ¥5,000/週（タブレット）、¥10,000/週（大型）
- 地図上でエリア選択（半径100m単位）
- デバイスタイプ選択（タブレット/大型ディスプレイ）
- ターゲティング: 年齢・性別・時間帯・業種
- クリエイティブ: MP4/画像（タブレット5MB、大型20MB）
- 決済方法: クレジットカード/PayPay/請求書

#### 2.1.2 レポート要件
```yaml
リアルタイム指標:
  - 実視認インプレッション（2秒以上視聴）
  - デバイスタイプ別配信状況
  - 業種別パフォーマンス
  - 消化予算

日次レポート:
  - CPM/CPC（デバイスタイプ別）
  - 視聴完了率
  - タップ数（タブレット）/QRスキャン数（大型）
  - 推定来店数
  - 時間帯別パフォーマンス
  - ROI分析
```

### 2.2 ディスプレイ所有者要件

#### 2.2.1 登録要件（タブレット版）
- アプリダウンロードで3分登録
- 必要情報: 店舗名、住所、業種
- 本人確認: SMS認証
- 初期費用: ¥0（条件付き無償提供）
- 保証金: ¥5,000（クレジットカード仮押さえ）

#### 2.2.2 登録要件（MiniPC版）
- Web/LINEから5分で登録
- 必要情報: 住所、ディスプレイサイズ、設置写真
- 保証金: ¥10,000（クレジットカード仮押さえ）
- 本人確認: SMS認証＋身分証明書

#### 2.2.3 収益モデル
```
タブレット版:
- 基本収益: 視聴者数×¥0.2-0.5/視聴
- 最低保証: ¥1,000/月（稼働率80%以上）
- 店舗支援機能: 無料〜¥1,000/月

MiniPC版:
- 基本収益: 視聴者数×¥0.3-1.0/視聴
- 最低保証: ¥3,000/月（条件付き）
- 自社広告枠: 15秒/5分（無料）

共通特典:
- 友達紹介: ¥2,000/台
- ランキングボーナス: 上位10%に+20%
- 支払い: 月末締め翌月15日振込
```

### 2.3 システム管理者要件

#### 2.3.1 監視項目
- デバイス稼働率（目標99%）
- デバイスタイプ別パフォーマンス
- 広告配信成功率
- 不正検知（視聴数異常）
- アプリクラッシュ率（タブレット）
- バッテリー状態（タブレット）

#### 2.3.2 運用ツール
- 統合管理ダッシュボード
- MDM（Mobile Device Management）
- リモートデバイス制御
- A/Bテスト管理
- CSツール連携

---

## 3. システムアーキテクチャ

### 3.1 技術スタック
```yaml
共通基盤:
  認証/DB: Supabase (PostgreSQL + Auth)
  リアルタイム: EMQX Cloud (MQTT)
  Edge Functions: Deno Deploy
  CDN: Bunny.net
  決済: Stripe Connect
  監視: Datadog

Phase1 - タブレット版:
  OS: Android 10以上
  開発: Kotlin + Jetpack Compose
  AI: TensorFlow Lite
  通信: Retrofit + OkHttp
  ローカルDB: Room
  DI: Hilt
  
Phase2 - MiniPC版:
  ハードウェア: Intel N100 Mini PC
  OS: Ubuntu 22.04 LTS
  ランタイム: Python 3.10 + OpenVINO
  通信: Soracom IoT SIM

Web/管理画面:
  広告主UI: Next.js 14 + TypeScript
  所有者UI: React + Vite
  管理画面: Refine + Ant Design
```

### 3.2 システム構成図
```
┌─────────────────────────────────────────────────────┐
│                   クラウド層                          │
├─────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │Supabase  │  │  EMQX    │  │ Bunny.net│        │
│  │(DB/Auth) │  │ (MQTT)   │  │  (CDN)   │        │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
│       │             │             │                │
│  ┌────▼─────────────▼─────────────▼────┐          │
│  │      Edge Functions (Deno)          │          │
│  │  - /match (広告マッチング)            │          │
│  │  - /metrics (集計処理)              │          │
│  │  - /payout (収益計算)               │          │
│  └─────────────────────────────────────┘          │
└─────────────────────────────────────────────────────┘
                        │
                    Internet
           ┌────────────┴────────────┐
           │                         │
    ┌──────▼──────┐         ┌───────▼──────┐
    │  Phase1     │         │   Phase2     │
    │  タブレット   │         │   MiniPC     │
    └──────────────┘         └──────────────┘
```

### 3.3 タブレット版アーキテクチャ
```yaml
アプリ構成:
  UI層:
    - Jetpack Compose
    - Material Design 3
    - カスタムアニメーション
    
  ビジネスロジック層:
    - ViewModels (MVVM)
    - Use Cases
    - Repository Pattern
    
  データ層:
    - Room Database
    - Retrofit API Client
    - DataStore Preferences
    
  AI処理層:
    - TensorFlow Lite
    - ML Kit Vision
    - エッジ推論
```

---

## 4. 機能要件

### 4.1 Phase1: タブレット版機能

#### 4.1.1 コンテンツ表示機能
```kotlin
class TabletContentManager {
    private val displayMode = MutableStateFlow(DisplayMode.ADVERTISEMENT)
    private val contentQueue = mutableListOf<Content>()
    
    fun switchContent(trigger: Trigger) {
        when (trigger) {
            is CustomerDetected -> {
                // 0.05秒以内に切替
                val bestAd = localMatcher.findBestAd(trigger.profile)
                displayAd(bestAd)
            }
            is IdleTimeout -> {
                displayMode.value = DisplayMode.STORE_INFO
            }
            is TouchInteraction -> {
                showInteractiveContent()
            }
        }
    }
    
    fun displayModes() = listOf(
        ADVERTISEMENT,      // 広告表示
        STORE_INFO,        // 店舗情報
        PRODUCT_CATALOG,   // 商品カタログ
        PAYMENT_WAITING    // 決済待機画面
    )
}
```

#### 4.1.2 視聴者検出（軽量版）
```kotlin
class TabletViewerDetection {
    private val cameraManager = CameraManager()
    private val mlKit = FaceDetection.getClient()
    
    suspend fun detectViewers(): ViewerInfo {
        // フロントカメラで顔検出（5fps）
        val faces = mlKit.process(cameraFrame)
        
        return ViewerInfo(
            count = faces.size,
            hasAttention = faces.any { it.headEulerAngleY < 15 },
            timestamp = System.currentTimeMillis()
        )
    }
}
```

#### 4.1.3 店舗支援機能
```kotlin
class StoreAssistantFeatures {
    // 在庫確認
    fun checkInventory(barcode: String): ProductInfo
    
    // スタッフ呼出
    fun callStaff(type: StaffType)
    
    // 商品検索
    fun searchProduct(query: String): List<Product>
    
    // 多言語対応
    fun translate(text: String, targetLang: Language): String
    
    // 電子レシート
    fun digitalReceipt(transactionId: String)
}
```

#### 4.1.4 オフライン対応
```kotlin
class OfflineManager {
    private val localCache = ContentCache(maxSize = 1.GB)
    
    fun preloadContent() {
        // 営業時間前に当日分を事前取得
        scope.launch {
            val todayContents = api.getTodayContents()
            localCache.store(todayContents)
        }
    }
    
    fun handleOffline() {
        // オフライン時は店舗情報と基本広告のみ
        displayMode = OfflineMode(
            contents = localCache.getEssentials(),
            syncInterval = 30.minutes
        )
    }
}
```

### 4.2 Phase2: MiniPC版機能（拡張）

#### 4.2.1 高度な視聴者分析
```python
class AdvancedViewerAnalytics:
    def __init__(self):
        self.model = load_openvino_model("yolov8n_optimized.xml")
        self.demographic_estimator = AgeGenderEstimator()
        self.attention_tracker = GazeTracker()
        
    def analyze_viewers(self, frame):
        # 20fps処理
        detections = self.model.infer(frame)
        
        analytics = {
            'count': len(detections),
            'demographics': self.estimate_demographics(detections),
            'attention_quality': self.measure_attention(detections),
            'dwell_time': self.calculate_dwell_time(detections),
            'emotional_response': self.estimate_emotion(detections)
        }
        
        return analytics
```

#### 4.2.2 リアルタイムコンテンツ最適化
```python
class RealtimeOptimizer:
    def __init__(self):
        self.performance_tracker = PerformanceTracker()
        self.ab_test_manager = ABTestManager()
        
    def optimize_content(self, viewer_response):
        # A/Bテスト自動実行
        if self.ab_test_manager.is_testing():
            variant = self.ab_test_manager.select_variant()
            self.performance_tracker.record(variant, viewer_response)
            
        # パフォーマンスに基づく自動最適化
        if viewer_response.attention_loss > 0.5:
            self.switch_to_more_engaging_content()
```

### 4.3 共通機能

#### 4.3.1 統合広告配信
```typescript
interface UnifiedAdDelivery {
  // デバイスタイプに応じた配信
  deliverAd(request: AdRequest): AdResponse {
    const device = request.deviceType;
    
    switch(device) {
      case 'tablet':
        return this.deliverTabletAd(request);
      case 'minipc':
        return this.deliverLargeScreenAd(request);
      default:
        return this.deliverDefaultAd(request);
    }
  }
  
  // 統一されたターゲティング
  targeting: {
    location: GeoRadius,
    demographics: AgeGender,
    deviceTypes: DeviceType[],
    businessTypes: BusinessCategory[],
    timeSlots: TimeSlot[]
  }
}
```

#### 4.3.2 収益計算エンジン
```typescript
class RevenueCalculator {
  calculatePayout(metrics: ViewerMetrics, device: DeviceInfo) {
    const baseRate = device.type === 'tablet' ? 0.3 : 0.5;
    const locationMultiplier = this.getLocationScore(device.location);
    const qualityBonus = this.getQualityBonus(metrics);
    
    const revenue = metrics.validViews * baseRate * locationMultiplier;
    const bonus = revenue * qualityBonus;
    
    return {
      base: revenue,
      bonus: bonus,
      total: revenue + bonus,
      ownerShare: (revenue + bonus) * 0.5
    };
  }
}
```

---

## 5. 非機能要件

### 5.1 性能要件

#### 5.1.1 タブレット版
```yaml
アプリ性能:
  起動時間: < 3秒
  画面切替: < 50ms
  メモリ使用: < 500MB
  バッテリー消費: < 10%/日
  
AI処理:
  顔検出: 5fps（省電力）
  推論時間: < 100ms
  精度: > 85%
  
通信:
  API応答: < 500ms
  オフライン耐性: 24時間
  データ使用量: < 1GB/月
```

#### 5.1.2 MiniPC版
```yaml
処理性能:
  人物検出: 20fps
  広告切替: < 10ms
  起動時間: < 30秒
  
キャッシュ性能:
  ヒット率: > 85%
  プリロード: 95%完了
  メモリ効率: > 80%
```

### 5.2 可用性要件

```yaml
共通要件:
  サービス可用性: 99.9%
  データ耐久性: 99.999999999%
  
タブレット版:
  アプリクラッシュ率: < 0.1%
  自動再起動: 実装必須
  MDM管理: 必須
  
MiniPC版:
  デバイス稼働率: 99.5%
  自動復旧機能: 必須
  リモート管理: 必須
```

### 5.3 互換性要件

```yaml
タブレット版:
  OS: Android 10以上
  RAM: 3GB以上
  ストレージ: 32GB以上
  画面: 10インチ以上
  カメラ: フロントカメラ必須
  
推奨デバイス:
  - Lenovo Tab M10 Plus
  - Samsung Galaxy Tab A8
  - Xiaomi Pad 5
```

---

## 6. データ設計

### 6.1 デバイス管理（統合版）

```sql
-- デバイス管理テーブル（更新）
CREATE TABLE devices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  device_id TEXT UNIQUE NOT NULL,
  device_type TEXT CHECK (device_type IN ('tablet', 'minipc')),
  owner_id UUID REFERENCES users(id),
  location GEOGRAPHY(POINT, 4326),
  business_type TEXT,
  business_name TEXT,
  
  -- タブレット固有
  android_version TEXT,
  app_version TEXT,
  battery_health INTEGER,
  
  -- MiniPC固有
  display_size TEXT,
  
  -- 共通
  status TEXT DEFAULT 'pending',
  activated_at TIMESTAMP,
  last_seen_at TIMESTAMP DEFAULT NOW(),
  settings JSONB DEFAULT '{}',
  performance_score FLOAT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 店舗支援機能（タブレット向け）
CREATE TABLE store_features (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  device_id UUID REFERENCES devices(id),
  feature_type TEXT,
  configuration JSONB,
  usage_count INTEGER DEFAULT 0,
  last_used TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 7. ハードウェア仕様

### 7.1 Phase1: タブレット仕様

```yaml
推奨スペック:
  CPU: Snapdragon 680以上
  RAM: 4GB以上（3GB最小）
  ストレージ: 64GB以上（32GB最小）
  画面: 10.1インチ以上、1920×1200以上
  カメラ: 5MP以上フロントカメラ
  通信: Wi-Fi 5以上
  OS: Android 10以上
  
アクセサリ:
  保護ケース: 耐衝撃・スタンド付き
  セキュリティ: 盗難防止ワイヤー
  充電: USB-C PD対応
  
設置要件:
  高さ: 100-150cm（視線の高さ）
  角度: 15-30度（見やすい角度）
  電源: 常時給電推奨
```

### 7.2 Phase2: MiniPC仕様

```yaml
本体:
  CPU: Intel N100 (4C/4T, 3.4GHz)
  RAM: 8GB LPDDR5
  Storage: 128GB NVMe SSD
  OS: Ubuntu 22.04 LTS
  
接続:
  Display: HDMI 2.0
  Camera: USB 3.0
  Network: Wi-Fi 6 + Ethernet
  IoT SIM: Soracom
```

---

## 8. セキュリティ要件

### 8.1 タブレット版セキュリティ

```yaml
アプリセキュリティ:
  - コード難読化（ProGuard/R8）
  - 証明書ピンニング
  - Root検出
  - 改ざん検知
  
デバイス管理:
  - MDM必須
  - リモートワイプ
  - アプリ制限
  - キオスクモード
  
データ保護:
  - ローカル暗号化
  - Secure Storage使用
  - 個人情報非保持
```

### 8.2 共通セキュリティ

```yaml
通信:
  - TLS 1.3必須
  - API認証（JWT）
  - Rate Limiting
  
プライバシー:
  - カメラ画像非保存
  - 匿名化処理
  - GDPR/個情法準拠
```

---

## 9. 運用要件

### 9.1 タブレット運用

```yaml
導入プロセス:
  Day 1: 申込・審査
  Day 2: アプリ設定ガイド送信
  Day 3: 店舗でアプリインストール
  Day 4: リモート動作確認
  Day 5: 広告配信開始
  
日常運用:
  - 自動アップデート（深夜）
  - バッテリー監視
  - 使用状況レポート
  - リモートサポート
  
トラブル対応:
  - アプリ再起動ガイド
  - FAQ・チャットボット
  - 電話サポート（平日）
```

### 9.2 統合監視

```yaml
ダッシュボード:
  - デバイス稼働状況
  - タイプ別収益
  - エラー率
  - 地域別分布
  
アラート:
  - オフライン30分
  - バッテリー低下
  - 異常な視聴数
  - アプリクラッシュ
```

---

## 10. 開発計画

### 10.1 Phase1: タブレット版（0-6ヶ月）

```yaml
Month 1-2: 基礎開発
  Week 1-2: 
    - 環境構築・設計
    - Supabase/API設計
  Week 3-4:
    - UIプロトタイプ
    - 基本画面実装
  Week 5-6:
    - 広告表示機能
    - ローカルDB実装
  Week 7-8:
    - AI統合（TensorFlow Lite）
    - パフォーマンス最適化

Month 3-4: 店舗機能
  Week 9-10:
    - 店舗支援機能
    - 多言語対応
  Week 11-12:
    - オフライン対応
    - セキュリティ実装
  Week 13-14:
    - MDM統合
    - β版テスト準備
  Week 15-16:
    - 10店舗パイロット
    - フィードバック反映

Month 5-6: 本格展開
  Week 17-18:
    - 100店舗展開
    - 運用体制構築
  Week 19-20:
    - マーケティング開始
    - 代理店開拓
  Week 21-22:
    - 機能改善
    - スケール準備
  Week 23-24:
    - 1,000台達成
    - Phase2準備
```

### 10.2 Phase2: MiniPC版（6-12ヶ月）

```yaml
Month 7-8: MiniPC開発
  - EdgeKit設計
  - Ubuntu環境構築
  - 高度なAI実装
  - ハードウェア最適化

Month 9-10: 統合
  - 統一API
  - 共通ダッシュボード
  - クロスデバイス広告
  - 収益最適化

Month 11-12: 拡大
  - 大型店舗開拓
  - 全国展開
  - 追加機能開発
  - 海外展開準備
```

### 10.3 開発体制

```yaml
Phase1体制:
  - PM: 1名
  - Androidエンジニア: 2名
  - バックエンド: 1名
  - UI/UXデザイナー: 1名
  - QA: 1名
  
Phase2追加:
  - 組込みエンジニア: 1名
  - AIエンジニア: 1名
  
外部リソース:
  - MDMサポート
  - カスタマーサポート
```

### 10.4 投資計画

```yaml
Phase1（6ヶ月）:
  開発費: ¥10,000,000
  タブレット費: ¥15,000,000（1,000台）
  マーケティング: ¥3,000,000
  運営費: ¥2,000,000
  合計: ¥30,000,000

Phase2（6ヶ月）:
  開発費: ¥5,000,000
  ハードウェア: ¥12,500,000（500台）
  運営費: ¥3,000,000
  合計: ¥20,500,000

収益予測（12ヶ月後）:
  タブレット1,000台: ¥3,000,000/月
  MiniPC500台: ¥2,500,000/月
  合計売上: ¥5,500,000/月
  営業利益率: 30%
```

---

## 11. リスク管理

### 11.1 Phase1リスク

| リスク | 影響度 | 対策 |
|--------|--------|------|
| アプリ不具合 | 高 | 段階的リリース・自動テスト |
| タブレット故障 | 中 | 保険加入・交換体制 |
| 店舗離脱 | 高 | 最低契約期間・インセンティブ |
| 低収益 | 高 | 付加価値サービス追加 |

### 11.2 Phase2リスク

| リスク | 影響度 | 対策 |
|--------|--------|------|
| 開発遅延 | 中 | アジャイル開発・MVP優先 |
| 在庫リスク | 高 | 受注生産・需要予測 |
| 競合参入 | 中 | 特許出願・差別化 |

---

## 12. 成功指標（KPI）

### 12.1 Phase1目標（6ヶ月）

```yaml
事業KPI:
  - アクティブタブレット: 1,000台
  - 月間広告収入: ¥3,000,000
  - 店舗継続率: 90%以上
  - アプリ評価: 4.0以上

技術KPI:
  - アプリクラッシュ率: < 0.1%
  - 広告切替時間: < 50ms
  - オフライン耐性: 24時間
  - 顔検出精度: > 85%
```

### 12.2 Phase2目標（12ヶ月）

```yaml
事業KPI:
  - 総デバイス数: 1,500台
  - 月間広告収入: ¥5,500,000
  - 営業利益率: 30%
  - NPS: 50以上

技術KPI:
  - システム稼働率: 99.9%
  - 統合管理実現
  - キャッシュヒット率: 85%
  - API応答: < 100ms
```

---

## 13. 付録

### 13.1 用語集

| 用語 | 定義 |
|------|------|
| MDM | Mobile Device Management（モバイルデバイス管理） |
| キオスクモード | 特定アプリのみ動作する制限モード |
| 実視認インプレッション | 2秒以上実際に視聴された広告表示 |
| エッジAI | デバイス上で実行されるAI処理 |

### 13.2 参考資料

- [Androidアプリ開発ガイド](./android-development.md)
- [MDM導入マニュアル](./mdm-setup.md)
- [店舗オンボーディング](./store-onboarding.md)
- [MiniPC移行ガイド](./minipc-migration.md)

### 13.3 改訂履歴

| バージョン | 日付 | 変更内容 | 承認者 |
|-----------|------|----------|--------|
| 1.0 | 2024-01-15 | 初版作成 | - |
| 2.0 | 2024-01-20 | 添付資料反映 | - |
| 3.0 | 2024-01-20 | リアルタイム切替最適化 | - |
| 4.0 | 2024-01-21 | タブレット優先展開に変更 | - |

---

**本要件定義書は、ステークホルダー全員の合意を得て正式版とする。**

## 主要な変更点（v3.0→v4.0）

1. **展開戦略の変更**
   - Phase1: Androidタブレット（0-6ヶ月）
   - Phase2: MiniPC追加（6-12ヶ月）
   - 段階的な拡大でリスク最小化

2. **タブレット版の詳細仕様**
   - Kotlin + Jetpack Compose
   - 店舗支援機能の統合
   - 省電力AI処理（5fps）
   - MDMによる一元管理

3. **ビジネスモデルの最適化**
   - 初期費用を抑制（¥15,000/台）
   - 複合収益モデル（広告＋機能）
   - 保証金を¥5,000に削減

4. **開発計画の具体化**
   - 6ヶ月で1,000台展開
   - 10店舗パイロットから開始
   - アジャイル開発でリスク軽減

5. **運用の簡素化**
   - アプリベースで導入容易
   - リモートサポート中心
   - 自動アップデート対応