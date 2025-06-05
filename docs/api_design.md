# 微信小游戏MUD量子叙事API设计

## API概述

本文档详细描述了微信小游戏MUD量子叙事项目的API设计，包括WebSocket消息格式、命令系统API、事件通知机制和错误处理。该API设计旨在支持微信小游戏前端与Node.js后端之间的实时通信，实现MUD游戏的核心功能。

## 通信协议

### 1. 通信方式

项目采用WebSocket作为主要的通信方式，实现前后端之间的实时双向通信。WebSocket连接建立后，前后端可以随时发送和接收消息，无需重新建立连接。

### 2. 连接建立

前端通过以下方式建立WebSocket连接：

```javascript
// 前端WebSocket连接示例
const socket = new WebSocket('wss://your-domain.com/ws');

// 连接建立事件
socket.onopen = (event) => {
  console.log('WebSocket连接已建立');
  
  // 发送认证消息
  const authMessage = {
    type: 'auth',
    data: {
      code: wxLoginCode,  // 微信登录凭证
      nickname: userInfo.nickName
    }
  };
  
  socket.send(JSON.stringify(authMessage));
};

// 接收消息事件
socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  handleMessage(message);
};

// 连接关闭事件
socket.onclose = (event) => {
  console.log('WebSocket连接已关闭:', event.code, event.reason);
  // 实现重连逻辑
};

// 连接错误事件
socket.onerror = (error) => {
  console.error('WebSocket错误:', error);
};
```

## 消息格式

### 1. 基本消息结构

所有WebSocket消息都使用JSON格式，包含以下基本结构：

```javascript
{
  "type": "message_type",  // 消息类型
  "data": {                // 消息数据
    // 根据消息类型不同，包含不同的数据字段
  },
  "meta": {                // 元数据（可选）
    "timestamp": 1625097600000,  // 时间戳
    "messageId": "abc123",       // 消息ID
    "clientId": "client123"      // 客户端ID
  }
}
```

### 2. 消息类型

系统支持以下主要消息类型：

| 消息类型 | 方向 | 描述 |
|---------|------|------|
| auth | 客户端 → 服务器 | 认证请求 |
| auth_response | 服务器 → 客户端 | 认证响应 |
| command | 客户端 → 服务器 | 游戏命令 |
| command_response | 服务器 → 客户端 | 命令响应 |
| event | 服务器 → 客户端 | 游戏事件通知 |
| event_response | 客户端 → 服务器 | 事件响应 |
| state | 服务器 → 客户端 | 状态更新 |
| chat | 双向 | 聊天消息 |
| error | 服务器 → 客户端 | 错误消息 |
| ping | 双向 | 心跳检测 |
| pong | 双向 | 心跳响应 |

## 认证API

### 1. 认证请求

客户端发送认证请求，使用微信登录凭证进行身份验证。

```javascript
// 认证请求
{
  "type": "auth",
  "data": {
    "code": "wx_login_code",  // 微信登录凭证
    "nickname": "玩家昵称"
  }
}
```

### 2. 认证响应

服务器返回认证结果，包含玩家信息和会话标识。

```javascript
// 认证成功响应
{
  "type": "auth_response",
  "data": {
    "success": true,
    "playerId": "player123",
    "sessionId": "session456",
    "playerInfo": {
      "nickname": "玩家昵称",
      "role": "citizen",
      "attributes": {
        "strength": 10,
        "intelligence": 10,
        "charisma": 10
      },
      "position": {
        "mapId": "start",
        "x": 0,
        "y": 0
      }
    }
  }
}

// 认证失败响应
{
  "type": "auth_response",
  "data": {
    "success": false,
    "error": "invalid_code",
    "message": "无效的登录凭证"
  }
}
```

## 命令系统API

命令系统是玩家与游戏交互的主要方式，通过发送命令消息实现各种游戏操作。

### 1. 命令请求

客户端发送命令请求，包含命令名称和参数。

```javascript
// 命令请求
{
  "type": "command",
  "data": {
    "command": "move",     // 命令名称
    "args": ["north"]      // 命令参数
  },
  "meta": {
    "messageId": "cmd123"  // 消息ID，用于匹配响应
  }
}
```

### 2. 命令响应

服务器返回命令执行结果，包含成功/失败状态、响应消息和状态更新。

```javascript
// 命令响应
{
  "type": "command_response",
  "data": {
    "success": true,       // 命令是否成功
    "message": "你向北移动了一步，来到了一个新的房间。",  // 响应消息
    "state": {             // 状态更新（可选）
      "position": {
        "mapId": "forest",
        "x": 5,
        "y": 3
      }
    }
  },
  "meta": {
    "messageId": "resp123",
    "requestId": "cmd123"  // 对应的请求ID
  }
}
```

### 3. 支持的命令列表

以下是系统支持的主要命令：

#### 移动命令

```javascript
// 移动命令
{
  "type": "command",
  "data": {
    "command": "move",
    "args": ["north"]  // 方向：north, south, east, west, up, down
  }
}

// 简写形式
{
  "type": "command",
  "data": {
    "command": "north"  // 直接使用方向作为命令
  }
}
```

#### 查看命令

```javascript
// 查看周围
{
  "type": "command",
  "data": {
    "command": "look"
  }
}

// 查看特定对象
{
  "type": "command",
  "data": {
    "command": "look",
    "args": ["chest"]  // 查看箱子
  }
}
```

#### 交互命令

```javascript
// 拿取物品
{
  "type": "command",
  "data": {
    "command": "take",
    "args": ["sword"]  // 拿起剑
  }
}

// 使用物品
{
  "type": "command",
  "data": {
    "command": "use",
    "args": ["potion"]  // 使用药水
  }
}

// 与NPC对话
{
  "type": "command",
  "data": {
    "command": "talk",
    "args": ["merchant"]  // 与商人交谈
  }
}
```

#### 社交命令

```javascript
// 发送聊天消息
{
  "type": "command",
  "data": {
    "command": "say",
    "args": ["Hello everyone!"]  // 说话内容
  }
}

// 私聊消息
{
  "type": "command",
  "data": {
    "command": "whisper",
    "args": ["player2", "Hello there!"]  // 私聊对象和内容
  }
}
```

#### 系统命令

```javascript
// 查看帮助
{
  "type": "command",
  "data": {
    "command": "help",
    "args": ["move"]  // 特定命令的帮助，可选
  }
}

// 查看状态
{
  "type": "command",
  "data": {
    "command": "status"
  }
}
```

## 事件系统API

事件系统用于服务器向客户端推送游戏事件，客户端可以对事件做出响应。

### 1. 事件通知

服务器向客户端发送事件通知，包含事件信息和可能的响应选项。

```javascript
// 事件通知
{
  "type": "event",
  "data": {
    "eventId": "event123",  // 事件ID
    "title": "神秘的声音",   // 事件标题
    "description": "你听到远处传来一阵神秘的声音，似乎在呼唤着你的名字。",  // 事件描述
    "options": [           // 可选的响应选项
      {
        "id": "option1",
        "text": "走向声音的方向"
      },
      {
        "id": "option2",
        "text": "忽略声音，继续前进"
      },
      {
        "id": "option3",
        "text": "警惕地观察四周"
      }
    ],
    "timeout": 60000       // 响应超时时间（毫秒），可选
  }
}
```

### 2. 事件响应

客户端向服务器发送事件响应，选择一个响应选项。

```javascript
// 事件响应
{
  "type": "event_response",
  "data": {
    "eventId": "event123",  // 事件ID
    "optionId": "option1"   // 选择的选项ID
  }
}
```

### 3. 事件结果

服务器向客户端发送事件结果，包含选择的结果和影响。

```javascript
// 事件结果
{
  "type": "event_result",
  "data": {
    "eventId": "event123",  // 事件ID
    "result": "你走向声音的方向，发现了一个隐藏的洞穴入口。",  // 结果描述
    "effects": [           // 事件效果
      {
        "type": "discover",
        "target": "cave_entrance",
        "description": "发现了洞穴入口"
      },
      {
        "type": "attribute",
        "target": "courage",
        "value": 1,
        "description": "勇气+1"
      }
    ]
  }
}
```

## 状态更新API

状态更新API用于服务器向客户端推送游戏状态的变化。

### 1. 玩家状态更新

```javascript
// 玩家状态更新
{
  "type": "state",
  "data": {
    "type": "player",      // 状态类型
    "attributes": {        // 属性更新
      "health": 95,
      "energy": 80
    },
    "position": {          // 位置更新
      "mapId": "forest",
      "x": 5,
      "y": 3
    },
    "inventory": [         // 物品栏更新
      {
        "itemId": "item123",
        "name": "木剑",
        "quantity": 1
      }
    ]
  }
}
```

### 2. 环境状态更新

```javascript
// 环境状态更新
{
  "type": "state",
  "data": {
    "type": "environment",  // 状态类型
    "time": "day",          // 时间
    "weather": "rain",      // 天气
    "visibility": "low"     // 能见度
  }
}
```

### 3. 社会状态更新

```javascript
// 社会状态更新
{
  "type": "state",
  "data": {
    "type": "social",       // 状态类型
    "civilizationLevel": 2, // 文明等级
    "dominantRole": "merchant", // 主导角色
    "publicEvents": [       // 公共事件
      {
        "id": "event456",
        "title": "市场日",
        "description": "今天是每周的市场日，商人们聚集在广场。"
      }
    ]
  }
}
```

## 聊天系统API

聊天系统API用于玩家之间的通信。

### 1. 发送聊天消息

客户端向服务器发送聊天消息。

```javascript
// 发送聊天消息
{
  "type": "chat",
  "data": {
    "channel": "global",    // 聊天频道：global, local, private
    "message": "Hello everyone!",  // 消息内容
    "target": "player2"     // 私聊目标，仅当channel为private时需要
  }
}
```

### 2. 接收聊天消息

服务器向客户端推送聊天消息。

```javascript
// 接收聊天消息
{
  "type": "chat",
  "data": {
    "channel": "global",    // 聊天频道
    "sender": {             // 发送者信息
      "id": "player1",
      "name": "玩家1",
      "role": "warrior"
    },
    "message": "Hello everyone!",  // 消息内容
    "timestamp": 1625097600000     // 时间戳
  }
}
```

## 错误处理

### 1. 错误消息格式

服务器向客户端发送错误消息，包含错误代码、错误消息和详细信息。

```javascript
// 错误消息
{
  "type": "error",
  "data": {
    "code": "invalid_command",  // 错误代码
    "message": "无效的命令",     // 错误消息
    "details": {               // 错误详情（可选）
      "command": "fly",
      "reason": "未知命令"
    }
  },
  "meta": {
    "requestId": "cmd456"      // 对应的请求ID（如果有）
  }
}
```

### 2. 错误代码列表

| 错误代码 | 描述 | HTTP状态码等价 |
|---------|------|--------------|
| invalid_auth | 认证失败 | 401 |
| invalid_session | 会话无效或过期 | 401 |
| invalid_command | 无效的命令 | 400 |
| invalid_args | 无效的命令参数 | 400 |
| not_found | 请求的资源不存在 | 404 |
| forbidden | 没有权限执行该操作 | 403 |
| conflict | 操作冲突 | 409 |
| server_error | 服务器内部错误 | 500 |
| timeout | 操作超时 | 408 |

### 3. 错误处理策略

客户端应该实现以下错误处理策略：

1. **显示错误消息**：向用户显示友好的错误消息
2. **重试机制**：对于网络错误或超时，实现自动重试
3. **重新认证**：当收到认证失败或会话过期错误时，自动重新认证
4. **错误日志**：记录错误信息，便于调试和问题排查

## 心跳机制

为了保持WebSocket连接的活跃状态，客户端和服务器之间实现心跳机制。

### 1. 心跳请求

客户端定期向服务器发送心跳请求。

```javascript
// 心跳请求
{
  "type": "ping",
  "data": {
    "timestamp": 1625097600000  // 当前时间戳
  }
}
```

### 2. 心跳响应

服务器收到心跳请求后，返回心跳响应。

```javascript
// 心跳响应
{
  "type": "pong",
  "data": {
    "timestamp": 1625097600100,  // 服务器时间戳
    "clientTimestamp": 1625097600000  // 客户端时间戳
  }
}
```

### 3. 心跳实现

客户端应该实现以下心跳逻辑：

```javascript
// 心跳实现
let heartbeatInterval;
let missedHeartbeats = 0;
const MAX_MISSED_HEARTBEATS = 3;

// 启动心跳
function startHeartbeat() {
  heartbeatInterval = setInterval(() => {
    if (socket.readyState === WebSocket.OPEN) {
      // 发送心跳请求
      socket.send(JSON.stringify({
        type: 'ping',
        data: {
          timestamp: Date.now()
        }
      }));
      
      // 增加未响应心跳计数
      missedHeartbeats++;
      
      // 如果连续多次未收到响应，认为连接已断开
      if (missedHeartbeats >= MAX_MISSED_HEARTBEATS) {
        console.log('连接似乎已断开，尝试重新连接');
        socket.close();
        reconnect();
      }
    }
  }, 30000);  // 每30秒发送一次心跳
}

// 处理心跳响应
function handlePong(message) {
  // 重置未响应心跳计数
  missedHeartbeats = 0;
}
```

## 重连机制

客户端应该实现WebSocket连接断开后的重连机制。

```javascript
// 重连机制
let reconnectAttempts = 0;
const MAX_RECONNECT_ATTEMPTS = 5;
const RECONNECT_DELAY = 3000;  // 3秒

// 重连函数
function reconnect() {
  if (reconnectAttempts >= MAX_RECONNECT_ATTEMPTS) {
    console.log('达到最大重连次数，停止重连');
    return;
  }
  
  reconnectAttempts++;
  
  console.log(`尝试重新连接 (${reconnectAttempts}/${MAX_RECONNECT_ATTEMPTS})...`);
  
  // 延迟重连，避免立即重连
  setTimeout(() => {
    // 创建新的WebSocket连接
    socket = new WebSocket('wss://your-domain.com/ws');
    
    // 设置事件处理器
    socket.onopen = handleOpen;
    socket.onmessage = handleMessage;
    socket.onclose = handleClose;
    socket.onerror = handleError;
  }, RECONNECT_DELAY * reconnectAttempts);  // 递增延迟
}

// 连接成功后重置重连计数
function handleOpen(event) {
  console.log('WebSocket连接已建立');
  reconnectAttempts = 0;
  
  // 重新认证
  authenticate();
  
  // 启动心跳
  startHeartbeat();
}

// 连接关闭处理
function handleClose(event) {
  console.log('WebSocket连接已关闭:', event.code, event.reason);
  
  // 清除心跳定时器
  clearInterval(heartbeatInterval);
  
  // 尝试重连
  reconnect();
}
```

## API使用示例

### 1. 玩家登录流程

```javascript
// 1. 建立WebSocket连接
const socket = new WebSocket('wss://your-domain.com/ws');

// 2. 连接建立后发送认证请求
socket.onopen = async () => {
  // 获取微信登录凭证
  const { code } = await wx.login();
  
  // 获取用户信息
  const { userInfo } = await wx.getUserInfo();
  
  // 发送认证请求
  socket.send(JSON.stringify({
    type: 'auth',
    data: {
      code: code,
      nickname: userInfo.nickName
    }
  }));
};

// 3. 处理认证响应
socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  
  if (message.type === 'auth_response') {
    if (message.data.success) {
      // 认证成功，保存玩家信息
      const { playerId, sessionId, playerInfo } = message.data;
      
      // 存储会话信息
      localStorage.setItem('playerId', playerId);
      localStorage.setItem('sessionId', sessionId);
      
      // 更新UI显示玩家信息
      updatePlayerInfo(playerInfo);
      
      // 显示游戏界面
      showGameInterface();
      
      // 启动心跳
      startHeartbeat();
    } else {
      // 认证失败，显示错误信息
      showError(message.data.message);
    }
  }
};
```

### 2. 发送游戏命令

```javascript
// 发送移动命令
function sendMoveCommand(direction) {
  const messageId = generateMessageId();
  
  socket.send(JSON.stringify({
    type: 'command',
    data: {
      command: 'move',
      args: [direction]
    },
    meta: {
      messageId: messageId
    }
  }));
  
  // 记录待响应的命令
  pendingCommands.set(messageId, {
    command: 'move',
    direction: direction,
    timestamp: Date.now()
  });
}

// 处理命令响应
function handleCommandResponse(message) {
  const { success, message: responseMessage, state } = message.data;
  const requestId = message.meta.requestId;
  
  // 查找对应的命令请求
  const pendingCommand = pendingCommands.get(requestId);
  
  if (pendingCommand) {
    // 移除待响应命令
    pendingCommands.delete(requestId);
    
    // 显示命令响应
    displayMessage(responseMessage);
    
    // 如果命令成功且返回了状态更新，更新游戏状态
    if (success && state) {
      updateGameState(state);
    }
  }
}
```

### 3. 处理游戏事件

```javascript
// 处理事件通知
function handleEvent(message) {
  const { eventId, title, description, options } = message.data;
  
  // 显示事件信息
  displayEvent(title, description);
  
  // 显示选项按钮
  displayEventOptions(eventId, options);
}

// 发送事件响应
function sendEventResponse(eventId, optionId) {
  socket.send(JSON.stringify({
    type: 'event_response',
    data: {
      eventId: eventId,
      optionId: optionId
    }
  }));
}

// 处理事件结果
function handleEventResult(message) {
  const { eventId, result, effects } = message.data;
  
  // 显示事件结果
  displayEventResult(result);
  
  // 应用事件效果
  if (effects && effects.length > 0) {
    for (const effect of effects) {
      applyEffect(effect);
    }
  }
  
  // 隐藏事件界面
  hideEventInterface();
}
```

## 结论

本API设计文档详细描述了微信小游戏MUD量子叙事项目的通信协议和API接口。通过WebSocket实现实时通信，支持命令系统、事件系统、状态更新和聊天系统等核心功能。API设计遵循简单、一致和可扩展的原则，为游戏的开发和维护提供了清晰的指导。

开发者可以根据本文档实现前后端通信，确保游戏功能的正确实现和良好的用户体验。随着游戏的发展，API可能需要进行扩展和优化，但基本的通信架构和消息格式应保持稳定。