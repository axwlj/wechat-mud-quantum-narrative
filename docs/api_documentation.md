# MUD量子叙事游戏API文档

## 服务发现相关接口

### 服务注册

#### POST `/register`

**请求参数**

| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| serviceName | string | 是 | 服务名称（如player-service） |
| host | string | 是 | 服务主机地址 |
| port | number | 是 | 服务端口 |
| healthCheckPath | string | 否 | 健康检查路径 |

**请求示例**
```json
{
  "serviceName": "player-service",
  "host": "192.168.1.10",
  "port": 3000,
  "healthCheckPath": "/health"
}
```

**响应格式**
```json
{
  "success": true,
  "serviceId": "player-service-1234"
}
```

### 健康检查

#### GET `/health`

**响应格式**
```json
{
  "status": "healthy",
  "services": {
    "player-service": "healthy",
    "story-service": "healthy"
  },
  "timestamp": 1678901234567
}
```

## 量子叙事相关接口

### 生成叙事分支

#### POST `/api/ai/generate-narrative`

**请求参数**

| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| playerId | string | 是 | 玩家唯一标识 |
| context | object | 是 | 上下文对象 |

**请求示例**
```json
{
  "playerId": "player123",
  "context": {
    "currentState": "量子叠加态",
    "availableActions": ["观察", "交互"]
  }
}
```

**响应格式**

```json
{
  "branch": {
    "id": "branch456",
    "description": "坍缩后的叙事分支",
    "outcomes": ["事件A", "事件B"],
    "probability": 0.8
  }
}
```

**错误码**

- 400: 请求参数错误
- 500: 内部服务器错误