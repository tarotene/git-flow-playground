# Git-flow playground

Git-flow CLI ツールを用いてブランチを並行に編集し取り込むための練習台．

特に，`develop` と `main` を保護していることを想定（Github Pro の使用を推奨）．

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

    このとき，下記についてはデフォルトとは異なる設定が必要:

    | 設定項目           | 値  | デフォルト値 (参考) |
    | ------------------ | --- | ------------------- |
    | Version tag prefix | `v` | (空)                |

3. After `git flow init`

    ブランチが `develop` に切り替わったことを確認，もしくは手動で `git checkout develop` したら，一旦リモート側に反映:

    ```bash
    git push --all
    ```

    ここで作成された `2` つのブランチ---`main` ならびに `develop`---が今後 git flow において HEAD の役割を果たす．

    次に，リモートのホスティングサービス側で下記を実施する:

    - デフォルトブランチを `main` から `develop` に変更
    - `main` を削除から保護

    詳しくは下記のページを参照:

    - <https://fwywd.com/tech/github-switch-default-branch>
    - <https://zenn.dev/kehonda/articles/a5b08e9df03d13>

    > 上記のページには「`main` を保護する理由の一つは hotfix 時に `main` -> `develop` のマージで誤って `main` を削除する危険の回避である（意訳）」という記述があるが，`main` 削除のチャンスはこれ以外にもある．実は `hotfix` 時に限らず，release を実施すると `release` -> `develop`, `release` -> `main` という順番に個別マージが実施されるので`，main` のタイムスタンプが常に `develop` のそれよりも新しくなり，結果として `main` -> `develop` の取り込みを Github 上で促されるようになる．しかし，こうした状況で求められる作業フローは至って機械的なものであるため，リモート（ホスティングサービス）側で Github Actions か何かで自動化しつつ，これにローカル側を追従させる作業を開発者に（手順書か何かで）覚えさせるというのが良い落とし所になるだろう．このことさえ分かっておけば，git flow によるマージ順序が `release` -> `master`, `release` -> `develop` になった場合にも対応できる．
