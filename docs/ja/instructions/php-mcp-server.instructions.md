---
description: 'attribute ベースの discovery と複数の transport option を備えた公式 PHP SDK を使って Model Context Protocol サーバーを構築するためのベスト プラクティス'
applyTo: '**/*.php'
---

# PHP MCP Server 開発のベスト プラクティス

このガイドは、The PHP Foundation との協力のもと保守されている公式 PHP SDK を使って Model Context Protocol (MCP) server を構築するための best practice を提供します。

## インストールとセットアップ

### Composer でインストールする

```bash
composer require mcp/sdk
```

### プロジェクト構造

PHP MCP server project は次のように整理します。

```
my-mcp-server/
├── composer.json
├── src/
│   ├── Tools/
│   │   ├── Calculator.php
│   │   └── FileManager.php
│   ├── Resources/
│   │   ├── ConfigProvider.php
│   │   └── DataProvider.php
│   ├── Prompts/
│   │   └── PromptGenerator.php
│   └── Server.php
├── server.php           # Server entry point
└── tests/
    └── ToolsTest.php
```

### Composer 構成

```json
{
    "name": "your-org/mcp-server",
    "description": "MCP Server for...",
    "type": "project",
    "require": {
        "php": "^8.2",
        "mcp/sdk": "^0.1"
    },
    "require-dev": {
        "phpunit/phpunit": "^10.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

## Server 実装

### Attribute Discovery を使う基本 Server

server の entry point を作成します。

```php
#!/usr/bin/env php
<?php

declare(strict_types=1);

require_once __DIR__ . '/vendor/autoload.php';

use Mcp\Server;
use Mcp\Server\Transport\StdioTransport;

$server = Server::builder()
    ->setServerInfo('My MCP Server', '1.0.0')
    ->setDiscovery(__DIR__, ['.'])
    ->build();

$transport = new StdioTransport();

$server->run($transport);
```

### Caching を使う Server

より良いパフォーマンスのために PSR-16 cache を使います。

```php
use Symfony\Component\Cache\Adapter\FilesystemAdapter;
use Symfony\Component\Cache\Psr16Cache;

$cache = new Psr16Cache(new FilesystemAdapter('mcp-discovery'));

$server = Server::builder()
    ->setServerInfo('My MCP Server', '1.0.0')
    ->setDiscovery(
        basePath: __DIR__,
        scanDirs: ['.', 'src'],
        excludeDirs: ['vendor', 'tests'],
        cache: $cache
    )
    ->build();
```

### 手動登録

capability をプログラムで登録します。

```php
use App\Tools\Calculator;
use App\Resources\Config;

$server = Server::builder()
    ->setServerInfo('My MCP Server', '1.0.0')
    ->addTool([Calculator::class, 'add'], 'add')
    ->addTool([Calculator::class, 'multiply'], 'multiply')
    ->addResource([Config::class, 'getSettings'], 'config://app/settings')
    ->build();
```

## Tool 開発

### Attribute を使うシンプルな Tool

```php
<?php

namespace App\Tools;

use Mcp\Capability\Attribute\McpTool;

class Calculator
{
    /**
     * Adds two numbers together.
     *
     * @param int $a The first number
     * @param int $b The second number
     * @return int The sum of the two numbers
     */
    #[McpTool]
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }
}
```

### カスタム名を持つ Tool

```php
use Mcp\Capability\Attribute\McpTool;

class FileManager
{
    /**
     * Reads file content from the filesystem.
     */
    #[McpTool(name: 'read_file')]
    public function readFileContent(string $path): string
    {
        if (!file_exists($path)) {
            throw new \InvalidArgumentException("File not found: {$path}");
        }

        return file_get_contents($path);
    }
}
```

### Validation と Schema を持つ Tool

```php
use Mcp\Capability\Attribute\{McpTool, Schema};

class UserManager
{
    #[McpTool(name: 'create_user')]
    public function createUser(
        #[Schema(format: 'email')]
        string $email,

        #[Schema(minimum: 18, maximum: 120)]
        int $age,

        #[Schema(
            pattern: '^[A-Z][a-z]+$',
            description: 'Capitalized first name'
        )]
        string $firstName
    ): array
    {
        return [
            'id' => uniqid(),
            'email' => $email,
            'age' => $age,
            'firstName' => $firstName
        ];
    }
}
```

### 複雑な Return Type を持つ Tool

```php
use Mcp\Schema\Content\{TextContent, ImageContent};

class ReportGenerator
{
    #[McpTool]
    public function generateReport(string $type): array
    {
        return [
            new TextContent('Report generated:'),
            TextContent::code($this->generateCode($type), 'php'),
            new TextContent('Summary: All checks passed.')
        ];
    }

    #[McpTool]
    public function getChart(string $chartType): ImageContent
    {
        $imageData = $this->generateChartImage($chartType);

        return new ImageContent(
            data: base64_encode($imageData),
            mimeType: 'image/png'
        );
    }
}
```

### Match 式を使う Tool

```php
#[McpTool(name: 'calculate')]
public function performCalculation(float $a, float $b, string $operation): float
{
    return match($operation) {
        'add' => $a + $b,
        'subtract' => $a - $b,
        'multiply' => $a * $b,
        'divide' => $b != 0 ? $a / $b :
            throw new \InvalidArgumentException('Division by zero'),
        default => throw new \InvalidArgumentException('Invalid operation')
    };
}
```

## Resource 実装

### Static Resource

```php
<?php

namespace App\Resources;

use Mcp\Capability\Attribute\McpResource;

class ConfigProvider
{
    /**
     * Provides the current application configuration.
     */
    #[McpResource(
        uri: 'config://app/settings',
        name: 'app_settings',
        mimeType: 'application/json'
    )]
    public function getSettings(): array
    {
        return [
            'version' => '1.0.0',
            'debug' => false,
            'features' => ['auth', 'logging']
        ];
    }
}
```

### 変数付き Resource Template

```php
use Mcp\Capability\Attribute\McpResourceTemplate;

class UserProvider
{
    /**
     * Retrieves user profile information by ID and section.
     */
    #[McpResourceTemplate(
        uriTemplate: 'user://{userId}/profile/{section}',
        name: 'user_profile',
        description: 'User profile data by section',
        mimeType: 'application/json'
    )]
    public function getUserProfile(string $userId, string $section): array
    {
        // Variable order must match URI template order
        return $this->users[$userId][$section] ??
            throw new \InvalidArgumentException("Profile section not found");
    }
}
```

### File Content を返す Resource

```php
use Mcp\Schema\Content\{TextResourceContents, BlobResourceContents};

class FileProvider
{
    #[McpResource(uri: 'file://readme.txt', mimeType: 'text/plain')]
    public function getReadme(): TextResourceContents
    {
        return new TextResourceContents(
            uri: 'file://readme.txt',
            mimeType: 'text/plain',
            text: file_get_contents(__DIR__ . '/README.txt')
        );
    }

    #[McpResource(uri: 'file://image.png', mimeType: 'image/png')]
    public function getImage(): BlobResourceContents
    {
        $imageData = file_get_contents(__DIR__ . '/image.png');

        return new BlobResourceContents(
            uri: 'file://image.png',
            mimeType: 'image/png',
            blob: base64_encode($imageData)
        );
    }
}
```

## Prompt 実装

### 基本 Prompt

```php
<?php

namespace App\Prompts;

use Mcp\Capability\Attribute\McpPrompt;

class PromptGenerator
{
    /**
     * Generates a code review request prompt.
     */
    #[McpPrompt(name: 'code_review')]
    public function reviewCode(string $language, string $code, string $focus = 'general'): array
    {
        return [
            ['role' => 'assistant', 'content' => 'You are an expert code reviewer.'],
            ['role' => 'user', 'content' => "Review this {$language} code focusing on {$focus}:\n\n```{$language}\n{$code}\n```"]
        ];
    }
}
```

### Mixed Content を持つ Prompt

```php
use Mcp\Schema\Content\{TextContent, ImageContent};
use Mcp\Schema\PromptMessage;
use Mcp\Schema\Enum\Role;

#[McpPrompt]
public function analyzeImage(string $imageUrl, string $question): array
{
    $imageData = file_get_contents($imageUrl);

    return [
        new PromptMessage(Role::Assistant, [
            new TextContent('You are an image analysis expert.')
        ]),
        new PromptMessage(Role::User, [
            new TextContent($question),
            new ImageContent(
                data: base64_encode($imageData),
                mimeType: 'image/jpeg'
            )
        ])
    ];
}
```

## Completion Provider

### Value List Completion

```php
use Mcp\Capability\Attribute\{McpPrompt, CompletionProvider};

#[McpPrompt]
public function generateContent(
    #[CompletionProvider(values: ['blog', 'article', 'tutorial', 'guide'])]
    string $contentType,

    #[CompletionProvider(values: ['beginner', 'intermediate', 'advanced'])]
    string $difficulty
): array
{
    return [
        ['role' => 'user', 'content' => "Create a {$difficulty} level {$contentType}"]
    ];
}
```

### Enum Completion

```php
enum Priority: string
{
    case LOW = 'low';
    case MEDIUM = 'medium';
    case HIGH = 'high';
}

enum Status
{
    case DRAFT;
    case PUBLISHED;
    case ARCHIVED;
}

#[McpResourceTemplate(uriTemplate: 'tasks/{taskId}')]
public function getTask(
    string $taskId,

    #[CompletionProvider(enum: Priority::class)]
    string $priority,

    #[CompletionProvider(enum: Status::class)]
    string $status
): array
{
    return $this->tasks[$taskId] ?? [];
}
```

### Custom Completion Provider

```php
use Mcp\Capability\Prompt\Completion\ProviderInterface;

class UserIdCompletionProvider implements ProviderInterface
{
    public function __construct(
        private DatabaseService $db
    ) {}

    public function getCompletions(string $currentValue): array
    {
        return $this->db->searchUserIds($currentValue);
    }
}

#[McpResourceTemplate(uriTemplate: 'user://{userId}/profile')]
public function getUserProfile(
    #[CompletionProvider(provider: UserIdCompletionProvider::class)]
    string $userId
): array
{
    return $this->users[$userId] ??
        throw new \InvalidArgumentException('User not found');
}
```

## Transport Option

### Stdio Transport

command-line integration 向け (既定):

```php
use Mcp\Server\Transport\StdioTransport;

$transport = new StdioTransport();
$server->run($transport);
```

### HTTP Transport

web ベースの integration 向け:

```php
use Mcp\Server\Transport\StreamableHttpTransport;
use Nyholm\Psr7\Factory\Psr17Factory;

$psr17Factory = new Psr17Factory();

$request = $psr17Factory->createServerRequestFromGlobals();

$transport = new StreamableHttpTransport(
    $request,
    $psr17Factory,  // Response factory
    $psr17Factory   // Stream factory
);

$response = $server->run($transport);

// Send response in your web framework
foreach ($response->getHeaders() as $name => $values) {
    foreach ($values as $value) {
        header("$name: $value", false);
    }
}

http_response_code($response->getStatusCode());
echo $response->getBody();
```

## Session 管理

### In-Memory Session (既定)

```php
$server = Server::builder()
    ->setServerInfo('My Server', '1.0.0')
    ->setSession(ttl: 7200) // 2 hours
    ->build();
```

### File-Based Session

```php
use Mcp\Server\Session\FileSessionStore;

$server = Server::builder()
    ->setServerInfo('My Server', '1.0.0')
    ->setSession(new FileSessionStore(__DIR__ . '/sessions'))
    ->build();
```

### Custom Session Store

```php
use Mcp\Server\Session\InMemorySessionStore;

$server = Server::builder()
    ->setServerInfo('My Server', '1.0.0')
    ->setSession(new InMemorySessionStore(3600))
    ->build();
```

## Error Handling

### Tool 内の例外処理

```php
#[McpTool]
public function divideNumbers(float $a, float $b): float
{
    if ($b === 0.0) {
        throw new \InvalidArgumentException('Division by zero is not allowed');
    }

    return $a / $b;
}

#[McpTool]
public function processFile(string $filename): string
{
    if (!file_exists($filename)) {
        throw new \InvalidArgumentException("File not found: {$filename}");
    }

    if (!is_readable($filename)) {
        throw new \RuntimeException("File not readable: {$filename}");
    }

    return file_get_contents($filename);
}
```

### カスタム Error Response

SDK は例外を自動的に、MCP client が理解できる JSON-RPC error response に変換します。

## Testing

### Tool 向け PHPUnit Test

```php
<?php

namespace Tests;

use PHPUnit\Framework\TestCase;
use App\Tools\Calculator;

class CalculatorTest extends TestCase
{
    private Calculator $calculator;

    protected function setUp(): void
    {
        $this->calculator = new Calculator();
    }

    public function testAdd(): void
    {
        $result = $this->calculator->add(5, 3);
        $this->assertSame(8, $result);
    }

    public function testDivideByZero(): void
    {
        $this->expectException(\InvalidArgumentException::class);
        $this->expectExceptionMessage('Division by zero');

        $this->calculator->divide(10, 0);
    }
}
```

### Server Discovery のテスト

```php
public function testServerDiscoversTools(): void
{
    $server = Server::builder()
        ->setServerInfo('Test Server', '1.0.0')
        ->setDiscovery(__DIR__ . '/../src', ['.'])
        ->build();

    $capabilities = $server->getCapabilities();

    $this->assertArrayHasKey('tools', $capabilities);
    $this->assertNotEmpty($capabilities['tools']);
}
```

## パフォーマンスのベスト プラクティス

### Discovery Caching を使う

本番では常に caching を使ってください。

```php
use Symfony\Component\Cache\Adapter\RedisAdapter;
use Symfony\Component\Cache\Psr16Cache;

$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);

$cache = new Psr16Cache(new RedisAdapter($redis));

$server = Server::builder()
    ->setServerInfo('My Server', '1.0.0')
    ->setDiscovery(
        basePath: __DIR__,
        scanDirs: ['src'],
        excludeDirs: ['vendor', 'tests', 'var'],
        cache: $cache
    )
    ->build();
```

### Scan Directory を最適化する

必要な directory だけを scan します。

```php
$server = Server::builder()
    ->setDiscovery(
        basePath: __DIR__,
        scanDirs: ['src/Tools', 'src/Resources'],  // Specific dirs
        excludeDirs: ['vendor', 'tests', 'var', 'cache']
    )
    ->build();
```

### OPcache を使う

より良い PHP パフォーマンスのため、本番で OPcache を有効にします。

```ini
; php.ini
opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=10000
opcache.validate_timestamps=0
```

## Framework Integration

### Laravel Integration

```php
// app/Console/Commands/McpServer.php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use Mcp\Server;
use Mcp\Server\Transport\StdioTransport;

class McpServer extends Command
{
    protected $signature = 'mcp:serve';
    protected $description = 'Start MCP server';

    public function handle()
    {
        $server = Server::builder()
            ->setServerInfo('Laravel MCP Server', '1.0.0')
            ->setDiscovery(app_path(), ['Tools', 'Resources'])
            ->build();

        $transport = new StdioTransport();
        $server->run($transport);
    }
}
```

### Symfony Integration

```php
// Use symfony/mcp-bundle for native integration
composer require symfony/mcp-bundle
```

## Deployment

### Docker Deployment

```dockerfile
FROM php:8.2-cli

# Install extensions
RUN docker-php-ext-install pdo pdo_mysql

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /app

# Copy application
COPY . /app

# Install dependencies
RUN composer install --no-dev --optimize-autoloader

# Make server executable
RUN chmod +x /app/server.php

CMD ["php", "/app/server.php"]
```

### Systemd Service

```ini
[Unit]
Description=MCP PHP Server
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/mcp-server
ExecStart=/usr/bin/php /var/www/mcp-server/server.php
Restart=always

[Install]
WantedBy=multi-user.target
```

## MCP Client 向け構成

### Claude Desktop 構成

```json
{
  "mcpServers": {
    "php-server": {
      "command": "php",
      "args": ["/absolute/path/to/server.php"]
    }
  }
}
```

### MCP Inspector Testing

```bash
npx @modelcontextprotocol/inspector php /path/to/server.php
```

## 追加リソース

- [Official PHP SDK Repository](https://github.com/modelcontextprotocol/php-sdk)
- [MCP Elements Documentation](https://github.com/modelcontextprotocol/php-sdk/blob/main/docs/mcp-elements.md)
- [Server Builder Documentation](https://github.com/modelcontextprotocol/php-sdk/blob/main/docs/server-builder.md)
- [Transport Documentation](https://github.com/modelcontextprotocol/php-sdk/blob/main/docs/transports.md)
- [Examples](https://github.com/modelcontextprotocol/php-sdk/blob/main/docs/examples.md)
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
