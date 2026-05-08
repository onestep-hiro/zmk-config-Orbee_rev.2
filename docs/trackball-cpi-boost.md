# トラックボール CPI ブースト機能 調査メモ

## 目的

ボタンを押下している間、トラックボールのCPIを通常の2倍に変更できるボタンを追加したい。

## 現状

- 通常CPI: `CONFIG_PMW3610_CPI=1200`（2026-05-09に400→1200へ変更済み）
- ZMKバージョン: v0.3
- ドライバ: `MeowCatpawkittie/zmk-pmw3610-driver-alt @ main`

## 調査結果

### ZMK v0.3での制約

- キーを押している間だけCPIを変更する標準ビヘイビアは存在しない
- X軸・Y軸を個別にスケーリングする機能（`input-processors` 等）も未サポート

### 実現可能な方法: Snipeレイヤー方式

PMW3610ドライバが持つ「Snipeモード」を利用する。  
**特定のレイヤーがアクティブな間、センサーのCPIを自動で別の値に切り替える**機能。

| 状態 | CPI | 用途 |
|------|-----|------|
| 通常 | 1200 | 普段の操作 |
| ボタン押下中（Snipeレイヤー） | 2400 | 素早い移動が必要なとき |

## 実装方針

### 変更が必要なファイル（3箇所）

**① `config/boards/shields/Orbee/Orbee_R.conf`**

コメントアウトを外してSnipe CPIを有効化：

```conf
CONFIG_PMW3610_SNIPE_CPI=2400
CONFIG_PMW3610_SNIPE_CPI_DIVIDOR=1
```

**② `config/boards/shields/Orbee/Orbee_R.overlay`**

`trackball` ノードにSnipeレイヤー番号を追加：

```devicetree
trackball: trackball@0 {
    ...
    automouse-layer = <6>;
    scroll-layers = <5>;
    snipe-layers = <7>;  // 未使用のレイヤー番号を割り当てる
};
```

**③ `config/Orbee.keymap`**

Snipeレイヤー（例: レイヤー7）を定義し、任意のキーに `&mo 7` を割り当てる：

```keymap
// 好きなキーに割り当て（押している間だけSnipeモード）
&mo 7
```

## 未決事項

- [ ] どのキーに割り当てるか（物理的な配置を検討中）
- [ ] Snipeレイヤーの番号（`config/Orbee.keymap` の内容確認後に確定）
- [ ] Snipeレイヤー自体のキーマップ定義（通常は他のキーはトランスペアレント `&trans`）
