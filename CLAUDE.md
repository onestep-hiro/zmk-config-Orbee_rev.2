# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

このリポジトリは、分割ワイヤレスキーボード **Orbee rev.2** の ZMK ファームウェア設定です。マイコンは **Seeeduino XIAO BLE**（nRF52840）を使用。右側ボードが Central（ホスト接続）、左側が Peripheral として動作します。

## ビルド方法

ローカルビルド環境は不要。**GitHub Actions** によってビルドが実行されます。

- `main` ブランチへの push / PR 作成で自動ビルドが走る
- Actions タブ → "Build ZMK firmware" → アーティファクトからダウンロード
- `build.yaml` でビルドターゲットを定義（`Orbee_R rgbled_adapter`, `Orbee_L rgbled_adapter`, `settings_reset`）

キーマップの GUI 編集は [Keymap-Editor](https://nickcoutsos.github.io/keymap-editor/) を使用。

## ファームウェア書き込み手順

1. ボードのリセットボタンを 2 回素早く押す → ブートローダー起動（"XIAO SENSE" として認識）
2. `settings_reset-seeeduino_xiao_ble-zmk.uf2` を書き込む（右→左の順）
3. 続けて `Orbee_R/L rgbled_adapter-seeeduino_xiao_ble-zmk.uf2` を書き込む
4. 更新後は PC 側でペアリング情報を削除して再ペアリング

## アーキテクチャ

### ハードウェア構成

| 項目 | 内容 |
|------|------|
| マイコン | Seeeduino XIAO BLE (nRF52840) |
| 分割方式 | 左右 BLE 分割（右=Central, 左=Peripheral） |
| キーマトリクス | 4行×11列（diode: col2row） |
| トラックボール | 右側: PMW3610（SPI接続） |
| エンコーダ | 左右各1個（alps EC11） |
| LED | RGB LED ウィジェット（caksoylar/zmk-rgbled-widget v0.3） |
| タッチ入力 | 左側: GPIO タッチ入力 x3 |

### ファイル構成

```
config/
  west.yml              # ZMK 依存関係（zmk v0.3、pmw3610ドライバ、rgbled-widget）
  Orbee.json            # Keymap-Editor 用レイアウト定義
  boards/shields/Orbee/
    Orbee.dtsi          # 共通ハードウェア定義（マトリクス、エンコーダ、センサー）
    Orbee_L.overlay     # 左側固有設定（col-gpios、タッチ入力）
    Orbee_R.overlay     # 右側固有設定（col-gpios、PMW3610 SPI、col-offset=6）
    Orbee_L.conf        # 左側 Kconfig（EC11、バッテリー監視）
    Orbee_R.conf        # 右側 Kconfig（PMW3610設定、ポインティング、LED層表示）
    Kconfig.defconfig   # 分割ロール設定（右=Central）
    Kconfig.shield      # シールド検出
    Orbee.zmk.yml       # シールドメタデータ
build.yaml              # GitHub Actions ビルドマトリクス
```

### キーマップ

`.keymap` ファイルはリポジトリに含まれていません。Keymap-Editor が GitHub リポジトリに直接コミットする方式で管理されます（`config/Orbee.keymap` として生成される）。

### ZMK 依存ライブラリ（west.yml）

- `zmkfirmware/zmk` @ v0.3
- `MeowCatpawkittie/zmk-pmw3610-driver-alt` @ main（カスタム PMW3610 ドライバ）
- `caksoylar/zmk-rgbled-widget` @ v0.3

### PMW3610 トラックボール設定（Orbee_R.conf）

- CPI: 400、orientation: 90度回転、X軸反転
- automouse-layer: 6、scroll-layer: 5（Orbee_R.overlay で定義）
- スマートアルゴリズム有効、移動閾値: 5

### LED ウィジェット

- バッテリー高: 30%、クリティカル: 10%
- 右側のみ `CONFIG_RGBLED_WIDGET_SHOW_LAYER_COLORS=y` でレイヤー色表示

## 設定変更のポイント

- **PMW3610 の感度調整**: `Orbee_R.conf` の `CONFIG_PMW3610_CPI` を変更
- **automouse/scroll レイヤー変更**: `Orbee_R.overlay` の `automouse-layer` / `scroll-layers` を変更
- **新しいビルドターゲット追加**: `build.yaml` の `include` に追記
- **ZMK バージョン更新**: `config/west.yml` の `revision` を変更
