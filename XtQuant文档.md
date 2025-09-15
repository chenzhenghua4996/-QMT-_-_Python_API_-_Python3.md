### XtQuant 能提供哪些服务

XtQuant是基于迅投MiniQMT衍生出来的一套完善的Python策略运行框架，对外以Python库的形式提供策略交易所需要的行情和交易相关的API接口。

### XtQuant 运行依赖环境

XtQuant 目前提供的库包括 64 位 Python `3.6`、`3.7`、`3.8`、`3.9`、`3.10`、`3.11`、`3.12`版本，不同版本的 Python 导入时会自动切换。 在运行使用 XtQuant 的程序前需要先启动 MiniQMT 客户端。

### XtQuant 运行逻辑

**Xtdata** 作为行情模块，本模块旨在提供精简直接的数据满足量化交易者的数据需求，主要提供行情数据（历史和实时的K线和分笔）、财务数据、合约基础信息、板块和行业分类信息等通用的行情数据。

**Xttrader** 作为交易模块，封装了策略交易所需要的 Python API 接口，可以和 MiniQMT 客户端交互进行报单、撤单、查询资产、查询委托、查询成交、查询持仓以及接收资金、委托、成交和持仓等变动的主推消息。

# XtQuant.XtData 行情模块

xtdata是xtquant库中提供行情相关数据的模块，本模块旨在提供精简直接的数据满足量化交易者的数据需求，作为python库的形式可以被灵活添加到各种策略脚本中。

主要提供行情数据（历史和实时的K线和分笔）、财务数据、合约基础信息、板块和行业分类信息等通用的行情数据。

## 版本信息

- 2020-09-01
  - 初稿
- 2020-09-07
  - 添加获取除权数据的接口`get_divid_factors`，附录添加除权数据字段说明
  - 获取合约信息、获取合约类型接口完善
  - 获取交易日列表接口`get_trading_dates`支持指定日期范围
- 2020-09-13
  - 添加财务数据接口，调整获取和下载财务数据接口的说明，添加财务数据报表字段列表
  - 将 “补充” 字样调整为 “下载”，“supply” 接口调整为 “download”
- 2020-09-13
  - 将`volumn`拼写错误修正为`volume`，影响范围：
    - `tick`和`l2quote`周期行情数据 - 成交量字段
    - 合约基础信息 - 总股本、流通股本
- 2020-11-23
  - 合约基础信息`CreateDate` `OpenDate`字段类型由`int`调整为`str`
  - 添加数据字典部分，添加level2数据字段枚举值说明
- 2021-07-20
  - 添加新版下载数据接口
    - 下载行情数据 `download_history_data2`
    - 下载财务数据 `download_financial_data2`
- 2021-12-30
  - 数据字典调整
    - 委托方向、成交类型添加关于上交所、深交所撤单信息的区分说明
- 2022-06-27
  - 数据字典调整
    - K线添加前收价、停牌标记字段
- 2022-09-30
  - 添加交易日历相关接口
    - 获取节假日数据 `get_holidays`
    - 获取交易日历 `get_trading_calendar`
    - 获取交易时段 `get_trade_times`
- 2023-01-04
  - 添加千档行情获取
- 2023-01-31
  - 可转债基础信息的下载 `download_cb_data`
  - 可转债基础信息的获取 `get_cb_info`
- 2023-02-06
  - 添加连接到指定ip端口的接口 `reconnect`
- 2023-02-07
  - 支持QMT的本地Python模式
  - 优化多个QMT同时存在的场景，自动选择xtdata连接的端口
- 2023-03-27
  - 新股申购信息获取 `get_ipo_info`
- 2023-04-13
  - 本地python模式下运行VBA函数
- 2023-07-27
  - 文档部分描述修改
- 2023-08-21
  - 数据接口支持投研版特色数据
    - 参考 `接口概述` - `常用类型说明` - `周期` - `投研版 - 特色数据`
  - 获取合约基础信息 `get_instrument_detail` 返回字段调整
    - 增加 `ExchangeCode` `UniCode`
  - 添加获取可用周期列表的接口 `get_period_list`
- 2023-10-11
  - `get_market_data_ex`支持获取ETF申赎清单数据
  - 数据字典添加 现金替代标志
- 2023-11-09
  - `download_history_data`添加增量下载参数，支持指定起始时间的增量下载
- 2023-11-22
  - `get_trading_calendar`不再支持`tradetimes`参数
- 2023-11-27
  - ETF申赎清单信息下载 `download_etf_info`
  - ETF申赎清单信息获取 `get_etf_info`
- 2023-11-28
  - 添加节假日下载`download_holiday_data`
- 2023-12-27
  - 获取板块成份股列表接口增加北交所板块
- 2024-01-19
  - `get_market_data_ex`支持获取期货历史主力合约数据
  - `get_option_detail_data`支持获取商品期权品种的数据
  - `get_market_data_ex`支持获取日线以上周期的K线数据
    - 周线`1w`、月线`1mon`、季度线`1q`、半年线`1hy`、年线`1y`
- 2024-01-22
  - `get_trade_times`改名为`get_trading_time`
  - `get_trading_time`更新实现逻辑
- 2024-01-26
  - 获取合约基础信息 `get_instrument_detail` 支持获取全部合约信息字段
- 2024-05-15
  - 获取最新交易日k线数据`get_full_kline`
- 2024-05-27
  - `get_stock_list_in_sector` 增加`real_timetag`参数

## 接口概述

### 运行逻辑

xtdata提供和MiniQmt的交互接口，本质是和MiniQmt建立连接，由MiniQmt处理行情数据请求，再把结果回传返回到python层。使用的行情服务器以及能获取到的行情数据和MiniQmt是一致的，要检查数据或者切换连接时直接操作MiniQmt即可。

对于数据获取接口，使用时需要先确保MiniQmt已有所需要的数据，如果不足可以通过补充数据接口补充，再调用数据获取接口获取。

对于订阅接口，直接设置数据回调，数据到来时会由回调返回。订阅接收到的数据一般会保存下来，同种数据不需要再单独补充。

### 接口分类

- 行情数据（K线数据、分笔数据，订阅和主动获取的接口）
  - 功能划分（接口前缀）
    - subscribe_ / unsubscribe_ 订阅/反订阅
    - get_ 获取数据
    - download_ 下载数据
  - 常见用法
    - level1数据的历史部分用`download_history_data`补充，实时部分用`subscribe_XXX`订阅，使用`get_XXX`获取
    - level2数据实时部分用`subscribe_XXX`订阅，用`get_l2_XXX`获取。level2函数无历史数据存储，跨交易日后数据清理
- 财务数据
- 合约基础信息
- 基础行情数据板块分类信息等基础信息

### 常用类型说明

- stock_code - 合约代码
  - 格式为 `code.market`，例如`000001.SZ` `600000.SH` `000300.SH`
- period - 周期，用于表示要获取的周期和具体数据类型
  - level1数据
    - `tick` - 分笔数据
    - `1m` - 1分钟线
    - `5m` - 5分钟线
    - `15m` - 15分钟线
    - `30m` - 30分钟线
    - `1h` - 1小时线
    - `1d` - 日线
    - `1w` - 周线
    - `1mon` - 月线
    - `1q` - 季度线
    - `1hy` - 半年线
    - `1y` - 年线
  - 投研版 - 特色数据
    - `warehousereceipt` - 期货仓单
    - `futureholderrank` - 期货席位
    - `interactiveqa` - 互动问答
    - 逐笔成交统计
      - `transactioncount1m` - 逐笔成交统计1分钟级
      - `transactioncount1d` - 逐笔成交统计日级
    - `delistchangebond` - 退市可转债信息
    - `replacechangebond` - 待发可转债信息
    - `specialtreatment` - ST 变更历史
    - 港股通（深港通、沪港通）资金流向
      - `northfinancechange1m` - 港股通资金流向1分钟级
      - `northfinancechange1d` - 港股通资金流向日级
    - `dividendplaninfo` - 红利分配方案信息
    - `historycontract` - 过期合约列表
    - `optionhistorycontract` - 期权历史信息
    - `historymaincontract` - 历史主力合约
    - `stoppricedata` - 涨跌停数据
    - `snapshotindex` - 快照指标数据
- 时间范围，用于指定数据请求范围，表示的范围是`[start_time, end_time]`区间（包含前后边界）中最后不多于`count`个数据
  - start_time - 起始时间，为空则认为是最早的起始时间
  - end_time - 结束时间，为空则认为是最新的结束时间
  - count - 数据个数，大于0为正常限制返回个数，等于0为不需要返回，-1为返回全部
  - 通常以`[start_time = '', end_time = '', count = -1]`表示完整数据范围，但数据请求范围过大会导致返回时间变长，需要按需裁剪请求范围
- dividend_type - 除权方式，用于K线数据复权计算，对`tick`等其他周期数据无效
  - `none` 不复权
  - `front` 前复权
  - `back` 后复权
  - `front_ratio` 等比前复权
  - `back_ratio` 等比后复权
- 其他依赖库 numpy、pandas会在数据返回的过程中使用
  - 本模块会尽可能减少对numpy和pandas库的直接依赖，以允许使用者在不同版本的库之间自由切换
  - pandas库中旧的三维数据结构Panel没有被使用，而是以dict嵌套DataFrame代替（后续可能会考虑使用xarray等的方案，也欢迎使用者提供改进建议）
  - 后文中会按常用规则分别简写为np、pd，如np.ndarray、pd.DataFrame

### 请求限制

- 全推数据是市场全部合约的切面数据，是高订阅数场景下的有效解决方案。持续订阅全推数据可以获取到每个合约最新分笔数据的推送，且流量和处理效率都优于单股订阅
- 单股订阅行情是仅返回单股数据的接口，建议单股订阅数量不超过50。如果订阅数较多，建议直接使用全推数据
- 板块分类信息等静态信息更新频率低，无需频繁下载，按周或按日定期下载更新即可

## 接口说明

### 行情接口

#### 订阅单股行情

```
subscribe_quote(stock_code, period='1d', start_time='', end_time='', count=0, callback=None)
```

- 释义
  
  - 订阅单股的行情数据，返回订阅号
  - 数据推送从callback返回，数据类型和period指定的周期对应
  - 数据范围代表请求的历史部分的数据范围，数据返回后会进入缓存，用于保证数据连续，通常情况仅订阅数据时传`count = 0`即可

- 参数
  
  - stock_code - string 合约代码
  
  - period - string 周期
  
  - start_time - string 起始时间
  
  - end_time - string 结束时间
  
  - count - int 数据个数
  
  - callback - 数据推送回调
    
    - 回调定义形式为`on_data(datas)`，回调参数`datas`格式为 { stock_code : [data1, data2, ...] }
    
    ```
    def on_data(datas):
        for stock_code in datas:
                print(stock_code, datas[stock_code])
    ```

- 返回
  
  - 订阅号，订阅成功返回`大于0`，失败返回`-1`

- 备注
  
  - 单股订阅数量不宜过多，详见 接口概述-请求限制

#### 订阅全推行情

```
subscribe_whole_quote(code_list, callback=None)
```

- 释义
  
  - 订阅全推行情数据，返回订阅号
  - 数据推送从callback返回，数据类型为分笔数据

- 参数
  
  - code_list - 代码列表，支持传入市场代码或合约代码两种方式
    
    - 传入市场代码代表订阅全市场，示例：`['SH', 'SZ']`
    - 传入合约代码代表订阅指定的合约，示例：`['600000.SH', '000001.SZ']`
  
  - callback - 数据推送回调
    
    - 回调定义形式为`on_data(datas)`，回调参数`datas`格式为 { stock1 : data1, stock2 : data2, ... }
    
    ```
    def on_data(datas):
        for stock_code in datas:
                print(stock_code, datas[stock_code])
    ```

- 返回
  
  - 订阅号，订阅成功返回`大于0`，失败返回`-1`

- 备注
  
  - 订阅后会首先返回当前最新的全推数据

#### 反订阅行情数据

```
unsubscribe_quote(seq)
```

- 释义
  - 反订阅行情数据
- 参数
  - seq - 订阅时返回的订阅号
- 返回
  - 无
- 备注
  - 无

#### 阻塞线程接收行情回调

```
run()
```

- 释义
  - 阻塞当前线程来维持运行状态，一般用于订阅数据后维持运行状态持续处理回调
- 参数
  - seq - 订阅时返回的订阅号
- 返回
  - 无
- 备注
  - 实现方式为持续循环sleep，并在唤醒时检查连接状态，若连接断开则抛出异常结束循环

#### 订阅模型

```
subscribe_formula(formula_name, stock_code, period, start_time = '', end_time = '', count = -1, dividend_type = None, extend_param = {}, callback = None)
```

- 释义
  
  - 订阅vba模型运行结果，需连接投研端使用

- 参数
  
  - formula_name:str,模型名
  
  - stock_code:str,模型主图代码形式如'stkcode.market',如'000300.SH'；
  
  - period:str,K线周期类型 可选范围： 'tick':分笔线 '1d':日线 '1m':分钟线 '3m':三分钟线 '5m':5分钟线 '15m':15分钟线 '30m':30分钟线 '1h':小时线 '1w':周线 '1mon':月线 '1q':季线 '1hy':半年线 '1y':年线
  
  - start_time:str,模型运行起始时间,形如:'20200101';默认为空视为最早
  
  - end_time:str,模型运截止时间,形如:'20200101';默认为空视为最新
  
  - count:int,模型运行范围为向前count根bar,默认为-1运行所有bar
  
  - dividend_type:str,复权方式,默认为主图除权方式,可选范围： 'none':不复权 'front':向前复权 'back':向后复权 'front_ratio':等比向前复权 'back_ratio':等比向后复权
  
  - extend_param:dict,模型的入参,{参数名:参数值},形如{'a':1,'__basket':{}};
  
  - __basket:dict,可选参数,组合模型的股票池权重,形如{'600000.SH':0.06,'000001.SZ':0.01}

- 返回：
  
  - int 订阅成功时为订阅ID，可用于后续反订阅,失败返回-1

- 备注:
  
  - 使用该函数时需要补充号本地K线或分笔数据

#### 反订阅模型

```
unsubscribe_formula(subID)
```

- 释义
  - 反订阅模型
- 参数
  - subID:int 模型订阅号
- 返回
  - bool ,反订阅成功为True,失败为False

#### 调用模型

```
call_formula(formula_name,stock_code,period,start_time="",end_time="",count=-1,dividend_type="none",extend_param={})
```

- 释义
  
  - 获取vba模型运行结果，使用前要注意补充本地K线数据或分笔数据

- 参数：
  
  - formula_name: str，模型名称名
  - stock_code: str，模型主图代码形式如'stkcode.market'，如'000300.SH'
  - period: str，K线周期类型
    - 可选范围：
      - 'tick': 分笔线
      - '1d': 日线
      - '1m': 分钟线
      - '3m': 三分钟线
      - '5m': 5分钟线
      - '15m': 15分钟线
      - '30m': 30分钟线
      - '1h': 小时线
      - '1w': 周线
      - '1mon': 月线
      - '1q': 季线
      - '1hy': 半年线
      - '1y': 年线
  - start_time: str，模型运行起始时间，形如:'20200101'，默认为空视为最早
  - end_time: str，模型运行截止时间，形如:'20200101'，默认为空视为最新
  - count: int，模型运行范围为向前 count 根 bar，默认为 -1 运行所有 bar
  - dividend_type: str，复权方式，默认为主图除权方式
    - 可选范围：
      - 'none': 不复权
      - 'front': 向前复权
      - 'back': 向后复权
      - 'front_ratio': 等比向前复权
      - 'back_ratio': 等比向后复权
  - extend_param: dict，模型的入参，例如 {"模型名:参数名": 参数值}，例如在跑模型 MA 时，{'MA:n1': 1}
    - 入参可以添加 `__basket: dict`，组合模型的股票池权重，形如 `{'__basket': {'600000.SH': 0.06, '000001.SZ': 0.01}}`
    - 如果在跑一个模型1的时候，模型1调用了模型2，如果只想修改模型2的参数可以传 `{'模型2: 参数': 参数值}`

- 返回
  
  - dict{ 'dbt':0,#返回数据类型，0:全部历史数据 'timelist':[...],#返回数据时间范围list, 'outputs':{'var1':[...],'var2':[...]}#输出变量名：变量值list }

#### 批量调用模型

```
call_formula_batch(formula_names,stock_codes,period,start_time="",end_time="",count=-1,dividend_type="none",extend_params=[])
```

- 释义
  
  - 批量获取vba模型运行结果，使用前要注意补充本地K线数据或分笔数据

- 参数：
  
  - formula_names: list，包含要批量运行的模型名
  - stock_codes: list，包含要批量运行的模型主图代码形式 'stkcode.market'，如 '000300.SH'
  - period: str，K线周期类型
    - 可选范围：
      - 'tick': 分笔线
      - '1d': 日线
      - '1m': 分钟线
      - '3m': 三分钟线
      - '5m': 5分钟线
      - '15m': 15分钟线
      - '30m': 30分钟线
      - '1h': 小时线
      - '1w': 周线
      - '1mon': 月线
      - '1q': 季线
      - '1hy': 半年线
      - '1y': 年线
  - start_time: str，模型运行起始时间，形如:'20200101'，默认为空视为最早
  - end_time: str，模型运行截止时间，形如:'20200101'，默认为空视为最新
  - count: int，模型运行范围为向前 count 根 bar，默认为 -1 运行所有 bar
  - dividend_type: str，复权方式，默认为主图除权方式
    - 可选范围：
      - 'none': 不复权
      - 'front': 向前复权
      - 'back': 向后复权
      - 'front_ratio': 等比向前复权
      - 'back_ratio': 等比向后复权
  - extend_params: list，包含每个模型的入参，形如 [{"模型名:参数名": 参数值}]，例如在跑模型 MA 时，{'MA:n1': 1}
    - 入参可以添加 `__basket: dict`，组合模型的股票池权重，形如 `{'__basket': {'600000.SH': 0.06, '000001.SZ': 0.01}}`
    - 如果在跑一个模型1的时候，模型1调用了模型2，如果只想修改模型2的参数可以传 `{'模型2: 参数': 参数值}`

- 返回
  
  - list[dict]
    - dict说明:
      - formula:模型名
      - stock:品种代码
      - argument:参数
      - result:dict参考call_formula返回结果

#### 生成因子数据

```
generate_index_data(formula_name, formula_param = {}, stock_list = [], period = '1d', dividend_type = 'none', start_time = '', end_time = '', fill_mode = 'fixed', fill_value = float('nan'), result_path = None)
```

- 释义
  
  - 在本地生成因子数据文件，文件格式为feather

- 参数
  
  - formula_name:str 模型名称
  - formula_param:dict 模型参数,例如 {'param1': 1.0, 'param2': 'sym'}
  - stock_list:list 股票列表
  - period:str 周期
    - 可选范围
      - '1m' '5m' '1d'
  - dividend_type:str 复权方式
    - 可选范围
      - 'none' - 不复权
      - 'front_ratio' - 等比前复权
      - 'back_ratio' - 等比后复权
  - start_time:str 起始时间 格式为'20240101' 或 '20240101000000'
  - end_time: str 结束时间 格式为'20241231' 或 '20241231235959'
  - fill_mode:str 空缺填充方式
    - 可选范围
      - 'fixed' - 固定值填充
      - 'forward' - 向前延续
  - fill_value:float 填充数值
    - float('nan') - 以NaN填充
  - result_path:str 结果文件路径，feather格式

- 返回 None

- 备注 必须连接投研端使用，传入的formula_name需要存在于投研端中

#### 获取行情数据

```
get_market_data(field_list=[], stock_list=[], period='1d', start_time='', end_time='', count=-1, dividend_type='none', fill_data=True)
```

- 释义
  - 从缓存获取行情数据，是主动获取行情的主要接口
- 参数
  - field_list - list 数据字段列表，传空则为全部字段
  - stock_list - list 合约代码列表
  - period - string 周期
  - start_time - string 起始时间
  - end_time - string 结束时间
  - count - int 数据个数
  - 默认参数，大于等于0时，若指定了start_time，end_time，此时以end_time为基准向前取count条；若start_time，end_time缺省，默认取本地数据最新的count条数据；若start_time，end_time，count都缺省时，默认取本地全部数据
  - dividend_type - string 除权方式
  - fill_data - bool 是否向后填充空缺数据
- 返回
  - period为`1m` `5m` `1d`等K线周期时
    - 返回dict { field1 : value1, field2 : value2, ... }
    - field1, field2, ... ：数据字段
    - value1, value2, ... ：pd.DataFrame 数据集，index为stock_list，columns为time_list
    - 各字段对应的DataFrame维度相同、索引相同
  - period为`tick`分笔周期时
    - 返回dict { stock1 : value1, stock2 : value2, ... }
    - stock1, stock2, ... ：合约代码
    - value1, value2, ... ：np.ndarray 数据集，按数据时间戳`time`增序排列
- 备注
  - 获取lv2数据时需要数据终端有lv2数据权限
  - 时间范围为闭区间

#### 获取本地行情数据

```
get_local_data(field_list=[], stock_list=[], period='1d', start_time='', end_time='', count=-1,
               dividend_type='none', fill_data=True, data_dir=data_dir)
```

- 释义
  - 从本地数据文件获取行情数据，用于快速批量获取历史部分的行情数据
- 参数
  - field_list - list 数据字段列表，传空则为全部字段
  - stock_list - list 合约代码列表
  - period - string 周期
  - start_time - string 起始时间
  - end_time - string 结束时间
  - count - int 数据个数
  - dividend_type - string 除权方式
  - fill_data - bool 是否向后填充空缺数据
  - data_dir - string MiniQmt配套路径的userdata_mini路径，用于直接读取数据文件。默认情况下xtdata会通过连接向MiniQmt直接获取此路径，无需额外设置。如果需要调整，可以将数据路径作为`data_dir`传入，也可以直接修改`xtdata.data_dir`以改变默认值
- 返回
  - period为`1m` `5m` `1d`K线周期时
    - 返回dict { field1 : value1, field2 : value2, ... }
    - field1, field2, ... ：数据字段
    - value1, value2, ... ：pd.DataFrame 数据集，index为stock_list，columns为time_list
    - 各字段对应的DataFrame维度相同、索引相同
  - period为`tick`分笔周期时
    - 返回dict { stock1 : value1, stock2 : value2, ... }
    - stock1, stock2, ... ：合约代码
    - value1, value2, ... ：np.ndarray 数据集，按数据时间戳`time`增序排列
- 备注
  - 仅用于获取level1数据

#### 获取全推数据

```
get_full_tick(code_list)
```

- 释义
  - 获取全推数据
- 参数
  - code_list - 代码列表，支持传入市场代码或合约代码两种方式
    - 传入市场代码代表订阅全市场，示例：`['SH', 'SZ']`
    - 传入合约代码代表订阅指定的合约，示例：`['600000.SH', '000001.SZ']`
- 返回
  - dict 数据集 { stock1 : data1, stock2 : data2, ... }
- 备注
  - 无

#### 获取除权数据

```
get_divid_factors(stock_code, start_time='', end_time='')
```

- 释义
  - 获取除权数据
- 参数
  - stock_code - 合约代码
  - start_time - string 起始时间
  - end_time - string 结束时间
- 返回
  - pd.DataFrame 数据集
- 备注
  - 无

#### 下载历史行情数据

```
download_history_data(stock_code, period, start_time='', end_time='', incrementally = None)
```

- 释义
  - 补充历史行情数据
- 参数
  - stock_code - string 合约代码
  - period - string 周期
  - start_time - string 起始时间
  - end_time - string 结束时间
  - incrementally - 是否增量下载
    - `bool` - 是否增量下载
    - `None` - 使用`start_time`控制，`start_time`为空则增量下载，增量下载时会从本地最后一条数据往后下载
- 返回
  - 无
- 备注
  - 同步执行，补充数据完成后返回

```
download_history_data2(stock_list, period, start_time='', end_time='', callback=None,incrementally = None)
```

- 释义
  
  - 补充历史行情数据，批量版本

- 参数
  
  - stock_list - list 合约列表
  
  - period - string 周期
  
  - start_time - string 起始时间
  
  - end_time - string 结束时间
  
  - callback - func 回调函数
    
    - 参数为进度信息dict
      
      - total - 总下载个数
      - finished - 已完成个数
      - stockcode - 本地下载完成的合约代码
      - message - 本次信息
    
    - ```
      def on_progress(data):
          print(data)
          # {'finished': 1, 'total': 50, 'stockcode': '000001.SZ', 'message': ''}
      ```

- 返回
  
  - 无

- 备注
  
  - 同步执行，补充数据完成后返回
  - 有任务完成时通过回调函数返回进度信息

#### 下载过期（退市）合约信息

```
download_history_contracts()
```

- 释义
  - 下载过期（退市）合约信息，过期（退市）标的列表可以通过get_stock_list_in_sector获取
- 参数
  - None
- 返回
  - 无
- 备注
  - 同步执行，补充数据完成后返回
  - 过期板块名称可以通过 `print([i for i in xtdata.get_sector_list() if "过期" in i])` 查看
  - 下载完成后，可以通过 `xtdata.get_instrument_detail()` 查看过期（退市）合约信息

#### 获取节假日数据

```
get_holidays()
```

- 释义
  - 获取截止到当年的节假日日期
- 参数
  - 无
- 返回
  - list，为8位的日期字符串格式
- 备注
  - 无

#### 获取交易日历

```
get_trading_calendar(market, start_time = '', end_time = '')
```

- 释义
  - 获取指定市场交易日历
- 参数
  - market - str 市场
  - start_time - str 起始时间，8位字符串。为空表示当前市场首个交易日时间
  - end_time - str 结束时间，8位字符串。为空表示当前时间
- 返回
  - 返回list，完整的交易日列表
- 备注
  - 结束时间可以填写未来时间，获取未来交易日。需要下载节假日列表。

#### 获取交易时段

```
get_trading_time(stockcode)
```

- 释义
  
  - 返回指定代码的交易时段

- 参数
  
  - stockcode - str 合约代码（例如`600000.SH`）

- 返回
  
  - list，返回交易时段列表，第一位是开始时间，第二位结束时间，第三位交易类型 （2 - 开盘竞价， 3 - 连续交易， 8 - 收盘竞价， 9 - 盘后定价）。时间单位为“秒”

- 备注
  
  - 股票代码错误时返回空列表
  
  - 跨天时以当前天0点为起始，前一天为负，下一天多86400
  
  - ```
    #需要转换为datetime时，可以用以下方法转换
    import datetime as dt
    dt.datetime.combine(dt.date.today(), dt.time()) + dt.timedelta(seconds = 34200)
    ```

#### 可转债基础信息的下载

```
download_cb_data()
```

- 释义
  - 下载全部可转债信息
- 参数
  - 无
- 返回
  - 无
- 备注
  - 无

#### 获取可转债基础信息

```
get_cb_info(stockcode)
```

- 释义
  - 返回指定代码的可转债信息
- 参数
  - stockcode - str 合约代码（例如`600000.SH`）
- 返回
  - dict，可转债信息
- 备注
  - 需要先下载可转债数据

#### 获取新股申购信息

```
get_ipo_info(start_time, end_time)
```

- 释义
  
  - 返回所选时间范围的新股申购信息

- 参数
  
  - start_time: 开始日期（如：'20230327'）
  - end_time: 结束日期（如：'20230327'）
  - start_time 和 end_time 为空则返回全部数据

- 返回
  
  - list[dict]，新股申购信息
  
  - ```
    securityCode - string 证券代码
    codeName - string 代码简称
    market - string 所属市场
    actIssueQty - int 发行总量，单位：股
    onlineIssueQty - int 网上发行量, 单位：股
    onlineSubCode - string 申购代码
    onlineSubMaxQty - int 申购上限, 单位：股
    publishPrice - float 发行价格
    isProfit - int 是否已盈利 0：上市时尚未盈利 1：上市时已盈利
    industryPe - float 行业市盈率
    afterPE - float 发行后市盈率
    ```

#### 获取可用周期列表

```
get_period_list()
```

- 释义
  
  - 返回可用周期列表

- 参数
  
  - 无

- 返回
  
  - list 周期列表

#### ETF申赎清单信息下载

```
download_etf_info()
```

- 释义
  
  - 下载所有ETF申赎清单信息

- 参数
  
  - 无

- 返回
  
  - 无

#### ETF申赎清单信息获取

```
get_etf_info()
```

- 释义
  
  - 获取所有ETF申赎清单信息

- 参数
  
  - 无

- 返回
  
  - dict 所有申赎数据

#### 节假日下载

```
download_holiday_data()
```

- 释义
  
  - 下载节假日数据

- 参数
  
  - 无

- 返回
  
  - 无

#### 获取最新交易日k线数据

```
get_full_kline(field_list = [], stock_list = [], period = '1m'
    , start_time = '', end_time = '', count = 1
    , dividend_type = 'none', fill_data = True)
```

- 释义
  
  - 获取最新交易日k线全推数据,仅支持最新一个交易日，不包含历史值

- 参数
  
  - 参考`get_market_data`函数

- 返回
  
  - dict - {field: DataFrame}

### 财务数据接口

#### 获取财务数据

```
get_financial_data(stock_list, table_list=[], start_time='', end_time='', report_type='report_time')
```

- 释义
  
  - 获取财务数据

- 参数
  
  - stock_list - list 合约代码列表
  
  - table_list - list 财务数据表名称列表
    
    - ```
      'Balance'          #资产负债表
      'Income'           #利润表
      'CashFlow'         #现金流量表
      'Capital'          #股本表
      'Holdernum'        #股东数
      'Top10holder'      #十大股东
      'Top10flowholder'  #十大流通股东
      'Pershareindex'    #每股指标
      ```
  
  - start_time - string 起始时间
  
  - end_time - string 结束时间
  
  - report_type - string 报表筛选方式
    
    - ```
      'report_time'     #截止日期
      'announce_time' #披露日期
      ```

- 返回
  
  - dict 数据集 { stock1 : datas1, stock2 : data2, ... }
  - stock1, stock2, ... ：合约代码
  - datas1, datas2, ... ：dict 数据集 { table1 : table_data1, table2 : table_data2, ... }
    - table1, table2, ... ：财务数据表名
    - table_data1, table_data2, ... ：pd.DataFrame 数据集，数据字段详见附录 - 财务数据字段列表

- 备注
  
  - 无

#### 下载财务数据

```
download_financial_data(stock_list, table_list=[])
```

- 释义
  - 下载财务数据
- 参数
  - stock_list - list 合约代码列表
  - table_list - list 财务数据表名列表
- 返回
  - 无
- 备注
  - 同步执行，补充数据完成后返回

```
download_financial_data2(stock_list, table_list=[], start_time='', end_time='', callback=None)
```

- 释义
  
  - 下载财务数据

- 参数
  
  - stock_list - list 合约代码列表
  
  - table_list - list 财务数据表名列表
  
  - start_time - string 起始时间
  
  - end_time - string 结束时间
    
    - 以`m_anntime`披露日期字段，按`[start_time, end_time]`范围筛选
  
  - callback - func 回调函数
    
    - 参数为进度信息dict
      
      - total - 总下载个数
      - finished - 已完成个数
      - stockcode - 本地下载完成的合约代码
      - message - 本次信息
    
    - ```
      def on_progress(data):
          print(data)
          # {'finished': 1, 'total': 50, 'stockcode': '000001.SZ', 'message': ''}
      ```

- 返回
  
  - 无

- 备注
  
  - 同步执行，补充数据完成后返回

### 基础行情信息

#### 获取合约基础信息

```
get_instrument_detail(stock_code, iscomplete)
```

- 释义
  
  - 获取合约基础信息

- 参数
  
  - stock_code - string 合约代码
  - iscomplete - bool 是否获取全部字段，默认为False

- 返回
  
  - dict 数据字典，{ field1 : value1, field2 : value2, ... }，找不到指定合约时返回`None`
  
  - iscomplete为False时，返回以下字段
    
    ```
    ExchangeID - string 合约市场代码
    InstrumentID - string 合约代码
    InstrumentName - string 合约名称
    ProductID - string 合约的品种ID(期货)
    ProductName - string 合约的品种名称(期货)
    ExchangeCode - string 交易所代码
    UniCode - string 统一规则代码
    CreateDate - str 上市日期(期货)
    OpenDate - str IPO日期(股票)
    ExpireDate - int 退市日或者到期日
    PreClose - float 前收盘价格
    SettlementPrice - float 前结算价格
    UpStopPrice - float 当日涨停价
    DownStopPrice - float 当日跌停价
    FloatVolume - float 流通股本
    TotalVolume - float 总股本
    LongMarginRatio - float 多头保证金率
    ShortMarginRatio - float 空头保证金率
    PriceTick - float 最小价格变动单位
    VolumeMultiple - int 合约乘数(对期货以外的品种，默认是1)
    MainContract - int 主力合约标记，1、2、3分别表示第一主力合约，第二主力合约，第三主力合约
    LastVolume - int 昨日持仓量
    InstrumentStatus - int 合约停牌状态
    IsTrading - bool 合约是否可交易
    IsRecent - bool 是否是近月合约
    OpenInterestMultiple - int 交割月持仓倍数 
    ```
  
  - iscomplete为True时，增加会返回更多合约信息字段，例如
    
    ```
    ChargeType - int 期货和期权手续费方式 0表示未知，1表示按元/手，2表示按费率，单位为万分比，‱
    ChargeOpen - float 开仓手续费(率) 返回-1时该值无效，其余情况参考ChargeType
    ChargeClose - float 平仓手续费(率) 返回-1时该值无效，其余情况参考ChargeType
    ChargeTodayOpen - float 开今仓(日内开仓)手续费(率) 返回-1时该值无效，其余情况参考ChargeType
    ChargeTodayClose - float 平今仓(日内平仓)手续费(率)  返回-1时该值无效，其余情况参考ChargeType
    OptionType - int 期权类型 返回-1表示合约为非期权 返回0为期权认购  返回1为期权认沽
    ......
    ```
  
  - 详细合约信息字段见`附录-合约信息字段列表`

- 备注
  
  - 可用于检查合约代码是否正确
  - 合约基础信息`CreateDate` `OpenDate`字段类型由`int`调整为`str`

#### 获取合约类型

```
get_instrument_type(stock_code)
```

- 释义
  
  - 获取合约类型

- 参数
  
  - stock_code - string 合约代码

- 返回
  
  - dict 数据字典，{ type1 : value1, type2 : value2, ... }，找不到指定合约时返回`None`
    
    - type1, type2, ... ：string 合约类型
    - value1, value2, ... ：bool 是否为该类合约
  
  - ```
    'index'        #指数
    'stock'        #股票
    'fund'        #基金
    'etf'        #ETF
    ```

- 备注
  
  - 无

#### 获取交易日列表

```
get_trading_dates(market, start_time='', end_time='', count=-1)
```

- 释义
  - 获取交易日列表
- 参数
  - market - string 市场代码
  - start_time - string 起始时间
  - end_time - string 结束时间
  - count - int 数据个数
- 返回
  - list 时间戳列表，[ date1, date2, ... ]
- 备注
  - 无

#### 获取板块列表

```
get_sector_list()
```

- 释义
  - 获取板块列表
- 参数
  - 无
- 返回
  - list 板块列表，[ sector1, sector2, ... ]
- 备注
  - 需要下载板块分类信息

#### 获取板块成分股列表

```
get_stock_list_in_sector(sector_name)
```

- 释义
  - 获取板块成分股列表
- 参数
  - sector_name - string 版块名称
- 返回
  - list 成分股列表，[ stock1, stock2, ... ]
- 备注
  - 需要板块分类信息

#### 下载板块分类信息

```
download_sector_data()
```

- 释义
  - 下载板块分类信息
- 参数
  - 无
- 返回
  - 无
- 备注
  - 同步执行，下载完成后返回

#### 创建板块目录节点

```
create_sector_folder(parent_node, folder_name, overwrite)
```

- 释义
  - 创建板块目录节点
- 参数
  - parent_node - string 父节点，’ ‘为 '我的‘ （默认目录）
  - folder_name - string 要创建的板块目录名称
  - overwrite- bool 是否覆盖，如果目标节点已存在，为True时跳过，为False时在folder_name后增加数字编号，编号为从1开始自增的第一个不重复的值。 默认为True
- 返回
  - folder_name2 - string 实际创建的板块目录名
- 备注
  - 无

#### 创建板块

```
create_sector(parent_node, sector_name, overwrite)
```

- 释义
  - 创建板块
- 参数
  - parent_node - string 父节点，’ ‘为 '我的‘ （默认目录）
  - sector_name - string 板块名称
  - overwrite- bool 是否覆盖，如果目标节点已存在，为True时跳过，为False时在sector_name后增加数字编号，编号为从1开始自增的第一个不重复的值。 默认为True
- 返回
  - sector_name2 - string 实际创建的板块名
- 备注
  - 无

#### 添加自定义板块

```
add_sector(sector_name, stock_list)
```

- 释义
  - 添加自定义板块
- 参数
  - sector_name - string 板块名称
  - stock_list - list 成分股列表
- 返回
  - 无
- 备注
  - 无

#### 移除板块成分股

```
remove_stock_from_sector(sector_name, stock_list)
```

- 释义
  - 创建板块
- 参数
  - sector_name - string 板块名称
  - stock_list- list 成分股列表
- 返回
  - result - bool 操作成功为True，失败为False
- 备注
  - 无

#### 移除自定义板块

```
remove_sector(sector_name)
```

- 释义
  - 移除自定义板块
- 参数
  - sector_name - string 板块名称
- 返回
  - 无
- 备注
  - 无

#### 重置板块

```
reset_sector(sector_name, stock_list)
```

- 释义
  - 重置板块
- 参数
  - sector_name - string 板块名称
  - stock_list- list 成分股列表
- 返回
  - result - bool 操作成功为True，失败为False
- 备注
  - 无

#### 获取指数成分权重信息

```
get_index_weight(index_code)
```

- 释义
  - 获取指数成分权重信息
- 参数
  - index_code - string 指数代码
- 返回
  - dict 数据字典，{ stock1 : weight1, stock2 : weight2, ... }
- 备注
  - 需要下载指数成分权重信息

#### 下载指数成分权重信息

```
download_index_weight()
```

- 释义
  - 下载指数成分权重信息
- 参数
  - 无
- 返回
  - 无
- 备注
  - 同步执行，下载完成后返回

## 附录

### 行情数据字段列表

#### tick - 分笔数据

```
'time'                  #时间戳
'lastPrice'             #最新价
'open'                  #开盘价
'high'                  #最高价
'low'                   #最低价
'lastClose'             #前收盘价
'amount'                #成交总额
'volume'                #成交总量
'pvolume'               #原始成交总量
'stockStatus'           #证券状态
'openInt'               #持仓量
'lastSettlementPrice'   #前结算
'askPrice'              #委卖价
'bidPrice'              #委买价
'askVol'                #委卖量
'bidVol'                #委买量
'transactionNum'        #成交笔数
```

#### 1m / 5m / 1d - K线数据

```
'time'                  #时间戳
'open'                  #开盘价
'high'                  #最高价
'low'                   #最低价
'close'                 #收盘价
'volume'                #成交量
'amount'                #成交额
'settelementPrice'      #今结算
'openInterest'          #持仓量
'preClose'              #前收价
'suspendFlag'           #停牌标记 0 - 正常 1 - 停牌 -1 - 当日起复牌
```

#### 除权数据

```
'interest'                #每股股利（税前，元）
'stockBonus'              #每股红股（股）
'stockGift'               #每股转增股本（股）
'allotNum'                #每股配股数（股）
'allotPrice'              #配股价格（元）
'gugai'                   #是否股改, 对于股改，在算复权系数时，系统有特殊算法
'dr'                      #除权系数
```

#### l2quote - level2实时行情快照

```
'time'                  #时间戳
'lastPrice'             #最新价
'open'                  #开盘价
'high'                  #最高价
'low'                   #最低价
'amount'                #成交额
'volume'                #成交总量
'pvolume'               #原始成交总量
'openInt'               #持仓量
'stockStatus'           #证券状态
'transactionNum'        #成交笔数
'lastClose'             #前收盘价
'lastSettlementPrice'   #前结算
'settlementPrice'       #今结算
'pe'                    #市盈率
'askPrice'              #多档委卖价
'bidPrice'              #多档委买价
'askVol'                #多档委卖量
'bidVol'                #多档委买量
```

#### l2order - level2逐笔委托

```
'time'                  #时间戳
'price'                 #委托价
'volume'                #委托量
'entrustNo'             #委托号
'entrustType'           #委托类型
'entrustDirection'      #委托方向
```

#### l2transaction - level2逐笔成交

```
'time'                  #时间戳
'price'                 #成交价
'volume'                #成交量
'amount'                #成交额
'tradeIndex'            #成交记录号
'buyNo'                 #买方委托号
'sellNo'                #卖方委托号
'tradeType'             #成交类型
'tradeFlag'             #成交标志
```

#### l2quoteaux - level2实时行情补充（总买总卖）

```
'time'                  #时间戳
'avgBidPrice'           #委买均价
'totalBidQuantity'      #委买总量
'avgOffPrice'           #委卖均价
'totalOffQuantity'      #委卖总量
'withdrawBidQuantity'   #买入撤单总量
'withdrawBidAmount'     #买入撤单总额
'withdrawOffQuantity'   #卖出撤单总量
'withdrawOffAmount'     #卖出撤单总额
```

#### l2orderqueue - level2委买委卖一档委托队列

```
'time'                  #时间戳
'bidLevelPrice'         #委买价
'bidLevelVolume'        #委买量
'offerLevelPrice'       #委卖价
'offerLevelVolume'      #委卖量
'bidLevelNumber'        #委买数量
'offLevelNumber'        #委卖数量
```

### 数据字典

#### 证券状态

```
0,10 - 默认为未知
11 - 开盘前S
12 - 集合竞价时段C
13 - 连续交易T
14 - 休市B
15 - 闭市E
16 - 波动性中断V
17 - 临时停牌P
18 - 收盘集合竞价U
19 - 盘中集合竞价M
20 - 暂停交易至闭市N
21 - 获取字段异常
22 - 盘后固定价格行情
23 - 盘后固定价格行情完毕
```

#### 委托类型

- level2逐笔委托 - `entrustType` 委托类型
- level2逐笔成交 - `tradeType` 成交类型

```
0 - 未知
1 - 正常交易业务
2 - 即时成交剩余撤销
3 - ETF基金申报
4 - 最优五档即时成交剩余撤销
5 - 全额成交或撤销
6 - 本方最优价格
7 - 对手方最优价格
```

#### 委托方向

- level2逐笔委托 - `entrustDirection` 委托方向
  - 注：上交所的撤单信息在逐笔委托的委托方向，区分撤买撤卖

```
1 - 买入
2 - 卖出
3 - 撤买（上交所）
4 - 撤卖（上交所）
```

#### 成交标志

- level2逐笔成交 - `tradeFlag` 成交标志
  - 注：深交所的在逐笔成交的成交标志，只有撤单，没有方向

```
0 - 未知
1 - 外盘
2 - 内盘
3 - 撤单（深交所）
```

#### 现金替代标志

- ETF申赎清单成份股现金替代标志

```
0 - 禁止现金替代（必须有股票）
1 - 允许现金替代（先用股票，股票不足的话用现金替代
2 - 必须现金替代
3 - 非沪市（股票）退补现金替代
4 - 非沪市（股票）必须现金替代
5 - 非沪深退补现金替代
6 - 非沪深必须现金替代
7 - 港市退补现金替代（仅适用于跨沪深ETF产品）
8 - 港市必须现金替代（仅适用于跨沪深港ETF产品）
```

### 财务数据字段列表

#### Balance - 资产负债表

```
'm_anntime'                                 #披露日期
'm_timetag'                                 #截止日期
'internal_shoule_recv'                      #内部应收款
'fixed_capital_clearance'                   #固定资产清理
'should_pay_money'                          #应付分保账款
'settlement_payment'                        #结算备付金
'receivable_premium'                        #应收保费
'accounts_receivable_reinsurance'           #应收分保账款
'reinsurance_contract_reserve'              #应收分保合同准备金
'dividends_payable'                         #应收股利
'tax_rebate_for_export'                     #应收出口退税
'subsidies_receivable'                      #应收补贴款
'deposit_receivable'                        #应收保证金
'apportioned_cost'                          #待摊费用
'profit_and_current_assets_with_deal'       #待处理流动资产损益
'current_assets_one_year'                   #一年内到期的非流动资产
'long_term_receivables'                     #长期应收款
'other_long_term_investments'               #其他长期投资
'original_value_of_fixed_assets'            #固定资产原值
'net_value_of_fixed_assets'                 #固定资产净值
'depreciation_reserves_of_fixed_assets'     #固定资产减值准备
'productive_biological_assets'              #生产性生物资产
'public_welfare_biological_assets'          #公益性生物资产
'oil_and_gas_assets'                        #油气资产
'development_expenditure'                   #开发支出
'right_of_split_share_distribution'         #股权分置流通权
'other_non_mobile_assets'                   #其他非流动资产
'handling_fee_and_commission'               #应付手续费及佣金
'other_payables'                            #其他应交款
'margin_payable'                            #应付保证金
'internal_accounts_payable'                 #内部应付款
'advance_cost'                              #预提费用
'insurance_contract_reserve'                #保险合同准备金
'broker_buying_and_selling_securities'      #代理买卖证券款
'acting_underwriting_securities'            #代理承销证券款
'international_ticket_settlement'           #国际票证结算
'domestic_ticket_settlement'                #国内票证结算
'deferred_income'                           #递延收益
'short_term_bonds_payable'                  #应付短期债券
'long_term_deferred_income'                 #长期递延收益
'undetermined_investment_losses'            #未确定的投资损失
'quasi_distribution_of_cash_dividends'      #拟分配现金股利
'provisions_not'                            #预计负债
'cust_bank_dep'                             #吸收存款及同业存放
'provisions'                                #预计流动负债
'less_tsy_stk'                              #减:库存股
'cash_equivalents'                          #货币资金
'loans_to_oth_banks'                        #拆出资金
'tradable_fin_assets'                       #交易性金融资产
'derivative_fin_assets'                     #衍生金融资产
'bill_receivable'                           #应收票据
'account_receivable'                        #应收账款
'advance_payment'                           #预付款项
'int_rcv'                                   #应收利息
'other_receivable'                          #其他应收款
'red_monetary_cap_for_sale'                 #买入返售金融资产
'agency_bus_assets'                         #以公允价值计量且其变动计入当期损益的金融资产
'inventories'                               #存货
'other_current_assets'                      #其他流动资产
'total_current_assets'                      #流动资产合计
'loans_and_adv_granted'                     #发放贷款及垫款
'fin_assets_avail_for_sale'                 #可供出售金融资产
'held_to_mty_invest'                        #持有至到期投资
'long_term_eqy_invest'                      #长期股权投资
'invest_real_estate'                        #投资性房地产
'accumulated_depreciation'                  #累计折旧
'fix_assets'                                #固定资产
'constru_in_process'                        #在建工程
'construction_materials'                    #工程物资
'long_term_liabilities'                     #长期负债
'intang_assets'                             #无形资产
'goodwill'                                  #商誉
'long_deferred_expense'                     #长期待摊费用
'deferred_tax_assets'                       #递延所得税资产
'total_non_current_assets'                  #非流动资产合计
'tot_assets'                                #资产总计
'shortterm_loan'                            #短期借款
'borrow_central_bank'                       #向中央银行借款
'loans_oth_banks'                           #拆入资金
'tradable_fin_liab'                         #交易性金融负债
'derivative_fin_liab'                       #衍生金融负债
'notes_payable'                             #应付票据
'accounts_payable'                          #应付账款
'advance_peceipts'                          #预收账款
'fund_sales_fin_assets_rp'                  #卖出回购金融资产款
'empl_ben_payable'                          #应付职工薪酬
'taxes_surcharges_payable'                  #应交税费
'int_payable'                               #应付利息
'dividend_payable'                          #应付股利
'other_payable'                             #其他应付款
'non_current_liability_in_one_year'         #一年内到期的非流动负债
'other_current_liability'                   #其他流动负债
'total_current_liability'                   #流动负债合计
'long_term_loans'                           #长期借款
'bonds_payable'                             #应付债券
'longterm_account_payable'                  #长期应付款
'grants_received'                           #专项应付款
'deferred_tax_liab'                         #递延所得税负债
'other_non_current_liabilities'             #其他非流动负债
'non_current_liabilities'                   #非流动负债合计
'tot_liab'                                  #负债合计
'cap_stk'                                   #实收资本(或股本)
'cap_rsrv'                                  #资本公积
'specific_reserves'                         #专项储备
'surplus_rsrv'                              #盈余公积
'prov_nom_risks'                            #一般风险准备
'undistributed_profit'                      #未分配利润
'cnvd_diff_foreign_curr_stat'               #外币报表折算差额
'tot_shrhldr_eqy_excl_min_int'              #归属于母公司股东权益合计
'minority_int'                              #少数股东权益
'total_equity'                              #所有者权益合计
'tot_liab_shrhldr_eqy'                      #负债和股东权益总计
```

#### Income - 利润表

```
'm_anntime'                                 #披露日期
'm_timetag'                                 #截止日期
'revenue_inc'                               #营业收入
'earned_premium'                            #已赚保费
'real_estate_sales_income'                  #房地产销售收入
'total_operating_cost'                      #营业总成本
'real_estate_sales_cost'                    #房地产销售成本
'research_expenses'                         #研发费用
'surrender_value'                           #退保金
'net_payments'                              #赔付支出净额
'net_withdrawal_ins_con_res'                #提取保险合同准备金净额
'policy_dividend_expenses'                  #保单红利支出
'reinsurance_cost'                          #分保费用
'change_income_fair_value'                  #公允价值变动收益
'futures_loss'                              #期货损益
'trust_income'                              #托管收益
'subsidize_revenue'                         #补贴收入
'other_business_profits'                    #其他业务利润
'net_profit_excl_merged_int_inc'            #被合并方在合并前实现净利润
'int_inc'                                   #利息收入
'handling_chrg_comm_inc'                    #手续费及佣金收入
'less_handling_chrg_comm_exp'               #手续费及佣金支出
'other_bus_cost'                            #其他业务成本
'plus_net_gain_fx_trans'                    #汇兑收益
'il_net_loss_disp_noncur_asset'             #非流动资产处置收益
'inc_tax'                                   #所得税费用
'unconfirmed_invest_loss'                   #未确认投资损失
'net_profit_excl_min_int_inc'               #归属于母公司所有者的净利润
'less_int_exp'                              #利息支出
'other_bus_inc'                             #其他业务收入
'revenue'                                   #营业总收入
'total_expense'                             #营业成本
'less_taxes_surcharges_ops'                 #营业税金及附加
'sale_expense'                              #销售费用
'less_gerl_admin_exp'                       #管理费用
'financial_expense'                         #财务费用
'less_impair_loss_assets'                   #资产减值损失
'plus_net_invest_inc'                       #投资收益
'incl_inc_invest_assoc_jv_entp'             #联营企业和合营企业的投资收益
'oper_profit'                               #营业利润
'plus_non_oper_rev'                         #营业外收入
'less_non_oper_exp'                         #营业外支出
'tot_profit'                                #利润总额
'net_profit_incl_min_int_inc'               #净利润
'net_profit_incl_min_int_inc_after'         #净利润(扣除非经常性损益后)
'minority_int_inc'                          #少数股东损益
's_fa_eps_basic'                            #基本每股收益
's_fa_eps_diluted'                          #稀释每股收益
'total_income'                              #综合收益总额
'total_income_minority'                     #归属于少数股东的综合收益总额
'other_compreh_inc'                         #其他收益
```

#### CashFlow - 现金流量表

```
'm_anntime'                                 #披露日期
'm_timetag'                                 #截止日期
'cash_received_ori_ins_contract_pre'        #收到原保险合同保费取得的现金
'net_cash_received_rei_ope'                 #收到再保险业务现金净额
'net_increase_insured_funds'                #保户储金及投资款净增加额
'Net'                                       #处置交易性金融资产净增加额 increase_in_disposal
'cash_for_interest'                         #收取利息、手续费及佣金的现金
'net_increase_in_repurchase_funds'          #回购业务资金净增加额
'cash_for_payment_original_insurance'       #支付原保险合同赔付款项的现金
'cash_payment_policy_dividends'             #支付保单红利的现金
'disposal_other_business_units'             #处置子公司及其他收到的现金
'cash_received_from_pledges'                #减少质押和定期存款所收到的现金
'cash_paid_for_investments'                 #投资所支付的现金
'net_increase_in_pledged_loans'             #质押贷款净增加额
'cash_paid_by_subsidiaries'                 #取得子公司及其他营业单位支付的现金净额
'increase_in_cash_paid'                     #增加质押和定期存款所支付的现金 
'cass_received_sub_abs'                     #其中子公司吸收现金
'cass_received_sub_investments'             #其中:子公司支付给少数股东的股利、利润
'minority_shareholder_profit_loss'          #少数股东损益
'unrecognized_investment_losses'            #未确认的投资损失
'ncrease_deferred_income'                   #递延收益增加(减:减少)
'projected_liability'                       #预计负债
'increase_operational_payables'             #经营性应付项目的增加
'reduction_outstanding_amounts_less'        #已完工尚未结算款的减少(减:增加)
'reduction_outstanding_amounts_more'        #已结算尚未完工款的增加(减:减少)
'goods_sale_and_service_render_cash'        #销售商品、提供劳务收到的现金
'net_incr_dep_cob'                          #客户存款和同业存放款项净增加额
'net_incr_loans_central_bank'               #向中央银行借款净增加额(万元
'net_incr_fund_borr_ofi'                    #向其他金融机构拆入资金净增加额
'net_incr_fund_borr_ofi'                    #拆入资金净增加额
'tax_levy_refund'                           #收到的税费与返还
'cash_paid_invest'                          #投资支付的现金
'other_cash_recp_ral_oper_act'              #收到的其他与经营活动有关的现金
'stot_cash_inflows_oper_act'                #经营活动现金流入小计
'goods_and_services_cash_paid'              #购买商品、接受劳务支付的现金
'net_incr_clients_loan_adv'                 #客户贷款及垫款净增加额
'net_incr_dep_cbob'                         #存放中央银行和同业款项净增加额
'handling_chrg_paid'                        #支付利息、手续费及佣金的现金
'cash_pay_beh_empl'                         #支付给职工以及为职工支付的现金
'pay_all_typ_tax'                           #支付的各项税费
'other_cash_pay_ral_oper_act'               #支付其他与经营活动有关的现金
'stot_cash_outflows_oper_act'               #经营活动现金流出小计
'net_cash_flows_oper_act'                   #经营活动产生的现金流量净额
'cash_recp_disp_withdrwl_invest'            #收回投资所收到的现金
'cash_recp_return_invest'                   #取得投资收益所收到的现金
'net_cash_recp_disp_fiolta'                 #处置固定资产、无形资产和其他长期投资收到的现金
'other_cash_recp_ral_inv_act'               #收到的其他与投资活动有关的现金
'stot_cash_inflows_inv_act'                 #投资活动现金流入小计
'cash_pay_acq_const_fiolta'                 #购建固定资产、无形资产和其他长期投资支付的现金
'other_cash_pay_ral_oper_act'               #支付其他与投资的现金
'stot_cash_outflows_inv_act'                #投资活动现金流出小计
'net_cash_flows_inv_act'                    #投资活动产生的现金流量净额
'cash_recp_cap_contrib'                     #吸收投资收到的现金
'cash_recp_borrow'                          #取得借款收到的现金
'proc_issue_bonds'                          #发行债券收到的现金
'other_cash_recp_ral_fnc_act'               #收到其他与筹资活动有关的现金
'stot_cash_inflows_fnc_act'                 #筹资活动现金流入小计
'cash_prepay_amt_borr'                      #偿还债务支付现金
'cash_pay_dist_dpcp_int_exp'                #分配股利、利润或偿付利息支付的现金
'other_cash_pay_ral_fnc_act'                #支付其他与筹资的现金
'stot_cash_outflows_fnc_act'                #筹资活动现金流出小计
'net_cash_flows_fnc_act'                    #筹资活动产生的现金流量净额
'eff_fx_flu_cash'                           #汇率变动对现金的影响
'net_incr_cash_cash_equ'                    #现金及现金等价物净增加额
'cash_cash_equ_beg_period'                  #期初现金及现金等价物余额
'cash_cash_equ_end_period'                  #期末现金及现金等价物余额
'net_profit'                                #净利润
'plus_prov_depr_assets'                     #资产减值准备
'depr_fa_coga_dpba'                         #固定资产折旧、油气资产折耗、生产性物资折旧
'amort_intang_assets'                       #无形资产摊销
'amort_lt_deferred_exp'                     #长期待摊费用摊销
'decr_deferred_exp'                         #待摊费用的减少
'incr_acc_exp'                              #预提费用的增加
'loss_disp_fiolta'                          #处置固定资产、无形资产和其他长期资产的损失
'loss_scr_fa'                               #固定资产报废损失
'loss_fv_chg'                               #公允价值变动损失
'fin_exp'                                   #财务费用
'invest_loss'                               #投资损失
'decr_deferred_inc_tax_assets'              #递延所得税资产减少
'incr_deferred_inc_tax_liab'                #递延所得税负债增加
'decr_inventories'                          #存货的减少
'decr_oper_payable'                         #经营性应收项目的减少
'others'                                    #其他
'im_net_cash_flows_oper_act'                #经营活动产生现金流量净额
'conv_debt_into_cap'                        #债务转为资本
'conv_corp_bonds_due_within_1y'             #一年内到期的可转换公司债券
'fa_fnc_leases'                             #融资租入固定资产
'end_bal_cash'                              #现金的期末余额
'less_beg_bal_cash'                         #现金的期初余额
'plus_end_bal_cash_equ'                     #现金等价物的期末余额
'less_beg_bal_cash_equ'                     #现金等价物的期初余额
'im_net_incr_cash_cash_equ'                 #现金及现金等价物的净增加额
'tax_levy_refund'                           #收到的税费返还
```

#### PershareIndex - 主要指标

```
's_fa_ocfps'                                #每股经营活动现金流量
's_fa_bps'                                  #每股净资产
's_fa_eps_basic'                            #基本每股收益
's_fa_eps_diluted'                          #稀释每股收益
's_fa_undistributedps'                      #每股未分配利润
's_fa_surpluscapitalps'                     #每股资本公积金
'adjusted_earnings_per_share'               #扣非每股收益
'du_return_on_equity'                       #净资产收益率
'sales_gross_profit'                        #销售毛利率
'inc_revenue_rate'                          #主营收入同比增长
'du_profit_rate'                            #净利润同比增长
'inc_net_profit_rate'                       #归属于母公司所有者的净利润同比增长
'adjusted_net_profit_rate'                  #扣非净利润同比增长
'inc_total_revenue_annual'                  #营业总收入滚动环比增长
'inc_net_profit_to_shareholders_annual'     #归属净利润滚动环比增长
'adjusted_profit_to_profit_annual'          #扣非净利润滚动环比增长
'equity_roe'                                #加权净资产收益率
'net_roe'                                   #摊薄净资产收益率
'total_roe'                                 #摊薄总资产收益率
'gross_profit'                              #毛利率
'net_profit'                                #净利率
'actual_tax_rate'                           #实际税率
'pre_pay_operate_income'                    #预收款 / 营业收入
'sales_cash_flow'                           #销售现金流 / 营业收入
'gear_ratio'                                #资产负债比率
'inventory_turnover'                        #存货周转率
'm_anntime'                                 #公告日
'm_timetag'                                 #报告截止日
```

#### Capital - 股本表

```
'total_capital'                             #总股本
'circulating_capital'                       #已上市流通A股
'restrict_circulating_capital'              #限售流通股份
'm_timetag'                                 #报告截止日
'm_anntime'                                 #公告日
```

#### Top10holder/Top10flowholder - 十大股东/十大流通股东

```
'declareDate'                                #公告日期
'endDate'                                    #截止日期
'name'                                       #股东名称
'type'                                       #股东类型
'quantity'                                   #持股数量
'reason'                                     #变动原因
'ratio'                                      #持股比例
'nature'                                     #股份性质
'rank'                                       #持股排名
```

#### Holdernum - 股东数

```
'declareDate'                                 #公告日期
'endDate'                                     #截止日期
'shareholder'                                 #股东总数
'shareholderA'                                #A股东户数
'shareholderB'                                #B股东户数
'shareholderH'                                #H股东户数
'shareholderFloat'                            #已流通股东户数
'shareholderOther'                            #未流通股东户数
```

### 合约信息字段列表

```
'ExchangeID'                 #合约市场代码
'InstrumentID'                 #合约代码
'InstrumentName'             #合约名称
'Abbreviation'                 #合约名称的拼音简写
'ProductID'                 #合约的品种ID（期货）
'ProductName'                 #合约的品种名称（期货）
'UnderlyingCode'             #标的合约
'ExtendName'                 #扩位名称
'ExchangeCode'                 #交易所代码
'RzrkCode'                     #rzrk代码
'UniCode'                     #统一规则代码
'CreateDate'                 #上市日期（期货）
'OpenDate'                     #IPO日期（股票）
'ExpireDate'                 #退市日或者到期日
'PreClose'                     #前收盘价格
'SettlementPrice'             #前结算价格
'UpStopPrice'                 #当日涨停价
'DownStopPrice'             #当日跌停价
'FloatVolume'                 #流通股本
'TotalVolume'                 #总股本
'AccumulatedInterest'         #自上市付息日起的累积未付利息额（债券）
'LongMarginRatio'             #多头保证金率
'ShortMarginRatio'             #空头保证金率
'PriceTick'                 #最小变价单位
'VolumeMultiple'             #合约乘数（对期货以外的品种，默认是1）
'MainContract'                 #主力合约标记，1、2、3分别表示第一主力合约，第二主力合约，第三主力合约
'MaxMarketOrderVolume'         #市价单最大下单量
'MinMarketOrderVolume'         #市价单最小下单量
'MaxLimitOrderVolume'         #限价单最大下单量
'MinLimitOrderVolume'         #限价单最小下单量
'MaxMarginSideAlgorithm'     #上期所大单边的处理算法
'DayCountFromIPO'             #自IPO起经历的交易日总数
'LastVolume'                 #昨日持仓量
'InstrumentStatus'             #合约停牌状态
'IsTrading'                 #合约是否可交易
'IsRecent'                     #是否是近月合约
'IsContinuous'                 #是否是连续合约
'bNotProfitable'             #是否非盈利状态
'bDualClass'                 #是否同股不同权
'ContinueType'                 #连续合约类型
'secuCategory'                 #证券分类
'secuAttri'                 #证券属性
'MaxMarketSellOrderVolume'     #市价卖单最大单笔下单量
'MinMarketSellOrderVolume'     #市价卖单最小单笔下单量
'MaxLimitSellOrderVolume'     #限价卖单最大单笔下单量
'MinLimitSellOrderVolume'     #限价卖单最小单笔下单量
'MaxFixedBuyOrderVol'         #盘后定价委托数量的上限（买）
'MinFixedBuyOrderVol'         #盘后定价委托数量的下限（买）
'MaxFixedSellOrderVol'         #盘后定价委托数量的上限（卖）
'MinFixedSellOrderVol'         #盘后定价委托数量的下限（卖）
'HSGTFlag'                     #标识港股是否为沪港通或深港通标的证券。沪港通:0-非标的，1-标的，2-历史标的；深港通:0-非标的，3-标的，4-历史标的，5-是沪港通也是深港通
'BondParValue'                 #债券面值
'QualifiedType'             #投资者适当性管理分类
'PriceTickType'             #价差类别（港股用），1-股票，3-债券，4-期权，5-交易所买卖基金
'tradingStatus'             #交易状态
'OptUnit'                     #期权合约单位
'MarginUnit'                 #期权单位保证金
'OptUndlCode'                 #期权标的证券代码或可转债正股标的证券代码
'OptUndlMarket'             #期权标的证券市场或可转债正股标的证券市场
'OptLotSize'                 #期权整手数
'OptExercisePrice'             #期权行权价或可转债转股价
'NeeqExeType'                 #全国股转转让类型，1-协议转让方式，2-做市转让方式，3-集合竞价+连续竞价转让方式（当前全国股转并未实现），4-集合竞价转让
'OptExchFixedMargin'         #交易所期权合约保证金不变部分
'OptExchMiniMargin'         #交易所期权合约最小保证金
'Ccy'                         #币种
'IbSecType'                 #IB安全类型，期货或股票
'OptUndlRiskFreeRate'         #期权标的无风险利率
'OptUndlHistoryRate'         #期权标的历史波动率
'EndDelivDate'                 #期权行权终止日
'RegisteredCapital'         #注册资本（单位:百万）
'MaxOrderPriceRange'         #最大有效申报范围
'MinOrderPriceRange'         #最小有效申报范围
'VoteRightRatio'             #同股同权比例
'm_nMinRepurchaseDaysLimit' #最小回购天数
'm_nMaxRepurchaseDaysLimit' #最大回购天数
'DeliveryYear'                 #交割年份
'DeliveryMonth'             #交割月
'ContractType'                 #标识期权，1-过期，2-当月，3-下月，4-下季，5-隔季，6-隔下季
'ProductTradeQuota'         #期货品种交易配额
'ContractTradeQuota'         #期货合约交易配额
'ProductOpenInterestQuota'     #期货品种持仓配额
'ContractOpenInterestQuota' #期货合约持仓配额
'ChargeType'                 #期货和期权手续费方式，0-未知，1-按元/手，2-按费率
'ChargeOpen'                 #开仓手续费率，-1表示没有
'ChargeClose'                 #平仓手续费率，-1表示没有
'ChargeClose'                #平仓手续费率，-1表示没有
'ChargeTodayOpen'            #开今仓（日内开仓）手续费率，-1表示没有
'ChargeTodayClose'            #平今仓（日内平仓）手续费率，-1表示没有
'OptionType'                #期权类型，-1为非期权，0为期权认购，1为期权认沽
'OpenInterestMultiple'        #交割月持仓倍数
```

### 代码示例

#### 时间戳转换

```
import time
def conv_time(ct):
    '''
    conv_time(1476374400000) --> '20161014000000.000'
    '''
    local_time = time.localtime(ct / 1000)
    data_head = time.strftime('%Y%m%d%H%M%S', local_time)
    data_secs = (ct - int(ct)) * 1000
    time_stamp = '%s.%03d' % (data_head, data_secs)
    return time_stamp
```

上次更新: 2

# XtQuant.Xttrade 交易模块

## 版本信息

- 2020-09-01
  
  - 初稿

- 2020-10-14
  
  - 持仓结构添加字段
  - 投资备注相关修正

- 2020-10-21
  
  - 添加信用交易相关委托类型（order_type）枚举
  - 调整XtQuant运行依赖环境说明，更新多版本支持相关说明

- 2020-11-13
  
  - 添加信用交易相关类型定义说明
  - 添加信用交易相关接口说明
  - 添加异步撤单委托反馈结构说明
  - 添加下单失败和撤单失败主推结构说明
  - 添加订阅和反订阅接口
  - 添加创建API实例，注册回调类，准备API环境，创建连接，停止运行，阻塞进程接口说明
  - 调整API接口说明
    - 将接口细分为"系统设置接口"，“操作接口”，“查询接口”，"信用相关查询接口"，“回调类”等五类
    - 接口返回“None”修改为“无”
    - 去掉回调类接口中的示例
    - 添加“备注”项
  - 所有“证券账号”改为“资金账号”
  - 英文“,”调整为中文“，”
  - 示例代码中增加XtQuant API实例对象，修正没有实例，直接调用的错误
  - 添加股票异步撤单接口说明，将原股票撤单修改为股票同步撤单

- 2020-11-19
  
  - 添加账号状态主推接口
  - 添加账号状态数据结构说明
  - 添加账号状态枚举值
  - 回调类接口说明调整
    - 将回调函数定义及函数说明标题调整一致
    - 补充异步下单回报推送、异步撤单回报推送接口说明

- 2021-07-20
  
  - 修改回调/主推函数实现机制，提升报撤单回报的速度，降低穿透延时波动
  - `XtQuantTrader.run_forever()`修改实现，支持`Ctrl+C`跳出

- 2022-06-27
  
  - 委托查询支持仅查询可撤委托
  - 添加新股申购相关接口
    - `query_new_purchase_limit` 查询新股申购额度
    - `query_ipo_data` 查询新股信息
  - 添加账号信息查询接口
    - `query_account_infos`

- 2022-11-15
  
  - 修复`XtQuantTrader.unsubscribe`的实现

- 2022-11-17
  
  - 交易数据字典格式调整

- 2022-11-28
  
  - 为主动请求接口的返回增加专用线程以及相关控制，以支持在on_stock_order等推送接口中调用同步请求
    - `XtQuantTrader.set_relaxed_response_order_enabled`

- 2023-07-17
  
  - 持仓结构`XtPosition` 成本价字段调整
    - `open_price` - 开仓价
    - `avg_price` - 成本价

- 2023-07-26
  
  - 添加资金划拨接口 `fund_transfer`

- 2023-08-11
  
  - 添加划拨业务查询普通柜台资金接口 `query_com_fund`
  - 添加划拨业务查询普通柜台持仓接口 `query_com_position`

- 2023-10-16
  
  - 添加期货市价的报价类型
    - `xtconstant.MARKET_BEST` - 市价最优价[郑商所]
    - `xtconstant.MARKET_CANCEL` - 市价即成剩撤[大商所]
    - `xtconstant.MARKET_CANCEL_ALL` - 市价全额成交或撤[大商所]
    - `xtconstant.MARKET_CANCEL_1` - 市价最优一档即成剩撤[中金所]
    - `xtconstant.MARKET_CANCEL_5` - 市价最优五档即成剩撤[中金所]
    - `xtconstant.MARKET_CONVERT_1` - 市价最优一档即成剩转[中金所]
    - `xtconstant.MARKET_CONVERT_5` - 市价最优五档即成剩转[中金所]

- 2023-10-20

- 委托结构`XtOrder`，成交结构`XtTrade`，持仓结构`XtPosition` 新增多空字段
  
  - `direction` - 多空，股票不需要

- 委托结构`XtOrder`，成交结构`XtTrade`新增交易操作字段
  
  - `offset_flag` - 交易操作，用此字段区分股票买卖，期货开、平仓，期权买卖等

- 2023-11-03
  
  - 添加券源行情查询接口 `smt_query_quoter`
  - 添加库存券约券申请接口 `smt_negotiate_order`
  - 添加约券合约查询接口 `smt_query_compact`

- 2024-01-02
  
  - 委托类型增加ETF申赎

- 2024-02-29
  
  - 添加期货持仓统计查询接口`query_position_statistics`

- 2024-04-25
  
  - 数据结构添加`stock_code1`字段以适配长代码

- 2024-05-24
  
  - 添加通用数据导出接口export_data
  - 添加通用数据查询接口query_data

- 2024-06-27
  
  - 添加外部成交导入接口sync_transaction_from_external

## 快速入门

### 创建策略

```
#coding=utf-8
from xtquant.xttrader import XtQuantTrader, XtQuantTraderCallback
from xtquant.xttype import StockAccount
from xtquant import xtconstant


class MyXtQuantTraderCallback(XtQuantTraderCallback):
    def on_disconnected(self):
        """
        连接断开
        :return:
        """
        print("connection lost")
    def on_stock_order(self, order):
        """
        委托回报推送
        :param order: XtOrder对象
        :return:
        """
        print("on order callback:")
        print(order.stock_code, order.order_status, order.order_sysid)
    def on_stock_trade(self, trade):
        """
        成交变动推送
        :param trade: XtTrade对象
        :return:
        """
        print("on trade callback")
        print(trade.account_id, trade.stock_code, trade.order_id)
    def on_order_error(self, order_error):
        """
        委托失败推送
        :param order_error:XtOrderError 对象
        :return:
        """
        print("on order_error callback")
        print(order_error.order_id, order_error.error_id, order_error.error_msg)
    def on_cancel_error(self, cancel_error):
        """
        撤单失败推送
        :param cancel_error: XtCancelError 对象
        :return:
        """
        print("on cancel_error callback")
        print(cancel_error.order_id, cancel_error.error_id, cancel_error.error_msg)
    def on_order_stock_async_response(self, response):
        """
        异步下单回报推送
        :param response: XtOrderResponse 对象
        :return:
        """
        print("on_order_stock_async_response")
        print(response.account_id, response.order_id, response.seq)
    def on_account_status(self, status):
        """
        :param response: XtAccountStatus 对象
        :return:
        """
        print("on_account_status")
        print(status.account_id, status.account_type, status.status)

if __name__ == "__main__":
    print("demo test")
    # path为mini qmt客户端安装目录下userdata_mini路径
    path = 'D:\\迅投极速交易终端 睿智融科版\\userdata_mini'
    # session_id为会话编号，策略使用方对于不同的Python策略需要使用不同的会话编号
    session_id = 123456
    xt_trader = XtQuantTrader(path, session_id)
    # 创建资金账号为1000000365的证券账号对象
    acc = StockAccount('1000000365')
    # StockAccount可以用第二个参数指定账号类型，如沪港通传'HUGANGTONG'，深港通传'SHENGANGTONG'
    # acc = StockAccount('1000000365','STOCK')
    # 创建交易回调类对象，并声明接收回调
    callback = MyXtQuantTraderCallback()
    xt_trader.register_callback(callback)
    # 启动交易线程
    xt_trader.start()
    # 建立交易连接，返回0表示连接成功
    connect_result = xt_trader.connect()
    print(connect_result)
    # 对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功
    subscribe_result = xt_trader.subscribe(acc)
    print(subscribe_result)
    stock_code = '600000.SH'
    # 使用指定价下单，接口返回订单编号，后续可以用于撤单操作以及查询委托状态
    print("order using the fix price:")
    fix_result_order_id = xt_trader.order_stock(acc, stock_code, xtconstant.STOCK_BUY, 200, xtconstant.FIX_PRICE, 10.5, 'strategy_name', 'remark')
    print(fix_result_order_id)
    # 使用订单编号撤单
    print("cancel order:")
    cancel_order_result = xt_trader.cancel_order_stock(acc, fix_result_order_id)
    print(cancel_order_result)
    # 使用异步下单接口，接口返回下单请求序号seq，seq可以和on_order_stock_async_response的委托反馈response对应起来
    print("order using async api:")
    async_seq = xt_trader.order_stock_async(acc, stock_code, xtconstant.STOCK_BUY, 200, xtconstant.FIX_PRICE, 10.5, 'strategy_name', 'remark')
    print(async_seq)
    # 查询证券资产
    print("query asset:")
    asset = xt_trader.query_stock_asset(acc)
    if asset:
        print("asset:")
        print("cash {0}".format(asset.cash))
    # 根据订单编号查询委托
    print("query order:")
    order = xt_trader.query_stock_order(acc, fix_result_order_id)
    if order:
        print("order:")
        print("order {0}".format(order.order_id))
    # 查询当日所有的委托
    print("query orders:")
    orders = xt_trader.query_stock_orders(acc)
    print("orders:", len(orders))
    if len(orders) != 0:
        print("last order:")
        print("{0} {1} {2}".format(orders[-1].stock_code, orders[-1].order_volume, orders[-1].price))
    # 查询当日所有的成交
    print("query trade:")
    trades = xt_trader.query_stock_trades(acc)
    print("trades:", len(trades))
    if len(trades) != 0:
        print("last trade:")
        print("{0} {1} {2}".format(trades[-1].stock_code, trades[-1].traded_volume, trades[-1].traded_price))
    # 查询当日所有的持仓
    print("query positions:")
    positions = xt_trader.query_stock_positions(acc)
    print("positions:", len(positions))
    if len(positions) != 0:
        print("last position:")
        print("{0} {1} {2}".format(positions[-1].account_id, positions[-1].stock_code, positions[-1].volume))
    # 根据股票代码查询对应持仓
    print("query position:")
    position = xt_trader.query_stock_position(acc, stock_code)
    if position:
        print("position:")
        print("{0} {1} {2}".format(position.account_id, position.stock_code, position.volume))
    # 阻塞线程，接收交易推送
    xt_trader.run_forever()
```

# 进阶篇

## XtQuant运行逻辑

XtQuant封装了策略交易所需要的Python API接口，可以和MiniQMT客户端交互进行报单、撤单、查询资产、查询委托、查询成交、查询持仓以及收到资金、委托、成交和持仓等变动的主推消息。

## XtQuant数据字典

### 交易市场(market)

- 上交所 - `xtconstant.SH_MARKET`
- 深交所 - `xtconstant.SZ_MARKET`
- 北交所 - `xtconstant.MARKET_ENUM_BEIJING`
- 沪港通 - `xtconstant.MARKET_ENUM_SHANGHAI_HONGKONG_STOCK`
- 深港通 - `xtconstant.MARKET_ENUM_SHENZHEN_HONGKONG_STOCK`
- 上期所 - `xtconstant.MARKET_ENUM_SHANGHAI_FUTURE`
- 大商所 - `xtconstant.MARKET_ENUM_DALIANG_FUTURE`
- 郑商所 - `xtconstant.MARKET_ENUM_ZHENGZHOU_FUTURE`
- 中金所 - `xtconstant.MARKET_ENUM_INDEX_FUTURE`
- 能源中心 - `xtconstant.MARKET_ENUM_INTL_ENERGY_FUTURE`
- 广期所 - `xtconstant.MARKET_ENUM_GUANGZHOU_FUTURE`
- 上海期权 - `xtconstant.MARKET_ENUM_SHANGHAI_STOCK_OPTION`
- 深证期权 - `xtconstant.MARKET_ENUM_SHENZHEN_STOCK_OPTION`

### 账号类型(account_type)

- 期货 - `xtconstant.FUTURE_ACCOUNT`
- 股票 - `xtconstant.SECURITY_ACCOUNT`
- 信用 - `xtconstant.CREDIT_ACCOUNT`
- 期货期权 - `xtconstant.FUTURE_OPTION_ACCOUNT`
- 股票期权 - `xtconstant.STOCK_OPTION_ACCOUNT`
- 沪港通 - `xtconstant.HUGANGTONG_ACCOUNT`
- 深港通 - `xtconstant.SHENGANGTONG_ACCOUNT`

### 委托类型(order_type)

- 股票
  
  - 买入 - `xtconstant.STOCK_BUY`
  - 卖出 - `xtconstant.STOCK_SELL`

- 信用
  
  - 担保品买入 - `xtconstant.CREDIT_BUY`
  - 担保品卖出 - `xtconstant.CREDIT_SELL`
  - 融资买入 - `xtconstant.CREDIT_FIN_BUY`
  - 融券卖出 - `xtconstant.CREDIT_SLO_SELL`
  - 买券还券 - `xtconstant.CREDIT_BUY_SECU_REPAY`
  - 直接还券 - `xtconstant.CREDIT_DIRECT_SECU_REPAY`
  - 卖券还款 - `xtconstant.CREDIT_SELL_SECU_REPAY`
  - 直接还款 - `xtconstant.CREDIT_DIRECT_CASH_REPAY`
  - 专项融资买入 - `xtconstant.CREDIT_FIN_BUY_SPECIAL`
  - 专项融券卖出 - `xtconstant.CREDIT_SLO_SELL_SPECIAL`
  - 专项买券还券 - `xtconstant.CREDIT_BUY_SECU_REPAY_SPECIAL`
  - 专项直接还券 - `xtconstant.CREDIT_DIRECT_SECU_REPAY_SPECIAL`
  - 专项卖券还款 - `xtconstant.CREDIT_SELL_SECU_REPAY_SPECIAL`
  - 专项直接还款 - `xtconstant.CREDIT_DIRECT_CASH_REPAY_SPECIAL`

- 期货六键风格
  
  - 开多 - `xtconstant.FUTURE_OPEN_LONG`
  - 平昨多 - `xtconstant.FUTURE_CLOSE_LONG_HISTORY`
  - 平今多 - `xtconstant.FUTURE_CLOSE_LONG_TODAY`
  - 开空 - `xtconstant.FUTURE_OPEN_SHORT`
  - 平昨空 - `xtconstant.FUTURE_CLOSE_SHORT_HISTORY`
  - 平今空 - `xtconstant.FUTURE_CLOSE_SHORT_TODAY`

- 期货四键风格
  
  - 平多，优先平今 - `xtconstant.FUTURE_CLOSE_LONG_TODAY_FIRST`
  - 平多，优先平昨 - `xtconstant.FUTURE_CLOSE_LONG_HISTORY_FIRST`
  - 平空，优先平今 - `xtconstant.FUTURE_CLOSE_SHORT_TODAY_FIRST`
  - 平空，优先平昨 - `xtconstant.FUTURE_CLOSE_SHORT_HISTORY_FIRST`

- 期货两键风格
  
  - 卖出，如有多仓，优先平仓，优先平今，如有余量，再开空 - `xtconstant.FUTURE_CLOSE_LONG_TODAY_HISTORY_THEN_OPEN_SHORT`
  - 卖出，如有多仓，优先平仓，优先平昨，如有余量，再开空 - `xtconstant.FUTURE_CLOSE_LONG_HISTORY_TODAY_THEN_OPEN_SHORT`
  - 买入，如有空仓，优先平仓，优先平今，如有余量，再开多 - `xtconstant.FUTURE_CLOSE_SHORT_TODAY_HISTORY_THEN_OPEN_LONG`
  - 买入，如有空仓，优先平仓，优先平昨，如有余量，再开多 - `xtconstant.FUTURE_CLOSE_SHORT_HISTORY_TODAY_THEN_OPEN_LONG`
  - 买入，不优先平仓 - `xtconstant.FUTURE_OPEN`
  - 卖出，不优先平仓 - `xtconstant.FUTURE_CLOSE`

- 期货 - 跨商品套利
  
  - 开仓 - `xtconstant.FUTURE_ARBITRAGE_OPEN`
  - 平, 优先平昨 - `xtconstant.FUTURE_ARBITRAGE_CLOSE_HISTORY_FIRST`
  - 平, 优先平今 - `xtconstant.FUTURE_ARBITRAGE_CLOSE_TODAY_FIRST`

- 期货展期
  
  - 看多, 优先平昨 - `xtconstant.FUTURE_RENEW_LONG_CLOSE_HISTORY_FIRST`
  - 看多，优先平今 - `xtconstant.FUTURE_RENEW_LONG_CLOSE_TODAY_FIRST`
  - 看空，优先平昨 - `xtconstant.FUTURE_RENEW_SHORT_CLOSE_HISTORY_FIRST`
  - 看空，优先平今 - `xtconstant.FUTURE_RENEW_SHORT_CLOSE_TODAY_FIRST`

- 股票期权
  
  - 买入开仓，以下用于个股期权交易业务 - `xtconstant.STOCK_OPTION_BUY_OPEN`
  - 卖出平仓 - `xtconstant.STOCK_OPTION_SELL_CLOSE`
  - 卖出开仓 - `xtconstant.STOCK_OPTION_SELL_OPEN`
  - 买入平仓 - `xtconstant.STOCK_OPTION_BUY_CLOSE`
  - 备兑开仓 - `xtconstant.STOCK_OPTION_COVERED_OPEN`
  - 备兑平仓 - `xtconstant.STOCK_OPTION_COVERED_CLOSE`
  - 认购行权 - `xtconstant.STOCK_OPTION_CALL_EXERCISE`
  - 认沽行权 - `xtconstant.STOCK_OPTION_PUT_EXERCISE`
  - 证券锁定 - `xtconstant.STOCK_OPTION_SECU_LOCK`
  - 证券解锁 - `xtconstant.STOCK_OPTION_SECU_UNLOCK`

- 期货期权
  
  - 期货期权行权 - `xtconstant.OPTION_FUTURE_OPTION_EXERCISE`

- ETF申赎
  
  - 申购 - `xtconstant.ETF_PURCHASE`
  - 赎回 - `xtconstant.ETF_REDEMPTION`

### 报价类型(price_type)

提示

1. 市价类型只在实盘环境中生效，模拟环境不支持市价方式报单
- 最新价 - `xtconstant.LATEST_PRICE`
- 指定价 - `xtconstant.FIX_PRICE`
- 郑商所 期货
  - 市价最优价 - `xtconstant.MARKET_BEST`
- 大商所 期货
  - 市价即成剩撤 - `xtconstant.MARKET_CANCEL`
  - 市价全额成交或撤 - `xtconstant.MARKET_CANCEL_ALL`
- 中金所 期货
  - 市价最优一档即成剩撤 - `xtconstant.MARKET_CANCEL_1`
  - 市价最优五档即成剩撤 - `xtconstant.MARKET_CANCEL_5`
  - 市价最优一档即成剩转 - `xtconstant.MARKET_CONVERT_1`
  - 市价最优五档即成剩转 - `xtconstant.MARKET_CONVERT_5`
- 上交所/北交所 股票
  - 最优五档即时成交剩余撤销 - `xtconstant.MARKET_SH_CONVERT_5_CANCEL`
  - 最优五档即时成交剩转限价 - `xtconstant.MARKET_SH_CONVERT_5_LIMIT`
  - 对手方最优价格委托 - `xtconstant.MARKET_PEER_PRICE_FIRST`
  - 本方最优价格委托 - `xtconstant.MARKET_MINE_PRICE_FIRST`
- 深交所 股票 期权
  - 对手方最优价格委托 - `xtconstant.MARKET_PEER_PRICE_FIRST`
  - 本方最优价格委托 - `xtconstant.MARKET_MINE_PRICE_FIRST`
  - 即时成交剩余撤销委托 - `xtconstant.MARKET_SZ_INSTBUSI_RESTCANCEL`
  - 最优五档即时成交剩余撤销 - `xtconstant.MARKET_SZ_CONVERT_5_CANCEL`
  - 全额成交或撤销委托 - `xtconstant.MARKET_SZ_FULL_OR_CANCEL`

### 委托状态(order_status)

| 枚举变量名                            | 值   | 含义                   |
| -------------------------------- | --- | -------------------- |
| xtconstant.ORDER_UNREPORTED      | 48  | 未报                   |
| xtconstant.ORDER_WAIT_REPORTING  | 49  | 待报                   |
| xtconstant.ORDER_REPORTED        | 50  | 已报                   |
| xtconstant.ORDER_REPORTED_CANCEL | 51  | 已报待撤                 |
| xtconstant.ORDER_PARTSUCC_CANCEL | 52  | 部成待撤                 |
| xtconstant.ORDER_PART_CANCEL     | 53  | 部撤（已经有一部分成交，剩下的已经撤单） |
| xtconstant.ORDER_CANCELED        | 54  | 已撤                   |
| xtconstant.ORDER_PART_SUCC       | 55  | 部成（已经有一部分成交，剩下的待成交）  |
| xtconstant.ORDER_SUCCEEDED       | 56  | 已成                   |
| xtconstant.ORDER_JUNK            | 57  | 废单                   |
| xtconstant.ORDER_UNKNOWN         | 255 | 未知                   |

### 账号状态(account_status)

| 枚举变量名                                   | 值   | 含义                |
| --------------------------------------- | --- | ----------------- |
| xtconstant.ACCOUNT_STATUS_INVALID       | -1  | 无效                |
| xtconstant.ACCOUNT_STATUS_OK            | 0   | 正常                |
| xtconstant.ACCOUNT_STATUS_WAITING_LOGIN | 1   | 连接中               |
| xtconstant.ACCOUNT_STATUSING            | 2   | 登陆中               |
| xtconstant.ACCOUNT_STATUS_FAIL          | 3   | 失败                |
| xtconstant.ACCOUNT_STATUS_INITING       | 4   | 初始化中              |
| xtconstant.ACCOUNT_STATUS_CORRECTING    | 5   | 数据刷新校正中           |
| xtconstant.ACCOUNT_STATUS_CLOSED        | 6   | 收盘后               |
| xtconstant.ACCOUNT_STATUS_ASSIS_FAIL    | 7   | 穿透副链接断开           |
| xtconstant.ACCOUNT_STATUS_DISABLEBYSYS  | 8   | 系统停用（总线使用-密码错误超限） |
| xtconstant.ACCOUNT_STATUS_DISABLEBYUSER | 9   | 用户停用（总线使用）        |

### 划拨方向(transfer_direction)

| 枚举变量名                                     | 值   | 含义               |
| ----------------------------------------- | --- | ---------------- |
| xtconstant.FUNDS_TRANSFER_NORMAL_TO_SPEED | 510 | 资金划拨-普通柜台到极速柜台   |
| xtconstant.FUNDS_TRANSFER_SPEED_TO_NORMAL | 511 | 资金划拨-极速柜台到普通柜台   |
| xtconstant.NODE_FUNDS_TRANSFER_SH_TO_SZ   | 512 | 节点资金划拨-上海节点到深圳节点 |
| xtconstant.NODE_FUNDS_TRANSFER_SZ_TO_SH   | 513 | 节点资金划拨-深圳节点到上海节点 |

### 多空方向(direction)

| 枚举变量名                           | 值   | 含义  |
| ------------------------------- | --- | --- |
| xtconstant.DIRECTION_FLAG_LONG  | 48  | 多   |
| xtconstant.DIRECTION_FLAG_SHORT | 49  | 空   |

### 交易操作(offset_flag)

| 枚举变量名                                  | 值   | 含义    |
| -------------------------------------- | --- | ----- |
| xtconstant.OFFSET_FLAG_OPEN            | 48  | 买入，开仓 |
| xtconstant.OFFSET_FLAG_CLOSE           | 49  | 卖出，平仓 |
| xtconstant.OFFSET_FLAG_FORCECLOSE      | 50  | 强平    |
| xtconstant.OFFSET_FLAG_CLOSETODAY      | 51  | 平今    |
| xtconstant.OFFSET_FLAG_ClOSEYESTERDAY  | 52  | 平昨    |
| xtconstant.OFFSET_FLAG_FORCEOFF        | 53  | 强减    |
| xtconstant.OFFSET_FLAG_LOCALFORCECLOSE | 54  | 本地强平  |

## XtQuant数据结构说明

### 资产XtAsset

| 属性           | 类型    | 注释                                                                                                                   |
| ------------ | ----- | -------------------------------------------------------------------------------------------------------------------- |
| account_type | int   | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id   | str   | 资金账号                                                                                                                 |
| cash         | float | 可用金额                                                                                                                 |
| frozen_cash  | float | 冻结金额                                                                                                                 |
| market_value | float | 持仓市值                                                                                                                 |
| total_asset  | float | 总资产                                                                                                                  |

### 委托XtOrder

| 属性            | 类型    | 注释                                                                                                                                                                                |
| ------------- | ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| account_type  | int   | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type)                                                              |
| account_id    | str   | 资金账号                                                                                                                                                                              |
| stock_code    | str   | 证券代码，例如"600000.SH"                                                                                                                                                                |
| order_id      | int   | 订单编号                                                                                                                                                                              |
| order_sysid   | str   | 柜台合同编号                                                                                                                                                                            |
| order_time    | int   | 报单时间                                                                                                                                                                              |
| order_type    | int   | 委托类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%A7%94%E6%89%98%E7%B1%BB%E5%9E%8B-order-type)                                                                |
| order_volume  | int   | 委托数量                                                                                                                                                                              |
| price_type    | int   | 报价类型，该字段在返回时为柜台返回类型，不等价于下单传入的price_type，枚举值不一样功能一样，参见[数据字典](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-ebrokerpricetype-%E4%BB%B7%E6%A0%BC%E7%B1%BB%E5%9E%8B) |
| price         | float | 委托价格                                                                                                                                                                              |
| traded_volume | int   | 成交数量                                                                                                                                                                              |
| traded_price  | float | 成交均价                                                                                                                                                                              |
| order_status  | int   | 委托状态，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%A7%94%E6%89%98%E7%8A%B6%E6%80%81-order-status)                                                              |
| status_msg    | str   | 委托状态描述，如废单原因                                                                                                                                                                      |
| strategy_name | str   | 策略名称                                                                                                                                                                              |
| order_remark  | str   | 委托备注，最大 24 个英文字符                                                                                                                                                                  |
| direction     | int   | 多空方向，股票不适用；参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%A4%9A%E7%A9%BA%E6%96%B9%E5%90%91-direction)                                                           |
| offset_flag   | int   | 交易操作，用此字段区分股票买卖，期货开、平仓，期权买卖等；参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E4%BA%A4%E6%98%93%E6%93%8D%E4%BD%9C-offset-flag)                                       |

### 成交XtTrade

| 属性            | 类型    | 注释                                                                                                                                          |
| ------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| account_type  | int   | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type)                        |
| account_id    | str   | 资金账号                                                                                                                                        |
| stock_code    | str   | 证券代码                                                                                                                                        |
| order_type    | int   | 委托类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%A7%94%E6%89%98%E7%B1%BB%E5%9E%8B-order-type)                          |
| traded_id     | str   | 成交编号                                                                                                                                        |
| traded_time   | int   | 成交时间                                                                                                                                        |
| traded_price  | float | 成交均价                                                                                                                                        |
| traded_volume | int   | 成交数量                                                                                                                                        |
| traded_amount | float | 成交金额                                                                                                                                        |
| order_id      | int   | 订单编号                                                                                                                                        |
| order_sysid   | str   | 柜台合同编号                                                                                                                                      |
| strategy_name | str   | 策略名称                                                                                                                                        |
| order_remark  | str   | 委托备注，最大 24 个英文字符(                                                                                                                           |
| direction     | int   | 多空方向，股票不适用；参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%A4%9A%E7%A9%BA%E6%96%B9%E5%90%91-direction)                     |
| offset_flag   | int   | 交易操作，用此字段区分股票买卖，期货开、平仓，期权买卖等；参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E4%BA%A4%E6%98%93%E6%93%8D%E4%BD%9C-offset-flag) |

### 持仓XtPosition

| 属性               | 类型    | 注释                                                                                                                      |
| ---------------- | ----- | ----------------------------------------------------------------------------------------------------------------------- |
| account_type     | int   | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type)    |
| account_id       | str   | 资金账号                                                                                                                    |
| stock_code       | str   | 证券代码                                                                                                                    |
| volume           | int   | 持仓数量                                                                                                                    |
| can_use_volume   | int   | 可用数量                                                                                                                    |
| open_price       | float | 开仓价                                                                                                                     |
| market_value     | float | 市值                                                                                                                      |
| frozen_volume    | int   | 冻结数量                                                                                                                    |
| on_road_volume   | int   | 在途股份                                                                                                                    |
| yesterday_volume | int   | 昨夜拥股                                                                                                                    |
| avg_price        | float | 成本价                                                                                                                     |
| direction        | int   | 多空方向，股票不适用；参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%A4%9A%E7%A9%BA%E6%96%B9%E5%90%91-direction) |

### 期货持仓统计XtPositionStatistics

| 属性                                 | 类型     | 注释                                                                                                                      |
| ---------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------- |
| account_id                         | string | 账户                                                                                                                      |
| exchange_id                        | string | 市场代码                                                                                                                    |
| exchange_name                      | string | 市场名称                                                                                                                    |
| product_id                         | string | 品种代码                                                                                                                    |
| instrument_id                      | string | 合约代码                                                                                                                    |
| instrument_name                    | string | 合约名称                                                                                                                    |
| direction                          | int    | 多空方向，股票不适用；参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%A4%9A%E7%A9%BA%E6%96%B9%E5%90%91-direction) |
| hedge_flag                         | int    | 投保类型；参见[投保类型](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-ehedge-flag-type)                          |
| position                           | int    | 持仓数量                                                                                                                    |
| yesterday_position                 | int    | 昨仓数量                                                                                                                    |
| today_position                     | int    | 今仓数量                                                                                                                    |
| can_close_vol                      | int    | 可平数量                                                                                                                    |
| position_cost                      | float  | 持仓成本                                                                                                                    |
| avg_price                          | float  | 持仓均价                                                                                                                    |
| position_profit                    | float  | 持仓盈亏                                                                                                                    |
| float_profit                       | float  | 浮动盈亏                                                                                                                    |
| open_price                         | float  | 开仓均价                                                                                                                    |
| open_cost                          | float  | 开仓成本                                                                                                                    |
| used_margin                        | float  | 已使用保证金                                                                                                                  |
| used_commission                    | float  | 已使用的手续费                                                                                                                 |
| frozen_margin                      | float  | 冻结保证金                                                                                                                   |
| frozen_commission                  | float  | 冻结手续费                                                                                                                   |
| instrument_value                   | float  | 市值，合约价值                                                                                                                 |
| open_times                         | int    | 开仓次数                                                                                                                    |
| open_volume                        | int    | 总开仓量 中间平仓不减                                                                                                             |
| cancel_times                       | int    | 撤单次数                                                                                                                    |
| last_price                         | float  | 最新价                                                                                                                     |
| rise_ratio                         | float  | 当日涨幅                                                                                                                    |
| product_name                       | string | 产品名称                                                                                                                    |
| royalty                            | float  | 权利金市值                                                                                                                   |
| expire_date                        | string | 到期日                                                                                                                     |
| assest_weight                      | float  | 资产占比                                                                                                                    |
| increase_by_settlement             | float  | 当日涨幅（结）                                                                                                                 |
| margin_ratio                       | float  | 保证金占比                                                                                                                   |
| float_profit_divide_by_used_margin | float  | 浮盈比例（保证金）                                                                                                               |
| float_profit_divide_by_balance     | float  | 浮盈比例（动态权益）                                                                                                              |
| today_profit_loss                  | float  | 当日盈亏（结）                                                                                                                 |
| yesterday_init_position            | int    | 昨日持仓                                                                                                                    |
| frozen_royalty                     | float  | 冻结权利金                                                                                                                   |
| today_close_profit_loss            | float  | 当日盈亏（收）                                                                                                                 |
| close_profit                       | float  | 平仓盈亏                                                                                                                    |
| ft_product_name                    | string | 品种名称                                                                                                                    |

### 异步下单委托反馈XtOrderResponse

| 属性            | 类型  | 注释                                                                                                                   |
| ------------- | --- | -------------------------------------------------------------------------------------------------------------------- |
| account_type  | int | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id    | str | 资金账号                                                                                                                 |
| order_id      | int | 订单编号                                                                                                                 |
| strategy_name | str | 策略名称                                                                                                                 |
| order_remark  | str | 委托备注                                                                                                                 |
| seq           | int | 异步下单的请求序号                                                                                                            |

### 异步撤单委托反馈XtCancelOrderResponse

| 属性            | 类型  | 注释                                                                                                                   |
| ------------- | --- | -------------------------------------------------------------------------------------------------------------------- |
| account_type  | int | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id    | str | 资金账号                                                                                                                 |
| order_id      | int | 订单编号                                                                                                                 |
| order_sysid   | str | 柜台委托编号                                                                                                               |
| cancel_result | int | 撤单结果                                                                                                                 |
| seq           | int | 异步撤单的请求序号                                                                                                            |

### 下单失败错误XtOrderError

| 属性            | 类型  | 注释                                                                                                                   |
| ------------- | --- | -------------------------------------------------------------------------------------------------------------------- |
| account_type  | int | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id    | str | 资金账号                                                                                                                 |
| order_id      | int | 订单编号                                                                                                                 |
| error_id      | int | 下单失败错误码                                                                                                              |
| error_msg     | str | 下单失败具体信息                                                                                                             |
| strategy_name | str | 策略名称                                                                                                                 |
| order_remark  | str | 委托备注                                                                                                                 |

### 撤单失败错误XtCancelError

| 属性           | 类型  | 注释                                                                                                                   |
| ------------ | --- | -------------------------------------------------------------------------------------------------------------------- |
| account_type | int | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id   | str | 资金账号                                                                                                                 |
| order_id     | int | 订单编号                                                                                                                 |
| market       | int | 交易市场 0:上海 1:深圳                                                                                                       |
| order_sysid  | str | 柜台委托编号                                                                                                               |
| error_id     | int | 下单失败错误码                                                                                                              |
| error_msg    | str | 下单失败具体信息                                                                                                             |

### 信用账号资产XtCreditDetail

| 属性                       | 类型    | 注释                                                                                                                   |
| ------------------------ | ----- | -------------------------------------------------------------------------------------------------------------------- |
| account_type             | int   | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id               | str   | 资金账号                                                                                                                 |
| m_nStatus                | int   | 账号状态                                                                                                                 |
| m_nUpdateTime            | int   | 更新时间                                                                                                                 |
| m_nCalcConfig            | int   | 计算参数                                                                                                                 |
| m_dFrozenCash            | float | 冻结金额                                                                                                                 |
| m_dBalance               | float | 总资产                                                                                                                  |
| m_dAvailable             | float | 可用金额                                                                                                                 |
| m_dPositionProfit        | float | 持仓盈亏                                                                                                                 |
| m_dMarketValue           | float | 总市值                                                                                                                  |
| m_dFetchBalance          | float | 可取金额                                                                                                                 |
| m_dStockValue            | float | 股票市值                                                                                                                 |
| m_dFundValue             | float | 基金市值                                                                                                                 |
| m_dTotalDebt             | float | 总负债                                                                                                                  |
| m_dEnableBailBalance     | float | 可用保证金                                                                                                                |
| m_dPerAssurescaleValue   | float | 维持担保比例                                                                                                               |
| m_dAssureAsset           | float | 净资产                                                                                                                  |
| m_dFinDebt               | float | 融资负债                                                                                                                 |
| m_dFinDealAvl            | float | 融资本金                                                                                                                 |
| m_dFinFee                | float | 融资息费                                                                                                                 |
| m_dSloDebt               | float | 融券负债                                                                                                                 |
| m_dSloMarketValue        | float | 融券市值                                                                                                                 |
| m_dSloFee                | float | 融券息费                                                                                                                 |
| m_dOtherFare             | float | 其它费用                                                                                                                 |
| m_dFinMaxQuota           | float | 融资授信额度                                                                                                               |
| m_dFinEnableQuota        | float | 融资可用额度                                                                                                               |
| m_dFinUsedQuota          | float | 融资冻结额度                                                                                                               |
| m_dSloMaxQuota           | float | 融券授信额度                                                                                                               |
| m_dSloEnableQuota        | float | 融券可用额度                                                                                                               |
| m_dSloUsedQuota          | float | 融券冻结额度                                                                                                               |
| m_dSloSellBalance        | float | 融券卖出资金                                                                                                               |
| m_dUsedSloSellBalance    | float | 已用融券卖出资金                                                                                                             |
| m_dSurplusSloSellBalance | float | 剩余融券卖出资金                                                                                                             |

### 负债合约StkCompacts

| 属性                   | 类型    | 注释                                                                                                                   |
| -------------------- | ----- | -------------------------------------------------------------------------------------------------------------------- |
| account_type         | int   | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id           | str   | 资金账号                                                                                                                 |
| compact_type         | int   | 合约类型                                                                                                                 |
| cashgroup_prop       | int   | 头寸来源                                                                                                                 |
| exchange_id          | int   | 证券市场                                                                                                                 |
| open_date            | int   | 开仓日期                                                                                                                 |
| business_vol         | int   | 合约证券数量                                                                                                               |
| real_compact_vol     | int   | 未还合约数量                                                                                                               |
| ret_end_date         | int   | 到期日                                                                                                                  |
| business_balance     | float | 合约金额                                                                                                                 |
| businessFare         | float | 合约息费                                                                                                                 |
| real_compact_balance | float | 未还合约金额                                                                                                               |
| real_compact_fare    | float | 未还合约息费                                                                                                               |
| repaid_fare          | float | 已还息费                                                                                                                 |
| repaid_balance       | float | 已还金额                                                                                                                 |
| instrument_id        | str   | 证券代码                                                                                                                 |
| compact_id           | str   | 合约编号                                                                                                                 |
| position_str         | str   | 定位串                                                                                                                  |

### 融资融券标的CreditSubjects

| 属性            | 类型    | 注释                                                                                                                   |
| ------------- | ----- | -------------------------------------------------------------------------------------------------------------------- |
| account_type  | int   | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id    | str   | 资金账号                                                                                                                 |
| slo_status    | int   | 融券状态                                                                                                                 |
| fin_status    | int   | 融资状态                                                                                                                 |
| exchange_id   | int   | 证券市场                                                                                                                 |
| slo_ratio     | float | 融券保证金比例                                                                                                              |
| fin_ratio     | float | 融资保证金比例                                                                                                              |
| instrument_id | str   | 证券代码                                                                                                                 |

### 可融券数据CreditSloCode

| 属性             | 类型  | 注释                                                                                                                   |
| -------------- | --- | -------------------------------------------------------------------------------------------------------------------- |
| account_type   | int | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id     | str | 资金账号                                                                                                                 |
| cashgroup_prop | int | 头寸来源                                                                                                                 |
| exchange_id    | int | 证券市场                                                                                                                 |
| enable_amount  | int | 融券可融数量                                                                                                               |
| instrument_id  | str | 证券代码                                                                                                                 |

### 标的担保品CreditAssure

| 属性            | 类型    | 注释                                                                                                                   |
| ------------- | ----- | -------------------------------------------------------------------------------------------------------------------- |
| account_type  | int   | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type) |
| account_id    | str   | 资金账号                                                                                                                 |
| assure_status | int   | 是否可做担保                                                                                                               |
| exchange_id   | int   | 证券市场                                                                                                                 |
| assure_ratio  | float | 担保品折算比例                                                                                                              |
| instrument_id | str   | 证券代码                                                                                                                 |

### 账号状态XtAccountStatus

| 属性           | 类型  | 注释                                                                                                                     |
| ------------ | --- | ---------------------------------------------------------------------------------------------------------------------- |
| account_type | int | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type)   |
| account_id   | str | 资金账号                                                                                                                   |
| status       | int | 账号状态，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%8A%B6%E6%80%81-account-status) |

### 账号信息XtAccountInfo

| 属性                     | 类型  | 注释                                                                                                                     |
| ---------------------- | --- | ---------------------------------------------------------------------------------------------------------------------- |
| account_type           | int | 账号类型，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B-account-type)   |
| account_id             | str | 资金账号                                                                                                                   |
| broker_type            | int | 同 account_type                                                                                                         |
| platform_id            | int | 平台号                                                                                                                    |
| account_classification | int | 账号分类                                                                                                                   |
| login_status           | int | 账号状态，参见[数据字典](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%8A%B6%E6%80%81-account-status) |

### 约券相关异步接口的反馈XtSmtAppointmentResponse

| 属性       | 类型   | 注释                   |
| -------- | ---- | -------------------- |
| seq      | int  | 异步请求序号               |
| success  | bool | 申请是否成功               |
| msg      | str  | 反馈信息                 |
| apply_id | str  | 若申请成功返回资券申请编号，否则返回-1 |

## XtQuant API说明

### 系统设置接口

#### 创建API实例

```
XtQuantTrader(path, session_id)
```

- 释义
  - 创建XtQuant API的实例
- 参数
  - path - str MiniQMT客户端userdata_mini的完整路径
  - session_id - int 与MiniQMT通信的会话ID，不同的会话要保证不重
- 返回
  - XtQuant API实例对象
- 备注
  - 后续对XtQuant API的操作都需要该实例对象
  - 通常情况下只需要创建一个XtQuant API实例
- 示例

```
path = 'D:\\迅投极速交易终端 睿智融科版\\userdata_mini'
# session_id为会话编号，策略使用方对于不同的Python策略需要使用不同的会话编号
session_id = 123456
#后续的所有示例将使用该实例对象
xt_trader = XtQuantTrader(path, session_id)
```

#### 注册回调类

```
register_callback(callback)
```

- 释义
  - 将回调类实例对象注册到API实例中，用以消息回调和主推
- 参数
  - callback - XtQuantTraderCallback 回调类实例对象
- 返回
  - 无
- 备注
  - 无
- 示例

```
# 创建交易回调类对象，并声明接收回调
class MyXtQuantTraderCallback(XtQuantTraderCallback)：
    ...
    pass
callback = MyXtQuantTraderCallback()
#xt_trader为XtQuant API实例对象
xt_trader.register_callback(callback)
```

#### 准备API环境

```
start()
```

- 释义
  - 启动交易线程，准备交易所需的环境
- 参数
  - 无
- 返回
  - 无
- 备注
  - 无
- 示例

```
# 启动交易线程
#xt_trader为XtQuant API实例对象
xt_trader.start()
```

#### 创建连接

```
connect()
```

- 释义
  - 连接MiniQMT
- 参数
  - 无
- 返回
  - 连接结果信息，连接成功返回0，失败返回非0
- 备注
  - 该连接为一次性连接，断开连接后不会重连，需要再次主动调用
- 示例

```
# 建立交易连接，返回0表示连接成功
#xt_trader为XtQuant API实例对象
connect_result = xt_trader.connect()
print(connect_result)
```

#### 停止运行

```
stop()
```

- 释义
  - 停止API接口
- 参数
  - 无
- 返回
  - 无
- 备注
  - 无
- 示例

```
#xt_trader为XtQuant API实例对象
xt_trader.stop()
```

#### 阻塞当前线程进入等待状态

```
run_forever()
```

- 释义
  - 阻塞当前线程，进入等待状态，直到stop函数被调用结束阻塞
- 参数
  - 无
- 返回
  - 无
- 备注
  - 无
- 示例

```
#xt_trader为XtQuant API实例对象
xt_trader.run_forever()
```

#### 开启主动请求接口的专用线程

```
set_relaxed_response_order_enabled(enabled)
```

- 释义
  
  - 控制主动请求接口的返回是否从额外的专用线程返回，以获得宽松的数据时序

- 参数
  
  - enabled - bool 是否开启，默认为False关闭

- 返回
  
  - 无

- 备注
  
  - 如果开启，在on_stock_order等推送回调中调用同步请求不会卡住，但查询和推送的数据在时序上会变得不确定
  
  - ```
    timeline    t1    t2    t3    t4
    callback    push1    push2    push3    resp4
    do        query4 ------------------^
    ```
  
  - 例如：分别在t1 t2 t3时刻到达三条委托数据，在on_push1中调用同步委托查询接口query_orders()
  
  - 未开启宽松时序时，查询返回resp4会在t4时刻排队到push3完成之后处理，这使得同步等待结果的查询不能返回而卡住执行
  
  - 开启宽松时序时，查询返回的resp4由专用线程返回，程序正常执行，但此时查到的resp4是push3之后的状态，也就是说resp4中的委托要比push2 push3这两个前一时刻推送的数据新，但在更早的t1时刻就进入了处理
  
  - 使用中请根据策略实际情况来开启，通常情况下，推荐在on_stock_order等推送回调中使用查询接口的异步版本，如`query_stock_orders_async`

### 操作接口

#### 订阅账号信息

```
subscribe(account)
```

- 释义
  - 订阅账号信息，包括资金账号、委托信息、成交信息、持仓信息
- 参数
  - account - StockAccount 资金账号
- 返回
  - 订阅结果信息，订阅成功返回0，订阅失败返回-1
- 备注
  - 无
- 示例
  - 订阅资金账号1000000365

```
account = StockAccount('1000000365')
#xt_trader为XtQuant API实例对象
subscribe_result = xt_trader.subscribe(account)
```

#### 反订阅账号信息

```
unsubscribe(account)
```

- 释义
  - 反订阅账号信息
- 参数
  - account - StockAccount 资金账号
- 返回
  - 反订阅结果信息，订阅成功返回0，订阅失败返回-1
- 备注
  - 无
- 示例
  - 订阅资金账号1000000365

```
account = StockAccount('1000000365')
#xt_trader为XtQuant API实例对象
unsubscribe_result = xt_trader.unsubscribe(account)
```

#### 股票同步报单

```
order_stock(account, stock_code, order_type, order_volume, price_type, price, strategy_name, order_remark)
```

- 释义
  - 对股票进行下单操作
- 参数
  - account - StockAccount 资金账号
  - stock_code - str 证券代码，如'600000.SH'
  - order_type - int 委托类型
  - order_volume - int 委托数量，股票以'股'为单位，债券以'张'为单位
  - price_type - int 报价类型
  - price - float 委托价格
  - strategy_name - str 策略名称
  - order_remark - str 委托备注
- 返回
  - 系统生成的订单编号，成功委托后的订单编号为大于0的正整数，如果为-1表示委托失败
- 备注
  - 无
- 示例
  - 股票资金账号1000000365对浦发银行买入1000股，使用限价价格10.5元, 委托备注为'order_test'

```
account = StockAccount('1000000365')
#xt_trader为XtQuant API实例对象
order_id = xt_trader.order_stock(account, '600000.SH', xtconstant.STOCK_BUY, 1000, xtconstant.FIX_PRICE, 10.5, 'strategy1', 'order_test')
```

#### 股票异步报单

```
order_stock_async(account, stock_code, order_type, order_volume, price_type, price, strategy_name, order_remark)
```

- 释义
  - 对股票进行异步下单操作，异步下单接口如果正常返回了下单请求序号seq，会收到on_order_stock_async_response的委托反馈
- 参数
  - account - StockAccount 资金账号
  - stock_code - str 证券代码， 如'600000.SH'
  - order_type - int 委托类型
  - order_volume - int 委托数量，股票以'股'为单位，债券以'张'为单位
  - price_type - int 报价类型
  - price - float 委托价格
  - strategy_name - str 策略名称
  - order_remark - str 委托备注
- 返回
  - 返回下单请求序号seq，成功委托后的下单请求序号为大于0的正整数，如果为-1表示委托失败
- 备注
  - 如果失败，则通过下单失败主推接口返回下单失败信息
- 示例
  - 股票资金账号1000000365对浦发银行买入1000股，使用限价价格10.5元，委托备注为'order_test'

```
account = StockAccount('1000000365')
#xt_trader为XtQuant API实例对象
seq = xt_trader.order_stock_async(account, '600000.SH', xtconstant.STOCK_BUY, 1000, xtconstant.FIX_PRICE, 10.5, 'strategy1', 'order_test')
```

#### 股票同步撤单

```
cancel_order_stock(account, order_id)
```

- 释义
  - 根据订单编号对委托进行撤单操作
- 参数
  - account - StockAccount 资金账号
  - order_id - int 同步下单接口返回的订单编号,对于期货来说，是order结构中的order_sysid字段
- 返回
  - 返回是否成功发出撤单指令，0: 成功, -1: 表示撤单失败
- 备注
  - 无
- 示例
  - 股票资金账号1000000365对订单编号为order_id的委托进行撤单

```
account = StockAccount('1000000365')
order_id = 100
#xt_trader为XtQuant API实例对象
cancel_result = xt_trader.cancel_order_stock(account, order_id)
```

#### 股票同步撤单

```
cancel_order_stock_sysid(account, market, order_sysid)
```

- 释义
  - 根据券商柜台返回的合同编号对委托进行撤单操作
- 参数
  - account - StockAccount 资金账号
  - market - int 交易市场
  - order_sysid - str 券商柜台的合同编号
- 返回
  - 返回是否成功发出撤单指令，0: 成功， -1: 表示撤单失败
- 备注
  - 无
- 示例
  - 股票资金账号1000000365对柜台合同编号为order_sysid的上交所委托进行撤单

```
account = StockAccount('1000000365')
market = xtconstant.SH_MARKET
order_sysid = "100" 
#xt_trader为XtQuant API实例对象
cancel_result = xt_trader.cancel_order_stock_sysid(account, market, order_sysid)
```

#### 股票异步撤单

```
cancel_order_stock_async(account, order_id)
```

- 释义
  - 根据订单编号对委托进行异步撤单操作
- 参数
  - account - StockAccount 资金账号
  - order_id - int 下单接口返回的订单编号，对于期货来说，是order结构中的order_sysid
- 返回
  - 返回撤单请求序号, 成功委托后的撤单请求序号为大于0的正整数, 如果为-1表示委托失败
- 备注
  - 如果失败，则通过撤单失败主推接口返回撤单失败信息
- 示例
  - 股票资金账号1000000365对订单编号为order_id的委托进行异步撤单

```
account = StockAccount('1000000365')
order_id = 100
#xt_trader为XtQuant API实例对象
cancel_result = xt_trader.cancel_order_stock_async(account, order_id)
```

#### 股票异步撤单

```
cancel_order_stock_sysid_async(account, market, order_sysid)
```

- 释义
  - 根据券商柜台返回的合同编号对委托进行异步撤单操作
- 参数
  - account - StockAccount 资金账号
  - market - int 交易市场
  - order_sysid - str 券商柜台的合同编号
- 返回
  - 返回撤单请求序号, 成功委托后的撤单请求序号为大于0的正整数, 如果为-1表示委托失败
- 备注
  - 如果失败，则通过撤单失败主推接口返回撤单失败信息
- 示例
  - 股票资金账号1000000365对柜台合同编号为order_sysid的上交所委托进行异步撤单

```
account = StockAccount('1000000365')
market = xtconstant.SH_MARKET
order_sysid = "100" 
#xt_trader为XtQuant API实例对象
cancel_result = xt_trader.cancel_order_stock_sysid_async(account, market, order_sysid)
```

#### 资金划拨

```
fund_transfer(account, transfer_direction, price)
```

- 释义
  - 资金划拨
- 参数
  - account - StockAccount 资金账号
  - transfer_direction - int 划拨方向，见数据字典划拨方向(transfer_direction)字段说明
  - price - float 划拨金额
- 返回
  - (success, msg)
    - success - bool 划拨操作是否成功
    - msg - str 反馈信息

#### 外部交易数据录入

```
sync_transaction_from_external(operation, data_type, account, deal_list)
```

- 释义
  
  - 通用数据导出

- 参数
  
  - operation - str 操作类型，有"UPDATE","REPLACE","ADD","DELETE"
  - data_type - str 数据类型，有"DEAL"
  - account - StockAccount 资金账号
  - deal_list - list 成交列表,每一项是Deal成交对象的参数字典,键名参考官网数据字典,大小写保持一致

- 返回
  
  - result - dict 结果反馈信息

- 示例
  
  ```
  deal_list = [
                  {'m_strExchangeID':'SF', 'm_strInstrumentID':'ag2407'
                  , 'm_strTradeID':'123456', 'm_strOrderSysID':'1234566'
                  , 'm_dPrice':7600, 'm_nVolume':1
                  , 'm_strTradeDate': '20240627'
                  }
  ]
  resp = xt_trader.sync_transaction_from_external('ADD', 'DEAL', acc, deal_list)
  print(resp)
  #成功输出示例：{'msg': 'sync transaction from external success'}
  #失败输出示例：{'error': {'msg': '[0-0: invalid operation type: ADDD], '}}
  ```

### 股票查询接口

#### 资产查询

```
query_stock_asset(account)
```

- 释义
  - 查询资金账号对应的资产
- 参数
  - account - StockAccount 资金账号
- 返回
  - 该账号对应的资产对象[XtAsset](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B5%84%E4%BA%A7xtasset)或者None
- 备注
  - 返回None表示查询失败
- 示例
  - 查询股票资金账号1000000365对应的资产数据

```
account = StockAccount('1000000365')
#xt_trader为XtQuant API实例对象
asset = xt_trader.query_stock_asset(account)
```

#### 委托查询

```
query_stock_orders(account, cancelable_only = False)
```

- 释义
  - 查询资金账号对应的当日所有委托
- 参数
  - account - StockAccount 资金账号
  - cancelable_only - bool 仅查询可撤委托
- 返回
  - 该账号对应的当日所有委托对象[XtOrder](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%A7%94%E6%89%98xtorder)组成的list或者None
- 备注
  - None表示查询失败或者当日委托列表为空
- 示例
  - 查询股票资金账号1000000365对应的当日所有委托

```
account = StockAccount('1000000365')
#xt_trader为XtQuant API实例对象
orders = xt_trader.query_stock_orders(account, False)
```

#### 成交查询

```
query_stock_trades(account)
```

- 释义
  - 查询资金账号对应的当日所有成交
- 参数
  - account - StockAccount 资金账号
- 返回
  - 该账号对应的当日所有成交对象[XtTrade](http://dict.thinktrader.net/nativeApi/xttrader.html#%E6%88%90%E4%BA%A4xttrade)组成的list或者None
- 备注
  - None表示查询失败或者当日成交列表为空
- 示例
  - 查询股票资金账号1000000365对应的当日所有成交

```
account = StockAccount('1000000365')
#xt_trader为XtQuant API实例对象
trades = xt_trader.query_stock_trades(account)
```

#### 持仓查询

```
query_stock_positions(account)
```

- 释义
  - 查询资金账号对应的持仓
- 参数
  - account - StockAccount 资金账号
- 返回
  - 该账号对应的最新持仓对象[XtPosition](http://dict.thinktrader.net/nativeApi/xttrader.html#%E6%8C%81%E4%BB%93xtposition)组成的list或者None
- 备注
  - None表示查询失败或者当日持仓列表为空
- 示例
  - 查询股票资金账号1000000365对应的最新持仓

```
account = StockAccount('1000000365')
#xt_trader为XtQuant API实例对象
positions = xt_trader.query_stock_positions(account)
```

#### 期货持仓统计查询

```
query_position_statistics(account)
```

- 释义
  - 查询期货账号的持仓统计
- 参数
  - account - StockAccount 资金账号
- 返回
  - 该账号对应的最新持仓对象[XtPositionStatistics](http://dict.thinktrader.net/nativeApi/xttrader.html#%E6%9C%9F%E8%B4%A7%E6%8C%81%E4%BB%93%E7%BB%9F%E8%AE%A1xtpositionstatistics)组成的list或者None
- 备注
  - None表示查询失败或者当日持仓列表为空
- 示例
  - 查询期货资金账号1000000365对应的最新持仓

```
account = StockAccount('1000000365', 'FUTURE')
#xt_trader为XtQuant API实例对象
positions = xt_trader.query_position_statistics(account)
```

### 信用查询接口

#### 信用资产查询

```
query_credit_detail(account)
```

- 释义
  - 查询信用资金账号对应的资产
- 参数
  - account - StockAccount 资金账号
- 返回
  - 该信用账户对应的资产对象[XtCreditDetail](http://dict.thinktrader.net/nativeApi/xttrader.html#%E4%BF%A1%E7%94%A8%E8%B4%A6%E5%8F%B7%E8%B5%84%E4%BA%A7xtcreditdetail)组成的list或者None
- 备注
  - None表示查询失败
  - 通常情况下一个资金账号只有一个详细信息数据
- 示例
  - 查询信用资金账号1208970161对应的资产信息

```
account = StockAccount('1208970161', 'CREDIT')
#xt_trader为XtQuant API实例对象
datas = xt_trader.query_credit_detail(account)
```

#### 负债合约查询

```
query_stk_compacts(account)
```

- 释义
  - 查询资金账号对应的负债合约
- 参数
  - account - StockAccount 资金账号
- 返回
  - 该账户对应的负债合约对象[StkCompacts](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%9F%E5%80%BA%E5%90%88%E7%BA%A6stkcompacts)组成的list或者None
- 备注
  - None表示查询失败或者负债合约列表为空
- 示例
  - 查询信用资金账号1208970161对应的负债合约

```
account = StockAccount('1208970161', 'CREDIT')
#xt_trader为XtQuant API实例对象
datas = xt_trader.query_stk_compacts(account)
```

#### 融资融券标的查询

```
query_credit_subjects(account)
```

- 释义
  - 查询资金账号对应的融资融券标的
- 参数
  - account - StockAccount 资金账号
- 返回
  - 该账户对应的融资融券标的对象[CreditSubjects](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%9E%8D%E8%B5%84%E8%9E%8D%E5%88%B8%E6%A0%87%E7%9A%84creditsubjects)组成的list或者None
- 备注
  - None表示查询失败或者融资融券标的列表为空
- 示例
  - 查询信用资金账号1208970161对应的融资融券标的

```
account = StockAccount('1208970161', 'CREDIT')
#xt_trader为XtQuant API实例对象
datas = xt_trader.query_credit_subjects(account)
```

#### 可融券数据查询

```
query_credit_slo_code(account)
```

- 释义
  - 查询资金账号对应的可融券数据
- 参数
  - account - StockAccount 资金账号
- 返回
  - 该账户对应的可融券数据对象[CreditSloCode](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%8F%AF%E8%9E%8D%E5%88%B8%E6%95%B0%E6%8D%AEcreditslocode)组成的list或者None
- 备注
  - None表示查询失败或者可融券数据列表为空
- 示例
  - 查询信用资金账号1208970161对应的可融券数据

```
account = StockAccount('1208970161', 'CREDIT')
#xt_trader为XtQuant API实例对象
datas = xt_trader.query_credit_slo_code(account)
```

#### 标的担保品查询

```
query_credit_assure(account)
```

- 释义
  
  - 查询资金账号对应的标的担保品

- 参数
  
  - account - StockAccount 资金账号

- 返回

- 该账户对应的标的担保品对象[CreditAssure](http://dict.thinktrader.net/nativeApi/xttrader.html#%E6%A0%87%E7%9A%84%E6%8B%85%E4%BF%9D%E5%93%81creditassure)组成的list或者None

- 备注
  
  - None表示查询失败或者标的担保品列表为空

- 示例
  
  - 查询信用资金账号1208970161对应的标的担保品

```
account = StockAccount('1208970161', 'CREDIT')
#xt_trader为XtQuant API实例对象
datas = xt_trader.query_credit_assure(account)
```

### 其他查询接口

#### 新股申购额度查询

```
query_new_purchase_limit(account)
```

- 释义
  - 查询新股申购额度
- 参数
  - account - StockAccount 资金账号
- 返回
  - dict 新股申购额度数据集
    - { type1: number1, type2: number2, ... }
      - type - str 品种类型
        - `KCB` - 科创板，`SH` - 上海，`SZ` - 深圳
      - number - int 可申购股数
- 备注
  - 数据仅代表股票申购额度，债券的申购额度固定10000张

#### 当日新股信息查询

```
query_ipo_data()
```

- 释义
  
  - 查询当日新股新债信息

- 参数
  
  - 无

- 返回
  
  - dict 新股新债信息数据集
    
    - { stock1: info1, stock2: info2, ... }
      
      - stock - str 品种代码，例如 '301208.SZ'
      - info - dict 新股信息
        - name - str 品种名称
        - type - str 品种类型
          - `STOCK` - 股票，`BOND` - 债券
        - minPurchaseNum / maxPurchaseNum - int 最小 / 最大申购额度
          - 单位为股（股票）/ 张（债券）
        - purchaseDate - str 申购日期
        - issuePrice - float 发行价
    
    - 返回值示例
      
      ```
      {'754810.SH': {'name': '丰山发债', 'type': 'BOND', 'maxPurchaseNum': 10000, 'minPurchaseNum': 10, 'purchaseDate': '20220627', 'issuePrice': 100.0}, '301208.SZ': {'name': '中亦科技', 'type': 'STOCK', 'maxPurchaseNum': 16500, 'minPurchaseNum': 500, 'purchaseDate': '20220627', 'issuePrice': 46.06}}
      ```

- 备注
  
  - 无

#### 账号信息查询

```
query_account_infos()
```

- 释义
  
  - 查询所有资金账号

- 参数
  
  - 无

- 返回
  
  - list 账号信息列表
    
    - [ XtAccountInfo ]

- 备注
  
  - 无

#### 账号状态查询

```
query_account_status()
```

- 释义
  
  - 查询所有账号状态

- 参数
  
  - 无

- 返回
  
  - list 账号状态列表
    
    - [ XtAccountStatus ]

- 备注
  
  - 无

#### 普通柜台资金查询

```
query_com_fund(account)
```

- 释义
  - 划拨业务查询普通柜台的资金
- 参数
  - account - StockAccount 资金账号
- 返回
  - result - dict 资金信息，包含以下字段
    - success - bool
    - erro - str
    - currentBalance - double 当前余额
    - enableBalance - double 可用余额
    - fetchBalance - double 可取金额
    - interest - double 待入账利息
    - assetBalance - double 总资产
    - fetchCash - double 可取现金
    - marketValue - double 市值
    - debt - double 负债

#### 普通柜台持仓查询

```
query_com_position(account)
```

- 释义
  - 划拨业务查询普通柜台的持仓
- 参数
  - account - StockAccount 资金账号
- 返回
  - result - list 持仓信息列表[position1, position2, ...]
    - position - dict 持仓信息，包含以下字段
      - success - bool
      - error - str
      - stockAccount - str 股东号
      - exchangeType - str 交易市场
      - stockCode - str 证券代码
      - stockName - str 证券名称
      - totalAmt - float 总量
      - enableAmount - float 可用量
      - lastPrice - float 最新价
      - costPrice - float 成本价
      - income - float 盈亏
      - incomeRate - float 盈亏比例
      - marketValue - float 市值
      - costBalance - float 成本总额
      - bsOnTheWayVol - int 买卖在途量
      - prEnableVol - int 申赎可用量

#### 通用数据导出

```
export_data(account, result_path, data_type, start_time = None, end_time = None, user_param = {})
```

- 释义
  
  - 通用数据导出

- 参数
  
  - account - StockAccount 资金账号
  - result_path - str 导出路径，包含文件名及.csv后缀，如'C:\Users\Desktop\test\deal.csv'
  - data_type - str 数据类型，如'deal'
  - start_time - str 开始时间（可缺省）
  - end_time - str 结束时间（可缺省）
  - user_param - dict 用户参数（可缺省）

- 返回
  
  - result - dict 结果反馈信息

- 示例
  
  ```
  resp = xt_trader.export_data(acc, 'C:\\Users\\Desktop\\test\\deal.csv', 'deal')
  print(resp)
  #成功输出示例：{'msg': 'export success'}
  #失败输出示例：{'error': {'errorMsg': 'can not find account info, accountID:2000449 accountType:2'}}
  ```

#### 通用数据查询

```
query_data(account, result_path, data_type, start_time = None, end_time = None, user_param = {})
```

- 释义
  
  - 通用数据查询，利用export_data接口导出数据后再读取其中的数据内容，读取完毕后删除导出的文件

- 参数
  
  同export_data

- 返回
  
  - result - dict 数据信息

- 示例
  
  ```
  data = xt_trader.query_data(acc, 'C:\\Users\\Desktop\\test\\deal.csv', 'deal')
  print(data)
  #成功输出示例：
  #    account_id    account_Type    stock_code    order_type    ...  
  #0    2003695    2    688488.SH    23    ...
  #1    2003695    2    000096.SZ    23    ...
  #失败输出示例：{'error': {'errorMsg': 'can not find account info, accountID:2000449 accountType:2'}}
  ```

### 约券相关接口

#### 券源行情查询

```
smt_query_quoter(account)
```

- 释义
  - 券源行情查询
- 参数
  - account - StockAccount 资金账号
- 返回
  - result - list 券源信息列表[quoter1, quoter2, ...]
    - quoter - dict 券源信息，包含以下字段
      - success - bool
      - error - str
      - finType - str 金融品种
      - stockType - str 证券类型
      - date - int 期限天数
      - code - str 证券代码
      - codeName - str 证券代码名称
      - exchangeType - str 市场
      - fsmpOccupedRate - float 资券占用利率
      - fineRate - float 罚息利率
      - fsmpreendRate - float 资券提前归还利率
      - usedRate - float 资券使用利率
      - unUusedRate - float 资券占用未使用利率
      - initDate - int 交易日期
      - endDate - int 到期日期
      - enableSloAmountT0 - float T+0可融券数量
      - enableSloAmountT3 - float T+3可融券数量
      - srcGroupId - str 来源组编号
      - applyMode - str 资券申请方式，"1":库存券，"2":专项券
      - lowDate - int 最低期限天数

#### 库存券约券申请

```
smt_negotiate_order_async(self, account, src_group_id, order_code, date, amount, apply_rate, dict_param={})
```

- 释义
  
  - 库存券约券申请的异步接口，异步接口如果正常返回了请求序号seq，会收到on_smt_appointment_async_response的反馈

- 参数
  
  - account - StockAccount 资金账号
  - src_group_id - str 来源组编号
  - order_code - str 证券代码，如'600000.SH'
  - date - int 期限天数
  - amount - int 委托数量
  - apply_rate - float 资券申请利率
  
  注：目前有如下参数通过一个可缺省的字典传递，键名与参数名称相同
  
  - dict_param - dict 可缺省的字典参数
    - subFareRate - float 提前归还利率
    - fineRate - float 罚息利率

- 返回
  
  - 返回请求序号seq，成功发起申请后的请求序号为大于0的正整数，如果为-1表示发起申请失败

- 示例

```
account = StockAccount('1000008', 'CREDIT')
dict_param = {'subFareRate':0.1, 'fineRate':0.1}
#xt_trader为XtQuant API实例对象
seq = xt_trader.smt_negotiate_order_async(account, '', '000001.SZ', 7, 100, 0.2, dict_param)
```

#### 约券合约查询

```
smt_query_compact(account)
```

- 释义
  - 约券合约查询
- 参数
  - account - StockAccount 资金账号
- 返回
  - result - list 约券合约信息列表[compact1, compact2, ...]
    - compact - dict 券源信息，包含以下字段
      - success - bool
      - error - str
      - createDate - int 创建日期
      - cashcompactId - str 头寸合约编号
      - oriCashcompactId - str 原头寸合约编号
      - applyId - str 资券申请编号
      - srcGroupId - str 来源组编号
      - comGroupId - str 资券组合编号
      - finType - str 金融品种
      - exchangeType - str 市场
      - code - str 证券代码
      - codeName - str 证券代码名称
      - date - int 期限天数
      - beginCompacAmount - float 期初合约数量
      - beginCompacBalance - float 期初合约金额
      - compacAmount - float 合约数量
      - compacBalance - float 合约金额
      - returnAmount - float 返还数量
      - returnBalance - float 返还金额
      - realBuyAmount - float 回报买入数量
      - fsmpOccupedRate - float 资券占用利率
      - compactInterest - float 合约利息金额
      - compactFineInterest - float 合约罚息金额
      - repaidInterest - float 已还利息
      - repaidFineInterest - float 归还罚息
      - fineRate - float 罚息利率
      - preendRate - float 资券提前归还利率
      - compactType - str 资券合约类型
      - postponeTimes - int 展期次数
      - compactStatus - str 资券合约状态，"0":未归还，"1":部分归还，"2":提前了结，"3":到期了结，"4":逾期了结，"5":逾期，"9":已作废
      - lastInterestDate - int 上次结息日期
      - interestEndDate - int 记息结束日期
      - validDate - int 有效日期
      - dateClear - int 清算日期
      - usedAmount - float 已使用数量
      - usedBalance - float 使用金额
      - usedRate - float 资券使用利率
      - unUusedRate - float 资券占用未使用利率
      - srcGroupName - str 来源组名称
      - repaidDate - int 归还日期
      - preOccupedInterest - float 当日实际应收利息
      - compactInterestx - float 合约总利息
      - enPostponeAmount - float 可展期数量
      - postponeStatus - str 合约展期状态，"0":未审核，"1":审核通过，"2":已撤销，"3":审核不通过
      - applyMode - str 资券申请方式，"1":库存券，"2":专项券

### 回调类

```
class MyXtQuantTraderCallback(XtQuantTraderCallback):
    def on_disconnected(self):
        """
        连接状态回调
        :return:
        """
        print("connection lost")
    def on_account_status(self, status):
        """
        账号状态信息推送
        :param response: XtAccountStatus 对象
        :return:
        """
        print("on_account_status")
        print(status.account_id, status.account_type, status.status)
    def on_stock_order(self, order):
        """
        委托信息推送
        :param order: XtOrder对象
        :return:
        """
        print("on order callback:")
        print(order.stock_code, order.order_status, order.order_sysid)
    def on_stock_trade(self, trade):
        """
        成交信息推送
        :param trade: XtTrade对象
        :return:
        """
        print("on trade callback")
        print(trade.account_id, trade.stock_code, trade.order_id)
    def on_order_error(self, order_error):
        """
        下单失败信息推送
        :param order_error:XtOrderError 对象
        :return:
        """
        print("on order_error callback")
        print(order_error.order_id, order_error.error_id, order_error.error_msg)
    def on_cancel_error(self, cancel_error):
        """
        撤单失败信息推送
        :param cancel_error: XtCancelError 对象
        :return:
        """
        print("on cancel_error callback")
        print(cancel_error.order_id, cancel_error.error_id, cancel_error.error_msg)
    def on_order_stock_async_response(self, response):
        """
        异步下单回报推送
        :param response: XtOrderResponse 对象
        :return:
        """
        print("on_order_stock_async_response")
        print(response.account_id, response.order_id, response.seq)
    def on_smt_appointment_async_response(self, response):
        """
        :param response: XtAppointmentResponse 对象
        :return:
        """
        print("on_smt_appointment_async_response")
        print(response.account_id, response.order_sysid, response.error_id, response.error_msg, response.seq)
```

#### 连接状态回调

```
on_disconnected()
```

- 释义
  - 失去连接时推送信息
- 参数
  - 无
- 返回
  - 无
- 备注
  - 无

#### 账号状态信息推送

```
on_account_status(data)
```

- 释义
  - 账号状态信息变动推送
- 参数
  - data - [XtAccountStatus](http://dict.thinktrader.net/nativeApi/xttrader.html#%E8%B4%A6%E5%8F%B7%E7%8A%B6%E6%80%81xtaccountstatus) 账号状态信息
- 返回
  - 无
- 备注
  - 无

#### 委托信息推送

```
on_stock_order(data)
```

- 释义
  - 委托信息变动推送,例如已成交数量，委托状态变化等
- 参数
  - data - [XtOrder](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%A7%94%E6%89%98xtorder) 委托信息
- 返回
  - 无
- 备注
  - 无

#### 成交信息推送

```
on_stock_trade(data)
```

- 释义
  - 成交信息变动推送
- 参数
  - data - [XtTrade](http://dict.thinktrader.net/nativeApi/xttrader.html#%E6%88%90%E4%BA%A4xttrade) 成交信息
- 返回
  - 无
- 备注
  - 无

#### 下单失败信息推送

```
on_order_error(data)
```

- 释义
  - 下单失败信息推送
- 参数
  - data - [XtOrderError](http://dict.thinktrader.net/nativeApi/xttrader.html#%E4%B8%8B%E5%8D%95%E5%A4%B1%E8%B4%A5%E9%94%99%E8%AF%AFxtordererror) 下单失败信息
- 返回
  - 无
- 备注
  - 无

#### 撤单失败信息推送

```
on_cancel_error(data)
```

- 释义
  - 撤单失败信息的推送
- 参数
  - data - [XtCancelError](http://dict.thinktrader.net/nativeApi/xttrader.html#%E6%92%A4%E5%8D%95%E5%A4%B1%E8%B4%A5%E9%94%99%E8%AF%AFxtcancelerror) 撤单失败信息
- 返回
  - 无
- 备注
  - 无

#### 异步下单回报推送

```
on_order_stock_async_response(data)
```

- 释义
  - 异步下单回报推送
- 参数
  - data - [XtOrderResponse](http://dict.thinktrader.net/nativeApi/xttrader.html#%E5%BC%82%E6%AD%A5%E4%B8%8B%E5%8D%95%E5%A7%94%E6%89%98%E5%8F%8D%E9%A6%88xtorderresponse) 异步下单委托反馈
- 返回
  - 无
- 备注
  - 无

#### 约券相关异步接口的回报推送

```
on_smt_appointment_async_response(data)
```

- 释义
  - 异步约券相关接口回报推送
- 参数
  - data - [XtSmtAppointmentResponse](http://dict.thinktrader.net/nativeApi/xttrader.html#%E7%BA%A6%E5%88%B8%E7%9B%B8%E5%85%B3%E5%BC%82%E6%AD%A5%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%8F%8D%E9%A6%88xtsmtappointmentresponse) 约券相关异步接口的反馈
- 返回
  - 无
- 备注
  
  ## 行情示例
  
  ### 获取行情示例
  
  新手示例
  
  ```
  # 用前须知
  
  ## xtdata提供和MiniQmt的交互接口，本质是和MiniQmt建立连接，由MiniQmt处理行情数据请求，再把结果回传返回到python层。使用的行情服务器以及能获取到的行情数据和MiniQmt是一致的，要检查数据或者切换连接时直接操作MiniQmt即可。
  
  ## 对于数据获取接口，使用时需要先确保MiniQmt已有所需要的数据，如果不足可以通过补充数据接口补充，再调用数据获取接口获取。
  
  ## 对于订阅接口，直接设置数据回调，数据到来时会由回调返回。订阅接收到的数据一般会保存下来，同种数据不需要再单独补充。
  
  # 代码讲解
  
  # 从本地python导入xtquant库，如果出现报错则说明安装失败
  from xtquant import xtdata
  import time
  
  # 设定一个标的列表
  code_list = ["000001.SZ"]
  # 设定获取数据的周期
  period = "1d"
  
  # 下载标的行情数据
  if 1:
      ## 为了方便用户进行数据管理，xtquant的大部分历史数据都是以压缩形式存储在本地的
      ## 比如行情数据，需要通过download_history_data下载，财务数据需要通过
      ## 所以在取历史数据之前，我们需要调用数据下载接口，将数据下载到本地
      for i in code_list:
          xtdata.download_history_data(i,period=period,incrementally=True) # 增量下载行情数据（开高低收,等等）到本地
  
      xtdata.download_financial_data(code_list) # 下载财务数据到本地
      xtdata.download_sector_data() # 下载板块数据到本地
      # 更多数据的下载方式可以通过数据字典查询
  
  # 读取本地历史行情数据
  history_data = xtdata.get_market_data_ex([],code_list,period=period,count=-1)
  print(history_data)
  print("=" * 20)
  
  # 如果需要盘中的实时行情，需要向服务器进行订阅后才能获取
  # 订阅后，get_market_data函数于get_market_data_ex函数将会自动拼接本地历史行情与服务器实时行情
  
  # 向服务器订阅数据
  for i in code_list:
      xtdata.subscribe_quote(i,period=period,count=-1) # 设置count = -1来取到当天所有实时行情
  
  # 等待订阅完成
  time.sleep(1)
  
  # 获取订阅后的行情
  kline_data = xtdata.get_market_data_ex([],code_list,period=period)
  print(kline_data)
  
  # 获取订阅后的行情，并以固定间隔进行刷新,预期会循环打印10次
  for i in range(10):
      # 这边做演示，就用for来循环了，实际使用中可以用while True
      kline_data = xtdata.get_market_data_ex([],code_list,period=period)
      print(kline_data)
      time.sleep(3) # 三秒后再次获取行情
  
  # 如果不想用固定间隔触发，可以以用订阅后的回调来执行
  # 这种模式下当订阅的callback回调函数将会异步的执行，每当订阅的标的tick发生变化更新，callback回调函数就会被调用一次
  # 本地已有的数据不会触发callback
  
  # 定义的回测函数
      ## 回调函数中，data是本次触发回调的数据，只有一条
  def f(data):
      # print(data)
  
      code_list = list(data.keys())    # 获取到本次触发的标的代码
  
      kline_in_callabck = xtdata.get_market_data_ex([],code_list,period = period)    # 在回调中获取klines数据
      print(kline_in_callabck)
  
  for i in code_list:
      xtdata.subscribe_quote(i,period=period,count=-1,callback=f) # 订阅时设定回调函数
  
  # 使用回调时，必须要同时使用xtdata.run()来阻塞程序，否则程序运行到最后一行就直接结束退出了。
  xtdata.run()
  ```
  
  ### 连接VIP服务器
  
  python
  
  ```
  # 导入 xtdatacenter 模块
  import sys
  
  print("Python 版本：", sys.version)
  
  
  import time
  import pandas as pd
  from xtquant import xtdatacenter as xtdc
  from xtquant import xtdata
  '''  
  设置用于登录行情服务的token，此接口应该先于 init_quote 调用
  
  token可以从投研用户中心获取
  https://xuntou.net/#/userInfo
  '''
  xtdc.set_token('这里输入token')
  
  '''
  设置连接池,使服务器只在连接池内优选
  
  建议将VIP服务器设为连接池
  '''
  addr_list = [
      '115.231.218.73:55310', 
      '115.231.218.79:55310', 
      '42.228.16.211:55300',
      '42.228.16.210:55300',
      '36.99.48.20:55300',
      '36.99.48.21:55300'
      ]
  xtdc.set_allow_optmize_address(addr_list)
  
  xtdc.set_kline_mirror_enabled(True) # 开启K线全推功能(vip),以获取全市场实时K线数据
  
  
  """
  初始化
  """
  xtdc.init()
  ## 监听端口
  port = xtdc.listen(port = 58621) # 指定固定端口进行连接
  # port = xtdc.listen(port = (58620, 58630))[1] 通过指定port范围，可以让xtdc在范围内自动寻找可用端口
  
  xtdata.connect(port=port)
  
  print('-----连接上了------')
  print(xtdata.data_dir)
  
  
  
  servers = xtdata.get_quote_server_status()
  # print(servers)
  for k, v in servers.items():
      print(k, v)
  
  xtdata.run()
  ```
  
  ### 连接指定服务器
  
  python
  
  ```
  import time
  from xtquant import xtdata
  
  #用token方式连接，不需要账号密码
  #其他连接方式，需要账号密码
  info = {"ip": '115.231.218.73', "port": 55300, "username": '', "pwd": ''}
  
  connect_success = 0
  def func(d):
      ip = d.get('ip', '')
      port = d.get('port')
      status = d.get('status', 'disconnected')
  
      global connect_success
      if ip == info['ip'] and port == info['port']:
          if status == 'connected':
              connect_success = 1
          else:
              connect_success = 2
  
  # 注册连接回调信息
  xtdata.watch_quote_server_status(func)
  
  # 行情连接
  qs = xtdata.QuoteServer(info)
  qs.connect()
  
  # 获取当前数据连接站点
  data_server_info = xtdata.get_quote_server_status()
  # 显示当前数据连接站点
  if 1:
      for k,v in data_server_info.items():
          print(f"data:{k}, connect info:{v.info}")
  
  
  # 等待连接状态
  while connect_success == 0:
      time.sleep(0.3)
  
  if connect_success == 2:
      print("连接失败")
  ```
  
  ### 指定初始化行情连接范围
  
  python
  
  ```
  if 1:
      from xtquant import xtdatacenter as xtdc
  
      ## 设置数据目录
      xtdc.set_data_home_dir('data')
  
      ## 设置token
      token = "你的token"
      xtdc.set_token(token)
  
      ## 限定行情站点的优选范围
      opt_list = [
          '115.231.218.73:55310',
          '115.231.218.79:55310',
          '42.228.16.210:55300',
          '42.228.16.211:55300',
          '36.99.48.20:55300',
          '36.99.48.21:55300',
      ]
      xtdc.set_allow_optmize_address(opt_list)
  
      ## 开启指定市场的K线全推
      xtdc.set_kline_mirror_markets(['SH', 'SZ', 'BJ'])
  
      ## 设置要初始化的市场列表
      init_markets = [
          'SH', 'SZ', 'BJ',
          #'DF', 'GF', 'IF', 'SF', 'ZF', 'INE',
          #'SHO', 'SZO',
      ]
      xtdc.set_init_markets(init_markets)
  
      ## 初始化xtdc模块
      xtdc.init(start_local_service = False)
  
      ## 监听端口
      #xtdc.listen(port = 58620)
      listen_port = xtdc.listen(port = (58620, 58650))
  
      #import code; code.interact(local = locals())
  
  
  import xtquant.xtdata as xtdata
  
  xtdata.connect(port = listen_port)
  
  
  
  import code; code.interact(local = locals())
  ```
  
  ### 订阅全推数据/下载历史数据
  
  python
  
  ```
  # coding:utf-8
  import time
  
  from xtquant import xtdata
  
  code = '600000.SH'
  
  #取全推数据
  full_tick = xtdata.get_full_tick([code])
  print('全推数据 日线最新值', full_tick)
  
  #下载历史数据 下载接口本身不返回数据
  xtdata.download_history_data(code, period='1m', start_time='20230701')
  
  #订阅最新行情
  def callback_func(data):
      print('回调触发', data)
  
  xtdata.subscribe_quote(code, period='1m', count=-1, callback= callback_func)
  data = xtdata.get_market_data(['close'], [code], period='1m', start_time='20230701')
  print('一次性取数据', data)
  
  #死循环 阻塞主线程退出
  xtdata.run()
  ```
  
  ### 获取对手价
  
  python返回值
  
  ```
  # 以卖出为例
  
  import pandas as pd
  import numpy as np
  from xtquant import xtdata
  
  to_do_trade_list = ["000001.SZ"]
  tick = xtdata.get_full_tick(to_do_trade_list)
  
  
  # 取买一价为对手价，若买一价为0，说明已经跌停，则取最新价
  for i in tick:
      fix_price = tick[i]["bidPrice"][0] if tick[i]["bidPrice"][0] != 0 else tick[i]["lastPrice"]
      print(fix_price)
  ```
  
  ### 复权计算方式
  
  python
  
  ```
  #coding:utf-8
  
  import numpy as np
  import pandas as pd
  
  from xtquant import xtdata
  
  #def gen_divid_ratio(quote_datas, divid_datas):
  #    drl = []
  #    for qi in range(len(quote_datas)):
  #        q = quote_datas.iloc[qi]
  #        dr = 1.0
  #        for di in range(len(divid_datas)):
  #            d = divid_datas.iloc[di]
  #            if d.name <= q.name:
  #                dr *= d['dr']
  #        drl.append(dr)
  #    return pd.DataFrame(drl, index = quote_datas.index, columns = quote_datas.columns)
  
  def gen_divid_ratio(quote_datas, divid_datas):
      drl = []
      dr = 1.0
      qi = 0
      qdl = len(quote_datas)
      di = 0
      ddl = len(divid_datas)
      while qi < qdl and di < ddl:
          qd = quote_datas.iloc[qi]
          dd = divid_datas.iloc[di]
          if qd.name >= dd.name:
              dr *= dd['dr']
              di += 1
          if qd.name <= dd.name:
              drl.append(dr)
              qi += 1
      while qi < qdl:
          drl.append(dr)
          qi += 1
      return pd.DataFrame(drl, index = quote_datas.index, columns = quote_datas.columns)
  
  def process_forward_ratio(quote_datas, divid_datas):
      drl = gen_divid_ratio(quote_datas, divid_datas)
      drlf = drl / drl.iloc[-1]
      result = (quote_datas * drlf).apply(lambda x: round(x, 2))
      return result
  
  def process_backward_ratio(quote_datas, divid_datas):
      drl = gen_divid_ratio(quote_datas, divid_datas)
      result = (quote_datas * drl).apply(lambda x: round(x, 2))
      return result
  
  def process_forward(quote_datas1, divid_datas):
      quote_datas = quote_datas1.copy()
      def calc_front(v, d):
          return ((v - d['interest'] + d['allotPrice'] * d['allotNum'])
              / (1 + d['allotNum'] + d['stockBonus'] + d['stockGift']))
      for qi in range(len(quote_datas)):
          q = quote_datas.iloc[qi]
          for di in range(len(divid_datas)):
              d = divid_datas.iloc[di]
              if d.name <= q.name:
                  continue
              q.iloc[0] = calc_front(q.iloc[0], d)
      return quote_datas
  
  def process_backward(quote_datas1, divid_datas):
      quote_datas = quote_datas1.copy()
      def calc_back(v, d):
          return ((v * (1.0 + d['stockGift'] + d['stockBonus'] + d['allotNum'])
              + d['interest'] - d['allotNum'] * d['allotPrice']))
      for qi in range(len(quote_datas)):
          q = quote_datas.iloc[qi]
          for di in range(len(divid_datas) - 1, -1, -1):
              d = divid_datas.iloc[di]
              if d.name > q.name:
                  continue
              q.iloc[0] = calc_back(q.iloc[0], d)
      return quote_datas
  
  
  #--------------------------------
  
  s = '002594.SZ'
  
  #xtdata.download_history_data(s, '1d', '20100101', '')
  
  dd = xtdata.get_divid_factors(s)
  print(dd)
  
  #复权计算用于处理价格字段
  field_list = ['open', 'high', 'low', 'close']
  datas_ori = xtdata.get_market_data(field_list, [s], '1d', dividend_type = 'none')['close'].T
  #print(datas_ori)
  
  #等比前复权
  datas_forward_ratio = process_forward_ratio(datas_ori, dd)
  print('datas_forward_ratio', datas_forward_ratio)
  
  #等比后复权
  datas_backward_ratio = process_backward_ratio(datas_ori, dd)
  print('datas_backward_ratio', datas_backward_ratio)
  
  #前复权
  datas_forward = process_forward(datas_ori, dd)
  print('datas_forward', datas_forward)
  
  #后复权
  datas_backward = process_backward(datas_ori, dd)
  print('datas_backward', datas_backward)
  ```
  
  ### 根据商品期货期权代码获取对应的商品期货合约代码
  
  python返回值
  
  ```
  from xtquant import xtdata
  
  def get_option_underline_code(code:str) -> str:
      """
      注意：该函数不适用于股指期货期权与ETF期权
      Todo: 根据商品期权代码获取对应的具体商品期货合约
      Args:
          code:str 期权代码
      Return:
          对应的期货合约代码
      """
      Exchange_dict = {
          "SHFE":"SF",
          "CZCE":"ZF",
          "DCE":"DF",
          "INE":"INE",
          "GFEX":"GF"
      }
  
      if code.split(".")[-1] not in [v for k,v in Exchange_dict.items()]:
          raise KeyError("此函数不支持该交易所合约")
      info = xtdata.get_option_detail_data(code)
      underline_code = info["OptUndlCode"] + "." + Exchange_dict[info["OptUndlMarket"]]
  
      return underline_code
  
  if __name__ == "__main__":
  
      symbol_code = get_option_underline_code('sc2403C465.INE') # 获取期权合约'sc2403C465.INE'对应的期货合约代码
      print(symbol_code)
  ```
  
  ### 根据指数代码，返回对应的期货合约
  
  python返回值
  
  ```
  from xtquant import xtdata
  import re
  
  def get_financial_futures_code_from_index(index_code:str) -> list:
      """
      ToDo:传入指数代码，返回对应的期货合约（当前）
      Args:
          index_code:指数代码，如"000300.SH","000905.SH"
      Retuen:
          list: 对应期货合约列表
      """
      financial_futures = xtdata.get_stock_list_in_sector("中金所")
      future_list = []
      pattern = r'^[a-zA-Z]{1,2}\d{3,4}\.[A-Z]{2}$'
      for i in financial_futures:
  
          if re.match(pattern,i):
              future_list.append(i)
      ls = []
      for i in future_list:
          _info = xtdata._get_instrument_detail(i)
          _index_code = _info["ExtendInfo"]['OptUndlCode'] + "." + _info["ExtendInfo"]['OptUndlMarket']
          if _index_code == index_code:
              ls.append(i)
      return ls
  
  if __name__ == "__main__":
      ls = get_financial_futures_code_from_index("000905.SH")
      print(ls)
  ```
  
  ## 交易示例
  
  ### 简单买卖各一笔示例
  
  **需要调整的参数：**
  - `98`行的`path`变量需要改为本地客户端路径,券商端指定到 f"{安装目录}\userdata_mini",投研端指定到f"{安装目录}\userdata"
  - `107`行的资金账号需要调整为自身资金账号
  
  ```
  # coding:utf-8
  import time, datetime, traceback, sys
  from xtquant import xtdata
  from xtquant.xttrader import XtQuantTrader, XtQuantTraderCallback
  from xtquant.xttype import StockAccount
  from xtquant import xtconstant
  
  
  # 定义一个类 创建类的实例 作为状态的容器
  class _a():
      pass
  
  
  A = _a()
  A.bought_list = []
  A.hsa = xtdata.get_stock_list_in_sector('沪深A股')
  
  
  def interact():
      """执行后进入repl模式"""
      import code
      code.InteractiveConsole(locals=globals()).interact()
  
  
  xtdata.download_sector_data()
  
  
  
  class MyXtQuantTraderCallback(XtQuantTraderCallback):
      def on_disconnected(self):
          """
          连接断开
          :return:
          """
          print(datetime.datetime.now(), '连接断开回调')
  
      def on_stock_order(self, order):
          """
          委托回报推送
          :param order: XtOrder对象
          :return:
          """
          print(datetime.datetime.now(), '委托回调 投资备注', order.order_remark)
  
      def on_stock_trade(self, trade):
          """
          成交变动推送
          :param trade: XtTrade对象
          :return:
          """
          print(datetime.datetime.now(), '成交回调', trade.order_remark, f"委托方向(48买 49卖) {trade.offset_flag} 成交价格 {trade.traded_price} 成交数量 {trade.traded_volume}")
  
      def on_order_error(self, order_error):
          """
          委托失败推送
          :param order_error:XtOrderError 对象
          :return:
          """
          # print("on order_error callback")
          # print(order_error.order_id, order_error.error_id, order_error.error_msg)
          print(f"委托报错回调 {order_error.order_remark} {order_error.error_msg}")
  
      def on_cancel_error(self, cancel_error):
          """
          撤单失败推送
          :param cancel_error: XtCancelError 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
      def on_order_stock_async_response(self, response):
          """
          异步下单回报推送
          :param response: XtOrderResponse 对象
          :return:
          """
          print(f"异步委托回调 投资备注: {response.order_remark}")
  
      def on_cancel_order_stock_async_response(self, response):
          """
          :param response: XtCancelOrderResponse 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
      def on_account_status(self, status):
          """
          :param response: XtAccountStatus 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
  
  if __name__ == '__main__':
      print("start")
      # 指定客户端所在路径, 券商端指定到 userdata_mini文件夹
      # 注意：如果是连接投研端进行交易，文件目录需要指定到f"{安装目录}\userdata"
      path = r'D:\qmt\投研\迅投极速交易终端睿智融科版\userdata'
      # 生成session id 整数类型 同时运行的策略不能重复
      session_id = int(time.time())
      xt_trader = XtQuantTrader(path, session_id)
      # 开启主动请求接口的专用线程 开启后在on_stock_xxx回调函数里调用XtQuantTrader.query_xxx函数不会卡住回调线程，但是查询和推送的数据在时序上会变得不确定
      # 详见: http://docs.thinktrader.net/vip/pages/ee0e9b/#开启主动请求接口的专用线程
      # xt_trader.set_relaxed_response_order_enabled(True)
  
      # 创建资金账号为 800068 的证券账号对象 股票账号为STOCK 信用CREDIT 期货FUTURE
      acc = StockAccount('2000128', 'STOCK')
      # 创建交易回调类对象，并声明接收回调
      callback = MyXtQuantTraderCallback()
      xt_trader.register_callback(callback)
      # 启动交易线程
      xt_trader.start()
      # 建立交易连接，返回0表示连接成功
      connect_result = xt_trader.connect()
      print('建立交易连接，返回0表示连接成功', connect_result)
      # 对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功
      subscribe_result = xt_trader.subscribe(acc)
      print('对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功', subscribe_result)
      #取账号信息
      account_info = xt_trader.query_stock_asset(acc)
      #取可用资金
      available_cash = account_info.m_dCash
  
      print(acc.account_id, '可用资金', available_cash)
      #查账号持仓
      positions = xt_trader.query_stock_positions(acc)
      #取各品种 总持仓 可用持仓
      position_total_dict = {i.stock_code : i.m_nVolume for i in positions}
      position_available_dict = {i.stock_code : i.m_nCanUseVolume for i in positions}
      print(acc.account_id, '持仓字典', position_total_dict)
      print(acc.account_id, '可用持仓字典', position_available_dict)
  
      #买入 浦发银行 最新价 两万元
      stock = '600000.SH'
      target_amount = 20000
      full_tick = xtdata.get_full_tick([stock])
      print(f"{stock} 全推行情： {full_tick}")
      current_price = full_tick[stock]['lastPrice']
      #买入金额 取目标金额 与 可用金额中较小的
      buy_amount = min(target_amount, available_cash)
      #买入数量 取整为100的整数倍
      buy_vol = int(buy_amount / current_price / 100) * 100
      print(f"当前可用资金 {available_cash} 目标买入金额 {target_amount} 买入股数 {buy_vol}股")
      async_seq = xt_trader.order_stock_async(acc, stock, xtconstant.STOCK_BUY, buy_vol, xtconstant.FIX_PRICE, current_price,
                                              'strategy_name', stock)
  
      #卖出 500股
      stock = '513130.SH'
      #目标数量
      target_vol = 500
      #可用数量
      available_vol = position_available_dict[stock] if stock in position_available_dict else 0
      #卖出量取目标量与可用量中较小的
      sell_vol = min(target_vol, available_vol)
      print(f"{stock} 目标卖出量 {target_vol} 可用数量 {available_vol} 卖出 {sell_vol}股")
      if sell_vol > 0:
          async_seq = xt_trader.order_stock_async(acc, stock, xtconstant.STOCK_SELL, sell_vol, xtconstant.LATEST_PRICE,
                                                  -1,
                                                  'strategy_name', stock)
      print(f"下单完成 等待回调")
      # 阻塞主线程退出
      xt_trader.run_forever()
      # 如果使用vscode pycharm等本地编辑器 可以进入交互模式 方便调试 （把上一行的run_forever注释掉 否则不会执行到这里）
      interact()
  ```
  
  ### 单股订阅实盘示例
  
  **需要调整的参数：**
  
  - `113`行的`path`变量需要改为本地客户端路径,券商端指定到 f"{安装目录}\userdata_mini",投研端指定到f"{安装目录}\userdata"
  - `122`行的资金账号需要调整为自身资金账号
  
  python
  
  ```
  # coding:utf-8
  import time, datetime, traceback, sys
  from xtquant import xtdata
  from xtquant.xttrader import XtQuantTrader, XtQuantTraderCallback
  from xtquant.xttype import StockAccount
  from xtquant import xtconstant
  
  
  # 定义一个类 创建类的实例 作为状态的容器
  class _a():
      pass
  
  
  A = _a()
  A.bought_list = []
  A.hsa = xtdata.get_stock_list_in_sector('沪深A股')
  
  
  def interact():
      """执行后进入repl模式"""
      import code
      code.InteractiveConsole(locals=globals()).interact()
  
  
  xtdata.download_sector_data()
  
  
  def f(data):
      print(data)
      now = datetime.datetime.now()
      for stock in data:
          if stock not in A.hsa:
              continue
          cuurent_price = data[stock][0]['close']
          pre_price = data[stock][0]['preClose']
          ratio = cuurent_price / pre_price - 1 if pre_price > 0 else 0
          if ratio > 0.09 and stock not in A.bought_list:
              print(f"{now} 最新价 买入 {stock} 100股")
              async_seq = xt_trader.order_stock_async(acc, stock, xtconstant.STOCK_BUY, 100, xtconstant.LATEST_PRICE, -1,
                                                      'strategy_name', stock)
              A.bought_list.append(stock)
  
  
  class MyXtQuantTraderCallback(XtQuantTraderCallback):
      def on_disconnected(self):
          """
          连接断开
          :return:
          """
          print(datetime.datetime.now(), '连接断开回调')
  
      def on_stock_order(self, order):
          """
          委托回报推送
          :param order: XtOrder对象
          :return:
          """
          print(datetime.datetime.now(), '委托回调', order.order_remark)
  
      def on_stock_trade(self, trade):
          """
          成交变动推送
          :param trade: XtTrade对象
          :return:
          """
          print(datetime.datetime.now(), '成交回调', trade.order_remark)
  
      def on_order_error(self, order_error):
          """
          委托失败推送
          :param order_error:XtOrderError 对象
          :return:
          """
          # print("on order_error callback")
          # print(order_error.order_id, order_error.error_id, order_error.error_msg)
          print(f"委托报错回调 {order_error.order_remark} {order_error.error_msg}")
  
      def on_cancel_error(self, cancel_error):
          """
          撤单失败推送
          :param cancel_error: XtCancelError 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
      def on_order_stock_async_response(self, response):
          """
          异步下单回报推送
          :param response: XtOrderResponse 对象
          :return:
          """
          print(f"异步委托回调 {response.order_remark}")
  
      def on_cancel_order_stock_async_response(self, response):
          """
          :param response: XtCancelOrderResponse 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
      def on_account_status(self, status):
          """
          :param response: XtAccountStatus 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
  
  if __name__ == '__main__':
      print("start")
      # 指定客户端所在路径, 券商端指定到 userdata_mini文件夹
      # 注意：如果是连接投研端进行交易，文件目录需要指定到f"{安装目录}\userdata"
      path = r'D:\qmt\投研\迅投极速交易终端睿智融科版\userdata'
      # 生成session id 整数类型 同时运行的策略不能重复
      session_id = int(time.time())
      xt_trader = XtQuantTrader(path, session_id)
      # 开启主动请求接口的专用线程 开启后在on_stock_xxx回调函数里调用XtQuantTrader.query_xxx函数不会卡住回调线程，但是查询和推送的数据在时序上会变得不确定
      # 详见: http://docs.thinktrader.net/vip/pages/ee0e9b/#开启主动请求接口的专用线程
      # xt_trader.set_relaxed_response_order_enabled(True)
  
      # 创建资金账号为 800068 的证券账号对象 股票账号为STOCK 信用CREDIT 期货FUTURE
      acc = StockAccount('2000128', 'STOCK')
      # 创建交易回调类对象，并声明接收回调
      callback = MyXtQuantTraderCallback()
      xt_trader.register_callback(callback)
      # 启动交易线程
      xt_trader.start()
      # 建立交易连接，返回0表示连接成功
      connect_result = xt_trader.connect()
      print('建立交易连接，返回0表示连接成功', connect_result)
      # 对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功
      subscribe_result = xt_trader.subscribe(acc)
      print('对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功', subscribe_result)
  
      #订阅的品种列表
      code_list = ['600000.SH', '000001.SZ']
  
      for code in code_list:
          xtdata.subscribe_quote(code, '1d', callback = f)
  
      # 阻塞主线程退出
      xt_trader.run_forever()
      # 如果使用vscode pycharm等本地编辑器 可以进入交互模式 方便调试 （把上一行的run_forever注释掉 否则不会执行到这里）
      interact()
  ```
  
  ### 全推订阅实盘示例
  
  本示例用于展示如何订阅上海及深圳市场全推，对于沪深A股品种策略进行判断当前涨幅超过 9 个点的买入 200 股
  
  **需要调整的参数：**
  
  - `111`行的`path`变量需要改为本地客户端路径
  - `116`行的资金账号需要调整为自身资金账号
  
  注意
  
  本策略只用于提供策略写法及参考，若您直接进行实盘下单，造成损失本网站不负担责任。
  
  python
  
  ```
  #coding:utf-8
  import time, datetime, traceback, sys
  from xtquant import xtdata
  from xtquant.xttrader import XtQuantTrader, XtQuantTraderCallback
  from xtquant.xttype import StockAccount
  from xtquant import xtconstant
  
  #定义一个类 创建类的实例 作为状态的容器
  class _a():
      pass
  A = _a()
  A.bought_list = []
  A.hsa = xtdata.get_stock_list_in_sector('沪深A股')
  
  def interact():
      """执行后进入repl模式"""
      import code
      code.InteractiveConsole(locals=globals()).interact()
  xtdata.download_sector_data()
  
  def f(data):
      now = datetime.datetime.now()
      for stock in data:
          if stock not in A.hsa:
              continue
          cuurent_price = data[stock][0]['lastPrice']
          pre_price = data[stock][0]['lastClose']
          ratio = cuurent_price / pre_price - 1 if pre_price > 0 else 0
          if ratio > 0.09 and stock not in A.bought_list:
              print(f"{now} 最新价 买入 {stock} 200股")
              async_seq = xt_trader.order_stock_async(acc, stock, xtconstant.STOCK_BUY, 200, xtconstant.LATEST_PRICE, -1, 'strategy_name', stock)
              A.bought_list.append(stock)
  
  class MyXtQuantTraderCallback(XtQuantTraderCallback):
      def on_disconnected(self):
          """
          连接断开
          :return:
          """
          print(datetime.datetime.now(),'连接断开回调')
  
      def on_stock_order(self, order):
          """
          委托回报推送
          :param order: XtOrder对象
          :return:
          """
          print(datetime.datetime.now(), '委托回调', order.order_remark)
  
  
      def on_stock_trade(self, trade):
          """
          成交变动推送
          :param trade: XtTrade对象
          :return:
          """
          print(datetime.datetime.now(), '成交回调', trade.order_remark)
  
  
      def on_order_error(self, order_error):
          """
          委托失败推送
          :param order_error:XtOrderError 对象
          :return:
          """
          # print("on order_error callback")
          # print(order_error.order_id, order_error.error_id, order_error.error_msg)
          print(f"委托报错回调 {order_error.order_remark} {order_error.error_msg}")
  
      def on_cancel_error(self, cancel_error):
          """
          撤单失败推送
          :param cancel_error: XtCancelError 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
      def on_order_stock_async_response(self, response):
          """
          异步下单回报推送
          :param response: XtOrderResponse 对象
          :return:
          """
          print(f"异步委托回调 {response.order_remark}")
  
      def on_cancel_order_stock_async_response(self, response):
          """
          :param response: XtCancelOrderResponse 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
      def on_account_status(self, status):
          """
          :param response: XtAccountStatus 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
  
  if __name__ == '__main__':
      print("start")
      #指定客户端所在路径,
      # 注意：如果是连接投研端进行交易，文件目录需要指定到f"{安装目录}\userdata"
      path = r'D:\qmt\sp3\迅投极速交易终端 睿智融科版\userdata_mini'
      # 生成session id 整数类型 同时运行的策略不能重复
      session_id = int(time.time())
      xt_trader = XtQuantTrader(path, session_id)
      # 开启主动请求接口的专用线程 开启后在on_stock_xxx回调函数里调用XtQuantTrader.query_xxx函数不会卡住回调线程，但是查询和推送的数据在时序上会变得不确定
      # 详见: http://docs.thinktrader.net/vip/pages/ee0e9b/#开启主动请求接口的专用线程
      # xt_trader.set_relaxed_response_order_enabled(True)
  
      # 创建资金账号为 800068 的证券账号对象
      acc = StockAccount('800068', 'STOCK')
      # 创建交易回调类对象，并声明接收回调
      callback = MyXtQuantTraderCallback()
      xt_trader.register_callback(callback)
      # 启动交易线程
      xt_trader.start()
      # 建立交易连接，返回0表示连接成功
      connect_result = xt_trader.connect()
      print('建立交易连接，返回0表示连接成功', connect_result)
      # 对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功
      subscribe_result = xt_trader.subscribe(acc)
      print('对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功', subscribe_result)
  
      #这一行是注册全推回调函数 包括下单判断 安全起见处于注释状态 确认理解效果后再放开
      # xtdata.subscribe_whole_quote(["SH", "SZ"], callback=f)
      # 阻塞主线程退出
      xt_trader.run_forever()
      # 如果使用vscode pycharm等本地编辑器 可以进入交互模式 方便调试 （把上一行的run_forever注释掉 否则不会执行到这里）
      interact()
  ```
  
  ### 定时判断实盘示例
  
  python
  
  ```
  # coding:utf-8
  import time, datetime, traceback, sys
  from xtquant import xtdata
  from xtquant.xttrader import XtQuantTrader, XtQuantTraderCallback
  from xtquant.xttype import StockAccount
  from xtquant import xtconstant
  
  
  # 定义一个类 创建类的实例 作为状态的容器
  class _a():
      pass
  
  
  A = _a()
  A.bought_list = []
  A.hsa = xtdata.get_stock_list_in_sector('沪深A股')
  
  
  def interact():
      """执行后进入repl模式"""
      import code
      code.InteractiveConsole(locals=globals()).interact()
  
  
  xtdata.download_sector_data()
  
  
  def f(data):
      now = datetime.datetime.now()
      # print(data)
      for stock in data:
          if stock not in A.hsa:
              continue
          cuurent_price = data[stock].iloc[-1, 0]
          pre_price = data[stock].iloc[-2, 0]
          ratio = cuurent_price / pre_price - 1 if pre_price > 0 else 0
          if ratio > 0.09 and stock not in A.bought_list:
              print(f"{now} 最新价 买入 {stock} 100股")
              async_seq = xt_trader.order_stock_async(acc, stock, xtconstant.STOCK_BUY, 100, xtconstant.LATEST_PRICE, -1,
                                                      'strategy_name', stock)
              A.bought_list.append(stock)
  
  
  class MyXtQuantTraderCallback(XtQuantTraderCallback):
      def on_disconnected(self):
          """
          连接断开
          :return:
          """
          print(datetime.datetime.now(), '连接断开回调')
  
      def on_stock_order(self, order):
          """
          委托回报推送
          :param order: XtOrder对象
          :return:
          """
          print(datetime.datetime.now(), '委托回调', order.order_remark)
  
      def on_stock_trade(self, trade):
          """
          成交变动推送
          :param trade: XtTrade对象
          :return:
          """
          print(datetime.datetime.now(), '成交回调', trade.order_remark)
  
      def on_order_error(self, order_error):
          """
          委托失败推送
          :param order_error:XtOrderError 对象
          :return:
          """
          # print("on order_error callback")
          # print(order_error.order_id, order_error.error_id, order_error.error_msg)
          print(f"委托报错回调 {order_error.order_remark} {order_error.error_msg}")
  
      def on_cancel_error(self, cancel_error):
          """
          撤单失败推送
          :param cancel_error: XtCancelError 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
      def on_order_stock_async_response(self, response):
          """
          异步下单回报推送
          :param response: XtOrderResponse 对象
          :return:
          """
          print(f"异步委托回调 {response.order_remark}")
  
      def on_cancel_order_stock_async_response(self, response):
          """
          :param response: XtCancelOrderResponse 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
      def on_account_status(self, status):
          """
          :param response: XtAccountStatus 对象
          :return:
          """
          print(datetime.datetime.now(), sys._getframe().f_code.co_name)
  
  
  if __name__ == '__main__':
      print("start")
      # 指定客户端所在路径, 券商端指定到 userdata_mini文件夹
      # 注意：如果是连接投研端进行交易，文件目录需要指定到f"{安装目录}\userdata"
      path = r'D:\qmt\投研\迅投极速交易终端睿智融科版\userdata'
      # 生成session id 整数类型 同时运行的策略不能重复
      session_id = int(time.time())
      xt_trader = XtQuantTrader(path, session_id)
      # 开启主动请求接口的专用线程 开启后在on_stock_xxx回调函数里调用XtQuantTrader.query_xxx函数不会卡住回调线程，但是查询和推送的数据在时序上会变得不确定
      # 详见: http://docs.thinktrader.net/vip/pages/ee0e9b/#开启主动请求接口的专用线程
      # xt_trader.set_relaxed_response_order_enabled(True)
  
      # 创建资金账号为 800068 的证券账号对象 股票账号为STOCK 信用CREDIT 期货FUTURE
      acc = StockAccount('2000128', 'STOCK')
      # 创建交易回调类对象，并声明接收回调
      callback = MyXtQuantTraderCallback()
      xt_trader.register_callback(callback)
      # 启动交易线程
      xt_trader.start()
      # 建立交易连接，返回0表示连接成功
      connect_result = xt_trader.connect()
      print('建立交易连接，返回0表示连接成功', connect_result)
      # 对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功
      subscribe_result = xt_trader.subscribe(acc)
      print('对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功', subscribe_result)
  
      #订阅的品种列表
      code_list = ['600000.SH', '000001.SZ']
      #遍历品种 下载历史k线 订阅当日行情
      for code in code_list:
          xtdata.download_history_data(code, period='1d', start_time='20200101')
          xtdata.subscribe_quote(code, '1d', callback = None)
  
      while True:
          now = datetime.datetime.now()
          now_time = now.strftime('%H%M%S')
          if not '093000' <= now_time < '150000':
              print(f"{now} 非交易时间 循环退出")
              break
          #取k线数据
          data = xtdata.get_market_data_ex(['close'], code_list, period= '1d', start_time= '20240101')
          #判断交易
          f(data)
          #每次循环 睡眠三秒后继续
          time.sleep(3)
  
  
      # 阻塞主线程退出
      xt_trader.run_forever()
      # 如果使用vscode pycharm等本地编辑器 可以进入交互模式 方便调试 （把上一行的run_forever注释掉 否则不会执行到这里）
      interact()
  ```
  
  ### 交易接口重连
  
  该示例演示交易连接断开时重连的代码处理。
  
  提示
  
  1. 该示例**不是线程安全**的，仅演示断开连接时应该怎么处理重连代码，实际使用时请注意避免潜在的问题
  2. 本策略只用于提供策略写法及参考，若您直接进行实盘下单，造成损失本网站不负担责任。
  
  python
  
  ```
  #本文用一个均线策略演示交易连接断开时怎么处理交易接口重连
  # 策略本身不严谨，不能作为实盘策略或者参考策略，本策略仅是演示重连用法
  import time
  from xtquant.xttrader import XtQuantTrader, XtQuantTraderCallback
  from xtquant.xttype import StockAccount
  from xtquant import xtconstant
  from xtquant import xtdata
  
  
  class MyXtQuantTraderCallback(XtQuantTraderCallback):
      # 更多说明见 http://dict.thinktrader.net/nativeApi/xttrader.html?id=I3DJ97#%E5%A7%94%E6%89%98xtorder
      def on_disconnected(self):
          """
          连接断开
          :return:
          """
          print("connection lost, 交易接口断开，即将重连")
          global xt_trader
          xt_trader = None
  
      def on_stock_order(self, order):
          print(f'委托回报: 股票代码:{order.stock_code} 账号:{order.account_id}, 订单编号:{order.order_id} 柜台合同编号:{order.order_sysid} \
              委托状态:{order.order_status} 成交数量:{order.order_status} 委托数量:{order.order_volume} 已成数量：{order.traded_volume}')
  
      def on_stock_trade(self, trade):
          print(f'成交回报: 股票代码:{trade.stock_code} 账号:{trade.account_id}, 订单编号:{trade.order_id} 柜台合同编号:{trade.order_sysid} \
              成交编号:{trade.traded_id} 成交数量:{trade.traded_volume} 委托数量:{trade.direction} ')
  
      def on_order_error(self, order_error):
          print(f"报单失败： 订单编号：{order_error.order_id} 下单失败具体信息:{order_error.error_msg} 委托备注:{order_error.order_remark}")
  
      def on_cancel_error(self, cancel_error):
          print(f"撤单失败: 订单编号：{cancel_error.order_id} 失败具体信息:{cancel_error.error_msg} 市场：{cancel_error.market}")
  
      def on_order_stock_async_response(self, response):
          print(f"异步下单的请求序号:{response.seq}, 订单编号：{response.order_id} ")
  
      def on_account_status(self, status):
          print(f"账号状态发生变化， 账号:{status.account_id} 最新状态：{status.status}")
  
  def create_trader(xt_acc,path, session_id):
      trader = XtQuantTrader(path, session_id,callback=MyXtQuantTraderCallback())
      trader.start()
      connect_result = trader.connect()
      trader.subscribe(xt_acc)
      return trader if connect_result == 0 else None
  
  
  def try_connect(xt_acc,path):
      session_id_range = [i for i in range(100, 120)]
  
      import random
      random.shuffle(session_id_range)
  
      # 遍历尝试session_id列表尝试连接
      for session_id in session_id_range:
          trader = create_trader(xt_acc,path, session_id)
          if trader:
              print('连接成功，session_id:{}', session_id)
              return trader
          else:
              print('连接失败，session_id:{}，继续尝试下一个id', session_id)
              continue
  
      print('所有id都尝试后仍失败，放弃连接')
      return None
  
  
  def get_xttrader(xt_acc,path):
      global xt_trader
      if xt_trader is None:
          xt_trader = try_connect(xt_acc,path)
      return xt_trader
  
  
  if __name__ == "__main__":
  
      # 注意实际连接XtQuantTrader时不要写类似while True 这种无限循环的尝试，因为每次连接都会用session_id创建一个对接文件，这样就会占满硬盘导致电脑运行异常
      # 要控制session_id在有限的范围内尝试，这里提供10个session_id供重连尝试
      # 当所有session_id都尝试后，程序会抛出异常。实际使用过程中当session_id用完时，可以增加邮件等通知方式提醒人工处理 
  
      #指定客户端所在路径
      path = 'E:\qmt\\userdata_mini'
      xt_trader = None
      xt_acc = StockAccount('2000204')
      xt_trader = get_xttrader(xt_acc,path)
      if not xt_trader:
          raise Exception('交易接口连接失败')
      print('交易接口连接成功， 策略开始')
  
      stock = '513050.SH'
      xtdata.subscribe_quote(stock, '5m','','',count=-1)
      time.sleep(1)
      order_record = []
      while '093000'<=time.strftime('%H%M%S')<'150000':
          time.sleep(3)
          xt_trader = get_xttrader(xt_acc,path)
  
          price = xtdata.get_market_data_ex(['close'],[stock],period='5m',)[stock]
          #计算均线
          ma5 = price['close'].rolling(5).mean()
          ma10 = price['close'].rolling(10).mean()
  
          if ma5.iloc[-1]>ma5.iloc[-10]:
              t = price.index[-1]
              order_flag = (t, '买')
              if order_flag not in order_record: #防止重复下单
                  print(f'发起买入 {stock}  k线时间:{t}')
  
                  # 用最新价买100股
                  xt_trader.order_stock_async(xt_acc, stock, xtconstant.STOCK_BUY,100,xtconstant.LATEST_PRICE,0)
                  order_record.append(order_flag)
          elif ma5.iloc[-1]<ma5[-10]:
              t = price.index[-1]
              order_flag = (t, '卖')
              if order_flag not in order_record: #防止重复下单
                  print(f'发起卖出 {stock} k线时间:{t}')
                  # 用最新价买100股
                  xt_trader.order_stock_async(xt_acc, stock, xtconstant.STOCK_SELL,100,xtconstant.LATEST_PRICE,0)
  
                  order_record.append(order_flag)
  ```
  
  ### 指定session id范围连接交易
  
  该示例演示指定session重试连接次数的代码处理。
  
  python
  
  ```
  #coding:utf-8
  
  def connect(path, session):
      from xtquant import xttrader
  
      trader = xttrader.XtQuantTrader(path, session)
      trader.start()
  
      connect_result = trader.connect()
      return trader if connect_result == 0 else None
  
  
  def try_connect_range():
      # 随机 session_id 的待尝试列表
      # 100以内的id保留
      ids = [i for i in range(100, 200)]
  
      import random
      random.shuffle(ids)
  
      # 要连接到的对接路径
      path = r'userdata_mini'
  
      # 遍历id列表尝试连接
      for session_id in ids:
          print(f'尝试id:{session_id}')
          trader = connect(path, session_id)
  
          if trader:
              print('连接成功')
              return trader
          else:
              print('连接失败，继续尝试下一个id')
              continue
  
      # 所有id都尝试后仍失败，放弃连接
      raise Exception('XtQuantTrader 连接失败,请重试')
  
  
  try:
      trader = try_connect_range()
  except Exception as e:
      import traceback
      print(e, traceback.format_exc())
  
  
  import time
  while True:
      print('.', end = '')
      time.sleep(2)
  ```
  
  ### 信用账号执行还款
  
  本示例用于展示如何使用xtquant库对信用账号执行还款的操作
  
  提示
  
  本策略只用于提供策略写法及参考，若您直接进行实盘下单，造成损失本网站不负担责任。
  
  python
  
  ```
  #coding=utf-8
  from xtquant.xttrader import XtQuantTrader, XtQuantTraderCallback
  from xtquant.xttype import StockAccount
  from xtquant import xtconstant
  
  # 修改参数
  # path为mini qmt客户端安装目录下userdata_mini路径
  path = 'E:\\qmt\\userdata_mini'
  # session_id为会话编号，策略使用方对于不同的Python策略需要使用不同的会话编号
  session_id = 1234567
  repay_money = 1000.51  # 元，需要执行还款的金额
  
  class MyXtQuantTraderCallback(XtQuantTraderCallback):
      def on_disconnected(self):
          """
          连接断开
          :return:
          """
          print("connection lost")
      def on_stock_order(self, order):
          """
          委托回报推送
          :param order: XtOrder对象
          :return:
          """
          print("on order callback:")
          print(order.stock_code, order.order_status, order.order_sysid)
      def on_stock_asset(self, asset):
          """
          资金变动推送
          :param asset: XtAsset对象
          :return:
          """
          print("on asset callback")
          print(asset.account_id, asset.cash, asset.total_asset)
      def on_stock_trade(self, trade):
          """
          成交变动推送
          :param trade: XtTrade对象
          :return:
          """
          print("on trade callback")
          print(trade.account_id, trade.stock_code, trade.order_id)
      def on_order_error(self, order_error):
          """
          委托失败推送
          :param order_error:XtOrderError 对象
          :return:
          """
          print("on order_error callback")
          print(order_error.order_id, order_error.error_id, order_error.error_msg)
      def on_cancel_error(self, cancel_error):
          """
          撤单失败推送
          :param cancel_error: XtCancelError 对象
          :return:
          """
          print("on cancel_error callback")
          print(cancel_error.order_id, cancel_error.error_id, cancel_error.error_msg)
      def on_order_stock_async_response(self, response):
          """
          异步下单回报推送
          :param response: XtOrderResponse 对象
          :return:
          """
          print("on_order_stock_async_response")
          print(response.account_id, response.order_id, response.seq)
      def on_account_status(self, status):
          """
          :param response: XtAccountStatus 对象
          :return:
          """
          print("on_account_status")
          print(status.account_id, status.account_type, status.status)
  
  
  if __name__ == "__main__":
      print("demo test")
  
  
      xt_trader = XtQuantTrader(path, session_id)
      # 创建资金账号为1000000365的证券账号对象
      acc = StockAccount('200035', 'CREDIT')
      # StockAccount可以用第二个参数指定账号类型，如沪港通传'HUGANGTONG'，深港通传'SHENGANGTONG'
      # acc = StockAccount('1000000365','STOCK')
      # 创建交易回调类对象，并声明接收回调
      callback = MyXtQuantTraderCallback()
      xt_trader.register_callback(callback)
      # 启动交易线程
      xt_trader.start()
      # 建立交易连接，返回0表示连接成功
      connect_result = xt_trader.connect()
      if connect_result != 0:
          import sys
          sys.exit('连接失败，程序即将退出 %d'%connect_result)
      # 对交易回调进行订阅，订阅后可以收到交易主推，返回0表示订阅成功
      subscribe_result = xt_trader.subscribe(acc)
      if subscribe_result != 0:
          print('账号订阅失败 %d'%subscribe_result)
      print(subscribe_result)
      stock_code = '600000.SH'  # 参数占位用，任意股票代码都可以
      volume = 200  # 参数占位用，任意数量
      # 使用指定价下单，接口返回订单编号，后续可以用于撤单操作以及查询委托状态
      fix_result_order_id = xt_trader.order_stock(acc, stock_code, xtconstant.CREDIT_DIRECT_CASH_REPAY, repay_money, xtconstant.FIX_PRICE, -1, 'strategy_name', 'remark')
  
      # 阻塞线程，接收交易推送
      xt_trader.run_forever()
  ```
  
  ### 下单后通过回调撤单
  
  ```
  import pandas as pd
  import numpy as np
  import datetime
  from xtquant import xtdata,xttrader
  from xtquant.xttype import StockAccount
  from xtquant import xtconstant
  from xtquant.xttrader import XtQuantTraderCallback
  import sys
  import time
  
  
  """
  异步下单委托流程为
  1.order_stock_async发出委托
  2.回调on_order_stock_async_response收到回调信息
  3.回调on_stock_order收到委托信息
  4.回调cancel_order_stock_sysid_async发出异步撤单指令
  5.回调on_cancel_order_stock_async_response收到撤单回调信息
  6.回调on_stock_order收到委托信息
  """
  strategy_name = "委托撤单测试"
  
  class MyXtQuantTraderCallback(XtQuantTraderCallback):
      # 用于接收回调信息的类
      def on_stock_order(self, order):
          """
          委托回报推送
          :param order: XtOrder对象
          :return:
          """
          # 属性赋值
          account_type = order.account_type  # 账号类型
          account_id = order.account_id  # 资金账号
          stock_code = order.stock_code  # 证券代码，例如"600000.SH"
          order_id = order.order_id  # 订单编号
          order_sysid = order.order_sysid  # 柜台合同编号
          order_time = order.order_time  # 报单时间
          order_type = order.order_type  # 委托类型，参见数据字典
          order_volume = order.order_volume  # 委托数量
          price_type = order.price_type  # 报价类型，该字段在返回时为柜台返回类型，不等价于下单传入的price_type，枚举值不一样功能一样，参见数据字典
          price = order.price  # 委托价格
          traded_volume = order.traded_volume  # 成交数量
          traded_price = order.traded_price  # 成交均价
          order_status = order.order_status  # 委托状态，参见数据字典
          status_msg = order.status_msg  # 委托状态描述，如废单原因
          strategy_name = order.strategy_name  # 策略名称
          order_remark = order.order_remark  # 委托备注
          direction = order.direction  # 多空方向，股票不适用；参见数据字典
          offset_flag = order.offset_flag  # 交易操作，用此字段区分股票买卖，期货开、平仓，期权买卖等；参见数据字典
  
          # 打印输出
          print(f"""
          =============================
                  委托信息
          =============================
          账号类型: {order.account_type}, 
          资金账号: {order.account_id},
          证券代码: {order.stock_code},
          订单编号: {order.order_id}, 
          柜台合同编号: {order.order_sysid},
          报单时间: {order.order_time},
          委托类型: {order.order_type},
          委托数量: {order.order_volume},
          报价类型: {order.price_type},
          委托价格: {order.price},
          成交数量: {order.traded_volume},
          成交均价: {order.traded_price},
          委托状态: {order.order_status},
          委托状态描述: {order.status_msg},
          策略名称: {order.strategy_name},
          委托备注: {order.order_remark},
          多空方向: {order.direction},
          交易操作: {order.offset_flag}
          """)
          if order.strategy_name == strategy_name:
              # 该委托是由本策略发出
              ssid = order.order_sysid
              status = order.order_status
              market = order.stock_code.split(".")[1]
              # print(ssid)
              if ssid and status in [50,55]:
                  ## 使用cancel_order_stock_sysid_async时，投研端market参数可以填写为0，券商端按实际情况填写
                  print(xt_trade.cancel_order_stock_sysid_async(account,0,ssid))
  
      def on_stock_trade(self, trade):
          """
          成交变动推送
          :param trade: XtTrade对象
          :return:
          """
          print(datetime.datetime.now(), '成交回调', trade.order_remark,trade.stock_code,trade.traded_volume,trade.offset_flag)
  
      def on_order_stock_async_response(self, response):
          """
          异步下单回报推送
          :param response: XtOrderResponse 对象
          :return:
          """
  
          print(datetime.datetime.now(),'异步下单编号为：',response.seq)
  
      def on_cancel_order_stock_async_response(self, response):
          """
          异步撤单回报
          :param response: XtCancelOrderResponse 对象
          :return:
          """
          account_type = response.account_type # 账号类型
          account_id = response.account_id  # 资金账号
          order_id = response.order_id  # 订单编号
          order_sysid = response.order_sysid  # 柜台委托编号
          cancel_result = response.cancel_result  # 撤单结果
          seq = response.seq  # 异步撤单的请求序号
  
          print(f"""
              ===========================
                     异步撤单回调信息
              ===========================
              账号类型: {response.account_type}, 
              资金账号: {response.account_id},
              订单编号: {response.order_id}, 
              柜台委托编号: {response.order_sysid},
              撤单结果: {response.cancel_result},
              异步撤单的请求序号: {response.seq}""")
          pass
  
  
  callback = MyXtQuantTraderCallback()
  # 填投研端的期货账号
  account = StockAccount("1000024",account_type = "FUTURE")
  # 填写投研端的股票账号
  # account = StockAccount("2000567")
  # 填投研端的userdata路径,miniqmt指定到userdata_mini
  xt_trade = xttrader.XtQuantTrader(r"C:\Program Files\测试1\迅投极速交易终端睿智融科版\userdata",int(time.time()))
  # 注册接受回调
  xt_trade.register_callback(callback) 
  # 启动交易线程
  xt_trade.start()
  # 链接交易
  connect_result = xt_trade.connect()
  # 订阅账号信息，接受这个账号的回调，回调是账号维度的
  subscribe_result = xt_trade.subscribe(account)
  print(subscribe_result)
  
  
  code = "rb2410.SF"
  # code = "000001.SZ"
  
  tick = xtdata.get_full_tick([code])[code]
  
  last_price = tick["lastPrice"] # 最新价
  
  ask_price = round(tick["askPrice"][0],3) # 卖方1档价
  bid_price = round(tick["bidPrice"][4],3) # 买方5档价
  
  symbol_info = xtdata.get_instrument_detail(code)
  
  up_limit = symbol_info["UpStopPrice"]
  down_limit = symbol_info["DownStopPrice"]
  
  lots = 1
  res_id = xt_trade.order_stock_async(account, code, xtconstant.FUTURE_OPEN_LONG, lots, xtconstant.FIX_PRICE, down_limit, strategy_name, "跌停价/固定手数")
  
  
  # lots = 100
  # res_id = xt_trade.order_stock_async(account, code, xtconstant.STOCK_BUY, lots, xtconstant.FIX_PRICE, bid_price, strategy_name, "跌停价/固定手数")
  
  
  xtdata.run()
  ```
  
  ### 导入xtquant库时提示 `NO module named 'xtquant.IPythonAPiClient'`
  
  1、目前xtquant支持的python版本为 64位python3.6----3.11，请使用支持的python版本重试
  
  ### 连接 xtquant 时失败，返回-1及解决方法
  
  1. 客户端是否以极简模式登录（登录qmt系统时需要勾选极简模式）
  
  2. 检查路径是否正确
  - `miniqmt`：路径指定到安装目录下`\userdata_mini`文件夹
  
  - `投研端`：路径指定到安装目录下`\userdata`文件夹
  3. 客户端安装在C盘的话，每次都需要用管理员权限运行策略，才能正常连接，否则有权限问题。
  
  提示
  
  不建议安装在C盘。
  
  可以通过以下测试来验证是否有写入权限
  
  ```
  file_path = r"d:\qmt\userdata_mini\example.txt"  # 设置文件路径和名称
  
  # 使用open函数创建文件，并指定写入模式("w"表示写入模式)
  with open(file_path, "w") as file:
      file.write("123")  # 向文件写入内容
  ```
  
  如果出现`PermissionError`，则说明存在文件权限问题
  
  4. 路径正确时换个`session`（任意整数即可）
  
  提示
  
  由于机制限制，同一个session的两次python进程 connect之间必须超过3秒钟
  
  5. 如果miniqmt开启后, userdata_mini文件夹内没有`up_queue_xtquant`文件，说明没有用户没有对应函数下单的权限，需要联系券商开启
  
  ### 执行xtdatacenter.init时提示监听58609端口失败
  
  说明当前环境的58609端口被其他程序占用,通常是启动了两个xtdc服务导致的
  
  **方法1.** 可用通过指定`xtdc.init(False)`后,使用`xtdc.listen(port)`指定自己需要的端口
  
  ```
  from xtquant import xtdatacenter as xtdc 
  xtdc.set_token("这里输入token")
  xtdc.init(False)
  port = 58601
  xtdc.listen(port=port)
  print(f"服务启动,开放端口：{port}")
  ```
  
  **方法2.** 关闭所有py程序，或重启电脑，再执行xtdc.init
  
  ### 下单后，查询委托的投资备注只有前半部分
  
  极简客户端的`order_remark`字段有长度限制，最大 24 个英文字符(一个中文占3个)， 超出的部分会丢弃。大qmt 没有长度限制。
  
  ### userdata_mini目录下，生成大量down_queue文件
  
  该文件是xttrade指定新的session产生的文件，可以参考下方示例来控制session产生的范围，避免该文件大量产生，**该文件可以删除**
  
  [指定session id范围连接交易](https://dict.thinktrader.net/nativeApi/code_examples.html?id=7zqjlm#%E6%8C%87%E5%AE%9Asession-id%E8%8C%83%E5%9B%B4%E8%BF%9E%E6%8E%A5%E4%BA%A4%E6%98%93)
