# n8n-nodes-avito

This is an n8n community node for integrating with the Avito API. It provides functionality to interact with Avito's shop listings, messaging system, and webhook handling.

[n8n](https://n8n.io/) is a [fair-code licensed](https://docs.n8n.io/reference/license/) workflow automation platform.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Operations](#operations)
- [Credentials](#credentials)
- [Usage Examples](#usage-examples)
- [Webhook Integration](#webhook-integration)
- [AI Integration](#ai-integration)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)
- [Version History](#version-history)

## Features

### Shop Management
- List shop items with filtering:
  - Status (active/removed/old/blocked/rejected)
  - Pagination (1-100 items per page)
  - Category filtering
  - Date-based filtering

### Messaging
- Send messages to users
- Webhook integration for real-time message updates
- AI-powered automated responses

### Webhook Support
- Automatic webhook registration
- Real-time message notifications
- Secure webhook validation
- Test and Production webhook URLs

### AI Integration
- Built-in AI consultant workflow
- Context-aware responses
- Customizable AI behavior
- Real-time item data integration

## Installation

Follow these steps to install this node in your n8n instance:

```bash
# Using npm
npm install n8n-nodes-avito

# Using n8n CLI
n8n-node-dev install n8n-nodes-avito
```

## Installation & Build

### Prerequisites
First, install n8n globally using npm:
```bash
npm install n8n -g
```

### Local Development
```bash
# Install dependencies
npm install

# Build the package
npm run build

# Create the package
npm pack
```

### Installing in n8n
To install this custom node in your local n8n installation:

1. Build the package as described above
2. Install the package in your n8n installation:
```bash
cd ~/.n8n/custom
npm install /path/to/n8n-nodes-avito-1.0.1.tgz
```
3. Restart n8n

Note: Replace `/path/to/n8n-nodes-avito-1.0.1.tgz` with the actual path to the generated .tgz file.

### Installing in n8n via bash scrypt build and deploy
```bash
#!/bin/bash

# Цвета для вывода
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# Проверка наличия файла конфигурации n8n
N8N_CONFIG_DIR="$HOME/.n8n"
N8N_CONFIG_FILE="$N8N_CONFIG_DIR/config"
N8N_CUSTOM_DIR="$N8N_CONFIG_DIR/custom"

echo -e "${GREEN}1. Проверка конфигурации n8n...${NC}"
if [ ! -d "$N8N_CONFIG_DIR" ]; then
    mkdir -p "$N8N_CONFIG_DIR"
fi

if [ ! -d "$N8N_CUSTOM_DIR" ]; then
    mkdir -p "$N8N_CUSTOM_DIR"
fi

# Установка правильных прав доступа
chmod 600 "$N8N_CONFIG_FILE" 2>/dev/null

# Генерация и установка ключа шифрования
if [ ! -f "$N8N_CONFIG_FILE" ]; then
    NEW_KEY=$(openssl rand -hex 32)
    echo "{\"encryptionKey\": \"$NEW_KEY\"}" > "$N8N_CONFIG_FILE"
    echo "export N8N_ENCRYPTION_KEY=$NEW_KEY" >> ~/.zshrc
    source ~/.zshrc
    echo -e "${GREEN}Создан новый ключ шифрования в конфигурационном файле${NC}"
else
    EXISTING_KEY=$(cat "$N8N_CONFIG_FILE" | grep -o '"encryptionKey": *"[^"]*"' | cut -d'"' -f4)
    if [ ! -z "$EXISTING_KEY" ]; then
        echo "export N8N_ENCRYPTION_KEY=$EXISTING_KEY" >> ~/.zshrc
        source ~/.zshrc
        echo -e "${GREEN}Использован существующий ключ шифрования${NC}"
    fi
fi

# Сохраняем текущую директорию
CURRENT_DIR=$(pwd)

echo -e "${GREEN}2. Очистка предыдущей сборки...${NC}"
npm run clean

echo -e "${GREEN}3. Сборка проекта...${NC}"
npm run build

echo -e "${GREEN}4. Создание пакета...${NC}"
npm pack

echo -e "${GREEN}5. Установка пакета...${NC}"
cd "$N8N_CUSTOM_DIR"
# Очищаем существующие node_modules
rm -rf node_modules package-lock.json
# Устанавливаем пакет и зависимости
npm install "$CURRENT_DIR"/n8n-nodes-avito-*.tgz --legacy-peer-deps

# Возвращаемся в исходную директорию
cd "$CURRENT_DIR"

echo -e "${GREEN}6. Перезапуск n8n...${NC}"
pkill -f "n8n"
sleep 2

# Запускаем n8n в фоновом режиме с перенаправлением вывода
export N8N_CUSTOM_NODES_URLS="file://$N8N_CUSTOM_DIR"
export N8N_RUNNERS_ENABLED=true
nohup n8n start --tunnel > n8n.log 2>&1 &

echo -e "${GREEN}Готово! n8n запущен в фоновом режиме.${NC}"
echo -e "${GREEN}Логи доступны в файле: n8n.log${NC}"
echo -e "${GREEN}Для просмотра логов используйте: tail -f n8n.log${NC}"
echo -e "${GREEN}Интерфейс n8n доступен по адресу: http://localhost:5678${NC}" 
```

## Operations

### Shop Resource
```typescript
// Get Items
{
  "resource": "shop",
  "operation": "getItems",
  "parameters": {
    "perPage": 25,          // 1-100
    "page": 1,              // min: 1
    "status": ["active"],   // active/removed/old/blocked/rejected
    "updatedAtFrom": "YYYY-MM-DD",
    "category": 123         // Category ID
  }
}
```

### Message Resource
```typescript
// Send Message
{
  "resource": "message",
  "operation": "send",
  "parameters": {
    "chat_id": "string",    // Required
    "user_id": "number",    // Required
    "message": "string"     // Required
  }
}
```

## Credentials

### Required Credentials
- Client ID
- Client Secret

### Authentication
- OAuth 2.0 implementation
- Automatic token refresh
- Token expiration handling

## Usage Examples

### 1. Basic Shop Items Workflow
```typescript
[
  {
    "node": "Avito",
    "parameters": {
      "resource": "shop",
      "operation": "getItems",
      "perPage": 25,
      "status": ["active"]
    }
  }
]
```

### 2. AI Consultant Workflow
```typescript
[
  {
    "name": "Avito Trigger",
    "type": "n8n-nodes-avito.avitoTrigger",
    "parameters": {}
  },
  {
    "name": "Get Active Items",
    "type": "n8n-nodes-avito.avito",
    "parameters": {
      "resource": "shop",
      "operation": "getItems",
      "status": ["active"]
    }
  },
  {
    "name": "AI Assistant",
    "type": "n8n-nodes-base.aiAgent",
    "parameters": {
      "prompt": "Your AI consultant prompt",
      "contextFields": {
        "activeItems": "={{ $node[\"Get Active Items\"].json.resources }}"
      }
    }
  },
  {
    "name": "Send Response",
    "type": "n8n-nodes-avito.avito",
    "parameters": {
      "resource": "message",
      "operation": "send"
    }
  }
]
```

## Webhook Integration

### Setup
1. Add Avito Trigger node to your workflow
2. Activate the workflow
3. Use the Production webhook URL in Avito settings

### Testing
1. Use Test webhook URL during development
2. 120-second timeout for test webhooks
3. Data visible in editor UI

### Production
1. Switch to Production webhook URL
2. Automatic webhook registration
3. Persistent webhook connection

## AI Integration

### Features
- Real-time item context
- Natural language processing
- Customizable responses
- Multi-turn conversations

### Configuration
```typescript
{
  "prompt": "Professional Avito consultant prompt",
  "contextFields": {
    "activeItems": "Items data",
    "messageInfo": "Message context"
  },
  "parameters": {
    "temperature": 0.7,
    "maxTokens": 500
  }
}
```

## Troubleshooting

### Common Issues
1. Webhook Registration
   - Check API credentials
   - Verify webhook URL accessibility
   - Ensure proper permissions

2. Message Sending
   - Validate chat_id and user_id
   - Check message format
   - Verify token validity

3. AI Integration
   - Check context data
   - Verify AI node configuration
   - Monitor response format

## Resources

- [Avito API Documentation](https://developers.avito.ru/api/doc)
- [n8n Documentation](https://docs.n8n.io)
- [Webhook Documentation](https://developers.avito.ru)

## Version History

### v1.0.1 (Current)
- Added AI consultant workflow
- Enhanced webhook handling
- Improved error handling
- Updated documentation

### v1.0.0
- Initial release
- Basic shop and message operations
- Webhook support

## Support

If you have any questions or issues:

1. Check the [issue tracker](https://github.com/GODCRM/n8n-nodes-avito/issues)
2. Create a new issue if needed

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

[MIT](LICENSE.md)
