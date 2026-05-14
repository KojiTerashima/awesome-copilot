---
description: 'TypeScript、React、Power Platform integration 向けの Power Apps Code Apps 開発標準とベスト プラクティス'
applyTo: '**/*.{ts,tsx,js,jsx}, **/vite.config.*, **/package.json, **/tsconfig.json, **/power.config.json'
---

# Power Apps Code Apps 開発ガイドライン

Microsoft の公式 best practice と preview capability に従い、TypeScript、React、Power Platform SDK を使って高品質な Power Apps Code Apps を生成するためのガイドラインです。

## プロジェクト コンテキスト

- **Power Apps Code Apps**: Power Platform integration を備えた code-first の web app 開発
- **TypeScript + React**: Vite bundler を使う推奨 frontend stack
- **Power Platform SDK**: connector integration 用の `@microsoft/power-apps` (現行 version ^1.0.3)
- **PAC CLI**: project 管理と deployment のための Power Platform CLI
- **Port 3000**: Power Platform SDK を使う local development に必須
- **Power Apps Premium**: 本番利用時の end-user licensing 要件

## 開発標準

### プロジェクト構造

- 関心の分離が明確な、整理された folder 構造を使います:
  ```
  src/
  ├── components/          # Reusable UI components
  ├── hooks/              # Custom React hooks for Power Platform
  ├── generated/
  │   ├── services/       # Generated connector services (PAC CLI)
  │   └── models/         # Generated TypeScript models (PAC CLI)
  ├── utils/             # Utility functions and helpers
  ├── types/             # TypeScript type definitions
  ├── PowerProvider.tsx  # Power Platform context wrapper
  └── main.tsx          # Application entry point
  ```
- 生成ファイル (`generated/services/`, `generated/models/`) は custom code と分離します
- 一貫した命名規則を使います (file は kebab-case、component は PascalCase)

### TypeScript 構成

- Power Apps SDK 互換性のため、tsconfig.json で `verbatimModuleSyntax: false` を設定します
- 型安全性のため strict mode を有効にし、推奨 tsconfig.json を使います:
  ```json
  {
    "compilerOptions": {
      "target": "ES2020",
      "useDefineForClassFields": true,
      "lib": ["ES2020", "DOM", "DOM.Iterable"],
      "module": "ESNext",
      "skipLibCheck": true,
      "verbatimModuleSyntax": false,
      "moduleResolution": "bundler",
      "allowImportingTsExtensions": true,
      "resolveJsonModule": true,
      "isolatedModules": true,
      "noEmit": true,
      "jsx": "react-jsx",
      "strict": true,
      "noUnusedLocals": true,
      "noUnusedParameters": true,
      "noFallthroughCasesInSwitch": true,
      "baseUrl": ".",
      "paths": {
        "@/*": ["./src/*"]
      }
    }
  }
  ```
- Power Platform connector response には適切な typing を使います
- import を簡潔にするため、`"@": path.resolve(__dirname, "./src")` で path alias を構成します
- app 固有の data structure には interface を定義します
- error boundary と適切な error handling type を実装します

### 高度な Power Platform Integration

#### Custom Control Frameworks (PCF Controls)
- **PCF control を統合する**: Power Apps Component Framework control を Code Apps に埋め込みます
  ```typescript
  // Example: Using custom PCF control for data visualization
  import { PCFControlWrapper } from './components/PCFControlWrapper';

  const MyComponent = () => {
    return (
      <PCFControlWrapper
        controlName="CustomChartControl"
        dataset={chartData}
        configuration={chartConfig}
      />
    );
  };
  ```
- **PCF control communication**: PCF と React 間の event と data binding を処理します
- **Custom control deployment**: PCF control を Code Apps とともに package 化して deploy します

#### Power BI Embedded Analytics
- **Power BI report を埋め込む**: interactive な dashboard と report を統合します
  ```typescript
  import { PowerBIEmbed } from 'powerbi-client-react';

  const DashboardComponent = () => {
    return (
      <PowerBIEmbed
        embedConfig={{
          type: 'report',
          id: reportId,
          embedUrl: embedUrl,
          accessToken: accessToken,
          tokenType: models.TokenType.Aad,
          settings: {
            panes: { filters: { expanded: false, visible: false } }
          }
        }}
      />
    );
  };
  ```
- **動的 report filtering**: Code App context に基づいて Power BI report を filter します
- **Report export functionality**: PDF、Excel、image export を有効にします

#### AI Builder Integration
- **Cognitive services integration**: AI Builder model を使って form processing や object detection を行います
  ```typescript
  // Example: Document processing with AI Builder
  const processDocument = async (file: File) => {
    const formData = new FormData();
    formData.append('file', file);

    const result = await AIBuilderService.ProcessDocument({
      modelId: 'document-processing-model-id',
      document: formData
    });

    return result.extractedFields;
  };
  ```
- **Prediction model**: 業務予測向けの custom AI model を統合します
- **Sentiment analysis**: AI Builder を使って text sentiment を分析します
- **Object detection**: image analysis と object recognition を実装します

#### Power Virtual Agents Integration
- **Chatbot embedding**: Code Apps 内に Power Virtual Agents bot を統合します
  ```typescript
  import { DirectLine } from 'botframework-directlinejs';
  import { WebChat } from 'botframework-webchat';

  const ChatbotComponent = () => {
    const directLine = new DirectLine({
      token: chatbotToken
    });

    return (
      <div style={{ height: '400px', width: '100%' }}>
        <WebChat directLine={directLine} />
      </div>
    );
  };
  ```
- **Context passing**: Code App context を chatbot conversation と共有します
- **Custom bot actions**: bot interaction から Code App function を起動します
- connector operation には PAC CLI が生成した TypeScript service を使います
- Microsoft Entra ID による適切な authentication flow を実装します
- connector consent dialog と permission management を処理します
- PowerProvider 実装パターン (v1.0 では SDK initialization 不要):
  ```typescript
  import type { ReactNode } from "react";

  export default function PowerProvider({ children }: { children: ReactNode }) {
    return <>{children}</>;
  }
  ```
- 正式にサポートされている connector pattern に従います:
  - SQL Server (Azure SQL を含む)
  - SharePoint
  - Office 365 Users/Groups
  - Azure Data Explorer
  - OneDrive for Business
  - Microsoft Teams
  - Dataverse (CRUD operation)

### React パターン

- 新規開発はすべて hooks を使う functional component で行います
- connector operation には適切な loading state と error state を実装します
- 公式 sample で使われているように、必要に応じて Fluent UI React component を検討します
- 適切な場合は data fetching と caching に React Query または SWR を使います
- component composition では React の best practice に従います
- mobile-first approach で responsive design を実装します
- 主要 dependency は公式 sample に従って install します:
  - `@microsoft/power-apps` for Power Platform SDK
  - `@fluentui/react-components` for UI components
  - `concurrently` for parallel script execution (dev dependency)

### データ管理

- 機密データは application code ではなく data source に保存します
- connector operation には generated model を使って型安全性を確保します
- 適切な data validation と sanitization を実装します
- 可能な範囲で offline scenario を適切に処理します
- 頻繁に参照される data は適切に cache します

#### 高度な Dataverse Relationship
- **多対多 relationship**: junction table と relationship service を実装します
  ```typescript
  // Example: User-to-Role many-to-many relationship
  const userRoles = await UserRoleService.getall();
  const filteredRoles = userRoles.filter(ur => ur.userId === currentUser.id);
  ```
- **Polymorphic lookup**: 複数 entity type を参照できる customer field を処理します
  ```typescript
  // Handle polymorphic customer lookup (Account or Contact)
  const customerType = record.customerType; // 'account' or 'contact'
  const customerId = record.customerId;
  const customer = customerType === 'account'
    ? await AccountService.get(customerId)
    : await ContactService.get(customerId);
  ```
- **複雑な relationship query**: 効率的な data retrieval に `$expand` と `$filter` を使います
- **Relationship validation**: relationship constraint 向けの business rule を実装します

### パフォーマンス最適化

- 高コストな計算には React.memo と useMemo を使います
- 大規模 application には code splitting と lazy loading を実装します
- tree shaking で bundle size を最適化します
- API call を最小化する効率的な connector query pattern を使います
- 大量 data set には適切な pagination を実装します

#### Sync Pattern を備えた Offline-First Architecture
- **Service Worker 実装**: offline functionality を有効にします
  ```typescript
  // Example: Service worker registration
  if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
      navigator.serviceWorker.register('/sw.js')
        .then(registration => console.log('SW registered:', registration))
        .catch(error => console.log('SW registration failed:', error));
    });
  }
  ```
- **Local data storage**: offline data persistence に IndexedDB を使います
  ```typescript
  // Example: IndexedDB wrapper for offline storage
  class OfflineDataStore {
    async saveData(key: string, data: any) {
      const db = await this.openDB();
      const transaction = db.transaction(['data'], 'readwrite');
      transaction.objectStore('data').put({ id: key, data, timestamp: Date.now() });
    }

    async loadData(key: string) {
      const db = await this.openDB();
      const transaction = db.transaction(['data'], 'readonly');
      return transaction.objectStore('data').get(key);
    }
  }
  ```
- **Sync conflict resolution**: online 復帰時の data conflict を処理します
- **Background sync**: 定期的な data synchronization を実装します
- **Progressive Web App (PWA)**: app install と offline capability を有効にします

### セキュリティのベスト プラクティス

- secret や機密構成は code に保存しません
- Power Platform の built-in authentication と authorization を使います
- 適切な input validation と sanitization を実装します
- web application 向け OWASP security guideline に従います
- Power Platform の data loss prevention policy を尊重します
- HTTPS のみの通信を実装します

### Error Handling

- React で包括的な error boundary を実装します
- connector 固有の error を適切に処理します
- user には意味のある error message を提供します
- 機密情報を露出させずに error を適切に log します
- 一時的な failure には retry logic を実装します
- network connectivity issue を処理します

### Testing 戦略

- business logic と utility には unit test を書きます
- React component は React Testing Library で test します
- test では Power Platform connector を mock します
- 重要な user flow には integration test を実装します
- test の安全性向上に TypeScript を使います
- error scenario と edge case を test します

### 開発ワークフロー

- project 初期化と connector management には PAC CLI を使います
- team 規模に適した git branching strategy に従います
- 適切な code review process を実装します
- linting と formatting tool (ESLint、Prettier) を使います
- development script は concurrently で構成します:
  - `"dev": "concurrently \"vite\" \"pac code run\""`
  - `"build": "tsc -b && vite build"`
- CI/CD pipeline で自動 test を実装します
- release には semantic versioning を使います

### Deployment と DevOps

- deployment には `npm run build` の後に `pac code push` を使います
- 適切な environment management (dev、test、prod) を実装します
- environment ごとの構成 file を使います
- 可能な場合は blue-green または canary deployment strategy を実装します
- 本番で application performance と error を監視します
- 適切な backup と disaster recovery procedure を実装します

#### Multi-Environment Deployment Pipeline
- **Environment ごとの構成**: dev / test / staging / prod environment を管理します
  ```json
  // Example: environment-specific config files
  // config/development.json
  {
    "powerPlatform": {
      "environmentUrl": "https://dev-env.crm.dynamics.com",
      "apiVersion": "9.2"
    },
    "features": {
      "enableDebugMode": true,
      "enableAnalytics": false
    }
  }
  ```
- **自動 deployment pipeline**: Azure DevOps または GitHub Actions を使います
  ```yaml
  # Example Azure DevOps pipeline step
  - task: PowerPlatformToolInstaller@2
  - task: PowerPlatformSetConnectionVariables@2
    inputs:
      authenticationType: 'PowerPlatformSPN'
      applicationId: '$(AppId)'
      clientSecret: '$(ClientSecret)'
      tenantId: '$(TenantId)'
  - task: PowerPlatformPublishCustomizations@2
  ```
- **Environment promotion**: dev → test → staging → prod の自動 promotion を実装します
- **Rollback strategy**: deployment failure 時に自動 rollback を実装します
- **Configuration management**: environment 固有の secret には Azure Key Vault を使います

## コード品質ガイドライン

### Component 開発

- 明確な props interface を持つ reusable component を作成します
- inheritance より composition を優先します
- TypeScript で適切な prop validation を実装します
- 単一責任原則に従います
- 明確な命名で自己説明的な code を書きます

### State 管理

- 単純な scenario では React 組み込みの state management を使います
- 複雑な state management では Redux Toolkit を検討します
- 適切な state normalization を実装します
- context や state management library を使い、prop drilling を避けます
- 派生 state と計算済み value を効率的に使います

### API Integration

- 一貫性のために PAC CLI 生成 service を使います
- 適切な request / response interceptor を実装します
- authentication token management を処理します
- request deduplication と caching を実装します
- 適切な HTTP status code handling を使います

### Styling と UI

- 一貫した design system または component library を使います
- CSS Grid / Flexbox で responsive design を実装します
- accessibility guideline (WCAG 2.1) に従います
- component styling には CSS-in-JS または CSS modules を使います
- 適切な場合は dark mode support を実装します
- mobile-friendly な user interface を確保します

#### 高度な UI / UX パターン

##### Component Library を使った Design System 実装
- **Component library 構造**: reusable component system を構築します
  ```typescript
  // Example: Design system button component
  interface ButtonProps {
    variant: 'primary' | 'secondary' | 'danger';
    size: 'small' | 'medium' | 'large';
    disabled?: boolean;
    onClick: () => void;
    children: React.ReactNode;
  }

  export const Button: React.FC<ButtonProps> = ({
    variant, size, disabled, onClick, children
  }) => {
    const classes = `btn btn-${variant} btn-${size} ${disabled ? 'btn-disabled' : ''}`;
    return <button className={classes} onClick={onClick} disabled={disabled}>{children}</button>;
  };
  ```
- **Design token**: spacing、color、typography の一貫性を実装します
- **Component documentation**: Storybook で component documentation を整備します

##### Dark Mode と Theming System
- **Theme provider 実装**: 複数 theme をサポートします
  ```typescript
  // Example: Theme context and provider
  const ThemeContext = createContext({
    theme: 'light',
    toggleTheme: () => {}
  });

  export const ThemeProvider: React.FC<{children: ReactNode}> = ({ children }) => {
    const [theme, setTheme] = useState<'light' | 'dark'>('light');

    const toggleTheme = () => {
      setTheme(prev => prev === 'light' ? 'dark' : 'light');
    };

    return (
      <ThemeContext.Provider value={{ theme, toggleTheme }}>
        <div className={`theme-${theme}`}>{children}</div>
      </ThemeContext.Provider>
    );
  };
  ```
- **CSS custom property**: 動的 theming に CSS variable を使います
- **System preference detection**: user の OS theme preference を尊重します

##### Responsive Design の高度なパターン
- **Container query**: container ベースの responsive design を使います
  ```css
  /* Example: Container query for responsive components */
  .card-container {
    container-type: inline-size;
  }

  @container (min-width: 400px) {
    .card {
      display: grid;
      grid-template-columns: 1fr 1fr;
    }
  }
  ```
- **Fluid typography**: responsive な font scaling を実装します
- **Adaptive layout**: screen size と context に応じて layout pattern を変更します

##### Animation と Micro-interaction
- **Framer Motion integration**: 滑らかな animation と transition を実装します
  ```typescript
  import { motion, AnimatePresence } from 'framer-motion';

  const AnimatedCard = () => {
    return (
      <motion.div
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -20 }}
        transition={{ duration: 0.3 }}
        whileHover={{ scale: 1.02 }}
        className="card"
      >
        Card content
      </motion.div>
    );
  };
  ```
- **Loading state**: animated skeleton と progress indicator を使います
- **Gesture recognition**: swipe、pinch、touch interaction を実装します
- **Performance optimization**: CSS transform と `will-change` property を使います

##### Accessibility の自動化と Testing
- **ARIA 実装**: 適切な semantic markup と ARIA attribute を使います
  ```typescript
  // Example: Accessible modal component
  const Modal: React.FC<{isOpen: boolean, onClose: () => void, children: ReactNode}> = ({
    isOpen, onClose, children
  }) => {
    useEffect(() => {
      if (isOpen) {
        document.body.style.overflow = 'hidden';
        const focusableElement = document.querySelector('[data-autofocus]') as HTMLElement;
        focusableElement?.focus();
      }
      return () => { document.body.style.overflow = 'unset'; };
    }, [isOpen]);

    return (
      <div
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        className={isOpen ? 'modal-open' : 'modal-hidden'}
      >
        {children}
      </div>
    );
  };
  ```
- **自動 accessibility testing**: accessibility testing に axe-core を統合します
- **Keyboard navigation**: 完全な keyboard accessibility を実装します
- **Screen reader optimization**: NVDA、JAWS、VoiceOver で test します

##### Internationalization (i18n) と Localization
- **React-intl integration**: 多言語対応を実装します
  ```typescript
  import { FormattedMessage, useIntl } from 'react-intl';

  const WelcomeMessage = ({ userName }: { userName: string }) => {
    const intl = useIntl();

    return (
      <h1>
        <FormattedMessage
          id="welcome.title"
          defaultMessage="Welcome, {userName}!"
          values={{ userName }}
        />
      </h1>
    );
  };
  ```
- **Language detection**: 自動 language detection と切り替えを実装します
- **RTL support**: Arabic、Hebrew 向け右から左の表示をサポートします
- **Date と number formatting**: locale 固有の formatting を使います
- **Translation management**: translation service との integration を実装します

## 現在の制限事項と回避策

### 既知の制限

- Content Security Policy (CSP) はまだ未対応です
- Storage SAS IP restriction は未対応です
- Power Platform Git integration はありません
- Dataverse solution はサポートされていますが、solution packager と source code integration は限定的です
- Application Insights は SDK logger configuration 経由でサポートされます (built-in の native integration はなし)

### 回避策

- 必要に応じて代替の error tracking solution を使います
- 手動 deployment workflow を実装します
- 高度な analytics には external tool を使います
- 将来サポートされる feature への migration を計画します

## Documentation 標準

- setup instruction を含む包括的な README.md を維持します
- すべての custom component と hook を document 化します
- よくある issue の troubleshooting guide を含めます
- deployment procedure と要件を document 化します
- version update 向けに changelog を維持します
- 主要な選択には architectural decision record を含めます

## よくある問題のトラブルシューティング

### 開発時の問題

- **Port 3000 の競合**: `netstat -ano | findstr :3000` で既存 process を特定し、`taskkill /PID {PID} /F` で終了します
- **Authentication failure**: `pac auth list` で environment setup と user permission を確認します
- **Package install failure**: `npm cache clean --force` で npm cache を消去して再 install します
- **TypeScript compile error**: verbatimModuleSyntax 設定と SDK compatibility を確認します
- **Connector permission error**: 適切な consent flow と admin permission を確認します
- **PowerProvider issue**: v1.0 app が SDK initialization を待たないことを確認します
- **Vite dev server issue**: host と port の構成が要件に一致していることを確認します

### Deployment 時の問題

- **Build failure**: `npm audit` で dependency を確認し、build configuration を見直します
- **Authentication error**: `pac auth clear` の後に `pac auth create` で PAC CLI を再認証します
- **Connector unavailable**: Power Platform 内の connector setup と connection status を確認します
- **Performance issue**: `npm run build --report` で bundle size を最適化し、caching を実装します
- **Environment mismatch**: `pac env list` で正しい environment 選択を確認します
- **App timeout error**: build output と network connectivity を確認します

### Runtime 時の問題

- **"App timed out" error**: `npm run build` が実行され、deployment output が有効であることを確認します
- **Connector authentication prompt**: 適切な consent flow 実装を確認します
- **Data loading failure**: network request と connector permission を確認します
- **UI rendering issue**: Fluent UI compatibility と responsive design 実装を確認します

## ベスト プラクティスの要約

1. **Microsoft の公式 documentation と best practice に従う**
2. **型安全性と developer experience 向上のために TypeScript を使う**
3. **適切な error handling と user feedback を実装する**
4. **パフォーマンスと user experience を最適化する**
5. **セキュリティの best practice と Power Platform policy に従う**
6. **保守しやすく、test しやすく、よく document 化された code を書く**
7. **PAC CLI が生成した service と model を使う**
8. **将来の feature update と migration を見据えて計画する**
9. **包括的な testing 戦略を実装する**
10. **適切な DevOps と deployment practice に従う**
