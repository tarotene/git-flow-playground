# Git-flow playground

Git-flow CLI ツールを用いてブランチを並行に編集し取り込むための練習台．Github Pro の適用を推奨．

## Prerequisites

- コマンドラインツール [`git flow`](https://danielkummer.github.io/git-flow-cheatsheet/index.ja_JP.html) が使えること．

## Getting started

1. Before `git flow init`

    > Github 等で作成したばかりのリポジトリをローカルに pull したばかりの状況を仮定．

    まずは `Initial commit` 相当の作業をローカルの `main` 上で実施．

    > `README.md` など最低限のファイル追加はこのタイミングで行っておくのが吉．

    何もすることがなければ空コミット:

    ```bash
    git commit -S --allow-empty -m "Initial commit"
    ```

    > `git flow` はこの辺の初期作業については何も制限を課さない．何事もそうだが，スタートアップのための作業とメインルーチン系の作業は性質を異にする．

2. `git flow init`

    ```bash
    git flow init
    ```

    基本的に全てデフォルト設定で大丈夫．間違えてしまった場合も `git flow init -f` で取り返せる．空文字列は `''` で表現．

3. After `git flow init`

    ブランチが `develop` に切り替わったことを確認，もしくは手動で `git checkout develop` したら，一旦リモート側に反映:

    ```bash
    git push --all
    ```

    ここで作成された `2` つのブランチ---`main` ならびに `develop`---が今後 git flow において HEAD の役割を果たす．

    次に，リモートのホスティングサービス側で下記を実施する:

    - デフォルトブランチを `main` から `develop` に変更
    - `main` を削除から保護
    - `develop` を幾つかの破壊的変更から保護

    > 特に，トピックブランチから `develop` へのマージ時に PR 作成を強制することで意図せずリモート側で `develop` が更新されることを防げる．

    詳しくは下記のページを参照:

    - <https://fwywd.com/tech/github-switch-default-branch>
    - <https://zenn.dev/kehonda/articles/a5b08e9df03d13>

    > 上記のページには「`main` を保護する理由の一つは hotfix 時に `main` -> `develop` のマージで誤って `main` を削除する危険の回避である（意訳）」という記述があるが，`main` 削除のチャンスはこれ以外にもある．実は `hotfix` 時に限らず，release を実施すると `release` -> `develop`, `release` -> `main` という順番に個別マージが実施されるので`，main` のタイムスタンプが常に `develop` のそれよりも新しくなり，結果として `main` -> `develop` の取り込みを Github 上で促されるようになる．しかし，こうした状況で求められる作業フローは至って機械的なものであるため，リモート（ホスティングサービス）側で Github Actions か何かで自動化しつつ，これにローカル側を追従させる作業を開発者に（手順書か何かで）覚えさせるというのが良い落とし所になるだろう．このことさえ分かっておけば，git flow によるマージ順序が `release` -> `master`, `release` -> `develop` になった場合にも対応できる．

## Regular development flow

### 機能追加

節目節目で Pull Request (PR) を作成し，トピックブランチから `develop` に取り込むフロー．

> TODO: 推敲する

ドキュメントの追記を例として説明する．

1. トピックブランチを作成:

    ```bash
    # ここでは仮に `append-description` としておく
    git flow feature start append-description
    ```

2. ブランチが `append-description` に切り替わっていることを確認し，適当な変更作業を実施:

    ```bash
    # 一例
    touch description.appended
    ```

3. ブランチをリモートにプッシュ:

    - 初回の場合は git flow の機能を使用:

        ```bash
        git flow feature publish append-description
        ```

    - `2` 回目以降の場合は（リモートに既に同名のブランチがあるので）git の機能を使用:

        ```bash
        git push
        ```

4. 変更作業 -> プッシュの区切りが一通り着いたら，interactive rebase で適当にコミットまとめ直すなど最後の仕上げを実施し，PR を作成．

    - `develop` を保護していないと，この手順を回避できてしまう．

5. PR が approved であることを確認し，`develop` への取り込みを実施．

    ```bash
    git flow feature finish append-description && git push --all
    ```

    - ここでの取り込みは，まずローカルでトピックブランチを `develop` に取り込み，その結果をリモートにミラーリングするイメージ．オプション `--all` が付いていることで，対象となるトピックブランチ以外の追従状況に異常があればそれをエラーとして検知，弾いてくれる（とはいえ `--all` は push merged 可能なブランチが `1` つでもあれば個別に push をトライする模様）．
    - PR が approve される前に `git flow feature finish append-description` を行うと詰むバグが存在する．
    - レビュアーとレビュイーの役割をよく整理しておく必要がある．

- 並行開発の場合のプラクティスについても述べる．
- 定期的に prune 系コマンドを叩くことについても述べる．
  - 候補は `git push --prune origin "refs/heads/*:refs/heads/*"` ...?

### リリース

> 作成途中．

1. リリースブランチを作成:

    ```bash
    # ここでは仮に `v0.1.0` としておく．セマンティックバージョンに接頭辞 `v` を付加したものを使用するのは一般的である．
    ```

2. ブランチが `release/v0.1.0` に切り替わっていることを確認し，リリースノート作成等・QA を実施．

3. ブランチをリモートにプッシュ（公開）:

    - 初回の場合は git flow の機能を使用:

        ```bash
        git flow release publish v0.1.0
        ```

    - `2` 回目以降の場合は（リモートに既に同名のブランチがあるので）git の機能を使用:

        ```bash
        git push
        ```

4. 必要なことを全て済ませた状態で，`develop` および `main` への取り込みを実施し，`main` 上でタグ打ちを実施．

    ```bash
    git flow release finish v0.1.0 && git push --tags
    ```

5. ホスティングサービス上で，適当な方法で tag から release を生成する．

6. 最後に `main` および `develop` を push して終了:

    ```bash
    git push --all
    ```

ここで，どうしても `main` と `develop` の間にタグコミットの分だけ差分が生まれてしまうので，これを解消する方法を考えなくてはいけない．

### ホットフィックス

> 作成途中．

## Misc

- Issue のテンプレ化
- PR の作成自動化・テンプレ化
- Release note や Change log の作成自動化

についても触れる．自動化は一旦 Github Actions ありきで考える．
