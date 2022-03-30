---
title: TypeScriptでdotenvを使う
date: 2022-03-30 20:19:56
updated: 2022-03-30 21:41:00
categories: [TypeScript, React-Native, dotenv]
tags:
- TypeScript
- React-Native
- dotenv
description: "Firebaseを使うときに、githubにAPIキーを公開したくないため、dotenvを用いる"
---

### APIキーをgithubに公開したくない
Firebaseを使うときに、githubにAPIキーを公開したくないため、今回[react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv)を用いる

<!-- more -->
<!-- toc -->

### 使い方
まずnpmにインストールする
{% codeblock terminal lang:bash line_number:false %}
npm install --save react-native-dotenv
{% endcodeblock %}

次に、`babel.config.js`に記述する
{% codeblock babel.config.js lang:diff line_number:true %}
module.exports = {
  presets: ['module:metro-react-native-babel-preset'],
+  plugins: [
+    [
+      'module:react-native-dotenv',
+      {
+        moduleName: '@env',
+        path: '.env',
+      },
+    ],
+  ],
};
{% endcodeblock %}

型定義を用意する。rootフォルダにenv.d.tsを新規作成する(どこでもいい気はする、未検証)
{% codeblock env.d.ts lang:typescript line_number:true %}
declare module '@env' {
  export const FIREBASE_API_KEY: string;
  export const FIREBASE_AUTH_DOMAIN: string;
  export const FIREBASE_DATABASE_URL: string;
  export const FIREBASE_PROJECT_ID: string;
  export const FIREBASE_STORAGE_BUCKET: string;
  export const FIREBASE_MESSEGING_SENDER_ID: string;
  export const FIREBASE_APP_ID: string;
  export const FIREBASE_MEASUREMENT_ID: string;
}
{% endcodeblock %}

次に、`firebaseAuth.ts`に記述する(Firebaseの認証情報等が書かれたファイル)
{% codeblock firebaseAuth.ts lang:diff line_number:true %}
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";
+ import {
  + FIREBASE_API_KEY,
  + FIREBASE_AUTH_DOMAIN,
  + FIREBASE_PROJECT_ID,
  + FIREBASE_STORAGE_BUCKET,
  + FIREBASE_MESSEGING_SENDER_ID,
  + FIREBASE_APP_ID,
  + FIREBASE_MEASUREMENT_ID,
} from '@env';
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
-  apiKey: "<Firebase API Key>",
+  apiKey: FIREBASE_API_KEY,
-  authDomain: "<Firebase Auth Domain>",
+  authDomain: FIREBASE_AUTH_DOMAIN,
-  projectId: "<Firebase Project ID>",
+  projectId: FIREBASE_PROJECT_ID,
-  storageBucket: "<Firebase Storage Bucket>",
+  storageBucket: FIREBASE_STORAGE_BUCKET,
-  messagingSenderId: "<Firebase Messaging Sender ID>",
+  messagingSenderId: FIREBASE_MESSEGING_SENDER_ID,
-  appId: "<Firebase App ID>",
+  appId: FIREBASE_APP_ID,
-  measurementId: "<Firebase Measurement ID>",
+  measurementId: FIREBASE_MEASUREMENT_ID
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);
{% endcodeblock %}

rootフォルダに`.env`ファイルを作成する
{% codeblock .env lang:diff line_number:true %}
FIREBASE_API_KEY=<Firebase API Key>
FIREBASE_AUTH_DOMAIN=<Firebase Auth Domain>
FIREBASE_PROJECT_ID=<Firebase Project ID>
FIREBASE_STORAGE_BUCKET=<Firebase Storage Bucket>
FIREBASE_MESSEGING_SENDER_ID=<Firebase Messaging Sender ID>
FIREBASE_APP_ID=<Firebase App ID>
FIREBASE_MEASUREMENT_ID=<Firebase Measurement ID>
{% endcodeblock %}

`.gitignore`に`.env`ファイルを追加する **!ここが一番大事!**
{% codeblock .gitignore lang:diff line_number:true first_line:59 %}
\# CocoaPods
/ios/Pods/
+ .env
{% endcodeblock %}

### 終わりに
あとはeslintやらVSCode上で問題が出ないことを確認すれば、Firebaseの認証情報をgithubに公開することなく、使うことができる

githubに今回変更箇所をプッシュしておいたのでもしよければ→[github](https://github.com/m0r016/replog/commit/db92ba2133df3becb82af1e071b0259e07c4892d)
また、疑問点や間違ってる箇所があった[issue](https://github.com/m0r016/blog.m0r016.net/issues)や[Twitter](https://twitter.com/m0r016)まで