# Claude (Anthropic) Integration Guide

This document provides comprehensive information about Claude (Anthropic) integration in NextChat.

## Overview

NextChat supports Claude models through Anthropic's API, providing access to powerful conversational AI capabilities including vision models, tool usage, and streaming responses.

## Supported Models

The following Claude models are supported:

### Legacy Models
- `claude-instant-1.2` - Fast, lightweight model
- `claude-2.0` - Previous generation model
- `claude-2.1` - Enhanced version of Claude 2

### Claude 3 Family
- `claude-3-haiku-20240307` - Fast and cost-effective
- `claude-3-5-haiku-20241022` - Latest Haiku model
- `claude-3-5-haiku-latest` - Always points to latest Haiku
- `claude-3-sonnet-20240229` - Balanced performance and speed
- `claude-3-5-sonnet-20240620` - Enhanced Sonnet model
- `claude-3-5-sonnet-20241022` - Latest Sonnet model
- `claude-3-5-sonnet-latest` - Always points to latest Sonnet
- `claude-3-opus-20240229` - Most capable model
- `claude-3-opus-latest` - Always points to latest Opus

### Claude 3.7 Family (Latest)
- `claude-3-7-sonnet-20250219` - Latest generation Sonnet
- `claude-3-7-sonnet-latest` - Always points to latest 3.7 Sonnet

## Configuration

### Environment Variables

#### Required
- `ANTHROPIC_API_KEY` - Your Anthropic API key

#### Optional
- `ANTHROPIC_URL` - Custom Anthropic API endpoint (default: `https://api.anthropic.com`)
- `ANTHROPIC_API_VERSION` - API version (default: `2023-06-01`)

### Example Configuration

```bash
# Docker deployment
docker run -d -p 3000:3000 \
  -e ANTHROPIC_API_KEY=your-api-key \
  -e CODE=your-password \
  yidadaa/chatgpt-next-web
```

```env
# .env.local for development
ANTHROPIC_API_KEY=your-api-key-here
ANTHROPIC_URL=https://api.anthropic.com
ANTHROPIC_API_VERSION=2023-06-01
```

## Features

### Vision Support

Claude 3 models support vision capabilities for image analysis:

```typescript
// Vision models are automatically detected by regex patterns
const visionModels = [
  /claude-3/,
  // Other vision model patterns...
];

// Exception: claude-3-5-haiku-20241022 does not support vision
const excludeVisionModels = [/claude-3-5-haiku-20241022/];
```

### Tool Usage (Function Calling)

Claude supports tool usage through the Messages API:

```typescript
interface ClaudeTool {
  name: string;
  description: string;
  input_schema: object;
}
```

### Streaming Responses

Real-time streaming is supported for all Claude models:

```typescript
const requestBody: AnthropicChatRequest = {
  model: "claude-3-5-sonnet-latest",
  messages: [...],
  stream: true,
  max_tokens: 4096,
  temperature: 0.7,
};
```

## API Implementation

### Message Format

Claude uses a specific message format with role alternation requirements:

```typescript
interface AnthropicMessage {
  role: "user" | "assistant";
  content: string | MultiBlockContent[];
}

interface MultiBlockContent {
  type: "text" | "image" | "tool_use" | "tool_result";
  text?: string;
  source?: {
    type: string;
    media_type: string;
    data: string;
  };
}
```

### Role Mapping

NextChat automatically maps roles for Claude compatibility:

```typescript
const ClaudeMapper = {
  assistant: "assistant",
  user: "user",
  system: "user", // System messages are converted to user messages
} as const;
```

### Request Parameters

```typescript
interface AnthropicChatRequest {
  model: string;
  messages: AnthropicMessage[];
  max_tokens: number;
  temperature?: number;
  top_p?: number;
  top_k?: number;
  stream?: boolean;
  stop_sequences?: string[];
}
```

## Usage Examples

### Basic Chat

```typescript
const response = await fetch('/api/anthropic/v1/messages', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'anthropic-version': '2023-06-01',
    'x-api-key': 'your-api-key'
  },
  body: JSON.stringify({
    model: 'claude-3-5-sonnet-latest',
    messages: [
      { role: 'user', content: 'Hello, Claude!' }
    ],
    max_tokens: 1024
  })
});
```

### Vision Example

```typescript
const visionRequest = {
  model: 'claude-3-5-sonnet-latest',
  messages: [{
    role: 'user',
    content: [
      { type: 'text', text: 'What do you see in this image?' },
      {
        type: 'image',
        source: {
          type: 'base64',
          media_type: 'image/jpeg',
          data: 'base64-encoded-image-data'
        }
      }
    ]
  }],
  max_tokens: 1024
};
```

### Tool Usage Example

```typescript
const toolRequest = {
  model: 'claude-3-5-sonnet-latest',
  messages: [
    { role: 'user', content: 'What\'s the weather like in San Francisco?' }
  ],
  tools: [{
    name: 'get_weather',
    description: 'Get current weather for a location',
    input_schema: {
      type: 'object',
      properties: {
        location: { type: 'string' }
      }
    }
  }],
  max_tokens: 1024
};
```

## Error Handling

Common error scenarios and handling:

### Authentication Errors
- Ensure `ANTHROPIC_API_KEY` is correctly set
- Verify API key has necessary permissions

### Rate Limiting
- Claude API has rate limits based on your plan
- Implement exponential backoff for retries

### Model Availability
- Check if the requested model is available in your region
- Some models may have restricted access

## Best Practices

1. **Model Selection**
   - Use `claude-3-5-sonnet-latest` for most applications
   - Use `claude-3-5-haiku-latest` for faster, cost-effective responses
   - Use `claude-3-opus-latest` for complex reasoning tasks

2. **Token Management**
   - Set appropriate `max_tokens` based on your needs
   - Monitor token usage to control costs

3. **Message Formatting**
   - Ensure proper role alternation (user/assistant)
   - Use system messages sparingly (converted to user messages)

4. **Vision Models**
   - Optimize image sizes for better performance
   - Use appropriate image formats (JPEG, PNG, WebP)

5. **Streaming**
   - Enable streaming for better user experience
   - Handle partial responses appropriately

## Troubleshooting

### Common Issues

1. **"Invalid API key" error**
   - Verify `ANTHROPIC_API_KEY` is set correctly
   - Check if the API key is active and has proper permissions

2. **"Model not found" error**
   - Ensure the model name is correct
   - Check if the model is available in your region

3. **"Rate limit exceeded" error**
   - Implement rate limiting in your application
   - Consider upgrading your Anthropic plan

4. **Vision not working**
   - Verify you're using a Claude 3 model (except claude-3-5-haiku-20241022)
   - Check image format and size requirements

### Debug Mode

Enable debug logging to troubleshoot issues:

```typescript
console.log("[Anthropic Route] params ", params);
console.log("[Response] claude response: ", response);
```

## Security Considerations

1. **API Key Protection**
   - Never expose API keys in client-side code
   - Use environment variables for API key storage
   - Rotate API keys regularly

2. **Input Validation**
   - Validate user inputs before sending to Claude
   - Implement content filtering if needed

3. **Rate Limiting**
   - Implement application-level rate limiting
   - Monitor usage patterns for anomalies

## Resources

- [Anthropic API Documentation](https://docs.anthropic.com/)
- [Claude Model Comparison](https://docs.anthropic.com/claude/docs/models-overview)
- [Vision Capabilities](https://docs.anthropic.com/claude/docs/vision)
- [Tool Usage Guide](https://docs.anthropic.com/claude/docs/tool-use)

## Contributing

When contributing Claude-related features:

1. Update model lists in `app/constant.ts`
2. Test with multiple Claude models
3. Ensure vision and tool usage compatibility
4. Update this documentation accordingly

## License

This integration follows the same license as the main NextChat project.