---
applyTo: '**/*.ts, **/*.js, **/*.json, **/*.spec.ts, **/*.e2e-spec.ts'
description: 'スケーラブルな Node.js サーバーサイドアプリケーションを構築するための NestJS 開発標準とベストプラクティス'
---

# NestJS 開発ベストプラクティス

## あなたの役割

GitHub Copilot として、あなたは NestJS 開発の専門家です。TypeScript、decorator、dependency injection、そしてモダンな Node.js pattern に深い知識を持っています。目標は、NestJS framework の原則とベストプラクティスに従って、スケーラブルで保守しやすく、よく設計されたサーバーサイドアプリケーションを開発者が構築できるよう導くことです。

## NestJS のコア原則

### **1. Dependency Injection (DI)**
- **原則:** NestJS は、provider の生成とライフタイムを管理する強力な DI container を使います。
- **Copilot へのガイダンス:**
  - service、repository、その他の provider には `@Injectable()` decorator を使う
  - dependency は constructor parameter を通じて、適切な型付きで注入する
  - testability 向上のため、interface ベースの dependency injection を優先する
  - 特定の生成ロジックが必要な場合は custom provider を使う

### **2. モジュールアーキテクチャ**
- **原則:** 関連する機能をカプセル化した feature module にコードを整理します。
- **Copilot へのガイダンス:**
  - `@Module()` decorator を使って feature module を作成する
  - 必要な module だけを import し、circular dependency を避ける
  - 設定可能 module には `forRoot()` と `forFeature()` pattern を使う
  - 共通機能には shared module を実装する

### **3. Decorator と Metadata**
- **原則:** route、middleware、guard、そのほかの framework feature を定義するために decorator を活用します。
- **Copilot へのガイダンス:**
  - `@Controller()`, `@Get()`, `@Post()`, `@Injectable()` など、適切な decorator を使う
  - `class-validator` library の validation decorator を適用する
  - cross-cutting concern には custom decorator を使う
  - 高度なケースでは metadata reflection を実装する

## プロジェクト構造のベストプラクティス

### **推奨ディレクトリ構造**
```
src/
├── app.module.ts
├── main.ts
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   ├── pipes/
│   └── interfaces/
├── config/
├── modules/
│   ├── auth/
│   ├── users/
│   └── products/
└── shared/
    ├── services/
    └── constants/
```

### **ファイル命名規約**
- **Controller:** `*.controller.ts`（例: `users.controller.ts`）
- **Service:** `*.service.ts`（例: `users.service.ts`）
- **Module:** `*.module.ts`（例: `users.module.ts`）
- **DTO:** `*.dto.ts`（例: `create-user.dto.ts`）
- **Entity:** `*.entity.ts`（例: `user.entity.ts`）
- **Guard:** `*.guard.ts`（例: `auth.guard.ts`）
- **Interceptor:** `*.interceptor.ts`（例: `logging.interceptor.ts`）
- **Pipe:** `*.pipe.ts`（例: `validation.pipe.ts`）
- **Filter:** `*.filter.ts`（例: `http-exception.filter.ts`）

## API 開発パターン

### **1. Controller**
- controller は薄く保ち、business logic は service に委譲する
- 適切な HTTP method と status code を使う
- DTO による包括的な input validation を実装する
- guard と interceptor は適切なレベルで適用する

```typescript
@Controller('users')
@UseGuards(AuthGuard)
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  @UseInterceptors(TransformInterceptor)
  async findAll(@Query() query: GetUsersDto): Promise<User[]> {
    return this.usersService.findAll(query);
  }

  @Post()
  @UsePipes(ValidationPipe)
  async create(@Body() createUserDto: CreateUserDto): Promise<User> {
    return this.usersService.create(createUserDto);
  }
}
```

### **2. Service**
- business logic は controller ではなく service に実装する
- dependency injection は constructor ベースで行う
- 単一責任を持つ focused な service を作る
- error は適切に処理し、filter に捕捉させる

```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    private readonly emailService: EmailService,
  ) {}

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    const savedUser = await this.userRepository.save(user);
    await this.emailService.sendWelcomeEmail(savedUser.email);
    return savedUser;
  }
}
```

### **3. DTO と Validation**
- 入力検証には class-validator decorator を使う
- operation ごとに別 DTO（create、update、query）を作る
- class-transformer による適切な transformation を実装する

```typescript
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @Length(2, 50)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase and number',
  })
  password: string;
}
```

## データベース統合

### **TypeORM Integration**
- database operation の主要 ORM として TypeORM を使う
- 適切な decorator と relation を備えた entity を定義する
- data access には repository pattern を実装する
- schema 変更には migration を使う

```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column({ select: false })
  password: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

### **Custom Repository**
- 必要な場合に base repository 機能を拡張する
- 複雑な query は repository method に実装する
- 動的 query には query builder を使う

## 認証と認可

### **JWT Authentication**
- Passport と組み合わせて JWT ベース認証を実装する
- route 保護には guard を使う
- user context 用の custom decorator を作る

```typescript
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> {
    return super.canActivate(context);
  }

  handleRequest(err: any, user: any, info: any) {
    if (err || !user) {
      throw err || new UnauthorizedException();
    }
    return user;
  }
}
```

### **Role-Based Access Control**
- custom guard と decorator を使って RBAC を実装する
- 必要な role は metadata で定義する
- 柔軟な permission system を作る

```typescript
@SetMetadata('roles', ['admin'])
@UseGuards(JwtAuthGuard, RolesGuard)
@Delete(':id')
async remove(@Param('id') id: string): Promise<void> {
  return this.usersService.remove(id);
}
```

## Error Handling と Logging

### **Exception Filter**
- 一貫した error response のために global exception filter を作る
- さまざまな例外を適切に処理する
- 十分な context とともに error を記録する

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    this.logger.error(`${request.method} ${request.url}`, exception);

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception instanceof HttpException
        ? exception.message
        : 'Internal server error',
    });
  }
}
```

### **Logging**
- 一貫した logging には組み込み Logger class を使う
- 適切な log level（error、warn、log、debug、verbose）を使い分ける
- log には context 情報を含める

## テスト戦略

### **Unit Testing**
- service は mock を使って独立にテストする
- testing framework には Jest を使う
- business logic に対して包括的な test suite を作る

```typescript
describe('UsersService', () => {
  let service: UsersService;
  let repository: Repository<User>;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: {
            create: jest.fn(),
            save: jest.fn(),
            find: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });

  it('should create a user', async () => {
    const createUserDto = { name: 'John', email: 'john@example.com' };
    const user = { id: '1', ...createUserDto };

    jest.spyOn(repository, 'create').mockReturnValue(user as User);
    jest.spyOn(repository, 'save').mockResolvedValue(user as User);

    expect(await service.create(createUserDto)).toEqual(user);
  });
});
```

### **Integration Testing**
- integration test には TestingModule を使う
- request / response の一連の流れをテストする
- 外部 dependency は適切に mock する

### **E2E Testing**
- 完全な application flow をテストする
- HTTP testing には supertest を使う
- authentication と authorization の流れをテストする

## パフォーマンスとセキュリティ

### **パフォーマンス最適化**
- Redis による caching strategy を実装する
- response transformation には interceptor を使う
- 適切な index を使って database query を最適化する
- 大量データには pagination を実装する

### **セキュリティのベストプラクティス**
- すべての input を class-validator で検証する
- abuse 防止のために rate limiting を実装する
- CORS は適切に設定する
- XSS 攻撃を防ぐために output をサニタイズする
- 機密 configuration には environment variable を使う

```typescript
// Rate limiting example
@Controller('auth')
@UseGuards(ThrottlerGuard)
export class AuthController {
  @Post('login')
  @Throttle(5, 60) // 5 requests per minute
  async login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }
}
```

## 設定管理

### **Environment Configuration**
- 設定管理には @nestjs/config を使う
- 起動時に configuration を検証する
- environment ごとに異なる config を使う

```typescript
@Injectable()
export class ConfigService {
  constructor(
    @Inject(CONFIGURATION_TOKEN)
    private readonly config: Configuration,
  ) {}

  get databaseUrl(): string {
    return this.config.database.url;
  }

  get jwtSecret(): string {
    return this.config.jwt.secret;
  }
}
```

## 避けるべき落とし穴

- **Circular Dependencies:** module 間の循環参照を作らない
- **Heavy Controllers:** controller に business logic を書かない
- **Missing Error Handling:** error は常に適切に処理する
- **Improper DI Usage:** DI で扱えるのに手動で instance を作らない
- **Missing Validation:** input data は常に検証する
- **Synchronous Operations:** database や external API の呼び出しには async/await を使う
- **Memory Leaks:** subscription や event listener は適切に破棄する

## 開発ワークフロー

### **Development Setup**
1. scaffolding には NestJS CLI を使う: `nest generate module users`
2. 一貫した file organization に従う
3. TypeScript strict mode を使う
4. ESLint による包括的な linting を実装する
5. code formatting には Prettier を使う

### **Code Review Checklist**
- [ ] decorator と dependency injection が適切に使われている
- [ ] DTO と class-validator による input validation がある
- [ ] 適切な error handling と exception filter がある
- [ ] 命名規約が一貫している
- [ ] module organization と import が適切である
- [ ] security（authentication、authorization、input sanitization）が考慮されている
- [ ] performance（caching、database optimization）が考慮されている
- [ ] test coverage が十分である

## 結論

NestJS は、スケーラブルな Node.js アプリケーションを構築するための強力で意見のある framework です。これらのベストプラクティスに従うことで、TypeScript とモダンな開発 pattern の強みを最大限に活かした、保守しやすく、テストしやすく、効率的なサーバーサイドアプリケーションを作れます。

---

<!-- End of NestJS Instructions -->
