[English project overview](README.md)

# DIKWP CareWeave Robot OS v1.0.0

Created by Yucong Duan (段玉聪).

面向老人自主陪护与儿童成长学习的、本地优先、机器人无关的产品级开源平台  
A local-first, robot-agnostic product platform for elder autonomy, companionship and child growth.

CareWeave不是某一台机器人的聊天应用。它把“人—家庭/机构—AI—机器人”之间的目的、同意、记忆、行动、审批、警报与责任记录做成一个可部署的控制平面，再通过适配器连接ROS 2、temi、Pepper、Unitree、Misty、Furhat或通用HTTP设备。

> 核心产品式：两种前台产品 + 一个可信内核 + 一个机器人适配市场
>
> - CareWeave Elder：老人自主陪护、日常连续、真实关系升级与机构运营。
> - CareWeave Grow：儿童苏格拉底学习、教回证据、项目成长与监护协同。
> - CareWeave Trust Kernel：Purpose合同、同意、记忆治理、动作门控、审计链与机器人回执。
> - CareWeave Adapter Protocol：硬件能力协商和高层动作信封，使产品不绑定单一机器人厂商。

## 1. 可运行能力

### Elder 老人自主陪护

- 日常问候、情绪陪伴和结构化检查；
- 饮水、活动、社交、休息和已确认计划内的用药提醒；
- 孤独、摔倒语言、胸痛/呼吸困难语言等信号升级为监护警报；
- 一键视频通话或远程呈现请求；
- 经目的合同、目标地点白名单、机器人能力和人工审批后的导航请求；
- 个人偏好、生活史和关系记忆的来源、同意、敏感度、有效期与撤销；
- 家庭版和养老机构版的监护控制台、警报队列、动作审批和审计导出。

### Grow 儿童成长学习

- 苏格拉底提问、最小提示、教回法和反思；
- 学习目标、作品/项目、测验、观察和教回证据；
- “不直接代做”的成长模式，而不是无限答案倾倒；
- 家长或教育者可见的学习证据，不把全量监控包装为成长；
- 禁止机器人要求儿童向监护人保密、制造排他依恋、定向广告或购买；
- 音视频、定位、记忆和远程呈现均受监护同意与保留期限控制；
- 紧急或自伤语言触发真实成年人升级，不声称机器人可以替代家长或急救服务。

### 共同可信内核

- 多租户账户与角色；
- 每个陪护对象一份可版本化Purpose/Care Contract；
- 动作风险分级：`allow / hold / block`；
- 人工审批后重新评估，而不是“点击批准就绕过硬边界”；
- 机器人一次性注册令牌、能力声明、心跳、命令租约和结果回执；
- PBKDF2密码哈希、JWT、租户隔离、安全响应头；
- 记忆同意、敏感度、来源、置信度、到期、批准与撤销；
- 哈希链审计及篡改检测；
- 数据导出和主体级硬删除；
- 可选OpenAI兼容LLM规划器，但所有动作仍必须通过确定性门控；
- 本地规则规划器在无云、无模型API时仍可工作。

## 2. 默认安全边界

CareWeave v1.0.0 不会：

- 作出医疗诊断、改变药物或剂量、自动配药；
- 抬举、搬运、约束人体；
- 解锁外门、转账、购买或管理资产；
- 对儿童进行秘密关系、排他依恋、广告画像或隐蔽录音；
- 声称提供无人托育、急救派单或医疗器械功能；
- 通过通用适配器输出轮速、关节角、扭矩、抓取或其他低级执行值；
- 把语言模型的私有思维链当作审计证据。

`stop`是特殊动作：始终允许，并应由机器人端独立实现，不依赖云端可用性。

## 3. 快速部署

### A. Python本地部署

```bash
python -m venv .venv
. .venv/bin/activate        # Windows: .venv\Scripts\activate
python -m pip install -e '.[dev]'
cp .env.example .env
# 修改 CAREWEAVE_SECRET 和管理员密码
python run.py
```

打开：

- 控制台：`http://127.0.0.1:8848/`
- OpenAPI：`http://127.0.0.1:8848/docs`
- 健康检查：`http://127.0.0.1:8848/api/v1/health`

### B. Docker Compose

```bash
cp .env.example .env
# 生成强随机 CAREWEAVE_SECRET，修改 CAREWEAVE_BOOTSTRAP_PASSWORD
# 将 docker-compose.yml 中端口保持绑定到 127.0.0.1，外网通过TLS反向代理访问
docker compose up --build -d
```

### C. Wheel安装

```bash
python -m pip install dikwp_careweave-1.0.0-py3-none-any.whl
careweave doctor
careweave serve --host 127.0.0.1 --port 8848
```

## 4. 第一次使用

1. 使用管理员账号登录；生产环境必须立即更改默认密码和密钥。
2. 新建`elder`或`grow`陪护对象；系统自动创建相应默认Purpose合同。
3. 记录记忆范围同意，例如`memory.personal`或`memory.child`。
4. 注册机器人。返回的机器人令牌只显示一次。
5. 先使用`mock`适配器验证工作流，再连接真实硬件。
6. 所有真实移动、远程呈现和敏感记忆必须在目标场景完成风险评估和人工批准。

## 5. 机器人接入

### 内置可直接调用的适配器

| 适配器 | 当前映射 | 默认物理移动 |
|---|---|---|
| `mock` | 说话、显示、姿态、媒体、视频、模拟导航、停止 | 仅模拟 |
| `generic_http` | 将批准的动作POST到机器人网关 | 由网关决定，CareWeave仍门控 |
| `misty` | TTS、文本显示、受限头部动作、Halt | 禁止 |
| `furhat` | Realtime WebSocket说话、表情、停止说话 | 不适用 |

### SDK/ROS侧车模板

| 平台 | 路径 | 交付状态 |
|---|---|---|
| ROS 2 Lyrical/Jazzy | `bridges/ros2/` | 高层批准意图发布器；不输出`cmd_vel`/关节值 |
| temi Android SDK | `bridges/temi_android/` | SDK 1.138.0模板；说话、白名单导航、停止 |
| Pepper QiSDK | `bridges/pepper_qisdk/` | 安全子集模板；说话与本地取消 |
| Unitree SDK2 / ROS 2 | `bridges/unitree_sdk2/` | 安全插件边界；默认禁止步态/关节/操作 |
| 任意机器人 | `bridges/generic_robot_agent.py` | Python轮询代理参考实现 |

每个生产适配器必须提供：

- 设备身份与令牌保护；
- 能力清单；
- 本地紧急停止；
- 超时与重试；
- 实际完成或失败回执；
- 网络中断行为；
- 物理安全状态检查；
- 目标场景测试和版本兼容表。

详见[`docs/ROBOT_ADAPTER_SPEC.md`](docs/ROBOT_ADAPTER_SPEC.md)。

## 6. API示例

Configure unique local credentials before using authenticated endpoints. Consult the local OpenAPI documentation for request schemas. Use only fictional participants and synthetic care records during development; keep real credentials and personal records out of public examples.

## 7. 架构

```text
家庭 / 养老机构 / 学校 / 儿童监护人
                  │
        Guardian & Operations Console
                  │
   ┌──────────────┼────────────────┐
   │ Purpose Contract & Consent     │
   │ Memory Governance              │
   │ Learning / Care Plan           │
   │ Alert & Human Approval         │
   │ Audit Chain & Export/Delete    │
   └──────────────┼────────────────┘
                  │
     Deterministic Action Safety Gate
          allow / hold / block
                  │
   Robot Command Lease + Safety Receipt
                  │
 ROS2 | temi | Pepper | Unitree | Misty | Furhat | HTTP
                  │
 Physical robot safety controller / local stop
```

详见[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)和[`docs/architecture.svg`](docs/architecture.svg)。

## 8. 盈利结构（开源核心，不是无偿定制）

Apache-2.0开源核心可以降低采购阻力并吸引机器人厂商适配，但持续盈利来自可验证的服务层：

1. 家庭订阅：监护应用、多人家庭、备份、远程协作和内容包；
2. 机构SaaS/私有部署：按床位、班级、机器人或活跃对象计费；
3. 机器人OEM授权与适配认证：联合品牌、预装、设备管理、兼容性测试；
4. 行业实施：场景设计、系统集成、数据迁移、值班与SLA；
5. 安全与证据包：审计导出、红队、版本兼容、合规材料和第三方复核；
6. 成长内容与照护流程市场：经审查的课程、活动、文化陪伴和机构流程分成；
7. 硬件捆绑：与成熟机器人产品合作，不优先承担重资产整机制造。

参考定价、单位经济和首批市场进入策略见[`docs/COMMERCIALIZATION.md`](docs/COMMERCIALIZATION.md)。文档中的价格是战略假设，不是已实现收入。

## 9. 测试

```bash
pytest
careweave doctor
```

v1.0.0发布前测试覆盖：认证、默认合同、儿童关系边界、老人医疗边界、记忆同意、导航审批、原始密钥阻断、Mock执行、机器人轮询、命令回执、紧急升级、学习对话、例行任务、主体导出和审计篡改检测。

## 10. 生产上线前清单

- [ ] 更换密钥、默认密码和域名；
- [ ] 配置TLS、备份加密、设备网络和日志监控；
- [ ] 明确数据控制者、处理者、保留期和儿童监护同意；
- [ ] 对机器人场景执行碰撞、跌落、夹伤、速度、误导航和网络中断测试；
- [ ] 验证本地急停不依赖CareWeave服务器；
- [ ] 审计每个模型、内容源和第三方服务的数据路径；
- [ ] 对医疗、教育、养老和未成年人场景完成当地法律与伦理审查；
- [ ] 用真实家庭/机构指标验证价值，不把对话时长当作健康或学习成效；
- [ ] 签署付费发现、SOW、验收、知识产权和支持边界；
- [ ] 在任何“认证”“医疗”“儿童安全”宣传前取得独立证据。

## 11. 状态与非主张

v1.0.0是可运行的产品化参考实现和付费试点底座。核心软件测试已在本交付环境完成；Misty/Furhat网络适配器以及ROS 2、temi、Pepper、Unitree模板需要目标硬件和相应SDK环境完成实机验证。本项目不保证市场主导地位，但通过硬件无关架构、双产品共用内核、监护闭环和证据化安全，设计为具备形成类别领导力的基础。

## 12. License

Apache License 2.0。商标、官方兼容性、认证和背书规则另见[`TRADEMARKS.md`](TRADEMARKS.md)。

---

# English overview

DIKWP CareWeave Robot OS is a local-first, robot-agnostic platform that turns elder companionship and child growth into governed, inspectable services rather than an unbounded chatbot. The same control plane manages purpose contracts, consent, memory, learning/care plans, alerts, human approvals, robot capability negotiation, command receipts and a tamper-evident audit chain.

The open core is designed for household subscriptions, institution deployments, robot-OEM partnerships, compatibility testing, managed operations and curated content/process marketplaces. Physical safety remains a robot-side responsibility: generic CareWeave adapters issue approved high-level intents, never low-level motor values. The release is not a medical device, autonomous childcare service, emergency dispatcher or certified robot safety controller.
