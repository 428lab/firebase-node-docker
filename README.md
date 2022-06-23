# Firebase Project in Firebase Emulator

## firabse console

[firabseコンソール](https://console.firebase.google.com/)から事前にプロジェクトを作成し、Blazeプランに変更。
firestore、functions、storageを有効化しておく。

## dockerコンテナをビルドして起動

```sh
$ docker-compose build
$ docker-compose up -d
```

## コンテナに入る

```sh
$ docker-compose exec firebase bash
```

## firebaseプロジェクトの構築

Dockerコンテナ上でFirebaseプロジェクトを立ち上げる手順です。

### Firebaseにログインする

```sh
$ firebase login --no-localhost
```

コンテナで使用中の場合は`--no-localhost`オプションを指定して、ログインする。  
これがないとlocalhostへのリダイレクトが発生して失敗する。

以下のように成功するとログインのメッセージが出る

```text
✔  Success! Logged in as ******@****.****
```

**デプロイ用のトークンを取得する**

FirebaseをCIデプロイする場合はトークンを取得する必要があるため、以下のコマンドを実行する。

```
$ firebase login:ci --no-localhost
```

認証完了すると、以下のように表示される。  
`1//xxx...`で始まるトークンを保存しておこう。

```text
Waiting for authentication...

✔  Success! Use this token to login on a CI server:

1//xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Firebaseプロジェクトの初期化と設定を行う

```text
$ firebase init

     ######## #### ########  ######## ########     ###     ######  ########
     ##        ##  ##     ## ##       ##     ##  ##   ##  ##       ##
     ######    ##  ########  ######   ########  #########  ######  ######
     ##        ##  ##    ##  ##       ##     ## ##     ##       ## ##
     ##       #### ##     ## ######## ########  ##     ##  ######  ########
```

プロジェクトディレクトリを聞かれるので、/appを選択

```text
You're about to initialize a Firebase project in this directory:

  /app

Before we get started, keep in mind:

  * You are currently outside your home directory

? Which Firebase features do you want to set up for this directory? Press Space to select features, then Enter to confirm your choices. Firestore: Configure security rule
s and indexes files for Firestore, Functions: Configure a Cloud Functions directory and its files, Hosting: Configure files for Firebase Hosting and (optionally) set up G
itHub Action deploys, Hosting: Set up GitHub Action deploys, Storage: Configure a security rules file for Cloud Storage, Emulators: Set up local emulators for Firebase pr
oducts

=== Project Setup

First, let's associate this project directory with a Firebase project.
You can create multiple project aliases by running firebase use --add, 
but for now we'll just set up a default project.

i  Using project test-func-kojira (test-func-kojira)

=== Firestore Setup

Firestore Security Rules allow you to define how and when to allow
requests. You can keep these rules in your project directory
and publish them with firebase deploy.

? What file should be used for Firestore Rules? firestore.rules

Firestore indexes allow you to perform complex queries while
maintaining performance that scales with the size of the result
set. You can keep index definitions in your project directory
and publish them with firebase deploy.

? What file should be used for Firestore indexes? firestore.indexes.json

=== Functions Setup

A functions directory will be created in your project with sample code
pre-configured. Functions can be deployed with firebase deploy.

? What language would you like to use to write Cloud Functions? JavaScript
? Do you want to use ESLint to catch probable bugs and enforce style? No
✔  Wrote functions/package.json
✔  Wrote functions/index.js
✔  Wrote functions/.gitignore
? Do you want to install dependencies with npm now? Yes

added 233 packages, and audited 234 packages in 33s

11 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities

=== Hosting Setup

Your public directory is the folder (relative to your project directory) that
will contain Hosting assets to be uploaded with firebase deploy. If you
have a build process for your assets, use your build's output directory.

? What do you want to use as your public directory? public
? Configure as a single-page app (rewrite all urls to /index.html)? Yes
? Set up automatic builds and deploys with GitHub? No
✔  Wrote public/index.html

=== Storage Setup

Firebase Storage Security Rules allow you to define how and when to allow
uploads and downloads. You can keep these rules in your project directory
and publish them with firebase deploy.

? What file should be used for Storage Rules? storage.rules
✔  Wrote storage.rules

=== Emulators Setup
? Which Firebase emulators do you want to set up? Press Space to select emulators, then Enter to confirm your choices. Authentication Emulator, Functions Emulator, Firest
ore Emulator, Hosting Emulator, Storage Emulator
? Which port do you want to use for the auth emulator? 9099
? Which port do you want to use for the functions emulator? 5001
? Which port do you want to use for the firestore emulator? 8080
? Which port do you want to use for the hosting emulator? 5000
? Which port do you want to use for the storage emulator? 9199
? Would you like to enable the Emulator UI? Yes
? Which port do you want to use for the Emulator UI (leave empty to use any available port)? 4000
? Would you like to download the emulators now? Yes
i  firestore: downloading cloud-firestore-emulator-v1.13.1.jar...
Progress: ================================================================================================================================================> (100% of 61MB
i  storage: downloading cloud-storage-rules-runtime-v1.0.1.jar...
Progress: ================================================================================================================================================> (100% of 33MB
i  ui: downloading ui-v1.6.4.zip...

i  Writing configuration info to firebase.json...
i  Writing project information to .firebaserc...

✔  Firebase initialization complete!
```

## firebase.jsonを以下のように書き換え

`"host": "0.0.0.0",`を各emulatorの行に書き足す。

```json
{
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  },
  "storage": {
    "rules": "storage.rules"
  },
  "emulators": {
    "auth": {
      "host": "0.0.0.0",
      "port": 9099
    },
    "functions": {
      "host": "0.0.0.0",
      "port": 5001
    },
    "firestore": {
      "host": "0.0.0.0",
      "port": 8080
    },
    "hosting": {
      "host": "0.0.0.0",
      "port": 5000
    },
    "storage": {
      "host": "0.0.0.0",
      "port": 9199
    },
    "ui": {
      "host": "0.0.0.0",
      "enabled": true,
      "port": 4000
    }
  }
}
```

## エミュレータ起動

コンテナ内で`firebase emulators:start` でエミュレータを起動

```sh
$ firebase emulators:start
```

```text
i  emulators: Starting emulators: auth, functions, firestore, hosting, storage
⚠  functions: The following emulators are not running, calls to these services from the Functions emulator will affect production: database, pubsub
✔  functions: Using node@16 from host.
i  firestore: Firestore Emulator logging to firestore-debug.log
i  hosting: Serving hosting files from: public
✔  hosting: Local server: http://localhost:5000
i  ui: downloading ui-v1.6.4.zip...
Progress: =================================================================================================================================================> (100% of 4MB
i  ui: Emulator UI logging to ui-debug.log
i  functions: Watching "/app/functions" for Cloud Functions...

┌─────────────────────────────────────────────────────────────┐
│ ✔  All emulators ready! It is now safe to connect your app. │
│ i  View Emulator UI at http://localhost:4000                │
└─────────────────────────────────────────────────────────────┘

┌────────────────┬────────────────┬─────────────────────────────────┐
│ Emulator       │ Host:Port      │ View in Emulator UI             │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Authentication │ localhost:9099 │ http://localhost:4000/auth      │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Functions      │ localhost:5001 │ http://localhost:4000/functions │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Firestore      │ localhost:8080 │ http://localhost:4000/firestore │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Hosting        │ localhost:5000 │ n/a                             │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Storage        │ localhost:9199 │ http://localhost:4000/storage   │
└────────────────┴────────────────┴─────────────────────────────────┘
  Emulator Hub running at localhost:4400
  Other reserved ports: 4500

Issues? Report them at https://github.com/firebase/firebase-tools/issues and attach the *-debug.log files.
```

[http://localhost:4000](http://localhost:4000) からエミュレータのUIにアクセスできるようになる。
