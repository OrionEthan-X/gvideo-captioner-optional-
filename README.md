# 🎬 Video Captioner — AI-Powered Video Captioning with Film Analysis Methodology

<p align="center">
  <img src="https://img.shields.io/badge/Claude%20Code-Skill-blue?logo=claude" alt="Claude Code Skill">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License: MIT">
  <img src="https://img.shields.io/badge/methodology-RSLST%20Five--Layer-orange" alt="RSLST Framework">
  <img src="https://img.shields.io/badge/output-Sora%20|%20Kling%20|%20ComfyUI-purple" alt="Multi-model output">
</p>

<p align="center">
  <b>English</b> | <a href="#中文说明">中文说明</a>
</p>

> A Claude Code Skill that applies professional film analysis methodology to generate precise, structured video captions and AI video generation prompts. Powered by the RSLST Five-Layer Analysis Model.

---

## ✨ Why Video Captioner?

Most video captioning tools describe what they see. **Video Captioner thinks like a cinematographer.**

Instead of flat, surface-level descriptions, it decomposes every frame through five professional lenses — composition, space, light, subject, and motion — producing captions with technical precision and artistic sensibility.

### What makes it different

| Typical Approach | Video Captioner |
|-----------------|-----------------|
| "A woman by a window" | "MCU, low angle ~15°, 35mm lens, f/2.8. East Asian woman, late 20s, linen blouse, melancholic gaze. Soft 5600K key from window-left, 3200K warm rim back-right. Teal & orange grade..." |
| Flat, single-layer | Five-layer structured decomposition |
| One model, one format | Sora + Kling + ComfyUI, three prompt styles |
| No consistency check | Cross-frame subject/lighting/space validation |

---

## 🎯 The RSLST Five-Layer Model

| Layer | Dimension | What It Analyzes |
|-------|-----------|-----------------|
| **L1** | Shot & Composition | Shot type, camera angle, focal length, composition rules, camera movement |
| **L2** | Space & Environment | Location, depth layering (FG/MG/BG), materials, era/style |
| **L3** | Light & Color | Key/fill/rim light direction & temp, tonal range, color scheme, saturation |
| **L4** | Subject & Character | Appearance, facial features, hair, clothing, expression, gesture, posture |
| **L5** | Motion & Temporality | Subject motion, secondary motion, background motion, motion blur, time processing |

Each layer produces both **technical keywords** (for structured data) and **natural language description** (for prompts).

---

## 🚀 Installation

### Prerequisites

- **Claude Code** installed
- **FFmpeg** (optional — for automatic frame extraction from video files; not needed for image input)
- **OpenCV** (optional — Python fallback for frame extraction when FFmpeg is unavailable)

### Install the Skill

```bash
# Clone to your project's .claude/skills/ directory
git clone https://github.com/YOUR_USERNAME/video-captioner.git
cp -r video-captioner /your-project/.claude/skills/
```

Or as a standalone Claude Code project:

```bash
git clone https://github.com/YOUR_USERNAME/video-captioner.git
cd video-captioner
# Start Claude Code — the skill auto-loads
claude
```

### Manual Installation

1. Copy the entire `video-captioner/` folder to your project
2. Copy `tạo-captioner.md` to your project's `.claude/skills/` directory
3. Restart Claude Code

---

## 📖 Quick Start

```
/tạo-captioner input.mp4 --granularity standard --mode batch
```

### Input Formats

| Input | Description |
|-------|-------------|
| Video file (`.mp4`, `.mov`, `.avi`) | Auto scene detection + keyframe extraction via FFmpeg |
| Image directory | Process pre-extracted keyframes sorted by filename |
| Single image | Quick single-frame analysis |

### Analysis Granularity

| Level | Per-layer output | Total | Best for |
|-------|-----------------|-------|----------|
| `--quick` | 20-40 words | ~200 words | Batch rough labeling |
| `--standard` | 60-100 words | ~500 words | General annotation (default) |
| `--deep` | 150-250 words | ~1200 words | Training data refinement |
| `--director` | 300+ words | ~2000 words | High-quality generation source |

### Run Modes

| Mode | Behavior |
|------|----------|
| `--batch` | Fully automatic, process all frames |
| `--interactive` | Frame-by-frame with user confirmation |
| `--pick` | Show thumbnails, user selects which frames to analyze |

---

## 📤 Output Formats

Every run generates a complete output suite in `output/`:

```
output/
├── captions.json          # Full structured five-layer data
├── captions.csv           # Table format for annotation platforms
├── prompts_sora.txt       # Sora / Veo style (English cinematic narrative)
├── prompts_kling.txt      # Kling / Jimeng style (Chinese natural language)
├── prompts_comfyui.txt    # ComfyUI + AnimateDiff style (keywords + weights)
└── summary.md             # Complete film analysis report
```

### Prompt Style Comparison

**Sora / Veo (English cinematic):**
```
Cinematic medium close-up shot, low angle. East Asian woman in her
late 20s with tied-back dark hair, wearing a light beige linen blouse,
seated by a rain-streaked window during blue hour. Soft cool daylight
from left illuminates her face, warm rim light from back right...
```

**Kling / Jimeng (Chinese natural):**
```
电影级中近景镜头，低角度微微仰拍。一位东亚女性，二十多岁，
束起深色长发，身穿浅米色亚麻衬衫，独自坐在简约咖啡厅的窗边。
她凝视窗外，眉头轻蹙，神情忧郁而若有所思...
```

**ComfyUI / AnimateDiff (Keywords + weights):**
```
masterpiece, best quality, photorealistic, (cinematic medium close-up:1.2),
low angle, (East Asian woman, late 20s:1.1), tied-back dark hair,
light beige linen blouse, sitting by window, (rain-streaked window:1.1),
melancholic expression, contemplative gaze, shallow depth of field...
```

---

## 📁 Project Structure

```
video-captioner/
├── CLAUDE.md                    # Skill main file (core instructions)
├── README.md                    # This file
├── LICENSE                      # MIT License
├── .gitignore
├── prompts/
│   ├── layer1-shot.md           # L1 — Shot & Composition analysis guide
│   ├── layer2-space.md          # L2 — Space & Environment analysis guide
│   ├── layer3-light.md          # L3 — Light & Color analysis guide
│   ├── layer4-subject.md        # L4 — Subject & Character analysis guide
│   ├── layer5-motion.md         # L5 — Motion & Temporality analysis guide
│   └── synthesizer.md           # Synthesis & prompt engineering guide
└── examples/
    ├── example-01-cafe/         # Example: Café window scene (static, moody)
    └── example-02-action/       # Example: Outdoor sports (dynamic, golden hour)
```

---

## 🔧 Advanced Usage

```bash
# Deep analysis with interactive mode for Kling + Sora
/tạo-captioner scene.mp4 -g deep -m interactive --target-models sora,kling

# Director-level single frame analysis, bilingual output
/tạo-captioner poster.png -g director --language bilingual

# Batch processing with cross-frame consistency check
/tạo-captioner film.mp4 -g deep -m batch --consistency-check

# Custom sample interval (every 2 seconds)
/tạo-captioner video.mp4 --sample-interval 2 -g standard

# Quick overview of a pre-extracted keyframe directory
/tạo-captioner ./keyframes/ -g quick -m batch -o ./captions
```

---

## 🎓 The Methodology Behind It

This skill is built on the **RSLST (Réalisation / Space / Light / Subject / Time)** framework — a structured decomposition method derived from professional film school training (known as *拉片* in Chinese cinema education).

Unlike "describe what you see," RSLST forces the analyst to examine every frame through five independent dimensions, then synthesize them. This ensures:

- **No visual information is missed** — each layer has its own checklist
- **Cross-layer contradictions are caught** — e.g., L3's "overcast light" can't coexist with L2's "harsh shadows"
- **The output is both precise and evocative** — technical metrics + natural language
- **Ready for AI video generation** — prompts optimized for specific model preferences

---

## 🤝 Use Cases

- 🏷️ **Video dataset captioning** — structured JSON/CSV output ready for ML pipelines
- 🎥 **AI video model training data** — Sora, Kling, Veo, AnimateDiff prompt datasets
- 🎬 **Film analysis education** — learn cinematography through structured decomposition
- 📱 **Short video content breakdown** — reverse-engineer viral content shot by shot
- 🎨 **Prompt engineering research** — study what makes a good video generation prompt

---

## ⚙️ Dependencies

| Tool | Purpose | Required? |
|------|---------|-----------|
| **Claude Code** | Runtime environment | ✓ Required |
| **FFmpeg** | Video frame extraction & scene detection | For video input |
| **OpenCV-Python** | Fallback frame extraction | Optional (if FFmpeg unavailable) |
| **Claude Vision** | Semantic image understanding | ✓ For full L4 (subject) analysis |

> **Note:** L4 (subject layer) reaches full capability with Claude's vision models. When running on non-vision backends, the skill falls back to quantitative analysis (face detection, color histograms, edge maps, etc.) and flags what needs vision-based enrichment.

---

## 🤗 Contributing

Contributions are welcome! Areas that could use help:

- [ ] Additional prompt style templates (Pika, Runway, Moonvalley, etc.)
- [ ] More example pairs with real video frames
- [ ] Automated testing pipeline for output quality
- [ ] Video generation model compatibility matrix
- [ ] Non-human subject analysis guide (animals, objects, CG characters)

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- The film analysis methodology is inspired by professional cinematography education
- Prompt engineering patterns refined through extensive testing with video generation models
- Built for the Claude Code ecosystem

---

<p align="center">
  <b>Made with 🎬 by <a href="https://github.com/OrionEthan">OrionEthan</a> | 2026</b>
</p>

---

<a name="中文说明"></a>
## 🇨🇳 中文说明

Video Captioner（视频标注助手）是一个 Claude Code Skill，以专业**影视拉片方法论**为核心，对视频画面中的构图、空间、光影、主体、动态五大维度进行分层精准文字描述，产出结构化 Caption 和适配 Sora / 可灵 / ComfyUI 的 AI 视频生成提示词。

### 核心方法论：RSLST 五层拉片模型

| 层级 | 维度 | 分析内容 |
|------|------|----------|
| L1 构图层 | Shot & Composition | 景别、角度、焦段、构图法则、摄影机运动 |
| L2 空间层 | Space & Environment | 场景、纵深分层(前/中/后景)、材质、年代风格 |
| L3 光影层 | Light & Color | 主/辅/轮廓光、色温、影调、色彩方案 |
| L4 主体层 | Subject & Character | 外貌、面部、发型、服饰、表情神态、肢体动作 |
| L5 动态层 | Motion & Temporality | 主体运动、次级运动、背景运动、运动模糊、时间处理 |

### 快速开始

```bash
git clone https://github.com/YOUR_USERNAME/video-captioner.git
cd video-captioner
claude  # 启动 Claude Code，Skill 自动加载
```

在 Claude Code 中直接使用：

```
/tạo-captioner my_video.mp4 --granularity standard --mode batch
```
