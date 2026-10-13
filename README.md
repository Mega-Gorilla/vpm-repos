# VHub Packages

VHub の VRChat 向けパッケージを [VCC（VRChat Creator Companion）](https://vcc.docs.vrchat.com/) で配布するための VPM リポジトリです。

- 公開ページ：https://mega-gorilla.github.io/vpm-repos/
- リスティング URL：`https://mega-gorilla.github.io/vpm-repos/index.json`

## VCC への追加

次のどちらかの方法でリポジトリを VCC に追加します。

- **公開ページから**：[公開ページ](https://mega-gorilla.github.io/vpm-repos/)の「Add to VCC」を押すと、VCC のリポジトリ追加画面が開きます。
- **手動で**：VCC の **Settings > Packages > Add Repository** にリスティング URL を貼り付けて **Add** を押し、内容を確認して **I Understand** を押します。

追加後は、VCC の **Manage Project** で対象のプロジェクトを開くと、下記のパッケージを追加できます。更新も VCC から行えます。

## 配布中のパッケージ

| パッケージ | パッケージ ID | 概要 |
|---|---|---|
| [KawaPlayer](https://github.com/Mega-Gorilla/KawaPlayer) | `com.vhub.kawaplayer` | [YamaPlayer](https://github.com/koorimizuw/YamaPlayer) をフォークした動画プレイヤー。[VHub PlayList](https://playlist.vrc-hub.com) のプレイリスト読み込みや、入室時に自動再生する動画 URL の設定を VRChat 内から行えます。YamaPlayer とは同時にインストールできません。 |
| [VHub PlaylistViewer](https://github.com/Mega-Gorilla/VhubPlaylistViewer) | `com.vhub.kawaplayer-playlistviewer` | ワールド内で VHub PlayList のプレイリストを閲覧し、その URL を KawaPlayer などの動画プレイヤーに渡すための UI です。 |

どちらも Unity 2022.3 と VRChat World SDK（`com.vrchat.worlds`）3.8.1 以上が必要です。使い方・ライセンス・VCC を使わない導入方法は、各パッケージのリポジトリを参照してください。最新のバージョンは公開ページで確認できます。

## メンテナー向け情報

### 構成

| パス | 役割 |
|---|---|
| `source.json` | リスティングの名前・ID・URL と、配布するパッケージのリポジトリ（`githubRepos`） |
| `Website/` | 公開ページ。`index.html` と `app.js` は [Scriban](https://github.com/scriban/scriban) のテンプレートで、ビルド時に描画されます |
| `.github/workflows/build-listing.yml` | 各リポジトリのリリースを集めて `index.json` を生成し、GitHub Pages にデプロイします |

### リスティングが更新されるタイミング

`build-listing.yml` は [vrchat-community/package-list-action](https://github.com/vrchat-community/package-list-action) で `index.json` と公開ページを生成し、GitHub Pages にデプロイします。次のときに実行されます。

- **KawaPlayer のリリース時**：KawaPlayer の `Build Release` ワークフローを `publish` 付きで実行すると、最後に `repository_dispatch` でこのワークフローが起動します。
- **VHub PlaylistViewer のリリース時**：自動連携はありません。リリース後に手動で実行してください。
- **`main` への push**：`source.json`・`Website/` 配下・ワークフロー自体を変更したときに実行されます。README などの変更では実行されません。

手動で実行する場合：

```bash
gh workflow run build-listing.yml
```

### パッケージを追加するとき

1. 追加するリポジトリで GitHub Release を作成し、パッケージの `.zip` をアセットとして添付します。`package.json` は zip の直下に置いてください。
2. `source.json` の `githubRepos` に `"オーナー/リポジトリ名"` を追記して `main` にマージします。マージするとリスティングが再構築されます。

リリースに添付された `.zip` はすべてダウンロードされ、直下に `package.json` があるものだけが一覧に載ります。

### 注意点

- `source.json` の `id`（`com.vhub.vpm`）と `url` は変更しないでください。VCC はこの URL から一覧と更新を取得します。
- 公開ページの Fluent UI は、`index.html` と `app.js` の両方で同じバージョン（2.6.1）に固定しています。バージョンを指定しない URL にすると、互換性のない新バージョンが読み込まれてページが動かなくなります（[#3](https://github.com/Mega-Gorilla/vpm-repos/issues/3)）。
- 公開ページのローカルでの描画手順や作業上の注意は [CLAUDE.md](CLAUDE.md) と [AGENTS.md](AGENTS.md) にまとめています。

このリポジトリは [vrchat-community/template-package-listing](https://github.com/vrchat-community/template-package-listing) をもとに作成しています。
