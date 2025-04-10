# Kibo-RPC 第6回大会 設計書

## 1. 概要

このドキュメントは、「きぼう」ロボットプログラミング競技会（Kibo-RPC）第6回大会に向けたAstrobeeロボット制御プログラムの設計と実装方針について説明するものである。本プロジェクトでは、ISS内の「きぼう」モジュールに設置された隠し財宝（Treasure Item）を効率的に探索し、正確に識別するためのシステムを開発する。

## 2. システム構成

### 2.1 全体アーキテクチャ

プログラムは以下の主要コンポーネントで構成される：

1. **メインコントロールモジュール**
   - Astrobeeロボットの状態管理
   - 各モジュール間の連携制御
   - タスク実行の管理と監視

2. **経路計画モジュール**
   - 効率的な経路の生成と最適化
   - オアシスゾーンを考慮した経路計画
   - 障害物回避と安全な移動制御

3. **画像処理・認識モジュール**
   - Lost Itemの検出と識別
   - アイテム数のカウント
   - ARタグからの位置推定

4. **データ管理モジュール**
   - 認識したアイテムの情報管理
   - エリアとアイテムの紐付け
   - 収集データの一時保存と解析

### 2.2 データフロー

```
[Astrobee センサー] → [画像取得] → [画像処理・認識] → [データ分析]
         ↑                                      ↓
         ↑                                      ↓
[移動制御] ← [経路計画] ← [目標位置決定] ← [データ管理]
```

## 3. 実装戦略

### 3.1 ロボット制御と経路計画

#### 3.1.1 基本的な移動戦略
- Area間の最短経路計算
- オアシスゾーンを優先的に通過する経路の生成
- 最適な移動速度とスムーズな加速・減速制御

#### 3.1.2 オアシスゾーン活用戦略
- オアシスゾーン内滞在時間と得点の関係分析 → 最速でクリアした方が得点が高い? or オアシスゾーン滞在時間が長いほど得点が高い?
- 時間制限内でのオアシスゾーン滞在時間の最適化
- 複数オアシスゾーンの効率的な経由計画

#### 3.1.3 位置把握と姿勢制御
- ARタグからの位置・姿勢推定
- 画像認識のための最適な姿勢制御
- 移動中の安定性確保

### 3.2 画像処理と認識

#### 3.2.1 ディープラーニングによる画像認識
- YOLOv8/TensorFlowによるLost Item検出モデルの実装
- トレーニングデータセットの作成と拡張
- モデルの最適化と軽量化

#### 3.2.2 OpenCVを用いた画像処理
- 画像前処理（ノイズ除去、コントラスト調整等）
- アイテム数のカウントアルゴリズム(yolo等のモデルのみで正確にカウント可能なら不要)
- 画像の切り抜きと保存機能

#### 3.2.3 認識結果の信頼性向上
- 複数フレームの結果統合による精度向上
- 距離・角度による認識結果の補正
- 失敗時のリカバリー手法

### 3.3 データ管理とレポート機能

#### 3.3.1 Area-Item紐付け管理
- 認識したItemとAreaの対応関係の記録
- Landmark ItemとTreasure Itemの区別と管理
- 信頼度スコアによる認識結果の評価

#### 3.3.2 Target Item探索機能
- 宇宙飛行士からの情報に基づくTarget Item特定
- Target Itemの位置へのナビゲーション
- 正確な撮影と報告機能

## 4. 技術的課題と対策

### 4.1 認識精度向上のための対策
- 多様な光条件に対応するためのデータ拡張
- ARタグ認識の安定化
- 複雑背景からのItem抽出精度向上

### 4.2 経路最適化の課題
- 時間効率とオアシスゾーン滞在のトレードオフ解析
- 異なる得点計算方式に対応する柔軟な戦略
- 予期せぬ障害物や状況変化への対応

### 4.3 ロバスト性向上策
- 自己位置推定の誤差対策
- カメラ視野角外のItem検出手法
- 画像認識失敗時のフォールバック処理

## 5. テスト戦略

### 5.1 単体テスト
- 画像認識モジュールの精度評価 → androidでさまざまなテストデータをシミュレーション
- 経路計画アルゴリズムの効率性検証
- ARタグからの位置推定精度評価

### 5.2 統合テスト
- エンドツーエンドの動作確認
- 全体フローの時間計測と最適化
- エラー発生時の回復機能確認

### 5.3 シミュレーション環境
- Webシミュレーションの効率的な活用方法
- ローカルシミュレーション環境の構築
- 実際の競技環境に近い条件での検証

## 6. 開発スケジュールと役割分担

### 6.1 開発フェーズ
1. **基本機能実装フェーズ** (〜4月30日)
   - 基本的な移動制御
   - 画像認識の基礎機能
   - データ管理の基本構造
   - 全機能の統合 → シミュレーションでミッション完了を目指す

2. **機能拡張フェーズ** (〜5月14日)
   - 経路最適化の実装
   - 認識精度の向上
   - オアシスゾーン活用戦略の実装

3. **最適化フェーズ** (〜6月20日)
   - パフォーマンス最適化
   - 徹底したテストと調整(シミュレーション自動化)
   - エラー対策

### 6.2 チーム構成と役割
- **画像認識チーム**：画像処理、機械学習モデル開発、認識精度向上
- **経路計画チーム**：移動制御、経路最適化、オアシスゾーン戦略

- **統合・検証チーム**：全体統合、テスト環境構築、パフォーマンス評価

## 7. まとめ

本プロジェクトでは、AIを活用した高精度な画像認識と効率的な経路計画を組み合わせることで、Kibo-RPCの課題に対する最適なソリューションを目指す。開発プロセスでは、機能ごとの単体テストと継続的な改善を重視し、シミュレーション環境を最大限に活用することで、本番環境での高いパフォーマンスを実現する。

チーム間の密な連携と、**1pagerによる情報共有**を通じて、統合的なシステム開発を進めていく。