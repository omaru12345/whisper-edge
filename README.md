# whisper-edge

ブラウザ完結（WebGPU / WebAssembly）で動くリアルタイム超軽量音声認識ライブラリ。
タブ音声を **クラウドに一切送らずに** 文字起こしできる JS SDK と、それを使った Google Meet 議事録 Chrome 拡張のリファレンス実装。

---

## 🎯 Goal: Googleに買収されること

このプロジェクトは「Google による Jetpac 型 acqui-hire（2〜5人 / 技術コア + UX デモ）」をエグジット目標とする。

**買収対象は SDK（コア技術）であり、Chrome 拡張は配布チャネル兼ショーケース。**

| 想定買収先 | 統合シナリオ |
|---|---|
| **Google Meet チーム** | サーバー側の文字起こし負荷をゼロにし、エンタープライズ顧客のセキュリティ懸念を解消する "ローカル文字起こしモード" |
| **Chrome チーム** | 任意のタブ音声に対するブラウザ標準字幕 API（Live Caption の Web 版）|
| **Workspace** | Docs/Tasks への自動アクション抽出、Driveへの議事録自動保存 |

### 評価される技術の堀（moat）
1. **モデルサイズ**（Whisper を 50MB 以下に量子化＋蒸留）
2. **レイテンシ**（300ms 以内のストリーミング応答）
3. **ブラウザでの安定動作**（WebGPU フォールバック → WASM SIMD）
4. **多言語**（日本語の精度で勝つ）

---

## 技術スタック

| レイヤー | 技術 |
|---|---|
| モデル | Whisper / Distil-Whisper を ONNX 化 → INT8 量子化 |
| 推論ランタイム | onnxruntime-web（WebGPU バックエンド優先 / WASM SIMD フォールバック） |
| SDK | TypeScript（npm: `whisper-edge`） |
| デモ拡張 | Chrome Manifest V3 + WebAudio API（タブキャプチャ） |
| ベンチ | Playwright + 公開音声データセット（Common Voice 日本語など） |

---

## ディレクトリ構成

```
whisper-edge/
├── src/                    # コア SDK（TypeScript）
├── examples/
│   └── meet-extension/     # Google Meet 用 Chrome 拡張デモ
├── bench/                  # レイテンシ・精度ベンチ
└── docs/                   # アーキテクチャ / 買収ピッチ素材
```

---

## 3 フェーズ計画

### Phase 1（〜1か月）: 技術 PoC
- [ ] WebGPU 上で Whisper-tiny を 1 文字起こしできる
- [ ] 日本語 5 分音声で WER < 20%
- [ ] レイテンシ計測スクリプト

### Phase 2（〜3か月）: SDK 化
- [ ] `whisper-edge` npm パッケージ公開
- [ ] ストリーミング API（30秒チャンク → 部分結果）
- [ ] Distil-Whisper への切替で 50MB 以下達成

### Phase 3（〜6か月）: ショーケース & 露出
- [ ] Meet 議事録 Chrome 拡張公開（Chrome Web Store）
- [ ] HackerNews / Product Hunt 露出
- [ ] Google DeepMind / Workspace の関係者にデモ送付

---

## 開発コマンド

```bash
npm install
npm run dev            # SDK + デモ拡張のホットリロード
npm test
npm run bench          # レイテンシ・WER 計測
npm run build:ext      # Chrome 拡張の zip 生成
```

（実装は Phase 1 着手時に追加）
