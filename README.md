# 机器狗控制台 · 四川轻化工大学 / 开放原子开源协会

给协会招新、换届演示用的机器狗遥控页。纯静态：双击 `index.html` 就能用，不需要服务器、不需要联网。

---

## 一、怎么用

1. 打开 `index.html`（手机建议「添加到主屏幕」）。
2. 右上角填 IP（狗背屏「设置」里看）→ 点 **连接**。
3. **连接成功的判定 = 发一次 `GET /move?dir=stand` 并返回 200**：按钮变绿「已连接」，失败变红「重试」。
4. 想显示协会徽：把徽标图存成 `oaa.jpg` 放同目录（缺了显示灰底「OAA」占位，不影响功能）。

> 手机/电脑要和狗连**同一个 WiFi**。

---

## 二、页面结构

| 区块 | 内容 |
| --- | --- |
| **顶栏** | 左：徽标 + 「四川轻化工大学 / 开放原子开源协会」；右：IP 输入 + 连接。 |
| **中间三栏** | 左「动作」两列；中间十字方向键；右「表情」两列。 |
| **自定义灯光** | 8 色 + 关灯 + 亮度（默认 50%）+ 闪烁 / 流水 / 呼吸 / 彩虹。 |
| **绘图** | 128×64 画板 + `上屏` / `只上传` / `选图片·视频`，下面列出设备里已有的表情（播放 / 配音乐 / 删）。 |
| **自定义动作** | 每步 4 个舵机角（0–180°）+ 延时（0–65535 ms），可加步/删步（≤100 步）。 |

**方向键**：按住走、松手发 `dir=stop`；键盘 WASD / 方向键也行，空格急停。切走窗口或抬起事件丢失也会自动补一次 stop。

---

## 三、用到的接口（全部实测，和 `api文档.md` 不完全一样）

### 运动 / 姿态 / 表情

| 用途 | 请求 |
| --- | --- |
| 方向 | `GET /move?dir=forward\|back\|left\|right\|stop` |
| 姿态 | `GET /move?dir=stand\|sitdown\|sleep\|wake\|stretch\|scratching\|swing\|petwave\|jump` |
| 表情 | `GET /emotion?emotion=<名>`（含 `random`） |

### 灯光（颜色是中文名，且 `dir` 写两遍）

| 用途 | 请求 |
| --- | --- |
| 换色 | `GET /move?dir=light&dir=light&color=红灯&brightness=50` |
| 关灯 | `GET /move?dir=light&dir=light&color=关灯&brightness=0` |
| 灯效 | `GET /move?dir=light_effect&dir=light_effect&effect=rainbow_light&brightness=50` |

- 色名：`红灯 / 橙灯 / 黄灯 / 绿灯 / 青灯 / 蓝灯 / 紫灯 / 白灯`（绿/青/紫 是按规律补的，未逐个实测）。
- 灯效那条的 `brightness` 会被忽略，所以关灯必须走颜色通道的 `关灯`。

### 自定义表情（GIF，不是 `/canvas`）

`/canvas` 在这台固件上已删除（返回 `410 Canvas feature removed`）。真机是：

| 步骤 | 请求 |
| --- | --- |
| 建目录 | `POST /sdcard/mkdir?path=%2Fcustom_emo` |
| 上传 | `POST /sdcard/upload`，头 `X-Sd-Path: /custom_emo/<名>.gif`、`Content-Type: application/octet-stream`，body = 128×64 黑白 GIF（≤115KB），成功返回纯文本 `OK` |
| 生效 | `cmd=reload` → `cmd=set_config…&loop=1` → `cmd=play` |
| 列表 | `GET /custom_emo?cmd=list` → `{ items: [{name, kind, size, music, …}] }` |
| 删 | `GET /custom_emo?cmd=delete&name=<名>` |
| 配音乐 | `GET /custom_emo?cmd=set_config&name=<名>&music=<文件名>&loop=1` |

- 画板：像素 → **内置 GIF 编码器**（每像素前发 Clear，码宽恒 3 bit，体积约 6KB）→ 上传，像素是一对一的，不缩放。
- 音乐来自设备 `/music/mp3/`：`GET /sdcard?path=%2Fmusic%2Fmp3` → `{ entries: [{name,type}] }`，过滤 `type==='dir'` 和 `mp3/wav/m4a`。

### 自定义动作

| 用途 | 请求 |
| --- | --- |
| 列表 | `GET /custom_action` → 字符串数组 |
| 播放 | `GET /custom_action?action=<名>` |
| 读取 | `GET /custom_action?name=<名>` |
| 保存 | `POST /custom_action?name=<名>`，`Content-Type: application/json`，body `{"steps":[{"lf":90,"rf":90,"lb":90,"rb":90,"delay":500}, …]}`（≤8192 字节、≤100 步） |
| 删除 | `DELETE /custom_action?name=<名>` |

> 名字坑：设备有时把动作名以 **URL 编码后的形式**返回（`%E7%AB%99%E7%AB%8B`）。页面统一先解码再编码一次，避免双重编码导致 500。

---

## 四、注意事项

- 这些接口**没有鉴权**：同一 WiFi 下谁打开都能控制狗，演示完关掉页面，别把狗暴露到公网。
- 页面用 `https` 托管时请求 `http://192.168.x.x` 属于混合内容，部分浏览器会拦——现场用本机文件打开最稳。
- 表情名、自定义动作名随固件不同会变。
- `jump`、`swing` 和走动类动作，展台上先在地面试。
- 配置只存在浏览器 localStorage（IP、表情列表），不落服务器；删除浏览器数据会重置。

---

## 五、可选文件

| 文件 | 作用 |
| --- | --- |
| `oaa.jpg` | 顶栏协会徽（缺了用「OAA」占位） |
| `emotion_gif_convert.js` | 只有「选图片/视频做表情」需要（把 GIF/图片/视频抽帧转成 128×64 黑白 GIF）。画板不需要它 |

---

## 六、迎宾展示页 `welcome.html`

单独一个页面，专门给招新现场摆着看：

- **固定文字**：四川轻化工大学 / **欢迎新同学** / 开放原子开源协会 · 招新现场（大标题是流动的彩虹渐变）；
- **点「开始迎宾」后的顺序**：
  1. **上屏**（只做一次）：把「狗屏文字」渲染成 128×64 黑白 GIF（`/custom_emo/welcome.gif`）→ 上传 → `reload` → `set_config`（不带音乐）→ `play`；
     默认**三行**、字号封顶 12px（放不下自动降 11/10/9/8）：
     ```
     四川轻化工大学
     欢迎XXX同学
     SUSE开放原子开源协会
     ```
     自动换行、每行居中、整块垂直居中；在「✎ 改文案 → 狗屏文字」里可写任意多行（3 行是这块屏的极限，每行别超 12 个汉字）；
  2. **先点亮一个颜色**：`dir=light&color=白灯&brightness=50`（失败退 `红灯`）；
  3. **摇摆 + 彩虹灯，重复 N 次**（N = 开始按钮旁边的「次动作」，默认 5）：每次同时发 `dir=swing` 和
     `GET /move?dir=light_effect&dir=light_effect&effect=rainbow_light&brightness=50`，间隔 **1.8 秒**；跑满自动停。

> **固件的坑**：`light_effect` 只在「灯已经亮着」的时候才生效。刚关过灯、或狗刚开机时直接发 rainbow 会被忽略，所以开始时会先给一个底色（白灯）再上彩虹；运行中如果某次彩虹失败，会自动补一次底色再重试。
- **不放音乐**：`set_config` 的 `music` 传空，避免和上屏混在一起。
- **停止 = 停 + 关灯**：`custom_emo?cmd=stop` → `/emotion?emotion=neutral` → `dir=stop` → 关灯，600ms 后再补一次关灯防止残留。跑满 N 次自动停也走这条路径（不会自动趴下）。
- **文案可改**：右下角铅笔 ✎ / 「✎ 改文案」按钮 / 直接点大标题，打开面板可填「学校 / 主标题 / 副标题 / 狗屏文字」，保存后存在本机（localStorage `welcome_text_v1`），刷新还在；留空则恢复默认。
- 和主控制台**共用同一个 IP 配置**（localStorage `dog_console_v4`），主页面连过的狗，这里点「连接」即可；
- 连上后自动申请屏幕常亮（Wake Lock），摆摊/投屏不会自动息屏。

用法：打开 `welcome.html` → 确认 IP → 「连接」→「开始迎宾」。
