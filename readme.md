
# message - aardio 优雅的全局/窗口消息提示组件

基于 aardio 与 GDI+ 构建的高性能、平滑动画消息提示组件。支持独立宿主窗口吸附、暗黑模式切换、DPI 自动适配以及多行文本自适应排版。

## ✨ 特性

- **丝滑动画**：内置 60FPS 定时器，支持阻尼位移与淡入淡出；5 号位使用中心缩放入退场。
- **GDI+ 渲染**：利用 `WS_EX_LAYERED` 分层窗口，实现高质量圆角、阴影与无彩边抗锯齿文本；内置 `fonts.FluentIcons` 图标字体，字体 icon 按实际字形轮廓与底图圆光学居中。
- **多实例隔离**：按窗口句柄（HWND）隔离消息队列，互不干扰，支持依附于指定窗口或全局桌面。
- **智能排版**：内置 9 宫格定位；1-3、7-9 自动纵向堆叠，4-6 在中线覆盖堆叠；支持超限自动剔除旧消息。
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

// 使用图片 icon；暗色主题会自动柔和压暗高光，并保留原透明度
import inet.http;
var img = inet.http().get("https://mat1.gtimg.com/qqcdn/qqindex2021/favicon.ico");
msg.show("图片 icon 加载演示", {
    icon = {
        image = img;             // 图片路径、图片数据或 gdip.bitmap
        size = 20;               // 图片大小
        // darkBrightness = 0.76; // 可选：仅覆盖当前图片的暗色亮度
        // darkSaturation = 0.92; // 可选：仅覆盖当前图片的暗色饱和度
        // darkLift = 0.025;      // 可选：仅覆盖当前图片的暗部提升量
        // darkAdjust = false;    // 可选：当前图片不做暗色调节
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
| `pos` | number | `8` | 小键盘 1-9 布局；1-3、7-9 纵向堆叠，4-6 中线覆盖堆叠，5 从中心缩放 |
| `duration` | number | `3000` | 消息停留时间 (ms)。`0` 为不自动关闭 |
| `h` | number | `48` | 消息框基础高度 (多行模式下会根据内容自动撑开) |
| `margin` | number | `18` | 距离屏幕/窗口边缘的安全边距 |
| `spacing` | number | `10` | 多条消息同时出现时的间距 |
| `fontSize` | number | `15` | 字体大小 (逻辑像素) |
| `topmost` | boolean | `true` | 是否强制置顶 |
| `showClose` | boolean | `true` | 当 `duration=0` 时，是否显示并启用关闭小叉号 |
| `clickToClose` | boolean | `false` | 是否允许点击消息主体任意位置关闭窗口；启用后消息会接收鼠标 |
| `multiLine` | boolean | `false` | **重要：** 长文本时是否开启多行换行支持 |
| `maxWidth` | number | `600` | 消息框最大允许宽度 |
| `maxLines` | number | `3` | 开启多行时，最大显示的行数 (超出会截断) |
| `enterScale` | number | `0.60` | 5 号位中心缩放入场的初始比例 |
| `exitScale` | number | `0.60` | 5 号位中心缩放退场的目标比例 |
| `icon` | table | null | icon 配置 |
| `autoCenterIcon` | boolean | `true` | 按字体 icon 的实际可见边界与圆形底图自动光学居中；不影响图片 icon |
| `darkIconAdjust` | boolean | `true` | 暗色模式是否自动调节图片 icon；不影响字体 icon |
| `darkIconBrightness` | number | `0.76` | 暗色模式图片高光亮度，范围 `0-1` |
| `darkIconSaturation` | number | `0.92` | 暗色模式图片饱和度，范围 `0-1` |
| `darkIconLift` | number | `0.025` | 暗部提升量，避免深色细节丢失，范围 `0-1` |

鼠标交互采用按需命中：`duration=0 && showClose=true` 或 `clickToClose=true` 的消息会接收鼠标，但保持 `WS_EX_NOACTIVATE`，点击不会抢走当前输入焦点；其他消息保留鼠标穿透，避免短暂提示遮挡下方程序操作。消息开始退场后也会立即恢复穿透。

icon 配置

```
icon = {
    image = pathOrImageData; // 文件路径、图片数据或 gdip.bitmap；image 优先于 text
    text = '\uE73E';         // FluentIcons 字体图标
    font = "FluentIcons";    // 省略时默认使用 FluentIcons
    color = 0xFF67C23A;
    size = 18;
    style = 0;               // 可选：GDI+ 字体样式
    autoCenter = true;       // 可选：是否自动按当前字形的实际可见边界居中
    offsetX = 0;             // 可选：自动居中后再水平微调（逻辑像素）
    offsetY = 0;             // 可选：自动居中后再垂直微调（逻辑像素）

    // 以下参数仅作用于图片 icon，并覆盖同名全局默认值
    darkAdjust = true;
    darkBrightness = 0.76;
    darkSaturation = 0.92;
    darkLift = 0.025;
}

```

组件会自动 `import fonts.FluentIcons`，默认语义图标均使用 `FluentIcons`；其中 loading 使用 `\uF16A`（ProgressRingDots）。字体 icon 默认会按光栅化后的实际 Alpha 边界与底图圆心对齐，而不是按字体行高或布局框机械居中；校正结果会缓存，不会在动画每帧重复测量。loading 旋转图标也固定围绕底图圆心旋转。特殊字体需要保留其设计偏移时，可设置 `icon.autoCenter=false`；也可用 `icon.offsetX/offsetY` 在自动居中后继续微调。

暗色调节采用柔和色调压缩矩阵：压低高光、轻抬暗部并保留适量饱和度，因此亮色图片不会刺眼，深色细节也不易糊成一团；PNG/ICO 的 Alpha 透明边缘保持不变。设置 `darkIconAdjust=false` 可全局关闭，也可用 `icon.darkAdjust=false` 只关闭单张图片的调节。

## 📦 目录结构说明

该库采用高内聚、低耦合的模块化设计，各文件职责清晰：

```text
message/
├── _.aardio          # [模块入口] 定义命名空间、全局默认配置与主题色板，封装便捷 API
├── manager.aardio    # [逻辑调度] 核心管理器，负责维护消息队列、实例隔离及防重叠逻辑
├── render.aardio     # [绘图计算] 专注 GDI+ 逻辑，处理文本排版、尺寸计算与位图缓存生成
└── window.aardio     # [UI 载体] 处理分层透明窗口 (WS_EX_LAYERED)，负责动画驱动与鼠标交互
