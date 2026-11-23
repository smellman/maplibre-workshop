# MapLibre と Planetiler で自分だけのベースマップを作ろう

[MapLibre](https://maplibre.org/) が主催するワークショップへようこそ！楽しみながら、役に立つことを学んでいただければ幸いです。

## ゴール

1. [Planetiler](https://github.com/onthegomap/planetiler) で独自のベクトルタイルを作成する。
2. 作成したベクトルタイルを [Martin](https://martin.maplibre.org/) で配信する。
3. [Maputnik](https://github.com/maplibre/maputnik) で MapLibre スタイルの作り方を学ぶ。
4. [MapLibre GL JS](https://github.com/maplibre/maplibre-gl-js) を使って Web ページに地図を組み込む。

## 開発環境のセットアップ

ワークショップの限られた時間を有効に使うため、開発環境には [GitHub Codespaces](https://github.com/features/codespaces) を使います。

[ワークショップのリポジトリ](https://github.com/maplibre/workshop) を開き、右上の *Code* をクリックしてから *Codespaces* タブに移動し、右側のボタンをクリックして新しい codespace を起動してください。

<img src="assets/codespaces.png" width="50%"  alt="codespaces"/>

あらかじめ用意した Docker イメージのダウンロードが終わると、シェルが開きます。開かない場合は、ターミナルウィンドウの上にある小さな「+」ボタンをクリックして、新しいシェルを作成してください。

## 1. タイルの生成

[slice.openstreetmap.us](https://slice.openstreetmap.us/) で作成した OSM の抽出データは、すでにダウンロード済みです。

次のコマンドを実行してください。

```
java -jar /planetiler.jar --download_dir=/data/sources --minzoom=0 --maxzoom=14 --osm_path=/data/sources/workshop.osm.pbf --tmpdir=/data/tmp  --output=/data/workshop.mbtiles
```

#### 期待される結果
Planetiler によって新しいファイル `/data/workshop.mbtiles` が生成されます。`ls /data/workshop.mbtiles` で確認できます。

## 2. タイルの配信

前のステップで生成した MBTiles ファイルは、タイルサーバーで配信できます。このワークショップでは、開発コンテナにプリインストールされている Martin を使います。

次のコマンドを実行してください。

```
martin /data/workshop.mbtiles
```

Martin はポート 3000 で起動します。このポートを公開するかどうか尋ねられるので、インターネットに公開してください。

VS Code の「ポート」タブを開くと、Martin インスタンスの公開 URL を確認できます。

![image](https://github.com/user-attachments/assets/c5bcf8c4-416d-49b1-b0be-2ca04ea6267b)

表示範囲（Visibility）が「Public」になっていることを確認してください。右クリックで変更できます。

表示された URL を開くと、次のメッセージが表示されるはずです。

```
Martin server is running.


    The WebUI feature can be enabled with the --webui enable-for-all CLI flag or in the config file, making it available to all users.


    A list of all available sources is at /catalog

See documentation https://github.com/maplibre/martin
```

URL の末尾に `/catalog` を付けて、カタログのエンドポイントを確認してみましょう。`workshop` ソースが利用可能になっているはずです。

また、`/workshop` の URL も確認してください。これは、そのパスで配信されているタイルの情報を記述した TileJSON エンドポイントです。この URL は次のステップで使うので控えておいてください。

### Web UI

`ctrl-c` でサーバーを停止し、`--webui enable-for-all` フラグを付けて再起動します。

```shell
martin /data/workshop.mbtiles --webui enable-for-all
```

Web UI、特にカタログを見てみましょう。

## 3. 地図のスタイリング

* [Maputnik](https://maplibre.org/maputnik) を開きます。
* 「Open」をクリックして `OSM Liberty` スタイルを開きます。
* まず上部バーの検索ボックスでワークショップの開催地を検索し、地図をそこへ移動しておきます。こうしておくと、地図がタイルの収録範囲を表示した状態になります。次のステップで自分のソースに差し替えた瞬間に、その場で地図が変わる様子を確認できます。
* データソースエディタで、既存のアクティブなソース `#openmaptiles` を**変更**します。前のステップで確認した、Martin インスタンスの `/workshop` TileJSON エンドポイントの URL を使います。このリンクは、データソースエディタの `inspect` ボタンをクリックし、`TileJSON URL` の横にある `Copy link` をクリックすると取得できます。`https://{Martin インスタンスの公開 URL}/workshop` のような形式になっているはずです。

![edit datasource](assets/datasource.png)

* `Sources` の横にある `x` ボタンをクリックして、データソースエディタを閉じます。

* 使用した OSM 抽出データの都合で、詳細なタイルがあるのは狭い範囲だけです。すでに地図をワークショップの開催地に移動してあるので、データはすぐに表示されるはずです。その範囲から大きく離れたり、ズームアウトしすぎたりすると、地図は空っぽに見えます。全世界のベクトルタイルを生成するには、GitHub Codespaces が提供するものより強力なサーバーが必要です。

* 「Map」ビューから「Inspect」ビューに切り替えると、タイルに含まれるデータを確認できます。次のような表示になっていれば、ここまでは順調です！

![maputnik](assets/maputnik.png)

* レイヤーを追加してみましょう。どんなレイヤーを追加できるかは [MapLibre Style Spec](https://maplibre.org/maplibre-style-spec/) を参照してください。海やその他の水域のレイヤーなどが手軽でおすすめです。

* どのような属性があるかは、Inspect ビューで確認するか、[OpenMapTiles Schema](https://openmaptiles.org/schema/) のドキュメントを参照してください。これは Planetiler がデフォルトで使うタイルスキーマです。

## 4. MapLibre で Web ページを作成する

作成したスタイルに満足したら、上部バーの *Save* から *Create HTML* を選びます。

Codespace に index.html を作成し、ダウンロードした HTML ファイルの内容を貼り付けます。次に、簡易 Web サーバーを起動し、先ほどと同じようにインターネットに公開します。

```
python3 -m http.server 1234
```

すべてうまくいけば、1 時間足らずで自分だけのベースマップを作成し、デプロイできたことになります！

生成された HTML をよく読んで、MapLibre GL JS がどのようにセットアップされているかを理解しましょう。[MapLibre GL JS](https://maplibre.org/maplibre-gl-js/docs/) のドキュメントを見て、どんな API があるか調べてみてください。たとえば、地図上のどこかをクリックしたときにマーカーを追加する機能を作ってみましょう。

## 5. Planetiler で独自のオーバーレイを作成する

[`scripts/benches.yaml`](./scripts/benches.yaml) には、ベンチをすべて含む [カスタムマップタイル](https://github.com/onthegomap/planetiler/blob/main/planetiler-custommap/README.md) を Planetiler で作成する方法が記述されています。次のコマンドで実行してください。

```
java -jar /planetiler.jar scripts/benches.yaml --download_dir=/data/sources --minzoom=0 --maxzoom=14 --osm_path=/data/sources/workshop.osm.pbf  --tmpdir=/data/tmp --output=/data/benches.mbtiles
```

続いて、Martin を再度起動してこのタイルを配信します。

```
martin /data/workshop.mbtiles /data/benches.mbtiles
```

Maputnik で、`https://{Martin インスタンスの公開 URL}/benches` を新しいソースとして追加します。名前は `benches` にしましょう。その後、`benches` ソースを使った新しいレイヤーを地図に追加できます。たとえば、ワークショップ開催地のベンチを表示する circle レイヤーを追加してみましょう。

![maputnik](assets/add-source.png)

## データベースへの接続

コンテナ内で動いている PostgreSQL データベースへの接続を試してみましょう。次のコマンドでデータベースに接続できます。psql シェルを終了するには `quit` または Ctrl-D を使います。

```sh
psql postgres://postgres:password@db/maplibre

osm2pgsql -d postgresql://postgres:password@db/maplibre -O flex -S scripts/bicycle_parking.lua /data/sources/workshop.osm.pbf

# 新しいテーブルが作成され、データが入っていることを確認する
psql postgres://postgres:password@db/maplibre -c 'select * from bicycle_parking limit 3;'

martin /data/workshop.mbtiles postgres://postgres:password@db/maplibre
```
