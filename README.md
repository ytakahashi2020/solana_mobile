# Solana Mobile Wallet App 作成マニュアル（Expo版）

Solana Mobile公式ドキュメントに基づく、**Expo**での実用的なモバイルウォレットアプリ構築ガイドです。今回作成したアプリと同じものをExpoで一から作成できます。

> **注意:** Expoを使用することで、React Nativeの複雑な環境設定を回避し、より簡単にSolanaモバイルアプリを構築できます。

## 目次

1. [前提条件](#前提条件)
2. [環境構築](#環境構築)
3. [プロジェクト作成とセットアップ](#プロジェクト作成とセットアップ)
4. [依存関係のインストール](#依存関係のインストール)
5. [基本設定ファイル](#基本設定ファイル)
6. [プロバイダーコンポーネントの実装](#プロバイダーコンポーネントの実装)
7. [UI コンポーネントの実装](#ui-コンポーネントの実装)
8. [メイン画面の実装](#メイン画面の実装)
9. [アプリエントリーポイント](#アプリエントリーポイント)
10. [ビルドと実行](#ビルドと実行)
11. [トラブルシューティング](#トラブルシューティング)

## 前提条件

### 必要なもの
- **Android開発環境**（Solana MobileはAndroidのみサポート）
- **Androidデバイスまたはエミュレーター**
- **Node.js 18.x 以上**
- **Java Development Kit (JDK) 17**

### 推奨ツール
- **Android Studio**
- **Yarn**（パッケージ管理）
- **Visual Studio Code**（エディタ）

## 環境構築

### 1. Android開発環境

```bash
# Android Studioをインストール
# https://developer.android.com/studio からダウンロード

# Android SDK設定（Mac/Linux）
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Windows の場合
setx ANDROID_HOME "%LOCALAPPDATA%\\Android\\Sdk"
setx PATH "%PATH%;%LOCALAPPDATA%\\Android\\Sdk\\platform-tools"
```

### 2. 必要なツールのインストール

```bash
# Yarnをインストール（推奨）
npm install -g yarn

# React Native CLIをインストール
npm install -g @react-native-community/cli
```

### 3. 開発環境の確認

```bash
# React Native環境診断
npx react-native doctor

# Android デバイス/エミュレーターの確認
adb devices
```

## プロジェクト作成とセットアップ

### 1. Expo プロジェクトの作成

```bash
# Expo CLIのインストール（グローバル）
npm install -g @expo/cli

# TypeScriptテンプレートでプロジェクト作成
npx create-expo-app SimpleSolanaMobileApp --template blank-typescript

# プロジェクトディレクトリに移動
cd SimpleSolanaMobileApp

# 依存関係をインストール
npm install
```

### 1.1. Expo開発ビルド設定

Solana関連パッケージはカスタムネイティブコードを含むため、Expo開発ビルドが必要です：

```bash
# EAS CLIのインストール
npm install -g eas-cli

# EAS設定の初期化
eas build:configure

# eas.jsonが作成されることを確認
cat eas.json
```

### 2. プロジェクト構造の作成

```bash
# 必要なディレクトリを作成
mkdir -p components/providers
mkdir screens
mkdir util
```

最終的なディレクトリ構造：
```
SimpleSolanaMobileApp/
├── components/
│   ├── providers/
│   │   ├── AuthorizationProvider.tsx
│   │   └── ConnectionProvider.tsx
│   ├── AccountInfo.tsx
│   ├── ConnectButton.tsx
│   ├── DisconnectButton.tsx
│   ├── Header.tsx
│   ├── MockSendButton.tsx
│   ├── MockWalletButton.tsx
│   ├── RefreshBalanceButton.tsx
│   ├── RequestAirdropButton.tsx
│   ├── Section.tsx
│   ├── SendTokenButton.tsx
│   ├── SignMessageButton.tsx
│   ├── SignTransactionButton.tsx
│   └── SimpleNFTMint.tsx
├── screens/
│   └── MainScreen.tsx
├── util/
│   └── alertAndLog.ts
├── App.tsx
└── package.json
```

## 依存関係のインストール

### 1. Solana関連パッケージ

```bash
# 必須のSolana Mobile Wallet Adapterパッケージ
npm install @solana-mobile/mobile-wallet-adapter-protocol
npm install @solana-mobile/mobile-wallet-adapter-protocol-web3js

# Solana Web3.js
npm install @solana/web3.js

# Expo/React Native用polyfills
npm install expo-crypto
npm install expo-clipboard
npm install @craftzdog/react-native-buffer
npm install react-native-get-random-values
```

### 2. Expo UI関連パッケージ

```bash
# セーフエリア対応
npm install react-native-safe-area-context

# Expo開発ビルド用設定ファイル
npx expo install --fix
```

### 3. Expo設定プラグイン

`app.json`または`app.config.js`に以下の設定を追加：

```json
{
  "expo": {
    "name": "SimpleSolanaMobileApp",
    "slug": "simple-solana-mobile-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "platforms": ["android"],
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "permissions": [
        "android.permission.INTERNET"
      ]
    },
    "plugins": [
      [
        "expo-build-properties",
        {
          "android": {
            "minSdkVersion": 23
          }
        }
      ]
    ]
  }
}
```

### 3. Android権限設定

`android/app/src/main/AndroidManifest.xml`に以下を追加：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- インターネット接続権限 -->
    <uses-permission android:name="android.permission.INTERNET" />
    
    <!-- その他の既存設定... -->
</manifest>
```

## 基本設定ファイル

### 1. Metro設定

`metro.config.js`を以下の内容で作成（Expo用）：

```javascript
const { getDefaultConfig } = require('expo/metro-config');

const config = getDefaultConfig(__dirname);

// Solana用polyfill設定
config.resolver.alias = {
  crypto: '@craftzdog/react-native-buffer',
  stream: 'stream-browserify',
  buffer: '@craftzdog/react-native-buffer',
};

module.exports = config;
```

### 2. Polyfill設定

`App.tsx`の最初に以下のimportを追加：

```typescript
// Polyfills for Solana Web3.js
import 'react-native-get-random-values';
import { Buffer } from '@craftzdog/react-native-buffer';

// グローバルBufferの設定
global.Buffer = global.Buffer || Buffer;

// その他のimport...
import React from 'react';
import { StatusBar } from 'expo-status-bar';
// ... 以下のコンポーネントimport
```

### 3. TypeScript設定

`tsconfig.json`を確認/更新：

```json
{
  "extends": "@tsconfig/react-native/tsconfig.json",
  "compilerOptions": {
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "isolatedModules": true,
    "moduleResolution": "node",
    "strict": true,
    "skipLibCheck": true
  },
  "exclude": [
    "node_modules",
    "babel.config.js",
    "metro.config.js"
  ]
}
```

## プロバイダーコンポーネントの実装

### 1. Connection Provider

`components/providers/ConnectionProvider.tsx`：

```typescript
import React, { ReactNode, createContext, useContext, useMemo } from 'react';
import { Connection, clusterApiUrl } from '@solana/web3.js';

export type Cluster = 'devnet' | 'testnet' | 'mainnet-beta';
export const RPC_ENDPOINT: Cluster = 'devnet';

export interface ConnectionContextState {
  connection: Connection;
}

const ConnectionContext = createContext<ConnectionContextState>({} as ConnectionContextState);

export function useConnection(): ConnectionContextState {
  return useContext(ConnectionContext);
}

interface Props {
  children: ReactNode;
  endpoint?: string;
  config?: any;
}

export function ConnectionProvider({ 
  children, 
  endpoint = clusterApiUrl(RPC_ENDPOINT),
  config = { commitment: 'processed' }
}: Props) {
  const connection = useMemo(() => new Connection(endpoint, config.commitment), [endpoint, config.commitment]);

  return (
    <ConnectionContext.Provider value={{ connection }}>
      {children}
    </ConnectionContext.Provider>
  );
}
```

### 2. Authorization Provider

`components/providers/AuthorizationProvider.tsx`：

```typescript
import React, { ReactNode, createContext, useContext, useState, useCallback } from 'react';
import { PublicKey } from '@solana/web3.js';

export interface Account {
  address: string;
  label?: string;
  publicKey: PublicKey;
}

export interface AuthorizationContextState {
  selectedAccount: Account | null;
  authorize: () => void;
  deauthorize: () => void;
}

const AuthorizationContext = createContext<AuthorizationContextState>({} as AuthorizationContextState);

export function useAuthorization(): AuthorizationContextState {
  return useContext(AuthorizationContext);
}

interface Props {
  children: ReactNode;
}

export function AuthorizationProvider({ children }: Props) {
  const [selectedAccount, setSelectedAccount] = useState<Account | null>(null);

  const authorize = useCallback(() => {
    // 実際のアプリではMobile Wallet Adapterを使用
    console.log('Authorization requested');
  }, []);

  const deauthorize = useCallback(() => {
    setSelectedAccount(null);
  }, []);

  return (
    <AuthorizationContext.Provider value={{ selectedAccount, authorize, deauthorize }}>
      {children}
    </AuthorizationContext.Provider>
  );
}
```

## UI コンポーネントの実装

### 1. ヘッダーコンポーネント

`components/Header.tsx`：

```typescript
import React from 'react';
import { StyleSheet, Text, View } from 'react-native';

export function Header() {
  return (
    <View style={styles.header}>
      <Text style={styles.headerText}>Solana</Text>
      <Text style={styles.headerSubText}>React Native</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  header: {
    backgroundColor: '#9945FF',
    paddingVertical: 40,
    paddingHorizontal: 20,
    alignItems: 'center',
  },
  headerText: {
    fontSize: 32,
    fontWeight: 'bold',
    color: '#ffffff',
    marginBottom: 5,
  },
  headerSubText: {
    fontSize: 18,
    color: '#000000',
    opacity: 0.8,
  },
});
```

### 2. セクションコンポーネント

`components/Section.tsx`：

```typescript
import React from 'react';
import { StyleSheet, Text, View } from 'react-native';

interface Props {
  title: string;
  children: React.ReactNode;
}

export function Section({ title, children }: Props) {
  return (
    <View style={styles.section}>
      <Text style={styles.sectionTitle}>{title}</Text>
      {children}
    </View>
  );
}

const styles = StyleSheet.create({
  section: {
    marginVertical: 8,
    padding: 16,
    backgroundColor: '#f8f9fa',
    borderRadius: 8,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 12,
    color: '#333',
  },
});
```

### 3. MockWallet接続ボタン

`components/MockWalletButton.tsx`：

```typescript
import React, { useState } from 'react';
import { TouchableOpacity, Text, StyleSheet, Alert } from 'react-native';
import { Keypair, PublicKey } from '@solana/web3.js';

type MockAccount = {
  address: string;
  label: string;
  publicKey: PublicKey;
  keypair: Keypair;
};

interface Props {
  onConnect: (account: MockAccount) => void;
}

export default function MockWalletButton({ onConnect }: Props) {
  const [connecting, setConnecting] = useState(false);

  const connectMockWallet = async () => {
    setConnecting(true);
    try {
      const keypair = Keypair.generate();
      const account: MockAccount = {
        address: keypair.publicKey.toString(),
        label: 'Mock Wallet',
        publicKey: keypair.publicKey,
        keypair: keypair,
      };

      onConnect(account);
      Alert.alert('接続成功', 'MockWalletに接続しました');
    } catch (error) {
      console.error('Mock wallet connection error:', error);
      Alert.alert('接続エラー', 'MockWalletの接続に失敗しました');
    } finally {
      setConnecting(false);
    }
  };

  return (
    <TouchableOpacity
      style={[styles.button, connecting && styles.buttonDisabled]}
      onPress={connectMockWallet}
      disabled={connecting}
    >
      <Text style={styles.buttonText}>
        {connecting ? '接続中...' : 'CONNECT MOCK WALLET (NO MWA)'}
      </Text>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  button: {
    backgroundColor: '#007AFF',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginVertical: 8,
  },
  buttonDisabled: {
    backgroundColor: '#6c757d',
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});
```

### 4. SOL送金コンポーネント

`components/MockSendButton.tsx`：

```typescript
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, Alert, StyleSheet } from 'react-native';
import { Connection, PublicKey, Transaction, SystemProgram, LAMPORTS_PER_SOL, sendAndConfirmTransaction } from '@solana/web3.js';
import * as Clipboard from 'expo-clipboard';

type MockAccount = {
  address: string;
  label: string;
  publicKey: PublicKey;
  keypair: any;
};

type Props = {
  mockAccount: MockAccount;
  connection: Connection;
  onSendComplete?: () => void;
};

export default function MockSendButton({ mockAccount, connection, onSendComplete }: Props) {
  const [recipientAddress, setRecipientAddress] = useState('');
  const [amount, setAmount] = useState('');
  const [sending, setSending] = useState(false);

  const pasteFromClipboard = async () => {
    try {
      const clipboardText = await Clipboard.getStringAsync();
      setRecipientAddress(clipboardText);
    } catch (error) {
      Alert.alert('エラー', 'クリップボードから貼り付けできませんでした');
    }
  };

  const sendSOL = async () => {
    if (!recipientAddress || !amount) {
      Alert.alert('エラー', '送付先アドレスと金額を入力してください');
      return;
    }

    const amountInSOL = parseFloat(amount);
    if (isNaN(amountInSOL) || amountInSOL <= 0) {
      Alert.alert('エラー', '有効な金額を入力してください');
      return;
    }

    setSending(true);
    try {
      const toPublicKey = new PublicKey(recipientAddress);
      const balance = await connection.getBalance(mockAccount.publicKey);
      const amountInLamports = amountInSOL * LAMPORTS_PER_SOL;
      
      if (balance < amountInLamports) {
        Alert.alert('エラー', '残高が不足しています');
        setSending(false);
        return;
      }

      const transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: mockAccount.publicKey,
          toPubkey: toPublicKey,
          lamports: amountInLamports,
        })
      );

      const signature = await sendAndConfirmTransaction(
        connection,
        transaction,
        [mockAccount.keypair]
      );

      Alert.alert(
        '送信完了',
        `${amountInSOL} SOLを正常に送信しました\\n\\nトランザクションID:\\n${signature}`
      );

      setRecipientAddress('');
      setAmount('');
      
      if (onSendComplete) {
        onSendComplete();
      }
    } catch (error: any) {
      console.error('送信エラー:', error);
      Alert.alert('送信エラー', error.message || '送信に失敗しました');
    } finally {
      setSending(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.label}>送付先アドレス:</Text>
      <View style={styles.addressInputContainer}>
        <TextInput
          style={styles.addressInput}
          value={recipientAddress}
          onChangeText={setRecipientAddress}
          placeholder="送付先アドレスを入力"
          multiline
        />
        <View style={styles.buttonContainer}>
          <TouchableOpacity style={styles.pasteButton} onPress={pasteFromClipboard}>
            <Text style={styles.pasteButtonText}>貼付</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.selfButton} onPress={() => {
            setRecipientAddress(mockAccount.publicKey.toString());
          }}>
            <Text style={styles.selfButtonText}>自分</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.testButton} onPress={() => {
            setRecipientAddress('11111111111111111111111111111112');
          }}>
            <Text style={styles.testButtonText}>テスト</Text>
          </TouchableOpacity>
        </View>
      </View>

      <Text style={styles.label}>金額 (SOL):</Text>
      <TextInput
        style={styles.amountInput}
        value={amount}
        onChangeText={setAmount}
        placeholder="0.0"
        keyboardType="numeric"
      />

      <TouchableOpacity
        style={[styles.sendButton, sending && styles.sendButtonDisabled]}
        onPress={sendSOL}
        disabled={sending}
      >
        <Text style={styles.sendButtonText}>
          {sending ? '送信中...' : 'SOL送付'}
        </Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 0,
    margin: 0,
  },
  label: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 8,
    color: '#333',
  },
  addressInputContainer: {
    marginBottom: 16,
  },
  addressInput: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 4,
    padding: 12,
    fontSize: 14,
    backgroundColor: '#fff',
    marginBottom: 8,
    minHeight: 50,
  },
  buttonContainer: {
    flexDirection: 'row',
    gap: 8,
  },
  pasteButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 4,
    justifyContent: 'center',
    flex: 1,
  },
  pasteButtonText: {
    color: '#fff',
    fontSize: 14,
    fontWeight: '600',
    textAlign: 'center',
  },
  selfButton: {
    backgroundColor: '#28a745',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 4,
    justifyContent: 'center',
    flex: 1,
  },
  selfButtonText: {
    color: '#fff',
    fontSize: 14,
    fontWeight: '600',
    textAlign: 'center',
  },
  testButton: {
    backgroundColor: '#ffc107',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 4,
    justifyContent: 'center',
    flex: 1,
  },
  testButtonText: {
    color: '#212529',
    fontSize: 14,
    fontWeight: '600',
    textAlign: 'center',
  },
  amountInput: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 4,
    padding: 12,
    fontSize: 16,
    backgroundColor: '#fff',
    marginBottom: 16,
  },
  sendButton: {
    backgroundColor: '#28a745',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 16,
  },
  sendButtonDisabled: {
    backgroundColor: '#6c757d',
  },
  sendButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});
```

## メイン画面の実装

`screens/MainScreen.tsx`：

```typescript
import React, { useCallback, useEffect, useState } from 'react';
import { ScrollView, StyleSheet, Text, View, Button, TouchableOpacity, Alert } from 'react-native';

import { useConnection } from '../components/providers/ConnectionProvider';
import MockWalletButton from '../components/MockWalletButton';
import MockSendButton from '../components/MockSendButton';

export default function MainScreen() {
  const { connection } = useConnection();
  const [mockAccount, setMockAccount] = useState<any>(null);
  const [mockBalance, setMockBalance] = useState<number | null>(null);

  const fetchMockBalance = useCallback(async () => {
    if (mockAccount) {
      try {
        const fetchedBalance = await connection.getBalance(mockAccount.publicKey);
        setMockBalance(fetchedBalance);
      } catch (error) {
        console.error('Mock残高取得エラー:', error);
      }
    }
  }, [connection, mockAccount]);

  useEffect(() => {
    if (mockAccount) {
      fetchMockBalance();
      const interval = setInterval(fetchMockBalance, 10000);
      return () => clearInterval(interval);
    }
  }, [fetchMockBalance, mockAccount]);

  return (
    <ScrollView style={styles.mainContainer} contentContainerStyle={styles.scrollContainer}>
      {mockAccount ? (
        <View style={styles.mockWalletInfo}>
          <Text style={styles.mockWalletTitle}>MockWallet接続済み</Text>
          <Text style={styles.addressLabel}>ウォレットアドレス:</Text>
          <Text style={styles.addressText}>{mockAccount.publicKey.toString()}</Text>
          <TouchableOpacity style={styles.consoleButton} onPress={() => {
              console.log('=== MockWallet Address ===');
              console.log(mockAccount.publicKey.toString());
              console.log('========================');
              Alert.alert(
                'アドレス表示',
                `MockWalletアドレス:\\n\\n${mockAccount.publicKey.toString()}\\n\\nコンソールにも出力しました。\\n\\n外部ウォレットからこのアドレスにテストSOLを送金してください。`,
                [{ text: 'OK' }]
              );
            }}>
              <Text style={styles.consoleButtonText}>📋 詳細表示</Text>
          </TouchableOpacity>
          <Text style={styles.mockBalanceText}>
            残高: {mockBalance !== null ? (mockBalance / 1000000000).toFixed(4) : '取得中...'} SOL
          </Text>
          <MockSendButton
            mockAccount={mockAccount}
            connection={connection}
            onSendComplete={fetchMockBalance}
          />
          <Button title="MockWallet切断" onPress={() => {
            setMockAccount(null);
            setMockBalance(null);
          }} />
        </View>
      ) : (
        <View>
          <MockWalletButton onConnect={setMockAccount} />
        </View>
      )}
      <Text style={styles.clusterText}>Selected cluster: {connection.rpcEndpoint}</Text>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  mainContainer: {
    flex: 1,
  },
  scrollContainer: {
    padding: 16,
    paddingBottom: 50,
  },
  mockWalletInfo: {
    padding: 16,
    backgroundColor: '#f0f0f0',
    borderRadius: 8,
    margin: 16,
  },
  mockWalletTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  addressLabel: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 8,
    color: '#333',
  },
  addressText: {
    fontSize: 12,
    fontFamily: 'monospace',
    marginBottom: 16,
    color: '#333',
  },
  consoleButton: {
    backgroundColor: '#28a745',
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 4,
    alignItems: 'center',
    marginBottom: 16,
  },
  consoleButtonText: {
    color: '#fff',
    fontSize: 14,
    fontWeight: '600',
  },
  mockBalanceText: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#28a745',
    marginBottom: 16,
    textAlign: 'center',
  },
  clusterText: {
    fontSize: 12,
    color: '#666',
    textAlign: 'center',
    marginTop: 16,
    marginBottom: 16,
  },
});
```

## アプリエントリーポイント

`App.tsx`：

```typescript
import {
  ConnectionProvider,
  RPC_ENDPOINT,
} from './components/providers/ConnectionProvider';
import {clusterApiUrl} from '@solana/web3.js';
import React from 'react';
import {SafeAreaView, StyleSheet} from 'react-native';
import {AuthorizationProvider} from './components/providers/AuthorizationProvider';
import {Header} from './components/Header';

import MainScreen from './screens/MainScreen';

export default function App() {
  return (
    <ConnectionProvider
      config={{commitment: 'processed'}}
      endpoint={clusterApiUrl(RPC_ENDPOINT)}>
      <AuthorizationProvider>
        <SafeAreaView style={styles.shell}>
          <Header />
          <MainScreen />
        </SafeAreaView>
      </AuthorizationProvider>
    </ConnectionProvider>
  );
}

const styles = StyleSheet.create({
  shell: {
    height: '100%',
  },
});
```

## ビルドと実行

### 1. 開発用ビルドの作成

Solanaパッケージを使用するため、Expo開発ビルドが必要です：

```bash
# EASにログイン（Expoアカウントが必要）
eas login

# 開発ビルドの作成（Android）
eas build --profile development --platform android

# ビルド完了後、APKをダウンロードしてデバイスにインストール
```

### 2. 開発サーバーの起動

```bash
# Expo開発サーバーを起動
npx expo start --dev-client

# または
npm start
```

### 3. デバイスでの実行

1. **開発ビルドAPKをインストール** - EASで作成したAPKをデバイスにインストール
2. **Expo開発サーバーに接続** - アプリを開いてQRコードをスキャンするかIPアドレスで接続

### 4. デバッグとテスト

```bash
# ログの確認
npx expo start --dev-client

# リロード（開発中）
# デバイスでアプリをシェイクしてデバッグメニューを開く
```

## トラブルシューティング

### 1. 一般的なエラー

#### Expo Metro bundler エラー
```bash
# キャッシュクリア
npx expo start --clear
```

#### EAS ビルドエラー
```bash
# 依存関係の問題の場合
npm install --legacy-peer-deps

# ビルド再実行
eas build --profile development --platform android --clear-cache
```

#### 依存関係エラー
```bash
# node_modules 再インストール
rm -rf node_modules package-lock.json
npm install
```

#### 開発ビルドが見つからない
```bash
# 開発ビルドを再作成
eas build --profile development --platform android

# または、プロファイルを確認
cat eas.json
```

### 2. Solana関連エラー

#### 接続エラー
- **Devnet接続確認:** `https://api.devnet.solana.com` が利用可能か確認
- **残高不足:** Devnet faucetでSOLを取得：`https://faucet.solana.com/`

#### Buffer/Polyfillエラー
- `index.js`のpolyfill設定を確認
- `metro.config.js`のalias設定を確認

### 3. Expo環境診断

```bash
# Expo環境確認
npx expo doctor

# Android環境確認
adb devices

# EAS設定確認
eas build:list
```

### 4. Expo特有の問題

#### Solanaパッケージエラー
- **問題:** Expo Goでは動作しない
- **解決:** 必ずEAS開発ビルドを使用

#### Buffer/Cryptoエラー
- **解決:** `@craftzdog/react-native-buffer`と`expo-crypto`を使用
- **metro.config.js**のalias設定を確認

## 機能拡張のアイデア

### 1. 実際のMobile Wallet Adapter統合

現在のMockWallet機能を実際のMobile Wallet Adapterに置き換え：

```typescript
// 実際のMWA統合例
import { transact } from '@solana-mobile/mobile-wallet-adapter-protocol-web3js';

const realWalletConnect = async () => {
  try {
    const result = await transact(async (wallet) => {
      const authorization = await wallet.authorize({
        cluster: 'devnet',
        identity: { name: 'My Solana App' },
      });
      return authorization;
    });
    // 認証結果の処理
  } catch (error) {
    console.error('MWA connection failed:', error);
  }
};
```

### 2. 追加機能

- **NFTミント機能**
- **SPLトークン送受信**
- **トランザクション履歴**
- **QRコードスキャナー**
- **マルチウォレット対応**

## Expo特有の利点

### 1. 簡単なセットアップ
- **Android Studio不要** - EASがクラウドでビルド
- **複雑な環境設定不要** - Expoが依存関係を管理
- **クロスプラットフォーム** - iOS/Android両対応

### 2. 開発効率
- **ホットリロード** - コード変更が即座に反映
- **デバッグツール** - Expo DevToolsで簡単デバッグ
- **Over-the-Air更新** - アプリストア経由なしで更新

### 3. デプロイメント
```bash
# 本番ビルドの作成
eas build --platform android

# アプリストア提出用ビルド
eas submit --platform android
```

## 注意事項

### 1. Expo制限事項
- **カスタムネイティブコード** - 開発ビルドが必要
- **Expo Go制限** - SolanaパッケージはExpo Goで動作しない
- **ビルド時間** - EASビルドには時間がかかる場合がある

### 2. Expoアカウント
- **EAS使用** - Expoアカウントが必要
- **ビルド制限** - 無料プランには月間ビルド制限あり

## 参考リンク

- [Solana Mobile公式ドキュメント](https://docs.solanamobile.com/)
- [Expo公式ドキュメント](https://docs.expo.dev/)
- [EAS Build](https://docs.expo.dev/build/introduction/)
- [Mobile Wallet Adapter仕様](https://github.com/solana-mobile/mobile-wallet-adapter)
- [Solana Web3.js](https://solana-labs.github.io/solana-web3.js/)

このExpo版マニュアルに従って実装すれば、React NativeやAndroid環境の複雑な設定なしに、今回作成したものと同じSolanaモバイルウォレットアプリを構築できます。