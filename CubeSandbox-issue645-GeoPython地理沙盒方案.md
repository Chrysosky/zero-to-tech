# CubeSandbox Issue #645：GeoPython 地理数据处理沙盒调研与可行性方案

> 结论版本：2026-07-27  
> 调研基线：CubeSandbox `master` @ [`03c543d`](https://github.com/TencentCloud/CubeSandbox/commit/03c543d3c3c6b09dcb03296cbf8827d788469ace)，最新正式版 [`v0.6.0`](https://github.com/TencentCloud/CubeSandbox/releases/tag/v0.6.0)  
> 用途：用于决定是否认领、如何收敛首个 PR，以及提交前需要完成哪些验证  
> 说明：本文区分“上游已确认事实”“方案建议”和“尚待真实环境验证的假设”。

## 1. 结论先行

### 1.1 总体判断

**GeoPython 方向与 Issue #645 的目标匹配，技术上可行，但当前只能评为“有条件可行”，不能评为已验证可行。**

成立的理由：

- Issue 明确接收数据科学环境、代码解释器和特定场景模板；GeoPandas/Rasterio 模板属于未被现有示例充分覆盖的场景。[S1][S7]
- CubeSandbox 官方支持从 OCI 镜像创建模板，并提供 `cubesandbox-base` 与 `envd` 接入契约。[S3][S4]
- GeoPandas、Pyogrio、Rasterio 在 x86_64 Linux 上均提供二进制 wheel，首版可以避免自行编译 GDAL。[S11][S12][S13]
- 截至 2026-07-27，Issue 评论和公开 PR 中未发现明确的 GeoPython、GIS、GDAL 或 geospatial 重复方向。

不能直接判定“稳妥可交付”的原因：

- 认领截止到 **2026-07-31**，从本文日期起只剩 4 天。[S1]
- 维护者在相关模板 PR 中明确要求提供**真实 CubeSandbox 集群/实例**上的验证截图和原始日志；只有 Docker 构建或 CI 结果不足以证明可合入。[S9][S10]
- 当前本机 Docker CLI 可用，但 Docker Desktop Linux Engine 未运行；本文没有完成镜像构建和真实集群验证。
- 原方案的镜像入口、SDK 类型、Python 版本、GeoJSON CRS 和栅格内存模型存在实质性错误，不能按原文直接实施。

### 1.2 可行性评分

| 维度 | 判断 | 依据 |
|---|---|---|
| Issue 匹配度 | 高 | 满足“模板 + 可运行示例 + README”的最低门槛 |
| 技术实现 | 中高 | 官方已有 BYOI/envd 路径；GIS wheels 可用 |
| 方向重复风险 | 中低 | 暂未发现同方向，但大量参与者只写了笼统的“已认领” |
| 环境风险 | 高 | 必须有 Linux、KVM/PVM、XFS 和可访问 OCI Registry |
| 时间风险 | 高 | 距认领截止仅 4 天，且真实集群验证是事实上的 review 门槛 |
| 合入把握 | 中 | 取决于真实集群证据、文档复现性和是否及时明确认领方向 |

### 1.3 Go / No-Go 条件

满足以下条件后再进入实现：

1. 已报名 2026 犀牛鸟活动，并在 Issue 下明确回复认领 `GeoPython / GeoPandas + Rasterio` 方向。
2. 最迟在 2026-07-28 确认能使用真实 CubeSandbox v0.6.0 集群或可自行部署的 Linux 主机。
3. 主机满足官方功能体验门槛：至少 4 核、8 GB 内存、`/data/cubelet` 至少 50 GB 且为 XFS；PVM 路径还要求 x86_64 和 root。[S2]
4. 有一个 CubeMaster 节点可以拉取的 OCI Registry。
5. 接受首个 PR 只做最小模板与 1 个端到端示例，不加入快照、MCP、对象存储和长任务编排。

若无法获得真实集群验证条件，应把本方案视为技术预研，不应承诺在本次活动截止前完成可合入 PR。

## 2. Issue 与上游现状

### 2.1 Issue 的硬性要求

Issue #645 的单份贡献最低门槛是：[S1]

1. 至少一个可构建并能拉起沙箱的模板；
2. 至少一个可复现的运行示例；
3. 一份与当前版本行为一致的 README；
4. 其他开发者可以在合理时间内按文档跑通；
5. 在 `docs/guide/tutorials` 或 `examples` 体系中登记。

长时任务、快照断点续跑、有状态工作区、多容器和受限出口属于**鼓励项**，不是单份贡献的最低验收项。首个 PR 不应为了“体现差异化”而同时实现这些能力。

### 2.2 认领与重复情况

截至 2026-07-27：

- Issue 状态为 Open，共有 58 条评论。
- 评论中已有大量笼统认领，明确方向包括 Node、Java 等；未检索到 GeoPython/GIS/GDAL/geospatial 明确认领。
- 公开关联 PR 至少包括通用模板集合 PR #787 和 Ruby Sinatra 模板 PR #926，二者均未覆盖 GIS。[S9][S10]
- 仓库 `examples` 已有 pandas/matplotlib 代码解释器示例，但没有预装 GeoPandas/Rasterio 的 GIS 模板。[S7]

因此，**目前没有直接撞题证据，但仍须在认领评论中写清具体技术栈和交付范围**，不能只回复“已认领本任务”。

建议认领文字：

```text
已认领本任务：计划贡献 geospatial-python-sandbox，基于 CubeSandbox envd
提供 GeoPandas/Pyogrio/Rasterio 环境、一个端到端文件上传-处理-下载示例、
README 与真实 CubeSandbox 集群验证记录。当前公开示例中未发现同类 GIS 模板。
```

## 3. 对原方案的关键纠错

| 原方案表述或做法 | 调研结论 | 修正 |
|---|---|---|
| `python:3.11-slim` 直接作为模板镜像 | 不满足当前 Cube BYOI 契约；自定义镜像必须运行 `envd` | 从 `cubesandbox-base:2026.16` 继承，或按官方方式复制 `envd` 和入口脚本 [S3] |
| `CMD` 运行一次环境检查后退出 | `envd` 必须持续监听 `49983`，否则模板探针和 SDK 文件/命令接口失败 | 环境检查放到构建阶段；运行阶段由 Cube 入口维持 `envd` |
| 创建模板只传 `--image` 和可写层大小 | 当前流程还要求暴露并探测 envd | 增加 `--expose-port 49983 --probe 49983 --probe-path /health` [S3][S4] |
| 自定义 envd 镜像使用 `e2b_code_interpreter.Sandbox.run_code()` | `run_code()` 依赖代码解释器/Jupyter 能力；单有 envd 只保证 commands/files/init | 首版使用通用 `e2b.Sandbox`、`sandbox.commands.run()` 和 `sandbox.files.*` [S3][S8] |
| Python 3.11 安装最新 Rasterio | Rasterio 1.5.0 要求 Python >=3.12 | 首版选 Python 3.12，或明确固定旧版 Rasterio [S12] |
| 同时安装系统 GDAL 开发包和 PyPI GIS wheels | Pyogrio/Rasterio wheels 自带 GDAL；混用系统 GDAL 容易造成 ABI、驱动和数据目录不一致 | 首版只用 wheels；需要额外驱动时整体切换到 conda-forge，不混装 [S11][S13] |
| 把 EPSG:3857 结果写为 GeoJSON | RFC 7946 规定 GeoJSON 使用 WGS 84、十进制度，并移除了 `crs` 成员 | 任意 CRS 输出改用 GeoPackage；GeoJSON 输出只允许 EPSG:4326 [S14] |
| “栅格窗口统计”调用 `dataset.read(1)` | 实际读取整幅第一波段，内存占用随栅格大小线性增长 | 使用 `block_windows(1)` 或显式 `Window` 分块聚合 |
| Shapefile 作为首版普通单文件输入 | Shapefile 是多文件集合，上传、编码、缺件和路径校验都更复杂 | 首版不支持 Shapefile；使用 GeoJSON 和 GeoPackage |
| 默认拒绝沙箱公网出口 | Cube 示例通过 `allow_internet_access=False` 显式关闭，不应写成平台默认行为 | 在 SDK 创建沙箱时显式设置并测试 [S6] |
| `e2b_000000` 可视为鉴权 | 默认部署若未配置 auth callback，会接受所有请求；示例 key 不是生产安全边界 | 仅限本地评估，生产环境必须配置鉴权回调、私网绑定或防火墙 [S5] |
| 4 核/8 GB 是 GeoPython 沙箱资源建议 | 这是 CubeSandbox 主机的功能体验配置，不是单沙箱 GIS 基准 | 分开写“集群主机要求”和“工作负载实测结果” |
| ARM64 不支持 CubeSandbox | 当前 CubeSandbox 可在原生 KVM 的 aarch64 裸金属/物理机上部署；只有 PVM 快速路径是 x86_64-only | 首版仍限定 x86_64，ARM64 标为未验证 [S2] |

## 4. 推荐的首个 PR 范围

### 4.1 必须交付

建议名称：`geospatial-python-sandbox`。

首个 PR 只包含：

- 一个遵循 Cube `envd` 契约的 OCI 镜像；
- 固定并可审计的 Python/GIS 依赖锁文件；
- 一个端到端 SDK 示例：创建沙箱、上传数据、执行固定脚本、下载结果、校验结果、关闭沙箱；
- 两个小而明确的处理能力：
  - `reproject`：GeoJSON（按 RFC 7946 视为 EPSG:4326）转换为指定 CRS 的 GeoPackage；
  - `raster-statistics`：单波段 GeoTIFF 的分块 count/min/max/mean；
- 单元测试、镜像 smoke test、真实集群验证脚本；
- 英文 `README.md` 与中文 `README_zh.md`；
- examples 索引登记。

### 4.2 明确不做

首个 PR 不包含：

- 任意用户 Python/Shell 代码执行封装；
- MCP 服务、GIS Agent 或 LLM 调用；
- Shapefile、PostGIS、云优化 GeoTIFF 写出、三维或在线地图；
- 快照断点续跑和长时任务状态机；
- 对象存储凭据注入；
- ARM64、多节点和高并发性能承诺。

这些能力与 Issue 最低门槛无关，会显著增加测试面和 review 成本。

## 5. 推荐架构

```text
本地 demo.py
  ├─ e2b.Sandbox.create(template=..., allow_internet_access=False)
  ├─ sandbox.files.write() 上传输入
  ├─ sandbox.commands.run() 调用固定 CLI
  ├─ sandbox.files.read() 下载输出
  └─ 校验结果并关闭沙箱
                 |
                 v
CubeSandbox MicroVM
  ├─ envd :49983（探针、命令、文件、初始化）
  ├─ Python 3.12
  ├─ GeoPandas + Pyogrio（矢量）
  ├─ Rasterio（栅格）
  ├─ /workspace/input（只读语义）
  └─ /workspace/output（结果）
```

这里选择通用 `e2b` SDK，而不是 `e2b-code-interpreter`。后者适合带 Jupyter 内核的代码解释器模板；本方案只需要 envd 的命令和文件接口，依赖更少、契约更清晰。[S3][S8]

## 6. 镜像与依赖设计

### 6.1 基础镜像路线

推荐使用官方文档中的“向 Python slim 镜像注入 envd”路线：[S3]

```dockerfile
# syntax=docker/dockerfile:1.7
ARG PYTHON_IMAGE=python:3.12-slim-bookworm
ARG CUBE_BASE_IMAGE=ghcr.io/tencentcloud/cubesandbox-base:2026.16

FROM ${CUBE_BASE_IMAGE} AS cube
FROM ${PYTHON_IMAGE}

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

COPY --from=cube /usr/bin/envd /usr/bin/envd
COPY --from=cube /usr/local/bin/cube-entrypoint.sh /usr/local/bin/cube-entrypoint.sh

RUN useradd --create-home --uid 1000 sandbox \
    && mkdir -p /workspace/input /workspace/output \
    && chown -R sandbox:sandbox /workspace

WORKDIR /workspace
COPY requirements.lock /workspace/requirements.lock
RUN python -m pip install --require-hashes -r /workspace/requirements.lock

COPY geosandbox /workspace/geosandbox
COPY scripts/check_environment.py /workspace/scripts/check_environment.py
COPY tests /workspace/tests
RUN python /workspace/scripts/check_environment.py

EXPOSE 49983
ENTRYPOINT ["/usr/local/bin/cube-entrypoint.sh"]
CMD ["sleep", "infinity"]
```

注意：

- 上述是待构建验证的设计骨架，不是已经验证过的最终 Dockerfile。
- 正式提交应把 `python:3.12-slim-bookworm` 固定到目标架构的 digest，避免 tag 漂移。
- 当前官方 base 的 `envd` 由 root 启动；地理处理命令应通过 SDK 的 `user` 参数以 `sandbox` 用户运行。不要未经真实模板验证就把整个容器切换为 `USER sandbox`。
- `check_environment.py` 在构建阶段运行，不能作为会立即退出的容器主进程。

模板创建命令应为：

```bash
cubemastercli tpl create-from-image \
  --image <registry>/geospatial-python-sandbox:<immutable-tag> \
  --writable-layer-size 2G \
  --expose-port 49983 \
  --probe 49983 \
  --probe-path /health

cubemastercli tpl watch --job-id <job-id>
```

`2G` 是首轮验证起点，不是经过基准测试的推荐上限；应根据镜像展开大小、样例输入和输出峰值调整。

### 6.2 Python 依赖策略

截至 2026-07-27，PyPI 的相关最新版本包括 [GeoPandas 1.1.4](https://pypi.org/project/geopandas/1.1.4/)、[Pyogrio 0.13.0](https://pypi.org/project/pyogrio/0.13.0/) 和 [Rasterio 1.5.0](https://pypi.org/project/rasterio/1.5.0/)；其中 Rasterio 1.5.0 要求 Python >=3.12。[S11][S12][S13]

建议流程：

1. 在 `requirements.in` 中只声明直接依赖，如 GeoPandas、Rasterio、pytest。
2. 在 Linux x86_64/Python 3.12 环境生成带 hash 的 `requirements.lock`。
3. Docker 构建使用 `pip --require-hashes`，禁止静默回退到源码编译。
4. `check_environment.py` 输出 Python、GeoPandas、Pyogrio、Rasterio、GDAL、PROJ 和 GEOS 实际版本。
5. smoke test 验证 `GeoJSON`、`GPKG` 和 `GTiff` 驱动确实可用，不能只验证 import。

首版不要安装 `gdal-bin`、`libgdal-dev`、`libgeos-dev` 和 `libproj-dev`。Pyogrio 与 Rasterio 的 PyPI wheels 已携带所需本地库；若后续需要 wheels 未包含的驱动，应改为一套完整的 conda-forge 环境并重新评估镜像体积。[S11][S13]

## 7. 地理处理正确性

### 7.1 矢量重投影

输入与输出契约建议为：

| 项目 | 约束 |
|---|---|
| 输入 | RFC 7946 GeoJSON，按 EPSG:4326 解释 |
| 参数 | 可由 `pyproj.CRS.from_user_input()` 解析的目标 CRS |
| 输出 | GeoPackage（`.gpkg`），保留目标 CRS |
| 拒绝 | 空数据、无几何、非有限坐标、无法解析的 CRS、输出越界路径 |

若用户要求 GeoJSON 输出，则目标 CRS 必须限制为 EPSG:4326。不要把 Web Mercator 坐标写入标准 GeoJSON。[S14]

### 7.2 栅格统计

首版只支持单个指定波段，默认第 1 波段。实现必须：

- 优先遍历 `dataset.block_windows(band)`；若源数据的单个 block 超过设定内存预算，则进一步切成固定上限的 `Window`，不一次性读取整幅栅格；
- 使用 masked read 排除 nodata；
- 对浮点栅格同时排除 NaN 和无穷值；
- 累计有效像元数、最小值、最大值和 float64 sum，最后计算 mean；
- 输出 `band`、`dtype`、`shape`、`crs`、`nodata` 和统计值；
- 对“没有有效像元”返回明确的非零退出码和结构化错误。

这能把内存复杂度从 `O(整幅栅格像元数)` 降为 `O(受控窗口大小)`。但总执行时间仍与像元数线性相关，必须通过真实样本测量，不能预先承诺“大文件支持”。

### 7.3 为什么首版不选 clip、buffer 和 Shapefile

- `clip` 需要处理 mask CRS、无效几何、空间索引和空结果，测试面大于重投影。
- `buffer` 的距离单位依赖投影 CRS；对经纬度直接缓冲通常没有可靠业务含义。
- Shapefile 是多个关联文件，涉及归档解压、文件名、编码、缺件与路径穿越。

这些功能可以在模板基础能力合入后单独增加，不应阻塞首个可复现示例。

## 8. SDK 调用边界

端到端示例应使用仓库现有的通用模式：[S6][S8]

```python
from e2b import Sandbox

with Sandbox.create(
    template=template_id,
    allow_internet_access=False,
) as sandbox:
    # 1. files.write 上传测试输入
    # 2. commands.run 以 sandbox 用户执行固定模块和固定路径
    # 3. 检查 exit_code、stdout、stderr
    # 4. files.read 下载二进制输出
    # 5. 在宿主侧重新打开结果并断言 CRS/统计值
    pass
```

正式代码应以锁定的 `e2b` 版本为准核对二进制读写参数，不要凭示例猜测 SDK 签名。当前仓库的 `cubesandbox-base-nginx` 示例使用 `e2b>=2.4.1`。[S8]

首版不提供“用户传入任意命令”的接口。demo 中执行的模块名、输入和输出根目录由程序固定；可变参数通过严格解析后传入。若以后提供 JSON 任务接口，再单独设计 schema、错误码与兼容策略。

## 9. 安全边界

### 9.1 沙箱内

- 处理进程使用非 root 用户，envd 保持其所需权限。
- 输入只从 `/workspace/input` 读取，输出只写入 `/workspace/output`。
- 对路径执行 `resolve()` 后再检查其是否位于允许根目录内；仅拒绝字符串中的 `../` 不足以防止符号链接逃逸。
- 不解压用户归档，不支持 VRT 和远程 `/vsicurl/` 数据源，避免间接联网和引用外部文件。
- 使用驱动识别和实际打开结果校验格式，不只检查扩展名。
- 创建沙箱时显式设置 `allow_internet_access=False`，并在真实实例中验证公网访问失败、GIS 处理仍成功。[S6]
- 日志记录脚本版本、输入 SHA-256、参数、依赖版本、耗时、退出码和错误摘要，不记录输入数据内容或密钥。

### 9.2 CubeSandbox 部署侧

CubeSandbox 一键部署面向开发和评估，多个管理端口默认绑定 `0.0.0.0`，部分管理接口默认无认证或 TLS。`E2B_API_KEY=e2b_000000` 只适用于本地示例。[S5]

真实集群至少应：

- 不把 CubeMaster `8089`、Cubelet `9998/9999`、CubeAPI `3000` 暴露给不可信公网；
- 使用私网绑定和防火墙白名单；
- 非本地评估环境配置 auth callback，并同时校验请求 path 与 method；
- 不把 Registry、对象存储或模型的长期密钥写入镜像层。

## 10. 推荐目录结构

最终目录应服从仓库当时的 examples 风格。建议起点：

```text
examples/geospatial-python-sandbox/
├── Dockerfile
├── .dockerignore
├── .env.example
├── requirements.in
├── requirements.lock
├── README.md
├── README_zh.md
├── demo.py
├── geosandbox/
│   ├── __init__.py
│   ├── cli.py
│   ├── paths.py
│   ├── reproject.py
│   └── raster_statistics.py
├── scripts/
│   ├── check_environment.py
│   └── generate_sample_data.py
└── tests/
    ├── test_paths.py
    ├── test_reproject.py
    └── test_raster_statistics.py
```

不要提交大型二进制样例。由固定随机种子和明确 CRS 的脚本生成小型 GeoJSON/GeoTIFF，既减少仓库体积，也便于断言精确结果。

## 11. 验证与验收证据

### 11.1 本地镜像验证

必须保存完整命令和日志：

```bash
docker build --pull -t <registry>/geospatial-python-sandbox:<tag> .
docker run --rm -d -p 49983:49983 --name geo-sandbox <image>
docker exec geo-sandbox /usr/bin/envd -version
docker exec geo-sandbox python /workspace/scripts/check_environment.py
curl -o /dev/null -w '%{http_code}\n' http://127.0.0.1:49983/health
docker exec geo-sandbox python -m pytest -q /workspace/tests
docker rm -f geo-sandbox
```

预期至少证明：

- envd `/health` 返回 204；
- 镜像进程不会立即退出；
- 所有依赖可导入且目标驱动可用；
- 重投影输出 CRS 与几何符合预期；
- 栅格统计忽略 nodata/NaN，并在多 block 数据上得到正确结果；
- 路径越界、符号链接逃逸、非法 CRS 和空数据被拒绝。

### 11.2 真实 CubeSandbox 验证

这是提交前阻断项，不是可选增强：[S9][S10]

| 步骤 | 必须记录的证据 |
|---|---|
| 集群环境 | CubeSandbox 版本、OS、架构、CPU/内存、KVM/PVM、XFS 挂载 |
| 模板创建 | 完整命令、job ID、template ID、`READY` 状态和原始日志 |
| 沙箱启动 | SDK 版本、创建耗时、sandbox ID（敏感部分可脱敏） |
| 文件流程 | 上传成功、远端文件摘要、下载后 SHA-256 或语义校验 |
| 矢量任务 | 输入/输出 CRS、feature 数、处理耗时、退出码 |
| 栅格任务 | shape、block 数、有效像元数、统计结果、峰值 RSS |
| 网络限制 | 公网请求失败，离线 GIS 任务仍通过 |
| 清理 | 沙箱正常关闭，失败路径也能清理 |

维护者已在相关 PR 中同时要求“截图和日志”，因此两种证据都应提供。截图展示关键结果，原始文本日志放到可审计位置并标明对应 commit、镜像 digest 和模板 ID。

### 11.3 不应提前填写的资源结论

README 可以引用 CubeSandbox 主机最低配置，但 GeoPython 工作负载的资源建议必须来自实测。建议至少测量：

- 小型矢量：1 千、10 万个简单要素；
- 小型栅格：256×256；
- 中型栅格：4096×4096；
- 每组记录执行时间、峰值 RSS、输入/输出大小和可写层增量。

在得到数据前，只能写“已验证样本范围”，不能写“支持大文件”“适合某并发规模”或给出无来源的内存上限。

## 12. 实施顺序与停止条件

1. **认领与环境确认**：明确回复方向，确认真实集群和 Registry。
2. **最小镜像**：先只安装 Python 与 GIS 依赖，验证 envd 204 和模板 READY。
3. **矢量闭环**：完成 GeoJSON -> GeoPackage 重投影及宿主侧断言。
4. **栅格闭环**：完成分块统计；若时间不足，可从首个 PR 移除，不能提交未验证实现。
5. **安全与失败路径**：非 root、路径边界、离线运行、错误码。
6. **真实集群证据**：在与提交 commit 一致的镜像上重跑并归档日志/截图。
7. **双语文档和索引**：最后按实际行为撰写，不先写不存在的能力。

停止条件：

- 2026-07-29 前仍无法使用真实 CubeSandbox 集群；
- envd 探针或模板构建无法稳定达到 READY；
- GIS wheels 在目标镜像中发生不可解释的 GDAL/PROJ 冲突；
- 已有人提交并验证了高度重合的 GIS 模板，维护者不接受并行方案。

## 13. 风险登记

| 风险 | 概率 | 影响 | 缓解措施 |
|---|---:|---:|---|
| 无真实集群证据 | 高 | 高 | 实现前先锁定环境；没有环境则不承诺活动期合入 |
| 认领方向不明确或撞题 | 中 | 高 | 评论写明 GeoPandas/Rasterio、交付物和范围 |
| envd/入口配置错误 | 中 | 高 | 直接复用官方 `2026.16` 入口并验证 49983/health |
| GIS wheel/本地库冲突 | 中 | 高 | wheels-only；禁止系统 GDAL 混装；记录实际版本 |
| 镜像 tag 漂移 | 中 | 中 | 基础镜像 pin digest，输出最终镜像 digest |
| 大栅格内存耗尽 | 中 | 高 | block/window 流式处理、输入限制、峰值 RSS 实测 |
| 格式语义错误 | 中 | 高 | GeoJSON 仅 WGS84；任意 CRS 使用 GeoPackage |
| 路径或外部数据引用绕过 | 中 | 高 | resolved-path 检查，禁归档/VRT/远程 VSI，非 root |
| 文档与当前 SDK 不一致 | 中 | 高 | 固定 SDK 版本，按该版本重跑 README 全流程 |
| 控制面暴露 | 中 | 高 | 私网、防火墙、auth callback、生产证书 |

## 14. 最终建议

**推荐继续，但必须先解决真实集群访问，再开始扩展功能。** 技术路线应以 `envd + 通用 e2b SDK + wheels-only GIS 环境` 为核心；首个 PR 只证明“模板能构建、沙箱能启动、文件能进出、地理处理正确、离线仍可运行”。

最有说服力的差异化不是堆叠快照、MCP 或 Agent，而是提供一个当前仓库没有、且别人能按 README 在真实 CubeSandbox 中复现的 GIS 模板。若四天窗口内资源有限，优先保证一个重投影闭环和完整验证证据，栅格统计可以作为第二个增量 PR。

## 15. 资料来源

以下资料均于 2026-07-27 核验。CubeSandbox 文档基线为 `master` commit `03c543d3c3c6b09dcb03296cbf8827d788469ace`。

- [S1 Issue #645：CubeSandbox 沙箱模板与示例生态](https://github.com/TencentCloud/CubeSandbox/issues/645)：任务范围、认领规则、最低验收。
- [S2 Quick Start](https://github.com/TencentCloud/CubeSandbox/blob/master/docs/guide/quickstart.md)：主机架构、PVM/KVM、glibc、XFS、磁盘和功能体验配置。
- [S3 Bring Your Own Image (envd)](https://github.com/TencentCloud/CubeSandbox/blob/master/docs/guide/tutorials/bring-your-own-image.md)：envd 端口、接口、基础镜像和入口契约。
- [S4 Creating Templates from OCI Images](https://github.com/TencentCloud/CubeSandbox/blob/master/docs/guide/tutorials/template-from-image.md)：模板创建参数、探针、状态和 Registry 要求。
- [S5 Network Hardening](https://github.com/TencentCloud/CubeSandbox/blob/master/docs/guide/network-hardening.md)：默认监听面、默认鉴权状态、auth callback 和防火墙建议。
- [S6 network_no_internet.py](https://github.com/TencentCloud/CubeSandbox/blob/master/examples/code-sandbox-quickstart/network_no_internet.py)：`allow_internet_access=False` 的官方示例。
- [S7 Examples index](https://github.com/TencentCloud/CubeSandbox/blob/master/docs/guide/tutorials/examples.md)：现有示例范围与登记方式。
- [S8 cubesandbox-base-nginx](https://github.com/TencentCloud/CubeSandbox/tree/master/examples/cubesandbox-base-nginx)：通用 `e2b.Sandbox`、envd 镜像和文件接口示例。
- [S9 PR #787](https://github.com/TencentCloud/CubeSandbox/pull/787)：通用模板集合及维护者对真实验证截图/日志的要求。
- [S10 PR #926](https://github.com/TencentCloud/CubeSandbox/pull/926)：模板贡献的真实集群验证证据与 review 反馈。
- [S11 GeoPandas installation](https://geopandas.org/en/stable/getting_started/install.html)：GIS 原生依赖与安装策略。
- [S12 Rasterio installation](https://rasterio.readthedocs.io/en/stable/installation.html)：Python/GDAL 要求与 wheels 说明。
- [S13 Pyogrio installation](https://pyogrio.readthedocs.io/en/latest/install.html)：wheels 自带 GDAL、驱动范围和系统 GDAL 差异。
- [S14 RFC 7946](https://www.rfc-editor.org/rfc/rfc7946.html)：GeoJSON 的 WGS 84、坐标顺序和 CRS 规则。
