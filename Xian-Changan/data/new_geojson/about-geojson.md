# GeoJSON 文件说明

## 坐标系统

所有 13 个文件均使用 **WGS84 经纬度坐标**（已从原始 SVG 自定义坐标配准）。

| 轴 | 范围 | 说明 |
|---|---|---|
| lng（经度） | 108.883 ~ 108.990 | 东经 |
| lat（纬度） | 34.206 ~ 34.283 | 北纬 |

### 配准方法

使用 **轴对齐仿射变换**（无旋转/剪切，仅缩放+平移），以三个今日尚存、原址未移的唐代城门遗址为控制点：

| 城门 | SVG 中心 (x, y) | WGS84 (lng, lat) | 配准误差 |
|---|---|---|---|
| 含光门 | (4177.5, 3273.0) | (108.927645, 34.253239) | ~(42m, 49m) |
| 丹凤门 | (7102.5, 0.0) | (108.959490, 34.282863) | ~(13m, -30m) |
| 明德门 | (4844.5, 8637.0) | (108.935882, 34.206458) | ~(-55m, -18m) |

变换公式：

```
lng = 1.07771e-5 × svg_x + 108.88308
lat = -8.83396e-6 × svg_y + 34.28259
```

配准误差主要来源于 SVG 地图的示意性绘制（非实测地形图），~50m 级别误差对于唐代长安城遗址分布图而言在可接受范围内。

---

## 通用属性约定

每个 Feature 的 `properties` 中可能包含以下通用字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| `css` | `string[]` | 原始 SVG 的 class 列表，**保留所有语义**。前端可用此恢复渲染样式 |
| `name` | `string?` | 对应 `data-name` 属性（里坊名、城门名、街名等） |
| `description` | `string?` | 对应子元素 `<title>` 文本，通常是中文描述或通行规则 |
| `_elementType` | `string` | 原始 SVG 标签类型：`rect` / `polygon` / `path` / `line` / `text` |
| `_group` | `string?` | 子群组上下文，如 `imperial-office-layer`、`palace-subdivision-layer` |
| `text` | `string?` | 仅 `<text>` 元素，表示标签的显示文字（含 `<tspan>` 拼接） |

---

## 各文件详解

### 1. 道路底纹.geojson

**来源层**: `<g class="layer road-zones">`

地图最底层的道路区域底纹，用于在视觉上标记环城街和禁行区的范围（实际道路线在 街道基底 层）。

| 字段 | 说明 |
|---|---|
| `css` | 含 `road-zone` + 区域类型标记：`public-ring-zone-base`（居民环城街底纹）、`restricted-road-zone-base`（禁行区底纹） |
| `name` | 路段名称，如"西侧居民环城街""宫城北侧禁行带""东侧夹城御道" |
| `description` | 中文通行说明，如"墙内居民街，普通人可通行；与夹城御道分层" |

**几何**: `Polygon` — 均为矩形道路区域。

---

### 2. 街道基底.geojson

**来源层**: `<g class="layer streets-base">`

唐长安城全部公共街道的精确路面范围。这是导航路网的视觉基底。

| 字段 | 说明 |
|---|---|
| `css` | 恒为 `["street-band"]` |
| `name` | 街道名称 + 类型标记，如"朱雀大街（明德门—朱雀门街）""金光门—春明门街""西市西侧周街" |
| `description` | `街道名 \| 类型标签`，如 `public_priority`（主干道）、`public_local`（普通街道）、`public_inner_wall_resident_street`（环城居民街）、`restricted_imperial_only`（禁行道）、`public_market_perimeter`（市场周街） |

**几何**: `Polygon`

**街道分类（根据 description 中的类型标签）**:

| 类型标签 | 含义 | 数量 |
|---|---|---|
| `public_priority` | 主干道，车马可行 | 6 条 |
| `public_local` | 普通公共街道 | 约 20 条 |
| `public_inner_wall_resident_street` | 墙内居民环城街 | 6 段 |
| `public_market_perimeter` | 东西市周街 | 8 段 |
| `restricted_imperial_only` | 夹城御道/宫禁道路，禁行 | 3 段 |

---

### 3. 里坊.geojson

**来源层**: `<g class="layer wards">`

长安城 108 坊（含大安国寺、入苑等特殊区块）。每个坊为一个矩形 Polygon。

| 字段 | 说明 |
|---|---|
| `css` | `["ward", "division-16"]` 或 `["ward", "division-12"]`，**区分坊制**：`division-16` = 四面坊门 + 十字街，`division-12` = 东西二门 + 东西横街 |
| `name` | 坊名，如"群贤""平康""朱雀" |
| `description` | 完整描述：`坊名 \| 区域 \| 坊制 \| 坊制说明`，如"平康 \| 南区 \| 16分 \| 16分：东西南北四门 + 对称十字街" |

**几何**: `Polygon` — 严格矩形，坐标来自 SVG `<rect>` 的 x/y/width/height。

---

### 4. 特殊地物.geojson

**来源层**: `<g class="layer features">`

宫城、皇城、市场、园林、水体、官署等非坊的特殊区域。结构最复杂的图层，包含 3 个子群组。

**总览**:

| css 类 | 含义 | 数量 |
|---|---|---|
| `feature-market` | 东市 / 西市 | 2 |
| `feature-palace` | 宫城、皇城、兴庆宫 | 3 |
| `feature-garden` | 芙蓉园 | 1 |
| `feature-water` | 曲江池 | 1 |
| `feature-wall` | 夹城隔墙 | 2 |
| `office-block` | 皇城官署（将作监、尚书省、太常寺等） | ~60+ |
| `office-label` | 官署名称文字标注 | ~40 |
| `imperial-road` | 皇城内纵横街巷（`LineString`） | ~15 |
| `palace-subzone` | 宫城分区：掖庭宫/太极宫/东宫（禁内） | 3 |
| `palace-zone-label` / `palace-note` | 宫城分区文字标注 | 6 |
| `restricted-zone` | 禁行区域标记 | 3 |
| `ritual` / `major` / `medium` / `small` / `large` | 官署重要性/大小标记 | — |

**子群组**（通过 `_group` 字段区分）:

| `_group` | 来源 | 说明 |
|---|---|---|
| `imperial-office-layer` | `<g class="imperial-office-layer">` | 皇城官署（146 个要素），含 office-block rect、office-label text、imperial-road line |
| `palace-subdivision-layer` | `<g class="palace-subdivision-layer">` | 宫城分区（9 个要素），含 palace-subzone rect 和 zone-label/note text |
| 无 `_group` | features 的直接子元素 | 西市/东市、兴庆宫、皇城、宫城、芙蓉园、曲江池、隔墙 |

**几何**: 混合类型 — 大部分 `Polygon`（rect/polygon），部分 `LineString`（imperial-road line），部分 `Point`（label text）。

---

### 5. 结构线.geojson

**来源层**: `<g class="layer structural-lines">`

长安城的结构性轮廓线：城墙、宫城/皇城边界、隔墙。

| css 类 | 说明 | 几何 |
|---|---|---|
| `outer-wall` | 外郭城墙，包含南墙芙蓉园断开段 | `MultiLineString`（2 段） |
| `palace-outline` | 宫城/皇城/兴庆宫轮廓 | `Polygon`（矩形） |
| `separator-wall` | 北侧/东侧夹城隔墙 | `Polygon`（细长矩形） |

**注意**: `outer-wall` 的 path 使用 `M`/`V`/`H` 命令，已正确解析为多段线。

---

### 6. 通行道覆盖.geojson

**来源层**: `<g class="layer access-overlays">`

半透明着色覆盖层，在视觉上区分不同通行等级的路段。（数据与 街道基底 重叠，但着色不同。）

| css 类 | 含义 |
|---|---|
| `priority-street-overlay` | 主干道覆盖色（红色调） |
| `public-ring-overlay` | 居民环城街覆盖色（青色调） |
| `restricted-street-overlay` | 禁行区覆盖色（紫色调） |

**几何**: `Polygon`

---

### 7. 坊内街道.geojson

**来源层**: `<g class="layer internal-streets">`

每个里坊内部的十字街或东西横街，连接坊门与坊内区域。导航时作为坊内通路使用。

| 字段 | 说明 |
|---|---|
| `css` | `["internal-street", "ward-internal-street"]`（坊内十字街）或 `["internal-street", "market-internal-street"]`（市场内街） |
| `name` | `坊名 \| 坊内街道`，如"群贤 坊内街道" |
| `description` | 含方向标记 `EW`（东西横街）或 `NS`（南北街），如"群贤 \| 坊内街道 \| EW \| 坊内东西横街，连接西坊门与东坊门。" |

**几何**: `Polygon` — 细长矩形。

---

### 8. 坊门.geojson

**来源层**: `<g class="layer ward-gates">`

每个里坊四面的坊门（东/西/南/北）。导航时坊门是连接坊内街道与外部公共路网的关键节点。

| 字段 | 说明 |
|---|---|
| `css` | `["ward-gate"]` 或 `["market-gate"]`（市场入口） |
| `name` | `坊名 \| 方向坊门`，如"群贤 西坊门""平康 南坊门" |
| `description` | 位置说明，如"群贤 \| 西坊门 \| 位于西侧坊墙正中；连接坊内东西横街。" |

**几何**: `Polygon` — 小矩形（宽度 34-54）。

**坊门命名规律**: 名称以空格分隔，前半为坊名，后半为方位+坊门，如 `"平康 南坊门"`。

---

### 9. 城门宫门.geojson

**来源层**: `<g class="layer gates">`

长安城的外郭城门、皇城门、宫城门。分为公众城门和禁门。

| css 类 | 说明 |
|---|---|
| `["gate", "gate-public"]` | 公众城门，所有人可通行。明德门、安化门、金光门、春明门等 |
| `["gate", "gate-restricted"]` | 宫门/禁门，普通人不通行。朱雀门、含光门、承天门、丹凤门等 |

**几何**: `Polygon` — 小矩形。

**城门列表**:

| 类型 | 城门 |
|---|---|
| 外郭城门（public） | 明德门、安化门、启夏门、金光门、延平门、开远门、春明门、延兴门、通化门、光化门、景耀门、芳林门、兴安门 |
| 宫门/禁门（restricted） | 丹凤门、朱雀门、含光门、安上门、安福门、延喜门、兴庆门、金明门、顺义门、景风门、承天门、太极门、朱明门、玄武门、建福门、望仙门 |

---

### 10. 文字标注.geojson

**来源层**: `<g class="layer labels">`

地图上所有文字标注，包括坊名、城门名、街名、地标名。每个标注为一个 Point，文字内容存储在 `properties.text` 中。

| css 类 | 类型 | 数量 | 说明 |
|---|---|---|---|
| `["label", "ward-label"]` | 坊名 | 111 | 里坊名称，如"群贤""平康" |
| `["label", "street-label"]` | 街名 | 38 | 街道名称，如"朱雀大街""金光门街" |
| `["label", "gate-label"]` | 城门名 | 30 | 城门/宫门名称，如"明德门""承天门" |
| `["label"]` | 其他标注 | 8 | 特殊标注 |
| `["label", "feature-label"]` | 地标名 | 4 | 如"西市""东市""兴庆宫" |

| 字段 | 说明 |
|---|---|
| `name` | 标注对象的名称，与 `text` 通常一致 |
| `text` | 显示文字内容 |
| `css` | 标注类型，前端可用此选择字号/字重 |

**几何**: `Point`

---

### 11. 图框组件.geojson

**来源层**: `<g class="map-furniture">`

地图的装饰性 UI 组件：图例、比例尺、指北针。主要用于视觉参考，导航逻辑不依赖此层。

| css 类 | 说明 |
|---|---|
| `legend-*` | 图例各条目（legend-box 背景、legend-title 标题、legend-priority/public-ring/restricted/internal/road/park/gate 色块+文字） |
| `scale-line` / `scale-tick` | 比例尺线段 |
| `north-line` / `north-arrow` / `north-label` | 指北针 |
| `furniture-label` | 比例尺数字标注 |

**几何**: 混合 — `Polygon`（图例背景/色块）、`LineString`（比例尺/指北针线）、`Polygon`（指北针箭头）、`Point`（文字）。

---

### 12. 路网节点.geojson

**来源**: `route_data.js` 中的 `ROUTE_DATA.nodes`

导航路网的图节点。每个节点代表一个路口或路网关键点（道路交叉口、坊门连接点、城门连接点等）。

| 字段 | 类型 | 说明 |
|---|---|---|
| `index` | `int` | 节点索引（与 路网边 的 `from`/`to` 对应） |
| `x_svg` | `float` | SVG x 坐标（原始像素，配准前的值） |
| `y_svg` | `float` | SVG y 坐标（原始像素，配准前的值，y 向下递增） |

**几何**: `Point` — `[lng, lat]`

**注意**: 原始 `route_data.js` 使用 y 向上递增的坐标系，提取时先用 `svgY = cityHeight - y` 翻转对齐到 SVG 坐标系，再经轴对齐仿射变换配准到 WGS84。`properties` 中的 `x_svg`/`y_svg` 保留配准前的像素值以供参考。

---

### 13. 路网边.geojson

**来源**: `route_data.js` 中的 `ROUTE_DATA.edges`

导航路网的图边（路段）。每条边连接两个节点，带有通行属性。

| 字段 | 类型 | 说明 |
|---|---|---|
| `from` | `int` | 起点节点索引 |
| `to` | `int` | 终点节点索引 |
| `distance_m` | `float` | 路段长度（米） |
| `name` | `string` | 路段所属街道名称 |
| `type` | `string` | 类型：`street`（普通路段）或 `connector`（门禁连接器，如皇城入口） |
| `class` | `string` | **通行等级**，导航引擎的核心分类 |

**几何**: `LineString` — `[[lng1, lat1], [lng2, lat2]]`

**通行等级（class）分布**:

| class | 数量 | 说明 | 步行 | 马车 | 牛车 | 骑马 |
|---|---|---|---|---|---|---|
| `ordinary_public_street` | 480 | 普通公共街道 | ✓ | ✗（需牵行） | ✗（需牵行） | ✓ |
| `connector` | 367 | 门禁连接器/皇城入口 | ✓ | ✓ | ✓ | ✓ |
| `primary_public_street` | 189 | 主干道（朱雀大街、金光门街等） | ✓ | ✓ | ✓ | ✓ |
| `public_inner_wall_street` | 123 | 墙内居民环城街 | ✓ | ✗ | ✗ | ✓ |
| `market_perimeter_public` | 44 | 东西市周街 | ✓ | ✓ | ✓ | ✓ |

**出行模式说明**（来自原始 HTML 的 `MODES` 定义）：

| 模式 | 速度 | 主干道限制 | 说明 |
|---|---|---|---|
| 步行 | 4.5 km/h | 无限制 | 全路网可通行 |
| 马车 | 17.5 km/h | 仅主干道 | 非主干路段需下车牵行（步行速度） |
| 牛车 | 4.5 km/h | 仅主干道 | 非主干路段需下车牵行 |
| 骑马 | 6 km/h | 无限制 | 城内慢行，不可纵马 |

---

## 文件关系图

```
路网节点.geojson + 路网边.geojson        ← 导航引擎核心数据（WGS84）
         │
         ▼
┌─────────────────────────────────┐
│         导航引擎                  │
│  (Dijkstra / A* + 出行模式过滤)   │
└─────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│        交互层                    │
│  搜索(里坊/城门/官署名称)         │
│  点击选择起终点                    │
└─────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│         渲染层（WGS84）           │
│                                 │
│  道路底纹     →  道路底纹.geojson  │
│  街道基底     →  街道基底.geojson  │
│  里坊         →  里坊.geojson     │
│  特殊地物     →  特殊地物.geojson  │
│  结构线       →  结构线.geojson    │
│  通行道覆盖   →  通行道覆盖.geojson │
│  坊内街道     →  坊内街道.geojson  │
│  坊门         →  坊门.geojson     │
│  城门宫门     →  城门宫门.geojson  │
│  文字标注     →  文字标注.geojson  │
│  图框组件     →  图框组件.geojson  │
└─────────────────────────────────┘
```

渲染时推荐按此顺序叠放：道路底纹 → 街道基底 → 通行道覆盖 → 里坊 → 结构线 → 特殊地物 → 坊内街道 → 坊门 → 城门宫门 → 文字标注 → 图框组件 → 路线层（动态绘制）→ 标记层（动态绘制）。
