
# message - aardio 优雅的全局/窗口消息提示组件

基于 aardio 与 GDI+ 构建的高性能、平滑动画消息提示组件。支持独立宿主窗口吸附、暗黑模式切换、DPI 自动适配以及多行文本自适应排版。

## ✨ 特性

- **丝滑动画**：内置 60FPS 定时器，支持阻尼弹簧效果的位移与淡入淡出。
- **GDI+ 渲染**：利用 `WS_EX_LAYERED` 分层窗口，实现高质量圆角、阴影与 ClearType 抗锯齿文本。
- **多实例隔离**：按窗口句柄（HWND）隔离消息队列，互不干扰，支持依附于指定窗口或全局桌面。
- **智能排版**：内置 9 宫格定位，自动堆叠避让；支持超限自动剔除旧消息。
- **DPI 自动适配**：所有参数按逻辑像素配置，底层自动计算宿主窗口的 DPI 缩放。
- **动态换肤**：内置 `light` 与 `dark` 两套主题，并支持运行中无缝热切换。

---

## 📦 安装扩展库

```aardio
import ide
ide.installLib("message","https://github.com/nlysh007/message-aardio/releases/latest/download/message.tar.lzma")
```
## 🚀 快速上手

### 1. 初始化

你可以创建一个全局消息提示，或者让它依附在某个特定窗口上：

```aardio
import message;

// 依附于主窗口（推荐：消息不会飞出窗口外）
var msg = message(mainForm.hwnd);

// 或：全局桌面消息
// var msg = message();

```

### 2. 基础调用

显示消息

```aardio
msg.show("这是一条的default消息");//默认消息，无icon
```

内置了 5 种默认语境，开箱即用：

```aardio
msg.success("操作成功！");
msg.error("抱歉，网络连接失败。");
msg.warning("请注意检查输入格式");
msg.info("这是一条普通通知");
msg.loading("正在处理中，请稍候..."); // 持续旋转且不会自动消失

```

### 3. 高级自定义配置

在调用时，可以通过传入第二个 `table` 参数来覆盖默认行为。

```aardio
//多行文本
msg.info("这是一段长文本超过3行会被截断...", {
    pos = 5;            // 位置设置在屏幕正中 (1-9宫格，5为居中)
    duration = 0;       // 停留时间：0 表示不自动关闭
    showClose = true;   // duration为0时，显示关闭按钮
    maxWidth = 300;     // 限制最大宽度（逻辑像素）
    multiLine = true;   // 开启多行支持，防止长文本被截断
    maxLines = 3;       // 限制最多显示3行
    
});

//使用图片icon
import inet.http;
var img = inet.http().get("https://mat1.gtimg.com/qqcdn/qqindex2021/favicon.ico");; 
	msg.show("图片icon加载演示", {
	icon = { 
    	image = img; // 图片路径
    	size = 20;    // 图片大小
	}
	});

```
### 4. loading消息回调
```aardio
var loadMsg = msg.loading("这是一条耗时加载消息",5);
var progress = 0;
winform.setInterval(
    function(){
        progress += 10;
        if(progress <= 100){
            // 核心功能展现：直接针对对象的属性赋值，卡片自适应宽度和平滑吸附
            loadMsg.text = "正在下载文件: " + progress + " % / 100 % (动态自适应宽度演示)";
        } else {
            loadMsg.close();
            msg.success("文件下载并处理完成！",5);
            winform.btnLoading.disabled = false;
            return 0; // 停止定时器
        }
    }, 500
)

```
---

## ⚙️ 全局方法

### 🧹 清理消息窗口

```aardio
//立即清理全部消息窗口
message.clearAllGlobal(true) //true:直接关闭，不触发动画 false:触发关闭动画

```

### 🌙 深色模式切换

```aardio
// 切换到深色模式，并立即同步更新屏幕上现存的未关闭消息
message.setThemeMode("dark", true); //true:立即同步  false:修改主题配置

```

---

## 📐 配置参数表 (Options)

参数均支持在 **全局** (`message.config.default`)、**实例化** (`message(hwnd, cfg)`) 以及 **单次调用** (`msg.info("...", cfg)`) 三个级别进行设置。

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `pos` | number | `8` | 消息出现的位置，采用小键盘 1-9 布局（8 为正上方） |
| `duration` | number | `3000` | 消息停留时间 (ms)。`0` 为不自动关闭 |
| `h` | number | `44` | 消息框基础高度 (多行模式下会根据内容自动撑开) |
| `margin` | number | `20` | 距离屏幕/窗口边缘的安全边距 |
| `spacing` | number | `15` | 多条消息同时出现时的间距 |
| `fontSize` | number | `16` | 字体大小 (逻辑像素) |
| `topmost` | boolean | `true` | 是否强制置顶 |
| `showClose` | boolean | `true` | 当 `duration=0` 时，是否显示红色的关闭小叉号 |
| `clickToClose` | boolean | `false` | 是否允许点击消息主体任意位置关闭窗口 |
| `multiLine` | boolean | `false` | **重要：** 长文本时是否开启多行换行支持 |
| `maxWidth` | number | `600` | 消息框最大允许宽度 |
| `maxLines` | number | `3` | 开启多行时，最大显示的行数 (超出会截断) |
| `icon` | table | null | icon配置 |

icon配置

```
icon = { 
        image = path|imgData  //文件路径或图片数据 ，image优先显示,会忽略字体图标
        text = '\uE73E'; //字体图标
        font = "Segoe MDL2 Assets"; 
        color = 0xFF67C23A; 
        size = 18; 
        } 

```

## 📦 目录结构说明

该库采用高内聚、低耦合的模块化设计，各文件职责清晰：

```text
message/
├── _.aardio          # [模块入口] 定义命名空间、全局默认配置与主题色板，封装便捷 API
├── manager.aardio    # [逻辑调度] 核心管理器，负责维护消息队列、实例隔离及防重叠逻辑
├── render.aardio     # [绘图计算] 专注 GDI+ 逻辑，处理文本排版、尺寸计算与位图缓存生成
└── window.aardio     # [UI 载体] 处理分层透明窗口 (WS_EX_LAYERED)，负责动画驱动与鼠标交互
