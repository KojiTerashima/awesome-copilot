---
description: '公式 MCP Ruby SDK gem を使って Ruby で Model Context Protocol (MCP) サーバーを構築するためのベスト プラクティスとパターン。'
applyTo: "**/*.rb, **/Gemfile, **/*.gemspec, **/Rakefile"
---

# Ruby MCP Server Development Guidelines

Ruby で MCP server を構築するときは、公式 Ruby SDK を使って次の best practice と pattern に従ってください。

## Installation

Gemfile に MCP gem を追加します。

```ruby
gem 'mcp'
```

次に実行します。

```bash
bundle install
```

## Server Setup

MCP server instance を作成します。

```ruby
require 'mcp'

server = MCP::Server.new(
  name: 'my_server',
  version: '1.0.0'
)
```

## Adding Tools

class または block を使って tool を定義します。

### Tool as Class

```ruby
class GreetTool < MCP::Tool
  tool_name 'greet'
  description 'Generate a greeting message'

  input_schema(
    properties: {
      name: { type: 'string', description: 'Name to greet' }
    },
    required: ['name']
  )

  output_schema(
    properties: {
      message: { type: 'string' },
      timestamp: { type: 'string', format: 'date-time' }
    },
    required: ['message']
  )

  annotations(
    read_only_hint: true,
    idempotent_hint: true
  )

  def self.call(name:, server_context:)
    MCP::Tool::Response.new([{
      type: 'text',
      text: "Hello, #{name}! Welcome to MCP."
    }], structured_content: {
      message: "Hello, #{name}!",
      timestamp: Time.now.iso8601
    })
  end
end

server = MCP::Server.new(
  name: 'my_server',
  tools: [GreetTool]
)
```

### Tool with Block

```ruby
server.define_tool(
  name: 'calculate',
  description: 'Perform mathematical calculations',
  input_schema: {
    properties: {
      operation: { type: 'string', enum: ['add', 'subtract', 'multiply', 'divide'] },
      a: { type: 'number' },
      b: { type: 'number' }
    },
    required: ['operation', 'a', 'b']
  },
  annotations: {
    read_only_hint: true,
    idempotent_hint: true
  }
) do |args, server_context|
  operation = args['operation']
  a = args['a']
  b = args['b']

  result = case operation
  when 'add' then a + b
  when 'subtract' then a - b
  when 'multiply' then a * b
  when 'divide'
    return MCP::Tool::Response.new([{ type: 'text', text: 'Division by zero' }], is_error: true) if b == 0
    a / b
  else
    return MCP::Tool::Response.new([{ type: 'text', text: "Unknown operation: #{operation}" }], is_error: true)
  end

  MCP::Tool::Response.new([{ type: 'text', text: "Result: #{result}" }])
end
```

## Adding Resources

データ access 用の resource を定義します。

```ruby
# Register resources
resource = MCP::Resource.new(
  uri: 'resource://data/example',
  name: 'example-data',
  description: 'Example resource data',
  mime_type: 'application/json'
)

server = MCP::Server.new(
  name: 'my_server',
  resources: [resource]
)

# Define read handler
server.resources_read_handler do |params|
  case params[:uri]
  when 'resource://data/example'
    [{
      uri: params[:uri],
      mimeType: 'application/json',
      text: { message: 'Example data', timestamp: Time.now }.to_json
    }]
  else
    raise "Unknown resource: #{params[:uri]}"
  end
end
```

## Adding Prompts

prompt template を定義します。

### Prompt as Class

```ruby
class CodeReviewPrompt < MCP::Prompt
  prompt_name 'code_review'
  description 'Generate a code review prompt'

  arguments [
    MCP::Prompt::Argument.new(
      name: 'language',
      description: 'Programming language',
      required: true
    ),
    MCP::Prompt::Argument.new(
      name: 'focus',
      description: 'Review focus area',
      required: false
    )
  ]

  def self.template(args, server_context:)
    language = args['language'] || 'Ruby'
    focus = args['focus'] || 'general quality'

    MCP::Prompt::Result.new(
      description: "Code review for #{language} with focus on #{focus}",
      messages: [
        MCP::Prompt::Message.new(
          role: 'user',
          content: MCP::Content::Text.new("Please review this #{language} code with focus on #{focus}.")
        ),
        MCP::Prompt::Message.new(
          role: 'assistant',
          content: MCP::Content::Text.new("I'll review the code focusing on #{focus}. Please share the code.")
        )
      ]
    )
  end
end

server = MCP::Server.new(
  name: 'my_server',
  prompts: [CodeReviewPrompt]
)
```

### Prompt with Block

```ruby
server.define_prompt(
  name: 'analyze',
  description: 'Analyze a topic',
  arguments: [
    MCP::Prompt::Argument.new(name: 'topic', description: 'Topic to analyze', required: true),
    MCP::Prompt::Argument.new(name: 'depth', description: 'Analysis depth', required: false)
  ]
) do |args, server_context:|
  topic = args['topic']
  depth = args['depth'] || 'basic'

  MCP::Prompt::Result.new(
    description: "Analysis of #{topic} at #{depth} level",
    messages: [
      MCP::Prompt::Message.new(
        role: 'user',
        content: MCP::Content::Text.new("Please analyze: #{topic}")
      ),
      MCP::Prompt::Message.new(
        role: 'assistant',
        content: MCP::Content::Text.new("I'll provide a #{depth} analysis of #{topic}")
      )
    ]
  )
end
```

## Transport Configuration

### Stdio Transport

local command-line application 向け:

```ruby
require 'mcp'

server = MCP::Server.new(
  name: 'my_server',
  tools: [MyTool]
)

transport = MCP::Server::Transports::StdioTransport.new(server)
transport.open
```

### HTTP Transport (Rails)

Rails application 向け:

```ruby
class McpController < ApplicationController
  def index
    server = MCP::Server.new(
      name: 'rails_server',
      version: '1.0.0',
      tools: [SomeTool],
      prompts: [MyPrompt],
      server_context: { user_id: current_user.id }
    )

    render json: server.handle_json(request.body.read)
  end
end
```

### Streamable HTTP Transport

Server-Sent Events 向け:

```ruby
server = MCP::Server.new(name: 'my_server')
transport = MCP::Server::Transports::StreamableHTTPTransport.new(server)
server.transport = transport

# When tools change, notify clients
server.define_tool(name: 'new_tool') { |**args| { result: 'ok' } }
server.notify_tools_list_changed
```

## Server Context

tool や prompt に context 情報を渡します。

```ruby
server = MCP::Server.new(
  name: 'my_server',
  tools: [AuthenticatedTool],
  server_context: {
    user_id: current_user.id,
    request_id: request.uuid,
    auth_token: session[:token]
  }
)

class AuthenticatedTool < MCP::Tool
  def self.call(query:, server_context:)
    user_id = server_context[:user_id]
    # Use user_id for authorization

    MCP::Tool::Response.new([{ type: 'text', text: 'Authorized' }])
  end
end
```

## Configuration

### Exception Reporting

例外レポートを構成します。

```ruby
MCP.configure do |config|
  config.exception_reporter = ->(exception, server_context) {
    # Report to your error tracking service
    Bugsnag.notify(exception) do |report|
      report.add_metadata(:mcp, server_context)
    end
  }
end
```

### Instrumentation

MCP server のパフォーマンスを監視します。

```ruby
MCP.configure do |config|
  config.instrumentation_callback = ->(data) {
    # Log instrumentation data
    Rails.logger.info("MCP: #{data.inspect}")

    # Or send to metrics service
    StatsD.timing("mcp.#{data[:method]}.duration", data[:duration])
    StatsD.increment("mcp.#{data[:method]}.count")
  }
end
```

instrumentation data には次が含まれます。
- `method`: 呼び出された protocol method (例: "tools/call")
- `tool_name`: 呼び出された tool の名前
- `prompt_name`: 呼び出された prompt の名前
- `resource_uri`: 呼び出された resource の URI
- `error`: lookup が失敗した場合の error code
- `duration`: 秒単位の所要時間

### Protocol Version

protocol version を上書きします。

```ruby
configuration = MCP::Configuration.new(protocol_version: '2025-06-18')
server = MCP::Server.new(name: 'my_server', configuration: configuration)
```

## Tool Annotation

tool の振る舞いに関する metadata を提供します。

```ruby
class DataTool < MCP::Tool
  annotations(
    read_only_hint: true,      # Tool only reads data
    destructive_hint: false,   # Tool doesn't destroy data
    idempotent_hint: true,     # Same input = same output
    open_world_hint: false     # Tool operates in closed context
  )

  def self.call(**args, server_context:)
    # Implementation
  end
end
```

## Tool Output Schema

期待される output 構造を定義します。

```ruby
class WeatherTool < MCP::Tool
  output_schema(
    properties: {
      temperature: { type: 'number' },
      condition: { type: 'string' },
      humidity: { type: 'integer' }
    },
    required: ['temperature', 'condition']
  )

  def self.call(location:, server_context:)
    weather_data = {
      temperature: 72.5,
      condition: 'sunny',
      humidity: 45
    }

    # Validate against schema
    output_schema.validate_result(weather_data)

    MCP::Tool::Response.new(
      [{ type: 'text', text: weather_data.to_json }],
      structured_content: weather_data
    )
  end
end
```

## Response 内の Structured Content

text とともに structured data を返します。

```ruby
class APITool < MCP::Tool
  def self.call(endpoint:, server_context:)
    api_data = call_api(endpoint)

    MCP::Tool::Response.new(
      [{ type: 'text', text: api_data.to_json }],
      structured_content: api_data
    )
  end
end
```

## Custom Method

custom JSON-RPC method を定義します。

```ruby
server = MCP::Server.new(name: 'my_server')

# Custom method with result
server.define_custom_method(method_name: 'add') do |params|
  params[:a] + params[:b]
end

# Custom notification (returns nil)
server.define_custom_method(method_name: 'notify') do |params|
  puts "Notification: #{params[:message]}"
  nil
end
```

## Notifications

一覧変更通知を送ります。

```ruby
server = MCP::Server.new(name: 'my_server')
transport = MCP::Server::Transports::StreamableHTTPTransport.new(server)
server.transport = transport

# Notify when tools change
server.define_tool(name: 'new_tool') { |**args| { result: 'ok' } }
server.notify_tools_list_changed

# Notify when prompts change
server.define_prompt(name: 'new_prompt') { |args, **_| MCP::Prompt::Result.new(...) }
server.notify_prompts_list_changed

# Notify when resources change
server.notify_resources_list_changed
```

## Resource Templates

URI template を使って dynamic resource を定義します。

```ruby
resource_template = MCP::ResourceTemplate.new(
  uri_template: 'users://{user_id}/profile',
  name: 'user-profile',
  description: 'User profile data',
  mime_type: 'application/json'
)

server = MCP::Server.new(
  name: 'my_server',
  resource_templates: [resource_template]
)
```

## Error Handling

tool 内でエラーを適切に処理します。

```ruby
class RiskyTool < MCP::Tool
  def self.call(data:, server_context:)
    begin
      result = risky_operation(data)
      MCP::Tool::Response.new([{ type: 'text', text: result }])
    rescue ValidationError => e
      MCP::Tool::Response.new(
        [{ type: 'text', text: "Invalid input: #{e.message}" }],
        is_error: true
      )
    rescue => e
      # Will be caught and reported by exception_reporter
      raise
    end
  end
end
```

## Testing

MCP server のテストを書きます。

```ruby
require 'minitest/autorun'
require 'mcp'

class MyToolTest < Minitest::Test
  def test_greet_tool
    response = GreetTool.call(name: 'Ruby', server_context: {})

    assert_equal 1, response.content.length
    assert_match(/Ruby/, response.content.first[:text])
    refute response.is_error
  end

  def test_invalid_input
    response = CalculateTool.call(operation: 'divide', a: 10, b: 0, server_context: {})

    assert response.is_error
  end
end
```

## Client Usage

server に接続する MCP client を構築します。

```ruby
require 'mcp'
require 'faraday'

# HTTP transport
http_transport = MCP::Client::HTTP.new(
  url: 'https://api.example.com/mcp',
  headers: { 'Authorization' => "Bearer #{token}" }
)

client = MCP::Client.new(transport: http_transport)

# List tools
tools = client.tools
tools.each do |tool|
  puts "Tool: #{tool.name}"
  puts "Description: #{tool.description}"
end

# Call a tool
response = client.call_tool(
  tool: tools.first,
  arguments: { message: 'Hello, world!' }
)
```

## ベスト プラクティス

1. **複雑な tool には class を使う** - より良い構成とテスト容易性のため
2. **input / output schema を定義する** - 型安全性と validation を確保するため
3. **annotation を追加する** - client が tool の振る舞いを理解しやすくするため
4. **structured content を含める** - text と structured data の両方を提供するため
5. **server_context を使う** - 認証情報と request context を渡すため
6. **exception reporting を構成する** - 本番で error を監視するため
7. **instrumentation を実装する** - パフォーマンス指標を追跡するため
8. **notification を送る** - client に変更を知らせるため
9. **入力を検証する** - 処理前に parameter をチェックするため
10. **Ruby 規約に従う** - snake_case と適切な indentation を使うため

## よくあるパターン

### Authenticated Tool

```ruby
class AuthenticatedTool < MCP::Tool
  def self.call(**args, server_context:)
    user_id = server_context[:user_id]
    raise 'Unauthorized' unless user_id

    # Process authenticated request
    MCP::Tool::Response.new([{ type: 'text', text: 'Success' }])
  end
end
```

### Paginated Resource

```ruby
server.resources_read_handler do |params|
  uri = params[:uri]
  page = params[:page] || 1

  data = fetch_paginated_data(page)

  [{
    uri: uri,
    mimeType: 'application/json',
    text: data.to_json
  }]
end
```

### Dynamic Prompt

```ruby
class DynamicPrompt < MCP::Prompt
  def self.template(args, server_context:)
    user_id = server_context[:user_id]
    user_data = User.find(user_id)

    MCP::Prompt::Result.new(
      description: "Personalized prompt for #{user_data.name}",
      messages: generate_messages_for(user_data)
    )
  end
end
```
