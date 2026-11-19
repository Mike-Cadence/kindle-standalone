# Kindle Desk Standalone

把闲置的 Kindle Basic 7 / KT2 变成每天可用的墨水屏桌面看板。程序直接运行在设备端 Python 3.9 上，用 Pillow 绘制 600×800 单色界面并通过 `eips` 刷新，不依赖浏览器；电脑关机后照常显示。

> 已在 **Kindle Basic 7 / KT2（2014），固件 5.12.2.2，600×800** 上验证。其他机型、分辨率和触摸接口尚未适配，请按移植开发对待。

## 功能

| 功能 | 说明 |
| --- | --- |
| 天气与预报 | 首页显示温度，点天气查看未来预报；按网络出口估算城市，也可手动设置（Open-Meteo）。 |
| 日程与日历 | 首页显示五条即将到来的日程；点日期或日程进入列表与月历。 |
| 待办事项 | 首页显示三条；点方框完成，后续待办自动补位，完成记录仍可查看。 |
| 本机输入 | 看板内直接填写，支持离线拼音、英文、数字，无需切换浏览器。 |
| 手机管理 | 扫码后在手机/电脑上编辑日程、待办、城市、模块与图片。 |
| 自选模块 | AI 金融资讯、综合新闻、微博热搜、Codex 额度与 DeepSeek 余额、图片轮播。 |
| 待机桌面 | 电源键休眠/唤醒；待机显示大日历或全屏图片，唤醒后更新内容。 |
| 番茄钟 | 点时间进入，默认 45 分钟，可设 5–180 分钟，圆环随时间消失。 |
| 农历与节气 | 日期下方显示农历，星期上方显示节日或节气，断网可用。 |
| 墨水屏刷新 | 启动全刷，之后每 10 次画面变化全刷一次，减少残影。 |
| 省电 | 空闲后进入深度休眠，配合 RTC 定时唤醒；休眠窗口避开 firmware 的自动挂起。 |
| 无线更新 | 电脑在同一局域网内更新程序与拼音词库，保留设备上的日程和待办。 |

日程、待办、计时和农历完全在本地运行；天气和资讯需要联网；手机管理需要设备与手机在同一 Wi-Fi。

## 适用前提

- 已越狱的 Kindle Basic 7 / KT2（2014），600×800 竖屏。
- 一个可执行 shell 脚本的启动环境（书库脚本入口或 KUAL）。
- Kindle 版 Python 3.9，包含 Pillow、SSL、FreeType、zlib，路径默认为 `/mnt/us/python3/bin/python3.9`。可参考 [NiLuJe 的 Kindle 工具发布帖](https://www.mobileread.com/forums/showthread.php?t=225030) 中适配 **K5 / KT2** 的包。
- 本仓库不包含越狱载荷、固件和 Python 运行时。

## 安装

本仓库只包含设备端程序。**带自动安装器、KUAL 扩展、文档与测试的完整工程见 [kindle-desk](https://github.com/fangheihei-137/kindle-desk)**，首次安装推荐使用那里的 `install.py`。

手动安装：

1. 在 Kindle 上退出其他全屏应用，连接 USB。
2. 把本仓库所有文件复制到 `/mnt/us/kindle-standalone/`（USB 视图中为 `kindle-standalone` 文件夹）。
3. 任选一种启动入口：
   - 将 `IndependentDashboard.sh` 放到 `/mnt/us/documents/`，从书库打开；
   - 或从 KUAL 菜单执行 `/mnt/us/kindle-standalone/start.sh`。
4. 安全弹出，连接 Wi-Fi，打开看板。首次启动会自动创建 `data/`，可能需要约 30 秒。

> `start.sh` 需要 root 权限运行，并会暂停系统框架避免抢占屏幕；请通过书库脚本或 KUAL 启动，不要直接在普通 shell 里手动运行。

## 手机管理

看板首页点右下角手机图标会显示二维码。手机连接同一 Wi-Fi 后扫码，即可打开响应式管理页面（设备本地 HTTP 服务，端口 8766）。

- 首次启动会生成随机连接凭据，保存在设备的 `data/` 中。
- 公共或访客 Wi-Fi 常启用客户端隔离，此时手机无法访问看板。
- 不要公开自己设备的二维码或 `wireless-device.json`。

## 数据与隐私

| 路径 | 内容 | 是否入库 |
| --- | --- | --- |
| `data/` | 日程、待办、设置、资讯缓存、连接凭据 | 否（`.gitignore`） |
| `wireless-device.json` | 电脑无线更新凭据 | 否（`.gitignore`） |

程序联网仅用于获取天气、按 IP 估算城市、读取资讯与金句；没有账号系统，也不上传日程。备份时退出看板、连接 USB，复制整个 `kindle-standalone/data` 文件夹。

## 目录结构

```text
app.py                    主程序：HTTP 服务、触摸、电源与主循环
renderer.py               600×800 单色界面绘制、农历离线表
modules.py                资讯、微博热搜、用量、图片模块
pages.py                  Kindle 浏览器可用的服务端表单页
phone.py                  手机端响应式管理页面
native_input.py           原生输入法与表单
device_control.py         Wi-Fi、电源键、休眠与 RTC 唤醒
store.py                  日程数据存储与并发控制
updater.py                启动更新与失败恢复
start.sh / IndependentDashboard.sh   设备端启动脚本
assets/                   字体、CA 证书、离线拼音词库
vendor/                   qrcode、pypng、typing_extensions
sync_standalone.py        Windows：USB 同步到已安装设备
wireless_update.py        Windows：局域网无线更新
usb_editor.pyw            Windows：USB 日程编辑器
codex_usage_sync.py       电脑侧 Codex 用量同步
```

## 本地开发与自检

电脑端只用于预览和自检，**不能**替代 Kindle 上的 Python 运行时。

```sh
python -m pip install "Pillow>=9,<13"
python app.py --preview     # 本地预览，输出 data/preview.png
python app.py --self-test   # 校验渲染、存储、电源和输入流程
```

## 已知限制

- 仅 KT2 实测；其他机型需要调整屏幕尺寸、刷新命令和触摸事件。
- 无线更新不会覆盖 `start.sh` 与 `updater.py`，这两个文件变动时需用 USB 同步。
- 第三方数据源（天气、资讯、微博）接口变化会影响对应模块。
- 电脑关闭时，Codex 用量保留上次同步值；DeepSeek 显示的是余额而非历史用量。

## 许可与第三方

原创代码使用 [MIT License](LICENSE)。随附第三方组件保留各自许可：

- 思源黑体 CN（SIL OFL 1.1）：`assets/OFL-SourceHanSans.txt`
- 离线拼音数据（Apache 2.0）：`assets/ime/LICENSE`、`AUTHORS`、`NOTICE.txt`
- qrcode（BSD）、pypng（MIT）、typing_extensions（PSF）：`vendor/*.dist-info/LICENSE`
- 农历与节气离线表由 [lunar-python](https://github.com/6tail/lunar-python) 生成（MIT），来源声明见 `renderer.py`

Kindle 是 Amazon 的商标，本项目为社区项目，与 Amazon 无关联。
