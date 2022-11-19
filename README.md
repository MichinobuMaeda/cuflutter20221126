# cuflutter20221126

[コンピュータ・ユニオン関西IT支部 IT技術者・クリエイターカフェ](https://cu-kansai-it.org/) 2022年11月 サンプルコード

[報告資料](https://pages.michinobu.jp/t/20221126firebaseflutter.html)

このサンプルは Flutter のプロジェクト作成時に自動で生成される、ボタンを押すと数値がインクリメントされるサンプルのカウンターを [Firebase](https://firebase.google.com/) の Cloud Firestore のバックエンドに差し替えただけのものです。

## A. 開発環境

## A.1. 必要なもの

- git
- fvm ( [Flutter Version Management](https://fvm.app/) ): Flutter のバージョンアップが速いので、一定のバージョンで作業するために利用する。
- nvm ( [Node Version Manager](https://github.com/nvm-sh/nvm) ): Node.js 16 を firebase-tools で使用する。
- Java ( >= 11 ): [Firebase Local Emulator Suite](https://firebase.google.com/docs/emulator-suite) で使用する。

以下のバージョンは、このサンプルの作成時のものです。

```bash
$ git --version
git version 2.38.1

$ fvm --version
2.4.1

$ fvm use 3.3.7
$ fvm flutter --version
Flutter 3.3.7 ... ... ...

$ nvm --version
0.39.2

$ echo 16 > .nvmrc
$ node --version
v16.18.0

$ java --version
openjdk 17.0.4.1 2022-08-12
 ... ... ...
```

## A.2. 開発環境の構築

このサンプルをローカルで動かすには以下のようにしてください。

```bash
$ git clone git@github.com:MichinobuMaeda/cuflutter20221126.git
$ cd cuflutter20221126
$ fvm flutter pub get
$ npm install
$ npx firebase --version
11.16.0

$ npm start
```

`npm start` で Firebase Emulators と Flutter のテストサーバを起動します。

## B. 参考：このプロジェクトの作成手順

### B.1. Flutter のプロジェクトの作成

```bash
$ fvm --version
2.4.1

$ fvm global 3.3.7
$ fvm flutter --version
Flutter 3.3.7 ... ... ...

$ fvm flutter create cuflutter20221126 --platforms web
```

### B.2. Firebase の設定の追加

```bash
$ npx firebase login
$ npx firebase init

? Which Firebase features do you want to set up for this directory?
- Firestore: Configure security rules and indexes files for Firestore,
- Functions: Configure a Cloud Functions directory and its files,
- Hosting: Configure files for Firebase Hosting and (optionally) set up GitHub Action deploys,
- Hosting: Set up GitHub Action deploys,
- Storage: Configure a security rules file for Cloud Storage,
- Emulators: Set uplocal emulators for Firebase products

##### 注 ##### Functions と Storage はこのサンプルソースでは使わない。

=== Project Setup

? Please select an option: Use an existing project
? Select a default Firebase project for this directory: cuflutter20221126 (cuflutter20221126)

=== Firestore Setup

? What file should be used for Firestore Rules? firestore.rules
? What file should be used for Firestore indexes? firestore.indexes.json

=== Functions Setup

? What language would you like to use to write Cloud Functions? JavaScript
? Do you want to use ESLint to catch probable bugs and enforce style? No
? Do you want to install dependencies with npm now? No

=== Hosting Setup

? What do you want to use as your public directory? build/web

##### 注 ##### デフォルトの public から変更が必要。

? Configure as a single-page app (rewrite all urls to /index.html)? No
? Set up automatic builds and deploys with GitHub? Yes

✔  Wrote build/web/404.html
✔  Wrote build/web/index.html

##### 注 ##### 404.html と index.html は不要なので後で削除する。

? For which GitHub repository would you like to set up a GitHub workflow? (format: user/repository) MichinobuMaeda/cuflutter20221126
? Set up the workflow to run a build script before every deploy? No
? Set up automatic deployment to your site's live channel when a PR is merged? Yes
? What is the name of the GitHub branch associated with your site's live channel? main

i  Action required: Visit this URL to revoke authorization for the Firebase CLI GitHub OAuth App:
https://github.com/settings/connections/applications/89cf50f02ac6aaed3484

##### 注 ##### 必ずこの URL を開いて Firebase のプロジェクトのアクセス許可を削除すること。

=== Storage Setup

? What file should be used for Storage Rules? storage.rules

=== Emulators Setup

? Which Firebase emulators do you want to set up? Press Space to select emulators, then Enter to confirm your choices.
- Authentication Emulator,
- Functions Emulator,
- Firestore Emulator,
- Storage Emulator

##### 注 ##### Flutter SDK のテストサーバを利用するので Hosting Emulator は不要。

? Which port do you want to use for the auth emulator? 9099
? Which port do you want to use for the functions emulator? 5001
? Which port do you want to use for the firestore emulator? 8080
? Which port do you want to use for the storage emulator? 9199
? Would you like to enable the Emulator UI? Yes
? Which port do you want to use for the Emulator UI (leave empty to use any available port)? 4040

##### 注 ##### Cloud Workstations などを利用する場合は Emulator UI に固定のポートを指定する。ランダムに指定されると面倒。

? Would you like to download the emulators now? No

✔  Firebase initialization complete!

```

正常に設定できると GitHub のリポジトリの設定の Actions secrets に
`FIREBASE_SERVICE_ACCOUNT_CUFLUTTER20221126`
が設定されているはず。
GitHub Actions から Firebase Hosting にデプロイする際にこのアカウント情報が使われる。
Actions の設定内容は、生成された `.github/workflows/*.yaml` 参照。

Actions secrets には以下の2点も設定しておくとよい。

- Firebase の API Key: 公開リポジトリの場合はソース中に入れない方がいいので Actions の中で設定する。
- CI用のアカウント情報: 下記の手順で取得したトークンを使う。

```bash
$ npx firebase login:ci
 ... ... ...
1// ... ... ...

Example: firebase deploy --token "$FIREBASE_TOKEN"
```

それぞれ `FIREBASE_API_KEY_CUFLUTTER20221126` および `FIREBASE_TOKEN_CUFLUTTER20221126` として設定した。

[Add Firebase to your Flutter app](https://firebase.google.com/docs/flutter/setup?platform=web)
の手順で Firebase の初期化コードを追加する。
`firebase_options.dart` が自動で生成されるので、その中の `apiKey` をダミーの文字列に置き換えて GitHub Actions の中で設定し直すようにする。ローカルのテスト環境はダミーの文字列のままでよい。

GitHub Actions の中の処理は以下の通り。

```yaml
      - name: Set firebase api key
        run: |
          sed 's/FIREBASE_API_KEY/${{ secrets.FIREBASE_API_KEY_CUFLUTTER20221126 }}/' \
            -i lib/firebase_options.dart
```
