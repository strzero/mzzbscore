# MZZB Score - 动画评分相关数据提取工具

## 项目介绍

MZZB Score 是一个动画评分聚合工具，能够自动从多个知名动画评分网站（Bangumi、MyAnimeList、AniList、Filmarks）获取动画作品的评分数据，并集成Twitter粉丝数据获取功能，将这些数据整合到Excel表格中，方便用户进行比较和分析。

## 安装步骤

### 方法一：从源码运行

1. 克隆或下载本仓库
2. 安装依赖项：
   ```
   pip install -r requirements.txt
   ```

3. 填写好同目录下的`mzzb.xlsx`Excel文件
4. 运行主程序：
   ```
   python main.py
   ```

### 方法二：使用打包的可执行文件

1. 下载Releases发布页面提供的最新的`mzzb_score.exe`文件
2. 将`mzzb_score.exe`文件与填写好的`mzzb.xlsx`表格放在同一目录下
3. 双击运行`mzzb_score.exe`文件

## 使用方法

1. 准备一个Excel文件（默认名称为`mzzb.xlsx`），格式如下：
   - A1单元格：当前年份（如"2025年XXXXXXXXXXX"）（一定要改）
   - 从第二行开始，包含一个"原名"列，填入需要查询的动画名称
   - **可选**：预填各平台URL列（`Bangumi_url`、`Anilist_url`、`Myanimelist_url`、`Filmarks_url`），程序将优先使用这些链接直接提取数据
2. 运行程序，程序会自动执行以下流程：
   - 自动检测和配置网络代理
   - 配置Twitter功能（可选，支持跳过）
   - 检查每行是否已有平台链接
   - 对于有链接的平台，直接从链接提取数据（快速准确）
   - 对于没有链接的平台，通过名称搜索获取数据（兜底保障）
   - 并发获取所有平台数据
   - 执行交叉验证和评分标准化
   - 获取Twitter粉丝数据
   - 验证日期一致性
3. 程序会自动将获取到的数据更新到Excel表格中

## 运行流程

1. **程序启动**：启动日志系统，显示欢迎信息
2. **代理检测与配置**：
   - 自动检测Windows系统代理设置
   - 通过访问Twitter验证代理可用性
   - 智能降级策略：代理可用→使用代理，代理不可用→测试直连，直连可用→使用直连，直连不可用→禁用Twitter功能
   - 配置完成后，所有网络请求都将使用检测到的代理配置
   - 保存Twitter网络可用性状态供后续使用
3. **Twitter配置**：交互式配置Twitter粉丝数获取功能
   - 读取之前保存的Twitter网络可用性状态
   - 如果网络可用，进行Twitter账号配置
   - 如果网络不可用或配置失败，跳过Twitter功能
4. **Excel加载**：读取Excel文件，初始化列映射和全局变量
5. **逐行处理**：对每个动画条目执行以下步骤：
   - **链接检查**：检测现有平台链接（超链接/纯文本URL）
   - **模式选择**：有链接的平台使用直接提取，无链接的使用搜索模式
   - **并发提取**：使用ThreadPoolExecutor同时从四个平台获取数据
   - **交叉验证**：MAL/AniList失败时使用对方名称重试
   - **数据标准化**：转换评分制度，验证日期一致性
   - **Twitter数据**：获取相关Twitter账号粉丝数（如果网络可用且配置成功）
   - **Excel更新**：写入获取到的数据和超链接
6. **结果输出**：保存Excel文件，生成日志报告，汇总日期错误

## 主要功能

### 网络代理配置
- **自动代理检测**：自动读取Windows系统代理设置
- **智能验证机制**：通过访问Twitter验证代理可用性
- **多层降级策略**：
  - 有代理且可用 → 所有网络请求使用代理
  - 有代理但不可用 → 测试直连，可用则使用直连
  - 无代理但直连可用 → 使用直连模式
  - 直连也不可用 → 禁用Twitter功能，继续其他功能
- **全局代理应用**：代理配置影响所有网络请求（Bangumi、MAL、AniList、Filmarks、Twitter）
- **状态透明显示**：实时显示代理状态和网络可用性

### 数据获取功能
- **Excel数据处理**：从Excel表格中读取动画作品名称列表，支持动态列映射
- **链接优先提取策略**：自动检测Excel中的平台链接列（`Bangumi_url`、`Anilist_url`、`Myanimelist_url`、`Filmarks_url`），优先使用已有链接直接提取数据
- **双模式数据获取**：
  - 链接模式：直接从Excel中的URL链接提取数据，速度快且准确
  - 搜索模式：当没有链接时，通过名称搜索获取数据
- **并发数据获取**：使用ThreadPoolExecutor同时从四个网站获取数据，提高处理效率
- **统一的提取器架构**：所有平台提取器都基于BaseExtractor，确保一致的行为和错误处理
- 自动从以下网站获取动画评分数据：
  - **Bangumi (番组计划)**：使用官方API，支持subject ID直接提取
  - **MyAnimeList (MAL)**：网页解析，支持完整URL直接提取
  - **AniList (AL)**：使用GraphQL API，支持anime ID直接提取
  - **Filmarks (FM)**：网页解析，支持完整URL直接提取
- **智能评分标准化**：自动转换不同平台的评分制度
  - AniList：100分制 → 10分制
  - Filmarks：5分制保持原始评分和乘2转换为10分制两种模式
  - Bangumi/MyAnimeList：10分制格式化
- **日期一致性检查**：验证各平台开播日期的一致性，自动检测和汇总日期错误信息
- **智能匹配算法**：基于发布年份筛选，确保获取正确的动画条目
- **Twitter粉丝数获取**：自动获取动画官方Twitter账号的粉丝数
  - 使用twscrape库进行数据获取
  - 支持Cookies鉴权
  - 内置内存缓存机制，24小时内不重复请求同一账号
  - 交互式配置，支持跳过功能
  - 智能错误处理，配置失败时自动跳过
  - 网络不可用时自动禁用，不影响其他功能
- **交叉验证功能**：当某个网站未找到匹配条目时，尝试使用其他网站的名称进行二次搜索
- **超链接和纯文本URL自动识别**：支持Excel中的超链接和纯文本URL格式
- **详细日志记录**：记录详细的运行日志、错误和警告信息
- **动态全局变量管理**：使用getter函数避免Python导入陷阱，确保配置正确更新


## 项目结构

```
mzzbscore/
├── main.py                     # 主程序入口，协调各模块工作
├── models/                     # 数据模型定义
│   └── anime_model.py         # 动画数据模型
├── src/                       # 核心业务逻辑
│   ├── extractors/            # 数据提取器模块 
│   │   ├── __init__.py        # 提取器导出接口
│   │   ├── base_extractor.py  # 基础提取器类和通用组件
│   │   ├── bangumi.py         # Bangumi数据提取器
│   │   ├── anilist.py         # AniList数据提取器
│   │   ├── myanimelist.py     # MyAnimeList数据提取器
│   │   ├── filmarks.py        # Filmarks数据提取器
│   │   └── twitter.py            # Twitter粉丝数提取器
│   ├── parsers/               # 业务专用解析器
│   │   ├── __init__.py        # 解析器导出接口
│   │   ├── base_parser.py     # 基础解析器类
│   │   ├── myanimelist_parser.py  # MAL页面解析器
│   │   ├── filmarks_parser.py # Filmarks页面解析器
│   │   ├── twitter_parser.py  # Twitter数据解析器
│   │   └── link_parser.py     # 链接解析和ID提取器
│   └── data_process/          # 数据处理模块
│       ├── excel_handler.py   # Excel数据写入处理
│       ├── score_transformers.py  # 评分标准化转换器
│       └── date_validator.py  # 日期一致性验证器
├── utils/                     # 工具函数模块
│   ├── __init__.py           # 工具函数导出接口
│   ├── core/                 # 核心工具
│   │   ├── global_variables.py  # 全局变量管理
│   │   ├── logger.py         # 日志管理
│   │   └── twitter_config.py # Twitter配置管理
│   ├── network/              # 网络请求工具
│   │   ├── network.py        # 网络请求封装和缓存
│   │   ├── proxy_config.py   # 代理配置和验证
│   │   └── headers.py        # 请求头定义
│   ├── parsers/              # 通用解析工具
│   │   ├── __init__.py       # 解析工具导出接口
│   │   └── text_processor.py # 文本预处理
│   ├── excel/                # Excel操作工具
│   │   ├── excel_utils.py    # Excel工具函数
│   │   └── excel_columns.py  # Excel列定义
│   ├── validators/           # 数据验证工具
│   │   └── data_validators.py # 数据验证和清理
│   └── date/                 # 日期处理工具
│       └── date_processors.py # 日期处理器
├── requirements.txt          # 项目依赖
├── ruff.toml                # 代码格式化配置
└── README.md                # 项目说明文档
```


## 技术特色

### 核心架构
- **统一的BaseExtractor架构**：所有平台提取器都继承自统一的BaseExtractor基类，确保一致的行为和接口
- **链接优先双模式运行**：智能检测Excel中的平台链接，优先使用直接提取模式，搜索模式作为兜底
- **模块化设计**：清晰的分层架构，核心业务逻辑在`src/`目录，工具函数在`utils/`目录
- **业务解析器分离**：专门的`src/parsers/`目录处理各平台特定的解析逻辑，提高代码可维护性
- **并发处理**：使用ThreadPoolExecutor同时从四个网站获取数据，显著提高处理效率

### 网络代理特色
- **自动代理检测**：自动读取Windows系统代理设置，支持多种代理格式解析
- **智能验证机制**：通过访问Twitter域名验证代理可用性，确保代理对实际业务有效
- **多层降级策略**：代理不可用时自动降级到直连模式，网络问题时智能禁用Twitter功能
- **全局代理应用**：所有网络请求（包括Twitter API）自动使用配置的代理
- **状态透明管理**：实时显示代理状态和网络可用性，提供详细的状态日志

### 数据处理特色
- **智能链接解析**：支持超链接和纯文本URL，自动从各平台URL中提取关键ID信息
- **评分标准化系统**：自动处理不同平台的评分制度差异
  - AniList：100分制 → 10分制 (÷10)
  - Filmarks：5分制保持原始 + 乘2转换为10分制
  - Bangumi/MAL：10分制格式化
- **日期一致性检测**：自动检测各平台开播日期的一致性，识别数据冲突并生成错误报告
- **交叉验证机制**：当某网站搜索失败时，使用其他网站的名称重试，提高数据获取成功率
- **动态全局变量管理**：解决Python导入陷阱，确保全局配置在运行时正确更新

## Twitter粉丝数功能

### 安全建议
- 建议使用专门的Twitter账号，避免使用主要账号
- 支持多账号轮换，降低封禁风险
- 所有配置信息仅本地存储，不会上传到服务器

