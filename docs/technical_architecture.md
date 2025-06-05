# 微信小游戏MUD量子叙事技术架构

## 架构概览

微信小游戏MUD量子叙事项目采用前后端分离的架构，前端基于微信小游戏平台，后端部署在单台腾讯云服务器上。整体技术架构如下图所示：

```
┌───────────────────────────────────────────────────────────────────────────┐
│                           微信小游戏前端                                   │
│                                                                           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │             │    │             │    │             │    │             │ │
│  │  UI渲染模块  │    │  命令系统   │    │ WebSocket  │    │  状态管理   │ │
│  │             │    │             │    │   客户端    │    │             │ │
│  └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘ │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ WebSocket
                                     ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                           Node.js后端服务                                 │
│                                                                           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │             │    │             │    │             │    │             │ │
│  │ WebSocket  │    │  命令处理器  │    │ 量子叙事引擎 │    │  玩家管理   │ │
│  │   服务器    │    │             │    │             │    │             │ │
│  └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘ │
│                                                                           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │             │    │             │    │             │    │             │ │
│  │  世界系统   │    │  事件系统   │    │  数据访问层  │    │  日志系统   │ │
│  │             │    │             │    │             │    │             │ │
│  └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘ │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
                 │                                  │
                 │                                  │
                 ▼                                  ▼
      ┌─────────────────────┐            ┌─────────────────────┐
      │                     │            │                     │
      │       MongoDB       │            │        Redis        │
      │                     │            │                     │
      └─────────────────────┘            └─────────────────────┘
```

## 前端架构

### 1. 微信小游戏框架

微信小游戏前端基于微信小游戏框架开发，主要使用JavaScript/TypeScript语言。

#### 核心模块

- **游戏初始化模块**：负责初始化游戏环境、加载资源和建立WebSocket连接
- **渲染模块**：负责渲染游戏界面，包括文本显示和简单UI元素
- **输入处理模块**：负责处理用户输入，包括文本命令和UI交互
- **状态管理模块**：负责管理游戏前端状态，包括玩家状态、游戏世界状态等
- **网络通信模块**：负责与后端服务器通信，包括WebSocket连接管理和消息处理

#### 目录结构

```
/js
  /core
    game.js           # 游戏核心逻辑
    renderer.js       # 渲染器
    input.js          # 输入处理
    state.js          # 状态管理
  /network
    websocket.js      # WebSocket客户端
    message.js        # 消息处理
  /ui
    components.js     # UI组件
    layout.js         # 布局管理
    themes.js         # 主题管理
  /commands
    parser.js         # 命令解析器
    executor.js       # 命令执行器
    commands.js       # 命令定义
  /utils
    logger.js         # 日志工具
    storage.js        # 本地存储工具
    helpers.js        # 辅助函数
  app.js              # 应用入口
```

### 2. UI设计

MUD游戏以文本为主，UI设计相对简单，但需要考虑移动设备的特点和微信小游戏的限制。

#### 界面布局

```
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│                         游戏标题                              │
│                                                               │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│                                                               │
│                                                               │
│                                                               │
│                      游戏文本显示区域                         │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│                      命令输入框                               │
│                                                               │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  [移动]  [查看]  [交互]  [物品]  [状态]  [帮助]  [设置]      │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

#### UI组件

- **文本显示区**：显示游戏描述、对话和事件
- **命令输入框**：允许玩家输入文本命令
- **快捷命令按钮**：提供常用命令的快捷访问
- **状态栏**：显示玩家基本状态信息
- **物品栏**：显示玩家物品
- **地图视图**：简单的地图显示（可选）
- **设置菜单**：游戏设置和选项

#### 渲染技术

- 使用Canvas进行文本渲染，支持富文本效果
- 使用微信小游戏的原生UI组件处理输入和按钮
- 实现自定义滚动文本区域，支持历史记录浏览
- 使用CSS样式定制UI主题和外观

### 3. 命令系统

命令系统是玩家与游戏交互的主要方式，需要设计灵活且易用的命令解析和执行机制。

#### 命令格式

```
<动词> [目标] [参数]
```

例如：
- `look` - 查看当前位置
- `look chest` - 查看箱子
- `take sword` - 拿起剑
- `talk to merchant` - 与商人交谈
- `move north` - 向北移动

#### 命令解析流程

1. 用户输入命令
2. 前端解析命令，识别动词和参数
3. 检查本地可执行的命令（如帮助、设置等）
4. 如果是需要后端处理的命令，通过WebSocket发送到服务器
5. 接收服务器响应并更新界面

#### 命令类型

- **移动命令**：控制玩家在游戏世界中的移动
- **交互命令**：与NPC、物品或环境交互
- **查看命令**：查看游戏世界、物品或状态信息
- **社交命令**：与其他玩家交流和互动
- **系统命令**：控制游戏设置、显示帮助等

#### 命令实现

```javascript
// 命令解析器
class CommandParser {
  constructor() {
    this.commands = new Map();
    this.registerCommands();
  }
  
  // 注册命令
  registerCommands() {
    // 移动命令
    this.registerCommand('move', ['go', 'walk'], this.handleMove);
    this.registerCommand('look', ['examine', 'inspect'], this.handleLook);
    this.registerCommand('take', ['get', 'pickup'], this.handleTake);
    this.registerCommand('talk', ['speak', 'chat'], this.handleTalk);
    // 更多命令...
  }
  
  // 注册单个命令
  registerCommand(name, aliases, handler) {
    this.commands.set(name, { name, aliases, handler });
    
    // 注册别名
    for (const alias of aliases) {
      this.commands.set(alias, { name, aliases, handler });
    }
  }
  
  // 解析命令
  parse(input) {
    // 分割命令和参数
    const parts = input.trim().split(' ');
    const verb = parts[0].toLowerCase();
    const args = parts.slice(1);
    
    // 查找命令
    const command = this.commands.get(verb);
    
    if (!command) {
      return {
        success: false,
        message: `未知命令: ${verb}`
      };
    }
    
    // 执行命令
    return command.handler(args);
  }
  
  // 命令处理函数
  handleMove(args) {
    const direction = args[0];
    
    if (!direction) {
      return {
        success: false,
        message: '请指定移动方向'
      };
    }
    
    return {
      success: true,
      type: 'move',
      direction
    };
  }
  
  handleLook(args) {
    const target = args.join(' ') || 'around';
    
    return {
      success: true,
      type: 'look',
      target
    };
  }
  
  handleTake(args) {
    const item = args.join(' ');
    
    if (!item) {
      return {
        success: false,
        message: '请指定要拿取的物品'
      };
    }
    
    return {
      success: true,
      type: 'take',
      item
    };
  }
  
  handleTalk(args) {
    if (args[0] === 'to') {
      args.shift();
    }
    
    const target = args.join(' ');
    
    if (!target) {
      return {
        success: false,
        message: '请指定交谈对象'
      };
    }
    
    return {
      success: true,
      type: 'talk',
      target
    };
  }
}
```

## 后端架构

### 1. Node.js服务器

后端服务基于Node.js开发，部署在单台腾讯云服务器上，提供游戏逻辑处理、数据存储和玩家通信功能。

#### 核心模块

- **HTTP服务器**：处理HTTP请求，主要用于健康检查和简单API
- **WebSocket服务器**：处理实时通信，是游戏的主要通信方式
- **命令处理器**：处理玩家命令
- **量子叙事引擎**：处理游戏事件和剧情
- **玩家管理器**：管理玩家状态和数据
- **世界系统**：管理游戏世界状态
- **数据访问层**：处理数据库操作
- **日志系统**：记录游戏日志和错误

#### 目录结构

```
/src
  /server
    index.js           # 服务器入口
    http.js            # HTTP服务器
    websocket.js       # WebSocket服务器
  /commands
    parser.js          # 命令解析器
    executor.js        # 命令执行器
    handlers.js        # 命令处理函数
  /engine
    quantum.js         # 量子叙事引擎
    markov.js          # 马尔可夫链处理器
    observer.js        # 观测者系统
    events.js          # 事件系统
  /world
    manager.js         # 世界管理器
    map.js             # 地图系统
    npc.js             # NPC系统
    items.js           # 物品系统
  /players
    manager.js         # 玩家管理器
    auth.js            # 认证系统
    inventory.js       # 物品栏系统
    relationships.js   # 社交关系系统
  /data
    mongodb.js         # MongoDB连接和操作
    redis.js           # Redis连接和操作
    models.js          # 数据模型
  /utils
    logger.js          # 日志工具
    config.js          # 配置管理
    helpers.js         # 辅助函数
  app.js               # 应用入口
```

### 2. WebSocket通信

WebSocket是游戏的主要通信方式，用于实时传输游戏状态和玩家命令。

#### 连接管理

```javascript
// WebSocket服务器
const WebSocket = require('ws');
const http = require('http');

// 创建HTTP服务器
const server = http.createServer((req, res) => {
  // 处理HTTP请求
});

// 创建WebSocket服务器
const wss = new WebSocket.Server({ server });

// 客户端连接
const clients = new Map();

// 处理连接
wss.on('connection', (ws, req) => {
  // 生成客户端ID
  const clientId = generateClientId();
  
  // 存储客户端连接
  clients.set(clientId, {
    ws,
    playerId: null,
    lastActivity: Date.now()
  });
  
  // 发送欢迎消息
  sendMessage(ws, {
    type: 'welcome',
    clientId
  });
  
  // 处理消息
  ws.on('message', (message) => {
    try {
      const data = JSON.parse(message);
      handleMessage(clientId, data);
    } catch (error) {
      console.error('Failed to parse message:', error);
      sendMessage(ws, {
        type: 'error',
        message: 'Invalid message format'
      });
    }
  });
  
  // 处理连接关闭
  ws.on('close', () => {
    console.log(`WebSocket connection closed: ${clientId}`);
    clients.delete(clientId);
  });
  
  // 处理错误
  ws.on('error', (error) => {
    console.error(`WebSocket error: ${clientId}`, error);
  });
});

// 发送消息
function sendMessage(ws, data) {
  ws.send(JSON.stringify(data));
}

// 生成客户端ID
function generateClientId() {
  return Math.random().toString(36).substring(2, 15);
}

// 处理消息
function handleMessage(clientId, data) {
  // 处理不同类型的消息
  // ...
}

// 启动服务器
server.listen(3000, () => {
  console.log('Server started on port 3000');
});
```

#### 消息类型

- **系统消息**：服务器状态、错误信息等
- **认证消息**：登录、注册等
- **命令消息**：玩家命令和响应
- **事件消息**：游戏事件通知
- **状态消息**：玩家和世界状态更新
- **聊天消息**：玩家之间的聊天

### 3. 数据处理

后端需要处理大量的游戏数据，包括玩家数据、世界状态、事件数据等。

#### 数据流程

1. 接收前端请求
2. 验证请求数据
3. 处理游戏逻辑
4. 更新数据库
5. 返回响应

#### 数据缓存策略

- 使用Redis缓存频繁访问的数据，如玩家基本信息、当前地图状态等
- 使用内存缓存热点数据，如事件模板、物品定义等
- 定期将缓存数据同步到MongoDB持久化存储

#### 数据一致性

- 使用事务确保关键操作的原子性
- 实现乐观锁或悲观锁处理并发修改
- 定期进行数据一致性检查和修复

## 数据库设计

### 1. MongoDB文档结构

MongoDB是游戏的主要持久化存储，用于存储玩家数据、世界状态、事件数据等。

#### 集合设计

- **players**：存储玩家数据
- **worlds**：存储游戏世界数据
- **events**：存储游戏事件数据
- **items**：存储物品定义
- **npcs**：存储NPC定义
- **maps**：存储地图数据
- **eventTemplates**：存储事件模板
- **logs**：存储游戏日志

#### 索引设计

```javascript
// 玩家集合索引
db.players.createIndex({ wxOpenId: 1 }, { unique: true });
db.players.createIndex({ nickname: 1 });
db.players.createIndex({ "position.mapId": 1 });

// 事件集合索引
db.events.createIndex({ status: 1 });
db.events.createIndex({ "observers": 1 });
db.events.createIndex({ createdAt: 1 });

// 地图集合索引
db.maps.createIndex({ mapId: 1 }, { unique: true });

// NPC集合索引
db.npcs.createIndex({ "position.mapId": 1 });
```

#### 数据模型示例

```javascript
// 玩家数据模型
const playerSchema = {
  _id: ObjectId,           // 玩家唯一ID
  wxOpenId: String,        // 微信OpenID
  nickname: String,        // 玩家昵称
  role: String,           // 社会角色
  attributes: {           // 玩家属性
    strength: Number,     // 力量
    intelligence: Number, // 智力
    charisma: Number      // 魅力
  },
  inventory: [            // 物品栏
    {
      itemId: ObjectId,   // 物品ID
      quantity: Number    // 数量
    }
  ],
  position: {             // 位置
    mapId: String,        // 地图ID
    x: Number,            // X坐标
    y: Number             // Y坐标
  },
  relationships: [        // 社交关系
    {
      playerId: ObjectId, // 关系玩家ID
      bondValue: Number,  // 羁绊值
      type: String        // 关系类型
    }
  ],
  questLog: [             // 任务日志
    {
      questId: ObjectId,  // 任务ID
      status: String,     // 任务状态
      progress: Number    // 任务进度
    }
  ],
  lastLogin: Date,        // 最后登录时间
  createdAt: Date         // 创建时间
};

// 事件数据模型
const eventSchema = {
  _id: ObjectId,           // 事件唯一ID
  type: String,            // 事件类型
  title: String,           // 事件标题
  description: String,     // 事件描述
  quantumStates: [         // 量子态
    {
      stateId: String,     // 状态ID
      probability: Number, // 概率
      description: String, // 状态描述
      outcomes: [          // 可能结果
        {
          outcomeId: String, // 结果ID
          description: String, // 结果描述
          effects: [        // 效果
            {
              type: String, // 效果类型
              target: String, // 效果目标
              value: Mixed   // 效果值
            }
          ]
        }
      ]
    }
  ],
  triggers: [              // 触发条件
    {
      type: String,        // 条件类型
      value: Mixed         // 条件值
    }
  ],
  observers: [ObjectId],   // 观测者列表
  status: String,          // 事件状态
  createdAt: Date,         // 创建时间
  resolvedAt: Date         // 解决时间
};
```

### 2. Redis缓存策略

Redis用于缓存频繁访问的数据和实现实时功能，如在线玩家列表、实时状态等。

#### 缓存内容

- **玩家会话**：存储玩家的会话信息和认证状态
- **在线玩家**：维护在线玩家列表
- **地图状态**：缓存当前地图状态，包括玩家位置、NPC位置等
- **事件状态**：缓存活跃事件的状态
- **消息队列**：实现玩家之间的消息传递

#### 数据结构

- **String**：存储简单键值对，如玩家基本信息
- **Hash**：存储复杂对象，如玩家详细信息
- **List**：存储有序列表，如消息队列
- **Set**：存储无序集合，如在线玩家列表
- **Sorted Set**：存储有序集合，如排行榜

#### 缓存示例

```javascript
// Redis客户端
const redis = require('redis');
const client = redis.createClient();

// 存储玩家会话
async function storePlayerSession(playerId, sessionData) {
  const key = `session:${playerId}`;
  await client.hSet(key, sessionData);
  await client.expire(key, 3600); // 1小时过期
}

// 获取玩家会话
async function getPlayerSession(playerId) {
  const key = `session:${playerId}`;
  return await client.hGetAll(key);
}

// 添加在线玩家
async function addOnlinePlayer(playerId, mapId) {
  await client.sAdd('online_players', playerId);
  await client.sAdd(`map:${mapId}:players`, playerId);
}

// 移除在线玩家
async function removeOnlinePlayer(playerId, mapId) {
  await client.sRem('online_players', playerId);
  await client.sRem(`map:${mapId}:players`, playerId);
}

// 获取在线玩家数量
async function getOnlinePlayerCount() {
  return await client.sCard('online_players');
}

// 获取地图上的玩家
async function getPlayersInMap(mapId) {
  return await client.sMembers(`map:${mapId}:players`);
}

// 缓存事件状态
async function cacheEventState(eventId, eventState) {
  const key = `event:${eventId}`;
  await client.hSet(key, eventState);
  await client.expire(key, 3600); // 1小时过期
}

// 获取事件状态
async function getEventState(eventId) {
  const key = `event:${eventId}`;
  return await client.hGetAll(key);
}
```

## 通信协议设计

### 1. 消息格式

前后端通信使用JSON格式的消息，包含消息类型、数据和元数据。

#### 基本消息格式

```javascript
{
  "type": "message_type",  // 消息类型
  "data": {                // 消息数据
    // 根据消息类型不同，包含不同的数据字段
  },
  "meta": {                // 元数据
    "timestamp": 1625097600000,  // 时间戳
    "messageId": "abc123",       // 消息ID
    "clientId": "client123"      // 客户端ID
  }
}
```

#### 常用消息类型

- **system**：系统消息
- **auth**：认证消息
- **command**：命令消息
- **event**：事件消息
- **state**：状态消息
- **chat**：聊天消息
- **error**：错误消息

### 2. 命令消息

命令消息用于玩家向服务器发送命令，格式如下：

```javascript
// 命令请求
{
  "type": "command",
  "data": {
    "command": "move",     // 命令名称
    "args": ["north"]      // 命令参数
  },
  "meta": {
    "timestamp": 1625097600000,
    "messageId": "cmd123",
    "clientId": "client123"
  }
}

// 命令响应
{
  "type": "command_response",
  "data": {
    "success": true,       // 命令是否成功
    "message": "你向北移动了一步，来到了一个新的房间。",  // 响应消息
    "state": {             // 状态更新
      "position": {
        "mapId": "forest",
        "x": 5,
        "y": 3
      }
    }
  },
  "meta": {
    "timestamp": 1625097600100,
    "messageId": "resp123",
    "clientId": "client123",
    "requestId": "cmd123"  // 对应的请求ID
  }
}
```

### 3. 事件消息

事件消息用于服务器向玩家推送事件，格式如下：

```javascript
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
    ]
  },
  "meta": {
    "timestamp": 1625097600200,
    "messageId": "event123",
    "clientId": "client123"
  }
}
```

### 4. 状态消息

状态消息用于服务器向玩家推送状态更新，格式如下：

```javascript
{
  "type": "state",
  "data": {
    "player": {            // 玩家状态
      "attributes": {
        "health": 95,
        "energy": 80
      },
      "position": {
        "mapId": "forest",
        "x": 5,
        "y": 3
      },
      "inventory": [
        {
          "itemId": "item123",
          "name": "木剑",
          "quantity": 1
        }
      ]
    },
    "world": {             // 世界状态
      "time": "day",
      "weather": "rain"
    }
  },
  "meta": {
    "timestamp": 1625097600300,
    "messageId": "state123",
    "clientId": "client123"
  }
}
```

### 5. 错误处理

错误消息用于服务器向玩家推送错误信息，格式如下：

```javascript
{
  "type": "error",
  "data": {
    "code": "invalid_command",  // 错误代码
    "message": "无效的命令",     // 错误消息
    "details": {               // 错误详情
      "command": "fly",
      "reason": "未知命令"
    }
  },
  "meta": {
    "timestamp": 1625097600400,
    "messageId": "error123",
    "clientId": "client123",
    "requestId": "cmd456"      // 对应的请求ID
  }
}
```

## 性能优化

### 1. 前端优化

- **资源加载优化**：
  - 使用分包加载，减少初始加载时间
  - 压缩代码和资源，减少下载大小
  - 使用缓存，减少重复加载

- **渲染优化**：
  - 使用Canvas批量渲