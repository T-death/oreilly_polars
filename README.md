# Python Polars: The Definitive Guide
引用元：https://github.com/jeroenjanssens/python-polars-the-definitive-guide

# 第1章（Fisrt Step）
* 通常のpolarsであれば`pip install polars`で良いが、データが大きい場合（=42億行以上が目安らしい）は`pip install polars-u64-idx`をインストールする
    * 通常のpolarsでは32ビットで扱うため、データが大きすぎるとバグるみたい
    * かと言って不必要に`polars-u64-idx`を使用すると、かえってパフォーマンスが低下する可能性がある点に注意
* `with pl.Config() as cfg:`のように操作すると、polarsの設定を特定のスコープ内だけで変更可能
# 第2章（Moving from Pandas to Polars）
* pandasとpolarsの共通点
    * DataFrameがメインのデータ構造
        * Series型もある
    * プリミティブ型やdatetimeといったデータ型をサポートしている
* polarsの命名由来
    * パンダ < ホッキョクグマという序列だから
    * Rust製であることにちなんで、Pola`rs`とRustファイルと同じ拡張子になるから
# 第3章（Data Types and Data Structures）
* Arrowデータ型
    * このデータ型をベースに構築されている
    * 通常であればそれぞれの**行**に対して異なるデータ型のカラム情報を保持しているが、Arrowデータ型によるメモリの使い方だと**列**ごとにデータを持つ
        * 列ごとにデータを持つことで、索引の際に不必要なメモリへのアクセスを抑制できる
        * 列ごとにデータを持つことで、実行時間がO(1)になる
        * この点でpolarsは大規模なデータセットに対して、pandasよりも有用性が高いってことなのか
    * Arrowデータ型は、様々なプログラミング言語で対応しているので、異なる言語間でデータをデシリアル化→シリアル化をする必要がない
        * このデシリアル化→シリアル化には時間がかかるので、これが削れる点でもpolarsはより優れている
        * このデータセットの共有のことを`プロセス間通信（=IPC）`と言うらしい
    * polarsではpandasでサポートされていた`object`型がなく、string型しかないため、polarsで最適化を実施しても`object`型のデータには適用されない
    * DataFrame型はSeries型のかたまりで、Series型は`ChukedArray`型のかたまり
        * `ChunkedArray`型は、メモリ管理の最適化などのいくつかの最適化が適用できる
    * `ChunkedArray`型データを追加すると既存のものに追加されるため、内部的にコピーされることなく、ガベージコレクションも不要になる
        * つまりメモリ領域も圧迫しないし、処理時間の節約にもなる
    * polarsではデータをチャンク分割できるため、並列処理で効率化が図れる
        * pandasでも並列処理は可能だが、確か`pandara.lel`というライブラリと組み合わせる必要がある
    * `LazyFrame`は即座に処理を実行するわけではなく、`lf.collect()`によって初めて実行される
        * `LazyFrame`の宣言時は、あくまで計算処理を持ったクエリってだけ
# 第4章（Eager and Lazy APIs）
* Eager API
    * 即時実行モデルを使用するため、リアルタイムでデータが操作される
    * pandasと同じ挙動
* Lazy API
    * 遅延実行モデルを使用するため、実行時に最適化を適用することができる
* 機能の違い
    * LazyFrameはあくまで遅延実行されるだけで、実行されたらDataFrameに相当する処理が扱える（大事！）
    * 集計
        * DataFrameでは列ごとの集計だけでなく、行ごとの集計も実行できるモジュールがある
        * LazyFrameでは列ごとの集計モジュールしかない
            * これは、Arrowデータ型によって列で管理する仕組みだからかな？
    * 属性
        * LazyFrameには`.shape()`モジュールが備わっていない
            * データが使用可能になったタイミング（=`collect()`後？）に使えるようになる
    * 説明性
        * DataFrameは、`.describe()`や`.is_unique()`といったpandasの機能に相当するモジュールがある
        * LazyFrameには`.explain()`しかない
    * DataFrame to LazyFrame -> `.lazy()`
    * LazyFrame to DataFrame -> `.collect()`
# 第5章（Reading and Writing Data）
# 第6章（Beginning Expressions）
* `pl.Expr`には様々なメソッドが用意されていて、`Expr.str / Expr.dt / Expr.cat`のように文字列 / 時間 / カテゴリといった分野の操作が可能
# 第7章（Continuing Expressions）
* 量が多過ぎるので、一旦スキップ
    * ページの真ん中らへんの`ローリング統計`の手前までやった
# 第8章（Combining Expressions）
* インライン演算子（=普通の計算式）とメソッドで基本的には対応しているが、ドット積に関してはメソッドにしかない
# 第9章（Selecting and Creating Columns）
* Polarsでの式は並列に実行されるため、式の中で作成した列を同時の式で参照できない
    * 例えば、1つの式の中で`sample / sample2`という2つの式を作成中に、sample2の計算式でsampleのデータを参照できない
    * `.with_columns()`で言えば、これを複数回に分けて処理することで、順序よく処理が走るため参照可能になる
# 第10章（Filtering and Sorting Rows）
# 第11章（Working with Special Data Types）
* 文字列は可変長である点が難点
    * 整数のように固定長であれば、次の整数を事前にメモリアドレスに追加できるが、可変である場合はメモリアドレスの予測ができない
    * 基本的には16バイトで文字列の様々な情報を保持するが、12バイト以下の文字列に対しては最適化が可能
        * 16バイトの文字列であればデータバッファに入れ、12バイト以下であればビューレイアウトに格納される（=インライン化）
        * このあたり難しいな..あとで詰める
* カテゴリカルな値
    * int型の場合は物理表現（=`physical representaton`）、string型の場合は語彙表現（=`lexical representation`）と呼ぶ
    * カテゴリカルな値が文字列で表現されている場合は、効率的にエンコードされる（=int型に畳む）


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
