# Python Polars: The Definitive Guide

# 第1章（Fisrt Step）
* 通常のpolarsであれば`pip install polars`で良いが、データが大きい場合（=42億行以上が目安らしい）は`pip install polars-u64-idx`をインストールする
    * 通常のpolarsでは32ビットで扱うため、データが大きすぎるとバグるみたい
    * かと言って不必要に`polars-u64-idx`を使用すると、かえってパフォーマンスが低下する可能性がある点に注意
* `with pl.Config() as cfg:`のように操作すると、polarsの設定を特定のスコープ内だけで変更可能


# 環境構築
* `rye init`
    * 初期化
* `rye pin [pythonバージョン]`
    * プロジェクト配下で使用するpythonバージョンを指定
    * `rye sync`を実行して反映させる
* `rye sync`
    * 同期させる
* `rye add [パッケージ名]`
    * パッケージのインストール
    * ただし、rye addしただけでは`pyproject.toml`に追加されるだけで反映されない
    * rye addが完了次第、忘れずに`rye sync`を行うことで追加される
* `rye list`
    * rye経由でインストールしたパッケージ一覧を確認できる
* `rye run jupyter-lab`
    * jupyter-labの起動コマンド
