---
title: Fellou AI 浏览器使用体验
date: 2025-09-03 14:27:00
categories:
- 技术
- AI
- 浏览器
tags: 
- AI
---

Fellou 是一款由95后团队打造的Agentic Browser--行动型浏览器，侧重端到端自主行动，一种集成了具备思考和行动能力的智能代理的浏览器，其不仅展示信息，更能根据用户高层目标自主拆解任务、跨界操作并完成端到端任务交付。今天拿到了邀请码，体验了一下。

## 开始

首次使用，输入邀请码登录已经，可以选择导入某个浏览器的BookMarks, History，Passwords和Addresses，我选择了Chrome的记录。

![Login](/images/fellou_01.png)

## 主要功能

除了正常的浏览器功能外，还有Deep Action和Agent的能力。UI展示上，右侧侧边栏会有AI对话框，Deep Action执行的过程中展示AI执行思路，已经调用原子操作的细节等。

## 附加功能

* 提示窗口，精细需求，AI给出提示，用户点击可以执行。比方说，提供用户进行酒店价格对比，点击以后，会开启一个新的DeepAction
* 控制权转移（用户登录，用户信息鉴权等）

## AI功能

我尝试了用它进行预定酒店，默认会去booking.com预定酒店，估计国内外都比较适配。

1. 先是多轮对话，明确需求 （有深度思考）
2. 根据需求，拆解任务
3. 用户点击运行，开始执行
4. 根据前边拆分的任务，利用原子操作执行操作

### 任务能力
 可以暂停、恢复和重新执行任务
![Task](/images/fellou_03.png)

### 摘取的一些原子操作

* navigate_to: 跳转到页面
* go_back: 退回上个页面
* current_page: 检查当前页面
* click_element: 点击元素
* click_by_point: 根据坐标点击元素
* input_text: 输入框输入文本
* get_select_option: 获取选项组件的所有选项
* select_option: 选择选项
* scroll_mouse_wheel: 滑动鼠标滚轮
* extract_page_content: 提取页面内容
* take_screen_shot: 截图（观察页面需要）
* human_interact: 用户交互，会中断当前任务，等用户操作完以后，继续操作

## Agent

* Deep Search Agent
* Image Agent
* Code Agent: 搭建网站
* Browser Agent： 网站发布
* Music Agent： 创作音乐作品

尝试了它官方给的任务指令：
> 搜索 Fellou 的信息，创建 2 个新 logo（图片 + SVG）和一首价值导向的音乐作品，搭建网站展示这些内容，并发布到 X.com

它执行了几个Agent 任务，Deep Search Agent、Music Agent、Image Agent、Code Agent和Browser Agent


### Deep Search Agent

* 并行搜索搜集信息
* 信息总结展示

### Music Agent

* variable_storage (存储一些变量，音乐信息，Fellou品牌信息)
* generate_lyrics
* generate_music

### Image Agent: 设计Logo, 生成Logo

### Code Agent

根据之前收集到的信息和生成的图片、音乐，生成代码搭建网站。

### Browser Agent

最后将搭建的网站的信息发布到了X.com 上。
![Twitter publish](/images/fellou_06_twitter_pub.png)


## 使用体验

### 优点

* 每一步都有反馈和说明
* 基本完整的流程能顺利完成（效果体验不错）

### 缺点

* 运行很慢，有时候单独一个步骤会卡很久
* 有可能会被网站的安全拦截
* 窗口会被强制占用，切到其他窗口会切回到Fellou的窗口


整体来说，确实是相对传统的浏览器有很大的跨越式进步，也结合了很多现在 AI 和 Agent的形态，能力上很完备，还是非常惊艳的。