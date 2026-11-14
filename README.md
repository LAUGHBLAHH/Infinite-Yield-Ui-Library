# InfiniteYieldUI

从用户指定的 `EdgeIY/infiniteyield` 源码中提取并解耦的客户端 UI 库。
这是保留原版视觉结构的模块化改编，不是原脚本全量复制或所有编辑器的逐项移植。

## 目录

- [安装与快速开始](#安装与快速开始)
- [从 GitHub 加载](#从-github-加载)
- [窗口配置](#窗口配置)
- [注册和执行命令](#注册和执行命令)
- [页面与控件](#页面与控件)
- [通知与主题](#通知与主题)
- [显示行为与资源清理](#显示行为与资源清理)
- [API](#api)
- [验证与限制](#验证与限制)

## 保留与剥离

- 保留 250×220 主窗口、SourceSans 字体、原版六组主题颜色、命令栏和滚动列表。
- 封装命令筛选、点击补全、描述提示、前缀唤起、自动收起、设置页、可关闭/固定通知。
- 主窗口固定在左下角且不可拖动。默认启动显示原版 Logo 放大、作者名单滑入及淡出，然后收起到 20 像素标题栏。
- 鼠标移入或点击标题展开，移出约 1 秒且未聚焦输入框时收起；展开使用 0.2 秒、收起使用 0.5 秒 Quart InOut 过渡。触屏可点击标题展开，点击左上角减号收起。
- 增加独立页面及通用按钮、开关、输入框、标签接口。
- 删除原脚本的游戏命令、远程调用、Hook、插件加载、网络加载、文件读写和全局状态。
- 不移植与业务数据强耦合的快捷键/别名/路径/插件编辑器、聊天监听和完整颜色选择器；颜色通过 `SetTheme` 配置。
- 默认挂到 `LocalPlayer.PlayerGui`，不依赖执行器 API；需要其他父级时显式传 `Parent`。

## 安装与快速开始

1. 将 `InfiniteYieldUI.luau` 内容放入 `ReplicatedStorage.InfiniteYieldUI` ModuleScript。
2. 将 `example.client.luau` 放进 `StarterPlayerScripts` LocalScript。
3. 运行客户端。`;` 聚焦命令栏，RightShift 显示/隐藏，左上角收起，右上角打开设置。

文件用途：

| 文件 | 用途 |
| --- | --- |
| [InfiniteYieldUI.luau](./InfiniteYieldUI.luau) | UI 库，模块返回 `Library` |
| [example.client.luau](./example.client.luau) | Studio 客户端使用示例 |
| [test.cjs](./test.cjs) | Node.js 模拟接口测试 |
| [LICENSE](./LICENSE) | 原项目 MIT 许可 |

也可在 LocalScript 中直接使用以下最小示例：

```lua
local UI = require(game:GetService("ReplicatedStorage"):WaitForChild("InfiniteYieldUI"))
local window = UI:CreateWindow({
    Title = "My UI",
    Prefix = ";",
    ToggleKey = Enum.KeyCode.RightShift,
})
window:AddCommand({
    Name = "hello [message]",
    Description = "显示问候通知",
    Callback = function(args, win)
        win:Notify("Hello", table.concat(args, " "), 5)
    end,
})
```

点击命令条目只填入命令；按 Enter 才调用回调。参数按空白拆分，不解析引号、转义或原版命令链。
按 Enter 提交时立即开始收起动画，不等待鼠标离开或延时；即使开启 Keep Menu Open，提交时也会收起。
搜索按命令名、显示文字、别名、描述做不区分大小写的字面子串匹配，不将输入解释为 Lua pattern。

## 从 GitHub 加载

README 使用 GitHub Markdown、相对文件链接和页内目录，可与库一起提交到仓库，无需修改本地绝对路径。
推荐把本目录内的文件放到 GitHub 仓库根目录；若保留 `infiniteyield-ui/` 子目录，Raw URL 也需要带上该目录。

在提供 `game:HttpGet` 与 `loadstring` 的客户端执行环境中，可通过 Raw URL 加载：

```lua
-- 替换 OWNER、REPOSITORY 和 main；main 是示例分支名。
local url = "https://raw.githubusercontent.com/OWNER/REPOSITORY/main/InfiniteYieldUI.luau"
local UI = loadstring(game:HttpGet(url))()
local window = UI:CreateWindow({ Title = "My UI" })

window:AddCommand({
    Name = "hello [message]",
    Callback = function(args, win)
        win:Notify("Hello", table.concat(args, " "), 5)
    end,
})
```

如果文件位于子目录，示例地址为：

```text
https://raw.githubusercontent.com/OWNER/REPOSITORY/main/infiniteyield-ui/InfiniteYieldUI.luau
```

GitHub 的 `/blob/` 页面地址不能代替 Raw 地址。需要固定版本时，可把 URL 中的分支名改为对应的提交 SHA。
普通 Studio LocalScript 使用上文的 ModuleScript / `require` 方式；此 HTTP 示例依赖执行环境提供额外能力。
本项目尚未代你发布到 GitHub，上述仓库地址均为占位符。

## 窗口配置

```lua
local window = UI:CreateWindow({
    Title = "My Command UI",
    Name = "MyScreenGui",
    Prefix = ";",
    KeepOpen = false,
    Intro = true,
    Scale = 1,
    ToggleKey = Enum.KeyCode.RightShift,
    DisplayOrder = 100,
    Logo = "rbxassetid://1352543873",
    Credits = "My project · UI adapted from Infinite Yield",
    OnSubmit = function(rawText, args, win)
        win:Notify("未知指令", rawText, 3)
    end,
})
```

| 选项 | 默认值 | 说明 |
| --- | --- | --- |
| `Title` | `Infinite Yield UI` | 标题文字 |
| `Name` | `InfiniteYieldUI` | ScreenGui 名称；不会自动替换同名窗口 |
| `Parent` | `LocalPlayer.PlayerGui` | 挂载 UI 的父实例 |
| `Prefix` | `;` | 聚焦命令栏的按键，推荐单字符 |
| `KeepOpen` | `false` | 保持展开；按 Enter 提交仍会立即收起 |
| `Intro` | `true` | 是否播放启动动画 |
| `Scale` | `1` | 缩放，范围为 0.5–2 |
| `Theme` | 原版深色主题 | 主题颜色表，可仅传需要覆盖的项 |
| `ToggleKey` | 无 | 显示/隐藏的 `Enum.KeyCode` |
| `DisplayOrder` | `100` | ScreenGui 显示顺序 |
| `Logo` | 原版 Logo 资源 | 启动图片资源字符串 |
| `Credits` | 原版作者名单 | 启动时显示的文字 |
| `OnSubmit` | 无 | 未匹配到注册命令时的回调 |

## 注册和执行命令

支持直接在 `Name` 中使用中括号标出多个参数。中括号是列表里的用法提示，不是实际输入的字符：

```lua
window:AddCommand({
    Name = "hello [args1] [arg2]",
    Aliases = { "hi" },
    Description = "接收两个参数",
    Callback = function(args, win)
        -- 输入 hello Alice Bob 后：args = { "Alice", "Bob" }
        local first = args[1]
        local second = args[2]
        win:Notify("Hello", (first ~= "" and first or "未填写") .. " / " .. (second ~= "" and second or "未填写"), 5)
    end,
})
```

也可使用 `Name = "hello", Args = { "args1", "arg2" }` 自动生成 `hello [args1] [arg2]` 显示文字；显式 `Text` 优先覆盖列表文字。
回调的第一个参数始终为参数数组 table，不包含命令名；依次用 `args[1]`、`args[2]`、`args[3]` 访问。
`args` 的长度严格等于声明中的中括号数量：缺少的参数补为 `""`（空字符串），超出的输入参数忽略；没有中括号时传入空表。所有参数保持字符串，需要数字时使用 `tonumber(args[1])`。
声明优先级：显式 `Args` 数组（包括空数组）→ `Name` 后的参数声明 → `Text` 中的中括号。`entry.ArgumentCount` 可读取识别出的参数数量。
例如 `hello [args1] [arg2]`：输入 `hello Alice` 得到 `{ "Alice", "" }`；输入 `hello Alice Bob Extra` 得到 `{ "Alice", "Bob" }`；只输入 `hello` 得到 `{ "", "" }`。使用 `args[2] == ""` 判断是否缺少第二个参数，不要仅用 `args[2] or 默认值`（Lua 中空字符串为真）。
参数按空白拆分，不支持引号分组；输入 `hello Alice Bob`，而不是 `hello [Alice] [Bob]`。

```lua
local entry = window:AddCommand({
    Name = "echo",                  -- 实际指令名，推荐使用不含空格的唯一名字
    Text = "echo [message]",         -- 列表显示文字
    Description = "将参数显示为通知",
    Aliases = { "say" },
    Callback = function(args, win)
        win:Notify({
            Title = "Echo",
            Text = table.concat(args, " "),
            Duration = 5,
        })
    end,
})

window:Submit("say hello world")  -- args = { "hello", "world" }
-- entry:Destroy()               -- 移除这一条命令及其事件连接
```

命令名和别名匹配不区分大小写。`Name` 的第一个单词是实际命令名，后面的 `[参数]` 用于显示用法。不要给 `Name` 添加启动前缀：`;` 是唤起按键，实际输入的是 `echo hello`。
`Text` 仅决定列表展示；真正执行时使用 `Name` 或 `Aliases`。出现重复名字时，会先匹配先注册的命令。
`Submit` 返回是否匹配到命令，不等待异步回调完成。代码直接调用 `Submit` 不改变窗口显示；用户在命令栏按 Enter 时会清空输入并立即开始收起。

## 页面与控件

创建页面后，需要添加入口。以下代码将入口加到内置设置页，点击右上角设置按钮即可看到：

```lua
local page = window:AddPage("Custom")
window.Pages.Settings:AddButton({
    Text = "自定义设置",
    Callback = function() window:ShowPage("Custom") end,
})

local label = page:AddLabel("当前状态：关闭")
local toggle = page:AddToggle({
    Text = "示例开关",
    Default = false,
    Callback = function(enabled)
        label:Set(enabled and "当前状态：开启" or "当前状态：关闭")
    end,
})

local textbox = page:AddTextbox({
    Text = "输入内容",
    Default = "",
    Placeholder = "在这里输入",
    Callback = function(value, pressedEnter)
        print("输入：", value, "通过 Enter 提交：", pressedEnter)
    end,
})

page:AddButton({
    Text = "显示输入内容",
    Callback = function() window:Notify("内容", textbox:Get(), 3) end,
})
page:AddButton({
    Text = "返回命令列表",
    Callback = function() window:ShowPage("Commands") end,
})

toggle:Set(true)             -- 更新值，发生变化时调用回调
toggle:Set(false, true)      -- 第二个参数 true 表示静默更新
textbox:Set("预设内容", true)
```

`Commands` 与 `Settings` 为保留页面名，不可再次创建；其他页面名也必须唯一。
输入框失焦时回调，普通按钮点击时回调，创建控件本身不会触发初始回调。
通用输入框按 Enter 只触发其回调；自动收起规则针对主命令栏。

## 通知与主题

```lua
local notice = window:Notify({ Title = "提示", Text = "操作完成", Duration = 5 })
notice:Pin(true)   -- 固定，避免定时自动关闭
notice:Close()     -- 手动关闭

window:SetTheme({
    Shade1 = Color3.fromRGB(36, 36, 37),  -- 主背景
    Shade2 = Color3.fromRGB(46, 46, 47),  -- 次级背景/按钮
    Shade3 = Color3.fromRGB(78, 78, 79),  -- 开关等控件背景
    Text1 = Color3.fromRGB(255, 255, 255),
    Text2 = Color3.fromRGB(0, 0, 0),      -- 预留的次级文字色槽
    Scroll = Color3.fromRGB(78, 78, 79),  -- 滚动条及输入占位文字
})
window:SetScale(1.2)
```

通知共用一个位置，新通知会替换旧通知。`Duration = 0` 表示不定时关闭；旧通知返回的句柄不会关闭后来的通知。
通知从屏幕底部以 0.5 秒 Quart InOut 动画滑入；计时从展开结束后开始，到期、点击关闭按钮或调用 `notice:Close()` 都以 0.5 秒滑回底部，再隐藏。
收回期间收到新通知会中断旧动画并重新展开，不会被旧通知的隐藏任务误关。
`Pin(false)` 取消固定，但不会重新启动已结束的计时器；需要重新计时时可再次调用 `Notify`。

## 显示行为与资源清理

1. 默认先在左下角播放原版启动动画，结束后收起为 20 像素标题栏。
2. 鼠标移入或点击标题展开；触屏点击标题展开，点击减号收起。
3. 鼠标离开约 1 秒后收起；输入框仍聚焦或开启 `KeepOpen` 时不自动收起。
4. 主命令栏按 Enter 后立即启动收起动画，不等待上述 1 秒，包含开启 `KeepOpen` 的情况。
5. 展开过渡为 0.2 秒，收起过渡为 0.5 秒；窗口始终固定左下角，不支持拖动。

```lua
window:Focus()                -- 显示、展开并聚焦主命令栏
window:SetMinimized(true)     -- 收起
window:SetMinimized(false)    -- 展开
window:SetVisible(false)      -- 隐藏整组 UI
window:SetVisible(true)       -- 恢复显示，保留收起状态
window:SetPrefix(";")

-- 在不再使用时调用；若需重新加载，请先销毁旧窗口。
window:Destroy()
```

`Destroy` 会断开库管理的事件连接并取消活动动画。你的业务回调自行创建的循环、任务及连接，需要由业务代码自行清理。

## API

| 接口 | 用途 |
| --- | --- |
| `UI:CreateWindow(options)` | 创建独立窗口 |
| `window:AddCommand({Name, Args?, Text?, Description?, Aliases?, Callback?})` | Name 可写 `hello [arg1] [arg2]`；回调接收参数数组；条目支持 `entry:Destroy()` |
| `window:Submit(text)` | 执行注册命令；匹配返回 true，否则回调 `OnSubmit` 并返回 false |
| `window:AddPage(name)` / `window:ShowPage(name)` | 创建/切换页面；`Commands` 和 `Settings` 是内置页面 |
| `page:AddLabel(text)` | 返回 `{Instance, Set}` |
| `page:AddButton({Text, Callback})` | 返回 `{Instance, Button}` |
| `page:AddToggle({Text, Default?, Callback?})` | 返回 `{Instance, Value, Set}`；`Set(value, silent?)` |
| `page:AddTextbox({Text, Default?, Placeholder?, Callback?})` | 返回 `{Instance, TextBox, Set, Get}`；失焦时回调 `(text, enter)` |
| `window:Notify({Title?, Text?, Duration?})` | 通知；也支持 `(title, text, duration)`；返回 `Close()` / `Pin(value?)` |
| `window:SetTheme(partialTheme)` | 更新已创建的控件及后续控件主题 |
| `window:SetScale(number)` | 设置 0.5–2 倍缩放 |
| `window:SetPrefix(string)` | 设置输入唤起前缀；推荐单字符 |
| `window:Focus()` | 显示、展开并聚焦命令栏 |
| `window:SetMinimized(boolean)` / `window:SetVisible(boolean)` | 收起/展开或显示/隐藏整组 UI |
| `window:Destroy()` | 销毁 GUI 并断开库管理的连接 |

窗口选项：`Title`, `Name`, `Parent`, `Prefix`, `KeepOpen`（默认 false）, `Scale`, `Theme`, `ToggleKey`, `DisplayOrder`, `OnSubmit`, `Intro`（默认 true）, `Logo`（启动图片资源）, `Credits`（启动作者文字）。
`Position` 不再生效，主窗口始终固定在左下角。`KeepOpen = true` 可以在启动结束后保持展开；`Intro = false` 跳过启动动画。
命令回调参数为 `(args, window)`；`OnSubmit` 参数为 `(rawText, argsWithoutName, window)`。
普通回调异步执行并捕获错误；创建控件不会触发初始回调。开关赋相同值不重复回调。
通知采用单槽替换，默认 10 秒；`Duration = 0` 持续显示；固定后需手动关闭或下一条通知替换。
`KeepOpen = false` 时鼠标离开约 1 秒后收起；不提供配置持久化。
窗口坐标使用未缩放的逻辑像素；可通过 `Scale` 调整大小，缩放后仍固定在左下角。

## 验证与限制

在已安装 Node.js 的环境中，可运行模拟接口测试：

```sh
npm install --no-save luaparse fengari
node test.cjs
```

测试涵盖语法、命令匹配、搜索、控件、主题、前缀、通知、启动完成后的收起，以及事件清理。
模拟测试不验证 Roblox 的真实渲染、动画时间和移动设备输入效果，这些需要在客户端检查。
没有 `Players.LocalPlayer` 的服务端 Script 无法创建此库；请使用客户端 LocalScript。
默认 Logo 依赖 Roblox 图片资源正常加载。库不会自动保存配置，也不内置原版游戏命令。

## 来源

原始项目：<https://github.com/EdgeIY/infiniteyield>

原始源码：<https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source>

原作者版权及 MIT 许可见 `LICENSE`。本目录中的 UI 库为提取/重构版本，不加载原项目脚本。
