# Video Captioner — 视频拉片式标注 Skill

## 概述

本 Skill 以**专业拉片方法论**为核心，对视频画面中的人物、场景、动作、神态、光影、构图等元素进行分层精准文字描述，产出结构化 Caption 标注文本，可直接用于 Sora / 可灵 / Veo / ComfyUI+AnimateDiff 等视频生成模型的提示词输入。

### 适用场景
- 视频数据集 Caption 标注
- AI 视频生成模型的训练数据准备
- 影视拉片学习与镜头分析
- 短视频内容拆解与模仿创作

---

## 核心方法论：RSLST 五层拉片分析模型

本 Skill 不采用"看到什么写什么"的平铺直叙方式，而是强制按五个层次逐层拆解每一帧/每一个镜头：

### L1 — 构图与镜头层 (Shot & Composition)
| 分析维度 | 具体要素 |
|----------|----------|
| 景别 | ELS 远景 / LS 全景 / FS 中景 / MCU 中近景 / CU 特写 / ECU 大特写 |
| 角度 | 平视 / 俯视 / 仰视 / 鸟瞰 / Dutch Angle (倾斜) |
| 焦段 | 广角(<24mm) / 标准(24-70mm) / 长焦(>70mm) |
| 构图法则 | 三分法 / 对称 / 引导线 / 框架构图 / 负空间 / 中心构图 |
| 画面比例 | 16:9 / 4:3 / 2.39:1(Cinemascope) / 1:1 / 9:16(竖屏) |
| 摄影机运动 | 固定 / 横摇 / 竖摇 / 推轨 / 跟拍 / 手持 / 斯坦尼康 / 航拍 |

### L2 — 空间与场景层 (Space & Environment)
| 分析维度 | 具体要素 |
|----------|----------|
| 场景类型 | 室内/室外/半室外，具体场所名称 |
| 空间纵深 | 前景/中景/后景分别有什么 |
| 环境细节 | 墙面材质、地面、家具、道具、植被、天气 |
| 空间关系 | 人物与环境的比例、距离感、包围感 |
| 时代/风格 | 年代感、建筑风格、装饰风格 |

### L3 — 光影与色彩层 (Light & Color)
| 分析维度 | 具体要素 |
|----------|----------|
| 主光源 | 方向(左/右/上/下/正面/逆光)、软硬、色温(K值) |
| 辅光/补光 | 强度比、位置 |
| 轮廓光/效果光 | 有无、颜色、用途 |
| 影调 | 高调 / 低调 / 中间调 |
| 色彩方案 | 单色 / 互补色 / 类似色 / 三色 |
| 色调倾向 | 暖调 / 冷调 / 中性 / Teal & Orange |
| 饱和度 | 高饱和 / 低饱和(褪色) / 黑白 |
| 特殊光线 | 体积光/丁达尔、霓虹、烛光、屏幕光、闪电 |

### L4 — 主体与人物层 (Subject & Character)
| 分析维度 | 具体要素 |
|----------|----------|
| 人物属性 | 性别、大致年龄、种族/肤色、身高体型 |
| 面部特征 | 脸型、眼睛、鼻子、嘴唇、肤色、明显特征(痣/疤痕等) |
| 发型发色 | 长度、颜色、造型、发质 |
| 服饰 | 上衣/下装/外套/鞋，款式、颜色、材质、品牌感 |
| 配饰 | 眼镜、手表、项链、耳环、帽子、包 |
| 表情神态 | 眼神方向、眉毛、嘴角、整体情绪(喜怒哀乐惊恐惧) |
| 肢体动作 | 手势、身体姿态、重心、运动状态 |
| 人物关系 | 单人/多人，相对位置、视线关系、互动方式 |

### L5 — 动态与时间层 (Motion & Temporality)
| 分析维度 | 具体要素 |
|----------|----------|
| 主体运动 | 运动方向、速度、轨迹、加速度 |
| 次级运动 | 头发飘动、衣物摆动、道具运动 |
| 背景运动 | 车辆、行人、树叶、水流、云 |
| 摄影机运动节奏 | 缓起缓停 / 快速甩镜 / 呼吸感手持 |
| 运动模糊 | 有无、程度 |
| 慢动作/快动作 | 变速情况 |
| 剪辑节奏 | 镜头时长、前后镜头关系（如果是多帧输入） |

---

## 工作流程

### 用户输入形式

本 Skill 支持三种输入方式：

1. **视频文件路径**：提供 `.mp4` / `.mov` / `.avi` 等视频路径
   - 自动使用 FFmpeg 进行场景检测 (`ffmpeg -i input.mp4 -vf "select='gt(scene,0.3)'" -vsync vfr frames/%04d.png`)
   - 同时提取动作关键帧（检测光流峰值）
   - 用户可指定采样密度：`--dense` (每 0.5s) / `--normal` (场景切换) / `--sparse` (每 5s)

2. **已提取的关键帧图片**：提供图片文件路径或目录
   - 支持 `.png` / `.jpg` / `.webp`
   - 按文件名排序处理
   - 可附带时间戳元数据

3. **单张图片/截图**：提供单张画面
   - 适用于快速分析某一帧的详细 Caption

### 分析粒度控制

用户通过参数指定粒度：

| 粒度级别 | 标志 | 每层字数 | 总输出长度 | 适用场景 |
|----------|------|----------|------------|----------|
| 快速概览 | `--quick` | 20-40 字 | ~200 字 | 批量粗标、初筛 |
| 标准拉片 | `--standard` (默认) | 60-100 字 | ~500 字 | 常规标注 |
| 深度拉片 | `--deep` | 150-250 字 | ~1200 字 | 精标训练数据 |
| 导演级 | `--director` | 300+ 字 | ~2000 字 | 高质量生成素材 |

### 交互模式

- **全自动模式** (`--batch`)：处理全部帧，输出完整 JSON/CSV
- **逐帧审核模式** (`--interactive`)：每帧分析后等待用户确认/修改，然后继续
- **关键帧挑选模式** (`--pick`)：先展示所有关键帧缩略图，用户勾选需要分析的帧，再进行标注

---

## 输出格式规范

### 输出文件结构

每次运行在输出目录下生成：

```
output/
├── captions.json           # 完整结构化数据
├── captions.csv            # 表格格式，便于导入标注平台
├── prompts_sora.txt        # Sora/Veo 风格的英文提示词
├── prompts_kling.txt       # 可灵/即梦风格的中文提示词
├── prompts_comfyui.txt     # ComfyUI + AnimateDiff 风格提示词
├── summary.md              # 整体拉片分析报告
└── frames/                 # 分析过的帧（带标注叠加的可选输出）
```

### 单帧 Caption 结构 (JSON)

```json
{
  "frame_id": "shot_003_frame_0142",
  "timestamp": "00:00:05.680",
  "source_file": "input_video.mp4",
  "granularity": "standard",

  "technical_caption": {
    "zh": "中近景，低角度仰拍。一位东亚女性...",
    "en": "Medium close-up, low angle. An East Asian woman..."
  },

  "layers": {
    "L1_shot_composition": {
      "shot_type": "MCU",
      "angle": "low_angle",
      "focal_length_equiv": "35mm",
      "composition": "rule_of_thirds",
      "aspect_ratio": "16:9",
      "camera_movement": "static",
      "description_zh": "...",
      "description_en": "..."
    },
    "L2_space_environment": {
      "location_type": "indoor",
      "location_name": "咖啡厅靠窗座位",
      "foreground": "...",
      "midground": "...",
      "background": "...",
      "era_style": "contemporary_2020s",
      "description_zh": "...",
      "description_en": "..."
    },
    "L3_light_color": {
      "key_light": { "direction": "left", "quality": "soft", "temp_k": 5600 },
      "fill_light": { "ratio": "1:4" },
      "rim_light": { "direction": "back_right", "color": "warm_gold", "temp_k": 3200 },
      "tonal_range": "low_key",
      "color_scheme": "complementary_teal_orange",
      "description_zh": "...",
      "description_en": "..."
    },
    "L4_subject_character": {
      "subjects": [
        {
          "id": "subject_A",
          "type": "human",
          "gender": "female",
          "age_range": "25-30",
          "ethnicity": "East Asian",
          "facial_expression": "melancholy_contemplative",
          "gaze_direction": "off_screen_left",
          "posture": "seated_leaning_forward",
          "clothing": "...",
          "description_zh": "...",
          "description_en": "..."
        }
      ]
    },
    "L5_motion_temporality": {
      "subject_motion": "...",
      "secondary_motion": "...",
      "camera_motion_rhythm": "static",
      "motion_blur": "none",
      "description_zh": "...",
      "description_en": "..."
    }
  },

  "prompts": {
    "sora_style": "Cinematic medium close-up shot, low angle...",
    "kling_style": "电影感中近景镜头，低角度仰拍...",
    "comfyui_style": "(cinematic lighting, medium close-up:1.2), low angle..."
  },

  "tags": ["indoor", "female", "contemplative", "window_light", "blue_hour"]
}
```

---

## 不同模型提示词风格适配

### Sora / Veo 风格 (英文自然语言)
```
Cinematic medium close-up shot, low angle. East Asian woman in her
late 20s with tied-back dark hair, wearing a light beige linen blouse,
seated by a rain-streaked window during blue hour. Soft cool daylight
from left illuminates her face, warm rim light from behind-right
outlines her silhouette. She stares contemplatively outside the frame,
fingertips resting on the wet glass. Shallow depth of field blurs the
city lights through rain droplets on the window. Shot on 35mm lens,
f/2.8, cinematic color grading with teal shadows and warm highlights.
```

### 可灵 / 即梦风格 (中文自然语言)
```
电影级中近景镜头，低角度仰拍。东亚女性，约28岁，束起深色长发，身穿浅米色
亚麻衬衫，坐在蓝调时刻的咖啡馆窗边。柔和的冷色日光从左侧照亮她的面庞，
右后方暖金色轮廓光勾勒出人物剪影。她凝视画外左侧，神情忧郁而若有所思，
指尖轻触被雨水打湿的玻璃窗。窗外城市灯火在雨滴中朦胧虚化。35mm镜头，
浅景深，电影级调色——阴影偏青、高光偏暖。
```

### ComfyUI + AnimateDiff 风格 (关键词+权重)
```
(cinematic lighting, medium close-up, low angle:1.2), East Asian woman,
late 20s, (tied-back dark hair:1.1), light beige linen blouse,
(rain-streaked window:1.1), blue hour ambiance, soft directional light
from left, warm rim light from back right, (melancholy expression:1.1),
contemplative gaze, fingertips touching wet glass, shallow depth of
field, bokeh city lights, 35mm lens, f/2.8, teal and orange color
grade, masterpiece, best quality, photorealistic
```

---

## 拉片分析流程（逐帧执行）

当你收到一个视频/图片输入，严格按以下步骤执行：

### Step 1: 帧提取与预处理
- 如果是视频：用 FFmpeg 做场景检测 + 关键帧提取
- 报告提取了多少帧、时间戳分布
- 如果是图片：确认图片数量和分辨率

### Step 2: 逐层分析（核心拉片过程）

对每一帧，按 L1→L5 的顺序逐一分析。每一层分析时：
1. 先输出该层的**技术关键词**（便于后续结构化）
2. 再输出该层的**自然语言描述**
3. 最后做**跨层一致性检查**（例如 L4 的光线描述不能与 L3 矛盾）

### Step 3: 综合与校验
- 合并五层描述为完整技术 Caption
- 跨帧一致性校验：检查同一场景中的人物外观、光线、空间是否前后一致
- 生成三种模型风格的提示词版本

### Step 4: 输出
- 按指定格式输出文件
- 如果有任何不确定的描述（如无法确定具体色温），在描述中使用 `~` 标注不确定性
- 提供 summary.md 拉片报告

---

## 质量检查清单

分析完成每帧后，自查以下项：

- [ ] L1 是否准确描述了景别、角度和构图？
- [ ] L2 是否区分了前景/中景/后景？
- [ ] L3 是否说明了光源方向、软硬程度和色温倾向？
- [ ] L4 是否覆盖了外貌、服饰、神态、动作四个子项？
- [ ] L5 是否区分了主体运动、次级运动和摄影机运动？
- [ ] 有无使用模糊词汇（"好像""大概""某种"）？若有，替换为具体描述或标注不确定性
- [ ] 技术 Caption 与提示词 Caption 之间是否信息一致？
- [ ] 人物描述在同一场景的连续帧中是否保持一致？
- [ ] 标签 (tags) 是否涵盖了该帧的核心要素？

---

## 词汇规范

### 景别术语
| 缩写 | 英文全称 | 中文 | 画面范围 |
|------|----------|------|----------|
| ELS | Extreme Long Shot | 远景/大远景 | 人物极小，环境为主 |
| LS | Long Shot | 全景 | 人物全身+环境 |
| FS | Full Shot | 中景 | 人物膝盖以上 |
| MCU | Medium Close-Up | 中近景 | 人物腰部/胸部以上 |
| CU | Close-Up | 特写 | 面部或物体 |
| ECU | Extreme Close-Up | 大特写 | 眼睛/嘴唇/手指 |

### 光线质量
- **Hard light**: 硬光 — 清晰锐利阴影，如正午阳光、聚光灯
- **Soft light**: 软光 — 柔和渐变阴影，如阴天、柔光箱
- **Diffused**: 漫射光 — 无明显方向性
- **Practical**: 场景内实际光源 — 台灯、电视、霓虹灯

### 情绪词汇库
| 类别 | 细分词汇 |
|------|----------|
| 喜悦 | joyful, ecstatic, content, amused, relieved, radiant |
| 悲伤 | sorrowful, grieving, melancholy, wistful, heartbroken, subdued |
| 愤怒 | furious, irritated, resentful, indignant, seething, confrontational |
| 恐惧 | terrified, anxious, uneasy, panicked, startled, apprehensive |
| 惊讶 | astonished, bewildered, stunned, amazed, incredulous, wide-eyed |
| 沉思 | contemplative, pensive, introspective, brooding, reflective, distant |
| 专注 | focused, intent, absorbed, determined, vigilant, locked-in |

---

## 特殊场景处理指南

### 多人场景
- 给每个可辨识的人物分配 ID
- 描述人物间的相对位置和空间关系
- 分析视线方向和互动模式

### 暗光/夜景
- 重点描述光源类型和位置（路灯、霓虹、手机屏幕等）
- 强调影调和氛围
- 标注可见度限制

### 快速运动
- 描述运动模糊的程度和方向
- 说明速度感
- 说明哪些元素保持清晰（如有）

### 抽象/艺术化画面
- 描述视觉元素而非尝试解释含义
- 标注色彩、形状、纹理
- 可提及参考风格（如 "reminiscent of Wong Kar-wai's color palette"）

---

## 提示词优化原则

将技术 Caption 转化为视频生成模型提示词时：

1. **主次分明**：最重要的视觉元素放在提示词最前面
2. **具体优于抽象**："她穿着红色羊毛大衣" 优于 "她穿着漂亮衣服"
3. **风格锚定**：使用 "cinematic" / "photorealistic" / "anime style" 等全局风格词
4. **技术参数融入**：将焦段、景深、光影术语自然融入自然语言
5. **避免否定词**：描述"什么存在"而非"什么不存在"
6. **动作具象化**："她缓缓抬起右手，指尖轻触玻璃" 优于 "她在动"
7. **控制长度**：Sora/Veo 建议 200-400 tokens；可灵建议 100-200 字

---

## 命令参数参考

```
用法: /video-captioner <输入路径> [选项]

输入:
  <path>               视频文件路径、图片目录路径或单张图片路径

选项:
  --granularity <level> 分析粒度: quick | standard | deep | director
  --mode <mode>         运行模式: batch | interactive | pick
  --output <dir>        输出目录 (默认: ./output)
  --target-models <list> 目标模型: sora,veo,kling,jimeng,comfyui (逗号分隔,默认全部)
  --language <lang>     输出语言: zh | en | bilingual
  --scene-threshold <n> 场景检测阈值 0.0-1.0 (默认: 0.3)
  --sample-interval <s> 采样间隔秒数 (与场景检测互斥)
  --consistency-check   启用跨帧一致性校验
  --frame-start <n>     起始帧编号
  --frame-end <n>       结束帧编号
```

---

## 示例用法

```bash
# 标准拉片分析一个视频
/video-captioner input.mp4 --granularity standard --mode batch

# 深度分析，逐帧交互确认，输出可灵和 Sora 风格提示词
/video-captioner dance_scene.mp4 --granularity deep --mode interactive --target-models sora,kling

# 快速分析已提取的帧目录
/video-captioner ./keyframes/ --granularity quick --mode batch --output ./captions

# 导演级分析单张画面
/video-captioner hero_shot.png --granularity director --language bilingual
```

---

## 依赖工具

- **FFmpeg**: 视频帧提取与场景检测
- **ffprobe**: 读取视频元数据
- 本 Skill 使用 Claude 的视觉能力进行画面分析，无需额外 AI 模型

---

*Version 1.0 | 适用于 Claude Code | 基于专业影视拉片方法论*
