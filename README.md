# cuflutter20221126

## A. 開発環境

## A.1. 必要なもの

- git
- fvm: flutter
- nvm: Node.js 16 を firebase-tools で使用
- Java ( 11 以降 ): Firebase Emulator で使用

```
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

```
$ git clone git@github.com:MichinobuMaeda/cuflutter20221126.git
$ cd cuflutter20221126
$ fvm flutter pub get
$ npm install
$ npx firebase --version
11.16.0

```

## B. このプロジェクトの作成手順

### B.1. Flutter のプロジェクトの作成

```
$ fvm --version
2.4.1

$ fvm global 3.3.7
$ fvm flutter --version
Flutter 3.3.7 ... ... ...

$ fvm flutter create cuflutter20221126 --platforms web
```

### B.2. Firebase の設定の追加

```
$ npx firebase login
$ npx firebase init
```
