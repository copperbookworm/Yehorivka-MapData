# Yehorivka-MapData

《解域》Squad 地图通行性分析 App（Yehorivka Passability）的**地图数据包 · 在线更新源**。

App 装包里自带一份数据，离线也能用；这里的文件是「更新包」：玩家在 App 里打开
「地图数据 · 在线更新」面板，点「检查更新」→「下载有更新的」，就会从本仓库拉取，不用重装 APK。
只有当地图数据（底图 / 坡度 / 树冠 / 水域 / 桥梁 / 边界 / 道路 / 障碍标注 / 旗点）变了，才需要重新发一次。

## 目录结构

```
maps-data/
    manifest.json     清单：schema / updated / base / builtin / maps
    albasrah.json     单图数据包（25 张，每张 1.6–3.7 MB，合计约 63 MB）
    yehorivka.json
    ...
```

raw 基址（App 面板里默认填的就是它，注意末尾带 `/`）：

```
https://raw.githubusercontent.com/copperbookworm/Yehorivka-MapData/main/maps-data/
```

## manifest.json

```json
{
  "schema": 1,
  "updated": "2026-10-27",
  "base": "https://raw.githubusercontent.com/copperbookworm/Yehorivka-MapData/main/maps-data/",
  "builtin": { "albasrah": "b85c3b38d302", "yehorivka": "8b23709cd41c" },
  "maps": {
    "yehorivka": {
      "name": "Yehorivka",
      "v": "8b23709cd41c",
      "size": 2843595,
      "sha256": "5dd934d389fb212d7e14f4b54cb5eab82046aadee7d7accaa81411ca6f9f92e8"
    }
  }
}
```

- `builtin`：某个 APK 版本内部打包的每张图的版本号，用来判断「装包里那份是不是已经旧了」。
- `maps[slug].v`：内容版本（取数据包去掉 `v` 后序列化 JSON 的 sha256 前 12 位），内容一变就变。
- `size` / `sha256`：App 下载时双重校验，对不上就丢弃，坏数据不会留在本机。

## 单图数据包（<slug>.json）

```json
{
  "schema": 1,
  "slug": "yehorivka",
  "name": "Yehorivka",
  "v": "8b23709cd41c",
  "minx": -3302, "maxx": 3048, "minz": -3302, "maxz": 3048,
  "lo": 0, "hi": 247.84,
  "img": {
    "BASE": "data:image/webp;base64,...",
    "TOPO": "...", "SLOPE": "...", "ELEV": "...", "ROAD": "...",
    "WALL": "...", "OBST": "...", "WATER": "...", "BRIDGE": "...",
    "BOUND": "...", "CLEAR": "...", "MBLOCK": "...", "BOAT": "...", "TREE": "..."
  },
  "flag_db": { "Yehorivka_AAS_v1": { "...": [x, y] } },
  "flag_cn": { "Bilovia": "比洛维亚" }
}
```

- `minx/maxx/minz/maxz`：这张图的 Squad 世界坐标范围（米），`lo/hi` 是高程区间。
- `img`：14 张 1024×1024 的 webp，内嵌成 data URL，App 直接当图片用。
  BASE=底图、TOPO=地形、SLOPE=坡度、ELEV=高程、ROAD=道路、WALL=围墙、OBST=障碍、
  WATER=水域、BRIDGE=桥、BOUND=地图边界、CLEAR=手动通畅、MBLOCK=手动障碍、
  BOAT=舰艇航道、TREE=树冠。
- `flag_db`：按图层（AAS / RAAS / Invasion …）分组的旗点像素坐标；`flag_cn`：地名中文对照。

## 怎么更新（数据维护者）

1. 在标注器 `map-water-bridge-annotator.exe` 里改好水域 / 桥梁 / 边界 / 道路等图层并保存；
2. 在项目根目录跑 `python tools/build_maps.py`，重新生成 `dist/maps-data/*.json` 与清单；
3. 更新本仓库：`git add -A && git commit -m "地图数据更新" && git push`；
   也可以直接用标注器工具栏的「上传数据到 GitHub」按钮，它会自动比对、只传有变化的文件；
4. 玩家在 App 里点「检查更新 → 下载有更新的」即可。

## 注意

- 仓库只放数据，不放代码。地图底图与 3D 物件实例来自 SquadCalc 的公开数据。
- 中文一律 UTF-8 存储；`.gitattributes` 设了 `* -text`，避免换行被自动改写导致 sha256 变化。
