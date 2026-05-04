# CLAUDE.md — whisper-edge

## プロジェクト概要

ブラウザ完結のリアルタイム音声認識 SDK。**コア技術 = 軽量モデル + WebGPU/WASM 推論**。
Chrome 拡張はデモ・配布チャネルにすぎない。**買収対象は SDK 側**である点を常に意識する。

ゴール: Google による acqui-hire。詳細は README 参照。

## 設計原則

- **クラウド依存ゼロ**: 音声を外部に送る選択肢は実装しない（"完全ローカル" が買収理由）
- **ブラウザ第一**: Node.js 専用 API は使わない
- **モデルは小さく、UX は磨く**: 50MB 以上のモデルは原則使わない
- **依存ライブラリは最小**: コアは `onnxruntime-web` のみで動くこと

## ディレクトリ構成

`src/` がコア SDK、`examples/meet-extension/` がデモ拡張、`bench/` が性能計測。
詳細は README の「ディレクトリ構成」を参照。

## ブランチ命名

```
feature-{name}-{description}
bug-{name}-{description}
refactoring-{name}-{description}
```

メインブランチは `main`、PR は `main` に向ける。

## コミット前チェック

- `npm test` を通す
- ブラウザでの動作確認（Chrome 最新 + Safari TP）
- バンドルサイズが膨らんでいないか確認

## やらないこと

- バックエンドサーバーの実装（このプロジェクトの本質に反する）
- 多機能化（議事要約・話者分離など）→ 文字起こしの精度・速度・サイズに集中
- 独自の Foundation Model 学習（Whisper / Distil-Whisper の蒸留・量子化に徹する）
