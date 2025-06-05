# 微信小游戏MUD量子叙事引擎设计

## 引擎概述

量子叙事引擎是MUD量子叙事游戏的核心组件，负责生成和管理游戏中的事件和剧情。该引擎基于量子力学的概念，将游戏事件视为处于量子叠加态，直到玩家观测时才坍缩为确定状态。这种设计使游戏剧情具有高度的不确定性和可能性，每个玩家的游戏体验都是独特的。

在微信小游戏版本中，我们对原有的量子叙事引擎进行了简化和优化，使其适合在单台服务器上运行，同时保留其核心特性。

## 核心组件

### 1. 量子态解析器

量子态解析器负责管理事件的量子态，包括事件的可能状态和概率分布。

#### 数据结构

```javascript
// 量子事件状态
class QuantumEventState {
  constructor(id, description, probability = 1.0) {
    this.id = id;                 // 状态ID
    this.description = description; // 状态描述
    this.probability = probability; // 状态概率
    this.outcomes = [];           // 可能的结果
    this.observers = [];          // 观测者列表
  }
  
  // 添加可能的结果
  addOutcome(outcome) {
    this.outcomes.push(outcome);
  }
  
  // 添加观测者
  addObserver(playerId) {
    if (!this.observers.includes(playerId)) {
      this.observers.push(playerId);
    }
  }
  
  // 计算坍缩后的状态
  collapse() {
    // 根据概率分布选择一个结果
    const random = Math.random();
    let cumulativeProbability = 0;
    
    for (const outcome of this.outcomes) {
      cumulativeProbability += outcome.probability;
      if (random <= cumulativeProbability) {
        return outcome;
      }
    }
    
    // 默认返回最后一个结果
    return this.outcomes[this.outcomes.length - 1];
  }
}

// 量子事件结果
class QuantumEventOutcome {
  constructor(id, description, probability = 1.0) {
    this.id = id;                 // 结果ID
    this.description = description; // 结果描述
    this.probability = probability; // 结果概率
    this.effects = [];            // 结果效果
  }
  
  // 添加效果
  addEffect(effect) {
    this.effects.push(effect);
  }
}

// 量子事件效果
class QuantumEventEffect {
  constructor(type, target, value) {
    this.type = type;     // 效果类型（属性变化、物品获得、状态改变等）
    this.target = target; // 效果目标（玩家、NPC、环境等）
    this.value = value;   // 效果值
  }
  
  // 应用效果
  apply(gameState) {
    // 根据效果类型应用效果
    switch (this.type) {
      case 'attribute':
        // 修改属性
        gameState.modifyAttribute(this.target, this.value);
        break;
      case 'item':
        // 添加或移除物品
        gameState.modifyInventory(this.target, this.value);
        break;
      case 'status':
        // 改变状态
        gameState.modifyStatus(this.target, this.value);
        break;
      // 其他效果类型...
    }
  }
}
```

### 2. 马尔可夫链处理器

马尔可夫链处理器负责管理事件之间的转换概率，使游戏剧情具有连贯性和因果关系。

#### 数据结构

```javascript
// 马尔可夫链处理器
class MarkovChainProcessor {
  constructor() {
    this.transitionMatrix = new Map(); // 状态转移矩阵
  }
  
  // 添加状态转移概率
  addTransition(fromState, toState, probability) {
    if (!this.transitionMatrix.has(fromState)) {
      this.transitionMatrix.set(fromState, new Map());
    }
    
    this.transitionMatrix.get(fromState).set(toState, probability);
  }
  
  // 获取下一个可能的状态
  getNextStates(currentState) {
    if (!this.transitionMatrix.has(currentState)) {
      return [];
    }
    
    const transitions = this.transitionMatrix.get(currentState);
    const nextStates = [];
    
    for (const [state, probability] of transitions.entries()) {
      nextStates.push({ state, probability });
    }
    
    return nextStates;
  }
  
  // 根据当前状态和条件选择下一个状态
  selectNextState(currentState, conditions = {}) {
    const nextStates = this.getNextStates(currentState);
    
    if (nextStates.length === 0) {
      return null;
    }
    
    // 根据条件调整概率
    const adjustedStates = nextStates.map(({ state, probability }) => {
      let adjustedProbability = probability;
      
      // 根据条件调整概率
      for (const [condition, value] of Object.entries(conditions)) {
        if (state.conditions && state.conditions[condition]) {
          adjustedProbability *= state.conditions[condition](value);
        }
      }
      
      return { state, probability: adjustedProbability };
    });
    
    // 归一化概率
    const totalProbability = adjustedStates.reduce((sum, { probability }) => sum + probability, 0);
    const normalizedStates = adjustedStates.map(({ state, probability }) => ({
      state,
      probability: probability / totalProbability
    }));
    
    // 根据概率选择下一个状态
    const random = Math.random();
    let cumulativeProbability = 0;
    
    for (const { state, probability } of normalizedStates) {
      cumulativeProbability += probability;
      if (random <= cumulativeProbability) {
        return state;
      }
    }
    
    // 默认返回最后一个状态
    return normalizedStates[normalizedStates.length - 1].state;
  }
}
```

### 3. 三层处理系统

三层处理系统负责处理事件的不同维度，包括道德抉择、环境变量和时间压力。

#### 数据结构

```javascript
// 三层处理系统
class ThreeLayerProcessor {
  constructor() {
    this.moralLayer = new MoralLayer();       // 道德抉择层
    this.environmentLayer = new EnvironmentLayer(); // 环境变量层
    this.timeLayer = new TimeLayer();         // 时间压力层
  }
  
  // 处理事件
  processEvent(event, player, gameState) {
    // 道德层处理
    const moralResult = this.moralLayer.process(event, player, gameState);
    
    // 环境层处理
    const environmentResult = this.environmentLayer.process(event, player, gameState);
    
    // 时间层处理
    const timeResult = this.timeLayer.process(event, player, gameState);
    
    // 综合处理结果
    return this.combineResults(moralResult, environmentResult, timeResult);
  }
  
  // 综合处理结果
  combineResults(moralResult, environmentResult, timeResult) {
    // 根据三层结果计算最终结果
    // 这里可以使用加权平均或其他方法
    return {
      description: this.combineDescriptions(moralResult.description, environmentResult.description, timeResult.description),
      effects: [...moralResult.effects, ...environmentResult.effects, ...timeResult.effects],
      nextEventProbabilities: this.combineNextEventProbabilities(moralResult.nextEventProbabilities, environmentResult.nextEventProbabilities, timeResult.nextEventProbabilities)
    };
  }
  
  // 综合描述
  combineDescriptions(moralDescription, environmentDescription, timeDescription) {
    // 根据三层描述生成最终描述
    return `${moralDescription}\n\n${environmentDescription}\n\n${timeDescription}`;
  }
  
  // 综合下一个事件概率
  combineNextEventProbabilities(moralProbabilities, environmentProbabilities, timeProbabilities) {
    // 合并三层的下一个事件概率
    const combinedProbabilities = new Map();
    
    // 处理道德层概率
    for (const [eventId, probability] of moralProbabilities.entries()) {
      combinedProbabilities.set(eventId, (combinedProbabilities.get(eventId) || 0) + probability * 0.4);
    }
    
    // 处理环境层概率
    for (const [eventId, probability] of environmentProbabilities.entries()) {
      combinedProbabilities.set(eventId, (combinedProbabilities.get(eventId) || 0) + probability * 0.3);
    }
    
    // 处理时间层概率
    for (const [eventId, probability] of timeProbabilities.entries()) {
      combinedProbabilities.set(eventId, (combinedProbabilities.get(eventId) || 0) + probability * 0.3);
    }
    
    return combinedProbabilities;
  }
}

// 道德抉择层
class MoralLayer {
  process(event, player, gameState) {
    // 处理道德抉择
    // ...
    
    return {
      description: "道德抉择结果描述",
      effects: [],
      nextEventProbabilities: new Map()
    };
  }
}

// 环境变量层
class EnvironmentLayer {
  process(event, player, gameState) {
    // 处理环境变量
    // ...
    
    return {
      description: "环境变量结果描述",
      effects: [],
      nextEventProbabilities: new Map()
    };
  }
}

// 时间压力层
class TimeLayer {
  process(event, player, gameState) {
    // 处理时间压力
    // ...
    
    return {
      description: "时间压力结果描述",
      effects: [],
      nextEventProbabilities: new Map()
    };
  }
}
```

### 4. 观测者系统

观测者系统负责处理玩家观测对事件状态的影响，实现量子态坍缩。

#### 数据结构

```javascript
// 观测者系统
class ObserverSystem {
  constructor() {
    this.observedEvents = new Map(); // 已观测事件
  }
  
  // 添加观测者
  addObserver(eventId, playerId) {
    if (!this.observedEvents.has(eventId)) {
      this.observedEvents.set(eventId, new Set());
    }
    
    this.observedEvents.get(eventId).add(playerId);
    
    return this.observedEvents.get(eventId).size;
  }
  
  // 检查事件是否被观测
  isObserved(eventId) {
    return this.observedEvents.has(eventId) && this.observedEvents.get(eventId).size > 0;
  }
  
  // 获取事件的观测者数量
  getObserverCount(eventId) {
    if (!this.observedEvents.has(eventId)) {
      return 0;
    }
    
    return this.observedEvents.get(eventId).size;
  }
  
  // 获取事件的观测者列表
  getObservers(eventId) {
    if (!this.observedEvents.has(eventId)) {
      return [];
    }
    
    return Array.from(this.observedEvents.get(eventId));
  }
  
  // 清除事件的观测者
  clearObservers(eventId) {
    if (this.observedEvents.has(eventId)) {
      this.observedEvents.delete(eventId);
    }
  }
}
```

## 事件系统与状态转换机制

### 事件生命周期

1. **事件创建**：系统根据游戏状态和触发条件创建事件，设置初始量子态。
2. **量子叠加态**：事件处于量子叠加态，包含多种可能的状态和结果。
3. **玩家观测**：玩家与事件交互，成为观测者。
4. **量子态坍缩**：根据观测者和概率分布，事件坍缩为确定状态。
5. **结果应用**：系统应用事件结果，更新游戏状态。
6. **事件链接**：根据马尔可夫链，当前事件可能触发后续事件。

### 状态转换流程

```
┌─────────────┐     触发条件满足     ┌─────────────┐     玩家观测     ┌─────────────┐
│             │──────────────────────►│             │─────────────────►│             │
│  潜在事件   │                      │ 量子叠加态  │                  │  确定状态   │
│             │                      │             │                  │             │
└─────────────┘                      └─────────────┘                  └─────────────┘
                                           │                                │
                                           │                                │
                                           │                                │
                                           ▼                                ▼
                                    ┌─────────────┐                  ┌─────────────┐
                                    │             │                  │             │
                                    │  事件取消   │                  │  结果应用   │
                                    │             │                  │             │
                                    └─────────────┘                  └─────────────┘
                                                                            │
                                                                            │
                                                                            │
                                                                            ▼
                                                                     ┌─────────────┐
                                                                     │             │
                                                                     │  后续事件   │
                                                                     │             │
                                                                     └─────────────┘
```

## 马尔可夫链实现方案

马尔可夫链是量子叙事引擎的核心机制之一，用于管理事件之间的转换概率。在微信小游戏版本中，我们采用简化的马尔可夫链实现，减少计算复杂度。

### 状态转移矩阵

状态转移矩阵存储事件之间的转换概率，使用稀疏矩阵表示，减少内存占用。

```javascript
// 状态转移矩阵示例
const transitionMatrix = {
  'event1': {
    'event2': 0.5,
    'event3': 0.3,
    'event4': 0.2
  },
  'event2': {
    'event5': 0.7,
    'event6': 0.3
  },
  // ...
};
```

### 条件概率调整

根据游戏状态和玩家行为调整转换概率，使事件转换更加动态和个性化。

```javascript
// 条件概率调整示例
function adjustProbability(baseProb, conditions) {
  let adjustedProb = baseProb;
  
  // 根据玩家属性调整
  if (conditions.playerAttributes) {
    for (const [attr, value] of Object.entries(conditions.playerAttributes)) {
      // 根据属性值调整概率
      adjustedProb *= (1 + (value - 50) / 100);
    }
  }
  
  // 根据环境因素调整
  if (conditions.environment) {
    for (const [factor, value] of Object.entries(conditions.environment)) {
      // 根据环境因素调整概率
      adjustedProb *= (1 + value / 10);
    }
  }
  
  // 确保概率在有效范围内
  return Math.max(0, Math.min(1, adjustedProb));
}
```

### 事件链生成

根据马尔可夫链生成事件链，形成连贯的剧情线。

```javascript
// 事件链生成示例
function generateEventChain(startEvent, length, conditions) {
  const chain = [startEvent];
  let currentEvent = startEvent;
  
  for (let i = 0; i < length - 1; i++) {
    const nextEvent = selectNextEvent(currentEvent, conditions);
    
    if (!nextEvent) {
      break;
    }
    
    chain.push(nextEvent);
    currentEvent = nextEvent;
    
    // 更新条件
    updateConditions(conditions, nextEvent);
  }
  
  return chain;
}

// 选择下一个事件
function selectNextEvent(currentEvent, conditions) {
  if (!transitionMatrix[currentEvent]) {
    return null;
  }
  
  const transitions = transitionMatrix[currentEvent];
  const adjustedTransitions = {};
  
  // 调整概率
  for (const [nextEvent, prob] of Object.entries(transitions)) {
    adjustedTransitions[nextEvent] = adjustProbability(prob, conditions);
  }
  
  // 归一化概率
  const totalProb = Object.values(adjustedTransitions).reduce((sum, prob) => sum + prob, 0);
  
  if (totalProb === 0) {
    return null;
  }
  
  for (const nextEvent in adjustedTransitions) {
    adjustedTransitions[nextEvent] /= totalProb;
  }
  
  // 根据概率选择下一个事件
  const random = Math.random();
  let cumulativeProb = 0;
  
  for (const [nextEvent, prob] of Object.entries(adjustedTransitions)) {
    cumulativeProb += prob;
    
    if (random <= cumulativeProb) {
      return nextEvent;
    }
  }
  
  // 默认返回第一个事件
  return Object.keys(adjustedTransitions)[0];
}
```

## Node.js环境中的实现细节

### 1. 事件处理模块

```javascript
// events/EventProcessor.js
const QuantumEventState = require('./QuantumEventState');
const MarkovChainProcessor = require('./MarkovChainProcessor');
const ThreeLayerProcessor = require('./ThreeLayerProcessor');
const ObserverSystem = require('./ObserverSystem');

class EventProcessor {
  constructor(db) {
    this.db = db;
    this.markovChain = new MarkovChainProcessor();
    this.threeLayerProcessor = new ThreeLayerProcessor();
    this.observerSystem = new ObserverSystem();
    
    // 加载事件模板
    this.eventTemplates = new Map();
    this.loadEventTemplates();
  }
  
  // 加载事件模板
  async loadEventTemplates() {
    try {
      const templates = await this.db.collection('eventTemplates').find().toArray();
      
      for (const template of templates) {
        this.eventTemplates.set(template.id, template);
      }
      
      console.log(`Loaded ${this.eventTemplates.size} event templates`);
    } catch (error) {
      console.error('Failed to load event templates:', error);
    }
  }
  
  // 创建事件
  async createEvent(templateId, triggerConditions = {}) {
    try {
      const template = this.eventTemplates.get(templateId);
      
      if (!template) {
        throw new Error(`Event template ${templateId} not found`);
      }
      
      // 创建事件
      const event = {
        _id: new ObjectId(),
        templateId,
        title: template.title,
        description: template.description,
        quantumStates: template.quantumStates.map(state => ({
          ...state,
          observers: []
        })),
        triggers: triggerConditions,
        status: 'quantum',
        createdAt: new Date(),
        resolvedAt: null
      };
      
      // 保存事件
      await this.db.collection('events').insertOne(event);
      
      return event;
    } catch (error) {
      console.error('Failed to create event:', error);
      throw error;
    }
  }
  
  // 观测事件
  async observeEvent(eventId, playerId) {
    try {
      // 添加观测者
      const observerCount = this.observerSystem.addObserver(eventId, playerId);
      
      // 更新数据库
      await this.db.collection('events').updateOne(
        { _id: eventId },
        { $addToSet: { observers: playerId } }
      );
      
      // 检查是否需要坍缩
      if (observerCount >= 1) {
        await this.collapseEvent(eventId);
      }
      
      return true;
    } catch (error) {
      console.error('Failed to observe event:', error);
      throw error;
    }
  }
  
  // 坍缩事件
  async collapseEvent(eventId) {
    try {
      // 获取事件
      const event = await this.db.collection('events').findOne({ _id: eventId });
      
      if (!event) {
        throw new Error(`Event ${eventId} not found`);
      }
      
      if (event.status !== 'quantum') {
        // 事件已经坍缩
        return event;
      }
      
      // 获取观测者
      const observers = this.observerSystem.getObservers(eventId);
      
      // 获取游戏状态
      const gameState = await this.getGameState(observers);
      
      // 处理事件
      const result = this.threeLayerProcessor.processEvent(event, observers[0], gameState);
      
      // 更新事件状态
      await this.db.collection('events').updateOne(
        { _id: eventId },
        {
          $set: {
            status: 'collapsed',
            result,
            resolvedAt: new Date()
          }
        }
      );
      
      // 应用结果
      await this.applyEventResult(result, observers, gameState);
      
      // 生成后续事件
      await this.generateNextEvents(event, result, gameState);
      
      return result;
    } catch (error) {
      console.error('Failed to collapse event:', error);
      throw error;
    }
  }
  
  // 获取游戏状态
  async getGameState(playerIds) {
    // 获取玩家数据
    const players = await this.db.collection('players').find({
      _id: { $in: playerIds }
    }).toArray();
    
    // 获取世界数据
    const world = await this.db.collection('worlds').findOne();
    
    return {
      players,
      world
    };
  }
  
  // 应用事件结果
  async applyEventResult(result, playerIds, gameState) {
    // 应用效果
    for (const effect of result.effects) {
      await this.applyEffect(effect, playerIds, gameState);
    }
  }
  
  // 应用效果
  async applyEffect(effect, playerIds, gameState) {
    switch (effect.type) {
      case 'attribute':
        // 修改属性
        await this.db.collection('players').updateOne(
          { _id: effect.target },
          { $inc: { [`attributes.${effect.attribute}`]: effect.value } }
        );
        break;
      case 'item':
        // 添加或移除物品
        if (effect.value > 0) {
          // 添加物品
          await this.db.collection('players').updateOne(
            { _id: effect.target },
            { $inc: { [`inventory.${effect.itemId}`]: effect.value } }
          );
        } else {
          // 移除物品
          await this.db.collection('players').updateOne(
            { _id: effect.target },
            { $inc: { [`inventory.${effect.itemId}`]: effect.value } }
          );
        }
        break;
      case 'status':
        // 改变状态
        await this.db.collection('players').updateOne(
          { _id: effect.target },
          { $set: { [`status.${effect.status}`]: effect.value } }
        );
        break;
      // 其他效果类型...
    }
  }
  
  // 生成后续事件
  async generateNextEvents(event, result, gameState) {
    // 获取下一个事件概率
    const nextEventProbabilities = result.nextEventProbabilities;
    
    // 选择下一个事件
    for (const [eventId, probability] of nextEventProbabilities.entries()) {
      // 根据概率决定是否生成事件
      if (Math.random() <= probability) {
        // 创建事件
        await this.createEvent(eventId, {
          parentEventId: event._id
        });
      }
    }
  }
}

module.exports = EventProcessor;
```

### 2. WebSocket通信模块

```javascript
// network/WebSocketManager.js
const WebSocket = require('ws');
const EventProcessor = require('../events/EventProcessor');

class WebSocketManager {
  constructor(server, db) {
    this.wss = new WebSocket.Server({ server });
    this.clients = new Map();
    this.db = db;
    this.eventProcessor = new EventProcessor(db);
    
    this.setupWebSocketServer();
  }
  
  // 设置WebSocket服务器
  setupWebSocketServer() {
    this.wss.on('connection', (ws, req) => {
      console.log('New WebSocket connection');
      
      // 生成客户端ID
      const clientId = this.generateClientId();
      
      // 存储客户端连接
      this.clients.set(clientId, {
        ws,
        playerId: null,
        lastActivity: Date.now()
      });
      
      // 发送欢迎消息
      this.sendMessage(ws, {
        type: 'welcome',
        clientId
      });
      
      // 处理消息
      ws.on('message', (message) => {
        try {
          const data = JSON.parse(message);
          this.handleMessage(clientId, data);
        } catch (error) {
          console.error('Failed to parse message:', error);
          this.sendMessage(ws, {
            type: 'error',
            message: 'Invalid message format'
          });
        }
      });
      
      // 处理连接关闭
      ws.on('close', () => {
        console.log(`WebSocket connection closed: ${clientId}`);
        this.clients.delete(clientId);
      });
      
      // 处理错误
      ws.on('error', (error) => {
        console.error(`WebSocket error: ${clientId}`, error);
      });
    });
    
    // 定期清理过期连接
    setInterval(() => {
      this.cleanupConnections();
    }, 60000);
  }
  
  // 生成客户端ID
  generateClientId() {
    return Math.random().toString(36).substring(2, 15);
  }
  
  // 处理消息
  async handleMessage(clientId, data) {
    const client = this.clients.get(clientId);
    
    if (!client) {
      return;
    }
    
    // 更新最后活动时间
    client.lastActivity = Date.now();
    
    switch (data.type) {
      case 'login':
        // 处理登录
        await this.handleLogin(clientId, data);
        break;
      case 'command':
        // 处理命令
        await this.handleCommand(clientId, data);
        break;
      case 'observe':
        // 处理观测事件
        await this.handleObserveEvent(clientId, data);
        break;
      // 其他消息类型...
      default:
        this.sendMessage(client.ws, {
          type: 'error',
          message: `Unknown message type: ${data.type}`
        });
    }
  }
  
  // 处理登录
  async handleLogin(clientId, data) {
    try {
      const client = this.clients.get(clientId);
      
      // 验证微信登录
      const wxOpenId = data.wxOpenId;
      
      // 查找玩家
      let player = await this.db.collection('players').findOne({ wxOpenId });
      
      if (!player) {
        // 创建新玩家
        player = {
          _id: new ObjectId(),
          wxOpenId,
          nickname: data.nickname || `Player_${wxOpenId.substring(0, 6)}`,
          role: 'citizen',
          attributes: {
            strength: 10,
            intelligence: 10,
            charisma: 10
          },
          inventory: {},
          position: {
            mapId: 'start',
            x: 0,
            y: 0
          },
          relationships: [],
          questLog: [],
          lastLogin: new Date(),
          createdAt: new Date()
        };
        
        await this.db.collection('players').insertOne(player);
      } else {