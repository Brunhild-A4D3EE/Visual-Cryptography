# Visual Cryptography Playground

一个交互式视觉密码学演示 + 物理道具生成工具。纯前端、单 HTML 文件、无任何依赖，浏览器打开就能用。

> 视觉密码学（Visual Cryptography）是 Naor & Shamir 1994 年提出的一种秘密分享方案：把一张图拆成 N 张"分片"(share)，每张单独看是无意义的噪声，但任意 k 张物理叠加（如打印到透明胶片上重叠）就能用肉眼直接看出原图——**完全不需要任何计算或解密算法**。

## 🎨 三种模式

### Tab 1: `2-out-of-2`（基础版）

经典 Naor-Shamir `(2, 2)`-threshold 方案。

- 上传一张图（任意尺寸，自动二值化）
- 生成 **2 张** share，叠加显形原图
- 单张 share 是均匀随机噪声，信息论意义上的完美保密（等同 one-time pad）

**算法：** 每个原像素 → 2×2 子像素块。从 6 种 2-黑-2-白 模式中随机选一个：
- 原像素白 → 两张 share 用**同一**模式 → 叠加 = 2 黑 2 白（50% 灰）
- 原像素黑 → 两张 share 用**互补**模式 → 叠加 = 4 个全黑（100% 黑）

### Tab 2: `3-shares / 3-secrets`（可打印）

**Halftone steganographic VC.** 3 张 share 各自看起来是均匀网点噪声，但**任意两张叠加显示不同的秘密图**：

| 叠加组合 | 显示秘密 |
|---|---|
| Share 1 + Share 2 | 秘密 #1 |
| Share 1 + Share 3 | 秘密 #2 |
| Share 2 + Share 3 | 秘密 #3 |

每张 share 是独立的 50% 随机网点（TV static），三张密度处处恒等 50%，**单张完全无秘密泄漏**。

**算法：** 每个秘密像素 → `N×N` 子像素块。块内随机抽 `6K` 个位置分给 3 个 pair：
- `Z_12` (2K 子像素): 编码秘密 #1，Share 1+2 用 2-of-2 VC pattern，Share 3 保留随机噪声
- `Z_13` (2K 子像素): 编码秘密 #2
- `Z_23` (2K 子像素): 编码秘密 #3
- `Z_rest` (N²−6K): 全部随机噪声

约束: `6K ≤ N²`，每个 pair 的对比信号 = `K/N²`。

**默认参数 N=4, K=2**：12.5% 对比，约束刚好填满。

### Tab 3: `彩色 CMYK`（beta）

RGB 图先 Floyd-Steinberg / Bayer 抖动转 CMYK 四通道二值"网点"，对每个通道**独立**跑 2-out-of-2 VC，再合成彩色 share。

每个子像素是 8 种基色之一（白/青/品红/黄/红/绿/蓝/黑），物理叠加用减色原理还原原图。

> ⚠️ **数字叠加要用 Multiply 混合模式**——Photoshop / Figma 默认是 Normal compositing（上层覆盖下层）不对。把上层 share 改成 **Multiply (正片叠底)** 才能模拟油墨吸光叠加。物理打印到胶片上叠加则没这个问题。

## 🖨️ 物理打印工作流

适用于 Tab 2 和 Tab 3。

1. 调好参数 → 上传秘密图
2. 点 **"导出 3 张打印 PNG (300 DPI 透明)"** 或 **"导出 2 张打印 PNG (300 DPI 透明)"**
3. 浏览器一次性下载所有 share PNG（间隔 400ms）

   > ⚠️ **必须一次性导出，不能分次！** 算法每次跑都用新的随机种子，分次导出会得到互不相干的 share，叠加无法解密。

4. 用**激光打印机**打到**透明胶片 (OHP transparency film)**

   > 喷墨打印在透明片上会糊，激光好用很多

5. 打印时务必关闭"自适应页面"，选**实际尺寸 100%**
6. 可选：勾选"加四角对位标记 +"得到 + 字角标帮助物理对齐
7. 任意两张叠加 → 显形对应秘密

## 🚀 GitHub Pages 部署

仓库已经初始化好 git，提交并启用 Pages：

```bash
cd ~/Desktop/Visual-Cryptography
git config user.name "你的名字"
git config user.email "你的邮箱"
git add .
git commit -m "Initial commit: visual cryptography playground"
git push -u origin main
```

然后：
1. 访问 `https://github.com/Brunhild-A4D3EE/Visual-Cryptography`
2. **Settings → Pages**
3. **Source**: `Deploy from a branch`
4. **Branch**: `main` / `(root)`
5. Save，等 1-2 分钟

部署完成后访问：
```
https://brunhild-a4d3ee.github.io/Visual-Cryptography/
```

## 📐 调参建议

### Tab 2 (3-shares / 3-secrets)

| 目标 | N | K | 秘密分辨率 | 备注 |
|---|---|---|---|---|
| 秘密最清晰 | 4 | 2 | 120+ | 默认，12.5% 对比 |
| 网点最细 | 3 | 1 | 200+ | 11% 对比，小道具用 |
| 最高保密 | 8 | 4 | 80 | 6% 对比，秘密图必须超简单 |

**秘密图选图：** 简单高对比形状最佳（图标、文字、剪影）。不要用照片当秘密——细节会被半色调网点淹没。

### Tab 3 (彩色 CMYK)

- Bayer 抖动建议**开启**（保留中间色调）
- 放大倍数 ≥ 2 才能看清网点
- 想要秘密清晰：用饱和度高的图，对比度强的图

## 📁 文件说明

- `index.html` — 主页面（GitHub Pages 默认入口）
- `visual_cryptography.html` — 与 `index.html` 相同的备份副本（可删除）
- `README.md` — 本文件
- `.gitignore` — 排除系统文件

## 🔬 数学背景

视觉密码学背后的对称性论文：

- **Naor, M. & Shamir, A.** (1995). *Visual Cryptography*. EUROCRYPT '94.
- **Zhou, Z., Arce, G. R., & Di Crescenzo, G.** (2006). *Halftone Visual Cryptography*. IEEE Trans. Image Processing.
- 多秘密视觉密码扩展见 Ateniese, Blundo et al. 系列论文

## 📜 License

MIT —— 随便用，记得说明出处。
