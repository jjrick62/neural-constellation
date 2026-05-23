---
name: project-overview
description: 个人主页"神经星座"项目概览 — Active Theory 风格沉浸式 3D 单页
metadata:
  type: project
---

# 个人主页项目

## 基本信息
- 项目名：Vincent 个人主页 — "神经星座"（Neural Constellation）
- 目录：`D:\大学作业文件夹\自制软件\个人主页\`
- 产品文件：`个人主页.html`（单文件）
- 对标参考：activetheory.net（已通过 MirrorKit 扒站分析）

## 当前架构
- Three.js r160 CDN + GSAP ScrollTrigger + 自写 GLSL Shader
- 13 个 OOP 类，单 HTML 文件
- 4 个 3D 场景，滚动驱动摄像机飞行
- 自建粒子系统（高斯聚类 + 外部光源反光，非自发光）
- 零外部美术素材

## 最新重构方向（2026-05-23）
- 所有内容放入高 24 单位、半径 4 单位的圆柱
- 4 个高度分区，每区不同摄像机轨迹
- 层间切屏过渡特效（径向擦除 + 粒子爆发 + 交叉淡化）
- 粒子系统重写：固定位置 + 外部光源照亮 + NormalBlending
