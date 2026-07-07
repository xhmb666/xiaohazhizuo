# 技术架构文档 · 怀星公告网站

## 1. 架构设计

```mermaid
flowchart TD
    subgraph "前端层 Frontend"
        "React + Vite + TailwindCSS"
        "公告展示页"
        "管理面板"
    end
    subgraph "数据层 Data"
        "Firebase Firestore"
        "announcements 集合"
        "votes 子集合"
    end
    subgraph "本地状态 Local"
        "localStorage 投票记录"
        "sessionStorage 管理员会话"
    end
    "公告展示页" --> "Firebase Firestore"
    "管理面板" --> "Firebase Firestore"
    "Firebase Firestore" --> "公告展示页"
    "投票交互" --> "localStorage"
    "投票交互" --> "Firebase Firestore"
```

**部署架构**：纯静态构建（Vite build）→ GitHub Pages 部署，无后端服务器，数据通过 Firebase Firestore 实现跨设备同步。

## 2. 技术栈
- **前端**：React@18 + TypeScript + Vite + TailwindCSS@3
- **状态管理**：Zustand
- **图标**：lucide-react
- **数据库**：Firebase Firestore（实时同步，跨设备）
- **初始化工具**：vite-init (react-ts 模板)
- **后端**：无（Firebase 作为 BaaS）

## 3. 路由定义
| 路由 | 用途 |
|------|------|
| `/` | 公告展示页（默认） |
| `/` (管理员模式) | 同一页面，验证后显示管理面板抽屉 |

> 单页应用，管理员面板通过状态切换显示，不使用独立路由

## 4. 数据模型

### 4.1 Firestore 集合结构

**announcements 集合**（每条公告一个文档）
```typescript
interface Announcement {
  id: string;           // 文档 ID（自动生成）
  title: string;        // 公告标题
  content: string;      // 公告正文
  tag: 'normal' | 'urgent';  // 标签：普通/紧急
  createdAt: number;    // 创建时间戳（用于排序）
  upvotes: number;      // 赞成数
  downvotes: number;    // 反对数
}
```

### 4.2 本地存储
- `voted_announcements`：localStorage，记录已投票的公告 ID 及投票类型，防止重复投票

## 5. 关键实现说明

### 5.1 管理员验证
- 左上角「怀星」元素绑定 click 计数器
- 2 秒内连续点击达 5 次时弹出密码输入框
- 密码：`qwertyuioplkjhgfdsazxcvbnm`
- 验证通过后写入 sessionStorage，刷新页面内保持管理员态
- 密码硬编码在前端（这是用户明确要求的简单方案，非生产级安全）

### 5.2 Firebase 配置
- Firebase 配置信息放在 `src/firebase/config.ts`
- 用户需替换为自己的 Firebase 项目配置
- Firestore 规则设为：读公开，写需管理员（前端先做密码校验，Firestore 规则做二次防护）

### 5.3 实时订阅
- 使用 Firestore `onSnapshot` 订阅 announcements 集合
- 任何设备发布/删除/投票，所有设备实时更新

### 5.4 投票逻辑
- 普通用户点击赞成/反对
- 同一设备同一公告只能投一次，可切换赞成/反对
- 切换时先减旧票再加新票
- 投票记录存 localStorage

## 6. 部署说明
1. `npm run build` 构建静态文件到 `dist/`
2. 推送到 GitHub 仓库
3. 启用 GitHub Pages（source: main branch / docs）
4. 在仓库 Settings → Pages 配置
5. 用户需自行创建 Firebase 项目并替换 `src/firebase/config.ts` 中的配置
