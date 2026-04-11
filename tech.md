# Tech Design for Multiplayer Networked UNO

## 1. Project Goal

本项目使用 **Java-only** 技术栈实现一个 **Multiplayer Networked UNO with Custom Action Cards**。系统支持 2 到 4 名参与者进入同一房间进行实时对战，采用 **client-server architecture**，其中 **server 是唯一可信的游戏状态来源**，负责回合推进、合法性校验、抽牌与弃牌堆管理、动作牌效果处理，以及对所有客户端广播最新状态。

在标准多人模式之外，系统还支持一个 **very simple rule-based automated player**。该 bot 不做复杂策略，只执行最简单的规则：

- 按当前手牌顺序扫描
- 打出第一张合法可出的牌
- 如果没有合法牌则抽一张
- 抽到可出则立即出，否则结束回合

项目还包含两个自定义卡牌：

- **Group Draw**
- **Triple Peek**

数据库部分仅用于保存玩家资料、对局历史和基础统计，不参与实时回合判定。

---

## 2. Design Principles

### 2.1 Server Authoritative

所有游戏规则必须由服务端统一判定，客户端只负责：

- 展示状态
- 接收用户输入
- 向服务端发送请求
- 接收服务端返回的新状态

客户端不能自行决定某张牌能否打出，也不能直接修改游戏状态。

### 2.2 Java-only and Pragmatic Scope

由于项目只能使用 Java，因此技术选型必须偏向稳定、容易调试、适合课程项目，而不是追求华丽框架。

### 2.3 Playable First, Features Later

开发顺序必须遵循：

1. 先打通最小可运行闭环
2. 再实现核心规则
3. 再加动作牌
4. 再加 bot
5. 再加自定义卡
6. 最后再加数据库

这意味着数据库和美术素材都不是第一优先级。

---

## 3. Technology Stack

### 3.1 Language and Build

- **Java 17**
- **Maven**
- IDE: **IntelliJ IDEA**

选择 Java 17 是因为语法和生态都足够稳定，Maven 则更适合课程项目管理依赖和构建流程。

### 3.2 GUI

- **Swing**

选择 Swing 的原因：

- JDK 原生支持
- 配置成本低
- 适合桌面课程项目
- 足够实现房间页、游戏桌面、手牌展示和按钮交互

### 3.3 Networking

- **ServerSocket**
- **Socket**
- **ObjectInputStream / ObjectOutputStream**

网络层使用 Java 原生 socket 完成客户端和服务端之间的实时通信。消息将使用可序列化对象传输，而不是手写字符串协议，以降低协议解析错误的风险。

### 3.4 Concurrency

- **Thread**
- **ExecutorService**
- 必要时使用 **synchronized** 或房间级锁

服务端会为多个客户端连接提供并发处理能力，但核心目标不是高吞吐量，而是保证：

- 同一时刻只有合法的当前玩家可以行动
- 状态更新不会乱序
- bot 不会重复执行动作
- 广播不会漏发

### 3.5 Persistence

- **JDBC**
- **SQLite**

SQLite 足以满足课程项目的玩家信息、比赛历史和统计数据存储需求，且不需要额外部署数据库服务。

### 3.6 Bot Module

- Java server-side module
- 简单规则驱动，不使用机器学习，不使用搜索算法

bot 必须运行在服务端，而不是客户端，否则会带来同步和作弊模型混乱的问题。

---

## 4. Rendering Strategy for Cards

### 4.1 Do We Need Image Assets?

前期 **不需要手动导入牌面素材**。

MVP 阶段可以直接使用 Java Swing 自绘：

- 用圆角矩形表示一张牌
- 用不同背景色表示颜色牌
- 用文本显示数字或动作类型
- 牌背统一显示为简单样式

### 4.2 Recommended Implementation

创建一个 `CardView` 组件，通过 `paintComponent(Graphics g)` 绘制：

- 红 / 黄 / 绿 / 蓝 / 黑底牌
- 中央绘制 `0-9`、`Skip`、`Reverse`、`+2`、`Wild`、`W+4`
- 后期如果时间充足，再替换为图片资源

### 4.3 Why Not Use Images First?

因为图片资源会带来以下额外成本：

- 文件路径管理
- 尺寸缩放和适配
- 透明背景处理
- 打包和加载问题

这些问题不会提升核心分数，只会拖慢关键逻辑开发。

---

## 5. High-Level Architecture

系统采用 **Client-Server Architecture**。

### 5.1 Client Responsibilities

客户端负责：

- 登录或输入用户名
- 创建 / 加入房间
- 显示手牌和游戏状态
- 发起出牌 / 抽牌请求
- 接收服务端推送的最新状态

### 5.2 Server Responsibilities

服务端负责：

- 管理客户端连接
- 管理房间和玩家
- 发牌与维护牌堆
- 校验出牌合法性
- 应用动作牌与自定义卡效果
- 控制回合推进
- 驱动 bot 行为
- 广播最新状态
- 在对局结束后保存历史数据

### 5.3 Persistence Responsibilities

数据库仅负责：

- 玩家资料
- 已结束比赛记录
- 胜场和总场次等统计信息

数据库不参与实时回合逻辑。

---

## 6. Module Breakdown

推荐模块划分如下：

```text
src/main/java/
  common/
  client/
  server/
  game/
  persistence/
```

### 6.1 `common/`

共享对象，客户端和服务端都需要使用。

建议包含：

- `Message`
- `MessageType`
- `GameSnapshot`
- `PlayerInfo`
- `CardDTO` 或直接共享 `Card`

职责：

- 定义网络消息结构
- 定义用于广播的快照对象
- 统一双方的数据协议

### 6.2 `client/`

客户端相关代码。

建议包含：

- `ClientMain`
- `ClientConnection`
- `LobbyFrame`
- `GameFrame`
- `CardView`
- `HandPanel`

职责：

- 建立与服务端的 socket 连接
- 展示房间和游戏 UI
- 将用户点击转成消息发送给服务端
- 根据服务端返回的状态刷新界面

### 6.3 `server/`

服务端入口、连接管理和房间管理。

建议包含：

- `ServerMain`
- `ClientHandler`
- `RoomManager`
- `GameRoom`
- `BotPlayer`
- `SimpleBotStrategy`

职责：

- 接收连接
- 管理房间生命周期
- 调用规则引擎处理操作
- 对 bot 触发自动动作
- 广播新状态

### 6.4 `game/`

纯游戏模型与规则层。

建议包含：

- `Card`
- `CardColor`
- `CardType`
- `Deck`
- `PlayerState`
- `GameState`
- `RuleEngine`

职责：

- 描述牌和玩家状态
- 管理抽牌堆 / 弃牌堆
- 判断牌是否合法
- 应用出牌效果
- 切换回合
- 判定胜负

### 6.5 `persistence/`

数据库相关。

建议包含：

- `DatabaseManager`
- `PlayerRepository`
- `MatchRepository`

职责：

- 初始化表
- 保存玩家资料
- 记录比赛结果
- 查询胜率和历史记录

---

## 7. Core Domain Model

### 7.1 Card

建议字段：

- `CardColor color`
- `CardType type`
- `int number` 仅数字牌需要

其中：

- `CardColor`: RED, YELLOW, GREEN, BLUE, BLACK
- `CardType`: NUMBER, SKIP, REVERSE, DRAW_TWO, WILD, WILD_DRAW_FOUR, GROUP_DRAW, TRIPLE_PEEK

### 7.2 PlayerState

建议字段：

- `String playerId`
- `String username`
- `List<Card> hand`
- `boolean isBot`

### 7.3 GameState

建议字段：

- `List<PlayerState> players`
- `Deque<Card> drawPile`
- `Deque<Card> discardPile`
- `int currentPlayerIndex`
- `int direction`
- `CardColor currentColor`
- `boolean started`
- `boolean finished`
- `String winnerId`
- `int pendingDrawCount`

### 7.4 GameSnapshot

广播给客户端的只读快照对象，应该只包含客户端需要的信息，例如：

- 当前弃牌堆顶部
- 当前颜色
- 当前轮到谁
- 自己的手牌
- 其他玩家的手牌数量
- 房间状态

对于 Triple Peek 等私有信息，不能通过普通广播暴露。

---

## 8. Message Protocol Design

建议采用可序列化消息对象。

### 8.1 Message Structure

```java
class Message implements Serializable {
    private MessageType type;
    private String playerId;
    private Object payload;
}
```

### 8.2 Client -> Server Messages

- `CONNECT`
- `CREATE_ROOM`
- `JOIN_ROOM`
- `START_GAME`
- `PLAY_CARD`
- `DRAW_CARD`
- `LEAVE_ROOM`
- `CHOOSE_COLOR`
- `TRIPLE_PEEK_CHOICE`

### 8.3 Server -> Client Messages

- `ROOM_UPDATE`
- `GAME_STATE`
- `TURN_UPDATE`
- `INVALID_MOVE`
- `GAME_OVER`
- `ERROR`
- `TRIPLE_PEEK_OPTIONS`
- `PRIVATE_UPDATE`

### 8.4 Protocol Rule

客户端只能发送请求，最终状态必须由服务端返回并覆盖本地显示。

---

## 9. Rule Engine Design

`RuleEngine` 是整个项目最核心的模块。

建议至少提供以下能力：

- `boolean canPlay(Card card, GameState state, PlayerState player)`
- `void playCard(GameState state, String playerId, Card card)`
- `Card drawCard(GameState state, String playerId)`
- `void advanceTurn(GameState state)`
- `void applyCardEffect(GameState state, Card card)`
- `boolean hasWinner(GameState state)`

### 9.1 Standard Rule Handling

必须支持：

- 颜色匹配
- 数字匹配
- 类型匹配
- 没牌可出时抽一张
- 抽后可立即出牌或保留
- 当前玩家之外的人不能行动

### 9.2 Action Cards

#### Skip

- 跳过下一名玩家

#### Reverse

- 多人局改变方向
- 两人局中相当于当前玩家再来一回合

#### Draw Two

- 下一位玩家抽两张
- 只允许与 Draw Two 叠加

#### Wild

- 玩家选择新的当前颜色

#### Wild Draw Four

- 玩家选择颜色
- 下一位玩家抽四张
- 只允许与 Wild Draw Four 叠加

### 9.3 Custom Cards

#### Group Draw

触发条件：

- 当前玩家没有合法可出的普通牌
- 原本即将进入抽牌流程

效果：

- 其他所有玩家各抽一张
- 当前玩家抽一张
- 按正常 draw rule 继续结算

#### Triple Peek

触发条件：

- 当前玩家没有合法可出的普通牌
- 原本即将进入抽牌流程

效果：

- 当前玩家私下查看牌堆顶部三张
- 选择其中一张加入手牌
- 若该牌合法可立即打出
- 其余牌放回牌堆底部
- 如果牌堆不足三张，则先 reshuffle

### 9.4 Private Information Requirement

Triple Peek 结果只能发给当前玩家本人，不能广播给所有客户端。

---

## 10. Bot Design

### 10.1 Scope

bot 只实现最简单逻辑：

- 不做最优策略
- 不预测对手
- 不做概率计算
- 不使用 AI / ML

### 10.2 Decision Rule

算法如下：

1. 按当前手牌顺序遍历
2. 找到第一张合法可出的牌
3. 立即打出
4. 如果没有，则抽一张
5. 如果新抽到的牌可出，则立即打出
6. 否则结束回合

### 10.3 Why Server-side?

bot 必须放在服务端，因为：

- 服务端掌握完整状态
- 回合推进由服务端负责
- 规则判定集中统一
- 避免额外 bot 客户端带来的同步复杂度

### 10.4 Suggested Classes

- `BotPlayer`
- `SimpleBotStrategy`

可以设计成：

```java
Card chooseFirstPlayableCard(List<Card> hand, GameState state)
```

---

## 11. Database Design

数据库只负责比赛后数据落库，不参与实时对局。

### 11.1 Suggested Tables

#### `players`

- `id`
- `username`
- `games_played`
- `wins`

#### `matches`

- `id`
- `winner_name`
- `played_at`
- `player_count`

#### `match_players`

- `match_id`
- `username`
- `result`

### 11.2 Repository Methods

建议提供：

- `createPlayerIfNotExists(String username)`
- `recordMatchResult(...)`
- `getPlayerStats(String username)`
- `getRecentMatches(String username)`

### 11.3 When to Persist

- 玩家登录或注册时
- 每局结束时

不要在每次出牌后写数据库。

---

## 12. Development Phases

以下分阶段路线决定了项目是否能稳定落地。

### Phase 1: Minimum Playable Loop

目标：

- 两个 client 能连接到同一个 server
- 能创建房间
- 能加入房间
- 能开始游戏
- 能显示自己的手牌和弃牌堆顶部

需要完成的内容：

#### GUI

- `LobbyFrame`
- `GameFrame`
- `CardView`

#### Networking

- `ServerSocket`
- `Socket`
- `ClientHandler`
- `ClientConnection`

#### Models

- `Card`
- `Deck`
- `GameState`
- `GameSnapshot`

#### Messages

- `CONNECT`
- `CREATE_ROOM`
- `JOIN_ROOM`
- `START_GAME`
- `ROOM_UPDATE`
- `GAME_STATE`

Done Definition：

- 可以启动 server
- 可以开两个客户端
- 可以进入同一房间
- 可以开始游戏
- 双方都能正确看到手牌和弃牌堆顶部

### Phase 2: Core Turn Logic

目标：

- 在没有复杂动作牌的情况下完成一整局基本 UNO

需要完成的内容：

#### Rule Engine

- 合法出牌判断
- 当前玩家限制
- 抽牌逻辑
- 回合推进
- 胜负判定

#### Messages

- `PLAY_CARD`
- `DRAW_CARD`
- `INVALID_MOVE`
- `TURN_UPDATE`
- `GAME_OVER`

#### Concurrency

- 房间级别同步控制
- 非当前玩家操作必须被拒绝

Done Definition：

- 玩家可以正常轮流出牌
- 抽牌和回合切换正确
- 某位玩家手牌为 0 时游戏结束

### Phase 3: Standard Action Cards

目标：

- 完整接入 Skip / Reverse / Draw Two / Wild / Wild Draw Four

需要完成的内容：

#### GameState extension

- `direction`
- `currentColor`
- `pendingDrawCount`

#### UI

- Wild / Wild Draw Four 出牌时弹出颜色选择

#### Rules

- Reverse 在 2 人局特殊处理
- Draw Two 只允许 Draw Two stacking
- Wild Draw Four 只允许自身 stacking
- Mixed stacking 不允许

Done Definition：

- 所有标准动作牌能按照规格正确结算

### Phase 4: Simple Bot

目标：

- 支持 human vs bot 模式

需要完成的内容：

#### Server

- `BotPlayer`
- `SimpleBotStrategy`
- 在轮到 bot 时自动触发决策

#### Rule Reuse

- bot 必须复用同一套 `RuleEngine`
- 不能单独写一套作弊逻辑

Done Definition：

- 玩家可以和 bot 开局
- bot 会自动行动
- bot 的出牌合法且符合简单规则

### Phase 5: Custom Cards

目标：

- 支持 Group Draw 和 Triple Peek

需要完成的内容：

#### Rule Layer

- 条件型触发逻辑
- 牌堆不足时 reshuffle
- 私有消息处理

#### UI

- Triple Peek 的选择窗口

#### Messages

- `TRIPLE_PEEK_OPTIONS`
- `TRIPLE_PEEK_CHOICE`
- `PRIVATE_UPDATE`

Done Definition：

- 两张自定义卡都能在正确条件下触发
- Triple Peek 内容只对当前玩家可见

### Phase 6: Persistence

目标：

- 保存玩家信息、比赛历史和胜率

需要完成的内容：

#### Database

- SQLite schema
- JDBC connection
- repositories

#### UI optional

- 在 Lobby 或统计页显示玩家历史数据

Done Definition：

- 一局结束后数据成功落库
- 重启程序后统计数据仍然可查询

---

## 13. Suggested Class Skeleton

```text
common/
  Message.java
  MessageType.java
  GameSnapshot.java
  PlayerInfo.java

client/
  ClientMain.java
  ClientConnection.java
  ui/
    LobbyFrame.java
    GameFrame.java
    CardView.java
    HandPanel.java

server/
  ServerMain.java
  ClientHandler.java
  RoomManager.java
  GameRoom.java
  bot/
    BotPlayer.java
    SimpleBotStrategy.java

game/
  model/
    Card.java
    CardColor.java
    CardType.java
    Deck.java
    PlayerState.java
    GameState.java
  logic/
    RuleEngine.java

persistence/
  DatabaseManager.java
  PlayerRepository.java
  MatchRepository.java
```

---

## 14. Team Split Suggestion

如果项目由两人完成，建议分工如下。

### Member A

- Swing GUI
- Lobby / Game 界面
- Client networking
- Card rendering
- Wild color selection UI
- Triple Peek UI

### Member B

- Server networking
- Room management
- Rule engine
- Turn control
- Bot module
- Database layer

### Shared Work

- Message protocol
- Game model definitions
- Integration testing
- Rule verification

注意：不要把项目粗暴分成“前端”和“后端”然后完全隔离，因为 socket 协议和快照结构必须双方共同设计。

---

## 15. Main Risks

### 15.1 Logic Split Across Client and Server

如果客户端也写规则判断，就会出现状态不一致。规则必须集中在服务端。

### 15.2 Bot Bypassing RuleEngine

如果 bot 直接修改状态，而不是复用和 human 相同的规则流程，后续会出现两套行为模型。

### 15.3 Triple Peek Leaks Private Information

Triple Peek 必须通过私有消息返回给当前玩家，不能通过普通状态广播暴露。

### 15.4 Doing Database Too Early

如果还没做出可玩回合就先接数据库，会极大拖慢项目节奏。

### 15.5 Overinvesting in Assets

过早投入牌面图片、动画和皮肤，会浪费时间且不会提高核心分数。

---

## 16. Final Recommendation

本项目最稳的落地路线是：

- **Java 17 + Maven**
- **Swing** 做 GUI
- **Socket / ServerSocket** 做实时通信
- **ExecutorService + synchronized** 控制服务端并发
- **RuleEngine** 集中处理所有游戏规则
- **Bot 放在 server 端**
- **JDBC + SQLite** 只负责历史和统计
- **前期使用 Java 自绘牌面，不依赖素材**

如果严格按照以上模块和 phase 推进，项目会更容易达到一个可以解释、可以演示、可以稳定运行的完成状态，而不是陷入功能很多但每一块都不稳的局面。
