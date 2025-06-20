# Solana Mobile Wallet App 作成マニュアル

React Native初心者向けの詳細なSolanaモバイルウォレットアプリ構築ガイドです。このマニュアルに従って作業すれば、全く同じアプリを作成できます。

## 目次

1. [環境構築](#環境構築)
2. [プロジェクト作成](#プロジェクト作成)
3. [依存関係のインストール](#依存関係のインストール)
4. [基本ファイル構成](#基本ファイル構成)
5. [コンポーネント作成](#コンポーネント作成)
6. [メイン画面の実装](#メイン画面の実装)
7. [スタイリング](#スタイリング)
8. [ビルドと実行](#ビルドと実行)
9. [トラブルシューティング](#トラブルシューティング)

## 環境構築

### 1. 必要なソフトウェアのインストール

#### Node.js (バージョン 18以上推奨)
```bash
# Node.jsのバージョン確認
node --version

# npmのバージョン確認
npm --version
```

#### React Native CLI
```bash
# React Native CLIをグローバルにインストール
npm install -g @react-native-community/cli
```

#### 開発環境（以下のいずれかを選択）

**Android開発環境:**
- Android Studio
- Android SDK
- Java Development Kit (JDK) 11

**iOS開発環境（Macのみ）:**
- Xcode 14以上
- iOS Simulator

### 2. Android開発環境の詳細設定

#### Android Studioのインストール
1. [Android Studio](https://developer.android.com/studio)をダウンロード
2. インストーラーを実行し、標準インストールを選択
3. SDK Manager で以下をインストール：
   - Android SDK Platform 33
   - Android SDK Build-Tools
   - Android Emulator

#### 環境変数の設定
```bash
# ~/.bashrc または ~/.zshrc に追加
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

変更後、ターミナルを再起動するか以下を実行：
```bash
source ~/.bashrc  # または source ~/.zshrc
```

## プロジェクト作成

### 1. 新しいReact Nativeプロジェクトを作成

```bash
# プロジェクト作成（TypeScript使用）
npx react-native init SimpleSolanaMobileApp --template react-native-template-typescript

# プロジェクトディレクトリに移動
cd SimpleSolanaMobileApp
```

### 2. プロジェクト構造の確認

作成されたプロジェクトの基本構造：
```
SimpleSolanaMobileApp/
├── android/          # Androidプロジェクトファイル
├── ios/             # iOSプロジェクトファイル
├── src/             # ソースコード（作成予定）
├── App.tsx          # メインアプリファイル
├── index.js         # エントリーポイント
└── package.json     # 依存関係とスクリプト
```

## 依存関係のインストール

### 1. Solana関連パッケージ

```bash
# Solana Web3.js
npm install @solana/web3.js

# Solana Mobile Wallet Adapter
npm install @solana-mobile/mobile-wallet-adapter-protocol-web3js
npm install @solana-mobile/mobile-wallet-adapter-protocol

# Buffer polyfill（React Nativeで必要）
npm install buffer
```

### 2. React Native関連パッケージ

```bash
# クリップボード機能
npm install @react-native-clipboard/clipboard

# AsyncStorage（データ永続化）
npm install @react-native-async-storage/async-storage

# 開発用依存関係
npm install --save-dev @types/buffer
```

### 3. Metro設定（重要）

`metro.config.js`ファイルを以下の内容で作成または更新：

```javascript
const {getDefaultConfig} = require('metro-config');

module.exports = (async () => {
  const {
    resolver: {sourceExts, assetExts},
  } = await getDefaultConfig();
  return {
    transformer: {
      getTransformOptions: async () => ({
        transform: {
          experimentalImportSupport: false,
          inlineRequires: true,
        },
      }),
    },
    resolver: {
      assetExts: assetExts.filter(ext => ext !== 'svg'),
      sourceExts: [...sourceExts, 'svg'],
      alias: {
        crypto: 'react-native-crypto',
      },
    },
  };
})();
```

### 4. Polyfillの設定

`index.js`ファイルを以下のように更新：

```javascript
import {AppRegistry} from 'react-native';
import App from './App';
import {name as appName} from './app.json';

// Buffer polyfill
import {Buffer} from 'buffer';
global.Buffer = global.Buffer || Buffer;

AppRegistry.registerComponent(appName, () => App);
```

## 基本ファイル構成

### 1. ディレクトリ構造を作成

```bash
# プロジェクトルートで実行
mkdir components
mkdir screens
mkdir util
```

最終的なディレクトリ構造：
```
SimpleSolanaMobileApp/
├── components/
│   ├── providers/
│   ├── ConnectButton.tsx
│   ├── MockWalletButton.tsx
│   ├── MockSendButton.tsx
│   └── その他のコンポーネント
├── screens/
│   └── MainScreen.tsx
├── util/
│   └── alertAndLog.ts
└── App.tsx
```

### 2. 設定ファイルの作成

#### `tsconfig.json`（TypeScript設定）
```json
{
  "extends": "@tsconfig/react-native/tsconfig.json",
  "compilerOptions": {
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "isolatedModules": true,
    "moduleResolution": "node",
    "strict": true,
    "declaration": false,
    "allowUnreachableCode": false,
    "allowUnusedLabels": false,
    "exactOptionalPropertyTypes": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitOverride": true,
    "noImplicitReturns": true,
    "noPropertyAccessFromIndexSignature": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  },
  "exclude": [
    "node_modules",
    "babel.config.js",
    "metro.config.js",
    "jest.config.js"
  ]
}
```

## コンポーネント作成

### 1. ユーティリティ関数

#### `util/alertAndLog.ts`
```typescript
import { Alert } from 'react-native';

export function alertAndLog(title: string, message: string, error?: any) {
  if (error) {
    console.error(title, message, error);
  } else {
    console.log(title, message);
  }
  Alert.alert(title, message);
}
```

### 2. プロバイダーコンポーネント

#### `components/providers/ConnectionProvider.tsx`
```typescript
import React, { ReactNode, createContext, useContext, useMemo } from 'react';
import { Connection, clusterApiUrl } from '@solana/web3.js';

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
}

export function ConnectionProvider({ children, endpoint = clusterApiUrl('devnet') }: Props) {
  const connection = useMemo(() => new Connection(endpoint, 'confirmed'), [endpoint]);

  return (
    <ConnectionContext.Provider value={{ connection }}>
      {children}
    </ConnectionContext.Provider>
  );
}
```

#### `components/providers/AuthorizationProvider.tsx`
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
    // 実際のアプリでは、ここでMobile Wallet Adapterを使用
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

### 3. UI コンポーネント

#### `components/MockWalletButton.tsx`
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
      // 新しいキーペアを生成（実際のアプリでは既存のウォレットを使用）
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

#### `components/MockSendButton.tsx`
```typescript
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, Alert, StyleSheet } from 'react-native';
import { Connection, PublicKey, Transaction, SystemProgram, LAMPORTS_PER_SOL, sendAndConfirmTransaction } from '@solana/web3.js';
import Clipboard from '@react-native-clipboard/clipboard';

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
      const clipboardText = await Clipboard.getString();
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
      // 送付先アドレスの検証
      const toPublicKey = new PublicKey(recipientAddress);
      
      // 残高確認
      const balance = await connection.getBalance(mockAccount.publicKey);
      const amountInLamports = amountInSOL * LAMPORTS_PER_SOL;
      
      if (balance < amountInLamports) {
        Alert.alert('エラー', '残高が不足しています');
        setSending(false);
        return;
      }

      // トランザクション作成
      const transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: mockAccount.publicKey,
          toPubkey: toPublicKey,
          lamports: amountInLamports,
        })
      );

      // トランザクション送信
      const signature = await sendAndConfirmTransaction(
        connection,
        transaction,
        [mockAccount.keypair]
      );

      Alert.alert(
        '送信完了',
        `${amountInSOL} SOLを正常に送信しました\n\nトランザクションID:\n${signature}`
      );

      // フォームをリセット
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
            // テスト用の有効なdevnetアドレス
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

### `screens/MainScreen.tsx`

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
      // 10秒ごとに自動更新
      const interval = setInterval(fetchMockBalance, 10000);
      return () => clearInterval(interval);
    }
  }, [fetchMockBalance, mockAccount]);

  return (
    <>
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
                  `MockWalletアドレス:\n\n${mockAccount.publicKey.toString()}\n\nコンソールにも出力しました。\n\n外部ウォレットからこのアドレスにテストSOLを送金してください。`,
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
    </>
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

### メインアプリファイル `App.tsx`

```typescript
import React from 'react';
import { StatusBar, StyleSheet, Text, View } from 'react-native';

import { ConnectionProvider } from './components/providers/ConnectionProvider';
import { AuthorizationProvider } from './components/providers/AuthorizationProvider';
import MainScreen from './screens/MainScreen';

function App(): JSX.Element {
  return (
    <ConnectionProvider>
      <AuthorizationProvider>
        <StatusBar barStyle="light-content" backgroundColor="#6a4c93" />
        <View style={styles.shell}>
          <View style={styles.header}>
            <Text style={styles.headerText}>Solana</Text>
            <Text style={styles.headerSubText}>React Native</Text>
          </View>
          <MainScreen />
        </View>
      </AuthorizationProvider>
    </ConnectionProvider>
  );
}

const styles = StyleSheet.create({
  shell: {
    flex: 1,
    backgroundColor: '#ffffff',
  },
  header: {
    backgroundColor: 'linear-gradient(135deg, #6a4c93 0%, #00d4aa 100%)',
    paddingVertical: 40,
    paddingHorizontal: 20,
    alignItems: 'center',
    // グラデーション効果のため、実際には複数の色を重ねる
    background: 'linear-gradient(135deg, #6a4c93 0%, #00d4aa 100%)',
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

export default App;
```

## ビルドと実行

### 1. 依存関係の最終確認

```bash
# package.jsonの内容確認
cat package.json

# node_modulesのクリーンアップ（必要に応じて）
rm -rf node_modules
npm install
```

### 2. Android実行

```bash
# Androidエミュレーターまたは実機の接続確認
adb devices

# Metro bundlerの起動（別ターミナル）
npx react-native start

# Androidアプリのビルドと実行
npx react-native run-android
```

### 3. iOS実行（Macのみ）

```bash
# CocoaPodsの依存関係インストール
cd ios && pod install && cd ..

# iOSアプリのビルドと実行
npx react-native run-ios
```

## トラブルシューティング

### よくある問題と解決方法

#### 1. Metro bundlerエラー
```bash
# キャッシュクリア
npx react-native start --reset-cache
```

#### 2. Android gradleエラー
```bash
# gradleクリーンアップ
cd android
./gradlew clean
cd ..
```

#### 3. Bufferエラー
`index.js`にBuffer polyfillが正しく設定されているか確認：
```javascript
import {Buffer} from 'buffer';
global.Buffer = global.Buffer || Buffer;
```

#### 4. TypeScriptエラー
```bash
# TypeScript型チェック
npx tsc --noEmit
```

### 開発のヒント

1. **デバッグ方法:**
   - React Native Debuggerを使用
   - `console.log`でログ出力
   - ChromeのDeveloper Toolsでリモートデバッグ

2. **ホットリロード:**
   - 開発中は`Ctrl+M`（Android）または`Cmd+D`（iOS）でデバッグメニューを開く
   - "Enable Fast Refresh"を有効にする

3. **パフォーマンス最適化:**
   - `React.memo`でコンポーネントの再レンダリングを最適化
   - `useCallback`と`useMemo`で計算結果をキャッシュ

### 次のステップ

1. **実際のWallet Adapterの統合**
2. **UIの改善とカスタマイズ**
3. **エラーハンドリングの強化**
4. **テストの追加**
5. **アプリストアへのデプロイ**

このマニュアルに従って作業すれば、基本的なSolanaモバイルウォレットアプリを構築できます。不明な点があれば、各セクションを再確認してください。