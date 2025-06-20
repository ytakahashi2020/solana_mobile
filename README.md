# Solana Mobile Wallet App 作成マニュアル

Solana Mobile公式ドキュメントに基づくReact Nativeアプリ構築ガイドです。

## 目次

1. [前提条件](#前提条件)
2. [開発環境セットアップ](#開発環境セットアップ)
3. [プロジェクト作成方法](#プロジェクト作成方法)
4. [アプリの実装](#アプリの実装)
5. [ビルドと実行](#ビルドと実行)
6. [トラブルシューティング](#トラブルシューティング)

## 前提条件

### 必要なもの
- **Android開発環境**（Solana MobileはAndroidのみサポート）
- **Androidデバイスまたはエミュレーター**
- **MWA対応ウォレットアプリ**（例：Phantom, Solflare）

### 推奨開発環境
- Node.js 18.x 以上
- Android Studio
- Java Development Kit (JDK) 17
- Yarn（パッケージ管理はyarnを推奨）

## 開発環境セットアップ

### 1. Android開発環境

```bash
# Android Studioをインストール
# https://developer.android.com/studio からダウンロード

# Android SDK設定
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

### 2. 必要なツールのインストール

```bash
# Yarnをインストール（推奨）
npm install -g yarn

# React Native CLIをインストール
npm install -g @react-native-community/cli
```

## プロジェクト作成方法

Solana Mobile公式では2つの方法を提供しています：

### 方法1: Expo Template（推奨）

```bash
# Solana Mobile Expo テンプレートを使用
yarn create expo-app MySolanaDapp --template @solana-mobile/solana-mobile-expo-template

# プロジェクトディレクトリに移動
cd MySolanaDapp

# 依存関係をインストール
yarn install
```

### 方法2: React Native Scaffold

```bash
# Solana Mobile React Native テンプレートを使用
npx react-native init MySolanaDapp --template @solana-mobile/solana-mobile-dapp-scaffold --npm

# プロジェクトディレクトリに移動
cd MySolanaDapp

# Yarnで依存関係を再インストール（推奨）
rm -rf node_modules package-lock.json
yarn install
```

## アプリの実装

### 1. 基本的なファイル構成

公式テンプレートには以下が含まれています：

```
MySolanaDapp/
├── components/
│   ├── Account/
│   ├── AuthorizationProvider.tsx
│   ├── ConnectionProvider.tsx
│   └── RequestAirdrop.tsx
├── screens/
│   └── MainScreen.tsx
├── App.tsx
└── package.json
```

### 2. 主要コンポーネントの理解

#### AuthorizationProvider
```typescript
// Mobile Wallet Adapter認証を管理
import { AuthorizationProvider } from './components/AuthorizationProvider';

function App() {
  return (
    <AuthorizationProvider cluster="devnet">
      {/* Your app content */}
    </AuthorizationProvider>
  );
}
```

#### ConnectionProvider
```typescript
// Solana RPCコネクションを管理
import { ConnectionProvider } from './components/ConnectionProvider';

function App() {
  return (
    <ConnectionProvider>
      <AuthorizationProvider>
        {/* Your app content */}
      </AuthorizationProvider>
    </ConnectionProvider>
  );
}
```

### 3. 基本的なウォレット操作

#### ウォレット接続
```typescript
import { useAuthorization } from './components/AuthorizationProvider';

function ConnectWallet() {
  const { authorizeSession } = useAuthorization();
  
  const handleConnect = async () => {
    try {
      await authorizeSession();
    } catch (error) {
      console.error('Connection failed:', error);
    }
  };

  return (
    <TouchableOpacity onPress={handleConnect}>
      <Text>ウォレットに接続</Text>
    </TouchableOpacity>
  );
}
```

#### 残高確認
```typescript
import { useConnection } from './components/ConnectionProvider';
import { useAuthorization } from './components/AuthorizationProvider';

function AccountBalance() {
  const { connection } = useConnection();
  const { selectedAccount } = useAuthorization();
  const [balance, setBalance] = useState<number | null>(null);

  useEffect(() => {
    if (selectedAccount) {
      connection.getBalance(selectedAccount.publicKey)
        .then(balance => setBalance(balance / 1000000000));
    }
  }, [connection, selectedAccount]);

  return (
    <Text>残高: {balance?.toFixed(4)} SOL</Text>
  );
}
```

#### SOL送金
```typescript
import { useTransact } from './components/AuthorizationProvider';
import { SystemProgram, Transaction } from '@solana/web3.js';

function SendTransaction() {
  const transact = useTransact();

  const sendSOL = async (toAddress: string, amount: number) => {
    try {
      const transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: selectedAccount.publicKey,
          toPubkey: new PublicKey(toAddress),
          lamports: amount * 1000000000, // SOLからlamportsに変換
        })
      );

      const signature = await transact(transaction);
      console.log('Transaction signature:', signature);
    } catch (error) {
      console.error('Transaction failed:', error);
    }
  };

  return (
    // UI implementation
  );
}
```

### 4. メイン画面の実装

```typescript
// screens/MainScreen.tsx
import React from 'react';
import { ScrollView, StyleSheet, View } from 'react-native';
import { AuthorizationProvider } from '../components/AuthorizationProvider';
import { ConnectionProvider } from '../components/ConnectionProvider';
import ConnectWallet from '../components/ConnectWallet';
import AccountInfo from '../components/AccountInfo';
import SendTransaction from '../components/SendTransaction';

export default function MainScreen() {
  return (
    <ConnectionProvider>
      <AuthorizationProvider cluster="devnet">
        <ScrollView style={styles.container}>
          <ConnectWallet />
          <AccountInfo />
          <SendTransaction />
        </ScrollView>
      </AuthorizationProvider>
    </ConnectionProvider>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
  },
});
```

## ビルドと実行

### Expo Template使用の場合

```bash
# Expo Development Buildの作成
npx expo install --fix
npx expo run:android

# または、Expo開発ビルドを作成
eas build --platform android --profile development
```

### React Native Scaffold使用の場合

```bash
# Androidでビルドと実行
npx react-native run-android

# メトロバンドラーを別途起動する場合
npx react-native start
```

### テスト用ウォレットアプリのインストール

```bash
# FakeWallet（開発用）をインストール
# Solana Mobile GitHub ReleasesからAPKをダウンロード
# または以下のコマンドでビルド

git clone https://github.com/solana-mobile/mobile-wallet-adapter.git
cd mobile-wallet-adapter/android
./gradlew :fakewallet:assembleDebug
adb install fakewallet/build/outputs/apk/debug/fakewallet-debug.apk
```

## 重要な設定

### Metro設定
```javascript
// metro.config.js
const {getDefaultConfig, mergeConfig} = require('@react-native/metro-config');

const defaultConfig = getDefaultConfig(__dirname);

const config = {
  resolver: {
    alias: {
      crypto: 'react-native-crypto',
      stream: 'readable-stream',
      buffer: '@craftzdog/react-native-buffer',
    },
  },
};

module.exports = mergeConfig(defaultConfig, config);
```

### Polyfillの設定
```javascript
// index.js
import 'react-native-get-random-values';
import {AppRegistry} from 'react-native';
import App from './App';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);
```

## トラブルシューティング

### 1. プロジェクト作成エラー

```bash
# キャッシュクリア
npm cache clean --force
yarn cache clean

# 最新バージョンで再試行
yarn create expo-app MySolanaDapp --template @solana-mobile/solana-mobile-expo-template
```

### 2. ビルドエラー

```bash
# Android Gradle キャッシュクリア
cd android
./gradlew clean
cd ..

# node_modules再インストール
rm -rf node_modules
yarn install
```

### 3. Metro bundler エラー

```bash
# Metro キャッシュクリア
npx react-native start --reset-cache
```

### 4. ウォレット接続エラー

- **MWA対応ウォレットがインストールされているか確認**
- **ウォレットアプリが最新バージョンか確認**
- **デバイスがdevnetに接続されているか確認**

### 5. 一般的な依存関係の問題

```bash
# React Native環境診断
npx react-native doctor

# 依存関係の強制更新
yarn install --force
```

## 次のステップ

1. **Solana Program Library (SPL) トークンの統合**
2. **NFTミント機能の追加**
3. **カスタムプログラムとの統合**
4. **本番環境用の最適化**
5. **Google Play Storeへのデプロイ**

## 参考リンク

- [Solana Mobile公式ドキュメント](https://docs.solanamobile.com/)
- [Mobile Wallet Adapter仕様](https://github.com/solana-mobile/mobile-wallet-adapter)
- [Solana Web3.js ドキュメント](https://solana-labs.github.io/solana-web3.js/)
- [React Native公式ドキュメント](https://reactnative.dev/)

このマニュアルに従って開発すれば、Solana Mobile公式の推奨方法でアプリを構築できます。