# 微信小游戏MUD量子叙事GM管理系统设计

## 1. 系统概述

GM管理系统是微信小游戏MUD量子叙事项目的重要组成部分，为游戏运营和管理提供一个集中化的控制平台。该系统允许管理员在不修改代码的情况下，动态管理游戏内容、监控游戏状态、处理玩家事务，以及配置AI内容生成系统。

### 1.1 设计目标

- **无代码操作**：所有管理功能通过界面操作完成，无需编写代码
- **全面管控**：覆盖游戏运营的各个方面，从内容管理到玩家服务
- **实时响应**：变更能够实时生效，提供即时反馈
- **安全可靠**：严格的权限控制和操作审计
- **易于使用**：直观的界面和工作流程，降低使用门槛
- **可扩展性**：模块化设计，便于添加新功能

### 1.2 用户角色

- **超级管理员**：拥有所有权限，负责系统配置和权限分配
- **内容管理员**：负责游戏内容的创建、编辑和审核
- **玩家管理员**：处理玩家账号、投诉和支持请求
- **运营管理员**：负责活动、礼包和公告的发布
- **系统监控员**：监控游戏性能和数据统计
- **AI内容审核员**：审核和调整AI生成的内容

## 2. 系统架构

### 2.1 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                      GM管理系统前端                          │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐         │
│  │ 内容管理 │  │ 玩家管理 │  │ 运营管理 │  │ 系统监控 │         │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘         │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐         │
│  │ AI管理   │  │ 日志查询 │  │ 权限管理 │  │ 系统设置 │         │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ HTTPS API
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      GM管理系统后端                          │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐         │
│  │ 认证服务 │  │ 权限服务 │  │ 操作日志 │  │ 数据缓存 │         │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘         │
│                                                             │
│  ┌─────────────────────────────────────────────────┐        │
│  │               业务逻辑处理层                      │        │
│  └─────────────────────────────────────────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ 内部API
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       游戏服务系统                           │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐         │
│  │ 游戏逻辑 │  │ 玩家数据 │  │ 内容系统 │  │ AI引擎  │         │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 技术选择

- **前端**：React + Ant Design，提供现代化、响应式的管理界面
- **后端**：Node.js + Express，与游戏主系统保持技术栈一致
- **数据库**：MongoDB，存储管理系统配置和操作日志
- **认证**：JWT (JSON Web Token)，安全的身份验证
- **API**：RESTful API + WebSocket，支持实时数据更新
- **部署**：与游戏服务器共享腾讯云服务器，通过Docker容器隔离

## 3. 核心功能模块

### 3.1 内容管理模块

#### 3.1.1 剧情内容管理

- **剧情浏览**：查看所有现有剧情，支持筛选和搜索
- **剧情编辑**：修改现有剧情的文本、选项和结果
- **剧情创建**：基于模板创建新的剧情内容
- **剧情测试**：在管理界面中预览剧情流程
- **剧情发布控制**：控制剧情的上线、下线和可见性

```javascript
// 剧情内容数据结构示例
{
  "id": "story_quantum_heist",
  "title": "量子金库抢劫",
  "author": "admin",
  "status": "published", // draft, published, archived
  "creationDate": "2023-11-20T10:30:00Z",
  "lastModified": "2023-11-21T15:45:00Z",
  "content": {
    "introduction": "你收到一封神秘邀请，要求你参与一场量子金库的抢劫行动...",
    "sections": [
      {
        "id": "section_1",
        "text": "你站在金库外，警卫正在巡逻。你决定...",
        "choices": [
          {
            "id": "choice_1_1",
            "text": "使用量子隐身能力",
            "nextSection": "section_2",
            "requirements": {
              "ability": "quantum_invisibility",
              "level": 2
            }
          },
          {
            "id": "choice_1_2",
            "text": "分散警卫注意力",
            "nextSection": "section_3"
          }
        ]
      }
      // 更多章节...
    ]
  },
  "rewards": {
    "experience": 500,
    "items": [
      {"id": "quantum_key", "count": 1},
      {"id": "energy_crystal", "count": 5}
    ]
  },
  "tags": ["heist", "quantum", "adventure"],
  "difficulty": "medium",
  "estimatedDuration": 30 // 分钟
}
```

#### 3.1.2 物品管理

- **物品库**：管理所有游戏物品，包括属性和效果
- **物品创建**：创建新物品，设置属性和量子状态
- **物品平衡调整**：修改物品属性以保持游戏平衡
- **物品掉落配置**：设置物品的获取途径和概率
- **物品批量操作**：批量导入/导出物品数据

#### 3.1.3 NPC管理

- **NPC列表**：查看和管理所有NPC角色
- **NPC编辑器**：编辑NPC的外观、对话和行为
- **对话树编辑**：可视化编辑NPC对话选项和分支
- **NPC调度**：设置NPC的出现位置和时间
- **NPC关系网络**：管理NPC之间的关系

#### 3.1.4 地图和区域管理

- **地图编辑器**：编辑游戏世界地图和区域
- **位置管理**：设置重要地点和传送点
- **环境配置**：设置区域的环境效果和事件
- **区域访问控制**：管理玩家对区域的访问权限
- **地图可视化**：直观展示世界地图和连接关系

### 3.2 玩家管理模块

#### 3.2.1 玩家账号管理

- **玩家列表**：查看所有玩家账号，支持多条件筛选
- **玩家详情**：查看玩家的详细信息和游戏数据
- **账号操作**：封禁/解封账号，重置密码
- **角色转移**：处理角色所有权转移请求
- **数据恢复**：从备份恢复玩家数据

```javascript
// 玩家管理界面数据结构示例
{
  "playerId": "player123456",
  "basicInfo": {
    "nickname": "量子旅行者",
    "registrationDate": "2023-10-15T08:20:00Z",
    "lastLogin": "2023-11-22T19:30:00Z",
    "totalPlayTime": 127.5, // 小时
    "status": "active" // active, suspended, banned
  },
  "gameData": {
    "level": 25,
    "experience": 12500,
    "quantumAffinity": 78,
    "completedQuests": 47,
    "discoveredAreas": 15,
    "inventory": {
      "itemCount": 86,
      "rareItems": 12,
      "legendaryItems": 3
    }
  },
  "socialData": {
    "friends": 8,
    "groupMemberships": 2,
    "reputationScore": 85
  },
  "monetization": {
    "totalSpent": 150, // 元
    "lastPurchase": "2023-11-10T14:25:00Z"
  },
  "flags": {
    "isVIP": true,
    "hasPendingReports": false,
    "isContentCreator": true
  }
}
```

#### 3.2.2 玩家支持系统

- **支持工单**：处理玩家提交的问题和请求
- **常见问题库**：管理常见问题和解答
- **自动回复配置**：设置自动回复规则
- **玩家通知**：向特定玩家或群体发送通知
- **问题分类与统计**：分析常见问题类型和解决率

#### 3.2.3 玩家行为管理

- **行为监控**：监控可疑的玩家行为
- **违规处理**：对违规行为进行警告或处罚
- **申诉处理**：处理玩家的申诉请求
- **行为规则配置**：设置和更新行为规则
- **自动检测设置**：配置自动违规检测规则

#### 3.2.4 玩家数据分析

- **玩家画像**：分析玩家的游戏习惯和偏好
- **留存分析**：跟踪玩家留存率和流失原因
- **行为热图**：可视化玩家活动和互动热点
- **进度分析**：分析玩家游戏进度分布
- **社交网络分析**：分析玩家社交关系网络

### 3.3 运营管理模块

#### 3.3.1 活动管理

- **活动创建**：创建各类游戏活动
- **活动调度**：设置活动的开始和结束时间
- **活动规则配置**：设置活动规则和奖励
- **活动数据跟踪**：监控活动参与度和效果
- **活动模板库**：管理可重用的活动模板

```javascript
// 活动配置示例
{
  "activityId": "quantum_festival_2023",
  "name": "2023量子节",
  "description": "庆祝量子裂隙世界的年度盛会，参与各种挑战赢取奖励！",
  "status": "scheduled", // draft, scheduled, active, ended, archived
  "schedule": {
    "startTime": "2023-12-01T00:00:00Z",
    "endTime": "2023-12-15T23:59:59Z",
    "reminderTime": "2023-11-28T12:00:00Z"
  },
  "visibility": {
    "minLevel": 5,
    "requiredQuests": ["quantum_basics"],
    "regions": ["all"]
  },
  "challenges": [
    {
      "id": "daily_quantum_puzzle",
      "name": "每日量子谜题",
      "description": "每天解开一个量子谜题",
      "type": "daily",
      "rewards": {
        "items": [{"id": "quantum_essence", "count": 10}],
        "experience": 200
      }
    },
    {
      "id": "reality_collector",
      "name": "现实收集者",
      "description": "收集5种不同的现实碎片",
      "type": "collection",
      "target": 5,
      "rewards": {
        "items": [{"id": "reality_shaper", "count": 1}],
        "experience": 500
      }
    }
  ],
  "globalRewards": {
    "participationThreshold": 3, // 完成3个挑战
    "rewards": {
      "items": [{"id": "festival_memory", "count": 1}],
      "title": "量子节参与者"
    }
  },
  "notifications": {
    "start": "量子节已经开始！快来参与各种挑战赢取奖励！",
    "reminder": "量子节即将开始，准备好了吗？",
    "ending": "量子节即将结束，别忘了领取你的奖励！"
  }
}
```

#### 3.3.2 礼包管理

- **礼包创建**：创建包含各种奖励的礼包
- **礼包码生成**：生成和管理兑换码
- **礼包分发**：向玩家分发礼包
- **兑换记录**：跟踪礼包的兑换情况
- **礼包策略**：设置礼包的发放策略和条件

#### 3.3.3 公告系统

- **公告创建**：创建游戏内公告
- **公告调度**：设置公告的显示时间和频率
- **公告模板**：管理可重用的公告模板
- **公告分发**：控制公告的目标受众
- **公告效果分析**：分析公告的阅读率和点击率

#### 3.3.4 商店管理

- **商品管理**：管理游戏内商店的商品
- **价格调整**：设置和调整商品价格
- **限时商品**：设置限时特卖商品
- **捆绑包**：创建和管理商品捆绑包
- **销售分析**：分析商品销售情况和趋势

### 3.4 系统监控模块

#### 3.4.1 性能监控

- **服务器状态**：监控服务器的运行状态
- **性能指标**：跟踪关键性能指标
- **负载分析**：分析系统负载和瓶颈
- **资源使用**：监控资源使用情况
- **警报设置**：配置性能警报阈值

```javascript
// 性能监控数据结构示例
{
  "serverStatus": {
    "id": "server-001",
    "status": "running", // running, warning, critical, maintenance
    "uptime": 1209600, // 秒，约14天
    "lastRestart": "2023-11-08T02:15:00Z"
  },
  "performance": {
    "cpu": {
      "usage": 45.2, // 百分比
      "temperature": 62.5 // 摄氏度
    },
    "memory": {
      "total": 16384, // MB
      "used": 10240, // MB
      "usage": 62.5 // 百分比
    },
    "disk": {
      "total": 500, // GB
      "used": 320, // GB
      "usage": 64 // 百分比
    },
    "network": {
      "inbound": 25.6, // Mbps
      "outbound": 18.3, // Mbps
      "connections": 1250
    }
  },
  "gameMetrics": {
    "onlinePlayers": 523,
    "peakPlayers": 1205,
    "activeInstances": 48,
    "averageResponseTime": 120, // ms
    "errorRate": 0.05 // 百分比
  },
  "alerts": [
    {
      "id": "alert-2023112201",
      "type": "warning",
      "message": "内存使用率超过60%",
      "timestamp": "2023-11-22T10:15:00Z",
      "acknowledged": false
    }
  ],
  "maintenanceSchedule": [
    {
      "id": "maint-20231125",
      "type": "routine",
      "description": "例行数据库优化",
      "scheduledStart": "2023-11-25T02:00:00Z",
      "estimatedDuration": 120, // 分钟
      "affectedServices": ["database"]
    }
  ]
}
```

#### 3.4.2 数据统计

- **玩家统计**：玩家数量、活跃度等统计
- **内容统计**：内容使用和参与度统计
- **经济统计**：游戏经济数据和趋势
- **自定义报表**：创建自定义统计报表
- **数据导出**：导出统计数据供进一步分析

#### 3.4.3 日志查询

- **操作日志**：查询管理员的操作记录
- **玩家日志**：查询玩家的活动日志
- **系统日志**：查询系统运行日志
- **错误日志**：查询系统错误和异常
- **日志分析**：分析日志中的模式和趋势

#### 3.4.4 预警系统

- **异常检测**：检测系统和游戏中的异常情况
- **预警规则**：配置预警触发条件
- **通知渠道**：设置预警通知方式
- **预警处理**：记录和跟踪预警处理过程
- **预警历史**：查看历史预警记录和处理结果

### 3.5 AI内容管理模块

#### 3.5.1 AI参数配置

- **模型选择**：选择和配置AI生成模型
- **参数调整**：调整AI生成的参数设置
- **提示词管理**：管理和优化AI提示词
- **风格控制**：控制AI生成内容的风格和语调
- **资源限制**：设置AI资源使用限制

```javascript
// AI配置示例
{
  "aiConfig": {
    "contentGeneration": {
      "model": "gpt-4",
      "temperature": 0.7,
      "maxTokens": 2000,
      "frequencyPenalty": 0.5,
      "presencePenalty": 0.5,
      "stopSequences": ["END_OF_CONTENT"]
    },
    "dialogueGeneration": {
      "model": "gpt-3.5-turbo",
      "temperature": 0.8,
      "maxTokens": 500,
      "characterConsistency": 0.9
    },
    "itemGeneration": {
      "model": "custom-item-generator",
      "balanceConstraints": true,
      "rarityDistribution": {
        "common": 0.6,
        "uncommon": 0.25,
        "rare": 0.1,
        "legendary": 0.05
      }
    }
  },
  "promptTemplates": {
    "storyEvent": "创建一个发生在{location}的量子事件，涉及{theme}主题，包含{numChoices}个玩家选择。事件应该具有{tone}的语调，并与玩家的{playerBackground}背景相关。",
    "npcDialogue": "为{npcName}创建一段对话，这个NPC是{npcDescription}。对话应该关于{topic}，并反映NPC的{personality}性格。如果玩家有{playerStatus}状态，对话应该有所不同。",
    "itemDescription": "创建一个{itemType}类型的物品描述，稀有度为{rarity}。物品应该与{theme}主题相关，并具有{effectType}效果。描述应该简洁但生动，突出物品的独特性。"
  },
  "generationLimits": {
    "dailyRequestLimit": 1000,
    "concurrentRequests": 10,
    "tokensPerDay": 1000000
  },
  "qualityControl": {
    "minimumAcceptanceScore": 0.7,
    "automaticRejectionThreshold": 0.3,
    "requireHumanReviewThreshold": 0.5
  }
}
```

#### 3.5.2 内容审核

- **生成队列**：查看和管理AI内容生成队列
- **内容审核**：审核AI生成的内容
- **修改建议**：提供内容修改建议
- **批量处理**：批量审核和处理内容
- **审核标准**：设置和更新内容审核标准

#### 3.5.3 内容调优

- **反馈分析**：分析玩家对AI内容的反馈
- **质量评分**：评估AI内容的质量
- **模型微调**：基于反馈微调AI模型
- **A/B测试**：设置不同AI参数的A/B测试
- **效果对比**：比较不同AI设置的效果

#### 3.5.4 内容发布

- **发布审批**：审批AI内容的发布
- **发布调度**：设置内容的发布时间
- **发布范围**：控制内容的可见范围
- **发布回滚**：在必要时回滚已发布的内容
- **发布历史**：查看内容发布的历史记录

### 3.6 系统设置模块

#### 3.6.1 用户管理

- **管理员账号**：创建和管理管理员账号
- **角色管理**：定义和分配用户角色
- **权限配置**：配置角色的权限
- **登录设置**：配置登录安全策略
- **操作日志**：查看用户操作日志

```javascript
// 用户角色配置示例
{
  "roles": [
    {
      "id": "super_admin",
      "name": "超级管理员",
      "description": "拥有系统所有权限",
      "permissions": ["*"],
      "maxUsers": 2
    },
    {
      "id": "content_manager",
      "name": "内容管理员",
      "description": "管理游戏内容",
      "permissions": [
        "content.view",
        "content.create",
        "content.edit",
        "content.publish",
        "content.delete",
        "ai.review"
      ],
      "restrictedAreas": []
    },
    {
      "id": "player_support",
      "name": "玩家支持",
      "description": "处理玩家问题和支持请求",
      "permissions": [
        "player.view",
        "player.edit",
        "player.support",
        "player.items.grant",
        "logs.view"
      ],
      "restrictedAreas": ["system.settings"]
    }
  ],
  "users": [
    {
      "id": "admin001",
      "username": "admin",
      "email": "admin@example.com",
      "role": "super_admin",
      "status": "active",
      "lastLogin": "2023-11-22T09:15:00Z",
      "twoFactorEnabled": true
    },
    {
      "id": "content001",
      "username": "content_editor",
      "email": "content@example.com",
      "role": "content_manager",
      "status": "active",
      "lastLogin": "2023-11-21T14:30:00Z",
      "twoFactorEnabled": false
    }
  ]
}
```

#### 3.6.2 系统配置

- **基本设置**：配置系统基本参数
- **集成设置**：配置与其他系统的集成
- **通知设置**：配置系统通知方式
- **备份设置**：配置数据备份策略
- **维护模式**：管理系统维护模式

#### 3.6.3 游戏参数

- **游戏规则**：配置游戏核心规则参数
- **平衡调整**：调整游戏平衡参数
- **经济参数**：设置游戏经济相关参数
- **进度参数**：配置游戏进度和难度参数
- **参数历史**：查看参数调整的历史记录

## 4. 权限管理系统

### 4.1 权限模型

GM管理系统采用基于角色的访问控制（RBAC）模型，将权限分配给角色，再将角色分配给用户。

#### 4.1.1 权限定义

权限使用`资源.操作`的格式定义，例如：
- `content.view`：查看内容
- `player.ban`：封禁玩家
- `system.settings`：修改系统设置

#### 4.1.2 权限分组

权限按功能模块分组，便于管理：
- **内容权限**：与游戏内容相关的权限
- **玩家权限**：与玩家管理相关的权限
- **运营权限**：与游戏运营相关的权限
- **系统权限**：与系统管理相关的权限
- **AI权限**：与AI内容管理相关的权限

#### 4.1.3 权限继承

角色可以继承其他角色的权限，形成权限层级：
- **基础角色**：包含基本操作权限
- **专业角色**：继承基础角色，添加专业操作权限
- **管理角色**：继承专业角色，添加管理权限
- **超级管理员**：拥有所有权限

### 4.2 权限控制实现

```javascript
// 权限检查中间件
function checkPermission(permission) {
  return async (req, res, next) => {
    try {
      const user = req.user;
      
      // 超级管理员拥有所有权限
      if (user.role === 'super_admin') {
        return next();
      }
      
      // 获取用户角色的权限
      const role = await RoleModel.findById(user.roleId);
      
      // 检查是否有通配符权限
      if (role.permissions.includes('*') || role.permissions.includes(permission.split('.')[0] + '.*')) {
        return next();
      }
      
      // 检查具体权限
      if (role.permissions.includes(permission)) {
        return next();
      }
      
      // 无权限
      return res.status(403).json({
        success: false,
        message: '没有权限执行此操作'
      });