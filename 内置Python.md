### 一、概述

QMT 极速策略交易系统，以下简称 **QMT 系统**，内置了 **`3.6 版本`** 的 `python` 运行环境，提供**行情数据**与**交易下单**两大核心功能。通过编写 python 脚本，可以完成指标计算，策略编写，策略回测，实盘下单等需求。

### 二、场景需求

QMT 系统支持**回测模型**与**实盘模型**。

![](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%9B%9E%E6%B5%8B%E6%A8%A1%E5%9E%8B_%E5%AE%9E%E7%9B%98%E6%A8%A1%E5%9E%8B-a800c759.png)

**回测模型：** 指在历史 k 线上，自左向右逐根遍历 k 线，以模拟的资金账号记录每日的买卖信号，持仓盈亏，最终展示策略在历史上的净值走势结果。

**实盘模型：** 指在盘中收取最新的动态行情，即时发送买卖信号到交易所，判断委托状态，需要实时重复报撤的模型。

**两类模型分别有各自的注意点：**

#### 回测模型

1. 回测是遍历固定的历史数据：
   
   ![](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E6%93%8D%E4%BD%9C_%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86_%E4%B8%8B%E8%BD%BD%E5%85%A8%E9%83%A8-d3b5864c.png)
   
   - 首先需要下载历史行情，首次下载可以在界面左上角，点击`操作`，选择`数据管理`补充行情，选择回测的周期，如`日线`，所需的板块数据，如`沪深A股板块`，时间范围选择`全部`，下载完整历史行情
   
   ![](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E6%89%B9%E9%87%8F%E4%B8%8B%E8%BD%BD-c98af807.png)
   
   - 其次设置每日定时更新，可以点击客户端右下角`行情`按钮，在`批量下载`界面选择需要每天更新的数据，勾选`定时下载`选项，之后每天在指定时间会自动下载行情数据到本地

2. 回测模型取本地数据遍历，不需要向服务器订阅实时行情，应使用 `get_market_data_ex`函数，指定`subscribe`参数为`False`，来读取本地行情数据。

3. 回测模型的撮合规则为，指定交易价格在当前k线高低点间的，按指定价格撮合，超过高低点的，按当前 k 线收盘价撮合。委托数量大于可用数量时，按可用数量撮合。

![](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF_%E5%9B%9E%E6%B5%8B%E5%8F%82%E6%95%B0-eec55083.png)

4. 回测模型右侧的基本信息，如默认周期，默认主图，在`我的界面`点击回测时会生效。在行情界面k线下点击回测，以当前 k 线的周期，品种为准。回测必须以`副图模式`执行，不要选择主图 /主图叠加.

#### 实盘模型

当你回测结束，你需要开始实盘模型，注意这里提到的实盘，指的是接收未来 K 线的数据，生成策略信号，进行交易下单。

提示

实盘模型也分`模拟柜台模拟交易`和`真实柜台实盘交易`两种。具体请参考[如何配置账号](https://dict.thinktrader.net/innerApi/interface_operation.md#%E9%85%8D%E7%BD%AE%E8%B4%A6%E5%8F%B7)

1. 你要运行实盘模型，QMT 系统提供两种交易模式：
   
   - 默认的交易模式为**逐 k 线生效** (`passorder`函数**快速交易**`quicktrade`参数填 `0` 即默认值)，适用与需要在盘中模拟历史上逐 k 线的效果需求。例如选择一分钟周期，将下单判断，下单函数放在`handlebar`函数内，盘中主图每个分笔 (三秒一次)会触发一次`handlebar`函数调用，系统会暂存当前`handlebar`产生的下单信号。三秒后下一个分笔到达时，如果是新的一分钟 k 线的第一个分笔，判断上一个分笔为前一根k线最后分笔，会将暂存的交易信号发送给交易所，完成交易。如到达的下一个分笔不是新一根 k 线的，则判定当前 k 线未完成，丢弃暂存的交易信号。1 分钟 k 线情形，每根k线内会有 20 个分笔，前 19 个分笔产生的信号会被丢弃，最后一个分笔的信号，会在下一根k线，首个分笔到达时，延迟三秒发出。系统自带的`ContxtInfo`也做了同样的等待，回退处理，逐 k 线模式的交易记录可以保存在`ContextInfo`对象的属性中。详细说明参见 [常见问题：系统对象ContextInfo 逐K线保存的机制](https://dict.thinktrader.net/innerApi/question_answer.html#%E7%B3%BB%E7%BB%9F%E5%AF%B9%E8%B1%A1-contextinfo-%E9%80%90-k-%E7%BA%BF%E4%BF%9D%E5%AD%98%E7%9A%84%E6%9C%BA%E5%88%B6)
   
   - QMT 系统也支持立即下单的交易模式，`passorder`函数的**快速交易**`quicktrade`参数填 `2`，可以在运行后立刻发出委托，不对信号进行等待，丢弃的操作。此时需要用普通的全局变量(如自定义一个`Class a()`)保存委托状态，不能存在`ContextInfo`的属性里。参见[使用快速交易参数委托](https://dict.thinktrader.net/innerApi/code_examples.html#%E4%BD%BF%E7%94%A8%E5%BF%AB%E9%80%9F%E4%BA%A4%E6%98%93%E5%8F%82%E6%95%B0%E5%A7%94%E6%89%98) 、[调整至目标持仓Demo](https://dict.thinktrader.net/innerApi/code_examples.html#%E8%B0%83%E6%95%B4%E8%87%B3%E7%9B%AE%E6%A0%87%E6%8C%81%E4%BB%93)

2. 实盘的撮合规则以交易所为准。股票品种的话，价格不能超过 2% 的价格笼子否则废单。数量超过可用数量时会废单。

3. 实盘模型需要在模型交易界面执行。模型交易界面，选择新建策略交易，添加需要的模型。运行模式可以选择`模拟`或`实盘`。
   
   - 选择`模拟信号模式`，在策略信号界面显示买卖信号，不实际发出委托。具体请参考[模拟信号模式](https://dict.thinktrader.net/innerApi/variable_convention.html#%E6%A8%A1%E6%8B%9F%E4%BF%A1%E5%8F%B7%E6%A8%A1%E5%BC%8F)
   - 选择`实盘交易模式`，显示的策略信号会实际发出到交易所。具体请参考[实盘交易模式](https://dict.thinktrader.net/innerApi/variable_convention.html#%E5%AE%9E%E7%9B%98%E4%BA%A4%E6%98%93%E6%A8%A1%E5%BC%8F)

提示

运行模式的`模拟`和`实盘`，与您使用的账号实际是`实盘账号（真实交易所柜台）`或是`模拟账号（模拟交易柜台）`无关。相关账号申请需要联系您做所在券商的工作人员，或者购买[投研端账号](https://xuntou.net/#/productvip)获取模拟柜台撮合服务。

### 三、运行机制对比

QMT 系统提供两大类(事件驱动与定时任务)，共三种运行机制。

![](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E4%B8%89%E7%A7%8D%E6%9C%BA%E5%88%B6%E5%AF%B9%E6%AF%94-bc32690d.png)

#### 逐 K 线驱动：`handlebar`

`handlebar`是**主图历史 k 线**+**盘中订阅推送**。运行开始时，所选周期历史 k 线从左向右每根触发一次`handlebar`函数调用。盘中时，主图品种每个新分笔数据到达，触发一次`handlebar`函数调用。

提示

盘中分笔驱动，但是逐 K 线生效。请参考[常见问题：系统对象ContextInfo 逐K线保存的机制](https://dict.thinktrader.net/innerApi/question_answer.html#%E7%B3%BB%E7%BB%9F%E5%AF%B9%E8%B1%A1-contextinfo-%E9%80%90-k-%E7%BA%BF%E4%BF%9D%E5%AD%98%E7%9A%84%E6%9C%BA%E5%88%B6)

#### 事件驱动 ：`subscribe` 订阅推送

盘中订阅指定品种的分笔数据，新分笔到达时，触发指定的回调函数。

#### 定时任务 ：`run_time` 定时运行

指定固定的时间间隔，持续触发指定的回调函数.

#### 不同机制匹配不同场景需求

![](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E4%B8%8D%E5%90%8C%E6%9C%BA%E5%88%B6%E5%8C%B9%E9%85%8D%E4%B8%8D%E5%90%8C%E5%9C%BA%E6%99%AF%E9%9C%80%E6%B1%82-6942d075.png)

| 机制                   | 分类   | 特点                  | 匹配需求           |
| -------------------- | ---- | ------------------- | -------------- |
| 逐 K 线运行（`handlebar`） | 事件驱动 | 同时支持历史回测和盘中可模拟逐K线效果 | 在实盘中模拟逐K线运行的效果 |
| 订阅推送（`subscribe`）    | 事件驱动 | 盘中行情分笔触发函数调用        | 盘中随分笔行情判断交易    |
| 定时运行（`run_time`）     | 定时任务 | 固定间隔触发调用            | 盘中固定时间间隔判断交易   |

### 四、逐 K 线驱动（handlebar）示例

因此，结合不同场景需求（回测或实盘），针对不同的机制（定时任务或事件驱动），我们分别给出回测与实盘的完整示例，复制到策略编辑器中即可使用。

在编写策略前，有以下注意事项：

警告

在编写一个策略时，首先需要在代码的最前一行写上： `#coding:gbk` 统一脚本的编码格式是`GBK`

缩进需要统一 全部统一为`····`或者`->`

#### 回测示例-基于 handlebar

回测的操作流程请参考：[界面操作-策略回测](https://dict.thinktrader.net/innerApi/interface_operation.html#%E7%AD%96%E7%95%A5%E5%9B%9E%E6%B5%8B)

复制代码以下代码到策略编辑器：

python

```
#coding:gbk

#导入常用库
import pandas as pd
import numpy as np
import talib
#示例说明：本策略，通过计算快慢双均线，在金叉时买入，死叉时做卖出 点击回测运行 主图选择要交易的股票品种

def init(C):
    #init handlebar函数的入参是ContextInfo对象 可以缩写为C
    #设置测试标的为主图品种
    C.stock= C.stockcode + '.' +C.market
    #line1和line2分别为两条均线期数
    C.line1=10   #快线参数
    C.line2=20   #慢线参数
    #accountid为测试的ID 回测模式资金账号可以填任意字符串
    C.accountid = "testS"  

def handlebar(C):
    #当前k线日期
    bar_date = timetag_to_datetime(C.get_bar_timetag(C.barpos), '%Y%m%d%H%M%S')
    #回测不需要订阅最新行情使用本地数据速度更快 指定subscribe参数为否. 如果回测多个品种 需要先下载对应周期历史数据 
    local_data = C.get_market_data_ex(['close'], [C.stock], end_time = bar_date, period = C.period, count = max(C.line1, C.line2), subscribe = False)
    close_list = list(local_data[C.stock].iloc[:, 0])
    #将获取的历史数据转换为DataFrame格式方便计算
    #如果目前未持仓，同时快线穿过慢线，则买入8成仓位
    if len(close_list) <1:
        print(bar_date, '行情不足 跳过')
    line1_mean = round(np.mean(close_list[-C.line1:]), 2)
    line2_mean = round(np.mean(close_list[-C.line2:]), 2)
    print(f"{bar_date} 短均线{line1_mean} 长均线{line2_mean}")
    account = get_trade_detail_data('test', 'stock', 'account')
    account = account[0]
    available_cash = int(account.m_dAvailable)
    holdings = get_trade_detail_data('test', 'stock', 'position')
    holdings = {i.m_strInstrumentID + '.' + i.m_strExchangeID : i.m_nVolume for i in holdings}
    holding_vol = holdings[C.stock] if C.stock in holdings else 0
    if holding_vol == 0 and line1_mean > line2_mean:
        vol = int(available_cash / close_list[-1] / 100) * 100
        #下单开仓
        passorder(23, 1101, C.accountid, C.stock, 5, -1, vol, C)
        print(f"{bar_date} 开仓")
        C.draw_text(1, 1, '开')
    #如果目前持仓中，同时快线下穿慢线，则全部平仓
    elif holding_vol > 0 and line1_mean < line2_mean:
        #状态变更为未持仓
        C.holding=False
        #下单平仓
        passorder(24, 1101, C.accountid, C.stock, 5, -1, holding_vol, C)
        print(f"{bar_date} 平仓")
        C.draw_text(1, 1, '平')
```

**基础信息设置** 请参考[基础信息-字段描述](https://dict.thinktrader.net/innerApi/interface_operation.html#%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF-%E5%AD%97%E6%AE%B5%E6%8F%8F%E8%BF%B0)

**回测参数设置** 请参考[回测参数-字段描述](https://dict.thinktrader.net/innerApi/interface_operation.html#%E5%9B%9E%E6%B5%8B%E5%8F%82%E6%95%B0-%E5%AD%97%E6%AE%B5%E6%8F%8F%E8%BF%B0)

#### 实盘示例-基于 handlebar

实盘的操作流程请参考：[界面操作-模型交易](https://dict.thinktrader.net/innerApi/interface_operation.html#%E7%AD%96%E7%95%A5%E8%BF%90%E8%A1%8C)

复制代码以下代码到策略编辑器：

python

```
#coding:gbk

# 导入包
import pandas as pd
import numpy as np
import datetime

"""
示例说明：双均线实盘策略，通过计算快慢双均线，在金叉时买入，死叉时做卖出
"""

class a():
    pass
A = a() #创建空的类的实例 用来保存委托状态 


def init(C):
    A.stock= C.stockcode + '.' + C.market #品种为模型交易界面选择品种
    A.acct= account #账号为模型交易界面选择账号
    A.acct_type= accountType #账号类型为模型交易界面选择账号
    A.amount = 10000 #单笔买入金额 触发买入信号后买入指定金额
    A.line1=17   #快线周期
    A.line2=27   #慢线周期
    A.waiting_list = [] #未查到委托列表 存在未查到委托情况暂停后续报单 防止超单
    A.buy_code = 23 if A.acct_type == 'STOCK' else 33 #买卖代码 区分股票 与 两融账号
    A.sell_code = 24 if A.acct_type == 'STOCK' else 34
    print(f'双均线实盘示例{A.stock} {A.acct} {A.acct_type} 单笔买入金额{A.amount}')

def handlebar(C):
    #跳过历史k线
    if not C.is_last_bar():
        return
    now = datetime.datetime.now()
    now_time = now.strftime('%H%M%S')
    # 跳过非交易时间
    if now_time < '093000' or now_time > "150000":
        return
    account = get_trade_detail_data(A.acct, A.acct_type, 'account')
    if len(account)==0:
        print(f'账号{A.acct} 未登录 请检查')
        return
    account = account[0]
    available_cash = int(account.m_dAvailable)
    #如果有未查到成交 查询成交
    if A.waiting_list:
        found_list = []
        deals = get_trade_detail_data(A.acct, A.acct_type, 'deal')
        for deal in deals:
            if deal.m_strRemark in A.waiting_list:
                found_list.append(deal.m_strRemark)
        A.waiting_list = [i for i in A.waiting_list if i not in found_list]
    if A.waiting_list:
        print(f"当前有未查到委托 {A.waiting_list} 暂停后续报单")
        return
    holdings = get_trade_detail_data(A.acct, A.acct_type, 'position')
    holdings = {i.m_strInstrumentID + '.' + i.m_strExchangeID : i.m_nCanUseVolume for i in holdings}
    #获取行情数据
    data = C.get_market_data_ex(["close"],[A.stock],period = '1d',count = max(A.line1, A.line2)+1)
    close_list = data[A.stock].values
    if len(close_list) < max(A.line1, A.line2)+1:
        print('行情长度不足(新上市或最近有停牌) 跳过运行')
        return
    pre_line1 = np.mean(close_list[-A.line1-1: -1])
    pre_line2 = np.mean(close_list[-A.line2-1: -1])
    current_line1 = np.mean(close_list[-A.line1:])
    current_line2 = np.mean(close_list[-A.line2:])
    #如果快线穿过慢线，则买入委托 当前无持仓 买入
    vol = int(A.amount / close_list[-1] / 100) * 100 #买入数量 向下取整到100的整数倍
    if A.amount < available_cash and vol >= 100 and A.stock not in holdings and pre_line1 < pre_line2 and current_line1 > current_line2:
        #下单开仓 ，参数说明可搜索PY交易函数 passorder
        msg = f"双均线实盘 {A.stock} 上穿均线 买入 {vol}股"
        passorder(A.buy_code, 1101, A.acct, A.stock, 14, -1, vol, '双均线实盘', 2 , msg, C)
        print(msg)
        A.waiting_list.append(msg)
    #如果快线下穿慢线，则卖出委托
    if A.stock in holdings and holdings[A.stock] > 0 and pre_line1 > pre_line2 and current_line1 < current_line2:
        msg = f"双均线实盘 {A.stock} 下穿均线 卖出 {holdings[A.stock]}股"
        passorder(A.sell_code, 1101, A.acct, A.stock, 14, -1, holdings[A.stock], '双均线实盘', 2 , msg, C)
        print(msg)
        A.waiting_list.append(msg)
```

警告

对于立刻下单的模型需要用普通的全局变量来保存状态不能ContextInfo对象存详细说明参考[常见问题：系统对象ContextInfo 逐K线保存的机制](https://dict.thinktrader.net/innerApi/question_answer.html#%E7%B3%BB%E7%BB%9F%E5%AF%B9%E8%B1%A1-contextinfo-%E9%80%90-k-%E7%BA%BF%E4%BF%9D%E5%AD%98%E7%9A%84%E6%9C%BA%E5%88%B6)

更多示例请参见[完整示例](https://dict.thinktrader.net/innerApi/code_examples.html)

### 五、事件驱动（subscribe）示例

#### 实盘示例-基于 subscribe

python

```
#coding:gbk

class a():pass
A = a()
A.bought_list = []

account = 'testaccount'
def init(C):
    #下单函数的参数需要 ContextInfo对象 在init中定义行情回调函数 可以用到init函数的入参 不用手动传入 
    def callback_func(data):
        #print(data)
        for stock in data:
            current_price = data[stock]['close']
            pre_price = data[stock]['preClose']
            ratio = current_price / pre_price - 1
            print(stock, C.get_stock_name(stock), '当前涨幅', ratio)
            if ratio > 0 and stock not in A.bought_list:
                msg = f"当前涨幅 {ratio} 大于0 买入100股"
                print(msg)
                #下单函数passorder 安全起见处于注释状态 需要实际测试下单交易时再放开
                #passorder(23, 1101, account, stock, 5, -1, 100, '订阅下单示例', 2, msg, C)
                A.bought_list.append(stock)
    stock_list = ['600000.SH', '000001.SZ']
    for stock in stock_list:
        C.subscribe_quote(stock, period = '1d', callback = callback_func)
```

### 六、定时任务（run_time）示例

#### 实盘示例-基于 run_time

python

```
#coding:gbk
import time, datetime

class a():
    pass
A = a()

def init(C):
    A.hsa = C.get_stock_list_in_sector('沪深A股')
    A.vol_dict = {}
    for stock in A.hsa:
        A.vol_dict[stock] = C.get_last_volume(stock)
    A.bought_list = []
    C.run_time("f", "1nSecond", "2019-10-14 13:20:00")

def f(C):
    t0 = time.time()
    now = datetime.datetime.now()
    full_tick = C.get_full_tick(A.hsa)
    total_market_value = 0
    total_ratio = 0
    count = 0
    for stock in A.hsa:
        ratio = full_tick[stock]['lastPrice'] / full_tick[stock]['lastClose'] - 1
        if ratio > 0.09 and stock not in A.bought_list:
            msg = f"{now} {stock} {C.get_stock_name(stock)} 当前涨幅 {ratio} 大于5% 买入100股"
            #下单示例 安全起见处于注释状态 需要实际测试下单时可以放开 
            #passorder(23, 1101, account, stock, 5, -1, 100, '示例策略', 2, msg, C)
            A.bought_list.append(stock)
        market_value = full_tick[stock]['lastPrice'] * A.vol_dict[stock]
        total_ratio += ratio * market_value
        total_market_value += market_value
        count += 1
    total_ratio /= total_market_value
    total_ratio *= 100
    print(f'{now} 当前A股加权涨幅 {round(total_ratio, 2)}% 函数运行耗时{round(time.time()- t0, 5)}秒')
```

上次更新: 2025/5/15 11:13:01

邀请注册送VIP优惠券

分享下方的内容给好友、QQ群、微信群,好友注册您即可获得VIP优惠券

玩转qmt,上迅投qmt知识库

## 安装路径的选择

在安装 QMT 软件时，**请不要安装在C盘，以避免因权限问题导致的使用问题**

若是只能安装到C盘，请在启动时选择`以管理员权限启动`

## 下载python库

初次使用 QMT 时，请确保补全所需的 Python 库。安装完毕后，不要忘记重启客户端。

提示

在盘中，下载速度会很慢，建议盘前或盘后更新。

![下载python库](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E4%B8%8B%E8%BD%BDpython%E5%BA%93-521fbb73.png)

## 关于ContextInfo

由于底层机制的限制，`ContextInfo`中存储的变量值将会回滚，即在对`ContextInfo`中的变量进行修改之后，在下一次`handlebar`调用时，这些修改将不会保留。具体细节请参阅[常见问题](https://dict.thinktrader.net/innerApi/question_answer.html#%E7%B3%BB%E7%BB%9F%E5%AF%B9%E8%B1%A1-contextinfo-%E9%80%90-k-%E7%BA%BF%E4%BF%9D%E5%AD%98%E7%9A%84%E6%9C%BA%E5%88%B6)。因此，在完全理解`ContextInfo`机制之前，请避免在其中存储任何变量。

### 推荐用法

```
class G(): pass

g = G()

def init(ContextInfo):
    g.stock_list = ['000001.SZ']

def handlebar(ContextInfo):
    g.stock_list.append('600000.SH')
```

### 错误用法

警告

下面的示例请勿使用

```
def init(ContextInfo):
    ContextInfo.stock_list = ['000001.SZ']

def handlebar(ContextInfo):
    ContextInfo.stock_list.append('600000.SH')
```

## 关于线程和进程

QMT中，python**无法**使用多线程和多进程，而且**所有策略都在同一线程中执行**，所以策略中应该尽量避免阻塞类的写法，否则会影响其他策略的执行。

## **主图**解析

如下图所示，策略执行依赖于K线图。这里所说的主图即是K线图，策略正是在K线图上运行，也是由它驱动的（也有非K线驱动的策略写法，详见快速入门）。

**K线回放**：策略在客户端运行时会从第一根K线开始，依次调用`handlebar`函数，直至最后一根K线。并且在盘中，每一个新的行情快照都会触发一次`handlebar`函数调用（无论主图的周期如何）。如果想要过滤掉某些K线，可以设置右侧的快速计算，或使用`ContextInfo. is_last_bar ()`函数进行过滤。

![Alt text](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_K%E7%BA%BF%E5%9B%9E%E6%94%BE-f0f166ba.png)

## 策略运行无反应/运行报错提示 "run script failed! "

最快解决方法是点击右上角**布局**按钮，选择**恢复默认布局**

如果策略运行后无任何反应，首先检查客户端是否有其他策略正在运行，如果有，请先将其停止，然后重试。检查方法如下图所示：

![Alt text](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%81%9C%E6%AD%A2%E7%AD%96%E7%95%A51-660e5a2b.png)

![Alt text](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%81%9C%E6%AD%A2%E7%AD%96%E7%95%A52-c93e6732.png)

![Alt text](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%81%9C%E6%AD%A2%E7%AD%96%E7%95%A53-2ebf0e22.png)

![Alt text](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%81%9C%E6%AD%A2%E7%AD%96%E7%95%A54-dfca1cc1.png)

提示

最后建议重启客户端

## 数据下载

QMT提供了许多接口来依赖数据下载功能。客户端的数据下载功能如下图所示：

![Alt text](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E6%95%B0%E6%8D%AE%E4%B8%8B%E8%BD%BD1-bd7db774.png)而且，在批量下载中可以设置定时下载，这样可以方便地每天自动下载当日的行情数据。![Alt text](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%AE%9A%E6%97%B6%E4%B8%8B%E8%BD%BD-034ff844.png)

上次更新: 2024/3/21 10:12:58

邀请注册送VIP优惠券

分享下方的内容给好友、QQ群、微信群,好友注册您即可获得VIP优惠券

玩转qmt,上迅投qmt知识库

 [快速开始](https://dict.thinktrader.net/innerApi/start_now.html)

## 新建策略

模型创建方法有三种：

方法一，在【模型研究】界面，使用系统预置的各种示例模型，点击后方“编辑”按钮，并在弹出的【策略编辑器】中以此示例模型代码为基础进行编写。

![](https://dict.thinktrader.net/assets/%E6%96%B0%E5%BB%BA%E7%AD%96%E7%95%A5_%E6%96%B9%E6%B3%95%E4%B8%80-0bcda3ed.png)

**我的主页-编辑模型**

方法二，在【模型研究】界面，点击新建模型，选择 Python 模型，在弹出的【策略编辑器】中从头到尾编写一个用户自己的量化模型。

![](https://dict.thinktrader.net/assets/%E6%96%B0%E5%BB%BA%E7%AD%96%E7%95%A5_%E6%96%B9%E6%B3%95%E4%BA%8C-82d129f1.png)

**模型研究-新建模型**

方法三，在模型管理面板右键，选择新建模型，并选择 Python 模型。

![](https://dict.thinktrader.net/assets/%E6%96%B0%E5%BB%BA%E7%AD%96%E7%95%A5_%E6%96%B9%E6%B3%95%E4%B8%89-f0c1eb6b.png)

**模型管理-新建模型**

## 导入导出策略

QMT系统支持将策略以**加密**的模式进行导出或导入，用户可以便捷的迁移系统本地策略。

![img](https://dict.thinktrader.net/assets/Snipaste_2022-08-10_18-32-20-fd9d8884.png)

方式一.模型研究界面导入导出

![img](https://dict.thinktrader.net/assets/Snipaste_2022-08-10_18-43-19-64ce2ce5.png)

方式二.模型研究界面导入导出

## 策略编写

【策略编辑器】是迅投专门为模型开发者设计的，集成了模型列表、函数列表、函数帮助、模型基本信息、参数设置、回测参数等多个部分，拥有代码高亮、自动补全等便捷功能于一体的便捷的模型编辑、开发环境。

![](https://dict.thinktrader.net/assets/image12-caa10435.png)

**模型编辑页面 右侧可选择策略默认的周期、品种**

编写 Python 策略需在开始时定义编码格式，如 `gbk`。

之后可选择导入第三方库，所选第三方库要在券商管理端白名单内才可运行。

Init 方法和 handlebar 方法的定义是必须的。Init 方法会在策略运行开始时调用一次，用以初始化所需对象（包裹在 ContextInfo 对象中传递），设定股票池等。

![](https://dict.thinktrader.net/assets/image13-1a01b323.png)

**Handlebar 方法会在历史 K 线上逐 K 线调用，系统会保存函数所做更改。**

在盘中交易时间，handlebar 函数会随行情推送（tick 数据）被调用，当一个 tick 数据为所在 K 线最后一个 tick 时，此 tick 调用的 handlebar 所做的更改会被系统保存，如有交易指令，会在下一根K 线的第一个 tick 到来时发送；其他 tick 可以打印运行结果，但 handlebar 所做更改不会被保存，也不会发送交易信号。

编写创建完模型后，对应模型的基本信息和回测参数进行设置。

### 基本信息-字段描述

| 字段          | 描述                                                                          |
| ----------- | --------------------------------------------------------------------------- |
| **名称**      | 填写模型名称                                                                      |
| **快捷码**     | 默认根据模型名称自动生成拼音首字母拼写，如需自定义可以手动进行更改，用于键盘精灵快速引用模型                              |
| **说明**      | 简单的说明模型功能                                                                   |
| **分类**      | 保存当前模型到某个分类下面                                                               |
| **位置**      | 模型回测或运行时的位置，有副图、主图叠加、主图三种显示位置                                               |
| **默认周期**    | 点击模型回测或运行时的默认主图周期，可手动切换                                                     |
| **默认品种**    | 点击模型回测或运行时的默认主图品种，可手动切换                                                     |
| **复权方式**    | 提供不复权、前复权、后复权、等比前复权、等比后复权 5 种复权方式                                           |
| **快速计算**    | 限制计算范围，默认为 0 时模型运行会从模型设置的默认品种（主图）的第一根 K 线开始计算，设置为 n 则从当前 K 线再往前 n 个 K 线开始计算 |
| **刷新间隔**    | 用来设置策略运行的时间间隔。设置了刷新间隔，即每隔一段时间策略按照当前行情运行一次                                   |
| **加密公式**    | 加密后的公式只有输入密码才可以查看源代码                                                        |
| **凭密码导出公式** | 此项只有在开启 “加密公式” 后才能生效，生效后只能使用密码导出到本地                                         |
| **用法注释**    | 简短的说明模型使用的一些注意项，可不填                                                         |

![](https://dict.thinktrader.net/assets/image14-b74f4979.png)

**策略编辑器-基本信息**

回测模式指策略以历史行情为依据进行运算，投资者可观察该策略在历史行情所获得的年化收益率、夏普比率、最大回撤、信息比率等指标表现。

### 回测参数-字段描述

| 字段               | 描述                                            |
| ---------------- | --------------------------------------------- |
| **开始时间<br>结束时间** | 设置模型回测时间区间                                    |
| **基准**           | 设置模型收益的参考基准                                   |
| **初始资金**         | 设置模型回测的初始资金                                   |
| **保证金比例**        | 设置期货的保证金比例                                    |
| **滑点**           | 设置回测撮合时的滑点，模拟真实交易的冲击成本                        |
| **手续费类型**        | 支持按成交额比例或者固定值计算手续费                            |
| **买入印花税**        | 设置买入印花税比例                                     |
| **卖出印花税**        | 设置卖出印花税比例                                     |
| **最低佣金**         | 设置单笔交易的最低佣金数额                                 |
| **买入佣金**         | 设置买入标的时的佣金比例                                  |
| **平昨佣金**         | 设置股票、期货平昨佣金比例                                 |
| **平今佣金**         | 设置期货平今佣金比例                                    |
| **最大成交比例**       | 控制回测中最大成交量不超过同期成交量*最大成交比例。可以点击此参数旁边的'？'按钮了解详情 |

![](https://dict.thinktrader.net/assets/image15-19860147.png)

**策略编辑器-回测参数**

使用者也可在参数设置中设置好参数值，参数名为变量名，模型中可以调用。最新值为变量默认值，运行/回测模式使用。

其中最小 / 最大 / 步长项，为遍历参数。初始项可不填。最小 / 最大都是包含在遍历区间内的，如图所示三个变量，测评时会遍历（20-3+1）*[（160-140）/10+1]*[（-150+250）/10+1] 种组合。

![](https://dict.thinktrader.net/assets/image16-304d077f.png)

*策略编辑器-参数设置*

点击公式测评，可选择回测模式支持的指标，如单位净值，最大回撤等，作为评价标准。

![](https://dict.thinktrader.net/assets/image17-caff9a62.png)

*策略编辑器-公式评测*

点击优化，评测结果弹窗显示不同参数变量组合下的回测结果，根据结果选择最优参数组合。可点击所需指标进行排序。需要注意的是，在测评之前，需要针对所选品种和周期补充数据。

![](https://dict.thinktrader.net/assets/image18-6b7b0244.png)

## 补充数据

在创建用户的模型之前，用户应使用客户端提供的“数据管理”功能，选择并补充模型所需的相应市场、品种以及对应周期的历史数据。

![](https://dict.thinktrader.net/assets/image19-c8a58204.png)

*操作-数据管理*

## 策略运行

策略编写完毕后，点击编译，可保存策略。编译按钮在 Python 策略中只起保存功能，不会检查语法与引用的正误。之后点击运行可以看到策略运行效果（如有错误，会在日志输出的位置报错）。

![](https://dict.thinktrader.net/assets/image20-d65db891.png)

*策略编辑器-运行*

如当前系统所处界面为“行情”界面或“交易”界面，点击运行之前，需在行情中手动设置好 K 线品种和周期，点击运行后，策略即可在当前主图下运行，如下图所示。

![](https://dict.thinktrader.net/assets/image21-b4348bce.png)

*策略运行状态之一*

如系统当前界面处于“我的主页”界面或“模型研究”和“模型交易”等非行情界面，点击运行时，会基于策略编辑器 - 基本信息中所设置的默认周期和默认品种运行。

![](https://dict.thinktrader.net/assets/image22-dd4db57c.png)

*策略运行状态之二*

当选择的运行位置为副图时，![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAPMAAAAfCAIAAAC9EmyRAAAAA3NCSVQICAjb4U/gAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAFBUlEQVR4nO2b709bVRjHn97ROkA7aMcItC+ktw5EzXQzi1mNLzpDBRLYRoyLb0z2wkViREzGf+C7JaRbiO4FJjNLNBJxIQqCo5uJxBdToo5ug9GL22xHGS2XZrZdC72+aLy5uT/OOf3lxZvzedXec+59vs8533t+FUy2fW6gUAwHo7cACqUiUGdTjAl1NsWYUGdTjAl1NsWYUGdTjAl1NsWYUGdTjEkVSaWpmYnOjh7l57IzNTOhVUQelGHM9tY+W4uvus4NACl+Ob4yHVv8OpfLkgeV5YsQhpBXZbG6fReq61wm0y4AEITtFM8tT5/eyiSwWWi1M6ESrWqyZ5bemzrmiBGGraF8aEXNLXaMNBA2VRFzTQPr9WeS0fD8SHI9CAB7nJ6mA+/a2Z5QYCCbfKgVkVAYqPWHqjyrw8N6hwVha31pfC14CQCch8/scXpeeOtKKDCYCM9hw2m1M4kS5e2V6DV9c0SDd7by0ZWzNUh05z+QexoAGMbMev383dnVG6PixY0/r2z+9RPrPccePbf03Tu5XAYRNI/07VKtKTWQqpIqi5X1DmceRYLfHBMvhgIfPvFkU/uxy6x3eGHMl01voNNBtzNWiWyqIem1gurvhBxR8gjrEc5uJVLiGGNv7csmo6s3RhnG7Dg0UM92gQAb3GT4V//Kj2fae8fsrSce3voSHRrxLuWLSGq6Oz4VhC1pl+d5/OjB4tSptu6L7Osjt799WysEWiGJEplHp2YmZP6QjSDidfJJUvcc0WCcXdzrUhzo7iHRYGvxhedHAKD55YGGtpP5iw3PntzefhyZPx9d+NzW4tNyNknDyXodoae6nl1fGlctSsaC6U2uup5FhBAlaYUgUaJaR/m10PWeyE7IEQHG2dIZrdLrttI3N9W2tmTsJgDYXF3S63v3H4/Mn+fvXW062E8SWhVlEyPcYDLtWrv1hVbp6u+jT7/2MTZiKUrEjhNHYtkQWDq654im4B1kpZEmUMz7KuS0SrLJNchtoyOKqC5bpfU7O3owja5xFAMAicjPWkUkkCiRDsayvXg5xyNdc0RTgLMrfSoiDST9Sv6ypuK3rQ4Pf3c2zk3u+3c1AgCxpcsAYK5pSPHLiIiEUzOJRQRh23F4iAsMqpY+1fyKIKi8Y8rQ6C2dlhLZxkv5tSydqG+OWIocs/8bixdKfGXa8WJ/IjwX+cVvEqDe1QUAG9zkg98+AQCbqzu+Mq16o3I7pYVyCFElxXN1zlctNY2ZZFRZ2nzw/RTPqd5Ivt5DK1EdhgjP5gnRPUc0O241IlLEyxNbHLe7e91H/dy1ofvXz96/flYsqm04YHUeufP9adUbVcds7LYMIS/0Q//zb04/d2JicepUMhaUFjkODVhqGxfG3igotYKUoCduxFFxQeibI5bCnF2RtVr5yOUyodkPWK+/vferaPASf+9q9u+oubbR1tJpdR7hrg2pHmYD2ZiNPbqRkk1vhAKDrHe4rftienNl9Y/PEuE5q8PT9NJ7ltrGUOAj7EEvAnIlRazryEf0HZKjFibs/0Gip7AyWhwrnTAWw5j37u+zubp217kAIM1zcW5yfQn167rWsYw4WmAnTdXSKov1Gd+F3XWsycQAgCDk0nzojvYvzyQtQKKkoOdIR8QiNpp65YgVhnc2hfJ/hP6tH8WYUGdTjAl1NsWYUGdTjAl1NsWYUGdTjAl1NsWYUGdTjAl1NsWY/APcf6iqAW8OywAAAABJRU5ErkJggg==)如想关闭策略，将主图下方策略运行的附图关闭即可。

![](https://dict.thinktrader.net/assets/image24-0b12386e.png)

*关闭运行中的策略之一*

![](https://dict.thinktrader.net/assets/image25-43ffcc17.png)

*关闭运行中的策略之二*

当选择的运行位置为主图叠加时，如想关闭策略，在主图上右键单击取消叠加指标即可。

当选择的运行位置为主图时，键盘精灵输入KLINE即可结束模型运行。

*关闭运行中的策略之三*

点击策略编辑器上放的停止按钮![](https://dict.thinktrader.net/assets/stop-9d614ae0.png)

## 策略调试

如果策略运行不成功，需要进行策略调试这一步。当运行出错时，报错信息会显示在日志输出面板，以供修改调试之用。

![](https://dict.thinktrader.net/assets/image26-32b1efd6.png)

*策略调试-输出日志*

## 独立python进程

注意

在您充分理解软件使用前，不建议开启该功能

勾选此功能后，程序将把代码作为main执行脚本，**不会触发init,handlebar等函数**

反之，系统会import策略，按规则触发init handlebar等函数

![](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E7%8B%AC%E7%AB%8Bpython-be2b2739.png)

## 策略回测

对某一策略编译成功后，点击回测，可以通过日志输出查看模型基于历史行情数据回测情况和表现。

在回测之前，需要设置好策略回测运行的主图品种和周期，以及相关的回测参数。回测主图和周期可以在策略编辑器-基本信息中进行设置，回测开始和结束时间、基准、费率等可以在策略编辑器 - 回测参数中设置。

用户在回测前，需根据此策略回测运行的主图、周期和时间，在【数据管理】中对行情数据进行下载补充。如回测时间设置为 20180930 至 20190530 ，运行主图为 SZ.000001 平安银行日线，补充数据时可进行如下设置。

![](https://dict.thinktrader.net/assets/image27-39de4e9a.png)

*操作-数据管理*

如果回测正常的话，主界面会跳转到模型设置的默认标的和默认周期界面，并输出模型绩效分析结果。

![](https://dict.thinktrader.net/assets/image28-ded030fa.png)

*策略编辑器-模型回测*

此时，可最小化或者关闭【策略编辑器】，并对回测结果进行分析，随着光标在 K 线主图上的移动，右边回测结果展示窗口会动态显示截止光标所在当日的绩效分析结果（包括年化收益，基准年化收益，单位净值，下方差，信息比率，夏普比率，波动率，索提诺比率，阿尔法系数，贝塔系数，跟踪误差，最大回撤，胜率等）、当日买入、当日卖出、持仓列表。

以上列表均可鼠标右键复制和导出数据。

![](https://dict.thinktrader.net/assets/image29-c5ed0bcd.png)

*回测结果分析-回测结果随十字光标移动动态展示*

如需根据模型生成的买入、卖出列表进行手工交易，则可直接点击“买入'或“卖出'按钮，系统会弹出下单界面由用户进行确认后进行普通交易下单或算法交易下单。交易方式可点击卖出按钮右侧的普通交易进行切换设置。

![](https://dict.thinktrader.net/assets/image30-42b93a46.png)

*回测结果买入*

副图回测指标：提供图形化的展示，除去绩效分析的相关指标外，用户可以通过编辑模型代码自定义输出一些特色指标，鼠标右键可以选择复制模型运行结果（每一天的数据）。

![](https://dict.thinktrader.net/assets/image31-c99b7e64.png)

*回测结果分析-附图指标输出*

另外，回测结果还提供了持仓分析、历史板块汇总、操作明细、日志输出等信息，方便用户进行深入分析。

说明

**持仓分析：** 可查看光标所在当天持仓的行业分布，展示在相关行业的市值情况、盈利情况、权重以及股票数量情况，鼠标右键可以复制和导出数据；可切换对比基准，和模型持仓进行对比。

**历史板块汇总：** 可查看模型自回测日期以来到光标所在日期该模型交易标的的汇总信息，包括累计盈亏、累计交易量、累计交易额、持仓天数等；点击选择板块，可以自行选择其常用板块进行板块的各项数据累计汇总；汇总数据均可以进行排序，鼠标右键可以复制和导出数据。

**操作明细：** 可查看模型回测的历史每一笔交易的明细。

**日志输出：** 可用于调试输出模型回测和运行情况。

![](https://dict.thinktrader.net/assets/image32-bd498984.png)

*回测结果分析-绩效分析、当日买入、当日卖出、当日持仓*

![](https://dict.thinktrader.net/assets/image33-b8a3fd0c.png)

*回测结果分析-持仓分析*

![](https://dict.thinktrader.net/assets/image34-4cb93cc9.png)

*回测结果分析-历史品种汇总、历史板块汇总*

![](https://dict.thinktrader.net/assets/image35-bdafc2cc.png)

*回测结果分析-操作明细、日志输出*

## 回测、运行两种模式的区别

在模型编辑器中，有“回测”和“运行”两个按钮，分别代表两种模式，它们之间的区别如下：

（1）回测模式指策略以历史行情为依据，以回测参数中的开始时间、结束时间为回测时间区间进行运算，投资者可观察该策略在历史行情所获得的年化收益率、夏普比率、最大回撤、信息比率等指标表现。

（2）运行模式指策略根据实时行情信号进行运算，以主图行情开始时间到当前时间为运行区间，进行策略的模拟运行，但不进行真实的委托。

注意

如果需要向模拟/实盘柜台发送真实的委托，请将策略加入到“模型交易”中。

## 配置/获取模拟账号

注意

1. **券商QMT的模拟账号通常联系对应券商解决相关问题**
2. 以下均为迅投模拟账号相关指南
3. 迅投模拟账号的格式如下
   1. 股票账号：200xxxx
   2. 期货账号：100xxxx
   3. 期权账号：600xxxx

提示

1. 新用户注册投研账号，可获得 14 天模拟仿真交易体验

2. **VIP权限用户**可以通过`用户中心 - 下载中心`的客户端进行模拟交易，交易支持**股票/期货/股票期权**三个市场

3. 在[投研服务平台](https://xuntou.net/#/home)登录您的投研账号，如您没有投研账号，请先[注册](https://xuntou.net/#/signup)

4. 点击右上角的**用户中心**，选择下方栏目的**模拟撮合**按钮，可以看到您的**投研模拟账号**

![模拟账号](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E6%A8%A1%E6%8B%9F%E8%B4%A6%E5%8F%B7-e6b27475.png)

3. **VIP权限用户**可以通过`用户中心 - 下载中心`的客户端进行模拟交易，交易支持**股票/期货/股票期权**三个市场

![模拟交易环境说明](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E6%A8%A1%E6%8B%9F%E7%8E%AF%E5%A2%83%E6%94%AF%E6%8C%81-5964479a.png)

4. 客户端配置账号

![](https://dict.thinktrader.net/assets/%E4%BA%A4%E6%98%93%E8%B4%A6%E5%8F%B7%E7%99%BB%E5%BD%95-f61933d6.png)

## 自动导出交易记录

![](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E8%87%AA%E5%8A%A8%E5%AF%BC%E5%87%BA%E4%BA%A4%E6%98%93%E8%AE%B0%E5%BD%95-1e572560.png)

## 操作界面快捷键

| 操作                | 描述                    |
| ----------------- | --------------------- |
| `SHIFT`+`Q`       | 分时 K线 附图变量查看器         |
| `SHIFT`+`G`       | k线 附图组合模型持仓界面         |
| `SHIFT`+`L`       | k线 锁定十字线              |
| `SHIFT`+`S`       | 星空图                   |
| `CTRL`+`Windows键` | 快速唤起多屏                |
| `CTRL`+`O`        | 分时 K线 叠加品种            |
| `CTRL`+`Z`        | 分时 K线 添加自选            |
| `CTRL`+`M`        | 跳转到多股同列界面             |
| `CTRL`+`X`        | 跳转到多周期同列界面            |
| `CTRL`+`←`        | K线 多股同列 向左快捷移动十字线     |
| `CTRL`+`→`        | K线 多股同列 向右快捷移动十字线     |
| `CTRL`+`R`        | k线 移动十字线到当前界面结尾       |
| `CTRL`+`V`        | k线 切换复权方式为等比前复权/不复权   |
| `CTRL`+`B`        | k线 切换复权方式为等比后复权/不复权   |
| `CTRL`+`A`        | 分时 k线 浏览列表 样板股分析      |
| `CTRL`+`E`        | 分时 k线 浏览列表 预警雷达       |
| `ALT`+`数字 (1~9)`  | 分时 K线 设置附图指标数量        |
| `ALT`+`←`         | 分时 K线 显示走势图           |
| `ALT`+`→`         | 分时 K线 显示走势图           |
| `F3`              | 切换到上证指数分时图            |
| `F4`              | 切换到深证成指分时图            |
| `F5`              | 切换到分时/K线图             |
| `F6`              | 切换到“我的自选”板块浏览列表       |
| `F8`              | K线 循环切换周期             |
| `F10`             | 个股切换财务数据界面，指数切换到成分股界面 |

## 策略编辑器快捷键

| 操作         | 描述       |
| ---------- | -------- |
| `Ctrl`+`C` | 复制       |
| `Ctrl`+`X` | 剪切       |
| `Ctrl`+`V` | 粘贴       |
| `Ctrl`+`Q` | 多行注释     |
| `Ctrl`+`Z` | 撤消       |
| `Ctrl`+`Y` | 恢复       |
| `Ctrl`+`A` | 全选       |
| `Ctrl`+`F` | 键查找对话框启动 |
| `Ctrl`+`D` | 复制并粘贴当行  |
| `Ctrl`+`L` | 删除当前行    |
| `Ctrl`+`T` | 当行向上移动一行 |
| `Ctrl`+`S` | 保存文件     |

## 函数命名规则

- 函数名以 `get_` 开头的，表示数据来源于客户端内存
- 函数名以 `query_` 开头的，表示数据是向服务查询

## 账号类型说明

- 'FUTURE' - 期货账号
- 'STOCK' - 股票账号
- 'CREDIT' - 信用账号
- 'FUTURE_OPTION' - 期货期权
- 'STOCK_OPTION' - 股票期权
- 'HUGANGTONG' - 沪港通
- 'SHENGANGTONG' - 深港通

## symbol_code - 代码表示

迅投代码(symbol_code)是迅投平台统一用于表示交易标的的代码 其格式为:**交易标的代码.交易所代码**,例如深圳证券交易所的平安银行,迅投代码为`000001.SZ`(不区分大小写)。代码表示可以在迅投研终端的行情列表或者按键精灵中查询。

![迅投研终端示例](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E4%BA%A4%E6%98%93%E4%BB%A3%E7%A0%81%E6%9F%A5%E8%AF%A2%E7%95%8C%E9%9D%A2%E7%A4%BA%E4%BE%8B-e39483ea.png)

### 交易所代码

目前迅投研支持国内12个交易所,12个交易所的代码缩写如下:

| 交易所名称      | 迅投简称 | 显示后缀  |
| ---------- | ---- | ----- |
| 上海证券交易所    | SH   | SH    |
| 深圳证券交易所    | SZ   | SZ    |
| 北京证券交易所    | BJ   | BJ    |
| 香港证券交易所    | HK   | HK    |
| 沪港通        | HGT  | HGT   |
| 深港通        | SGT  | SGT   |
| 中国金融期货交易所  | IF   | CFFEX |
| 上海期货交易所    | SF   | SHFE  |
| 大连商品交易所    | DF   | DCE   |
| 郑州商品交易所    | ZF   | CZCE  |
| 上海国际能源交易中心 | INE  | INE   |
| 广州期货交易所    | GF   | GFEX  |

迅投研系统目前支持一站式获取全球多市场数据，详情链接：[全球市场数据](https://xuntou.net/#/shoppingMall?goback=1&id=7zqjlm)

![全球行情展示](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%85%A8%E7%90%83%E5%B8%82%E5%9C%BA%E6%95%B0%E6%8D%AE-941c5b28.png)

### 交易标的代码

交易标的代码是指交易所给出的交易标的代码, 包括股票（如 600000）, 期货（如 rb2011）, 期权（如 10002498）, 指数（如 000001）, 基金（如 510300）等代码。

注意

对于期货合约代码来说，我们仅对`market`做了简化处理，`symbol`仍遵守交易所标准命名规则，且**严格区分大小写**，例如`AP401.ZF`不能写成`ap401.ZF`,`rb2401.SF`不能写成`RB2401.SF`

#### symbol示例

| 市场中文名      | 市场代码 | 示例代码         | 显示后缀  | 证券简称                      |
| ---------- | ---- | ------------ | ----- | ------------------------- |
| 上交所        | SH   | 600000.SH    | SH    | 浦发银行                      |
| 深交所        | SZ   | 000001.SZ    | SZ    | 平安银行                      |
| 北交所        | BJ   | 830779.BJ    | BJ    | 武汉蓝电                      |
| 中金所        | IF   | IC2311.IF    | CFFEX | 中证 500 指数 2023 年 11 月期货合约 |
| 上期所        | SF   | rb2311.SF    | SHFE  | 螺纹钢 2023 年 11 月期货合约       |
| 大商所        | DF   | m2311.DF     | DCE   | 豆粕 2023 年 11 月期货合约        |
| 郑商所        | ZF   | FG305.ZF     | CZCE  | 玻璃 2023 年 5 月期货合约         |
| 上海国际能源交易中心 | INE  | sc2311.INE   | INE   | 原油 2023 年 11 月期货合约        |
| 广期所        | GF   | lc2405.GF    | GFEX  | 碳酸锂 2024 年 05 月期货合约       |
| 上证期权       | SHO  | 10005334.SHO | SH    | 50ETF购12月2650             |
| 深证期权       | SZO  | 90002114.SZO | SZ    | 深证100ETF沽12月2700          |
| 板块指数       | BKZS | 290001.BKZS  | BKZS  | 工业品期货板块指数                 |

### 期货主力连续合约

仅支持回测模式下交易，`期货主力连续合约`为量价数据的简单拼接，未做平滑处理，如`rb00.SF`螺纹钢主连合约，其他[主连合约代码请参考]([期货数据 | 迅投知识库 (thinktrader.net)](https://dict.thinktrader.net/dictionary/future.html#%E4%B8%BB%E5%8A%9B%E8%BF%9E%E7%BB%AD%E5%90%88%E7%BA%A6%E5%8F%8A%E5%8A%A0%E6%9D%83))

### 期货加权连续合约

仅支持回测模式下交易，`期货加权连续合约`为迅投按照一定规则加权合成的连续合约，相比主力连续合约更加平滑,如`rbJQ00.SF`,其他[加权合约代码参考]([期货数据 | 迅投知识库 (thinktrader.net)](https://dict.thinktrader.net/dictionary/future.html#%E4%B8%AD%E9%87%91%E6%89%80))

## mode - 模式选择

迅投研终端中，策略可以以四种模式运行，分别为`调试运行模式`，`回测模式`,`模拟信号模式`,`实盘交易模式`，模式需要在运行策略时手动选择

### 调试运行模式

调试运行模式需要在策略编辑界面点击编辑栏上方的`运行`，该模式下策略会以实时行情进行运算，但迅投研终端`不会记录交易信号`![迅投研终端示例](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F-5e259f69.png)

### 回测模式

回测模式需要在策略编辑界面点击编辑栏上方的`回测`，该模式下策略会以右侧栏设定的回测周期推进行情进行运算，回测模式下，发生的交易会被记录在回测结果页面![迅投研终端示例](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%9B%9E%E6%B5%8B%E6%A8%A1%E5%BC%8F-922c1015.png)

### 模拟信号模式

模拟信号模式需要在策略交易界面，在左侧策略文件栏中选择要进行计算运行的策略，点击`右侧圆形按钮`选择**模拟**，点击`三角形运行按钮`后策略会以实时行情进行运算，该模式下调用的下单函数(passorder)**不会产生实际交易，仅会记录交易信号在下方的`策略信号`栏中**![迅投研终端示例](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E6%A8%A1%E6%8B%9F%E4%BF%A1%E5%8F%B7%E6%A8%A1%E5%BC%8F-f1af1ef8.png)

### 实盘交易模式

实盘交易模式需要在策略交易界面，在左侧策略文件栏中选择要进行计算运行的策略，点击`右侧圆形按钮`选择**实盘**，点击`三角形运行按钮`后策略会以实时行情进行运算，该模式下调用的下单函数(passorder)**会对账户实际下单，同时交易信号会记录在下方的`策略信号`栏中**![迅投研终端示例](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E5%AE%9E%E7%9B%98%E6%A8%A1%E5%BC%8F-81101369.png)

## ContextInfo - 上下文对象

### ContextInfo.start/ContextInfo.end - 回测开始/结束时间

注意

一、此属性只在回测模式生效；

二、仅在init中设置生效，应在init中设置完毕；

三、缺省值为策略编辑界面设定的回测时间范围；

四、回测起止时间也可在策略编辑器的回测参数面板中设置，若两处同时设置，则以代码中设置的值为准；

五、结束时间小于等于开始时间则计算范围为空。

**释义**

可通过此属性设定回测开始/结束的时间,以`%Y-%m-%d %H:%M:%S`格式传入

**原型**

内置python

```
ContextInfo.start # 回测开始时间属性
ContextInfo.end # 回测结束时间属性
```

**返回值**`none`

**示例**

内置python输出值

```
# coding:gbk
def init(ContextInfo):
    ContextInfo.start = "2017-01-01 00:00:00"# 回测开始时间为 2017-01-01
    ContextInfo.end = "2020-01-01 00:00:00"# 回测结束时间为 2020-01-01
def handlebar(ContextInfo):
    # 打印输出当前回测时间
    print(timetag_to_datetime(ContextInfo.get_bar_timetag(ContextInfo.barpos), "%Y-%m-%d %H%M%S"))
```

### ContextInfo.capital - 设定回测初始资金

注意

此函数只支持回测模式。回测初始资金也可在策略编辑器的回测参数面板中设置，若两处同时设置，则以代码中设置的值为准。

**释义** 设定回测初始资金，**支持读写**，默认为 1000000

**原型**

内置python

```
ContextInfo.capital = 10000000 # 设定ContextInfo.capital 值为10000000
```

**返回值**`float`类型的数值，代表当前策略设定的回测金额

**示例**

内置python输出值

```
# coding:gbk
def init(ContextInfo):
    ContextInfo.capital = 10000000
def handlebar(ContextInfo):
    print(ContextInfo.capital)
```

### ContextInfo.period - 获取当前周期

**释义** 获取当前周期，即基本信息中设置的默认周期，**只读**

**原型**

内置python

```
ContextInfo.period
```

**返回**`string`,返回值含义:

| 值      | 含义    |
| ------ | ----- |
| '1d'   | 日线    |
| '1m'   | 1分钟线  |
| '3m'   | 3分钟线  |
| '5m'   | 5分钟线  |
| '15m'  | 15分钟线 |
| '30m'  | 30分钟线 |
| '1h'   | 小时线   |
| '1w'   | 周线    |
| '1mon' | 月线    |
| '1q'   | 季线    |
| '1hy'  | 半年线   |
| '1y'   | 年线    |

**示例**

内置python返回值

```
# coding:gbk
def init(ContextInfo):
    pass
def handlebar(ContextInfo):
    print(ContextInfo.period)
```

### ContextInfo.barpos - 获取当前运行到 K 线索引号

**释义**

获取主图当前运行到的 K 线索引号，只读，索引号从0开始

**原型**

内置python

```
ContextInfo.barpos
```

**返回值**`int`类型值,代表着当前K线的索引号

**示例**

内置python返回值

```
# coding:gbk
def init(ContextInfo):
    pass
def handlebar(ContextInfo):
    print(ContextInfo.barpos)
```

### ContextInfo.time_tick_size - 获取当前图 K 线数目

**释义**

获取当前图 K 线bar的数量，只读

**原型**

内置python

```
ContextInfo.time_tick_size
```

**返回值**`int`

**示例**

内置python返回值

```
# coding:gbk
def init(ContextInfo):
    pass
def handlebar(ContextInfo):
    print(ContextInfo.time_tick_size)
```

### ContextInfo.stockcode - 获取当前图代码

**释义**

获取当前主图代码，只读

**原型**

内置python

```
ContextInfo.stockcode
```

**返回值**`string`：对应主图代码

**示例**

内置python返回值

```
# coding:gbk
def init(ContextInfo):
    pass
def handlebar(ContextInfo):
    print(ContextInfo.stockcode)
```

### ContextInfo.market - 获取当前主图市场

**释义**

获取当前主图市场，只读

**原型**

内置python

```
ContextInfo.market
```

**返回值**`string`：对应主图市场

**示例**

内置python返回值

```
# coding:gbk
def init(ContextInfo):
    pass
def handlebar(ContextInfo):
    print(ContextInfo.market)
```

### ContextInfo.dividend_type - 获取当前主图复权处理方式

**释义**

获取当前主图复权处理方式

**原型**

内置python

```
ContextInfo.dividend_type
```

**返回值**`string`，返回值含义：

| 值             | 含义     |
| ------------- | ------ |
| 'none'        | 不复权    |
| 'front'       | 向前复权   |
| 'back'        | 向后复权   |
| 'front_ratio' | 等比向前复权 |
| 'back_ratio'  | 等比向后复权 |

**示例**

内置python返回值

```
# coding:gbk
def init(ContextInfo):
    pass
def handlebar(ContextInfo):
    print(ContextInfo.dividend_type)
```

### ContextInfo.benchmark - 获取回测基准标的

注意

该属性只在回测模式可用

**释义** 获取回测基准的代码，只读

**原型**

内置python

```
ContextInfo.benchmark
```

**返回值**`string`

**示例**

内置python返回值

```
# coding:gbk
def init(ContextInfo):
    pass
def handlebar(ContextInfo):
    print(ContextInfo.benchmark)
```

### ContextInfo.do_back_test - 表示当前是否为回测模式

**释义**

表示当前是否为回测模式，只读，默认值为 False

**原型**

内置python

```
ContextInfo.do_back_test
```

**返回值**`bool`

上次更新: 2024/11/12 17:52:57

邀请注册送VIP优惠券

分享下方的内容给好友、QQ群、微信群,好友注册您即可获得VIP优惠券

玩转qmt,上迅投qmt知识库

 [界面操作](https://dict.thinktrader.net/innerApi/interface_operation.html)

## 数据类

### Tick - Tick 对象

行情快照数据

#### get_market_data_ex/get_full_tick返回对象：

| 字段名                   | 数据类型          | 含义                                                                                                                                                   |
| --------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `time`                | `int`         | `时间戳`                                                                                                                                                |
| `stime`               | `string`      | `时间戳字符串形式`                                                                                                                                           |
| `lastPrice`           | `float`       | `最新价`                                                                                                                                                |
| `open`                | `float`       | `开盘价`                                                                                                                                                |
| `high`                | `float`       | `最高价`                                                                                                                                                |
| `low`                 | `float`       | `最低价`                                                                                                                                                |
| `lastClose`           | `float`       | `前收盘价`                                                                                                                                               |
| `amount`              | `float`       | `成交总额`                                                                                                                                               |
| `volume`              | `int`         | `成交总量（手）`                                                                                                                                            |
| `pvolume`             | `int`         | `原始成交总量(未经过股手转换的成交总量)【不推荐使用】`                                                                                                                        |
| `stockStatus`         | `int`         | `证券状态`                                                                                                                                               |
| `openInt`             | `int`         | `若是股票，则openInt含义为股票状态，非股票则是持仓量`[openInt字段说明](https://dict.thinktrader.net/innerApi/data_structure.html#openint-%E8%AF%81%E5%88%B8%E7%8A%B6%E6%80%81) |
| `transactionNum`      | `float`       | `成交笔数(期货没有，单独计算)`                                                                                                                                    |
| `lastSettlementPrice` | `float`       | `前结算(股票为0)`                                                                                                                                          |
| `settlementPrice`     | `float`       | `今结算(股票为0)`                                                                                                                                          |
| `askPrice`            | `list[float]` | `多档委卖价`                                                                                                                                              |
| `askVol`              | `list[int]`   | `多档委卖量`                                                                                                                                              |
| `bidPrice`            | `list[float]` | `多档委买价`                                                                                                                                              |
| `bidVol`              | `list[int]`   | `多档委买量`                                                                                                                                              |

#### get_market_data返回对象：

| 字段                    | 数据类型     | 含义                                                                                                                                                   |
| --------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `timetag`             | `string` | `时间戳，格式为: %Y%m%d %H:%M:%S`                                                                                                                           |
| `lastPrice`           | `float`  | `最新价`                                                                                                                                                |
| `open`                | `float`  | `开盘价`                                                                                                                                                |
| `high`                | `float`  | `最高价`                                                                                                                                                |
| `low`                 | `float`  | `最低价`                                                                                                                                                |
| `lastClose`           | `float`  | `前收盘价`                                                                                                                                               |
| `amount`              | `float`  | `成交额`                                                                                                                                                |
| `volume`              | `float`  | `成交量（手）`                                                                                                                                             |
| `pvolume`             | `float`  | `原始成交量（股）【不推荐使用】`                                                                                                                                    |
| `stockStatus`         | `int`    | `作废 参考openInt`                                                                                                                                       |
| `openInt`             | `float`  | `若是股票，则openInt含义为股票状态，非股票则是持仓量`[openInt字段说明](https://dict.thinktrader.net/innerApi/data_structure.html#openint-%E8%AF%81%E5%88%B8%E7%8A%B6%E6%80%81) |
| `lastSettlementPrice` | `float`  | `昨结算价`                                                                                                                                               |
| `pe`                  | `float`  | `对于股票是市盈率,对于ETF是iopv值`                                                                                                                               |
| `askPrice`            | `list`   | `委卖价`                                                                                                                                                |
| `bidPrice`            | `list`   | `委买价`                                                                                                                                                |
| `askVol`              | `list`   | `委卖量`                                                                                                                                                |
| `bidVol`              | `list`   | `委买量`                                                                                                                                                |
| `settlementPrice`     | `float`  | `今结算价`                                                                                                                                               |

#### subscribe_quote/subscribe_whole_quote回调对象：

同 `get_full_tick` 返回结构

### Bar - Bar对象

bar数据是指各种频率的行情数据

| 字段                 | 数据类型    | 含义             |
| ------------------ | ------- | -------------- |
| `time`             | `int`   | `时间`           |
| `open`             | `float` | `开盘价`          |
| `high`             | `float` | `最高价`          |
| `low`              | `float` | `最低价`          |
| `close`            | `float` | `收盘价`          |
| `volume`           | `float` | `成交量`          |
| `amount`           | `float` | `成交额`          |
| `settelementPrice` | `float` | `今结算`          |
| `openInterest`     | `float` | `持仓量`          |
| `preClose`         | `float` | `前收盘价`         |
| `suspendFlag`      | `int`   | `停牌` 1停牌，0 不停牌 |

### l2quote - Level2行情快照

| 字段名                 | 数据类型        | 解释                   |
| ------------------- | ----------- | -------------------- |
| time                | int         | 时间戳                  |
| stime               | string      | 时间戳字符串形式             |
| lastPrice           | float       | 最新价                  |
| open                | float       | 开盘价                  |
| high                | float       | 最高价                  |
| low                 | float       | 最低价                  |
| amount              | float       | 成交额                  |
| volume              | int         | 成交总量                 |
| pvolume             | int         | 原始成交总量(未经过股手转换的成交总量) |
| stockStatus         | int         | 证券状态                 |
| openInt             | int         | 持仓量                  |
| transactionNum      | int         | 成交笔数(期货没有，单独计算)      |
| lastClose           | float       | 前收盘价                 |
| lastSettlementPrice | float       | 前结算(股票为0)            |
| settlementPrice     | float       | 今结算(股票为0)            |
| askPrice            | list[float] | 多档委卖价                |
| askVol              | list[int]   | 多档委卖量                |
| bidPrice            | list[float] | 多档委买价                |
| bidVol              | list[int]   | 多档委买量                |

### l2quoteaux - Level2行情快照补充

| 字段名                 | 数据类型   | 解释       |
| ------------------- | ------ | -------- |
| time                | int    | 时间戳      |
| stime               | string | 时间戳字符串形式 |
| avgBidPrice         | float  | 委买均价     |
| totalBidQuantity    | int    | 委买总量     |
| avgOffPrice         | float  | 委卖均价     |
| totalOffQuantity    | int    | 委卖总量     |
| withdrawBidQuantity | int    | 买入撤单总量   |
| withdrawBidAmount   | float  | 买入撤单总额   |
| withdrawOffQuantity | int    | 卖出撤单总量   |
| withdrawOffAmount   | float  | 卖出撤单总额   |

### l2order - Level2逐笔委托

| 字段名              | 数据类型  | 解释                                                                                                                 |
| ---------------- | ----- | ------------------------------------------------------------------------------------------------------------------ |
| time             | int   | 时间戳                                                                                                                |
| stime            | float | 时间戳浮点数形式                                                                                                           |
| price            | float | 委托价                                                                                                                |
| volume           | int   | 委托量                                                                                                                |
| entrustNo        | int   | 委托号                                                                                                                |
| entrustType      | int   | [委托类型](https://dict.thinktrader.net/innerApi/data_structure.html#entrusttype-%E5%A7%94%E6%89%98%E7%B1%BB%E5%9E%8B) |
| entrustDirection | int   | 委托方向                                                                                                               |

提示

注：上交所的撤单信息在逐笔委托的委托方向，区分撤买撤卖

- 0 - 未知
- 1 - 买入
- 2 - 卖出
- 3 - 撤买（上交所）
- 4 - 撤卖（上交所）

### l2transaction - Level2逐笔成交

| 字段名        | 数据类型   | 解释       |
| ---------- | ------ | -------- |
| time       | int    | 时间戳      |
| stime      | string | 时间戳字符串形式 |
| price      | float  | 成交价      |
| volume     | int    | 成交量      |
| amount     | float  | 成交额      |
| tradeIndex | int    | 成交记录号    |
| buyNo      | int    | 买方委托号    |
| sellNo     | int    | 卖方委托号    |
| tradeType  | int    | 成交类型     |
| tradeFlag  | int    | 成交标志     |

提示

深交所逐笔成交的撤单标志，没有方向

- 0 - 未知
- 1 - 外盘，主买
- 2 - 内盘，主卖
- 3 - 撤单

### l2transactioncount - Level2逐笔成交统计

| 字段名                       | 数据类型  | 解释          |
| ------------------------- | ----- | ----------- |
| time                      | int   | 时间戳         |
| bidNumber                 | int   | 主买单总单数      |
| offNumber                 | int   | 主卖单总单数      |
| ddx                       | float | 大单动向        |
| ddy                       | float | 涨跌动因        |
| ddz                       | float | 大单差分        |
| netOrder                  | int   | 净挂单量        |
| netWithdraw               | int   | 净撤单量        |
| withdrawBid               | int   | 总撤买量        |
| withdrawOff               | int   | 总撤卖量        |
| bidNumberDx               | int   | 主买单总单数增量    |
| offNumberDx               | int   | 主卖单总单数增量    |
| transactionNumber         | int   | 成交笔数增量      |
| bidMostAmount             | float | 主买特大单成交额    |
| bidBigAmount              | float | 主买大单成交额     |
| bidMediumAmount           | float | 主买中单成交额     |
| bidSmallAmount            | float | 主买小单成交额     |
| bidTotalAmount            | float | 主买累计成交额     |
| offMostAmount             | float | 主卖特大单成交额    |
| offBigAmount              | float | 主卖大单成交额     |
| offMediumAmount           | float | 主卖中单成交额     |
| offSmallAmount            | float | 主卖小单成交额     |
| offTotalAmount            | float | 主卖累计成交额     |
| unactiveBidMostAmount     | float | 被动买特大单成交额   |
| unactiveBidBigAmount      | float | 被动买大单成交额    |
| unactiveBidMediumAmount   | float | 被动买中单成交额    |
| unactiveBidSmallAmount    | float | 被动买小单成交额    |
| unactiveBidTotalAmount    | float | 被动买累计成交额    |
| unactiveOffMostAmount     | float | 被动卖特大单成交额   |
| unactiveOffBigAmount      | float | 被动卖大单成交额    |
| unactiveOffMediumAmount   | float | 被动卖中单成交额    |
| unactiveOffSmallAmount    | float | 被动卖小单成交额    |
| unactiveOffTotalAmount    | float | 被动卖累计成交额    |
| netInflowMostAmount       | float | 净流入超大单成交额   |
| netInflowBigAmount        | float | 净流入大单成交额    |
| netInflowMediumAmount     | float | 净流入中单成交额    |
| netInflowSmallAmount      | float | 净流入小单成交额    |
| bidMostVolume             | int   | 主买特大单成交量    |
| bidBigVolume              | int   | 主买大单成交量     |
| bidMediumVolume           | int   | 主买中单成交量     |
| bidSmallVolume            | int   | 主买小单成交量     |
| bidTotalVolume            | int   | 主买累计成交量     |
| offMostVolume             | int   | 主卖特大单成交量    |
| offBigVolume              | int   | 主卖大单成交量     |
| offMediumVolume           | int   | 主卖中单成交量     |
| offSmallVolume            | int   | 主卖小单成交量     |
| offTotalVolume            | int   | 主卖累计成交量     |
| unactiveBidMostVolume     | int   | 被动买特大单成交量   |
| unactiveBidBigVolume      | int   | 被动买大单成交量    |
| unactiveBidMediumVolume   | int   | 被动买中单成交量    |
| unactiveBidSmallVolume    | int   | 被动买小单成交量    |
| unactiveBidTotalVolume    | int   | 被动买累计成交量    |
| unactiveOffMostVolume     | int   | 被动卖特大单成交量   |
| unactiveOffBigVolume      | int   | 被动卖大单成交量    |
| unactiveOffMediumVolume   | int   | 被动卖中单成交量    |
| unactiveOffSmallVolume    | int   | 被动卖小单成交量    |
| unactiveOffTotalVolume    | int   | 被动卖累计成交量    |
| netInflowMostVolume       | int   | 净流入超大单成交量   |
| netInflowBigVolume        | int   | 净流入大单成交量    |
| netInflowMediumVolume     | int   | 净流入中单成交量    |
| netInflowSmallVolume      | int   | 净流入小单成交量    |
| bidMostAmountDx           | float | 主买特大单成交额增量  |
| bidBigAmountDx            | float | 主买大单成交额增量   |
| bidMediumAmountDx         | float | 主买中单成交额增量   |
| bidSmallAmountDx          | float | 主买小单成交额增量   |
| bidTotalAmountDx          | float | 主买累计成交额增量   |
| offMostAmountDx           | float | 主卖特大单成交额增量  |
| offBigAmountDx            | float | 主卖大单成交额增量   |
| offMediumAmountDx         | float | 主卖中单成交额增量   |
| offSmallAmountDx          | float | 主卖小单成交额增量   |
| offTotalAmountDx          | float | 主卖累计成交额增量   |
| unactiveBidMostAmountDx   | float | 被动买特大单成交额增量 |
| unactiveBidBigAmountDx    | float | 被动买大单成交额增量  |
| unactiveBidMediumAmountDx | float | 被动买中单成交额增量  |
| unactiveBidSmallAmountDx  | float | 被动买小单成交额增量  |
| unactiveBidTotalAmountDx  | float | 被动买累计成交额增量  |
| unactiveOffMostAmountDx   | float | 被动卖特大单成交额增量 |
| unactiveOffBigAmountDx    | float | 被动卖大单成交额增量  |
| unactiveOffMediumAmountDx | float | 被动卖中单成交额增量  |
| unactiveOffSmallAmountDx  | float | 被动卖小单成交额增量  |
| unactiveOffTotalAmountDx  | float | 被动卖累计成交额增量  |
| netInflowMostAmountDx     | float | 净流入超大单成交额增量 |
| netInflowBigAmountDx      | float | 净流入大单成交额增量  |
| netInflowMediumAmountDx   | float | 净流入中单成交额增量  |
| netInflowSmallAmountDx    | float | 净流入小单成交额增量  |
| bidMostVolumeDx           | int   | 主买特大单成交量增量  |
| bidBigVolumeDx            | int   | 主买大单成交量增量   |
| bidMediumVolumeDx         | int   | 主买中单成交量增量   |
| bidSmallVolumeDx          | int   | 主买小单成交量增量   |
| bidTotalVolumeDx          | int   | 主买累计成交量增量   |
| offMostVolumeDx           | int   | 主卖特大单成交量增量  |
| offBigVolumeDx            | int   | 主卖大单成交量增量   |
| offMediumVolumeDx         | int   | 主卖中单成交量增量   |
| offSmallVolumeDx          | int   | 主卖小单成交量增量   |
| offTotalVolumeDx          | int   | 主卖累计成交量增量   |
| unactiveBidMostVolumeDx   | int   | 被动买特大单成交量增量 |
| unactiveBidBigVolumeDx    | int   | 被动买大单成交量增量  |
| unactiveBidMediumVolumeDx | int   | 被动买中单成交量增量  |
| unactiveBidSmallVolumeDx  | int   | 被动买小单成交量增量  |
| unactiveBidTotalVolumeDx  | int   | 被动买累计成交量增量  |
| unactiveOffMostVolumeDx   | int   | 被动卖特大单成交量增量 |
| unactiveOffBigVolumeDx    | int   | 被动卖大单成交量增量  |
| unactiveOffMediumVolumeDx | int   | 被动卖中单成交量增量  |
| unactiveOffSmallVolumeDx  | int   | 被动卖小单成交量增量  |
| unactiveOffTotalVolumeDx  | int   | 被动卖累计成交量增量  |
| netInflowMostVolumeDx     | int   | 净流入超大单成交量增量 |
| netInflowBigVolumeDx      | int   | 净流入大单成交量增量  |
| netInflowMediumVolumeDx   | int   | 净流入中单成交量增量  |
| netInflowSmallVolumeDx    | int   | 净流入小单成交量增量  |

### l2orderqueue - Level2委买委卖队列

## 交易类

### Account - 账户对象

| 字段名                     | 数据类型  | 解释                            |
| ----------------------- | ----- | ----------------------------- |
| m_strAccountID          | str   | 资金账号，用于识别不同的资金账户              |
| m_nBrokerType           | int   | 账号类型，表示账号的具体种类                |
| m_dMaxMarginRate        | float | 保证金比率，通常用于期货账号                |
| m_dFrozenMargin         | float | 冻结保证金，指投资者在交易中被冻结的保证金金额       |
| m_dFrozenCash           | float | 冻结金额，指投资者在交易中被冻结的资金金额         |
| m_dFrozenCommission     | float | 冻结手续费，指投资者在交易中被冻结的手续费金额       |
| m_dRisk                 | float | 风险度，指投资者账户的风险程度               |
| m_dNav                  | float | 单位净值，用于表示基金的净值                |
| m_dPreBalance           | float | 期初权益，指期初时账户的资金金额              |
| m_dBalance              | float | 总资产，表示账户的总资金金额                |
| m_dAvailable            | float | 可用金额，指账户中可用于交易和提取的资金金额        |
| m_dCommission           | float | 手续费 (旧版本为 m_dComission)       |
| m_dPositionProfit       | float | 持仓盈亏，指当前持有的证券或期货合约的盈亏金额       |
| m_dCloseProfit          | float | 平仓盈亏，在期货交易中表示已经平仓的交易的盈亏金额     |
| m_dCashIn               | float | 出入金净值，表示账户中出入金的净额             |
| m_dCurrMargin           | float | 当前使用的保证金金额                    |
| m_dInitBalance          | float | 初始权益，指账户初始时的权益金额              |
| m_strStatus             | str   | 状态，表示账户的当前状态                  |
| m_dInitCloseMoney       | float | 期初平仓盈亏，指账户初始时的平仓盈亏金额          |
| m_dInstrumentValue      | float | 总市值，表示持有的证券或期货合约的总市值          |
| m_dDeposit              | float | 入金，指账户中的入金金额                  |
| m_dWithdraw             | float | 出金，指账户中的出金金额                  |
| m_dPreCredit            | float | 上次信用额度，用于表示上次的信用额度            |
| m_dPreMortgage          | float | 上次质押，指上次的质押金额                 |
| m_dMortgage             | float | 质押，指当前的质押金额                   |
| m_dCredit               | float | 信用额度，表示账户的信用额度                |
| m_dAssetBalance         | float | 证券初始资金，表示股票账户的初始资金            |
| m_strOpenDate           | str   | 起始日期，表示账户的起始日期                |
| m_dFetchBalance         | float | 可取金额，指账户中可取出的金额               |
| m_strTradingDate        | str   | 交易日，表示当前的交易日期                 |
| m_dStockValue           | float | 股票总市值，表示股票账户中持有的股票的总市值        |
| m_dLoanValue            | float | 债券总市值，表示账户中持有的债券的总市值          |
| m_dFundValue            | float | 基金总市值，包括ETF和封闭式基金在内的基金的总市值    |
| m_dRepurchaseValue      | float | 回购总市值，表示账户中持有的所有回购交易的总市值      |
| m_dLongValue            | float | 多单总市值，指现货账户中多单持仓的总市值          |
| m_dShortValue           | float | 空单总市值，指现货账户中空单持仓的总市值          |
| m_dNetValue             | float | 净持仓总市值，指现货账户中多单总市值减去空单总市值的差额  |
| m_dAssureAsset          | float | 净资产，表示账户的净资产金额                |
| m_dTotalDebit           | float | 总负债，表示账户的总负债金额                |
| m_dEntrustAsset         | float | 可信资产，用于校对账户资金的准确性             |
| m_dInstrumentValueRMB   | float | 总市值（人民币），指沪港通账户中的持仓证券的总市值     |
| m_dSubscribeFee         | float | 申购费，指申购基金时支付的费用               |
| m_dGoldValue            | float | 库存市值，表示黄金现货账户中黄金库存的市值         |
| m_dGoldFrozen           | float | 现货冻结，表示黄金现货账户中被冻结的黄金金额        |
| m_dMargin               | float | 占用保证金，用于维持保证金                 |
| m_strMoneyType          | str   | 币种，表示账户的资金所使用的货币种类            |
| m_dPurchasingPower      | float | 购买力，指账户可用于购买投资品的金额            |
| m_dRawMargin            | float | 原始保证金，指期货账户中的原始保证金金额          |
| m_dBuyWaitMoney         | float | 买入待交收金额（元），指账户中买入股票但尚未交收的金额   |
| m_dSellWaitMoney        | float | 卖出待交收金额（元），指账户中卖出股票但尚未交收的金额   |
| m_dReceiveInterestTotal | float | 本期间应计利息，指账户本期间内应计的利息金额        |
| m_dRoyalty              | float | 权利金收支，指期货期权交易中的权利金收支金额        |
| m_dFrozenRoyalty        | float | 冻结权利金，指期货期权交易中被冻结的权利金金额       |
| m_dRealUsedMargin       | float | 实时占用保证金，用于股票期权交易中表示实时占用的保证金金额 |
| m_dRealRiskDegree       | float | 实时风险度，用于股票期权交易中表示实时的风险度       |

### Order - 委托对象

| 字段                     | 数据类型  | 解释                                                                                                                                                                         |
| ---------------------- | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| m_strAccountID         | str   | 资金账号，账号，账号，资金账号                                                                                                                                                            |
| m_strExchangeID        | str   | 证券市场                                                                                                                                                                       |
| m_strExchangeName      | str   | 交易市场                                                                                                                                                                       |
| m_strProductID         | str   | 品种代码                                                                                                                                                                       |
| m_strProductName       | str   | 品种名称                                                                                                                                                                       |
| m_strInstrumentID      | str   | 证券代码                                                                                                                                                                       |
| m_strInstrumentName    | str   | 证券名称，合约名称                                                                                                                                                                  |
| m_nRef                 | int   | 订单编号                                                                                                                                                                       |
| m_strOrderRef          | str   | 内部委托号，下单引用等于股票的内部委托号                                                                                                                                                       |
| m_nOrderPriceType      | int   | [EBrokerPriceType 类型，例如市价单、限价单](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-ebrokerpricetype-%E4%BB%B7%E6%A0%BC%E7%B1%BB%E5%9E%8B)                      |
| m_nDirection           | int   | [EEntrustBS 类型，操作，多空，期货多空，股票买卖永远是 48，其他的 dir 同理](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eentrustbs-%E4%B9%B0%E5%8D%96%E6%96%B9%E5%90%91)           |
| m_nOffsetFlag          | int   | [EOffset_Flag_Type类型，买卖/开平，用此字段区分股票买卖，期货开、平仓，期权买卖等](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eoffset-flag-type-%E6%93%8D%E4%BD%9C%E7%B1%BB%E5%9E%8B) |
| m_nHedgeFlag           | int   | [EHedge_Flag_Type 类型，投保](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-ehedge-flag-type)                                                                  |
| m_dLimitPrice          | float | 委托价格，限价单的限价，即报价                                                                                                                                                            |
| m_nVolumeTotalOriginal | int   | 委托数量，最初的委托数量                                                                                                                                                               |
| m_nOrderSubmitStatus   | int   | [EEntrustSubmitStatus 类型，报单状态，提交状态，股票中不需要报单状态](https://dict.thinktrader.net/innerApi/enum_constants.html#eentrustsubmitstatus-%E6%8A%A5%E5%8D%95%E7%8A%B6%E6%80%81)        |
| m_strOrderSysID        | str   | 合同编号，委托号                                                                                                                                                                   |
| m_nOrderStatus         | int   | [EEntrustStatus，委托状态](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eentruststatus-%E5%A7%94%E6%89%98%E7%8A%B6%E6%80%81)                                  |
| m_nVolumeTraded        | int   | 成交数量，已成交量                                                                                                                                                                  |
| m_nVolumeTotal         | int   | 委托剩余量，当前总委托量，股票中表示总委托量减去成交量                                                                                                                                                |
| m_nErrorID             | int   | 状态ID                                                                                                                                                                       |
| m_strErrorMsg          | str   | 状态信息                                                                                                                                                                       |
| m_nTaskId              | int   | 任务号                                                                                                                                                                        |
| m_dFrozenMargin        | float | 冻结金额，冻结保证金                                                                                                                                                                 |
| m_dFrozenCommission    | float | 冻结手续费                                                                                                                                                                      |
| m_strInsertDate        | str   | 委托日期，报单日期                                                                                                                                                                  |
| m_strInsertTime        | str   | 委托时间                                                                                                                                                                       |
| m_dTradedPrice         | float | 成交均价（股票）                                                                                                                                                                   |
| m_dCancelAmount        | float | 已撤数量                                                                                                                                                                       |
| m_strOptName           | str   | 买卖标记，展示委托属性的中文                                                                                                                                                             |
| m_dTradeAmount         | float | 成交金额，期货的计算方式为均价乘以数量乘以合约乘数                                                                                                                                                  |
| m_eEntrustType         | int   | [EEntrustTypes，委托类别](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eentrusttypes-%E5%A7%94%E6%89%98%E7%B1%BB%E5%9E%8B)                                    |
| m_strCancelInfo        | str   | 废单原因                                                                                                                                                                       |
| m_strUnderCode         | str   | 标的证券代码                                                                                                                                                                     |
| m_eCoveredFlag         | int   | 备兑标记，'0’表示非备兑，'1’表示备兑                                                                                                                                                      |
| m_dOrderPriceRMB       | float | 委托价格（人民币），目前用于港股通                                                                                                                                                          |
| m_dTradeAmountRMB      | float | 成交金额（人民币），目前用于港股通                                                                                                                                                          |
| m_dReferenceRate       | float | 汇率，目前用于港股通                                                                                                                                                                 |
| m_strCompactNo         | str   | 合约编号                                                                                                                                                                       |
| m_eCashgroupProp       | int   | [EXTCompactBrushSource类型，头寸来源](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extcompactbrushsource-%E5%A4%B4%E5%AF%B8%E6%9D%A5%E6%BA%90)                  |
| m_dShortOccupedMargin  | float | 预估在途占用保证金，用于期权                                                                                                                                                             |
| m_strXTTrade           | str   | 是否是迅投交易                                                                                                                                                                    |
| m_strAccountKey        | str   | 账号key，唯一区别不同账号的key                                                                                                                                                         |
| m_strRemark            | str   | 投资备注                                                                                                                                                                       |

### Deal - 成交对象

| 字段                  | 数据类型  | 解释                                                                                                                                                                       |
| ------------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| m_strAccountID      | str   | 资金账号                                                                                                                                                                     |
| m_strExchangeID     | str   | 证券市场                                                                                                                                                                     |
| m_strExchangeName   | str   | 交易市场                                                                                                                                                                     |
| m_strProductID      | str   | 品种代码                                                                                                                                                                     |
| m_strProductName    | str   | 品种名称                                                                                                                                                                     |
| m_strInstrumentID   | str   | 证券代码                                                                                                                                                                     |
| m_strInstrumentName | str   | 证券名称                                                                                                                                                                     |
| m_strTradeID        | str   | 成交编号                                                                                                                                                                     |
| m_strOrderRef       | str   | 下单引用，等于股票的内部委托号                                                                                                                                                          |
| m_strOrderSysID     | str   | 合同编号，报单编号，委托号                                                                                                                                                            |
| m_nDirection        | int   | [EEntrustBS，买卖方向 对于股票该值始终是48](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eentrustbs-%E4%B9%B0%E5%8D%96%E6%96%B9%E5%90%91)                            |
| m_nOffsetFlag       | int   | [EOffset_Flag_Type，买卖/开平，用此字段区分股票买卖，期货开、平仓，期权买卖等](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eoffset-flag-type-%E6%93%8D%E4%BD%9C%E7%B1%BB%E5%9E%8B) |
| m_nHedgeFlag        | int   | [EHedge_Flag_Type 类型，投保](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-ehedge-flag-type)                                                                |
| m_dPrice            | float | 成交均价                                                                                                                                                                     |
| m_nVolume           | int   | 成交量，期货单位手，股票做到股                                                                                                                                                          |
| m_strTradeDate      | str   | 成交日期                                                                                                                                                                     |
| m_strTradeTime      | str   | 成交时间                                                                                                                                                                     |
| m_dCommission       | float | 手续费 (旧版本为 `m_dComission`)                                                                                                                                                |
| m_dTradeAmount      | float | 成交额，期货 = 均价 * 量 * 合约乘数                                                                                                                                                   |
| m_nTaskId           | int   | 任务号                                                                                                                                                                      |
| m_nOrderPriceType   | int   | [EBrokerPriceType 类型，例如市价单、限价单](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eentrusttypes-%E5%A7%94%E6%89%98%E7%B1%BB%E5%9E%8B)                       |
| m_strOptName        | str   | 买卖标记，展示委托属性的中文                                                                                                                                                           |
| m_eEntrustType      | int   | [EEntrustTypes，委托类别](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eentrusttypes-%E5%A7%94%E6%89%98%E7%B1%BB%E5%9E%8B)                                  |
| m_eFutureTradeType  | int   | [EFutureTradeType 类型，成交类型](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-efuturetradetype-%E6%88%90%E4%BA%A4%E7%B1%BB%E5%9E%8B)                         |
| m_nRealOffsetFlag   | int   | [EOffset_Flag_Type 类型，实际开平，主要是区分平今和平昨](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eoffset-flag-type-%E6%93%8D%E4%BD%9C%E7%B1%BB%E5%9E%8B)            |
| m_eCoveredFlag      | int   | ECoveredFlag类型，备兑标记 '0' - 非备兑，'1' - 备兑                                                                                                                                   |
| m_nCloseTodayVolume | int   | 平今量，不显示                                                                                                                                                                  |
| m_dOrderPriceRMB    | float | 委托价格（人民币），目前用于港股通                                                                                                                                                        |
| m_dPriceRMB         | float | 成交价格（人民币），目前用于港股通                                                                                                                                                        |
| m_dTradeAmountRMB   | float | 成交金额（人民币），目前用于港股通                                                                                                                                                        |
| m_dReferenceRate    | float | 汇率，目前用于港股通                                                                                                                                                               |
| m_strXTTrade        | str   | 是否是迅投交易                                                                                                                                                                  |
| m_strCompactNo      | str   | 合约编号                                                                                                                                                                     |
| m_dCloseProfit      | float | 平仓盈亏，目前用于外盘                                                                                                                                                              |
| m_strRemark         | str   | 投资备注                                                                                                                                                                     |
| m_strAccountKey     | str   | 账号key，唯一区别不同账号的key                                                                                                                                                       |
| m_nRef              | int   | 订单编号                                                                                                                                                                     |

### Position - 持仓对象

| 字段名                     | 数据类型   | 含义                                                                                                                                               |
| ----------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| m_strAccountID          | string | 资金账号                                                                                                                                             |
| m_strExchangeID         | string | 证券市场                                                                                                                                             |
| m_strExchangeName       | string | 市场名称                                                                                                                                             |
| m_strProductID          | string | 品种代码                                                                                                                                             |
| m_strProductName        | string | 品种名称                                                                                                                                             |
| m_strInstrumentID       | string | 证券代码                                                                                                                                             |
| m_strInstrumentName     | string | 证券名称                                                                                                                                             |
| m_nHedgeFlag            | int    | [EHedge_Flag_Type 类型，投保 ，股票不适用](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-ehedge-flag-type)                                 |
| m_nDirection            | int    | [EEntrustBS，买卖方向 对于股票该值始终是48](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eentrustbs-%E4%B9%B0%E5%8D%96%E6%96%B9%E5%90%91)    |
| m_strOpenDate           | string | 开仓日期 股票此字段无效                                                                                                                                     |
| m_strTradeID            | string | 成交号，最初开仓位的成交                                                                                                                                     |
| m_nVolume               | int    | 当前拥股/持仓量                                                                                                                                         |
| m_dOpenPrice            | float  | 持仓成本 ；持仓成本 = (总买入金额 - 总卖出金额) / 剩余数量                                                                                                              |
| m_strTradingDay         | string | 在实盘运行中是当前交易日，在回测中是股票最后交易过的日期                                                                                                                     |
| m_dMargin               | float  | 使用的保证金，历史的直接用ctp的，新的自己用成本价*存量*系数算，股票不适用                                                                                                          |
| m_dOpenCost             | float  | 开仓成本，等于成本价*第一次建仓的量，后续减持会影响，不算手续费，股票不适用                                                                                                           |
| m_dSettlementPrice      | float  | 最新结算价/当前价                                                                                                                                        |
| m_nCloseVolume          | int    | 平仓量（对于股票不适用）                                                                                                                                     |
| m_dCloseAmount          | float  | 平仓额（对于股票不适用）                                                                                                                                     |
| m_dFloatProfit          | float  | 浮动盈亏                                                                                                                                             |
| m_dCloseProfit          | float  | 平仓盈亏（对于股票不适用）                                                                                                                                    |
| m_dMarketValue          | float  | 市值/合约价值                                                                                                                                          |
| m_dPositionCost         | float  | 持仓成本（对于股票不适用）                                                                                                                                    |
| m_dPositionProfit       | float  | 持仓盈亏（对于股票不适用）                                                                                                                                    |
| m_dLastSettlementPrice  | float  | 最新结算价（对于股票不适用）                                                                                                                                   |
| m_dInstrumentValue      | float  | 合约价值（对于股票不适用）                                                                                                                                    |
| m_bIsToday              | bool   | 是否今仓                                                                                                                                             |
| m_strStockHolder        | string | 股东账号                                                                                                                                             |
| m_nFrozenVolume         | int    | 冻结数量                                                                                                                                             |
| m_nCanUseVolume         | int    | 可用余额                                                                                                                                             |
| m_nOnRoadVolume         | int    | 在途股份                                                                                                                                             |
| m_nYesterdayVolume      | int    | 昨夜拥股                                                                                                                                             |
| m_dLastPrice            | float  | 最新价/当前价                                                                                                                                          |
| m_dAvgOpenPrice         | float  | 开仓均价（对于股票不适用）                                                                                                                                    |
| m_dProfitRate           | float  | 盈亏比例                                                                                                                                             |
| m_eFutureTradeType      | int    | [EFutureTradeType 类型，成交类型](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-efuturetradetype-%E6%88%90%E4%BA%A4%E7%B1%BB%E5%9E%8B) |
| m_strExpireDate         | string | 到期日（针对逆回购）                                                                                                                                       |
| m_strComTradeID         | string | 组合成交号                                                                                                                                            |
| m_nLegId                | int    | 组合序号                                                                                                                                             |
| m_dTotalCost            | float  | 累计成本（自定义，股票信用用到）                                                                                                                                 |
| m_dSingleCost           | float  | 单股成本（自定义，股票信用用                                                                                                                                   |
| m_nCoveredVolume        | int    | 备兑数量，用于个股期权                                                                                                                                      |
| m_eSideFlag             | int    | 持仓类型 ，用于个股期权，标记 '0' - 权利，'1' - 义务，'2' - '备兑'                                                                                                     |
| m_dReferenceRate        | float  | 汇率，目前用于港股通                                                                                                                                       |
| m_dStructFundVol        | float  | 分级基金可用（可分拆或可合并）                                                                                                                                  |
| m_dRedemptionVolume     | float  | 分级基金可赎回量                                                                                                                                         |
| m_nPREnableVolume       | int    | 申赎可用量（记录当日申购赎回的股票或基金数量）                                                                                                                          |
| m_dRealUsedMargin       | float  | 实时占用保证金，用于期权                                                                                                                                     |
| m_dRoyalty              | float  | 权利金                                                                                                                                              |
| m_dStockLastPrice       | float  | 标的证券最新价，用于期权                                                                                                                                     |
| m_dStaticHoldMargin     | float  | 静态持仓占用保证金，用于期权                                                                                                                                   |
| m_nOptCombUsedVolume    | int    | 期权组合占用数量                                                                                                                                         |
| m_nEnableExerciseVolume | int    | 能够行使的数量，用于个股期权                                                                                                                                   |
| m_strAccountKey         | string | 账号key，唯一区别不同账号的key                                                                                                                               |

### PositionStatistics - 持仓统计对象

| 字段名                                | 数据类型     | 描述          |
| ---------------------------------- | -------- | ----------- |
| `m_strAccountID`                   | `string` | 账号          |
| `m_strExchangeID`                  | `string` | 市场代码        |
| `m_strExchangeName`                | `string` | 市场名称        |
| `m_strProductID`                   | `string` | 品种代码        |
| `m_strInstrumentID`                | `string` | 合约代码        |
| `m_strInstrumentName`              | `string` | 合约名称        |
| `m_nDirection`                     | `int`    | 多空          |
| `m_nHedgeFlag`                     | `int`    | 投保          |
| `m_nPosition`                      | `int`    | 持仓          |
| `m_nYestodayPosition`              | `int`    | 昨仓          |
| `m_nTodayPosition`                 | `int`    | 今仓          |
| `m_nCanCloseVol`                   | `int`    | 可平          |
| `m_dPositionCost`                  | `float`  | 持仓成本        |
| `m_dAvgPrice`                      | `float`  | 持仓均价        |
| `m_dPositionProfit`                | `float`  | 持仓盈亏        |
| `m_dFloatProfit`                   | `float`  | 浮动盈亏        |
| `m_dOpenPrice`                     | `float`  | 开仓均价        |
| `m_dUsedMargin`                    | `float`  | 已使用保证金      |
| `m_dUsedCommission`                | `float`  | 已使用的手续费     |
| `m_dFrozenMargin`                  | `float`  | 冻结保证金       |
| `m_dFrozenCommission`              | `float`  | 冻结手续费       |
| `m_dInstrumentValue`               | `float`  | 市值，合约价值     |
| `m_nOpenTimes`                     | `int`    | 开仓次数        |
| `m_nOpenVolume`                    | `int`    | 总开仓量 中间平仓不减 |
| `m_nCancelTimes`                   | `int`    | 撤单次数        |
| `m_dLastPrice`                     | `float`  | 最新价         |
| `m_dRiseRatio`                     | `float`  | 当日涨幅        |
| `m_strProductName`                 | `string` | 产品名称        |
| `m_dRoyalty`                       | `float`  | 权利金市值       |
| `m_strExpireDate`                  | `string` | 到期日         |
| `m_dAssestWeight`                  | `float`  | 资产占比        |
| `m_dIncreaseBySettlement`          | `float`  | 当日涨幅（结）     |
| `m_dMarginRatio`                   | `float`  | 保证金占比       |
| `m_dFloatProfitDivideByUsedMargin` | `float`  | 浮盈比例（保证金）   |
| `m_dFloatProfitDivideByBalance`    | `float`  | 浮盈比例（动态权益）  |
| `m_dTodayProfitLoss`               | `float`  | 当日盈亏（结）     |
| `m_nYestodayInitPosition`          | `int`    | 昨日持仓        |
| `m_dFrozenRoyalty`                 | `float`  | 冻结权利金       |
| `m_dTodayCloseProfitLoss`          | `float`  | 当日盈亏（收）     |
| `m_dCloseProfit`                   | `float`  | 平仓盈亏        |
| `m_strFtProductName`               | `string` | 品种名称        |
| `m_dOpenCost`                      | `float`  | 开仓成本        |

### CCreditAccountDetail - 信用账号对象(非查柜台)

| 字段名                             | 数据类型  | 解释                                                           |
| ------------------------------- | ----- | ------------------------------------------------------------ |
| m_strAccountID                  | str   | 资金账号                                                         |
| m_nBrokerType                   | int   | 账号类型，1-期货账号，2-股票账号，3-信用账号，5-期货期权账号，6-股票期权账号，7-沪港通账号，11-深港通账号 |
| m_strAccountKey                 | str   | 唯一区别不同账号的key                                                 |
| m_dMaxMarginRate                | float | 保证金比率，股票的保证金率等于1                                             |
| m_dFrozenMargin                 | float | 冻结保证金，外源性，股票的保证金就是冻结资金，股票不适用                                 |
| m_dFrozenCash                   | float | 冻结金额，内外源冻结保证金和手续费四个的和                                        |
| m_dFrozenCommission             | float | 冻结手续费，外源性冻结资金源                                               |
| m_dRisk                         | float | 风险度，冻结资金/可用资金                                                |
| m_dNav                          | float | 单位净值                                                         |
| m_dPreBalance                   | float | 期初权益，也叫静态权益，股票不适用                                            |
| m_dBalance                      | float | 总资产，动态权益，即市值                                                 |
| m_dAvailable                    | float | 可用金额                                                         |
| m_dCommission                   | float | 手续费(旧版本为 m_dComission)                                       |
| m_dPositionProfit               | float | 持仓盈亏                                                         |
| m_dCloseProfit                  | float | 平仓盈亏，股票不适用                                                   |
| m_dCashIn                       | float | 出入金净值                                                        |
| m_dCurrMargin                   | float | 当前使用的保证金，股票不适用                                               |
| m_dInitBalance                  | float | 初始权益                                                         |
| m_strStatus                     | str   | 状态                                                           |
| m_dInitCloseMoney               | float | 期初平仓盈亏，初始平仓盈亏                                                |
| m_dInstrumentValue              | float | 总市值，合约价值，合约价值                                                |
| m_dDeposit                      | float | 入金                                                           |
| m_dWithdraw                     | float | 出金                                                           |
| m_dPreCredit                    | float | 上次信用额度，股票不适用                                                 |
| m_dPreMortgage                  | float | 上次质押，股票不适用                                                   |
| m_dMortgage                     | float | 质押，股票不适用                                                     |
| m_dCredit                       | float | 信用额度，股票不适用                                                   |
| m_dAssetBalance                 | float | 证券初始资金，股票不适用                                                 |
| m_strOpenDate                   | str   | 起始日期股票不适用                                                    |
| m_dFetchBalance                 | float | 可取金额                                                         |
| m_strTradingDate                | str   | 交易日                                                          |
| m_dStockValue                   | float | 股票总市值，期货没有                                                   |
| m_dLoanValue                    | float | 债券总市值，期货没有                                                   |
| m_dFundValue                    | float | 基金总市值，包括 ETF 和封闭式基金，期货没有                                     |
| m_dRepurchaseValue              | float | 回购总市值，所有回购，期货没有                                              |
| m_dLongValue                    | float | 多单总市值，现货没有                                                   |
| m_dShortValue                   | float | 单总市值，现货没有                                                    |
| m_dNetValue                     | float | 净持仓总市值，净持仓市值 = 多 - 空                                         |
| m_dAssureAsset                  | float | 净资产                                                          |
| m_dEntrustAsset                 | float | 可信资产，用于校对                                                    |
| m_dInstrumentValueRMB           | float | 总市值（人民币），沪港通                                                 |
| m_dSubscribeFee                 | float | 申购费，申购费                                                      |
| m_dGoldValue                    | float | 库存市值，黄金现货库存市值                                                |
| m_dGoldFrozen                   | float | 现货冻结，黄金现货冻结                                                  |
| m_dMargin                       | float | 占用保证金，维持保证金                                                  |
| m_strMoneyType                  | str   | 币种                                                           |
| m_dPurchasingPower              | float | 购买力，盈透购买力                                                    |
| m_dRawMargin                    | float | 原始保证金                                                        |
| m_dBuyWaitMoney                 | float | 买入待交收金额（元），买入待交收                                             |
| m_dSellWaitMoney                | float | 卖出待交收金额（元），卖出待交收                                             |
| m_dReceiveInterestTotal         | float | 本期间应计利息                                                      |
| m_dRoyalty                      | float | 权利金收支，期货期权用                                                  |
| m_dFrozenRoyalty                | float | 冻结权利金，期货期权用                                                  |
| m_dRealUsedMargin               | float | 实时占用保证金，用于股票期权                                               |
| m_dRealRiskDegree               | float | 实时风险度                                                        |
| m_dPerAssurescaleValue          | float | 个人维持担保比例                                                     |
| m_dEnableBailBalance            | float | 可用保证金                                                        |
| m_dUsedBailBalance              | float | 已用保证金                                                        |
| m_dAssureEnbuyBalance           | float | 可买担保品资金                                                      |
| m_dFinEnbuyBalance              | float | 可买标的券资金                                                      |
| m_dSloEnrepaidBalance           | float | 可还券资金                                                        |
| m_dFinEnrepaidBalance           | float | 可还款资金                                                        |
| m_dFinMaxQuota                  | float | 融资授信额度                                                       |
| m_dFinEnableQuota               | float | 融资可用额度                                                       |
| m_dFinUsedQuota                 | float | 融资已用额度                                                       |
| m_dFinUsedBail                  | float | 融资已用保证金额                                                     |
| m_dFinCompactBalance            | float | 融资合约金额                                                       |
| m_dFinCompactFare               | float | 融资合约费用                                                       |
| m_dFinCompactInterest           | float | 融资合约利息                                                       |
| m_dFinMarketValue               | float | 融资市值                                                         |
| m_dFinIncome                    | float | 融资合约盈亏                                                       |
| m_dSloMaxQuota                  | float | 融券授信额度                                                       |
| m_dSloEnableQuota               | float | 融券可用额度                                                       |
| m_dSloUsedQuota                 | float | 融券已用额度                                                       |
| m_dSloUsedBail                  | float | 融券已用保证金额                                                     |
| m_dSloCompactBalance            | float | 融券合约金额                                                       |
| m_dSloCompactFare               | float | 融券合约费用                                                       |
| m_dSloCompactInterest           | float | 融券合约利息                                                       |
| m_dSloMarketValue               | float | 融券市值                                                         |
| m_dSloIncome                    | float | 融券合约盈亏                                                       |
| m_dOtherFare                    | float | 其它费用                                                         |
| m_dUnderlyMarketValue           | float | 标的证券市值                                                       |
| m_dFinEnableBalance             | float | 可融资金额                                                        |
| m_dDiffEnableBailBalance        | float | 可用保证金调整值                                                     |
| m_dBuySecuRepayFrozenMargin     | float | 买券还券冻结资金                                                     |
| m_dBuySecuRepayFrozenCommission | float | 买券还券冻结手续费                                                    |
| m_dSpecialEnableBalance         | float | 专项可融金额                                                       |
| m_dEncumberedAssets             | float | 担保资产                                                         |
| m_dSloSellBalance               | float | 融券卖出资金                                                       |
| m_dDiffAssureEnbuyBalance       | float | 可买担保品资金调整值                                                   |
| m_dDiffFinEnbuyBalance          | float | 可买标的券资金调整值                                                   |
| m_dDiffFinEnrepaidBalance       | float | 可还款资金调整值                                                     |
| m_dOtherRealCompactBalance      | float | 其他负债合约金额                                                     |
| m_dOtherFinCompactInterest      | float | 其他负债合约利息金额                                                   |
| m_dUsedSloSellBalance           | float | 已用融券卖出资金                                                     |
| m_dFetchAssetBalance            | float | 可提出资产总额                                                      |
| m_dTotalEnableQuota             | float | 可用总信用额度                                                      |
| m_dTotalUsedQuota               | float | 已用总信用额度                                                      |
| m_dDebtProfit                   | float | 负债总浮盈                                                        |
| m_dDebtLoss                     | float | 负债总浮亏                                                        |
| m_nContractEndDate              | int   | 合同到期日期                                                       |
| m_dFinDebt                      | float | 融资负债                                                         |
| m_dFinProfitAmortized           | float | 融资浮盈折算                                                       |
| m_dSloProfit                    | float | 融券浮盈                                                         |
| m_dSloProfitAmortized           | float | 融券浮盈折算                                                       |
| m_dFinLoss                      | float | 融资浮亏                                                         |
| m_dSloLoss                      | float | 融券浮亏                                                         |

### CCreditDetail - 两融资金信息(查柜台)

| 字段名                      | 数据类型   | 解释       |
| ------------------------ | ------ | -------- |
| m_dPerAssurescaleValue   | float  | 维持担保比例   |
| m_dBalance               | float  | 总资产      |
| m_dTotalDebt             | float  | 总负债      |
| m_dAssureAsset           | float  | 净资产      |
| m_dMarketValue           | float  | 总市值      |
| m_dEnableBailBalance     | float  | 可用保证金    |
| m_dAvailable             | float  | 可用资金     |
| m_dFinDebt               | float  | 融资负债     |
| m_dFinDealAvl            | float  | 融资本金     |
| m_dFinFee                | float  | 融资息费     |
| m_dSloDebt               | float  | 融券负债     |
| m_dSloMarketValue        | float  | 融券市值     |
| m_dSloFee                | float  | 融券息费     |
| m_dOtherFare             | float  | 其它费用     |
| m_dFinMaxQuota           | float  | 融资授信额度   |
| m_dFinEnableQuota        | float  | 融资可用额度   |
| m_dFinUsedQuota          | float  | 融资冻结额度   |
| m_dSloMaxQuota           | float  | 融券授信额度   |
| m_dSloEnableQuota        | float  | 融券可用额度   |
| m_dSloUsedQuota          | float  | 融券冻结额度   |
| m_dSloSellBalance        | float  | 融券卖出资金   |
| m_dUsedSloSellBalance    | float  | 已用融券卖出资金 |
| m_dSurplusSloSellBalance | float  | 剩余融券卖出资金 |
| m_dStockValue            | float  | 股票市值     |
| m_dFundValue             | float  | 基金市值     |
| error                    | string | 错误信息     |

### CreditSloEnableAmount - 可融券明细对象

提示

由于字段m_dSloRatio、m_dSloStatus提供来源和取担保品明细**get_assure_contract**重复，字段在2021年9月移除，后续用担保品明细接口获取,具体见 [担保标的对象字段说明](https://dict.thinktrader.net/innerApi/data_structure.html?id=null#stksubjects-%E6%8B%85%E4%BF%9D%E6%A0%87%E7%9A%84%E5%AF%B9%E8%B1%A1)

| 字段名               | 数据类型   | 解释                                                                                                                                                  |
| ----------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| m_nPlatformID     | int    | 平台号                                                                                                                                                 |
| m_strBrokerID     | string | 经纪公司编号                                                                                                                                              |
| m_strBrokerName   | string | 经纪公司                                                                                                                                                |
| m_strAccountID    | string | 资金账号                                                                                                                                                |
| m_strExchangeID   | string | 交易所                                                                                                                                                 |
| m_strInstrumentID | string | 证券代码                                                                                                                                                |
| m_nEnableAmount   | int    | 融券可融数量                                                                                                                                              |
| m_eQuerySloType   | enum   | [EXTSloTypeQueryMode](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extslotypequerymode-%E6%9F%A5%E8%AF%A2%E7%B1%BB%E5%9E%8B)，查询类型 |

### StkCompacts - 负债合约对象

| 字段名                         | 数据类型   | 解释                                                                                                                                                      |
| --------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| m_strAccountID              | string | 资金账号，账号，账号，资金账号                                                                                                                                         |
| m_strExchangeID             | string | 交易所                                                                                                                                                     |
| m_strInstrumentID           | string | 证券代码                                                                                                                                                    |
| m_strExchangeName           | string | 交易所名称                                                                                                                                                   |
| m_strInstrumentName         | string | 股票名称                                                                                                                                                    |
| m_nOpenDate                 | int    | 合约开仓日期                                                                                                                                                  |
| m_strCompactId              | string | 合约编号                                                                                                                                                    |
| m_dCrdtRatio                | float  | 融资融券保证金比例                                                                                                                                               |
| m_strEntrustNo              | string | 委托编号                                                                                                                                                    |
| m_dEntrustPrice             | float  | 委托价格                                                                                                                                                    |
| m_nEntrustVol               | int    | 委托数量                                                                                                                                                    |
| m_nBusinessVol              | int    | 合约开仓数量                                                                                                                                                  |
| m_dBusinessBalance          | float  | 合约开仓金额                                                                                                                                                  |
| m_dBusinessFare             | float  | 合约开仓费用                                                                                                                                                  |
| m_eCompactType              | enum   | [EXTCompactType](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extcompacttype-%E5%90%88%E7%BA%A6%E7%B1%BB%E5%9E%8B)，合约类型               |
| m_eCompactStatus            | enum   | [EXTCompactStatus](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extcompactstatus-%E5%90%88%E7%BA%A6%E7%8A%B6%E6%80%81)，合约状态           |
| m_dRealCompactBalance       | float  | 未还合约金额                                                                                                                                                  |
| m_nRealCompactVol           | int    | 未还合约数量                                                                                                                                                  |
| m_dRealCompactFare          | float  | 未还合约费用                                                                                                                                                  |
| m_dRealCompactInterest      | float  | 未还合约利息                                                                                                                                                  |
| m_dRepaidInterest           | float  | 已还利息                                                                                                                                                    |
| m_nRepaidVol                | int    | 已还数量                                                                                                                                                    |
| m_dRepaidBalance            | float  | 已还金额                                                                                                                                                    |
| m_dCompactInterest          | float  | 合约总利息                                                                                                                                                   |
| m_dUsedBailBalance          | float  | 占用保证金                                                                                                                                                   |
| m_dYearRate                 | float  | 合约年利率                                                                                                                                                   |
| m_nRetEndDate               | int    | 归还截止日                                                                                                                                                   |
| m_strDateClear              | string | 了结日期                                                                                                                                                    |
| m_strPositionStr            | string | 定位串                                                                                                                                                     |
| m_dPrice                    | float  | 最新价                                                                                                                                                     |
| m_nOpenTime                 | int    | 合约开仓时间                                                                                                                                                  |
| m_nCancelVol                | int    | 合约撤单数量                                                                                                                                                  |
| m_eCashgroupProp            | enum   | [EXTCompactBrushSource](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extcompactbrushsource-%E5%A4%B4%E5%AF%B8%E6%9D%A5%E6%BA%90)，头寸来源 |
| m_dUnRepayBalance           | float  | 负债金额                                                                                                                                                    |
| m_nRepayPriority            | int    | 偿还优先级                                                                                                                                                   |
| m_dRealDefaultInterest      | float  | 未还罚息                                                                                                                                                    |
| m_dOtherRealCompactBalance  | float  | 其他负债合约金额                                                                                                                                                |
| m_dOtherRealCompactInterest | float  | 其他负债合约利息金额                                                                                                                                              |

### StkSubjects - 担保标的对象

| 字段名               | 数据类型   | 解释                                                                                                                                                                  |
| ----------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| m_nPlatformID     | int    | 平台号//目前主要用于区别不同的行情，根据此来选择对应行情                                                                                                                                       |
| m_strBrokerID     | string | 经纪公司编号                                                                                                                                                              |
| m_strBrokerName   | string | 经纪公司名称                                                                                                                                                              |
| m_strExchangeID   | string | 交易所                                                                                                                                                                 |
| m_strInstrumentID | string | 证券代码                                                                                                                                                                |
| m_dSloRatio       | float  | 融券保证金比例                                                                                                                                                             |
| m_eSloStatus      | enum   | [EXTSubjectsStatus](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extsubjectsstatus-%E8%9E%8D%E8%B5%84%E8%9E%8D%E5%88%B8%E7%8A%B6%E6%80%81)，融券状态   |
| m_dFinRatio       | float  | 融资保证金比例                                                                                                                                                             |
| m_eFinStatus      | enum   | [EXTSubjectsStatus](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extsubjectsstatus-%E8%9E%8D%E8%B5%84%E8%9E%8D%E5%88%B8%E7%8A%B6%E6%80%81)，融资状态   |
| m_strAccountID    | string | 资金账号                                                                                                                                                                |
| m_eCreditFundCtl  | enum   | [EXTCreditFundCtl](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extcreditfundctl-%E8%9E%8D%E8%B5%84%E4%BA%A4%E6%98%93%E6%8E%A7%E5%88%B6)，融资交易控制   |
| m_eCreditStkCtl   | enum   | [EXTCreditStkCtl](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extcreditstkctl-%E8%9E%8D%E5%88%B8%E4%BA%A4%E6%98%93%E6%8E%A7%E5%88%B6)，融券交易控制     |
| m_eAssureStatus   | enum   | [EXTSubjectsStatus](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-extsubjectsstatus-%E8%9E%8D%E8%B5%84%E8%9E%8D%E5%88%B8%E7%8A%B6%E6%80%81)，是否可做担保 |
| m_dAssureRatio    | float  | 担保品折算比例                                                                                                                                                             |

### PassorderArguments - 下单函数参数对象

| 字段名          | 数据类型   | 解释                    |
| ------------ | ------ | --------------------- |
| opType       | int    | passorder的opType参数    |
| orderType    | int    | passorder的orderType参数 |
| accountID    | string | 资金账号                  |
| orderCode    | string | 交易代码                  |
| prType       | int    | passorder的prType，价格类型 |
| modelPrice   | float  | 下单价格                  |
| modelVolume  | int    | 下单量（手数或股数）            |
| strategyName | string | 策略名 _ &&& _ 投资备注      |

### CTaskDetail - 任务对象

| 字段名              | 数据类型   | 解释                                                                                                                                                                                                                                             |
| ---------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| m_nTaskId        | int    | 任务号                                                                                                                                                                                                                                            |
| m_eStatus        | enum   | 任务状态 [ETaskStatus类型,见ETaskStatus说明](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-etaskstatus-%E4%BB%BB%E5%8A%A1%E7%8A%B6%E6%80%81)                                                                                           |
| m_strMsg         | string | 任务状态消息                                                                                                                                                                                                                                         |
| m_startTime      | int    | 任务开始时间, 时间戳类型                                                                                                                                                                                                                                  |
| m_endTime        | int    | 任务结束时间, 时间戳类型                                                                                                                                                                                                                                  |
| m_cancelTime     | int    | 任务取消时间                                                                                                                                                                                                                                         |
| m_nBusinessNum   | int    | 已成交量                                                                                                                                                                                                                                           |
| m_nGroupId       | int    | 组合Id                                                                                                                                                                                                                                           |
| m_stockCode      | string | 下单代码(不针对组合下单)                                                                                                                                                                                                                                  |
| m_strAccountID   | string | 下单用户(单用户下单)                                                                                                                                                                                                                                    |
| m_eOperationType | enum   | 下单操作：[开平、多空……EOperationType类型, 见EOperationType说明](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eoperationtype-%E4%B8%8B%E5%8D%95%E6%93%8D%E4%BD%9C%E7%B1%BB%E5%9E%8B-%E4%B8%BB%E8%A6%81%E4%BA%A4%E6%98%93%E7%B1%BB%E5%9E%8B) |
| m_eOrderType     | enum   | 算法交易、普通交易 [EOrderType类型, 见EOrderType说明](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eordertype-%E7%AE%97%E6%B3%95%E4%BA%A4%E6%98%93%E3%80%81%E6%99%AE%E9%80%9A%E4%BA%A4%E6%98%93%E7%B1%BB%E5%9E%8B)                         |
| m_ePriceType     | enum   | 报价方式：对手、最新…… [EPriceType类型见EPriceType说明](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-epricetype-%E4%BB%B7%E6%A0%BC%E7%B1%BB%E5%9E%8B)                                                                                       |
| m_dFixPrice      | float  | 委托价                                                                                                                                                                                                                                            |
| m_nNum           | int    | 委托量                                                                                                                                                                                                                                            |
| m_strRemark      | string | 投资备注                                                                                                                                                                                                                                           |

### CLockPosition - 期权标的持仓

| 字段名                 | 数据类型   | 解释    |
| ------------------- | ------ | ----- |
| m_strAccountID      | string | 账号名   |
| m_strExchangeID     | string | 交易所   |
| m_strExchangeName   | string | 交易所名  |
| m_strInstrumentID   | string | 标的代码  |
| m_strInstrumentName | string | 标的名称  |
| m_totalVol          | int    | 总持仓量  |
| m_lockVol           | int    | 可用锁定量 |
| m_unlockVol         | int    | 未锁定量  |
| m_coveredVol        | int    | 备兑量   |
| m_nOnRoadcoveredVol | int    | 在途备兑量 |

### CStkOptCombPositionDetail - 期权组合持仓

| 字段名                  | 数据类型   | 解释                        |
| -------------------- | ------ | ------------------------- |
| m_strAccountID       | string | 账号名                       |
| m_strExchangeID      | string | 交易所                       |
| m_strExchangeName    | string | 交易所名                      |
| m_strContractAccount | string | 合约账号                      |
| m_strCombID          | string | 组合编号                      |
| m_strCombCode        | string | 组合策略编码                    |
| m_strCombCodeName    | string | 组合策略名称                    |
| m_nVolume            | int    | 持仓量                       |
| m_nFrozenVolume      | int    | 冻结数量                      |
| m_nCanUseVolume      | int    | 可用数量                      |
| m_strFirstCode       | string | 合约一                       |
| m_eFirstCodeType     | enum   | 合约一类型 认购:48,认沽:49         |
| m_strFirstCodeName   | string | 合约一名称                     |
| m_eFirstCodePosType  | enum   | 合约一持仓类型 认购:48,义务:49,备兑:50 |
| m_nFirstCodeAmt      | int    | 合约一数量                     |
| m_strSecondCode      | string | 合约二                       |
| m_eSecondCodeType    | enum   | 合约二类型 认购:48,认沽:49         |
| m_strSecondCodeName  | string | 合约二名称                     |
| m_eSecondCodePosType | enum   | 合约二持仓类型 权利:48,义务:49,备兑:50 |
| m_nSecondCodeAmt     | int    | 合约二数量                     |
| m_dCombBailBalance   | float  | 占用保证金                     |

### entrustType - 委托类型

- 0 - 未知
- 1 - 正常交易业务
- 2 - 即时成交剩余撤销
- 3 - ETF基金申报
- 4 - 最优五档即时成交剩余撤销
- 5 - 全额成交或撤销
- 6 - 本方最优价格
- 7 - 对手方最优价格

### openInt - 证券状态

| 编码   | 状态                                                                                                    |
| ---- | ----------------------------------------------------------------------------------------------------- |
| 0,10 | 默认为未知                                                                                                 |
| 1    | 停牌                                                                                                    |
| 11   | 开盘前S                                                                                                  |
| 12   | 集合竞价时段C                                                                                               |
| 13   | 连续交易T                                                                                                 |
| 14   | 休市B                                                                                                   |
| 15   | 闭市E                                                                                                   |
| 16   | 波动性中断V,例如(10006742.SHO)50ETF沽9月2300在2024/08/28 10:15:34 - 2024/08/28 10:18:34 触发熔断临时停牌，此时的openInt值为16 |
| 17   | 临时停牌P                                                                                                 |
| 18   | 收盘集合竞价U                                                                                               |
| 19   | 盘中集合竞价M                                                                                               |
| 20   | 暂停交易至闭市N                                                                                              |
| 21   | 获取字段异常                                                                                                |
| 22   | 盘后固定价格行情                                                                                              |
| 23   | 盘后固定价格行情完毕                                                                                            |

### ContextInfo 对象

ContextInfo 是策略运行环境对象，是 init, after_init, handlebar 等基本方法的入参，里面包括了终端自带的属性和方法。一般情况下不建议对ContextInfo添加自定义属性，ContextInfo会随着bar的切换而重置到上一根bar的结束状态，建议用自建的全局变量来存储。[详细说明请看这里](https://dict.thinktrader.net/innerApi/question_answer.html?id=null#%E7%B3%BB%E7%BB%9F%E5%AF%B9%E8%B1%A1-contextinfo-%E9%80%90-k-%E7%BA%BF%E4%BF%9D%E5%AD%98%E7%9A%84%E6%9C%BA%E5%88%B6)

### init - 初始化函数

初始化函数，只在整个策略开始时调用运行到一次。用于初始订阅行情，订阅账号信息使用。init函数执行完成前部分接口无法使用，如交易日获取函数get_trading_dates。

**系统函数 不可被手动调用**

**参数：**

| 名称            | 类型       | 描述                      |
| ------------- | -------- | ----------------------- |
| `ContextInfo` | `object` | 策略运行环境对象，可以用于存储自定义的全局变量 |

**返回：** 无

**示例：**

python

```
def init(ContextInfo):
    ContextInfo.initProfit = 0
```

**在init函数中订阅行情示例：**

python

```
#coding:gbk

def init(C):
    #init函数入参为ContextInfo对象 定义时可以选择更简短的形参名 如C
    #在init函数中 可以进行 订阅行情的操作
    #如需在行情回调函数中下单 下单函数需要传入ContextInfo对象 可以通过在init中定义回调函数 来使用外层的ContextInfo
    def my_callback_function(data):
        #自定义行情回调函数 入参为指数据字典
        print(data)
    stock = '600000.SH'
    C.subscribe_quote(stock, period = '5m', callback = my_callback_function)
    #init函数执行完成后 
    print('init函数执行完成')
```

### after_init - 初始化后函数

后初始化函数，在初始化函数执行完成后被调用一次。可以用于放置一次性触发的下单，取数据操作代码。

系统会在`init`函数执行完后和执行`handlebar`之前调用`after_init`, 有些`init`里不支持的函数比如[ContextInfo.get_trading_dates](https://dict.thinktrader.net/pages/fd9cbd/#_23-%E8%8E%B7%E5%8F%96%E6%8C%87%E5%AE%9A%E4%B8%AA%E8%82%A1-%E5%90%88%E7%BA%A6-%E6%8C%87%E6%95%B0%E7%9A%84-k-%E7%BA%BF-%E4%BA%A4%E6%98%93%E6%97%A5-%E5%88%97%E8%A1%A8-contextinfo-get-trading-dates)可以在`after_init`里调用。

**系统函数 不可被手动调用**

**参数：**

| 名称            | 类型       | 描述                      |
| ------------- | -------- | ----------------------- |
| `ContextInfo` | `object` | 策略运行环境对象，可以用于存储自定义的全局变量 |

**返回：** 无

**示例：**

python

```
#coding:gbk
def init(ContextInfo):
    print('init')  


def after_init(ContextInfo):
    print('系统会在init函数执行完后和执行handlebar之前调用after_init')


def handlebar(ContextInfo):
    if ContextInfo.is_last_bar():
        print('handlebar')
```

**after_init函数中立刻下单示例：**

python

```
#coding:gbk

def after_init(C):
    #after_init 函数 可以用于执行运行开始时 需要执行一次的代码 例如下一笔委托
    #account变量是模型交易界面 添加策略时选择的资金账号 不需要手动填写 交易模型需要在模型交易界面运行 才有效
    #快速交易参数(quickTrade )填2 passorder函数执行后立刻下单 不会等待k线走完再委托。 可以在after_init函数 run_time函数注册的回调函数里进行委托 
    msg = f"投资备注字符串 用来区分不同委托"
    passorder(23, 1101, account, '600000.SH', 5, -1, 100, '测试下单', 2, msg, C)
```

### handlebar - 行情事件函数

**系统函数 不可被手动调用**

**释义：** 行情事件函数，每根 K 线运行一次；实时行情获取状态下，先每根历史 K 线运行一次，再在每个 tick 数据来后驱动运行一次

历史k线上，按时间顺序每根K线触发一次调用；盘中，每个新到达的TICK数据驱动运行一次。可以作为行情驱动的函数，实现指标计算，回测，实盘下单的效果。

**参数：**

| 名称            | 类型       | 描述                      |
| ------------- | -------- | ----------------------- |
| `ContextInfo` | `object` | 策略运行环境对象，可以用于存储自定义的全局变量 |

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 输出当前运行到的 K 线的位置
    print(ContextInfo.barpos)
```

### ContextInfo.schedule_run - 设置定时器

说明

1. 该函数是新版设置定时器函数，相比旧版`run_time`，新版`schedule_run`新增了`任务分组`,`任务取消`等多种功能

**原型:**

python

```
ContextInfo.schedule_run(
    func:Callable, # 回调函数，到达定时器预定时间时触发调用，参数为ContextInfo类型，无需返回值
    time_point:Union[dt.datetime,str], # 表示预定的第一次触发时间，如果设置定时器时已经过了预定时间，会立即执行func以及后续逻辑；当使用str类型时，格式为'yyyymmddHHMMSS'如'20231231235959'，需要满足转换dt.datetime.strptime('20231231235959','%Y%m%d%H%M%S')
    repeat_times:int=0, # 表示在预定时间触发后按interval间隔再触发多少次
    interval:datetime.timedelta=None, # 表示预定时间触发后的后续重复执行的时间间隔
    name:str='' # 定时器任务组名，可用于定时器分组，多次设置同名定时任务不会互相覆盖，会计入同一个任务组，按任务组名取消时会全部取消
    )
```

**参数：**

| 名称             | 类型                             | 描述                                                                                                                                                                 |
| -------------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `func`         | `Callable`                     | 回调函数，到达定时器预定时间时触发调用，参数为ContextInfo类型，无需返回值，定义示例如下：<br>def on_timer(C:ContextInfo): pass                                                                            |
| `time_point`   | `Union[datetime.datetime,str]` | 表示预定的第一次触发时间，如果设置定时器时已经过了预定时间，会立即执行func以及后续逻辑；<br>当使用str类型时，格式为'yyyymmddHHMMSS'如'20231231235959'，需要满足转换datetime.datetime.strptime('20231231235959','%Y%m%d%H%M%S') |
| `repeat_times` | `int`                          | 表示在预定时间触发后按interval间隔再触发多少次，传`-1`表示不限制次数                                                                                                                           |
| `interval`     | `datetime.timedelta`           | 表示预定时间触发后的后续重复执行的时间间隔                                                                                                                                              |
| `name`         | `str`                          | 定时器任务组名，可用于定时器分组，多次设置同名定时任务不会互相覆盖，会计入同一个任务组，按任务组名取消时会全部取消                                                                                                          |

**回调函数参数：** ContextInfo：策略模型全局对象

**返回值：**

`int`类型，表示本次调用后生成的定时任务号，可用于取消本次定时任务，全局唯一不重复

**示例：**

python

```
import datetime as dt
def on_timer(C:ContextInfo):
    print('hello world')
def init(ContextInfo):
    tid=ContextInfo.schedule_run(on_timer,'20231231235959',-1,dt.timedelta(minutes=1),'my_timer')
def handlebar(ContextInfo):
    pass
#此例为自2023-12-31 23:59:59后每60s运行一次on_timer
```

### ContextInfo.cancel_schedule_run - 取消由schedule_run产生的定时任务

**原型：**

python

```
ContextInfo.cancel_schedule_run(
    key:Union[seq:int,name:str] # 定时任务号或定时任务组名称
    )
```

**参数：**

| 名称     | 类型                        | 描述                                            |
| ------ | ------------------------- | --------------------------------------------- |
| `key:` | `Union[seq:int,name:str]` | 类型为int时，表示按任务号取消;类型为str时，表示按任务组取消，会取消组内所有定时任务 |

**返回值：**

`bool`类型，表示是否取消成功，即是否能按key找到目标定时任务

**示例：**

示例

```
ContextInfo.cancel_schedule_run('my_timer') #取消my_timer任务组所有定时任务
ContextInfo.cancel_schedule_run(1) #取消任务号为1的定时任务
```

### ContextInfo.run_time - 设置定时器

设置定时器函数，可以指定时间间隔，定时触发用户定义的回调函数。适用与在盘中，持续判断交易信号的模型。

**用法：** `ContextInfo.run_time(funcName,period,startTime)` 定时触发指定的 funcName函数, funcName函数由用户定义, 入参为ContextInfo对象。

**参数：**

- funcName：回调函数名
- period：重复调用的时间间隔,'5nSecond'表示每5秒运行1次回调函数,'5nDay'表示每5天运行一次回调函数,'500nMilliSecond'表示每500毫秒运行1次回调函数
- startTime：表示定时器第一次启动的时间,如果要定时器立刻启动,可以设置历史的时间

**回调函数参数：** ContextInfo：策略模型全局对象

**示例：**

python

```
import time
def init(ContextInfo):
    ContextInfo.run_time("f","5nSecond","2019-10-14 13:20:00")
def f(ContextInfo):
    print('hello world')

#此例为自2019-10-14 13:20:00后每5s运行一次函数f
```

注意

1. 模型回测时无效
2. 定时器没有结束方法，会随着策略的结束而结束。
3. period有nMilliSecond、nSecond和Day三个周期单元，部分周期下定时器函数在第一次运行之前会先等待一个period

### stop - 停止处理函数

**系统函数 不可被手动调用**

**释义：** PY策略模型关闭停止前运行到的函数，复杂策略模型，如中间有起线程可通过在该函数内实现停止线程操作。注意, 当前版本stop函数被调用时交易连接已断开, 不能在stop函数中做报单 / 撤单操作.

**参数：**

| 名称            | 类型       | 描述                      |
| ------------- | -------- | ----------------------- |
| `ContextInfo` | `object` | 策略运行环境对象，可以用于存储自定义的全局变量 |

**示例：**

python

```
def stop(ContextInfo):
    print( 'strategy is stop !')
```

### ContextInfo.is_last_bar - 是否为最后一根K线

**用法：** ContextInfo.is_last_bar()

**释义：** 判定是否为最后一根 K 线

**参数：** 无

**返回：** bool，返回值含义：True 是右侧最新k线 False不是最新k线

> True：是
> 
> False：否

**示例：**

pythonresult

```
def handlebar(ContextInfo):
    print(ContextInfo.is_last_bar())
```

### ContextInfo.is_new_bar - 判定是否为新的 K 线

**用法：** ContextInfo.is_new_bar()

**释义：** 某根 K 线的第一个 tick 数据到来时，判定该 K 线为新的 K 线，其后的tick不会认为是新的 K 线

**参数：** 无

**返回：** bool，返回值含义：

> True：是
> 
> False：否

**示例：**

pythonresult

```
def handlebar(ContextInfo):
    print(ContextInfo.is_new_bar()) #历史k线每根都是新k线 盘中 每根新k线第一个分笔返回True 其他分笔返回False
```

### ContextInfo.get_stock_name - 根据代码获取名称

注意

我们计划后续版本抛弃这个函数，不建议继续使用，可以用ContextInfo.get_instrument_detail("stockcode")["InstrumentName"]来实现同样功能

**用法：** ContextInfo.get_stock_name('stockcode')

**释义：** 根据代码获取名称

**参数：** stockcode：股票代码，如'000001.SZ'，缺省值 ' ' 默认为当前图代码

**返回：** string（GBK编码）

**示例：**

示例返回值

```
def handlebar(ContextInfo):
    print(ContextInfo.get_stock_name('000001.SZ'))
```

### ContextInfo.get_open_date - 根据代码返回对应股票的上市时间

**用法：** ContextInfo.get_open_date('stockcode')

**释义：** 根据代码返回对应股票的上市时间

**参数：** stockcode：股票代码，如'000001.SZ'，缺省值 ' ' 默认为当前图代码

**返回：** number

**示例：**

pythonresult

```
def init(ContextInfo):
    print(ContextInfo.get_open_date('000001.SZ'))
```

### ContextInfo.set_output_index_property - 设定指标绘制的属性

**用法：** ContextInfo.set_output_index_property(index_name,draw_style=0,color='white',noaxis=False,nodraw=False,noshow=False)

**释义：** 设定指标绘制的属性，会最终覆盖掉指标对应的属性字段

**参数：**

- index_name:string,指标名称，不可缺省
- draw_style,同paint函数的drawstyle，可缺省默认为0
- color,同paint函数的color，可缺省默认为'white'
- noaxis:bool,是否无坐标，可缺省默认为False
- nodraw:bool,是否不画线，可缺省默认为False
- noshow:bool,是否不展示，可缺省默认为False

**返回：** 无

**示例：**

pythonpythonresult

```
def init(ContextInfo):
    ContextInfo.set_output_index_property('单位净值', nodraw = True)#使回测指标'单位净值'不画线
```

### create_sector - 创建板块

**用法：** create_sector(parent_node,sector_name,overwrite)

**释义：** 创建板块

**参数：**

- parent_node：str，父节点，''为'我的'（默认目录）
- sector_name：str，要创建的板块名
- overwrite：bool，是否覆盖。如果目标节点已存在，为True时跳过，为False时在sector_name后增加数字编号，编号为从1开始自增的第一个不重复的值。

**返回：** sector_name2：实际创建的板块名

**示例：**

### create_sector_folder - 创建板块目录节点

**用法：** create_sector_folder(parent_node,folder_name,overwrite)

**释义：** 创建板块目录节点

**参数：**

- parent_node：str，父节点，''为'我的'（默认目录）
- sector_name：str，要创建的节点名
- overwrite：bool，是否覆盖。如果目标节点已存在，为True时跳过，为False时在folder_name后增加数字编号，编号为从1开始自增的第一个不重复的值。

**返回：** sector_name2：实际创建的节点名

**示例：**

pythonresult

```
folder=create_sector_folder('我的','新建分类',False)
```

### get_sector_list - 获取板块目录信息

**用法：** get_sector_list(node)

**释义：** 获取板块目录信息

**参数：**

- node：str，板块节点名，''为顶层目录

**返回：** info_list：[[s1,s2,...],[f1,f2,...]]s为板块名，f为目录节点名，例如[['我的自选'],['新建分类1']]

**示例：**

pythonresult

```
get_sector_list('我的')
```

### reset_sector_stock_list - 设置板块成分股

**用法：** reset_sector_stock_list(sector,stock_list)

**释义：** 设置板块成分股

**参数：**

- sector：板块名
- stock_list：list，品种代码列表，例如['000001.SZ','600000.SH']

**返回：** result：bool，操作成功为True，失败为False

**示例：**

pythonresult

```
reset_sector_stock_list('我的自选',['000001.SZ','600000.SH'])
```

### remove_stock_from_sector - 移除板块成分股

**用法：** remove_stock_from_sector(sector,stock_code)

**释义：** 移除板块成分股

**参数：**

- sector：板块名
- stock_code：品种代码，例如'000001.SZ'

**返回：** result：bool，操作成功为True，失败为False

**示例：**

pythonresult

```
remove_stock_from_sector('我的自选','000001.SZ')
```

### add_stock_to_sector - 添加板块成分股

**用法：** add_stock_to_sector(sector,stock_code)

**释义：** 添加板块成分股

**参数：**

- sector：板块名
- stock_code：品种代码，例如'000001.SZ'

**返回：** result：bool，操作成功为True，失败为False

**示例：**

pythonresult

```
add_stock_to_sector('我的自选','000001.SZ')
```

上次更新: 2025/3/13 18:36:49

## 数据下载

### download_history_data - 下载指定合约代码指定周期对应时间范围的行情数据

提示

QMT提供的行情数据中，基础周期包含 tick 1m 5m 1d，这些是实际用于存储的周期 其他周期为合成周期，以基础周期合成得到

合成周期

- 3m， 由1m线合成
- 10m, 15m, 30m, 60m, 2h, 3h, 4h 由5分钟线合成
- 2d(2日线), 3d(3日线), 5d(5日线), 1w（周线）, 1mon（月线）, 1q(季线), 1hy(半年线), 1y（年线） 由日线数据合成

获取合成周期时

- 如果取历史，需要下载历史的基础周期（如取15m需要下载5m）
- 如果取实时，可以直接订阅原始周期（如直接订阅15m）

如果同时用到基础周期和合成周期，只需要下载基础周期,例如同时使用5m和15m，因为15m也是由5m合成，所以只需要下载一次5m的数据即可

**原型**

内置python

```
download_history_data(stockcode,period,startTime,endTime)
```

**释义**

下载指定合约代码指定周期对应时间范围的行情数据

**参数**

| 字段名             | 数据类型     | 解释                                                                   |
| --------------- | -------- | -------------------------------------------------------------------- |
| `stockcode`     | `string` | 股票代码，格式为'stkcode.market'，例如 '600000.SH'                              |
| `period`        | `string` | K线周期类型，包括:<br>`'tick'`：分笔线<br>`'1d'`：日线<br>`'1m'`：分钟线<br>`'5m'`：5分钟线 |
| `startTime`     | `string` | 起始时间，格式为 "20200101" 或 "20200101093000"，可以为空                          |
| `endTime`       | `string` | 结束时间，格式为 "20200101" 或 "20200101093000"，可以为空                          |
| `incrementally` | `bool`   | 默认为 `None` 是否从本地最后一条数据往后增量下载，部分版本客户端可能不支持此参数                         |

**返回值**

`none`

**示例**

示例

```
# coding:gbk
def init(C):
    download_history_data("000001.SZ","1d","20230101","") # 下载000001.SZ,从20230101至今的日线数据
    download_history_data("000001.SZ","1d","20230101","",incrementally=True) # 下载000001.SZ,从20230101至今的日线数据,增量下载

def handlebar(C):
    return
```

界面端进行数据下载还可参考:

提示

【攻略】K线/财务数据下载方式 https://www.xuntou.net/forum.php?mod=viewthread&tid=1354&user_code=7zqjlm 来自: 迅投QMT社区

## 获取行情数据

该目录下的函数用于获取实时行情,历史行情

### ContextInfo.get_market_data_ex - 获取行情数据

注意

1. **该函数不建议在`init`中运行,在`init`中运行时仅能取到本地数据**
2. 关于获取行情函数之间的区别与注意事项可在 - [常见问题-行情相关](https://dict.thinktrader.net/innerApi/question_answer.html#%E8%A1%8C%E6%83%85%E7%9B%B8%E5%85%B3) 查看
3. 除实时行情外，该函数还可用于获取特色数据，如`资金流向数据`,`订单流数据`等，获取方式见[数据字典](https://dict.thinktrader.net/dictionary/)

**原型**

内置python

```
ContextInfo.get_market_data_ex(
    fields=[], 
    stock_code=[], 
    period='follow', 
    start_time='', 
    end_time='', 
    count=-1, 
    dividend_type='follow', 
    fill_data=True, 
    subscribe=True)
```

**释义**

获取实时行情与历史行情数据

**参数**

| 名称              | 类型     | 描述                                                                                                                                                                                                                                                                                                                                                                      |
| --------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`         | `list` | `数据字段，详情见下方field字段表`                                                                                                                                                                                                                                                                                                                                                    |
| `stock_list`    | `list` | `合约代码列表`                                                                                                                                                                                                                                                                                                                                                                |
| `period`        | `str`  | `数据周期，可选字段为:`<br>`"tick"`<br>`"1m"：1分钟线`<br>`"5m"：5分钟线；"15m"：15分钟线；"30m"：30分钟线`<br>`"1h"小时线`<br>`"1d"：日线`<br>`"1w"：周线`<br>`"1mon"：月线`<br>`"1q"：季线`<br>`"1hy"：半年线`<br>`"1y"：年线`<br>`'l2quote'：Level2行情快照`<br>`'l2quoteaux'：Level2行情快照补充`<br>`'l2order'：Level2逐笔委托`<br>`'l2transaction'：Level2逐笔成交`<br>`'l2transactioncount'：Level2大单统计`<br>`'l2orderqueue'：Level2委买委卖队列` |
| `start_time`    | `str`  | `数据起始时间，格式为 %Y%m%d 或 %Y%m%d%H%M%S，填""为获取历史最早一天`                                                                                                                                                                                                                                                                                                                         |
| `end_time`      | `str`  | `数据结束时间，格式为 %Y%m%d 或 %Y%m%d%H%M%S ，填""为截止到最新一天`                                                                                                                                                                                                                                                                                                                         |
| `count`         | `int`  | `数据个数`                                                                                                                                                                                                                                                                                                                                                                  |
| `dividend_type` | `str`  | `除权方式,可选值为`<br>`'none'：不复权`<br>`'front':前复权`<br>`'back':后复权`<br>`'front_ratio': 等比前复权`<br>`'back_ratio': 等比后复权`                                                                                                                                                                                                                                                         |
| `fill_data`     | `bool` | `是否填充数据`                                                                                                                                                                                                                                                                                                                                                                |
| `subscribe`     | `bool` | `订阅数据开关，默认为True，设置为False时不做数据订阅，只读取本地已有数据。`                                                                                                                                                                                                                                                                                                                             |

- `field`字段可选：

| field          | 数据类型    | 含义             |
| -------------- | ------- | -------------- |
| `time`         | `int`   | `时间`           |
| `open`         | `float` | `开盘价`          |
| `high`         | `float` | `最高价`          |
| `low`          | `float` | `最低价`          |
| `close`        | `float` | `收盘价`          |
| `volume`       | `float` | `成交量`          |
| `amount`       | `float` | `成交额`          |
| `settle`       | `float` | `今结算`          |
| `openInterest` | `float` | `持仓量`          |
| `preClose`     | `float` | `前收盘价`         |
| `suspendFlag`  | `int`   | `停牌` 1停牌，0 不停牌 |

- `period`周期为tick时，`field`字段可选:

| field          | 数据类型    | 含义             |
| -------------- | ------- | -------------- |
| `time`         | `int`   | `时间`           |
| `lastPrice`    | `float` | `最新价`          |
| `lastClose`    | `float` | `前收盘价`         |
| `open`         | `float` | `开盘价`          |
| `high`         | `float` | `最高价`          |
| `low`          | `float` | `最低价`          |
| `close`        | `float` | `收盘价`          |
| `volume`       | `float` | `成交量`          |
| `amount`       | `float` | `成交额`          |
| `settle`       | `float` | `今结算`          |
| `openInterest` | `float` | `持仓量`          |
| `stockStatus`  | `int`   | `停牌` 1停牌，0 不停牌 |

- `period`周期为Level2数据时，字段参考[数据结构](https://dict.thinktrader.net/innerApi/data_structure.html#l2quote-level2%E8%A1%8C%E6%83%85%E5%BF%AB%E7%85%A7)

**返回值**

- 返回dict { stock_code1 : value1, stock_code2 : value2, ... }
- value1, value2, ... ：pd.DataFrame 数据集，index为time_list，columns为fields,可参考[Bar字段](https://dict.thinktrader.net/innerApi/data_structure.html#bar-bar%E5%AF%B9%E8%B1%A1)
- 各标的对应的DataFrame维度相同、索引相同

**示例**

示例data1返回值data2返回值data3返回值data4返回值历史tick期货五档盘口

```
# coding:gbk
import pandas as pd
import numpy as np

def init(C):    
    C.stock_list = ["000001.SZ","600519.SH", "510050.SH"]# 指定获取的标的
    C.start_time = "20230901"# 指定获取数据的开始时间
    C.end_time = "20231101"# 指定获取数据的结束时间

def handlebar(C):
    # 获取多只股票，多个字段，一条数据
    data1 = C.get_market_data_ex([],C.stock_list, period = "1d",count = 1)
    # 获取多只股票，多个字段，指定时间数据
    data2 = C.get_market_data_ex([],C.stock_list, period = "1d", start_time = C.start_time, end_time = C.end_time)
    # 获取多只股票，多个字段，指定时间15m数据
    data3 = C.get_market_data_ex([],C.stock_list, period = "15m", start_time = C.start_time, end_time = C.end_time)
    # 获取多只股票，指定字段，指定时间15m数据
    data4 = C.get_market_data_ex(["close","open"],C.stock_list, period = "15m", start_time = C.start_time, end_time = C.end_time)
    # 获取多只股票，历史tick
    tick = C.get_market_data_ex([],C.stock_list, period = "tick", start_time = C.start_time, end_time = C.end_time)
    # 获取期货5档盘口tick
    future_lv2_quote = C.get_market_data_ex([],["rb2405.SF","ec2404.INE"], period = "l2quote", count = 1)
    print(data1)
    print(data2["000001.SZ"].tail())
    print(data3)
    print(data4["000001.SZ"])
    print(data4["000001.SZ"].to_csv("your_path")) # 导出文件为csv格式，路径填本机路径
    print(tick["000001.SZ"])
    print(future_lv2_quote)
```

### ContextInfo.get_full_tick - 获取全推数据

提示

不能用于回测 只能取最新的分笔，不能取历史分笔

**原型**

内置python

```
ContextInfo.get_full_tick(stock_code=[])
```

**释义**

获取最新分笔数据

**参数**

| 名称           | 类型          | 描述                                               |
| ------------ | ----------- | ------------------------------------------------ |
| `stock_code` | `list[str]` | `合约代码列表，如['600000.SH','600036.SH']，不指定时为当前主图合约。` |

**返回值** 根据stock_code返回一个dict，该字典的key值是股票代码，其值仍然是一个dict，在该dict中存放股票代码对应的最新的数据。该字典数据key值参考[tick字段](https://dict.thinktrader.net/innerApi/data_structure.html#tick-tick-%E5%AF%B9%E8%B1%A1)

**示例**

示例返回值

```
# coding:gbk
import pandas as pd
import numpy as np

def init(C):
    C.stock_list = ["000001.SZ","600519.SH", "510050.SH"]

def handlebar(C):
    tick = C.get_full_tick(C.stock_list)
    print(tick["510050.SH"])
```

### ContextInfo.subscribe_quote - 订阅行情数据

提示

1. 该函数属于订阅函数，非VIP用户限制订阅数量

2. VIP用户支持全推市场指定周期K线

3. VIP用户权限请参考[vip-行情用户优势对比](https://dict.thinktrader.net/dictionary/#vip-%E8%A1%8C%E6%83%85%E7%94%A8%E6%88%B7%E4%BC%98%E5%8A%BF%E5%AF%B9%E6%AF%94)

**原型**

内置python

```
ContextInfo.subscribe_quote(
    stock_code,
    period='follow',
    dividend_type='follow',
    result_type='',
    callback=None)
```

**释义**

订阅行情数据,关于订阅机制请参考[运行机制对比](https://dict.thinktrader.net/innerApi/start_now.html#%E4%B8%89%E3%80%81%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6%E5%AF%B9%E6%AF%94)

**参数**

| 字段名             | 数据类型       | 解释                                                                                                                                                                                                              |
| --------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `stockcode`     | `string`   | `股票代码，'stkcode.market'，如'600000.SH'`                                                                                                                                                                            |
| `period`        | `string`   | `K线周期类型`                                                                                                                                                                                                        |
| `dividend_type` | `string`   | `除权方式,可选值为`<br>`'none'：不复权`<br>`'front':前复权`<br>`'back':后复权`<br>`'front_ratio': 等比前复权`<br>`'back_ratio': 等比后复权`<br>`注意：分笔周期返回数据均为不复权`                                                                           |
| `result_type`   | `string`   | `返回数据格式,可选范围：<br>'DataFrame'或''（默认）：返回{code:data}，data为pd.DataFrame数据集，index为字符串格式的时间序列，columns为数据字段<br>'dict'：返回{code:{k1:v1,k2:v2,...}}，k为数据字段名，v为字段值<br>'list'：返回{code:{k1:[v1],k2:[v2],...}}，k为数据字段名，v为字段值` |
| `callback`      | `function` | `指定推送行情的回调函数`                                                                                                                                                                                                   |

**返回值**

`int`：订阅号，用于反订阅

**示例**

示例返回值

```
# conding = gbk
def call_back(data):
    print(data)

def init(C):
    C.subID = C.subscribe_quote("000001.SZ","1d", callback = call_back)
def handlebar(C):
    print("============================")
    print("C.subID: ",C.subID)
```

### ContextInfo.subscribe_whole_quote - 订阅全推数据

提示

内置python

```
ContextInfo.subscribe_whole_quote(code_list,callback=None)
```

**释义**

订阅全推数据，全推数据只有分笔周期，每次增量推送数据有变化的品种

**参数**

| 字段名         | 数据类型            | 解释                                                         |
| ----------- | --------------- | ---------------------------------------------------------- |
| `code_list` | `list[str,...]` | `市场代码列表/品种代码列表,如 ['SH','SZ'] 或 ['600000.SH', '000001.SZ']` |
| `callback`  | `function`      | `数据推送回调`                                                   |

**返回值**`int`，订阅号，可用`ContextInfo.unsubscribe_quote`做反订阅

示例返回值

```
# conding = gbk
def call_back(data):
    print(data)

def init(C):
    C.stock_list = ["000001.SZ","600519.SH", "510050.SH"]
    C.subID = C.subscribe_whole_quote(C.stock_list,callback=call_back)
def handlebar(C):
    print("============================")
    print("C.subID: ",C.subID)
```

### ContextInfo.unsubscribe_quote - 反订阅行情数据

**原型**

内置python

```
ContextInfo.unsubscribe_quote(subId)
```

**释义**

反订阅行情数据，配合`ContextInfo.subscribe_quote()`或`ContextInfo.subscribe_whole_quote()`使用

**参数**

| 字段名     | 数据类型  | 解释           |
| ------- | ----- | ------------ |
| `subId` | `int` | `行情订阅返回的订阅号` |

**示例**

示例

```
# conding = gbk
def call_back(data):
    print(data)
def init(C):
    C.stock_list = ["000001.SZ","600519.SH", "510050.SH"]
    C.subID = C.subscribe_whole_quote(C.stock_list,callback=call_back)

def handlebar(C):
    print("============================")
    print("C.subID: ",C.subID)
    if C.subID > 0:
        C.unsubscribe_quote(C.subID) # 取消行情订阅
```

### subscribe_formula - 订阅模型

**原型**

内置python

```
subscribe_formula(
   formula_name,stock_code,period
   ,start_time="",end_time="",count=-1
   ,dividend_type="none"
   ,extend_param={}
   ,callback=None)
```

**释义** 订阅vba模型运行结果，使用前要注意补充本地K线数据或分笔数据

**参数**

| 字段名           | 类型   | 描述                                                                                                                                       |
| ------------- | ---- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| formula_name  | str  | 模型名称名                                                                                                                                    |
| stock_code    | str  | 模型主图代码形式如'stkcode.market'，如'000300.SH'                                                                                                   |
| period        | str  | K线周期类型，可选范围：'tick':分笔线，'1d':日线，'1m':分钟线，'3m':三分钟线，'5m':5分钟线，'15m':15分钟线，'30m':30分钟线，'1h':小时线，'1w':周线，'1mon':月线，'1q':季线，'1hy':半年线，'1y':年线 |
| start_time    | str  | 模型运行起始时间，形如:'20200101'，默认为空视为最早                                                                                                          |
| end_time      | str  | 模型运行截止时间，形如:'20200101'，默认为空视为最新                                                                                                          |
| count         | int  | 模型运行范围为向前 count 根 bar，默认为 -1 运行所有 bar                                                                                                    |
| dividend_type | str  | 复权方式，默认为主图除权方式，可选范围：'none':不复权，'front':向前复权，'back':向后复权，'front_ratio':等比向前复权，'back_ratio':等比向后复权                                         |
| extend_param  | dict | 模型的入参，形如 {'a': 1, '__basket': {}}                                                                                                        |
| __basket      | dict | 可选参数，组合模型的股票池权重，形如 {'600000.SH': 0.06, '000001.SZ': 0.01}                                                                                |

**返回值** 分两块，

- subscribe_formula返回模型的订阅号,可用于后续反订阅，失败返回 -1

- callback:
  
  - timelist： 数据时间戳
  - outputs：模型的输出值，结构为{变量名:值}

**示例**

示例

```
#encoding=gbk
def callback(data):
    print(data)

def init(ContextInfo):
    basket={
       '600000.SH':0.06,
       '000001.SZ':0.01
      }
    argsDict={'a':100,'__basket':basket}
    subID=subscribe_formula(
      '单股模型示范','000300.SH','1d',
      '20240101','20240201',-1,
      "none",
      argsDict,
      callback
   )
```

### unsubscribe_formula - 反订阅模型

**原型**

内置python

```
unsubscribe_formula(subID)
```

**释义** 反订阅模型

**参数**

| 字段名   | 类型  | 描述    |
| ----- | --- | ----- |
| subID | int | 模型订阅号 |

**返回值**

- bool:反订阅成功为True，失败为False

**示例**

示例

```
#encoding=gbk
def callback(data):
    print(data)

def init(ContextInfo):
    basket={
       '600000.SH':0.06,
       '000001.SZ':0.01
      }
    argsDict={'a':100,'__basket':basket}
    subID=subscribe_formula(
      '单股模型示范','000300.SH','1d',
      '20240101','20240201',-1,
      "none",
      argsDict,
      callback
   )

    unsubscribe_formula(subID)
```

### call_formula - 调用模型

**原型**

内置python

```
call_formula(formula_name,stock_code,period,start_time="",end_time="",count=-1,dividend_type="none",extend_param={})
```

**释义** 获取vba模型运行结果，使用前要注意补充本地K线数据或分笔数据

**参数**

| 字段名           | 类型   | 描述                                                                                                                                                                                 |
| ------------- | ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| formula_name  | str  | 模型名称名                                                                                                                                                                              |
| stock_code    | str  | 模型主图代码形式如'stkcode.market'，如'000300.SH'                                                                                                                                             |
| period        | str  | K线周期类型，可选范围：'tick':分笔线，'1d':日线，'1m':分钟线，'3m':三分钟线，'5m':5分钟线，'15m':15分钟线，'30m':30分钟线，'1h':小时线，'1w':周线，'1mon':月线，'1q':季线，'1hy':半年线，'1y':年线                                           |
| start_time    | str  | 模型运行起始时间，形如:'20200101'，默认为空视为最早                                                                                                                                                    |
| end_time      | str  | 模型运行截止时间，形如:'20200101'，默认为空视为最新                                                                                                                                                    |
| count         | int  | 模型运行范围为向前 count 根 bar，默认为 -1 运行所有 bar                                                                                                                                              |
| dividend_type | str  | 复权方式，默认为主图除权方式，可选范围：'none':不复权，'front':向前复权，'back':向后复权，'front_ratio':等比向前复权，'back_ratio':等比向后复权                                                                                   |
| extend_param  | dict | 模型的入参,{"模型名:参数名":参数值},例如在跑模型MA时，{'MA:n1':1};入参可以添加__basket:dict,组合模型的股票池权重,形如{'__basket':{'600000.SH':0.06,'000001.SZ':0.01}}，如果在跑一个模型1的时候，模型1调用了模型2，如果只想修改模型2的参数可以传{'模型2:参数':参数值} |

**返回值** 返回：dict{ 'dbt':0,#返回数据类型，0:全部历史数据 'timelist':[...],#返回数据时间范围list, 'outputs':{'var1':[...],'var2':[...]}#输出变量名：变量值list }

**示例**

示例

```
def handlebar(ContextInfo):
    basket={'600000.SH':0.06,'000001.SZ':0.01}
    argsDict={'a':100,'__basket':basket}
    modelRet=call_formula('单股模型示范','000300.SH','1d','20240101','20240201',-1,"none",argsDict)
    print(modelRet)
```

### call_formula_batch - 批量调用模型

**原型**

内置python

```
call_formula_batch(formula_names,stock_codes,period,start_time="",end_time="",count=-1,dividend_type="none",extend_params=[])
```

**释义** 批量获取vba模型运行结果，使用前要注意补充本地K线数据或分笔数据

**参数**

| 字段名           | 类型   | 描述                                                                                                                                                                                       |
| ------------- | ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| formula_names | list | 包含要批量运行的模型名                                                                                                                                                                              |
| stock_codes   | list | 包含要批量运行的模型主图代码形式'stkcode.market'，如'000300.SH'                                                                                                                                            |
| period        | str  | K线周期类型，可选范围：'tick':分笔线，'1d':日线，'1m':分钟线，'3m':三分钟线，'5m':5分钟线，'15m':15分钟线，'30m':30分钟线，'1h':小时线，'1w':周线，'1mon':月线，'1q':季线，'1hy':半年线，'1y':年线                                                 |
| start_time    | str  | 模型运行起始时间，形如:'20200101'，默认为空视为最早                                                                                                                                                          |
| end_time      | str  | 模型运行截止时间，形如:'20200101'，默认为空视为最新                                                                                                                                                          |
| count         | int  | 模型运行范围为向前 count 根 bar，默认为 -1 运行所有 bar                                                                                                                                                    |
| dividend_type | str  | 复权方式，默认为主图除权方式，可选范围：'none':不复权，'front':向前复权，'back':向后复权，'front_ratio':等比向前复权，'back_ratio':等比向后复权                                                                                         |
| extend_params | list | 包含每个模型的入参,[{"模型名:参数名":参数值}],例如在跑模型MA时，{'MA:n1':1};入参可以添加__basket:dict,组合模型的股票池权重,形如{'__basket':{'600000.SH':0.06,'000001.SZ':0.01}}，如果在跑一个模型1的时候，模型1调用了模型2，如果只想修改模型2的参数可以传{'模型2:参数':参数值} |

**返回值**

- list[dict]
  - dict说明:
    - formula:模型名
    - stock:品种代码
    - argument:参数
    - result:dict参考call_formula返回结果

**示例**

示例

```
def handlebar(ContextInfo):
    formulas=['testModel1','testModel2']
    codes=['600000.SH','000001.SZ']
    basket={'600000.SH':0.06,'000001.SZ':0.01}
    args=[{'a':100,'__basket':basket},{'a':200,'__basket':basket}]
    modelRet=call_formula_batch(formulas,codes,'1d',extend_params=args);
    print(modelRet)
```

### ContextInfo.get_svol - 根据代码获取对应股票的内盘成交量

**原型**

内置python

```
ContextInfo.get_svol(stockcode)
```

**释义**

根据代码获取对应股票的内盘成交量

**参数**

| 字段名         | 数据类型     | 解释                                |
| ----------- | -------- | --------------------------------- |
| `stockcode` | `string` | 股票代码，如 '000001.SZ'，缺省值''，默认为当前图代码 |

**返回值**`int`:内盘成交量

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    data = C.get_svol('000001.SZ')
    print(data)
```

### ContextInfo.get_bvol - 根据代码获取对应股票的外盘成交量

**原型**

内置python

```
ContextInfo.get_bvol(stockcode)
```

**释义**

根据代码获取对应股票的外盘成交量

**参数**

| 字段名         | 数据类型     | 解释                                |
| ----------- | -------- | --------------------------------- |
| `stockcode` | `string` | 股票代码，如 '000001.SZ'，缺省值''，默认为当前图代码 |

**返回值**

`int`:外盘成交量

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    data = C.get_bvol('000001.SZ')
    print(data)
```

### ContextInfo.get_turnover_rate - 获取换手率

提示

使用之前需要下载财务数据(在财务数据下载中)以及日线数据

如果不补充股本数据,将使用最新流通股本计算历史换手率,可能会造成历史换手率不正确

**原型**

内置python

```
ContextInfo.get_turnover_rate(stock_list,startTime,endTime)
```

**释义**

获取换手率

**参数**

| 字段名          | 数据类型     | 解释                              |
| ------------ | -------- | ------------------------------- |
| `stock_list` | `list`   | 股票列表，如['600000.SH','000001.SZ'] |
| `startTime`  | `string` | 起始时间，如'20170101'                |
| `endTime`    | `string` | 结束时间，如'20180101'                |

**返回值**

`pandas.Dataframe`

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    data = C.get_turnover_rate(['000002.SZ'],'20170101','20170301')
    print(data)
```

### ContextInfo.get_longhubang - 获取龙虎榜数据

**原型**

内置python

```
ContextInfo.get_longhubang(stock_list, startTime, endTime)
```

**释义**

获取龙虎榜数据

**参数**

| 参数名称         | 类型     | 描述                                |
| ------------ | ------ | --------------------------------- |
| `stock_list` | `list` | 股票列表，如 ['600000.SH', '600036.SH'] |
| `startTime`  | `str`  | 起始时间，如 '20170101'                 |
| `endTime`    | `str`  | 结束时间，如 '20180101'                 |

**返回值**

- 格式为`pandas.DataFrame`:

| 参数名称              | 数据类型               | 描述   |
| ----------------- | ------------------ | ---- |
| `reason`          | `str`              | 上榜原因 |
| `close`           | `float`            | 收盘价  |
| `spreadRate`      | `float`            | 涨跌幅  |
| `TurnoverVolune`  | `float`            | 成交量  |
| `Turnover_Amount` | `float`            | 成交金额 |
| `buyTraderBooth`  | `pandas.DataFrame` | 买方席位 |
| `sellTraderBooth` | `pandas.DataFrame` | 卖方席位 |

- `buyTraderBooth` 或 `sellTraderBooth` 包含字段：

| 参数名称          | 数据类型    | 描述         |
| ------------- | ------- | ---------- |
| `traderName`  | `str`   | 交易营业部名称    |
| `buyAmount`   | `float` | 买入金额       |
| `buyPercent`  | `float` | 买入金额占总成交占比 |
| `sellAmount`  | `float` | 卖出金额       |
| `sellPercent` | `float` | 卖出金额占总成交占比 |
| `totalAmount` | `float` | 该席位总成交金额   |
| `rank`        | `int`   | 席位排行       |
| `direction`   | `int`   | 买卖方向       |

**示例**

示例返回值

```
# coding:gbk

def init(C):
    return

def handlebar(C):
    print(C.get_longhubang(['000002.SZ'],'20100101','20180101'))
```

### ContextInfo.get_north_finance_change - 获取对应周期的北向数据

**原型**

内置python

```
ContextInfo.get_north_finance_change(period)
```

**释义**

获取对应周期的北向数据

**参数**

| 字段名      | 数据类型  | 描述   |
| -------- | ----- | ---- |
| `period` | `str` | 数据周期 |

**返回值**

- 根据`period`返回一个`dict`，该字典的`key`值是北向数据的时间戳，其值仍然是一个`dict`，其值的`key`值是北向数据的字段类型，其值是对应字段的值。该字典数据`key`值有：

| 字段名                  | 数据类型 | 描述          |
| -------------------- | ---- | ----------- |
| hgtNorthBuyMoney     | int  | HGT北向买入资金   |
| hgtNorthSellMoney    | int  | HGT北向卖出资金   |
| hgtSouthBuyMoney     | int  | HGT南向买入资金   |
| hgtSouthSellMoney    | int  | HGT南向卖出资金   |
| sgtNorthBuyMoney     | int  | SGT北向买入资金   |
| sgtNorthSellMoney    | int  | SGT北向卖出资金   |
| sgtSouthBuyMoney     | int  | SGT南向买入资金   |
| sgtSouthSellMoney    | int  | SGT南向卖出资金   |
| hgtNorthNetInFlow    | int  | HGT北向资金净流入  |
| hgtNorthBalanceByDay | int  | HGT北向当日资金余额 |
| hgtSouthNetInFlow    | int  | HGT南向资金净流入  |
| hgtSouthBalanceByDay | int  | HGT南向当日资金余额 |
| sgtNorthNetInFlow    | int  | SGT北向资金净流入  |
| sgtNorthBalanceByDay | int  | SGT北向当日资金余额 |
| sgtSouthNetInFlow    | int  | SGT南向资金净流入  |
| sgtSouthBalanceByDay | int  | SGT南向当日资金余额 |

**示例：**

示例返回值

```
# coding = gbk
def init(C):
    return
# 获取市场北向数据
def handlebar(C):
    print(C.get_north_finance_change('1d'))
```

### ContextInfo.get_hkt_details - 获取指定品种的持股明细

**原型**

内置python

```
ContextInfo.get_hkt_details(stockcode)
```

**释义**

获取指定品种的持股明细

**参数**

| 参数名称        | 数据类型     | 描述                  |
| ----------- | -------- | ------------------- |
| `stockcode` | `string` | 必须是'stock.market'形式 |

**返回值**

- 根据`stockcode`返回一个`dict`，该字典的key值是北向持股明细数据的时间戳，其值仍然是一个`dict`，其值的`key`值是北向持股明细数据的字段类型，其值是对应字段的值，该字典数据`key`值有：

| 参数名称                   | 数据类型/单位 | 描述                |
| ---------------------- | ------- | ----------------- |
| `stockCode`            | `str`   | 股票代码              |
| `ownSharesCompany`     | `str`   | 机构名称              |
| `ownSharesAmount`      | `int`   | 持股数量              |
| `ownSharesMarketValue` | `float` | 持股市值              |
| `ownSharesRatio`       | `float` | 持股数量占比            |
| `ownSharesNetBuy`      | `float` | 净买入金额（当日持股-前一日持股） |

**示例：**

示例返回值

```
# coding = gbk
def init(C):
    return
def handlebar(C):
    data = C.get_hkt_details('600000.SH')
    print(data)
```

### ContextInfo.get_hkt_statistics - 获取指定品种的持股统计

**原型**

内置python

```
ContextInfo.get_hkt_statistics(stockcode)
```

**释义**

获取指定品种的持股统计

**参数**

| 字段名         | 数据类型     | 解释                  |
| ----------- | -------- | ------------------- |
| `stockcode` | `string` | 必须是'stock.market'形式 |

**返回值**

根据stockcode返回一个dict，该字典的key值是北向持股统计数据的时间戳，其值仍然是一个dict，其值的key值是北向持股统计数据的字段类型，其值是对应字段的值，该字典数据key值有：

| 字段名                    | 数据类型     | 解释                       |
| ---------------------- | -------- | ------------------------ |
| `stockCode`            | `string` | 股票代码                     |
| `ownSharesAmount`      | `float`  | 持股数量，单位：股                |
| `ownSharesMarketValue` | `float`  | 持股市值，单位：元                |
| `ownSharesRatio`       | `float`  | 持股数量占比，单位：%              |
| `ownSharesNetBuy`      | `float`  | 净买入，单位：元，浮点数（当日持股-前一日持股） |

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):

    print(C.get_hkt_statistics('600000.SH'))
```

### get_etf_info - 根据ETF基金代码获取ETF申赎清单及对应成分股数据

**原型**

内置python

```
get_etf_info(stockcode)
```

**释义**

根据ETF基金代码获取ETF申赎清单及对应成分股数据,每日盘前更新

**参数**

| 字段名         | 数据类型     | 解释                  |
| ----------- | -------- | ------------------- |
| `stockcode` | `string` | ETF基金代码如"510050.SH" |

**返回值**

一个多层嵌套的`dict`

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    d = get_etf_info("510050.SH")
    print(d)
```

### get_etf_iopv - 根据ETF基金代码获取ETF的基金份额参考净值

**原型**

内置python

```
get_etf_iopv(stockcode)
```

**释义**

根据ETF基金代码获取ETF的基金份额参考净值

**参数**

| 字段名         | 数据类型     | 解释                  |
| ----------- | -------- | ------------------- |
| `stockcode` | `string` | ETF基金代码如"510050.SH" |

**返回值**

`float`类型值,IOPV，基金份额参考净值

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    print(get_etf_iopv("510050.SH"))
```

### ContextInfo.get_local_data - 获取本地行情数据【不推荐】

注意

本函数用于仅用于获取本地历史行情数据，使用前请确保已通过[download_history_data](https://dict.thinktrader.net/innerApi/data_function.html#download-history-data-%E4%B8%8B%E8%BD%BD%E6%8C%87%E5%AE%9A%E5%90%88%E7%BA%A6%E4%BB%A3%E7%A0%81%E6%8C%87%E5%AE%9A%E5%91%A8%E6%9C%9F%E5%AF%B9%E5%BA%94%E6%97%B6%E9%97%B4%E8%8C%83%E5%9B%B4%E7%9A%84%E8%A1%8C%E6%83%85%E6%95%B0%E6%8D%AE)下载过历史行情数据

**原型**

内置python

```
ContextInfo.get_local_data(
    stock_code,
    start_time='',
    end_time='',
    period='1d',
    divid_type='none',
    count=-1)
```

**释义**

获取本地行情数据

**参数**

| 字段名             | 数据类型     | 解释                                                                                                                                                                                                                                                                 |
| --------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `stock_code`    | `string` | 默认参数，合约代码格式为 `code.market`，不指定时为当前图合约                                                                                                                                                                                                                              |
| `start_time`    | `string` | 默认参数，开始时间，格式为 '20171209' 或 '20171209010101'                                                                                                                                                                                                                        |
| `end_time`      | `string` | 默认参数，结束时间，格式同 `start_time`                                                                                                                                                                                                                                         |
| `period`        | `string` | 默认参数，K线类型，可选值包括：<br>`'tick'`：分笔线（只用于获取'quoter'字段数据）、`'realtime'`: 实时线、`'1d'`：日线<br>`'md'`：多日线、`'1m'`：1分钟线、`'3m'`：3分钟线<br>`'5m'`：5分钟线、`'15m'`：15分钟线、`'30m'`：30分钟线<br>`'mm'`：多分钟线、`'1h'`：小时线、`'mh'`：多小时线<br>`'1w'`：周线、`'1mon'`：月线、`'1q'`：季线<br>`'1hy'`：半年线、`'1y'`：年线 |
| `dividend_type` | `string` | 除复权种类，可选值：<br>`'none'`：不复权<br>`'front'`：向前复权<br>`'back'`：向后复权<br>`'front_ratio'`：等比向前复权<br>`'back_ratio'`：等比向后复权                                                                                                                                                   |
| `count`         | `int`    | 当 `count` 大于等于0时：<br>如果指定了 `start_time` 和 `end_time`，则以 `end_time` 为基准向前取 `count` 条数据；<br>如果 `start_time` 和 `end_time` 缺省，则默认取本地数据最新的 `count` 条数据；<br>如果 `start_time`、`end_time` 和 `count` 都缺省时，则默认取本地全部数据。                                                        |

**返回值**

返回一个`dict`，键值为timetag，value为另一个dict(valuedict)

- period='tick'时函数获取分笔数据，valuedict字典数据key值有：

| 字段                    | 数据类型    | 含义                                                                                                                                                   |
| --------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `lastPrice`           | `float` | `最新价`                                                                                                                                                |
| `open`                | `float` | `开盘价`                                                                                                                                                |
| `high`                | `float` | `最高价`                                                                                                                                                |
| `low`                 | `float` | `最低价`                                                                                                                                                |
| `lastClose`           | `float` | `前收盘价`                                                                                                                                               |
| `amount`              | `float` | `成交额`                                                                                                                                                |
| `volume`              | `float` | `成交量`                                                                                                                                                |
| `pvolume`             | `float` | `原始成交量`                                                                                                                                              |
| `stockStatus`         | `int`   | `作废 参考openInt`                                                                                                                                       |
| `openInt`             | `float` | `若是股票，则openInt含义为股票状态，非股票则是持仓量`[openInt字段说明](https://dict.thinktrader.net/innerApi/data_structure.html#openint-%E8%AF%81%E5%88%B8%E7%8A%B6%E6%80%81) |
| `lastSettlementPrice` | `float` | `昨结算价`                                                                                                                                               |
| `askPrice`            | `list`  | `委卖价`                                                                                                                                                |
| `bidPrice`            | `list`  | `委买价`                                                                                                                                                |
| `askVol`              | `list`  | `委卖量`                                                                                                                                                |
| `bidVol`              | `list`  | `委买量`                                                                                                                                                |
| `settlementPrice`     | `float` | `今结算价`                                                                                                                                               |

- period为其他值时，valuedict字典数据key值有：

| 字段名      | 数据类型    | 解释  |
| -------- | ------- | --- |
| `amount` | `float` | 成交额 |
| `volume` | `float` | 成交量 |
| `open`   | `float` | 开盘价 |
| `high`   | `float` | 最高价 |
| `low`    | `float` | 最低价 |
| `close`  | `float` | 收盘价 |

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    data = C.get_local_data(stock_code='600000.SH',start_time='20220101',end_time='20220131',period='1d',divid_type='none')
    print(data)
```

### ContextInfo.get_history_data - 获取历史行情数据【不推荐】

警告

1. 此函数已不推荐使用，推荐使用[ContextInfo.get_market_data_ex()](https://dict.thinktrader.net/innerApi/data_function.html#contextinfo-get-market-data-ex-%E8%8E%B7%E5%8F%96%E8%A1%8C%E6%83%85%E6%95%B0%E6%8D%AE)
2. 此函数使用前需要先通过[ContextInfo.set_universe()](https://dict.thinktrader.net/innerApi/data_function.html?id=7zqjlm)设定股票池

**原型**

内置python

```
ContextInfo.get_history_data(
    len, 
    period, 
    field, 
    dividend_type = 0,
    skip_paused = True)
```

**释义**

获取历史行情数据

**参数**

| 名称              | 类型       | 描述                                                                                                                                                                                               |
| --------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `len`           | `int`    | `需获取的历史数据长度`                                                                                                                                                                                     |
| `period`        | `string` | 需获取的历史数据周期，可选值包括：<br>`'tick'`：分笔线、 `'1d'`：日线、 `'1m'`：1分钟线<br>`'3m'`：3分钟线、 `'5m'`：5分钟线、 `'15m'`：15分钟线<br>`'30m'`：30分钟线、 `'1h'`：小时线、 `'1w'`：周线<br>`'1mon'`：月线、 `'1q'`：季线、 `'1hy'`：半年线<br>`'1y'`：年线 |
| `field`         | `string` | 需获取的历史数据的类型，可选值包括：<br>`'open'`：开盘价<br>`'high'`：最高价<br>`'low'`：最低价<br>`'close'`：收盘价<br>`'quoter'`：详细报价（结构见 `get_market_data` 方法）                                                                  |
| `dividend_type` | `int`    | 默认参数，除复权，默认不复权，可选值包括：<br>`0`：不复权<br>`1`：向前复权<br>`2`：向后复权<br>`3`：等比向前复权<br>`4`：等比向后复权                                                                                                             |
| `skip_paused`   | `bool`   | 默认参数，是否停牌填充，默认填充                                                                                                                                                                                 |

**返回值** 一个字典`dict`结构，key 为 stockcode.market, value 为行情数据 list，list 中第 0 位为最早的价格，第 1 位为次早价格，依次下去。

**示例**

示例返回值

```
# coding = gbk
def init(C):
    C.stock_list = ["000001.SZ","600519.SH", "510050.SH"]
    C.set_universe(C.stock_list)

def handlebar(C):
    data = C.get_history_data(2, '1d', 'close')
    print(data)
```

### ContextInfo.get_market_data() - 获取行情数据【不推荐】

提示

推荐使用[ContextInfo.get_market_data_ex()](https://dict.thinktrader.net/innerApi/data_function.html#contextinfo-get-market-data-ex-%E8%8E%B7%E5%8F%96%E8%A1%8C%E6%83%85%E6%95%B0%E6%8D%AE)

**原型**

内置python

```
ContextInfo.get_market_data(
    fields, 
    stock_code = [], 
    start_time = '', 
    end_time = '',
    skip_paused = True, 
    period = 'follow', 
    dividend_type = 'follow', 
    count = -1)
```

**释义**

获取行情数据

**参数**

| 字段名             | 数据类型        | 解释                                                                                                                                                                                               |
| --------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `fields`        | 字段列表        | 可选值包括：<br>`'open'`: 开<br>`'high'`: 高<br>`'low'`: 低<br>`'close'`: 收<br>`'volume'`: 成交量<br>`'amount'`: 成交额<br>`'settle'`: 结算价<br>`'quoter'`: 分笔数据（包括历史）                                            |
| `stock_code`    | 默认参数，合约代码列表 | 合约格式为 `code.market`，例如 '600000.SH'，不指定时为当前图合约                                                                                                                                                    |
| `start_time`    | 默认参数，时间戳    | 开始时间，格式为 '20171209' 或 '20171209010101'                                                                                                                                                           |
| `end_time`      | 默认参数，时间戳    | 结束时间，格式为 '20171209' 或 '20171209010101'                                                                                                                                                           |
| `skip_paused`   | 默认参数，布尔值    | 如何处理停牌数据：<br>`true`：如果是停牌股，会自动填充未停牌前的价格作为停牌日的价格<br>`false`：停牌数据为 NaN                                                                                                                             |
| `period`        | `string`    | 需获取的历史数据周期，可选值包括：<br>`'tick'`：分笔线、 `'1d'`：日线、 `'1m'`：1分钟线<br>`'3m'`：3分钟线、 `'5m'`：5分钟线、 `'15m'`：15分钟线<br>`'30m'`：30分钟线、 `'1h'`：小时线、 `'1w'`：周线<br>`'1mon'`：月线、 `'1q'`：季线、 `'1hy'`：半年线<br>`'1y'`：年线 |
| `dividend_type` | 默认参数，字符串    | 缺省值为 'none'，除复权，可选值包括：<br>`'none'`：不复权<br>`'front'`：向前复权<br>`'back'`：向后复权<br>`'front_ratio'`：等比向前复权<br>`'back_ratio'`：等比向后复权                                                                     |
| `count`         | 默认参数，整数     | 缺省值为 -1。当大于等于 0 时，效果与 `get_history_data` 保持一致                                                                                                                                                    |

- count参数设置的几种情况

| count 取值   | 时间设置是否生效 | 开始时间和结束时间设置效果                  |
| ---------- | -------- | ------------------------------ |
| count >= 0 | **生效**   | 返回数量取决于开始时间与结束时间和count与结束时间的交集 |
| count = -1 | **生效**   | 同时设置开始时间和结束时间，在所设置的时间段内取值      |
| count = -1 | **生效**   | 开始时间结束时间都不设置，取当前最新bar的值        |
| count = -1 | **生效**   | 只设置开始时间，取所设开始时间到当前时间的值         |
| count = -1 | **生效**   | 只设置结束时间，取股票上市第一根 bar 到所设结束时间的值 |

**返回值**

- 返回值根据传入的参数情况，会返回不同类型的结果

| count | 字段数量 | 股票数量 | 时间点 | 返回类型                            |
| ----- | ---- | ---- | --- | ------------------------------- |
| =-1   | =1   | =1   | =1  | float                           |
| =-1   | >1   | =1   | 默认值 | pandas.Series                   |
| >=-1  | >=1  | =1   | >=1 | pandas.DataFrame(字段数量和时间点不同时为1) |
| =-1   | >=1  | >1   | 默认值 | pandas.DataFrame                |
| >1    | =1   | =1   | =1  | pandas.DataFrame                |
| >=-1  | >=1  | >1   | >=1 | pandas.Panel                    |

**示例**

示例data1返回值data2返回值data3返回值data4返回值

```
# coding = gbk
def init(C):
    C.stock_list = ["000001.SZ","600519.SH", "510050.SH"]

def handlebar(C):
    data1 = C.get_market_data(["close"],["000001.SZ"],start_time = "20231106",end_time = "20231106", count = -1) # 返回float值
    data2 = C.get_market_data(["close","open"],["000001.SZ"], count = -1) # 返回pandas.Series
    data3 = C.get_market_data(["close","open"],C.stock_list, count = -1) # 返回pandas.DataFrame
    data4 = C.get_market_data(["open","high", "low", "close"],C.stock_list,count = 20) # 返回pandas.Panel

    print(data1)
    print(data2)
    print(data3)
    print(data4)
```

## 获取财务数据

获取财务数据前，请先通过`界面端数据管理 - 财务数据`下载

![财务数据下载](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E4%B8%8B%E8%BD%BD%E8%B4%A2%E5%8A%A1%E6%95%B0%E6%8D%AE-61e927b9.png)

提示

财务数据接口通过读取下载本地的数据取数，使用前需要补充本地数据。除公告日期和报表截止日期为时间戳毫秒格式其他单位为元或 %，数据主要包括资产负债表(ASHAREBALANCESHEET)、利润表（ASHAREINCOME）、现金流量表（ASHARECASHFLOW）、股本表（CAPITALSTRUCTURE）的主要字段数据以及经过计算的主要财务指标数据（PERSHAREINDEX）。建议使用本文档对照表中的英文表名和迅投英文字段，表名不区分大小写。

### ContextInfo.get_financial_data - 获取财务数据

财务数据接口有两种用法，入参和返回值不同，具体如下

#### 用法1

**原型**

内置python

```
ContextInfo.get_financial_data(fieldList, stockList, startDate, enDate, report_type = 'announce_time')
```

**释义**

获取财务数据，方法1

**参数**

| 字段名           | 类型         | 释义与用例                                                                                  |
| ------------- | ---------- | -------------------------------------------------------------------------------------- |
| `fieldList`   | `List（必须）` | `财报字段列表：['ASHAREBALANCESHEET.fix_assets', '利润表.净利润']`                                  |
| `stockList`   | `List（必须）` | `股票列表：['600000.SH', '000001.SZ']`                                                      |
| `startDate`   | `Str（必须）`  | `开始时间：'20171209'`                                                                      |
| `endDate`     | `Str（必须）`  | `结束时间：'20171212'`                                                                      |
| `report_type` | `Str（可选）`  | `报表时间类型，可缺省，默认是按照数据的公告期为区分取数据，设置为 'report_time' 为按照报告期取数据，' announce_time' 为按照公告日期取数据` |

提示

选择按照公告期取数和按照报告期取数的区别：

报告日期是指财务报告所覆盖的会计时间段，而公告日期是指公司向外界公布该报告的具体时间点

若指定report_type为`report_time`，则不会考虑财报的公告日期，**可能会取到未来数据**

若指定report_type为`announce_time`，则会按财报实际发布日期返回数据，**不会取到未来数据**

例：

**返回值**

函数根据stockList代码列表,startDate,endDate时间范围，返回不同的的数据类型。如下：

| 代码数量 | 时间范围 | 返回类型                                                        |
| ---- | ---- | ----------------------------------------------------------- |
| =1   | =1   | pandas.Series (index = 字段)                                  |
| =1   | >1   | pandas.DataFrame (index = 时间, columns = 字段)                 |
| >1   | =1   | pandas.DataFrame (index = 代码, columns = 字段)                 |
| >1   | >1   | pandas.Panel (items = 代码, major_axis = 时间, minor_axis = 字段) |

**示例**

示例返回值

```
# coding:gbk
def init(C):
  pass

def handlebar(C):

  #取总股本和净利润
  fieldList = ['CAPITALSTRUCTURE.total_capital', '利润表.净利润']   
  stockList = ["000001.SZ","000002.SZ","430017.BJ"]
  startDate = '20171209'
  endDate = '20231204'
  data = C.get_financial_data(fieldList, stockList, startDate, endDate, report_type = 'report_time')
  print(data)
```

#### 用法2

**原型**

内置python

```
ContextInfo.get_financial_data(tabname, colname, market, code, report_type = 'report_time', barpos)
```

与用法 1 可同时使用

**释义**

获取财务数据，方法2

**参数**

| 字段名           | 类型        | 释义与用例                                                                                   |
| ------------- | --------- | --------------------------------------------------------------------------------------- |
| `tabname`     | `Str（必须）` | `表名：'ASHAREBALANCESHEET'`                                                               |
| `colname`     | `Str（必须）` | `字段名：'fix_assets'`                                                                      |
| `market`      | `Str（必须）` | `市场：'SH'`                                                                               |
| `code`        | `Str（必须）` | `代码：'600000'`                                                                           |
| `report_type` | `Str（可选）` | `报表时间类型，可缺省，默认是按照数据的公告期为区分取数据，设置为 'report_time' 为按照报告期取数据，' announce_time ' 为按照公告日期取数据` |
| `barpos`      | `number`  | `当前 bar 的索引`                                                                            |

**返回值**

`float` ：所取字段的数值

**示例**

示例返回值

```
# coding:gbk
def init(C):
  pass

def handlebar(C):
  index = C.barpos
  data = C.get_financial_data('ASHAREBALANCESHEET', 'fix_assets', 'SH', '600000', index)
  print(data)
```

### ContextInfo.get_raw_financial_data - 获取原始财务数据

提示

取原始财务数据,与get_financial_data相比不填充每个交易日的数据

**原型**

内置python

```
ContextInfo.get_raw_financial_data(fieldList,stockList,startDate,endDate,report_type='announce_time')
```

**释义**

取原始财务数据,与get_financial_data相比不填充每个交易日的数据

**参数**

| 字段名           | 类型         | 释义与用例                                                                                    |
| ------------- | ---------- | ---------------------------------------------------------------------------------------- |
| `fieldList`   | `List（必须）` | 字段列表：例如 ['资产负债表.固定资产','利润表.净利润']                                                         |
| `stockList`   | `List（必须）` | 股票列表：例如['600000.SH','000001.SZ']                                                         |
| `startDate`   | `Str（必须）`  | 开始时间：例如 '20171209'                                                                       |
| `endDate`     | `Str（必须）`  | 结束时间：例如 '20171212'                                                                       |
| `report_type` | `Str（可选）`  | 时间类型，可缺省，默认是按照数据的公告期为区分取数据，设置为 'report_time' 为按照报告期取数据，可选值:'announce_time','report_time' |

**返回值**

函数根据stockList代码列表,startDate,endDate时间范围，返回不同的的数据类型。如下：

| 代码数量 | 时间范围 | 返回类型                                                        |
| ---- | ---- | ----------------------------------------------------------- |
| =1   | =1   | pandas.Series (index = 字段)                                  |
| =1   | >1   | pandas.DataFrame (index = 时间, columns = 字段)                 |
| >1   | =1   | pandas.DataFrame (index = 代码, columns = 字段)                 |
| >1   | >1   | pandas.Panel (items = 代码, major_axis = 时间, minor_axis = 字段) |

**示例**

示例返回值

```
#encoding:gbk
'''
获取财务数据
'''
import pandas as pd
import numpy as np
import talib

def to_zw(a):
    '''0.中文价格字符串'''
    import numpy as np
    try:
        header = '' if a > 0 else '-'
        if np.isnan(a):
            return '问题数据'
        if abs(a) < 1000:
            return header + str(int(a)) + "元"
        if abs(a) < 10000:
            return header + str(int(a))[0] + "千"
        if abs(a) < 100000000:
            return header + str(int(a))[:-4] + "万" + str(int(a))[-4] + '千'
        else:
            return header + str(int(a))[:-8] + "亿" + str(int(a))[-8:-4] + '万'
    except:
        print(f"问题数据{a}")
        return '问题数据'


def after_init(C):
    fieldList = ['ASHAREINCOME.net_profit_excl_min_int_inc','ASHAREINCOME.revenue'] # 字段表
    stockList = ['000001.SZ'] # 标的
    a=C.get_raw_financial_data(fieldList,stockList,'20150101','20300101',report_type = 'report_time') # 获取原始财务数据
    # print(a)
    for stock in a:
        for key in a[stock]:
            for t in a[stock][key]:
                print(key, timetag_to_datetime(int(t),'%Y%m%d'), to_zw(a[stock][key][t]))
            print('-' *22)
        print('-' *22)
```

### ContextInfo.get_last_volume - 获取最新流通股本

**原型**

内置python

```
ContextInfo.get_last_volume(stockcode)
```

**释义**

获取最新流通股本

**参数**

| 字段名         | 数据类型     | 解释                         |
| ----------- | -------- | -------------------------- |
| `stockcode` | `string` | 标的名称，必须是 'stock.market' 形式 |

**返回值**

`int`类型值,代表流通股本数量

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    data = C.get_last_volume("000001.SZ")
    print(data)
```

### ContextInfo.get_total_share - 获取总股数

**原型**

内置python

```
ContextInfo.get_total_share(stockcode)
```

**释义**

获取总股数

**参数**

| 字段名         | 数据类型     | 解释                                  |
| ----------- | -------- | ----------------------------------- |
| `stockcode` | `string` | 股票代码，缺省值 ''，默认为当前图代码, 如：'600000.SH' |

**返回值**

`int`:总股数

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    data = C.get_total_share('600000.SH')
    print(data)
```

### 财务数据字段表

#### 资产负债表 (ASHAREBALANCESHEET)

| **中文字段**       | **迅投字段**                            |
| -------------- | ----------------------------------- |
| `应收利息`         | `int_rcv`                           |
| `可供出售金融资产`     | `fin_assets_avail_for_sale`         |
| `持有至到期投资`      | `held_to_mty_invest`                |
| `长期股权投资`       | `long_term_eqy_invest`              |
| `固定资产`         | `fix_assets`                        |
| `无形资产`         | `intang_assets`                     |
| `递延所得税资产`      | `deferred_tax_assets`               |
| `资产总计`         | `tot_assets`                        |
| `交易性金融负债`      | `tradable_fin_liab`                 |
| `应付职工薪酬`       | `empl_ben_payable`                  |
| `应交税费`         | `taxes_surcharges_payable`          |
| `应付利息`         | `int_payable`                       |
| `应付债券`         | `bonds_payable`                     |
| `递延所得税负债`      | `deferred_tax_liab`                 |
| `负债合计`         | `tot_liab`                          |
| `实收资本(或股本)`    | `cap_stk`                           |
| `资本公积金`        | `cap_rsrv`                          |
| `盈余公积金`        | `surplus_rsrv`                      |
| `未分配利润`        | `undistributed_profit`              |
| `归属于母公司股东权益合计` | `tot_shrhldr_eqy_excl_min_int`      |
| `少数股东权益`       | `minority_int`                      |
| `负债和股东权益总计`    | `tot_liab_shrhldr_eqy`              |
| `所有者权益合计`      | `total_equity`                      |
| `货币资金`         | `cash_equivalents`                  |
| `应收票据`         | `bill_receivable`                   |
| `应收账款`         | `account_receivable`                |
| `预付账款`         | `advance_payment`                   |
| `其他应收款`        | `other_receivable`                  |
| `其他流动资产`       | `other_current_assets`              |
| `流动资产合计`       | `total_current_assets`              |
| `存货`           | `inventories`                       |
| `在建工程`         | `constru_in_process`                |
| `工程物资`         | `construction_materials`            |
| `长期待摊费用`       | `long_deferred_expense`             |
| `非流动资产合计`      | `total_non_current_assets`          |
| `短期借款`         | `shortterm_loan`                    |
| `应付股利`         | `dividend_payable`                  |
| `其他应付款`        | `other_payable`                     |
| `一年内到期的非流动负债`  | `non_current_liability_in_one_year` |
| `其他流动负债`       | `other_current_liability`           |
| `长期应付款`        | `longterm_account_payable`          |
| `应付账款`         | `accounts_payable`                  |
| `预收账款`         | `advance_peceipts`                  |
| `流动负债合计`       | `total_current_liability`           |
| `应付票据`         | `notes_payable`                     |
| `长期借款`         | `long_term_loans`                   |
| `专项应付款`        | `grants_received`                   |
| `其他非流动负债`      | `other_non_current_liabilities`     |
| `非流动负债合计`      | `non_current_liabilities`           |
| `专项储备`         | `specific_reserves`                 |
| `商誉`           | `goodwill`                          |
| `报告截止日`        | `m_timetag`                         |
| `公告日`          | `m_anntime`                         |

#### 利润表 (ASHAREINCOME)

| **中文字段**         | **迅投字段**                        |
| ---------------- | ------------------------------- |
| `投资收益`           | `plus_net_invest_inc`           |
| `联营企业和合营企业的投资收益` | `incl_inc_invest_assoc_jv_entp` |
| `营业税金及附加`        | `less_taxes_surcharges_ops`     |
| `营业总收入`          | `revenue`                       |
| `营业总成本`          | `total_operating_cost`          |
| `营业收入`           | `revenue_inc`                   |
| `营业成本`           | `total_expense`                 |
| `资产减值损失`         | `less_impair_loss_assets`       |
| `营业利润`           | `oper_profit`                   |
| `营业外收入`          | `plus_non_oper_rev`             |
| `营业外支出`          | `less_non_oper_exp`             |
| `利润总额`           | `tot_profit`                    |
| `所得税`            | `inc_tax`                       |
| `净利润`            | `net_profit_incl_min_int_inc`   |
| `归母净利润`          | `net_profit_excl_min_int_inc`   |
| `管理费用`           | `less_gerl_admin_exp`           |
| `销售费用`           | `sale_expense`                  |
| `财务费用`           | `financial_expense`             |
| `综合收益总额`         | `total_income`                  |
| `归属于少数股东的综合收益总额` | `total_income_minority`         |
| `公允价值变动收益`       | `change_income_fair_value`      |
| `已赚保费`           | `earned_premium`                |
| `报告截止日`          | `m_timetag`                     |
| `公告日`            | `m_anntime`                     |

#### 现金流量表 (ASHARECASHFLOW)

| **中文字段**                    | **迅投字段**                                   |
| --------------------------- | ------------------------------------------ |
| `收到其他与经营活动有关的现金`            | `other_cash_recp_ral_oper_act`             |
| `经营活动现金流入小计`                | `stot_cash_inflows_oper_act`               |
| `支付给职工以及为职工支付的现金`           | `cash_pay_beh_empl`                        |
| `支付的各项税费`                   | `pay_all_typ_tax`                          |
| `支付其他与经营活动有关的现金`            | `other_cash_pay_ral_oper_act`              |
| `经营活动现金流出小计`                | `stot_cash_outflows_oper_act`              |
| `经营活动产生的现金流量净额`             | `net_cash_flows_oper_act`                  |
| `取得投资收益所收到的现金`              | `cash_recp_return_invest`                  |
| `处置固定资产、无形资产和其他长期投资收到的现金`   | `net_cash_recp_disp_fiolta`                |
| `投资活动现金流入小计`                | `stot_cash_inflows_inv_act`                |
| `投资支付的现金`                   | `cash_paid_invest`                         |
| `购建固定资产、无形资产和其他长期投资支付的现金`   | `cash_pay_acq_const_fiolta`                |
| `支付其他与投资的现金`                | `other_cash_pay_ral_inv_act`               |
| `投资活动产生的现金流出小计`             | `stot_cash_outflows_inv_act`               |
| `投资活动产生的现金流量净额`             | `net_cash_flows_inv_act`                   |
| `吸收投资收到的现金`                 | `cash_recp_cap_contrib`                    |
| `取得借款收到的现金`                 | `cash_recp_borrow`                         |
| `收到其他与筹资活动有关的现金`            | `other_cash_recp_ral_fnc_act`              |
| `筹资活动现金流入小计`                | `stot_cash_inflows_fnc_act`                |
| `偿还债务支付现金`                  | `cash_prepay_amt_borr`                     |
| `分配股利、利润或偿付利息支付的现金`         | `cash_pay_dist_dpcp_int_exp`               |
| `支付其他与筹资的现金`                | `other_cash_pay_ral_fnc_act`               |
| `筹资活动现金流出小计`                | `stot_cash_outflows_fnc_act`               |
| `筹资活动产生的现金流量净额`             | `net_cash_flows_fnc_act`                   |
| `汇率变动对现金的影响`                | `eff_fx_flu_cash`                          |
| `现金及现金等价物净增加额`              | `net_incr_cash_cash_equ`                   |
| `销售商品、提供劳务收到的现金`            | `goods_sale_and_service_render_cash`       |
| `收到的税费与返还`                  | `tax_levy_refund`                          |
| `购买商品、接受劳务支付的现金`            | `goods_and_services_cash_paid`             |
| `处置子公司及其他收到的现金`             | `net_cash_deal_subcompany`                 |
| `其中子公司吸收现金`                 | `cash_from_mino_s_invest_sub`              |
| `处置固定资产、无形资产和其他长期资产支付的现金净额` | `fix_intan_other_asset_dispo_cash_payment` |
| `报告截止日`                     | `m_timetag`                                |
| `公告日`                       | `m_anntime`                                |

#### 股本表 (CAPITALSTRUCTURE)

| **中文字段**  | **迅投字段**                                     |
| --------- | -------------------------------------------- |
| `总股本`     | `total_capital`                              |
| `已上市流通A股` | `circulating_capital`                        |
| `自由流通股本`  | `free_float_capital`（旧版本为`freeFloatCapital`） |
| `限售流通股份`  | `restrict_circulating_capital`               |
| `变动日期`    | `m_timetag`                                  |
| `公告日`     | `m_anntime`                                  |

#### 主要指标 (PERSHAREINDEX)

| **中文字段**            | **迅投字段**                                |
| ------------------- | --------------------------------------- |
| `每股经营活动现金流量`        | `s_fa_ocfps`                            |
| `每股净资产`             | `s_fa_bps`                              |
| `基本每股收益`            | `s_fa_eps_basic`                        |
| `稀释每股收益`            | `s_fa_eps_diluted`                      |
| `每股未分配利润`           | `s_fa_undistributedps`                  |
| `每股资本公积金`           | `s_fa_surpluscapitalps`                 |
| `扣非每股收益`            | `adjusted_earnings_per_share`           |
| `净资产收益率`            | `du_return_on_equity`                   |
| `销售毛利率`             | `sales_gross_profit`                    |
| `主营收入同比增长`          | `inc_revenue_rate`                      |
| `净利润同比增长`           | `du_profit_rate`                        |
| `归属于母公司所有者的净利润同比增长` | `inc_net_profit_rate`                   |
| `扣非净利润同比增长`         | `adjusted_net_profit_rate`              |
| `营业总收入滚动环比增长`       | `inc_total_revenue_annual`              |
| `归属净利润滚动环比增长`       | `inc_net_profit_to_shareholders_annual` |
| `扣非净利润滚动环比增长`       | `adjusted_profit_to_profit_annual`      |
| `加权净资产收益率`          | `equity_roe`                            |
| `摊薄净资产收益率`          | `net_roe`                               |
| `摊薄总资产收益率`          | `total_roe`                             |
| `毛利率`               | `gross_profit`                          |
| `净利率`               | `net_profit`                            |
| `实际税率`              | `actual_tax_rate`                       |
| `预收款营业收入`           | `pre_pay_operate_income`                |
| `销售现金流营业收入`         | `sales_cash_flow`                       |
| `资产负债比率`            | `gear_ratio`                            |
| `存货周转率`             | `inventory_turnover`                    |

#### 十大股东/十大流通股东 (TOP10HOLDER/TOP10FLOWHOLDER)

提示

对于公告内披露的十大股东数量大于10条的，我们会保留原始数据，以保持和公司公告信息一致

| **中文字段** | **迅投字段**      |
| -------- | ------------- |
| `公告日期`   | `declareDate` |
| `截止日期`   | `endDate`     |
| `股东名称`   | `name`        |
| `股东类型`   | `type`        |
| `持股数量`   | `quantity`    |
| `变动原因`   | `reason`      |
| `持股比例`   | `ratio`       |
| `股份性质`   | `nature`      |
| `持股排名`   | `rank`        |

#### 股东数 (SHAREHOLDER)

| **中文字段**  | **迅投字段**           |
| --------- | ------------------ |
| `公告日期`    | `declareDate`      |
| `截止日期`    | `endDate`          |
| `股东总数`    | `shareholder`      |
| `A股东户数`   | `shareholderA`     |
| `B股东户数`   | `shareholderB`     |
| `H股东户数`   | `shareholderH`     |
| `已流通股东户数` | `shareholderFloat` |
| `未流通股东户数` | `shareholderOther` |

## 获取合约信息

### ContextInfo.get_instrument_detail - 根据代码获取合约详细信息

提示

旧版本客户端中，函数名为ContextInfo.get_instrumentdetail；不支持iscomplete参数

**原型**

内置python

```
ContextInfo.get_instrument_detail(stockcode,iscomplete = Fasle)
```

**释义**

根据代码获取合约详细信息

**参数**

| 字段名          | 数据类型     | 解释                         |
| ------------ | -------- | -------------------------- |
| `stockcode`  | `string` | 标的名称，必须是 'stock.market' 形式 |
| `iscomplete` | `bool`   | 是否获取全部字段，默认为False          |

**返回值**

根据stockcode返回一个dict。该字典数据key值有：

| 名称                   | 类型     | 描述                                   |
| -------------------- | ------ | ------------------------------------ |
| ExchangeID           | string | 合约市场代码                               |
| InstrumentID         | string | 合约代码                                 |
| InstrumentName       | string | 合约名称                                 |
| ProductID            | string | 合约的品种ID(期货)                          |
| ProductName          | string | 合约的品种名称(期货)                          |
| ProductType          | int    | 合约的类型, 默认-1,枚举值可参考下方说明               |
| ExchangeCode         | string | 交易所代码                                |
| UniCode              | string | 统一规则代码                               |
| CreateDate           | str    | 创建日期                                 |
| OpenDate             | str    | 上市日期（特殊值情况见表末）                       |
| ExpireDate           | int    | 退市日或者到期日（特殊值情况见表末）                   |
| PreClose             | float  | 前收盘价格                                |
| SettlementPrice      | float  | 前结算价格                                |
| UpStopPrice          | float  | 当日涨停价                                |
| DownStopPrice        | float  | 当日跌停价                                |
| FloatVolume          | float  | 流通股本（注意，部分低等级客户端中此字段为FloatVolumn）    |
| TotalVolume          | float  | 总股本（注意，部分低等级客户端中此字段为FloatVolumn）     |
| LongMarginRatio      | float  | 多头保证金率                               |
| ShortMarginRatio     | float  | 空头保证金率                               |
| PriceTick            | float  | 最小价格变动单位                             |
| VolumeMultiple       | int    | 合约乘数(对期货以外的品种，默认是1)                  |
| MainContract         | int    | 主力合约标记，1、2、3分别表示第一主力合约，第二主力合约，第三主力合约 |
| LastVolume           | int    | 昨日持仓量                                |
| InstrumentStatus     | int    | 合约停牌状态(<=0:正常交易（-1:复牌）;>=1停牌天数;)     |
| IsTrading            | bool   | 合约是否可交易                              |
| IsRecent             | bool   | 是否是近月合约                              |
| ChargeType           | int    | 期货和期权手续费方式                           |
| ChargeOpen           | float  | 开仓手续费(率)                             |
| ChargeClose          | float  | 平仓手续费(率)                             |
| ChargeTodayOpen      | float  | 开今仓(日内开仓)手续费(率)                      |
| ChargeTodayClose     | float  | 平今仓(日内平仓)手续费(率)                      |
| OptionType           | int    | 期权类型                                 |
| OpenInterestMultiple | int    | 交割月持仓倍数                              |

提示

字段`OpenDate`有以下几种特殊值： 19700101=新股, 19700102=老股东增发, 19700103=新债, 19700104=可转债, 19700105=配股， 19700106=配号 字段`ExpireDate`为0 或 99999999 时，表示该标的暂无退市日或到期日

字段`ProductType` 对于**股票以外**的品种，有以下几种值

**国内期货市场：** 1-期货 2-期权(DF SF ZF INE GF) 3-组合套利 4-即期 5-期转现 6-期权(IF) 7-结算价交易(tas)

**沪深股票期权市场：**0-认购 1-认沽

**外盘：** 1-100：期货， 101-200：现货, 201-300:股票相关 1：股指期货 2：能源期货 3：农业期货 4：金属期货 5：利率期货 6：汇率期货 7：数字货币期货 99：自定义合约期货 107：数字货币现货 201：股票 202：GDR 203：ETF 204：ETN 300：其他

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    data = C.get_instrumentdetail("000001.SZ")
    print(data)
```

### get_st_status - 获取历史st状态

提示

本函数需要下载历史ST数据(过期合约K线),可通过`界面端数据管理 - 过期合约数据`下载

**原型**

内置python

```
get_st_status(stockcode)
```

**释义**

获取历史st状态

**参数**

| 字段名         | 数据类型     | 解释                            |
| ----------- | -------- | ----------------------------- |
| `stockcode` | `string` | 股票代码，如000004.SZ（可为空，为空时取主图代码） |

**返回值**

st范围字典 格式 {'ST': [['20210520', '20380119']], '*ST': [['20070427', '20080618'], ['20200611', '20210520']]}

示例：

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    print(get_st_status('600599.SH'))
```

### ContextInfo.get_his_st_data - 获取某只股票ST的历史

提示

本函数需要下载历史ST数据(过期合约K线),可通过`界面端数据管理 - 过期合约数据`下载

**原型**

内置python

```
ContextInfo.get_his_st_data(stockcode)
```

**释义**

获取某只股票ST的历史

**参数**

| 字段名         | 数据类型     | 解释                                 |
| ----------- | -------- | ---------------------------------- |
| `stockcode` | `string` | 股票代码，'stkcode.market'，如'000004.SZ' |

**返回值**

`dict`,st历史，key为ST,*ST,PT,历史未ST会返回{}

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    print(C.get_his_st_data('000004.SZ'))
```

### ContextInfo.get_main_contract - 获取期货主力合约

提示

1. 该函数支持实盘/回测两种模式
2. 若要使用该函数获取历史主力合约，必须要先下载`历史主力合约`数据
3. `历史主力合约`数据目前通过`界面端数据管理 - 过期合约数据 - 历史主力合约`下载

**原型**

内置python

```
ContextInfo.get_main_contract(codemarket)
ContextInfo.get_main_contract(codemarket,date="")
ContextInfo.get_main_contract(codemarket,startDate="",endDate="")
```

**释义**

获取当前期货主力合约

**参数**

| 字段名          | 数据类型     | 解释                                 |
| ------------ | -------- | ---------------------------------- |
| `codemarket` | `string` | 合约和市场，合约格式为品种名加00，如IF00.IF，zn00.SF |
| `startDate`  | `string` | 开始日期(可以不写),如20180608               |
| `endDate`    | `string` | 结束日期(可以不写),如20190608               |

**返回值**

`str`，合约代码

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    symbol1 = C.get_main_contract('IF00.IF')# 获取当前主力合约

    symbol2 = C.get_main_contract('IF00.IF',"20190101")# 获取指定日期主力合约

    symbol3 = C.get_main_contract('IF00.IF',"20181101","20190101") # 获取时间段内全部主力合约

    print(symbol1, symbol2)
    print("="*10)
    print(symbol3)
```

### ContextInfo.get_contract_multiplier - 获取合约乘数

**原型**

内置python

```
ContextInfo.get_contract_multiplier(contractcode)
```

**释义**

获取合约乘数

**参数**

| 字段名            | 数据类型     | 解释                                    |
| -------------- | -------- | ------------------------------------- |
| `contractcode` | `string` | 合约代码，格式为 'code.market'，例如 'IF1707.IF' |

**返回值**`int`,表示合约乘数

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    multiplier = C.get_contract_multiplier("rb2401.SF")
    print(multiplier)
```

### ContextInfo.get_contract_expire_date - 获取期货合约到期日

**原型**

内置python

```
ContextInfo.get_contract_expire_date(codemarket)
```

**释义**

获取期货合约到期日

**参数**

| 字段名          | 数据类型     | 解释                     |
| ------------ | -------- | ---------------------- |
| `Codemarket` | `string` | 合约和市场,如IF00.IF,zn00.SF |

**返回值**`str`，合约到期日

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    data = C.get_contract_expire_date("IF2311.IF")
    # print(type(data))
    print(data)
```

### ContextInfo.get_his_contract_list - 获取市场已退市合约

**原型**

内置python

```
ContextInfo.get_his_contract_list(market)
```

**释义**

获取市场已退市合约，需要手动补充过期合约列表

**参数**

| 字段名      | 数据类型     | 解释                   |
| -------- | -------- | -------------------- |
| `market` | `string` | 市场,SH,SZ,SHO,SZO,IF等 |

**返回值**

`list`,合约代码列表

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):

    print(C.get_his_contract_list('SHO')[:30])
```

## 获取期权信息

### ContextInfo.get_option_detail_data - 获取指定期权品种的详细信息

**原型**

内置python

```
ContextInfo.get_option_detail_data(optioncode)
```

**释义**

获取指定期权品种的详细信息

**参数**

| 字段名          | 数据类型     | 解释                                         |
| ------------ | -------- | ------------------------------------------ |
| `optioncode` | `string` | 期权代码,如'10001506.SHO',当填写空字符串时候默认为当前主图的期权品种 |

**返回值**`dict`,字段如下：

| 字段                   | 类型    | 说明        |
| -------------------- | ----- | --------- |
| ExchangeID           | str   | 期权市场代码    |
| InstrumentID         | str   | 期权代码      |
| ProductID            | str   | 期权标的的产品ID |
| OpenDate             | int   | 发行日期      |
| ExpireDate           | int   | 到期日       |
| PreClose             | float | 前收价格      |
| SettlementPrice      | float | 前结算价格     |
| UpStopPrice          | float | 当日涨停价     |
| DownStopPrice        | float | 当日跌停价     |
| LongMarginRatio      | float | 多头保证金率    |
| ShortMarginRatio     | float | 空头保证金率    |
| PriceTick            | float | 最小变价单位    |
| VolumeMultiple       | int   | 合约乘数      |
| MaxMarketOrderVolume | int   | 涨跌停价最大下单量 |
| MinMarketOrderVolume | int   | 涨跌停价最小下单量 |
| MaxLimitOrderVolume  | int   | 限价单最大下单量  |
| MinLimitOrderVolume  | int   | 限价单最小下单量  |
| OptUnit              | int   | 期权合约单位    |
| MarginUnit           | float | 期权单位保证金   |
| OptUndlCode          | str   | 期权标的证券代码  |
| OptUndlMarket        | str   | 期权标的证券市场  |
| OptExercisePrice     | float | 期权行权价     |
| NeeqExeType          | str   | 全国股转转让类型  |
| OptUndlRiskFreeRate  | float | 期权标的无风险利率 |
| OptUndlHistoryRate   | float | 期权标的历史波动率 |
| EndDelivDate         | int   | 期权行权终止日   |
| optType              | str   | 期权类型      |

**示例**

示例返回值

```
#encoding:gbk
def init(ContextInfo):
  pass

def after_init(ContextInfo):
  print(ContextInfo.get_option_detail_data('10002235.SHO'))
```

### ContextInfo.get_option_list - 获取指定期权列表

**原型**

内置python

```
ContextInfo.get_option_list(undl_code,dedate,opttype,isavailable)
```

**释义**

获取指定期权列表。如获取历史期权，需先下载过期合约列表

**参数**

| 字段名           | 数据类型     | 解释                                                                                      |
| ------------- | -------- | --------------------------------------------------------------------------------------- |
| `undl_code`   | `string` | 期权标的代码,如'510300.SH'                                                                     |
| `dedate`      | `string` | 期权到期月或当前交易日期，"YYYYMM"格式为期权到期月，"YYYYMMDD"格式为获取当前日期交易的期权                                  |
| `opttype`     | `string` | 期权类型，默认值为空，"CALL"，"PUT"，为空时认购认沽都取                                                       |
| `isavailable` | `bool`   | 是否可交易，当`dedate`的格式为"YYYYMMDD"格式为获取当前日期交易的期权时，`isavailable`为True时返回当前可用，为False时返回当前和历史可用 |

**返回值**

`list`，期权合约列表

**示例**

示例data1返回值data2返回值data3返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    # 获取到期月份为202101的上交所510300ETF认购合约
    data1=C.get_option_list('510300.SH','202101',"CALL")

    # 获取20210104当天上交所510300ETF可交易的认购合约
    data2=C.get_option_list('510300.SH','20210104',"CALL",True)

    # 获取20210104当天上交所510300ETF已经上市的认购合约(包括退市)
    data3=C.get_option_list('510300.SH','20210104',"CALL",False)
```

### ContextInfo.get_option_undl_data - 获取指定期权标的对应的期权品种列表

**原型**

内置python

```
ContextInfo.get_option_undl_data(undl_code_ref)
```

**释义**

获取指定期权标的对应的期权品种列表

**参数**

| 字段名             | 数据类型     | 解释                                 |
| --------------- | -------- | ---------------------------------- |
| `undl_code_ref` | `string` | 期权标的代码,如'510300.SH'，传空字符串时获取全部标的数据 |

**返回值**

指定期权标的代码时返回对应该标的的期权合约列表`list`

期权标的代码为空字符串时返回全部标的对应的品种列表的字典`dict`

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):

    print(C.get_option_undl_data('510300.SH')[:30])
```

### ContextInfo.bsm_price - 基于BS模型计算欧式期权理论价格

**原型**

内置python

```
ContextInfo.bsm_price(optionType,objectPrices,strikePrice,riskFree,sigma,days,dividend)
```

**释义**

基于Black-Scholes-Merton模型，输入期权标的价格、期权行权价、无风险利率、期权标的年化波动率、剩余天数、标的分红率、计算期权的理论价格

**参数**

| 字段             | 类型      | 说明                     |
| -------------- | ------- | ---------------------- |
| `optionType`   | `str`   | `期权类型，认购：'C'，认沽：'P'`   |
| `objectPrices` | `float` | `期权标的价格，可以是价格列表或者单个价格` |
| `strikePrice`  | `float` | `期权行权价`                |
| `riskFree`     | `float` | `无风险收益率`               |
| `sigma`        | `float` | `标的波动率`                |
| `days`         | `int`   | `剩余天数`                 |
| `dividend`     | `float` | `分红率`                  |

**返回**

提示

- objectPrices为float时，返回float
- objectPrices为list时，返回list
- 计算结果最小值0.0001，结果保留4位小数,输入非法参数返回nan

示例返回值

```
#encoding:gbk
import numpy as np


def init(ContextInfo):
  pass

def after_init(ContextInfo):
  object_prices=list(np.arange(3,4,0.01));
  #计算剩余15天的行权价3.5的认购期权,在无风险利率3%,分红率为0,标的年化波动率为23%时标的价格从3元到4元变动过程中期权理论价格序列
  prices=ContextInfo.bsm_price('C',object_prices,3.5,0.03,0.23,15,0)
  print(prices)
  #计算剩余15天的行权价3.5的认购期权,在无风险利率3%,分红率为0,标的年化波动率为23%时标的价格为3.51元的平值期权的理论价格
  price=ContextInfo.bsm_price('C',3.51,3.5,0.03,0.23,15,0)
  print(price)
```

### ContextInfo.bsm_iv - 基于BS模型计算欧式期权隐含波动率

**原型**

内置python

```
ContextInfo.bsm_iv(optionType,objectPrices,strikePrice,optionPrice,riskFree,days,dividend)
```

**释义** 基于Black-Scholes-Merton模型,输入期权标的价格、期权行权价、期权现价、无风险利率、剩余天数、标的分红率,计算期权的隐含波动率

**参数**

| 字段             | 类型      | 说明                     |
| -------------- | ------- | ---------------------- |
| `optionType`   | `str`   | `期权类型，认购：'C'，认沽：'P'`   |
| `objectPrices` | `float` | `期权标的价格，可以是价格列表或者单个价格` |
| `strikePrice`  | `float` | `期权行权价`                |
| `riskFree`     | `float` | `无风险收益率`               |
| `sigma`        | `float` | `标的波动率`                |
| `days`         | `int`   | `剩余天数`                 |
| `dividend`     | `float` | `分红率`                  |

**返回**

`double`

示例返回值

```
#encoding:gbk
import numpy as np

def init(ContextInfo):
    pass

def after_init(ContextInfo):
    # 计算剩余15天的行权价3.5的认购期权,在无风险利率3%,分红率为0时,标的现价3.51元,期权价格0.0725元时的隐含波动率
    iv=ContextInfo.bsm_iv('C',3.51,3.5,0.0725,0.03,15)
    print(iv)
```

## 获取除复权信息

### ContextInfo.get_divid_factors - 获取除权除息日和复权因子

**原型**

内置python

```
ContextInfo.get_divid_factors(stock.market)
```

**释义**

获取除权除息日和复权因子

**参数**

| 字段名            | 数据类型     | 解释                      |
| -------------- | -------- | ----------------------- |
| `stock.market` | `string` | 股票代码.市场代码，如 '600000.SH' |

**返回值**

`dict`

key:时间戳，

value:list[每股红利,每股送转,每股转赠,配股,配股价,是否股改,复权系数]

输入除权除息日非法时候返回空dict，合法时返回输入日期的对应的dict，不输入时返回查询股票的所有除权除息日及对应dict

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    Result = C.get_divid_factors('600000.SH')
    print(Result)
```

## 获取指数权重

### ContextInfo.get_weight_in_index - 获取某只股票在某指数中的绝对权重

**原型**

内置python

```
ContextInfo.get_weight_in_index(indexcode, stockcode)
```

**释义**

获取某只股票在某指数中的绝对权重

**参数**

| 字段名         | 数据类型     | 解释                                         |
| ----------- | -------- | ------------------------------------------ |
| `indexcode` | `string` | 指数代码，格式为 'stockcode.market'，例如 '000300.SH' |
| `stockcode` | `string` | 股票代码，格式为 'stockcode.market'，例如 '600004.SH' |

**返回值**

`float`：返回的数值单位是 %，如 1.6134 表示权重是 1.6134%

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass

def handlebar(C):
    data = C.get_weight_in_index('000300.SH', '000002.SZ')
    print(data)
```

## 获取成分股信息

### ContextInfo.get_stock_list_in_sector - 获取板块成份股

**原型**

内置python

```
ContextInfo.get_stock_list_in_sector(sectorname, realtime)
```

**释义**

获取板块成份股，支持客户端左侧板块列表中任意的板块，包括自定义板块

**参数**

| 字段名          | 数据类型     | 解释                                   |
| ------------ | -------- | ------------------------------------ |
| `sectorname` | `string` | 板块名，如 '沪深300'，'中证500'，'上证50'，'我的自选'等 |
| `realtime`   | 毫秒级时间戳   | 实时数据的毫秒级时间戳                          |

**返回值**

list：内含成份股代码，代码形式为 'stockcode.market'，如 '000002.SZ'

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass
def handlebar(C):
    print(C.get_stock_list_in_sector('上证50'))
```

## 获取交易日信息

注意

1. 该函数只能在`after_init`;`handlebar`运行

### ContextInfo.get_trading_dates - 获取交易日信息

**原型**

内置python

```
ContextInfo.get_trading_dates(stockcode,start_date,end_date,count,period='1d')
```

**释义**

ContextInfo.get_trading_dates(stockcode,start_date,end_date,count,period='1d')

**参数**

| 字段名          | 数据类型     | 解释                                                                                                                     |
| ------------ | -------- | ---------------------------------------------------------------------------------------------------------------------- |
| `stockcode`  | `string` | 股票代码,缺省值''默认为当前图代码，如:'600000.SH'                                                                                       |
| `start_date` | `string` | 开始时间，缺省值''为空时不使用，如:'20170101','20170101000000'                                                                         |
| `end_date`   | `string` | 结束时间，缺省值''默认为当前bar的时间，如:'20170102','20170102000000'                                                                    |
| `count`      | `int`    | K线个数，必须大于0，取包括end_date往前的count个K线，start_date不为空时此值无效，写为1即可                                                             |
| `period`     | `string` | k线类型,'1d':日线,'1m':分钟线,'3m':三分钟线,'5m':5分钟线,'15m':15分钟线,'30m':30分钟线,'1h':小时线,'1w':周线,'1mon':月线,'1q':季线,'1hy':半年线,'1y':年线 |

**返回值**

list:K线周期（交易日）列表 period为日线时返回如['20170101','20170102',...]样式 其它返回如['20170101010000','20170102020000',...]样式

**示例**

示例返回值

```
# coding:gbk
def init(C):
    pass
def after_init(C):
    print(C.get_trading_dates('600000.SH','','',30,'1d'))
def handlebar(C):
    pass
```

上次更新: 2025/4/22 14:38:1

## 交易下单函数

### passorder - 综合下单函数

综合下单函数，用于股票、期货、期权等下单和新股、新债申购、融资融券等交易操作**推荐使用**

提示

1. 推荐使用
2. 可覆盖多品种下单
3. 注意参数的变化

**调用方法：**

python示例

```
passorder(
    opType, orderType, accountid
    , orderCode, prType, price, volume
    , strategyName, quickTrade, userOrderId
    , ContextInfo
)
'''
passorder(
    2 #opType 操作号
    , 1101 #orderType 组合方式
    , '1000044' #accountid 资金账号
    , 'cu2403.SF' #orderCode 品种代码
    , 14 #prType 报价类型
    , 0.0 #price 价格
    , 2 #volume 下单量
    , '示例下单' #strategyName 策略名称
    , 1 #quickTrade 快速下单标记
    , '投资备注' #userOrderId 投资备注
    , C #ContextInfo 策略上下文
)
'''
```

**参数：**

| 参数名            | 类型       | 说明                  | 提示                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| -------------- | -------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `opType`       | `int`    | 交易类型                | 可选买、买，期货开仓、平仓等<br><br>可选值参考[opType-操作类型](https://dict.thinktrader.net/innerApi/enum_constants.html#optype-%E6%93%8D%E4%BD%9C%E7%B1%BB%E5%9E%8B)                                                                                                                                                                                                                                                                                                                                                  |
| `orderType`    | `int`    | <br><br>下单方式        | 可选值参考[orderType-下单方式](https://dict.thinktrader.net/innerApi/enum_constants.html#ordertype-%E4%B8%8B%E5%8D%95%E6%96%B9%E5%BC%8F)<br><br>可选按股票数量买卖或按照金额等方式买卖<br><br>一、期货不支持 1102 和 1202;<br><br>二、对所有账号组的操作相当于对账号组里的每个账号做一样的操作，如 `passorder (23, 1202, 'testS', '000001. SZ', 5, -1, 50000, ContextInfo)`，意思就是对账号组 `testS` 里的所有账号都以最新价开仓买入 50000 元市值的 `000001.SZ` 平安银行；`passorder (60,1101,"test",'510050. SH', 5,-1,1, ContextInfo)`意思就是账号`test`申购 1 个单位 (900000股)的华夏上证50ETF (只申购不买入成分股)。<br><br> |
| `accountID`    | `string` | 资金账号                | 下单的账号ID（可多个）或账号组名或套利组名（一个篮子一个套利账号，如 accountID = '股票账户名, 期货账号'）                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `orderCode`    | `string` | 下单代码                | 1. 如果是单股或单期货、港股，则该参数填合约代码；<br>2. 如果是组合交易, 则该参数填篮子名称，参考[组合交易](https://dict.thinktrader.net/innerApi/code_examples.html#%E7%BB%84%E5%90%88%E4%BA%A4%E6%98%93-1)；<br>3. 如果是组合套利，则填一个篮子名和一个期货合约名（如orderCode = '篮子名, 期货合约名'），请参考[组合套利交易](https://dict.thinktrader.net/innerApi/code_examples.html#%E7%BB%84%E5%90%88%E5%A5%97%E5%88%A9%E4%BA%A4%E6%98%93-1)<br><br>                                                                                                                                   |
| `prType`       | `int`    | 下单选价类型              | 可选值参考[prType-下单选价类型](https://dict.thinktrader.net/innerApi/enum_constants.html#prtype-%E4%B8%8B%E5%8D%95%E9%80%89%E4%BB%B7%E7%B1%BB%E5%9E%8B)<br><br>特别的对于套利，这个 prType 只对篮子起作用，期货的采用默认的方式）                                                                                                                                                                                                                                                                                                       |
| `price`        | `float`  | 下单价格                | 一、单股下单时，`prType` 是模型价/科创板盘后定价时 `price` 有效；其它情况无效；<br><br>1.1 即单股时， `prType` 参数为 `11`，`49` 时被使用。<br><br>1.2 `prType` 参数不为 `11`，`49` 时也需填写，填写的内容可为 `-1`，`0`，`2`，`100` 等任意数字；<br><br>二、组合下单时，是组合套利时，price 作套利比例有效，其它情况无效。                                                                                                                                                                                                                                                                           |
| `volume`       | `int`    | 下单数量（股 / 手 / 元 / %） | 根据 orderType 值最后一位确定 volume 的单位，可选值参考[volume - 下单](https://dict.thinktrader.net/innerApi/enum_constants.html#volume-%E4%B8%8B%E5%8D%95%E6%95%B0%E9%87%8F)                                                                                                                                                                                                                                                                                                                                        |
| `strategyName` | `string` | 自定义策略名              | <br><br>一、用来区分 `order` 委托和`deal` 成交来自不同的策略。<br><br>根据该策略名，`get_trade_detail_data`，`get_last_order_id` 函数可以获取相应策略名对应的委托或成交集合。<br><br>`strategyName` 只对同账号本地客户端有效，即 `strategyName` 只对当前客户端下的单进行策略区分，且该策略区分只能当前客户端使用。<br><br>                                                                                                                                                                                                                                                                       |
| `quickTrade`   | `int`    | 设定是否立即触发下单          | <br><br>可选值参考[quicktrade - 快速下单](https://dict.thinktrader.net/innerApi/enum_constants.html#quicktrade-%E5%BF%AB%E9%80%9F%E4%B8%8B%E5%8D%95)<br><br>`passorder`是对最后一根K线完全走完后生成的模型信号在下一根K线的第一个tick数据来时触发下单交易；<br><br>采用`quickTrade`参数设置为`1`时，非历史bar上执行时`（ContextInfo.is_last_bar()`为`True`），只要策略模型中调用到就触发下单交易。<br><br>`quickTrade`参数设置为`2`时，不判断bar状态，只要策略模型中调用到就触发下单交易，历史bar上也能触发下单，请谨慎使用。                                                                                                         |
| `userOrderId`  | `string` | 用户自设委托 ID           | 如果传入该参数，<br>则 `strategyName` 和 `quickTrade` 参数也填写。<br>对应 `order` 委托对象和 `deal` 成交对象中的 `m_strRemark` 属性，通过 `get_trade_detail_data` 函数或委托主推函数 `order_callback` 和成交主推函数 `deal_callback` 可拿到这两个对象信息。<br>                                                                                                                                                                                                                                                                                              |
| ContextInfo    | class    | 系统参数                | 含有k线信息和接口的上下文对象                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

**返回：**

无

**更多示例：**

1. [股票](https://dict.thinktrader.net/innerApi/code_examples.html#%E8%82%A1%E7%A5%A8)
2. [基金](https://dict.thinktrader.net/innerApi/code_examples.html#%E5%9F%BA%E9%87%91)
3. [两融](https://dict.thinktrader.net/innerApi/code_examples.html#%E4%B8%A4%E8%9E%8D-1)
4. [期货](https://dict.thinktrader.net/innerApi/code_examples.html#%E6%9C%9F%E8%B4%A7)
5. [期权](https://dict.thinktrader.net/innerApi/code_examples.html#%E6%9C%9F%E6%9D%83)
6. [新股申购](https://dict.thinktrader.net/innerApi/code_examples.html#%E6%96%B0%E8%82%A1%E7%94%B3%E8%B4%AD)
7. [债券](https://dict.thinktrader.net/innerApi/code_examples.html#%E5%80%BA%E5%88%B8)
8. [ETF](https://dict.thinktrader.net/innerApi/code_examples.html#etf)
9. [组合交易](https://dict.thinktrader.net/innerApi/code_examples.html#%E7%BB%84%E5%90%88%E4%BA%A4%E6%98%93)
10. [组合套利交易](https://dict.thinktrader.net/innerApi/code_examples.html#%E7%BB%84%E5%90%88%E5%A5%97%E5%88%A9%E4%BA%A4%E6%98%93)

### algo_passorder - 算法下单（拆单）函数

用于按固定时间间隔和固定规则把目标交易数量拆分成多次下单的交易函数

**调用用法：**

python

```
algo_passorder(opType,orderType,accountid,orderCode,prType,price,volume,[strategyName,quickTrade,userOrderId,userOrderParam],ContextInfo)`
```

提示

算法交易下单，此时使用**交易面板-程序交易-函数交易-函数交易**参数中设置的下单类型(普通交易,算法交易,随机量交易) 如果函数交易参数使用未修改的默认值,此函数和`passorder`函数一致， 设置了函数交易参数后，将会使用函数交易参数的超价等拆单参数，`algo_passorder`内的`prType`若赋值,则优先使用该参数，若`algo_passorder`内的`prType=-1`,将会使用`userOrderParam`内的`opType`，若`userOrderParam`未赋值，则使用界面上的函数交易参数的报价方式

**参数：**  
其他参数同`passorder`，详细解释可参考`passorder`的说明  
`userOrderParam` `dict[str:value]` 是用户自定义交易参数,主要用于修改算法交易的参数 其中`Key` `Value`定义如下

注：所有参数均为非必选

| Key                | Value类型 | Value                                                                                                                                 |
| ------------------ | ------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| OrderType          | int     | 普通交易:`0`<br>算法交易:`1`<br>随机量交易:`2`                                                                                                     |
| PriceType          | int     | 报价方式:数值同passorde prType                                                                                                               |
| MaxOrderCount      | int     | 最大下单次数                                                                                                                                |
| SinglePriceRange   | int     | 波动区间是否单向:<br>否:`0`，<br>是:`1`                                                                                                          |
| PriceRangeType     | int     | 波动区间类型按比例:`0`,按数值`1`                                                                                                                  |
| PriceRangeValue    | float   | 波动区间(按数值)                                                                                                                             |
| PriceRangeRate     | float   | 波动区间(按比例)[`0-1`]                                                                                                                      |
| SuperPriceType     | int     | 单笔超价类型:<br>按比例:`0`<br>按数值`1`                                                                                                          |
| SuperPriceRate     | float   | 单笔超价(按比例)[`0-1`]                                                                                                                      |
| SuperPriceValue    | float   | 单笔超价(按数值)                                                                                                                             |
| VolumeType         | int     | 单笔基准量类型卖1+2+3+4+5量:`0`<br>卖1+2+3+4量:`1`<br>...<br>卖1量:`4`<br>买1量:`5`<br>...<br>买1+2+3+4+5量:`9`<br>目标量:`10`<br>目标剩余量:`11`<br>持仓数量:`12` |
| VolumeRate         | float   | 单笔下单比率[`0-1`]                                                                                                                         |
| SingleNumMin       | float   | 单笔下单量最小值                                                                                                                              |
| SingleNumMax       | float   | 单笔下单量最大值                                                                                                                              |
| ValidTimeType      | int     | 有效时间类型:<br>`0`:按持续时间<br>`1` 按时间区间，默认为0                                                                                                |
| ValidTimeElapse    | int     | 有效持续时间,ValidTimeType设置为0时生效                                                                                                           |
| ValidTimeStart     | int     | 有效开始时间偏移,ValidTimeType设置为1时生效                                                                                                         |
| ValidTimeEnd       | int     | 有效结束时间偏移,ValidTimeType设置为1时生效                                                                                                         |
| UndealtEntrustRule | int     | 未成委托处理数值同prType                                                                                                                       |
| PlaceOrderInterval | int     | 下撤单时间间隔                                                                                                                               |
| UseTrigger         | int     | 是否触价:<br>否:`0`<br>是:`1`                                                                                                               |
| TriggerType        | int     | 触价类型:<br>最新价大于:`1`<br>最新价小于:`2`                                                                                                       |
| TriggerPrice       | float   | 触价价格                                                                                                                                  |
| SuperPriceEnable   | int     | 超价启用笔数                                                                                                                                |

**返回**  
无  
**示例**

python

```
#coding:gbk
userparam = {
    "OrderType": 1,
    "MaxOrderCount": 20,
    "SuperPriceType": 1,
    "SuperPriceValue": 1.12}
accid = '918800000818'  #资金账号
algo_passorder(23,1101,accid,'000001.SZ',5,15,1000,'',1,'strReMark',userparam,ContextInfo)
#表示修改算法交易的最大委托次数为20,单笔下单基准类型为按价格类型超价,单笔超价1.12元,其他参数同函数交易参数中设置
```

### smart_algo_passorder - 智能算法（VWAP 等）函数

提示

1. 调用该函数需要有【智能算法】使用权限

用于使用主动算法或被动算法交易的函数如VWAP TWAP等

**调用方法一：**  

python

```
smart_algo_passorder(opType,orderType,accountid,orderCode,prType,price,volume,strageName,quickTrade,userOrderId,smartAlgoType,limitOverRate,minAmountPerOrder,[targetPriceLevel,startTime,endTime,limitControl],ContextInfo)
```

提示

可选参数可缺省

**参数：**  
其他参数同`passorder`，详细解释可参考[passorder的说明](https://dict.thinktrader.net/innerApi/trading_function.html#passorder-%E7%BB%BC%E5%90%88%E4%B8%8B%E5%8D%95%E5%87%BD%E6%95%B0)

| 参数名               | 类型  | 说明                                                                                                                                                                                                                           | 提示                                                 |
| ----------------- | --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| prType            | int | **可选值**：<br>11:限价（只对单股情况支持,对组合交易不支持）<br>12:市价<br>**特别的对于套利：这个prType只对篮子起作用，期货的采用默认的方式**                                                                                                                                      |                                                    |
| smartAlgoType     | str | 智能算法类型 [[enum_constants#smartAlgoType智能算法类型](https://dict.thinktrader.net/innerApi/enum_constants.html#enum-eordertype-%E7%AE%97%E6%B3%95%E4%BA%A4%E6%98%93%E3%80%81%E6%99%AE%E9%80%9A%E4%BA%A4%E6%98%93%E7%B1%BB%E5%9E%8B)] |                                                    |
| limitOverRate     | int | 量比 数据范围0-100                                                                                                                                                                                                                 | 网格算法无此项<br>若在algoParam中填写量比，则填写范围0-1的小数。           |
| minAmountPerOrder | int | 智能算法最小委托金额，数据范围0-100000                                                                                                                                                                                                      |                                                    |
| targetPriceLevel  | int | 智能算法目标价格,可选值：<br>1：己方盘口 1<br>2：己方盘口2<br>3：己方盘口3<br>4：己方盘口4<br>5：己方盘口5<br>6：最新价<br>7：对方盘口                                                                                                                                     | 一、输入无效值则targetPriceLevel为1<br>二、本项只针对冰山算法,其他算法可缺省。 |
| startTime         | str | 智能算法开始时间                                                                                                                                                                                                                     | 格式"HH:MM:SS"，如"10:30:00"。如果缺省值，则默认为"09:30:00"      |
| endTime           | str | 智能算法截止时间                                                                                                                                                                                                                     | 格式"HH:MM:SS"，如"14:30:00"。如果缺省值，则默认为"15:30:00"      |
| limitControl      | int | 涨跌停控制                                                                                                                                                                                                                        | 默认值为1<br>1：涨停不卖跌停不买<br>0：无限制                       |

**返回**  
无

**示例：**

python

```
#coding:gbk


def init(ContextInfo):
    pass


def after_init(ContextInfo):
    # # 使用smart_algo_passorder 下单
    smart_algo_passorder(
        23,                # 买入
        1101,              # 表示volume的单位是股
        account,           # 资金账号
        '000001.SZ',
        12,                #  11限价，12市价
        0,                 # 限价时，价格填任意数量占位
        50000,             # 5000股
        '',
        2,                 # quickTrade
        '',
        'VWAP',
        25,                 # 量比25%
        0,                  # 智能算法最小委托金额
        1,                  # 智能算法目标价格 本项只针对冰山算法,其他算法可缺省。
        "10:25:00",         # 开始时间
        "14:50:00",         # 结束时间
        1,                  # 涨跌停控制 1为涨停不卖跌停不卖 0 为无限制
        ContextInfo
        )
```

**调用方法二：**  
当时用algoParam时，函数声明为：`smart_algo_passorder(opType,orderType,accountid,orderCode,prType,modelprice,volume,strageName,quickTrade,userid,smartAlgoType,startTime,endTime,algoParam,ContextInfo)`参数均不可缺省  
`smartAlgoType`,`startTime`,`endTime` 含义同上，`algoParam`请使用下面的方法获取：

#### 获取algoParam具体字段

**释义**

获取智能算法参数配置信息

**用法**

python

```
get_smart_algo_param(algoList)
```

**参数**

| 参数         | 类型   | 说明                                      |
| ---------- | ---- | --------------------------------------- |
| `algoList` | list | 需要查询参数配置信息的算法名称列表, 若传空则查询全部有权限的算法参数配置信息 |

**返回**

返回一个字典，键为算法名称，值为参数字典列表。

| 字段                   | 类型     | 说明                                              |
| -------------------- | ------ | ----------------------------------------------- |
| `key`                | string | 参数名称key值,即`smart_algo_order`中`algoList`字典需要传的键值 |
| `name`               | string | 参数名称                                            |
| `dataType`           | string | 参数类型                                            |
| `valueRange`         | string | 参数范围                                            |
| `defaultValue`       | string | 参数默认值                                           |
| `enumName`           | string | 参数枚举值的名称                                        |
| `enumValue`          | string | 参数实际的枚举值                                        |
| `unit`               | string | 参数的单位, 当单位为%时, 值要填写小数而非参数范围所示的百分数值              |
| `valueRangeByName`   | string | 不同算法参数范围                                        |
| `defaultValueByName` | string | 不同算法参数默认值                                       |

**示例**

python

```
#coding:gbk


def init(ContextInfo):
    pass

    # 方法2 使用algoParam 和smart_algo_passorder
    # 该方法部分旧版本客户端可能会不支持
    # algoParam
    # 先获取所有需要传入的参数
    #
    print(get_smart_algo_param(['VWAP']))
    '''
    输出：[2024-01-30 11:21:10][智能算法1][SH000300][日线] 
    {'VWAP': [
        {'key': 'm_dLimitOverRate', 'name': '量比比例', 'dataType': '浮点数', 'valueRange': '0.00-100.00', 'defaultValue': '20.00', 'enumName': '', 'enumValue': '', 'unit': '%', 'valueRangByName': '', 'defaultValueByName': ''}, 
        {'key': 'm_dMinAmountPerOrder', 'name': '委托最小金额', 'dataType': '整数', 'valueRange': '0-100000', 'defaultValue': '0', 'enumName': '', 'enumValue': '', 'unit': '', 'valueRangByName': '', 'defaultValueByName': ''},
        {'key': 'm_dMaxAmountPerOrder', 'name': '委托最大金额', 'dataType': '浮点数', 'valueRange': '0.00-100000000.00', 'defaultValue': '0', 'enumName': '', 'enumValue': '', 'unit': '', 'valueRangByName': '', 'defaultValueByName': ''}, 
        {'key': 'm_nStopTradeForOwnHiLow', 'name': '涨跌停控制', 'dataType': '整数', 'valueRange': '', 'defaultValue': '涨停不卖跌停不买', 'enumName': '无,涨停不卖跌停不买', 'enumValue': '0,1', 'unit': '', 'valueRangByName': '', 'defaultValueByName': ''}, 
        {'key': 'm_dMulitAccountRate', 'name': '多账号总量比', 'dataType': '浮点数', 'valueRange': '0.00-100.00', 'defaultValue': '0', 'enumName': '', 'enumValue': '', 'unit': '%', 'valueRangByName': '', 'defaultValueByName': ''}, 
        {'key': 'm_strCmdRemark', 'name': '投资备注', 'dataType': '字符串', 'valueRange': '', 'defaultValue': '', 'enumName': '', 'enumValue': '', 'unit': '', 'valueRangByName': '', 'defaultValueByName': ''}]}
    '''
    algoParam={
    'm_dLimitOverRate': 0.25,      # 量比 25%
    'm_dMinAmountPerOrder':0,      # 委托最小金额
    'm_dMaxAmountPerOrder':10000,  # 委托最大金额
    'm_nStopTradeForOwnHiLow': 1,  # 涨跌停控制
    'm_dMulitAccountRate':0.30,    # 多账号总量比
    'm_strCmdRemark':  '投资备注1'  # 投资备注
    }
    smart_algo_passorder(
        23,
        1101,
        account,
        '600000.SH',
        12,
        0,
        10000,
        '',
        2,               # quickTrade
        '投资备注',
        'VWAP',
        "10:25:00",      # 开始时间
        "14:50:00",      # 结束时间
        algoParam,       # 算法参数
        ContextInfo
        ) 
```

### cancel-撤销委托

**调用方法**`cancel(orderId, accountId, accountType, ContextInfo)`

**参数**

| 参数名         | 类型     | 含义                                                                                                                      | 说明  |
| ----------- | ------ | ----------------------------------------------------------------------------------------------------------------------- | --- |
| orderId     | string | 委托号                                                                                                                     | 必填  |
| accountID   | string | 资金账号                                                                                                                    | 必填  |
| AccountType | string | 账号类型 可选：<br>'FUTURE'：期货<br>'STOCK'：股票<br>'CREDIT'：信用<br>'HUGANGTONG'：沪港通<br>'SHENGANGTONG'：深港通<br>'STOCK_OPTION'：期权<br> | 必填  |
| ContextInfo | class  | 含有k线信息和接口的上下文对象                                                                                                         | 必填  |

**返回** bool，是否发出了取消委托信号，返回值含义：

> True：是  
> False：否

**示例**

python返回值

```
#coding:gbk
'''
（1）下单前,根据 get_trade_detail_data 函数返回账号的信息，判定资金是否充足，账号是否在登录状态，统计持仓情况等等。
（2）满足一定的模型条件，用 passorder 下单。
（3）下单后，时刻根据 get_last_order_id 函数获取委托和成交的最新id，注意如果委托生成了，就有了委托号（这个id需要自己保存做一个全局控制）。
（4）用该委托号根据 get_value_by_order_id 函数查看委托的状态，各种情况等。
当一个委托的状态变成“已成'后，那么对应的成交 deal 信息就有一条成交数据；用该委托号可查看成交情况。
*注：委托列表和成交列表中的委托号是一样的,都是这个 m_strOrderSysID 属性值。
可用 get_last_order_id 获取最新的 order 的委托号,然后根据这个委托号获取 deal 的信息，当获取成功后，也说明这笔交易是成了，可再根据 position 持仓信息再进一步验证。
（5）根据委托号获取委托信息，根据委托状态，或模型设定，用 cancel 取消委托。
'''


def init(ContextInfo):
    ContextInfo.accid = '6000000248'

def handlebar(ContextInfo):
    if ContextInfo.is_last_bar():
        orderid = get_last_order_id(ContextInfo.accid, 'stock', 'order')
        print(cancel(orderid, ContextInfo.accid, 'stock', ContextInfo))
```

### cancel_task - 撤销任务

**调用方法**`cancel_task(taskId,accountId,accountType,ContextInfo)`

**参数**

| 参数名         | 类型     | 含义                                                                                                                      | 说明  |
| ----------- | ------ | ----------------------------------------------------------------------------------------------------------------------- | --- |
| taskId      | string | 委托号                                                                                                                     | 必填  |
| accountID   | string | 资金账号                                                                                                                    | 必填  |
| AccountType | string | 账号类型 可选：<br>'FUTURE'：期货<br>'STOCK'：股票<br>'CREDIT'：信用<br>'HUGANGTONG'：沪港通<br>'SHENGANGTONG'：深港通<br>'STOCK_OPTION'：期权<br> | 必填  |
| ContextInfo | class  | 含有k线信息和接口的上下文对象                                                                                                         | 必填  |

**返回** bool，是否发出了撤销任务信号，返回值含义：

> True：是
> 
> False：否

**示例**

python

```
#coding:gbk
'''
（1）根据get_trade_detail_data函数返回任务的信息,获取任务编号（m_nTaskId），任务状态等等；
（2）根据任务编号，用cancel_task取消委托。
'''

def init(ContextInfo):
    ContextInfo.accid = '6000000248'

def handlebar(ContextInfo):
    # 获取当前客户端所有的任务
    if ContextInfo.is_last_bar():
        objlist = get_trade_detail_data(ContextInfo.accid,'stock','task')
        for obj in objlist:
            cancel_task(str(obj.m_nTaskId),ContextInfo.accid,'stock',ContextInfo)
```

### pause_task - 暂停任务

暂停智能算法任务

**调用方法** pause_task(taskId,accountId,accountType,ContextInfo)

**参数**

| 参数名         | 类型     | 含义                                                                                                                      | 说明  |
| ----------- | ------ | ----------------------------------------------------------------------------------------------------------------------- | --- |
| taskId      | string | 委托号                                                                                                                     | 必填  |
| accountID   | string | 资金账号                                                                                                                    | 必填  |
| AccountType | string | 账号类型 可选：<br>'FUTURE'：期货<br>'STOCK'：股票<br>'CREDIT'：信用<br>'HUGANGTONG'：沪港通<br>'SHENGANGTONG'：深港通<br>'STOCK_OPTION'：期权<br> | 必填  |
| ContextInfo | class  | 含有k线信息和接口的上下文对象                                                                                                         | 必填  |

**返回** bool，是否发出了暂停任务信号，返回值含义：

> True：是
> 
> False：否

**示例**

python

```
#coding:gbk
'''
（1）根据get_trade_detail_data函数返回任务的信息,获取任务编号（m_nTaskId），任务状态等等；
（2）根据任务编号，用pause_task暂停智能算法任务。
'''

def init(ContextInfo):
    ContextInfo.accid = '6000000248'    

def handlebar(ContextInfo):

    if ContextInfo.is_last_bar():
        # 获取当前客户端所有的任务
        objlist = get_trade_detail_data(ContextInfo.accid,'stock','task')
        for obj in objlist:
            pause_task(obj.m_nTaskId,ContextInfo.accid,'stock',ContextInfo)
```

### resume_task - 继续任务

继续智能算法任务

**调用方法**`resume_task(taskId,accountId,accountType,ContextInfo)`

**参数**

| 参数名         | 类型     | 含义                                                                                                                      | 说明  |
| ----------- | ------ | ----------------------------------------------------------------------------------------------------------------------- | --- |
| taskId      | string | 委托号                                                                                                                     | 必填  |
| accountID   | string | 资金账号                                                                                                                    | 必填  |
| AccountType | string | 账号类型 可选：<br>'FUTURE'：期货<br>'STOCK'：股票<br>'CREDIT'：信用<br>'HUGANGTONG'：沪港通<br>'SHENGANGTONG'：深港通<br>'STOCK_OPTION'：期权<br> | 必填  |
| ContextInfo | class  | 含有k线信息和接口的上下文对象                                                                                                         | 必填  |

**返回** bool，是否发出了重启任务信号，返回值含义：

> True：是
> 
> False：否

**示例**

python

```
#coding:gbk
'''
（1）根据get_trade_detail_data函数返回任务的信息,获取任务编号（m_nTaskId），任务状态等等；
（2）根据任务编号，用resume_task启动已暂停智能算法任务。
'''

def init(ContextInfo):
    ContextInfo.accid = '6000000248'    
def handlebar(ContextInfo):
    if ContextInfo.is_last_bar():
        # 获取当前客户端所有的任务
        objlist = get_trade_detail_data(ContextInfo.accid,'stock','task')
        for obj in objlist:
            resume_task(obj.m_nTaskId,ContextInfo.accid,'stock',ContextInfo)
```

### get_basket-获取股票篮子

**用法：** get_basket(basketName)

**释义：** 获取股票篮子

**参数：**

- basketName：股票篮子名称

**示例：**

```
print( get_basket('basket1') )
```

### set_basket-设置股票篮子

**用法：** set_basket(basketDict)

**释义：** 设置passorder的股票篮子,仅用于passorder进行篮子交易,设置成功后,用get_basket可以取出后即可进行passorder组合交易下单

**参数：**

- basketDict：股票篮子 {'name':股票篮子名称,'stocks':[{'stock':股票名称,'weight',权重,'quantity':数量,'optType':交易类型}]} 。

**示例：**

```
table=[
    {'stock':'600000.SH','weight':0.11,'quantity':100,'optType':23},
    {'stock':'600028.SH','weight':0.11,'quantity':200,'optType':24},
]
basket={'name':'basket1','stocks':table}
set_basket(basket)
#一键买卖2份(2101代表用篮子里quantity字段)basket1里面的股票组合，即600000.SH买入200股，600028.SH卖出400股
passorder(35,2101,ContextInfo.accid,'basket1',5,-1,2,'basketOrder',2,'basketOrder',ContextInfo)
```

## 交易查询函数

### get_trade_detail_data-查询账号资金信息、委托记录等

**调用方法** `get_trade_detail_data(accountID, strAccountType, strDatatype, strategyName)`  
或不区分策略  
`get_trade_detail_data(accountID, strAccountType, strDatatype)`

**参数**

| 参数名            | 类型     | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | 备注                           |
| -------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| accountID      | string | 资金账号                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | 必填                           |
| strAccountType | string | 账号类型 可选：<br>'FUTURE'：期货<br>'STOCK'：股票<br>'CREDIT'：信用<br>'HUGANGTONG'：沪港通<br>'SHENGANGTONG'：深港通<br>'STOCK_OPTION'：期权<br>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 必填                           |
| strDatatype    | string | 要查询数据类型 可选：<br>`ACCOUNT`：[账号对象](https://dict.thinktrader.net/innerApi/data_structure.html#account-%E8%B4%A6%E6%88%B7%E5%AF%B9%E8%B1%A1)或[信用账号对象](https://dict.thinktrader.net/innerApi/data_structure.html#ccreditaccountdetail-%E4%BF%A1%E7%94%A8%E8%B4%A6%E5%8F%B7%E5%AF%B9%E8%B1%A1-%E9%9D%9E%E6%9F%A5%E6%9F%9C%E5%8F%B0)<br>`POSITION`：[持仓](https://dict.thinktrader.net/innerApi/data_structure.html#position-statistics-%E6%8C%81%E4%BB%93%E7%BB%9F%E8%AE%A1%E5%AF%B9%E8%B1%A1)<br>`POSITION_STATISTICS`：[持仓统计](https://dict.thinktrader.net/innerApi/data_structure.html#position-statistics-%E6%8C%81%E4%BB%93%E7%BB%9F%E8%AE%A1%E5%AF%B9%E8%B1%A1)<br>`ORDER`：[委托](https://dict.thinktrader.net/innerApi/data_structure.html#order-%E5%A7%94%E6%89%98%E5%AF%B9%E8%B1%A1)<br>`DEAL` ：[成交](https://dict.thinktrader.net/innerApi/data_structure.html#deal-%E6%88%90%E4%BA%A4%E5%AF%B9%E8%B1%A1)<br>`TASK`：[任务](https://dict.thinktrader.net/innerApi/data_structure.html#ctaskdetail-%E4%BB%BB%E5%8A%A1%E5%AF%B9%E8%B1%A1) | 必填                           |
| strategyName   | string | 策略 当用`passorder`下单时指定了strategyName 参数时，当查询成交和委托时传入同样的`strageName`，则可以只返回包含`strategyName`的委托子集或成交子集                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | `strategyName`参数只对成交和委托有效,选填 |

**返回** list，list 中放的是对应strDatatype的 Python对象，通过 dir(pythonobj) 可返回某个对象的属性列表。

有五种交易相关信息，包括：

ACCOUNT：[账号对象](https://dict.thinktrader.net/innerApi/data_structure.html#account-%E8%B4%A6%E6%88%B7%E5%AF%B9%E8%B1%A1)或[信用账号对象](https://dict.thinktrader.net/innerApi/data_structure.html#ccreditaccountdetail-%E4%BF%A1%E7%94%A8%E8%B4%A6%E5%8F%B7%E5%AF%B9%E8%B1%A1-%E9%9D%9E%E6%9F%A5%E6%9F%9C%E5%8F%B0)

POSITION：[持仓明细](https://dict.thinktrader.net/innerApi/data_structure.html#position-%E6%8C%81%E4%BB%93%E5%AF%B9%E8%B1%A1)

POSITION_STATISTICS: [持仓统计](https://dict.thinktrader.net/innerApi/data_structure.html#position-statistics-%E6%8C%81%E4%BB%93%E7%BB%9F%E8%AE%A1%E5%AF%B9%E8%B1%A1)

ORDER：[委托](https://dict.thinktrader.net/innerApi/data_structure.html#order-%E5%A7%94%E6%89%98%E5%AF%B9%E8%B1%A1)

DEAL：[成交](https://dict.thinktrader.net/innerApi/data_structure.html#deal-%E6%88%90%E4%BA%A4%E5%AF%B9%E8%B1%A1)

TASK：[任务](https://dict.thinktrader.net/innerApi/data_structure.html#ctaskdetail-%E4%BB%BB%E5%8A%A1%E5%AF%B9%E8%B1%A1)

**示例：**

python返回值

```
#coding:gbk

account = '800174' # 在策略交易界面运行时，account的值会被赋值为策略配置中的账号，编辑器界面运行时，需要手动赋值；编译器环境里执行的下单函数不会产生实际委托

def init(ContextInfo):
    pass

def handlebar(ContextInfo):
    if not ContextInfo.is_last_bar():
        return

    orders = get_trade_detail_data(account, 'stock', 'order')
    print('查询委托结果：')
    for o in orders:
        print(f'股票代码: {o.m_strInstrumentID}, 市场类型: {o.m_strExchangeID}, 证券名称: {o.m_strInstrumentName}, 买卖方向: {o.m_nOffsetFlag}',
        f'委托数量: {o.m_nVolumeTotalOriginal}, 成交均价: {o.m_dTradedPrice}, 成交数量: {o.m_nVolumeTraded}, 成交金额:{o.m_dTradeAmount}')


    deals = get_trade_detail_data(account, 'stock', 'deal')
    print('查询成交结果：')
    for dt in deals:
        print(f'股票代码: {dt.m_strInstrumentID}, 市场类型: {dt.m_strExchangeID}, 证券名称: {dt.m_strInstrumentName}, 买卖方向: {dt.m_nOffsetFlag}', 
        f'成交价格: {dt.m_dPrice}, 成交数量: {dt.m_nVolume}, 成交金额: {dt.m_dTradeAmount}')

    positions = get_trade_detail_data(account, 'stock', 'position')
    print('查询持仓结果：')
    for dt in positions:
        print(f'股票代码: {dt.m_strInstrumentID}, 市场类型: {dt.m_strExchangeID}, 证券名称: {dt.m_strInstrumentName}, 持仓量: {dt.m_nVolume}, 可用数量: {dt.m_nCanUseVolume}',
        f'成本价: {dt.m_dOpenPrice:.2f}, 市值: {dt.m_dInstrumentValue:.2f}, 持仓成本: {dt.m_dPositionCost:.2f}, 盈亏: {dt.m_dPositionProfit:.2f}')


    accounts = get_trade_detail_data(account, 'stock', 'account')
    print('查询账号结果：')
    for dt in accounts:
        print(f'总资产: {dt.m_dBalance:.2f}, 净资产: {dt.m_dAssureAsset:.2f}, 总市值: {dt.m_dInstrumentValue:.2f}', 
        f'总负债: {dt.m_dTotalDebit:.2f}, 可用金额: {dt.m_dAvailable:.2f}, 盈亏: {dt.m_dPositionProfit:.2f}')

    position_statistics = get_trade_detail_data(account,"FUTURE",'POSITION_STATISTICS')
    for obj in position_statistics:
        if obj.m_nDirection == 49:
            continue
        PositionInfo_dict[obj.m_strInstrumentID+"."+obj.m_strExchangeID]={
        "持仓":obj.m_nPosition,
        "成本":obj.m_dPositionCost,
        "浮动盈亏":obj.m_dFloatProfit,
        "保证金占用":obj.m_dUsedMargin
        }
    print(PositionInfo_dict)
```

### get_history_trade_detail_data - 查询历史交易明细

**用法：** get_history_trade_detail_data(accountID,strAccountType,strDatatype,strStratDate,strEndDate);

**释义：** 获取历史成交明细数据，返回结果为一个([timetag,obj...])的元组

**参数：**

accountID：string,账号； strAccountType：string,账号类型,有"FUTURE","STOCK","CREDIT","HUGANGTONG","SHENGANGTONG","STOCK_OPTION"； strDatatype：string,交易明细数据类型,有：持仓"POSITION"、委托"ORDER"、成交"DEAL"； strStratDate：string,开始时间,如'20240513'； strEndDate：string,结束时间,如'20240514'；

**返回：**list,list中放的是PythonObj,通过dir(pythonobj)可返回某个对象的属性列表 **示例：**

示例

```
def handlebar(ContextInfo):
    obj_list = get_history_trade_detail_data('6000000248','stock','position','20240513','20240514')
    for time,data in obj_list:
        for obj in data:
            print(obj.m_strInstrumentID)
            print(dir(obj))#查看有哪些属性字段
```

### get_ipo_data-获取当日新股新债信息

**用法：** get_ipo_data([,type])

**释义：** 获取当日新股新债信息，返回结果为一个字典,包括新股申购代码,申购名称,最大申购数量,最小申购数量等数据

**参数：**

- type：为空时返回新股新债信息，type="STOCK"时只返回新股申购信息，type="BOND"时只返回新债申购信息

**示例：**

```
#coding:gbk
def init(ContextInfo):
    ipoData=get_ipo_data()# 返回新股新债信息
    ipoStock=get_ipo_data("STOCK")# 返回新股信息
    ipoCB=get_ipo_data("BOND")# 返回新债申购信息
```

### get_new_purchase_limit-获取账户新股申购额度

**用法：** get_new_purchase_limit(accid)

**释义：** 获取账户新股申购额度，返回结果为一个字典,包括上海主板,深圳市场,上海科创版的申购额度

**参数：**

- accid：资金账号，必须时股票账号或者信用账号

**示例：**

```
def init(ContextInfo):
    ContextInfo.accid="10000001"# 返回新股新债信息
    purchase_limit=get_new_purchase_limit(ContextInfo.accid)
```

### get_value_by_order_id-根据委托号获取委托或成交信息

**调用方法**`get_value_by_order_id(orderId, accountID, strAccountType, strDatatype)`

**参数**

| 参数名            | 类型     | 含义                                                                                                                      | 说明  |
| -------------- | ------ | ----------------------------------------------------------------------------------------------------------------------- | --- |
| orderId        | string | 委托号                                                                                                                     | 必填  |
| accountID      | string | 资金账号                                                                                                                    | 必填  |
| strAccountType | string | 账号类型 可选：<br>'FUTURE'：期货<br>'STOCK'：股票<br>'CREDIT'：信用<br>'HUGANGTONG'：沪港通<br>'SHENGANGTONG'：深港通<br>'STOCK_OPTION'：期权<br> | 必填  |
| strDatatype    | string | 要查询数据类型 可选：<br>'ORDER'：委托<br>'DEAL' ：成交                                                                                 | 必填  |

**返回**

[委托对象](https://dict.thinktrader.net/innerApi/trading_function.html?id=7zqjlm) 或 [成交对象](https://dict.thinktrader.net/innerApi/trading_function.html?id=7zqjlm)

**示例**

python返回值

```
def init(ContextInfo):
    ContextInfo.accid = '6000000248'

def handlebar(ContextInfo):
    orderid = get_last_order_id(ContextInfo.accid, 'stock', 'order')
    print(orderid)
    obj = get_value_by_order_id(orderid,ContextInfo.accid, 'stock', 'order')
    print(obj.m_strInstrumentID)
```

**调用方法**

python

```
# 区分策略，添加策略名称参数 strategyName
get_last_order_id(accountID, strAccountType, strDatatype, strategyName)

# 不区分策略
get_last_order_id(accountID, strAccountType, strDatatype)
```

**参数**

| 参数名            | 类型     | 含义                                                                                                                      | 说明  |
| -------------- | ------ | ----------------------------------------------------------------------------------------------------------------------- | --- |
| accountID      | string | 资金账号                                                                                                                    | 必填  |
| strAccountType | string | 账号类型 可选：<br>'FUTURE'：期货<br>'STOCK'：股票<br>'CREDIT'：信用<br>'HUGANGTONG'：沪港通<br>'SHENGANGTONG'：深港通<br>'STOCK_OPTION'：期权<br> | 必填  |
| strDatatype    | string | 要查询数据类型 可选：<br>'ORDER'：委托<br>'DEAL' ：成交                                                                                 | 必填  |
| strategyName   | string | 策略 当用passorder下单时指定了strategyName 参数时，当查询成交和委托时传入同样的strageName，则可以只返回包含strategyName的委托子集或成交子集                            | 选填  |

**返回**

String，委托号，如果没找到返回 '-1'。

**示例**

```
def init(ContextInfo):
    ContextInfo.accid = '6000000248'

def handlebar(ContextInfo):
    orderid = get_last_order_id(ContextInfo.accid, 'stock', 'order')
    print(orderid)
    obj = get_value_by_order_id(orderid,ContextInfo.accid, 'stock', 'order')
    print(obj.m_strInstrumentID)
```

### get_assure_contract-获取两融担保标的明细

**用法：** get_assure_contract(accId)

**释义：** 获取信用账户担保合约明细

**参数：**

- accId：信用账户

**返回：** list，list 中放的是 [StkSubjects](https://dict.thinktrader.net/innerApi/data_structure.html#stkcompacts-%E8%B4%9F%E5%80%BA%E5%90%88%E7%BA%A6%E5%AF%B9%E8%B1%A1)，通过 dir(pythonobj) 可返回某个对象的属性列表。

**示例：**

python

```
def show_data(data):
    tdata = {}
    for ar in dir(data):
        if ar[:2] != 'm_':continue
        try:
            tdata[ar] = data.__getattribute__(ar)
        except:
            tdata[ar] = '<CanNotConvert>'
    return tdata

def handlebar(ContextInfo): 
    obj = get_assure_contract('6000000248')
    for i in obj[:3]:
        print(show_data(i))


"""
{'m_dAssureRatio': 0.0, # 担保品折算比例
'm_dFinRatio': 0.8, # 融资保证金比例
'm_dSloRatio': 1.0,  # 融券保证金比例
'm_eAssureStatus': 50,  # 是否可做担保
'm_eCreditFundCtl': 50, # 融资交易控制
'm_eCreditStkCtl': 50, # 融券交易控制
'm_eFinStatus': 48, # 融资状态
'm_eSloStatus': 48, # 融券状态
'm_nPlatformID': 10064,  # 平台号
'm_strAccountID': '95000857',  # 资金账号
'm_strBrokerID': '003', # 经纪公司编号
'm_strBrokerName': '光大证券信用',  # 证券公司
'm_strExchangeID': 'SH', # 交易所
'm_strInstrumentID':'510150' # 证券代码
"""
```

### get_enable_short_contract-获取可融券明细

提示

注:由于字段m_dSloRatio、m_dSloStatus提供来源和取担保品明细([get_assure_contract](https://dict.thinktrader.net/innerApi/innerApi/trading_function.html#get-assure-contract-%E8%8E%B7%E5%8F%96%E4%B8%A4%E8%9E%8D%E6%8B%85%E4%BF%9D%E6%A0%87%E7%9A%84%E6%98%8E%E7%BB%86))重复，字段在2021年9月移除，后续用担保品明细接口获取,具体见 [担保标的对象字段说明](https://dict.thinktrader.net/innerApi/data_structure.html#stkcompacts-%E8%B4%9F%E5%80%BA%E5%90%88%E7%BA%A6%E5%AF%B9%E8%B1%A1)

**用法：** get_enable_short_contract(accId)

**释义：** 获取信用账户当前可融券的明细

**参数：**

- accId：信用账户

**返回：** list，list 中放的是 [CreditSloEnableAmount](https://dict.thinktrader.net/innerApi/data_structure.html#creditsloenableamount-%E5%8F%AF%E8%9E%8D%E5%88%B8%E6%98%8E%E7%BB%86%E5%AF%B9%E8%B1%A1)，通过 dir(pythonobj) 可返回某个对象的属性列表。

**示例：**

python

```
def show_data(data):
    tdata = {}
    for ar in dir(data):
        if ar[:2] != 'm_':continue
        try:
            tdata[ar] = data.__getattribute__(ar)
        except:
            tdata[ar] = '<CanNotConvert>'
    return tdata

def handlebar(ContextInfo):
    obj = get_enable_short_contract('6000000248')
    for i in obj[:3]:
        print(show_data(i))

"""
Rerutn:

{'m_eQuerySloType': 48, # 查询类型
'm_nEnableAmount': 0,  # 融券可融数量
'm_nPlatformID': 10064,  # 平台号
'm_strAccountID': '95000857',  # 资金账号
'm_strBrokerID': '003',  # 经纪公司编号
'm_strBrokerName': '光大证券信用',  # 证券公司
'm_strExchangeID': 'SH', # 标的市场
'm_strInstrumentID': '688321' # 证券代码
}

"""
```

### query_credit_account - 查询信用账户明细

注意

1. 本函数一次最多查询200只股票的两融最大下单量，且同时只能有一个查询,如果前面的查询正在进行中,后面的查询将会提前返回。本函数从服务器查询数据,建议平均查询时间间隔180s一次,不可频繁调用。
2. 该函数必须配合credit_account_callback回调才能使用，关于此回调的说明请看[credit_account_callback](https://dict.thinktrader.net/innerApi/callback_function.html?id=7zqjlm#credit-account-callback-%E6%9F%A5%E8%AF%A2%E4%BF%A1%E7%94%A8%E8%B4%A6%E6%88%B7%E6%98%8E%E7%BB%86%E5%9B%9E%E8%B0%83)
3. callback返回的对象是[CCreditAccountDetail](https://dict.thinktrader.net/innerApi/data_structure.html?id=7zqjlm#ccreditaccountdetail-%E4%BF%A1%E7%94%A8%E8%B4%A6%E5%8F%B7%E5%AF%B9%E8%B1%A1-%E9%9D%9E%E6%9F%A5%E6%9F%9C%E5%8F%B0)

调用query_credit_account，该接口的查询结果将会推送给credit_account_callback，所以程序里需要按照函数参数实现函数`credit_account_callback`,callback返回的对象是[CCreditAccountDetail](https://dict.thinktrader.net/innerApi/data_structure.html?id=7zqjlm#ccreditaccountdetail-%E4%BF%A1%E7%94%A8%E8%B4%A6%E5%8F%B7%E5%AF%B9%E8%B1%A1-%E9%9D%9E%E6%9F%A5%E6%9F%9C%E5%8F%B0)

**用法：** query_credit_account(accountId,seq,ContextInfo)  
**释义：** 查询信用账户明细。本函数只能有一个查询，如果前面的查询正在进行中，后面的查询将会提前返回。

**参数：**

- accountId：string，查询的两融账号

- seq：int，查询序列号，建议输入唯一值以便对应结果回调

**示例：**

python返回值

```
#coding:gbk


import time

def init(ContextInfo):
    ContextInfo.accid='200133'

def handlebar(ContextInfo):
    if ContextInfo.is_last_bar():
        query_credit_account(ContextInfo.accid,int(time.time()),ContextInfo)
# 该函数必须配合credit_account_callback回调才能使用
def credit_account_callback(ContextInfo,seq,result):
    print(seq)
    print(f":维持担保比例:{result.m_dPerAssurescaleValue:.2f},总负债:{result.m_dTotalDebt:.2f}")
```

**回调示例** 见[query_credit_account](https://dict.thinktrader.net/innerApi/code_examples.html?id=7zqjlm#%E8%8E%B7%E5%8F%96%E4%B8%A4%E8%9E%8D%E8%B4%A6%E5%8F%B7%E4%BF%A1%E6%81%AF%E7%A4%BA%E4%BE%8B)

### query_credit_opvolume - 查询两融最大可下单量

注意

1. 本函数一次最多查询200只股票的两融最大下单量，且同时只能有一个查询,如果前面的查询正在进行中,后面的查询将会提前返回。本函数从服务器查询数据,建议平均查询时间间隔180s一次,不可频繁调用。
2. 该函数必须配合credit_opvolume_callback回调才能使用,关于此回调的说明请看[credit_account_callback](https://dict.thinktrader.net/innerApi/callback_function.html?id=7zqjlm#credit-opvolume-callback-%E6%9F%A5%E8%AF%A2%E4%B8%A4%E8%9E%8D%E6%9C%80%E5%A4%A7%E5%8F%AF%E4%B8%8B%E5%8D%95%E9%87%8F%E7%9A%84%E5%9B%9E%E8%B0%83)

调用query_credit_opvolume，该接口的查询结果将会推送给credit_opvolume_callback，所以必须配合credit_opvolume_callback回调才能使用

**用法：** query_credit_opvolume(accountId,stockCode,opType,prType,price,seq,ContextInfo)

**释义：** 查询两融最大可下单量。

**参数：**

- `accountId`:查询的两融账号
- `stockCode`:需要查询的股票代码,stockCode为List的类型,可以查询多只股票
- `opType`:两融下单类型,同`passorder`的下单类型
- `prType`:报单价格类型,同`passorder`的报价类型
- `seq`:查询序列号,int型，建议输入唯一值以便对应结果回调
- `price`:报价(非限价单可以填任意值),**如果`stockCode`为`List`类型,报价也需要为长度相同的List**
- `ContextInfo`:ContextInfo类

**示例：**

python返回值

```
#coding:gbk

import time

def init(ContextInfo):
    ContextInfo.accid='200133'

def handlebar(ContextInfo):
    if ContextInfo.is_last_bar():
        #查询accid账号担保品买入600000,SH限价10元的最大可下单量
        query_credit_opvolume(ContextInfo.accid,'600000.SH',33,11,10,int(time.time()),C) # 查询两融最大可下单量。
        time.sleep(0.5)
        #查询accid账号担保品买入600000,SH限价10元,000001.SZ担保品买入限价20元的最大可下单量
        query_credit_opvolume(ContextInfo.accid,["600000.SH","000001.SZ"],33,11,[10,20],int(time.time()),C) # 查询两融最大可下单量。

# 该函数必须配合credit_opvolume_callback回调才能使用
def credit_opvolume_callback(ContextInfo,accid,seq,ret,result):
    print(seq)
    print(f'查询结果:{ret}') # 正常返回:1,正在查询中-1,输入账号非法:-2,输入查询参数非法:-3,超时等服务器返回报错:-4
    print(result)
```

### get_option_subject_position-取期权标的持仓

**用法：** get_option_subject_position(accountID)

**释义：** 取期权标的持仓

**参数：**

- accountID：string,账号

**返回：** list,list中放的是[CLockPosition](https://dict.thinktrader.net/innerApi/data_structure.html?id=7zqjlm#clockposition-%E6%9C%9F%E6%9D%83%E6%A0%87%E7%9A%84%E6%8C%81%E4%BB%93),通过dir(pythonobj)可返回某个对象的属性列表

**示例：**

```
data=get_option_subject_position('880399990383')
print(len(data));
forobjindata:
    print(obj.m_strInstrumentName,obj.m_lockVol,obj.m_coveredVol);
```

### get_comb_option-取期权组合持仓

**用法：** get_comb_option(accountID)

**释义：** 取期权组合持仓

**参数：**

- accountID：string,账号

**返回：** list,list中放的是[CStkOptCombPositionDetail](https://dict.thinktrader.net/innerApi/data_structure.html?id=7zqjlm#cstkoptcombpositiondetail-%E6%9C%9F%E6%9D%83%E7%BB%84%E5%90%88%E6%8C%81%E4%BB%93) ,通过dir(pythonobj)可返回某个对象的属性列表

**示例：**

```
obj_list=get_comb_option('880399990383')
print(len(obj_list));
forobjinobj_list:
    print(obj.m_strCombCodeName,obj.m_strCombID,obj.m_nVolume,obj.m_nFrozenVolume)
```

### get_unclosed_compacts-获取未了结负债合约明细

**用法：** get_unclosed_compacts(accountID,accountType)

**释义：** 获取未了结负债合约明细

**参数：**

- accountID：str，资金账号
- accountType：str，账号类型，这里应该填'CREDIT'

**返回：**

list([ CStkUnclosedCompacts, ... ]) 负债列表，CStkUnclosedCompacts属性如下：

| 字段名称                      | 类型     | 说明                                                                                |
| ------------------------- | ------ | --------------------------------------------------------------------------------- |
| `m_strAccountID`          | string | 账号ID                                                                              |
| `m_nBrokerType`           | int    | 账号类型<br>1-期货账号<br>2-股票账号<br>3-信用账号<br>5-期货期权账号<br>6-股票期权账号<br>7-沪港通账号<br>11-深港通账号 |
| `m_strExchangeID`         | string | 市场                                                                                |
| `m_strInstrumentID`       | string | 证券代码                                                                              |
| `m_eCompactType`          | int    | 合约类型<br>32-不限制<br>48-融资<br>49-融券                                                  |
| `m_eCashgroupProp`        | int    | 头寸来源<br>32-不限制<br>48-普通头寸<br>49-专项头寸                                              |
| `m_nOpenDate`             | int    | 开仓日期(如'20201231')                                                                 |
| `m_nBusinessVol`          | int    | 合约证券数量                                                                            |
| `m_nRealCompactVol`       | int    | 未还合约数量                                                                            |
| `m_nRetEndDate`           | int    | 到期日(如'20201231')                                                                  |
| `m_dBusinessBalance`      | float  | 合约金额                                                                              |
| `m_dBusinessFare`         | float  | 合约息费                                                                              |
| `m_dRealCompactBalance`   | float  | 未还合约金额                                                                            |
| `m_dRealCompactFare`      | float  | 未还合约息费                                                                            |
| `m_dRepaidFare`           | float  | 已还息费                                                                              |
| `m_dRepaidBalance`        | float  | 已还金额                                                                              |
| `m_strCompactId`          | string | 合约编号                                                                              |
| `m_strEntrustNo`          | string | 委托编号                                                                              |
| `m_nRepayPriority`        | int    | 偿还优先级                                                                             |
| `m_strPositionStr`        | string | 定位串                                                                               |
| `m_eCompactRenewalStatus` | int    | 合约展期状态<br>48-可申请<br>49-已申请<br>50-审批通过<br>51-审批不通过<br>52-不可申请<br>53-已执行<br>54-已取消  |
| `m_nDeferTimes`           | int    | 展期次数                                                                              |

**示例：**

```
get_unclosed_compacts('6000000248', 'CREDIT')
```

### get_closed_compacts-获取已了结负债合约明细

**用法：** get_closed_compacts(accountID,accountType)

**释义：** 获取已了结负债合约明细

**参数：**

- accountID：str，资金账号
- accountType：str，账号类型，这里应该填'CREDIT'

**返回：**

list([ CStkUnclosedCompacts, ... ]) 负债列表，CStkUnclosedCompacts属性如下：

| 字段名                  | 类型     | 描述                                                                                |
| -------------------- | ------ | --------------------------------------------------------------------------------- |
| `m_strAccountID`     | string | 账号ID                                                                              |
| `m_nBrokerType`      | int    | 账号类型<br>1-期货账号<br>2-股票账号<br>3-信用账号<br>5-期货期权账号<br>6-股票期权账号<br>7-沪港通账号<br>11-深港通账号 |
| `m_strExchangeID`    | string | 市场                                                                                |
| `m_strInstrumentID`  | string | 证券代码                                                                              |
| `m_eCompactType`     | int    | 合约类型<br>32-不限制<br>48-融资<br>49-融券                                                  |
| `m_eCashgroupProp`   | int    | 头寸来源<br>32-不限制<br>48-普通头寸<br>49-专项头寸                                              |
| `m_nOpenDate`        | int    | 开仓日期(如'20201231')                                                                 |
| `m_nBusinessVol`     | int    | 合约证券数量                                                                            |
| `m_nRetEndDate`      | int    | 到期日(如'20201231')                                                                  |
| `m_nDateClear`       | int    | 了结日期(如'20201231')                                                                 |
| `m_nEntrustVol`      | int    | 委托数量                                                                              |
| `m_dEntrustBalance`  | float  | 委托金额                                                                              |
| `m_dBusinessBalance` | float  | 合约金额                                                                              |
| `m_dBusinessFare`    | float  | 合约息费                                                                              |
| `m_dRepaidFare`      | float  | 已还息费                                                                              |
| `m_dRepaidBalance`   | float  | 已还金额                                                                              |
| `m_strCompactId`     | string | 合约编号                                                                              |
| `m_strEntrustNo`     | string | 委托编号                                                                              |
| `m_strPositionStr`   | string | 定位串                                                                               |

**示例：**

```
get_closed_compacts('6000000248', 'CREDIT')
```

## 其他交易函数（仅回测可用）

警告

以下函数仅回测生效，实盘和模拟盘交易均不可用

### order_lots-指定手数交易

**用法：** order_lots(stockcode, lots[, style, price], ContextInfo[, accId])

**释义：** 指定手数交易，指定手数发送买/卖单。如有需要落单类型当做一个参量传入，如果忽略掉落单类型，那么默认以最新价下单。

**参数：**

- stockcode：代码，string，如 '000002.SZ'

- lots：手数，int

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定 选此参数时必须指定有效的`price`参数，其他style值可不用传入price参数
  > 
  > 'HANG'：挂单 用己方盘口挂单，即买入时用盘口买一价下单，卖出时用卖一价挂单，
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE5', 'SALE4', 'SALE3', 'SALE2', 'SALE1'：卖5-1价
  > 
  > 'BUY1', 'BUY2', 'BUY3', 'BUY4', 'BUY5'：买1-5价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价下 1 手买入
    order_lots('000002.SZ', 1, ContextInfo, '600000248')

    # 用对手价下 1 手卖出
    order_lots('000002.SZ', -1, 'COMPETE', ContextInfo, '600000248')

    # 用指定价 37.5 下 2 手卖出
    order_lots('000002.SZ', -2, 'fix', 37.5, ContextInfo, '600000248')
```

### order_value-指定价值交易

**用法：** order_value(stockcode, value[, style, price], ContextInfo[, accId])

**释义：** 指定价值交易，使用想要花费的金钱买入 / 卖出股票，而不是买入 / 卖出想要的股数，正数代表买入，负数代表卖出。股票的股数总是会被调整成对应的 100 的倍数（在中国 A 股市场 1 手是 100 股）。当您提交一个卖单时，该方法代表的意义是您希望通过卖出该股票套现的金额，如果金额超出了您所持有股票的价值，那么您将卖出所有股票。需要注意，如果资金不足，该 API 将不会创建发送订单。

**参数：**

- stockcode：代码，string，如 '000002.SZ'

- value：金额（元），double

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE5', 'SALE4', 'SALE3', 'SALE2', 'SALE1'：卖5-1价
  > 
  > 'BUY1', 'BUY2', 'BUY3', 'BUY4', 'BUY5'：买1-5价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价下 10000 元买入
    order_value('000002.SZ', 10000, ContextInfo, '600000248')

    # 用对手价下 10000 元卖出
    order_value('000002.SZ', -10000, 'COMPETE', ContextInfo, '600000248')

    # 用指定价 37.5 下 20000 元卖出
    order_value('000002.SZ', -20000, 'fix', 37.5, ContextInfo, '600000248')
```

### order_percent-指定比例交易

**用法：** order_percent(stockcode, percent[, style, price], ContextInfo[, accId])

**释义：** 指定比例交易，发送一个等于目前投资组合价值（市场价值和目前现金的总和）一定百分比的买 / 卖单，正数代表买，负数代表卖。股票的股数总是会被调整成对应的一手的股票数的倍数（1 手是 100 股）。百分比是一个小数，并且小于或等于1（小于等于100%），0.5 表示的是 50%。需要注意，如果资金不足，该 API 将不会创建发送订单。

**参数：**

- stockcode：代码，string，如 '000002.SZ'

- percent：比例，double

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE5', 'SALE4', 'SALE3', 'SALE2', 'SALE1'：卖5-1价
  > 
  > 'BUY1', 'BUY2', 'BUY3', 'BUY4', 'BUY5'：买1-5价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价下 5.1% 价值买入
    order_percent('000002.SZ', 0.051, ContextInfo, '600000248')

    # 用对手价下 5.1% 价值卖出
    order_percent('000002.SZ', -0.051, 'COMPETE', ContextInfo, '600000248')

    # 用指定价 37.5 下 10.2% 价值卖出
    order_percent('000002.SZ', -0.102, 'fix', 37.5, ContextInfo, '600000248')
```

### order_target_value-指定目标价值交易

**用法：** order_target_value(stockcode, tar_value[, style, price], ContextInfo[, accId])

**释义：** 指定目标价值交易，买入 / 卖出并且自动调整该证券的仓位到一个目标价值。如果还没有任何该证券的仓位，那么会买入全部目标价值的证券；如果已经有了该证券的仓位，则会买入 / 卖出调整该证券的现在仓位和目标仓位的价值差值的数目的证券。需要注意，如果资金不足，该API将不会创建发送订单。

**参数：**

- stockcode：代码，string，如 '000002.SZ'

- tar_value：目标金额（元），double，非负数

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE5', 'SALE4', 'SALE3', 'SALE2', 'SALE1'：卖5-1价
  > 
  > 'BUY1', 'BUY2', 'BUY3', 'BUY4', 'BUY5'：买1-5价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价下调仓到 10000 元持仓   
    order_target_value('000002.SZ', 10000, ContextInfo, '600000248')

    # 用对手价调仓到 10000 元持仓   
    order_target_value('000002.SZ', 10000, 'COMPETE', ContextInfo, '600000248')

    # 用指定价 37.5 下调仓到 20000 元持仓
    order_target_value('000002.SZ', 20000, 'fix', 37.5, ContextInfo, '600000248')
```

### order_target_percent-指定目标比例交易

**用法：** order_target_percent(stockcode, tar_percent[, style, price], ContextInfo[, accId])

**释义：** 指定目标比例交易，买入 / 卖出证券以自动调整该证券的仓位到占有一个指定的投资组合的目标百分比。投资组合价值等于所有已有仓位的价值和剩余现金的总和。买 / 卖单会被下舍入一手股数（A 股是 100 的倍数）的倍数。目标百分比应该是一个小数，并且最大值应该小于等于1，比如 0.5 表示 50%，需要注意，如果资金不足，该API将不会创建发送订单。

**参数：**

- stockcode：代码，string，如 '000002.SZ'

- tar_percent：目标百分比 [0 ~ 1]，double

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE5', 'SALE4', 'SALE3', 'SALE2', 'SALE1'：卖5-1价
  > 
  > 'BUY1', 'BUY2', 'BUY3', 'BUY4', 'BUY5'：买1-5价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价下买入调仓到 5.1% 持仓
    order_target_percent('000002.SZ', 0.051, ContextInfo, '600000248')

    # 用对手价调仓到 5.1% 持仓   
    order_target_percent('000002.SZ', 0.051, 'COMPETE', ContextInfo, '600000248')

    # 用指定价 37.5 调仓到 10.2% 持仓
    order_target_percent('000002.SZ', 0.102, 'fix', 37.5, ContextInfo, '600000248')
```

### order_shares-指定股数交易

**用法：** order_shares(stockcode, shares[, style, price], ContextInfo[, accId])

**释义：** 指定股数交易，指定股数的买 / 卖单,最常见的落单方式之一。如有需要落单类型当做一个参量传入，如果忽略掉落单类型，那么默认以最新价下单。

**参数：**

- stockcode：代码，string，如 '000002.SZ'

- shares：股数，int

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE5', 'SALE4', 'SALE3', 'SALE2', 'SALE1'：卖5-1价
  > 
  > 'BUY1', 'BUY2', 'BUY3', 'BUY4', 'BUY5'：买1-5价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价下 100 股买入 
    order_shares('000002.SZ', 100, ContextInfo, '600000248')

    # 用对手价下 100 股卖出   
    order_shares('000002.SZ', -100, 'COMPETE', ContextInfo, '600000248')

    # 用指定价 37.5 下 200 股卖出
    order_shares('000002.SZ', -200, 'fix', 37.5, ContextInfo, '600000248')
```

### buy_open-期货买入开仓

**用法：** buy_open(stockcode, amount[, style, price], ContextInfo[, accId])

**释义：** 期货买入开仓

**参数：**

- stockcode：代码，string，如 'IF1805.IF'

- amount：手数，int

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE1'：卖一价
  > 
  > 'BUY1'：买一价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价 1 手买入开仓 
    buy_open('IF1805.IF', 1, ContextInfo, '110476')

    # 用对手价 1 手买入开仓   
    buy_open('IF1805.IF', 1, 'COMPETE', ContextInfo, '110476')

    # 用指定价 3750 元 2 手买入开仓
    buy_open('IF1805.IF', 2, 'fix', 3750, ContextInfo, '110476')
```

### buy_close_tdayfirst-期货买入平仓（平今优先）

**用法：** buy_close_tdayfirst(stockcode, amount[, style, price], ContextInfo[, accId])

**释义：** 期货买入平仓，平今优先

**参数：**

- stockcode：代码，string，如 'IF1805.IF'

- amount：手数，int

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE1'：卖一价
  > 
  > 'BUY1'：买一价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价 1 手买入平仓，平今优先  
    buy_close_tdayfirst('IF1805.IF', 1, ContextInfo, '110476')

    # 用对手价 1 手买入平仓，平今优先   
    buy_close_tdayfirst('IF1805.IF', 1, 'COMPETE', ContextInfo, '110476')

    # 用指定价 3750 元 2 手买入平仓，平今优先
    buy_close_tdayfirst('IF1805.IF', 2, 'fix', 3750, ContextInfo, '110476')
```

### buy_close_ydayfirst-期货买入平仓（平昨优先）

**用法：** buy_close_ydayfirst(stockcode, amount[, style, price], ContextInfo[, accId])

**释义：** 期货买入开仓，平昨优先

**参数：**

- stockcode：代码，string，如 'IF1805.IF'

- amount：手数，int

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE1'：卖一价
  > 
  > 'BUY1'：买一价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价 1 手买入平仓，平昨优先
    buy_close_ydayfirst('IF1805.IF', 1, ContextInfo, '110476')

    # 用对手价 1 手买入平仓，平昨优先   
    buy_close_ydayfirst('IF1805.IF', 1, 'COMPETE', ContextInfo, '110476')

    # 用指定价 3750 元 2 手买入平仓，平昨优先
    buy_close_ydayfirst('IF1805.IF', 2, 'fix', 3750, ContextInfo, '110476')
```

### sell_open-期货卖出开仓

**用法：** sell_open(stockcode, amount[, style, price], ContextInfo[, accId])

**释义：** 期货卖出开仓

**参数：**

- stockcode：代码，string，如 'IF1805.IF'

- amount：手数，int

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE1'：卖一价
  > 
  > 'BUY1'：买一价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价 1 手卖出开仓
    sell_open('IF1805.IF', 1, ContextInfo, '110476')

    # 用对手价 1 手卖出开仓   
    sell_open('IF1805.IF', 1, 'COMPETE', ContextInfo, '110476')

    # 用指定价 3750 元 2 手卖出开仓
    sell_open('IF1805.IF', 2, 'fix',3750, ContextInfo, '110476')
```

### sell_close_tdayfirst-期货卖出平仓（平今优先）

**用法：** sell_close_tdayfirst(stockcode, amount[, style, price], ContextInfo[, accId])

**释义：** 期货卖出平仓，平今优先

**参数：**

- stockcode：代码，string，如 'IF1805.IF'

- amount：手数，int

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE1'：卖一价
  > 
  > 'BUY1'：买一价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo):
    # 按最新价下 1 手卖出平仓，平今优先
    sell_close_tdayfirst('IF1805.IF', 1, ContextInfo, '110476')

    # 用对手价 1 手卖出平仓，平今优先
    sell_close_tdayfirst('IF1805.IF', 1, 'COMPETE', ContextInfo, '110476')

    # 用指定价 3750 元 2 手卖出平仓，平今优先
    sell_close_tdayfirst('IF1805.IF', 1, 'fix', 3750, ContextInfo, '110476')
```

### sell_close_ydayfirst-期货卖出平仓（平昨优先）

**用法：** sell_close_ydayfirst(stockcode, amount[, style, price], ContextInfo[, accId])

**释义：** 期货卖出平仓，平昨优先

**参数：**

- stockcode：代码，string，如 'IF1805.IF'

- amount：手数，int

- style：下单选价类型，string，默认为最新价 'LATEST'，可选值：
  
  > 'LATEST'：最新
  > 
  > 'FIX'：指定
  > 
  > 'HANG'：挂单
  > 
  > 'COMPETE'：对手
  > 
  > 'MARKET'：市价
  > 
  > 'SALE1'：卖一价
  > 
  > 'BUY1'：买一价

- price：价格，double

- ContextInfo：PythonObj，Python 对象，这里必须是 ContextInfo

- accId：账号，string

**返回：** 无

**示例：**

```
def handlebar(ContextInfo): 
    # 按最新价 1 手卖出平仓，平昨优先 
    sell_close_ydayfirst('IF1805.IF', 1, ContextInfo, '110476')

    # 用对手价 1 手卖出平仓，平昨优先   
    sell_close_ydayfirst('IF1805.IF', 1, 'COMPETE', ContextInfo, '110476')

    # 用指定价 3750 元 2 手卖出平仓，平昨优先
    sell_close_ydayfirst('IF1805.IF', 2, 'fix', 3750, ContextInfo, '110476')
```

### [已弃用] get_debt_contract-获取两融负债合约明细

**用法：** get_debt_contract(accId)

**释义：** 获取信用账户负债合约明细

此接口已弃用，替代接口为get_unclosed_compacts（获取未了结负债）和get_closed_compacts（获取已了结负债）

**参数：**

- accId：信用账户

**返回：** list，list 中放的是 [PythonObj](https://dict.thinktrader.net/pages/e148c4/#_5-3-7-stkcompacts%E8%B4%9F%E5%80%BA%E5%90%88%E7%BA%A6%E5%AF%B9%E8%B1%A1)，通过 dir(pythonobj) 可返回某个对象的属性列表。

**示例：**

```
def handlebar(ContextInfo):
    obj_list = get_debt_contract('6000000248')
    for obj in obj_list:
        # 输出负债合约名
        print(obj.m_strInstrumentName)
```

### get_hkt_exchange_rate-获取沪深港通汇率数据

**用法：** get_hkt_exchange_rate(accountID,accountType)

**释义：** 获取沪深港通汇率数据

**参数：**

- accountID：string,账号；
- accountType:string,账号类型,必须填HUGANGTONG或者SHENGANGTONG

**返回：**

> dict,字段释义：
> 
> bidReferenceRate:买入参考汇率
> 
> askReferenceRate:卖出参考汇率
> 
> dayBuyRiseRate:日间买入参考汇率浮动比例
> 
> daySaleRiseRate:日间卖出参考汇率浮动比例

**示例：**

```
def init(ContextInfo):
      data=get_hkt_exchange_rate('6000000248','HUGANGTONG')
      print(data)
```

上次更新: 2025/4/22 14:

## 实时主推函数

### account_callback - 资金账号状态变化主推

提示

1. 仅在实盘运行模式下生效。
2. 需要先在init里调用ContextInfo.set_account后生效。

**用法：** account_callback(ContextInfo, accountInfo)

**释义：** 当资金账号状态有变化时，这个函数被客户端调用

**参数：**

- ContextInfo：特定对象
- accountInfo：[账号对象](https://dict.thinktrader.net/innerApi/data_structure.html#account-%E8%B4%A6%E6%88%B7%E5%AF%B9%E8%B1%A1)或[信用账号对象](https://dict.thinktrader.net/innerApi/data_structure.html#ccreditaccountdetail-%E4%BF%A1%E7%94%A8%E8%B4%A6%E5%8F%B7%E5%AF%B9%E8%B1%A1-%E9%9D%9E%E6%9F%A5%E6%9F%9C%E5%8F%B0)

**返回：** 无

**示例：**

示例返回值

```
#coding:gbk
def show_data(data):
    tdata = {}
    for ar in dir(data):
        if ar[:2] != 'm_':continue
        try:
            tdata[ar] = data.__getattribute__(ar)
        except:
            tdata[ar] = '<CanNotConvert>'
    return tdata

def init(ContextInfo):
    # 设置对应的资金账号
    # 示例需要在策略交易界面运行
    ContextInfo.set_account(account)

def after_init(ContextInfo):
    # 在策略交易界面运行时，account的值会被赋值为策略配置中的账号，编辑器界面运行时，需要手动赋值
    # 编译器界面里执行的下单函数不会产生实际委托  
    passorder(23, 1101, account, "000001.SZ", 5, 0, 100, "示例", 2, "投资备注",ContextInfo)
    pass

def account_callback(ContextInfo, accountInfo):
    print(show_data(accountInfo)) 
```

### task_callback - 账号任务状态变化主推

提示

1. 仅在实盘运行模式下生效。
2. 需要先在init里调用ContextInfo.set_account后生效。

**用法：** task_callback(ContextInfo, taskInfo)

**释义：** 当账号任务状态有变化时，这个函数被客户端调用

**参数：**

- ContextInfo：特定对象
- taskInfo [任务对象](https://dict.thinktrader.net/innerApi/data_structure.html#ctaskdetail-%E4%BB%BB%E5%8A%A1%E5%AF%B9%E8%B1%A1)

**返回：** 无

**示例：**

示例返回值

```
#coding:gbk
def show_data(data):
    tdata = {}
    for ar in dir(data):
        if ar[:2] != 'm_':continue
        try:
            tdata[ar] = data.__getattribute__(ar)
        except:
            tdata[ar] = '<CanNotConvert>'
    return tdata

def init(ContextInfo):
    # 设置对应的资金账号
    # 示例需要在策略交易界面运行
    ContextInfo.set_account(account)

def after_init(ContextInfo):
    # 在策略交易界面运行时，account的值会被赋值为策略配置中的账号，编辑器界面运行时，需要手动赋值
    # 编译器界面里执行的下单函数不会产生实际委托  
    passorder(23, 1101, account, "000001.SZ", 5, 0, 100, "示例", 2, "投资备注",ContextInfo)
    pass

def task_callback(ContextInfo, taskInfo):
    print(show_data(taskInfo))
```

### order_callback - 账号委托状态变化主推

提示

1. 仅在实盘运行模式下生效。
2. 需要先在init里调用ContextInfo.set_account后生效。

**用法：** order_callback(ContextInfo, orderInfo)

**释义：** 当账号委托状态有变化时，这个函数被客户端调用

**参数：**

- ContextInfo：特定对象
- orderInfo：[委托](https://dict.thinktrader.net/innerApi/data_structure.html#order-%E5%A7%94%E6%89%98%E5%AF%B9%E8%B1%A1)

**返回：** 无

**示例：**

示例返回值

```
#coding:gbk
def show_data(data):
    tdata = {}
    for ar in dir(data):
        if ar[:2] != 'm_':continue
        try:
            tdata[ar] = data.__getattribute__(ar)
        except:
            tdata[ar] = '<CanNotConvert>'
    return tdata

def init(ContextInfo):
    # 设置对应的资金账号
    # 示例需要在策略交易界面运行
    ContextInfo.set_account(account)

def after_init(ContextInfo):
    # 在策略交易界面运行时，account的值会被赋值为策略配置中的账号，编辑器界面运行时，需要手动赋值
    # 编译器界面里执行的下单函数不会产生实际委托  
    passorder(23, 1101, account, "000001.SZ", 5, 0, 100, "示例", 2, "投资备注",ContextInfo)
    pass

def order_callback(ContextInfo, orderInfo):
    print(show_data(orderInfo))
```

### deal_callback - 账号成交状态变化主推

提示

1. 仅在实盘运行模式下生效。
2. 需要先在init里调用ContextInfo.set_account后生效。

**用法：** deal_callback(ContextInfo, dealInfo)

**释义：** 当账号成交状态有变化时，这个函数被客户端调用

**参数：**

- ContextInfo：特定对象
- dealInfo：[成交](https://dict.thinktrader.net/innerApi/data_structure.html#deal-%E6%88%90%E4%BA%A4%E5%AF%B9%E8%B1%A1)

**返回：** 无

**示例：**

示例返回值

```
#coding:gbk
def show_data(data):
    tdata = {}
    for ar in dir(data):
        if ar[:2] != 'm_':continue
        try:
            tdata[ar] = data.__getattribute__(ar)
        except:
            tdata[ar] = '<CanNotConvert>'
    return tdata

def init(ContextInfo):
    # 设置对应的资金账号
    # 示例需要在策略交易界面运行
    ContextInfo.set_account(account)

def after_init(ContextInfo):
    # 在策略交易界面运行时，account的值会被赋值为策略配置中的账号，编辑器界面运行时，需要手动赋值
    # 编译器界面里执行的下单函数不会产生实际委托  
    passorder(23, 1101, account, "000001.SZ", 5, 0, 100, "示例", 2, "投资备注",ContextInfo)
    pass

def deal_callback(ContextInfo, dealInfo):
    print(show_data(dealInfo))
```

### position_callback - 账号持仓状态变化主推

提示

1. 仅在实盘运行模式下生效。
2. 需要先在init里调用ContextInfo.set_account后生效。

**用法：** position_callback(ContextInfo, positonInfo)

**释义：** 当账号持仓状态有变化时，这个函数被客户端调用

**参数：**

- ContextInfo：特定对象
- positonInfo：[持仓](https://dict.thinktrader.net/innerApi/data_structure.html#position-%E6%8C%81%E4%BB%93%E5%AF%B9%E8%B1%A1)

**返回：** 无

**示例：**

示例返回值

```
#coding:gbk
def show_data(data):
    tdata = {}
    for ar in dir(data):
        if ar[:2] != 'm_':continue
        try:
            tdata[ar] = data.__getattribute__(ar)
        except:
            tdata[ar] = '<CanNotConvert>'
    return tdata

def init(ContextInfo):
    # 设置对应的资金账号
    # 示例需要在策略交易界面运行
    ContextInfo.set_account(account)

def after_init(ContextInfo):
    # 在策略交易界面运行时，account的值会被赋值为策略配置中的账号，编辑器界面运行时，需要手动赋值
    # 编译器界面里执行的下单函数不会产生实际委托  
    passorder(23, 1101, account, "000001.SZ", 5, 0, 100, "示例", 2, "投资备注",ContextInfo)
    pass

def position_callback(ContextInfo, positionInfo):
    print(show_data(positionInfo))
```

### orderError_callback - 账号异常下单主推

提示

1. 仅在实盘运行模式下生效。
2. 需要先在init里调用ContextInfo.set_account后生效。

**用法：** orderError_callback(ContextInfo,orderArgs,errMsg)

**释义：** 当账号下单异常时，这个函数被客户端调用

**参数：**

- ContextInfo：特定对象
- orderArgs：[下单参数](https://dict.thinktrader.net/innerApi/data_structure.html#passorderarguments-%E4%B8%8B%E5%8D%95%E5%87%BD%E6%95%B0%E5%8F%82%E6%95%B0%E5%AF%B9%E8%B1%A1)
- errMsg：错误信息

**返回：** 无

**示例：**

示例返回值

```
#coding:gbk
def show_data(data):
    tdata = {}
    for ar in dir(data):
        if ar[:2] != 'm_':continue
        try:
            tdata[ar] = data.__getattribute__(ar)
        except:
            tdata[ar] = '<CanNotConvert>'
    return tdata

def init(ContextInfo):
    # 设置对应的资金账号
    # 示例需要在策略交易界面运行
    ContextInfo.set_account(account)

def after_init(ContextInfo):
    # 在策略交易界面运行时，account的值会被赋值为策略配置中的账号，编辑器界面运行时，需要手动赋值
    # 编译器界面里执行的下单函数不会产生实际委托  
    passorder(23, 1101, account, "000001.SZ", 11, 0, 100, "示例", 2, "投资备注",ContextInfo)
    pass

def orderError_callback(ContextInfo,orderArgs,errMsg):
    print(show_data(orderArgs))
    print(errMsg)
```

## 其他主推函数

### credit_account_callback - 查询信用账户明细回调

**用法：** credit_account_callback(ContextInfo,seq,result)

**释义：** 查询信用账户明细回调

**参数：**

- ContextInfo：策略模型全局对象
- seq:query_credit_account时输入查询seq
- result: [信用账户明细](https://dict.thinktrader.net/innerApi/data_structure.html#ccreditdetail-%E4%B8%A4%E8%9E%8D%E8%B5%84%E9%87%91%E4%BF%A1%E6%81%AF-%E6%9F%A5%E6%9F%9C%E5%8F%B0)

### credit_opvolume_callback - 查询两融最大可下单量的回调

**用法：** credit_opvolume_callback(ContextInfo,accid,seq,ret,result)

**释义：** 查询两融最大可下单量的回调。

**参数：**

- `ContextInfo`：策略模型全局对象
- `accid`:查询的账号
- `seq`:`query_credit_opvolume`时输入查询`seq`
- `ret`:查询结果状态。正常返回:`1`,正在查询中`-1`,输入账号非法:`-2`,输入查询参数非法:`-3`,超时等服务器返回报错:`-4`
- `result`:查询到的结果

**示例** 见[query_credit_opvolume](https://dict.thinktrader.net/innerApi/callback_function.html?id=7zqjlm#query_credit_opvolume---%E6%9F%A5%E8%AF%A2%E4%B8%A4%E8%9E%8D%E6%9C%80%E5%A4%A7%E5%8F%AF%E4%B8%8B%E5%8D%95%E9%87%8F%60)

上次更新: 2024/11/4 19:27:12

邀请注册送VIP优惠券

分享下方的内容给好友、QQ群、微信群,好友注册您即可获得VIP优惠券

玩转qmt,上迅投qmt知识库

 [交易函数](https://dict.thinktrader.net/innerApi/trading_function.html)

## ext_data - 获取扩展数据

获取扩展数据

**调用方法：**`ext_data(extdataname, stockcode, deviation, ContextInfo)`

**参数：**

| 参数名           | 类型          | 说明        | 提示                         |
| ------------- | ----------- | --------- | -------------------------- |
| `extdataname` | `string`    | 扩展数据名     |                            |
| `stockcode`   | `string`    | 证券代码      | 形式如 '600000.SH'            |
| `deviation`   | `number`    | K 线偏移     | 0：不偏移，N：向右偏移N，-N：向左偏移N     |
| `ContextInfo` | `pythonObj` | Python 对象 | ython 对象，这里必须是 ContextInfo |

**返回：** number

** 示例：**

```
#coding:gbk
def init(ContextInfo):
    print(ext_data('CR', '600000.SH', 0, ContextInfo))
```

## ext_data_rank - 获取引用的扩展数据的数值在所有品种中的排名

获取引用的扩展数据的数值在所有品种中的排名

**调用方法：**`ext_data_rank(extdataname, stockcode, deviation, ContextInfo)`

**参数：**

| 参数名           | 类型          | 说明        | 提示                         |
| ------------- | ----------- | --------- | -------------------------- |
| `extdataname` | `string`    | 扩展数据名     |                            |
| `stockcode`   | `string`    | 证券代码      | 形式如 '600000.SH'            |
| `deviation`   | `number`    | K 线偏移     | 0：不偏移，N：向右偏移N，-N：向左偏移N     |
| `ContextInfo` | `pythonObj` | Python 对象 | ython 对象，这里必须是 ContextInfo |

**返回：** number

** 示例：**

```
#coding:gbk
def init(ContextInfo):
    print(ext_data_rank('mycci', '600000.SH', 0, ContextInfo))
```

## ext_data_rank_range - 获取引用的扩展数据的数值在指定时间区间内所有品种中的排名

获取引用的扩展数据的数值在指定时间区间内所有品种中的排名

** 调用方法： **`ext_data_rank_range(extdataname, stockcode, begintime, endtime, ContextInfo)`

**参数：**

| 参数名           | 类型          | 说明       | 提示                                   |
| ------------- | ----------- | -------- | ------------------------------------ |
| `extdataname` | `string`    | 扩展数据名    |                                      |
| `stockcode`   | `string`    | 证券代码     | 形式如 '600000.SH'                      |
| `begintime`   | `string`    | 区间的起始时间  | 格式为 '2016-08-02 12:12:30'（包括该时间点在内）  |
| `endtime`     | `string`    | 区间的结束时间  | 格式为 '2017-08-02 12:12:30' （包括该时间点在内） |
| `ContextInfo` | `pythonObj` | Python对象 | Python 对象，这里必须是 ContextInfo          |

**返回：** pythonDict

** 示例：**

```
#coding:gbk
def init(ContextInfo):
    print(ext_data_rank_range('mycci', '600000.SH','2022-08-02 12:12:30', '2023-08-02 12:12:30', ContextInfo))
```

## ext_data_range - 获取扩展数据在指定时间区间内的值

获取扩展数据在指定时间区间内的值

**调用方法：**`ext_data_range(extdataname, stockcode, begintime, endtime, ContextInfo)`

**参数：**

| 参数名           | 类型          | 说明       | 提示                                   |
| ------------- | ----------- | -------- | ------------------------------------ |
| `extdataname` | `string`    | 扩展数据名    |                                      |
| `stockcode`   | `string`    | 证券代码     | 形式如 '600000.SH'                      |
| `begintime`   | `string`    | 区间的起始时间  | 格式为 '2016-08-02 12:12:30'（包括该时间点在内）  |
| `endtime`     | `string`    | 区间的结束时间  | 格式为 '2017-08-02 12:12:30' （包括该时间点在内） |
| `ContextInfo` | `pythonObj` | Python对象 | Python 对象，这里必须是 ContextInfo          |

**返回：** pythonDict

**示例：**

```
#coding:gbk
def init(ContextInfo):
    print(ext_data_range('mycci', '600000.SH','2022-08-02 12:12:30', '2023-08-02 12:12:30', ContextInfo))
```

## get_factor_value - 获取因子数据

获取因子数据

**调用方法：**`get_factor_value(factorname, stockcode, deviation, ContextInfo)`

**参数：**

| 参数名           | 类型          | 说明       | 提示                          |
| ------------- | ----------- | -------- | --------------------------- |
| `factorname`  | `string`    | 因子名称     |                             |
| `stockcode`   | `string`    | 证券代码     | 形式如 '600000.SH'             |
| `deviation`   | `number`    | K 线偏移    | 0 不偏移，N 向右偏移 N，-N 向左偏移 N    |
| `ContextInfo` | `pythonObj` | Python对象 | Python 对象，这里必须是 ContextInfo |

**返回：** number

**示例：**

```
#coding:gbk
def init(ContextInfo):
    print(get_factor_value('zzz', '600000.SH', 0, ContextInfo))
```

## get_factor_rank - 获取引用的因子数据的数值在所有品种中排名

获取引用的因子数据的数值在所有品种中排名

**调用方法：**`get_factor_rank(factorname, stockcode, deviation, ContextInfo)`

**参数：**

| 参数名           | 类型          | 说明       | 提示                          |
| ------------- | ----------- | -------- | --------------------------- |
| `factorname`  | `string`    | 因子名称     |                             |
| `stockcode`   | `string`    | 证券代码     | 形式如 '600000.SH'             |
| `deviation`   | `number`    | K 线偏移    | 0 不偏移，N 向右偏移 N，-N 向左偏移 N    |
| `ContextInfo` | `pythonObj` | Python对象 | Python 对象，这里必须是 ContextInfo |

**示例：**

```
#coding:gbk
def init(ContextInfo):
    print(get_factor_rank('zzz', '600000.SH', 0, ContextInfo))
```

## call_vba - 获取引用的 VBA 模型运行的结果

获取引用的 VBA 模型运行的结果

提示

注意

1. 使用该函数时需补充好本地 K 线或分笔数据

**调用方法：** `call_vba(factorname, stockcode,[period, dividend_type, barpos],ContextInfo)`

**参数：**

| 参数名             | 类型          | 说明        | 提示                             |
| --------------- | ----------- | --------- | ------------------------------ |
| `factorname`    | `string`    | 因子名称      |                                |
| `stockcode`     | `string`    | 证券代码      | 形式如 '600000.SH'                |
| `period`        | `string`    | K 线偏移     | 可缺省，默认为当前主图周期线型                |
| `dividend_type` | `string`    | 复权方式      | 可缺省，默认当前图复权方式，具体可选值如下          |
| `barpos`        | `number`    | 对应 bar 下标 | 可缺省，默认当前主图调用到的 bar 的对应下标xtInfo |
| `ContextInfo`   | `pythonObj` | Python 对象 | Python 对象，这里必须是 ContextInfo    |

- period 可选值：
  
  > 'tick'：分笔线 '1d'：日线 '1m'：1分钟线 '3m'：3分钟线 '5m'：5分钟线 '15m'：15分钟线 '30m'：30分钟线 '1h'：小时线 '1w'：周线 '1mon'：月线 '1q'：季线 '1hy'：半年线 '1y'：年线

- dividend_type 可选值：
  
  > 'none'：不复权 'front'：向前复权 'back'：向后复权 'front_ratio'：等比向前复权 'back_ratio'：等比向后复权

**返回：** number

**示例：**

python

```
#coding:gbk
def init(ContextInfo):
    print(call_vba('MA.ma1', '600036.SH', ContextInfo))
```

@ tab 返回值

```
-1.0
```

上次更新:

## ContextInfo.paint - 在界面上画图

在界面上画图

**调用方法：** ContextInfo.paint(name, value, index, line_style, color = 'white', limit = '')

**参数：**

| 参数名          | 类型       | 说明          | 提示                                                                                     |
| ------------ | -------- | ----------- | -------------------------------------------------------------------------------------- |
| `name`       | `string` | 需显示的指标名     |                                                                                        |
| `value`      | `number` | 需显示的数值      |                                                                                        |
| `index`      | `number` | 显示索引位置      | 填 -1 表示按主图索引显示                                                                         |
| `line_style` | `number` | 线型          | 0：曲线<br>42：柱状线                                                                         |
| `color`      | `string` | 颜色（不填默认为白色） | blue：蓝色<br>brown：棕<br>cyan：蓝绿<br>green：绿<br>magenta：品红<br>red：红<br>white：白<br>yellow：黄 |
| `limit`      | `string` | 画线控制        | 'noaxis'：不影响坐标画线<br>'nodraw'：不画线                                                       |

**返回：**无

**示例：**

python

```
def init(ContextInfo):
    realtimetag = ContextInfo.get_bar_timetag(ContextInfo.barpos)
    value = ContextInfo.get_close_price('', '', realtimetag) 
    ContextInfo.paint('close', value, -1, 0, 'white','noaxis')
```

## ContextInfo.draw_text - 在图形上显示文字

在图形上显示数字   **调用方法：** `ContextInfo.draw_text(condition, position, text)`

**参数：**

| 参数名         | 类型       | 说明      | 提示  |
| ----------- | -------- | ------- | --- |
| `condition` | `bool`   | 条件      |     |
| `Position`  | `number` | 文字显示的位置 |     |
| `text`      | `string` | 文字      |     |

**返回值：**无

**示例：**

```
def init(ContextInfo):
    ContextInfo.draw_text(1, 10, '文字')
```

## ContextInfo.draw_number - 在图形上显示数字

在图形上显示数字

**调用方法：** `ContextInfo.draw_number(cond, height, number, precision)`

**参数：**

| 参数名         | 类型       | 说明        | 提示         |
| ----------- | -------- | --------- | ---------- |
| `cond`      | `bool`   | 条件        |            |
| `height`    | `number` | 显示文字的高度位置 |            |
| `text`      | `string` | 显示的数字     |            |
| `precision` | `number` | 为小数显示位数   | 取值范围 0 - 7 |

**返回值：**无

**示例：**

```
def init(ContextInfo):
    close = ContextInfo.get_market_data(['close'])   
    ContextInfo.draw_number(1 > 0, close, 66, 1)
```

## ContextInfo.draw_vertline - 在数字 1 和数字 2 之间绘垂直线

在数字1和数字2之间绘垂直线

**调用方法：** `ContextInfo.draw_vertline(cond, number1, number2, color = '', limit = '')`

**参数：**

| 参数名       | 类型       | 说明          | 提示                                                                                     |
| --------- | -------- | ----------- | -------------------------------------------------------------------------------------- |
| `cond`    | `bool`   | 条件          |                                                                                        |
| `number1` | `number` | 数字1         |                                                                                        |
| `number2` | `number` | 数字2         |                                                                                        |
| `color`   | `string` | 颜色（不填默认为白色） | blue：蓝色<br>brown：棕<br>cyan：蓝绿<br>green：绿<br>magenta：品红<br>red：红<br>white：白<br>yellow：黄 |
| `limit`   | `string` | 画线控制        | 'noaxis'：不影响坐标画线<br>'nodraw'：不画线                                                       |

**返回：** 无

**示例：**

```
def init(ContextInfo):
    close = ContextInfo.get_market_data(['close'])
    open = ContextInfo.get_market_data(['open'])
    ContextInfo.draw_vertline(1 > 0, close, open, 'cyan')
```

## ContextInfo.draw_icon - 在图形上绘制小图标

在图形上绘制小图标

**调用方法：** `ContextInfo.draw_icon(cond, height, type)`

**参数：**

| 参数名      | 类型       | 说明    | 提示           |
| -------- | -------- | ----- | ------------ |
| `cond`   | `bool`   | 条件    |              |
| `height` | `number` | 图标的位置 |              |
| `text`   | `number` | 图标的类型 | 1：椭圆<br>0：矩形 |

**返回值：**无

**示例：**

```
def init(ContextInfo):
    close = ContextInfo.get_market_data(['close'])
    ContextInfo.draw_icon(1 > 0, close, 0)
```

上次更新

## opType - 操作类型

### 期货/股指期权/商品期权 - 六键

| 数值  | 描述  |
| --- | --- |
| 0   | 开多  |
| 1   | 平昨多 |
| 2   | 平今多 |
| 3   | 开空  |
| 4   | 平昨空 |
| 5   | 平今空 |

### 期货/股指期权/商品期权 - 四键

| 数值  | 描述       |
| --- | -------- |
| 6   | 平多, 优先平今 |
| 7   | 平多, 优先平昨 |
| 8   | 平空, 优先平今 |
| 9   | 平空, 优先平昨 |

### 期货/股指期权/商品期权 - 两键

| 数值  | 描述                              |
| --- | ------------------------------- |
| 10  | 卖出, 如有多仓, 优先平仓, 优先平今, 如有余量, 再开空 |
| 11  | 卖出, 如有多仓, 优先平仓, 优先平昨, 如有余量, 再开空 |
| 12  | 买入, 如有空仓, 优先平仓, 优先平今, 如有余量, 再开多 |
| 13  | 买入, 如有空仓, 优先平仓, 优先平昨, 如有余量, 再开多 |
| 14  | 买入, 不优先平仓                       |
| 15  | 卖出, 不优先平仓                       |

### 股票/ETF/可转债买卖

| 数值  | 描述                        |
| --- | ------------------------- |
| 23  | 股票/ETF/可转债买入，或沪港通、深港通股票买入 |
| 24  | 股票/ETF/可转债卖出，或沪港通、深港通股票卖出 |

### 融资融券

| 数值  | 描述    |
| --- | ----- |
| 27  | 融资买入  |
| 28  | 融券卖出  |
| 29  | 买券还券  |
| 30  | 直接还券  |
| 31  | 卖券还款  |
| 32  | 直接还款  |
| 33  | 担保品买入 |
| 34  | 担保品卖出 |

### 组合交易

| 数值  | 描述                 |
| --- | ------------------ |
| 25  | 组合买入，或沪港通、深港通的组合买入 |
| 26  | 组合卖出，或沪港通、深港通的组合卖出 |
| 27  | 融资买入               |
| 28  | 融券卖出               |
| 29  | 买券还券               |
| 31  | 卖券还款               |
| 33  | 担保品买入              |
| 34  | 担保品卖出              |
| 35  | 普通账号一键买卖           |
| 36  | 信用账号一键买卖           |
| 40  | 期货组合开多             |
| 43  | 期货组合开空             |
| 46  | 期货组合平多, 优先平今       |
| 47  | 期货组合平多, 优先平昨       |
| 48  | 期货组合平空, 优先平今       |
| 49  | 期货组合平空, 优先平昨       |

### ETF期权交易

| 数值  | 描述   |
| --- | ---- |
| 50  | 买入开仓 |
| 51  | 卖出平仓 |
| 52  | 卖出开仓 |
| 53  | 买入平仓 |
| 54  | 备兑开仓 |
| 55  | 备兑平仓 |
| 56  | 认购行权 |
| 57  | 认沽行权 |
| 58  | 证券锁定 |
| 59  | 证券解锁 |

### ETF申赎交易

| 数值  | 描述  |
| --- | --- |
| 60  | 申购  |
| 61  | 赎回  |

### 专项两融

| 数值  | 描述     |
| --- | ------ |
| 70  | 专项融资买入 |
| 71  | 专项融券卖出 |
| 72  | 专项买券还券 |
| 73  | 专项直接还券 |
| 74  | 专项卖券还款 |
| 75  | 专项直接还款 |

### 可转债转股/回售

| 数值  | 描述     |
| --- | ------ |
| 80  | 普通账户转股 |
| 81  | 普通账户回售 |
| 82  | 信用账户转股 |
| 83  | 信用账户回售 |

## orderType - 下单方式

提示

注意

一、期货不支持 1102 和 1202

二、对所有账号组的操作相当于对账号组里的每个账号做一样的操作，如：

1. `passorder(23, 1202, 'testS', '000001.SZ', 5, -1, 50000, ContextInfo)`，意思就是对账号组`testS` 里的所有账号都以最新价开仓买入 `50000` 元市值的 `000001.SZ`平安银行；
2. `passorder (60,1101,"test",'510050. SH', 5,-1,1, ContextInfo)`意思就是账号`test`申购`1`个单位 (`900000股`)的`华夏上证50ETF` (只申购不买入成分股)。

### 单股交易

| 数值   | 描述                         |
| ---- | -------------------------- |
| 1101 | 单股、单账号、普通、股/手方式下单          |
| 1102 | 单股、单账号、普通、金额（元）方式下单（只支持股票） |
| 1113 | 单股、单账号、总资产、比例 [0 ~ 1] 方式下单 |
| 1123 | 单股、单账号、可用、比例[0 ~ 1]方式下单    |

### 单股交易（账号组）

| 数值   | 描述                              |
| ---- | ------------------------------- |
| 1201 | 单股、账号组（无权重）、普通、股/手方式下单          |
| 1202 | 单股、账号组（无权重）、普通、金额（元）方式下单（只支持股票） |
| 1213 | 单股、账号组（无权重）、总资产、比例 [0 ~ 1] 方式下单 |
| 1223 | 单股、账号组（无权重）、可用、比例 [0 ~ 1] 方式下单  |

### 组合交易（单账号）

| 数值   | 描述                                                                              |
| ---- | ------------------------------------------------------------------------------- |
| 2101 | 组合、单账号、普通、按组合股票数量（篮子中股票设定的数量）方式下单 > 对应 volume 的单位为篮子的份                          |
| 2102 | 组合、单账号、普通、按组合股票权重（篮子中股票设定的权重）方式下单 > 对应 volume 的单位为元                             |
| 2103 | 组合、单账号、普通、按账号可用方式下单 > （底层篮子股票怎么分配？答：按可用资金比例后按篮子中股票权重分配，如用户没填权重则按相等权重分配）只对股票篮子支持 |

### 组合交易（账号组）

| 数值   | 描述                               |
| ---- | -------------------------------- |
| 2201 | 组合、账号组（无权重）、普通、按组合股票数量方式下单       |
| 2202 | 组合、账号组（无权重）、普通、按组合股票权重方式下单       |
| 2203 | 组合、账号组（无权重）、普通、按账号可用方式下单只对股票篮子支持 |

## prType - 下单选价类型

关于使用市价指令的说明

1. 对于上交所（42,43,44,45）
   
   1. 当`prType`选择**市价类型**时时，`price`为保护限价，范围为（0 - 9999）表示投资者能够接受的最高买入价或最低卖出价，即买入申报的成交价格和转限价的价格不高于保护限价，卖出申报的成交价格和转限价的价格不低于保护限价，当`price`指定为 `0` 时，保护限价为对应的涨跌停价
   2. 融券卖出不允许使用市价指令
   3. 集合竞价阶段不允许使用市价指令

2. 对于深交所（44,45,46,47,48）
   
   1. 市价申报只适用于有价格涨跌幅限制证券。
   2. 集合竞价阶段不允许使用市价指令

3. 对于北交所(42,43,44,45)
   
   1. 当`prType`选择**市价类型**时时，`price`为保护限价，范围为（0 - 9999）表示投资者能够接受的最高买入价或最低卖出价，即买入申报的成交价格和转限价的价格不高于保护限价，卖出申报的成交价格和转限价的价格不低于保护限价，当`price`指定为 `0` 时，保护限价为对应的涨跌停价
   2. 融券卖出不允许使用市价指令
   3. 集合竞价阶段不允许使用市价指令

| 数值  | 描述                                                    |
| --- | ----------------------------------------------------- |
| -1  | 无效(只对于algo_passorder起作用)                              |
| 0   | 卖5价                                                   |
| 1   | 卖4价                                                   |
| 2   | 卖3价                                                   |
| 3   | 卖2价                                                   |
| 4   | 卖1价                                                   |
| 5   | 最新价                                                   |
| 6   | 买1价                                                   |
| 7   | 买2价(组合不支持)                                            |
| 8   | 买3价(组合不支持)                                            |
| 9   | 买4价(组合不支持)                                            |
| 10  | 买5价(组合不支持)                                            |
| 11  | 指定价（只对单股情况支持,对组合交易不支持）                                |
| 12  | 涨跌停价(对手方最远端价格)                                        |
| 13  | 挂单价(本方一档价格)                                           |
| 14  | 对手价(对方一档价格)                                           |
| 18  | 市价最优价[郑商所][期货] (不支持模拟交易中使用)                           |
| 19  | 市价即成剩撤[大商所][期货] (不支持模拟交易中使用)                          |
| 20  | 市价全额成交或撤[大商所][期货] (不支持模拟交易中使用)                        |
| 21  | 市价最优一档即成剩撤[中金所][期货] (不支持模拟交易中使用)                      |
| 22  | 市价最优五档即成剩撤[中金所][期货] (不支持模拟交易中使用)                      |
| 23  | 市价最优一档即成剩转[中金所][期货] (不支持模拟交易中使用)                      |
| 24  | 市价最优五档即成剩转[中金所][期货] (不支持模拟交易中使用)                      |
| 26  | 限价即时全部成交否则撤单[上交所[期权]] [深交所[期权]] (不支持模拟交易中使用)          |
| 27  | 市价即成剩撤[上交所][期权] (不支持模拟交易中使用)                          |
| 28  | 市价即全成否则撤[上交所][期权] (不支持模拟交易中使用)                        |
| 29  | 市价剩转限价[上交所][期权] (不支持模拟交易中使用)                          |
| 42  | 最优五档即时成交剩余撤销申报[上交所[股票]][北交所[股票]] (不支持模拟交易中使用)         |
| 43  | 最优五档即时成交剩转限价申报[上交所[股票]][北交所[股票]] (不支持模拟交易中使用)         |
| 44  | 对手方最优价格委托[上交所[股票]][深交所[股票][北交所[股票]][期权]] (不支持模拟交易中使用) |
| 45  | 本方最优价格委托[上交所[股票]][深交所[股票][北交所[股票]][期权]] (不支持模拟交易中使用)  |
| 46  | 即时成交剩余撤销委托[深交所][股票][期权] (不支持模拟交易中使用)                  |
| 47  | 最优五档即时成交剩余撤销委托[深交所][股票][期权] (不支持模拟交易中使用)              |
| 48  | 全额成交或撤销委托[深交所][股票][期权] (不支持模拟交易中使用)                   |
| 49  | 盘后定价                                                  |

## volume - 下单数量

提示

根据 orderType 值最后一位确定 volume 的单位

### 单股下单时

| 数值  | 描述                                      |
| --- | --------------------------------------- |
| 1   | 股 / 手 （股票: 股，股票期权: 张，期货: 手，可转债: 张，基金：份） |
| 2   | 金额（元）                                   |
| 3   | 比例（%）                                   |

### 组合下单时

| 数值  | 描述         |
| --- | ---------- |
| 1   | 按组合股票数量（份） |
| 2   | 按组合股票权重（元） |
| 3   | 按账号可用（%）   |

## quicktrade - 快速下单

| 数值  | 描述  |
| --- | --- |
| 0   | 否   |
| 1   | 是   |
| 2   | 是   |

提示

`passorder`是对最后一根K线完全走完后生成的模型信号在下一根K线的第一个 tick 数据来时触发下单交易；

采用`quickTrade`参数设置为`1`时，非历史 bar 上执行时（`ContextInfo.is_last_bar()`为`True`），只要策略模型中调用到就触发下单交易。

`quickTrade`参数设置为`2`时，不判断 bar 状态，只要策略模型中调用到就触发下单交易，历史 bar 上也能触发下单，请谨慎使用。

## enum_ - 对象属性状态字段释义

### enum_EEntrustBS - 买卖方向

| 变量                   | 数值  | 描述   |
| -------------------- | --- | ---- |
| `ENTRUST_BUY`        | 48  | 买入，多 |
| `ENTRUST_SELL`       | 49  | 卖出，空 |
| `ENTRUST_PLEDGE_IN`  | 81  | 质押入库 |
| `ENTRUST_PLEDGE_OUT` | 66  | 质押出库 |

### EEntrustSubmitStatus - 报单状态

| 数值  | 描述      |
| --- | ------- |
| 48  | 已经提交    |
| 49  | 撤单已经提交  |
| 50  | 修改已经提交  |
| 51  | 已经接受    |
| 52  | 报单已经被拒绝 |
| 53  | 撤单已经被拒绝 |
| 54  | 改单已经被拒绝 |

### enum_EEntrustTypes - 委托类型

| 变量名称                                      | 数值  | 描述        |
| ----------------------------------------- | --- | --------- |
| ENTRUST_BUY_SELL                          | 48  | 买卖        |
| ENTRUST_QUERY                             | 49  | 查询        |
| ENTRUST_CANCE                             | 50  | 撤单        |
| ENTRUST_APPEND                            | 51  | 补单        |
| ENTRUST_COMFIRM                           | 52  | 确认        |
| ENTRUST_BIG                               | 53  | 大宗        |
| ENTRUST_FIN                               | 54  | 融资委托      |
| ENTRUST_SLO                               | 55  | 融券委托      |
| ENTRUST_CLOSE                             | 56  | 信用平仓      |
| ENTRUST_CREDIT_NORMAL                     | 57  | 信用普通委托    |
| ENTRUST_CANCEL_OPEN                       | 58  | 撤单补单      |
| ENTRUST_TYPE_OPTION_EXERCISE              | 59  | 行权        |
| ENTRUST_TYPE_OPTION_SECU_LOCK             | 60  | 锁定        |
| ENTRUST_TYPE_OPTION_SECU_UNLOCK           | 61  | 解锁        |
| ENTRUST_QUOTATION_REPURCHASE              | 62  | 报价回购      |
| ENTRUST_TYPE_OPTION_ABANDON               | 63  | 放弃行权      |
| ENTRUST_AGREEMENT_REPURCHASE              | 64  | 协议回购      |
| ENTRUST_TYPE_OPTION_COMB_EXERCISE         | 65  | 组合行权      |
| ENTRUST_TYPE_OPTION_BUILD_COMB_STRATEGY   | 66  | 构建组合策略持仓  |
| ENTRUST_TYPE_OPTION_RELEASE_COMB_STRATEGY | 67  | 解除组合策略持仓  |
| ENTRUST_TYPE_LMT_LOAN                     | 68  | 转融通出借     |
| ENTRUST_TYPE_LMT_LOAN_DEFER               | 69  | 转融通出借展期   |
| ENTRUST_TYPE_LMT_LOAN_FINISH_AHEAD        | 70  | 转融通出借提前了结 |
| ENTRUST_CROSS_MARKET_IN                   | 71  | 跨市场场内     |
| ENTRUST_CROSS_MARKET_OUT                  | 72  | 跨市场场外     |

### enum_EEntrustStatus - 委托状态

| 变量名称                             | 数值  | 描述                                       |
| -------------------------------- | --- | ---------------------------------------- |
| `ENTRUST_STATUS_WAIT_REPORTING`  | 49  | 待报                                       |
| `ENTRUST_STATUS_REPORTED`        | 50  | 已报（已报出到柜台，待成交）                           |
| `ENTRUST_STATUS_REPORTED_CANCEL` | 51  | 已报待撤（对已报状态的委托撤单吗，等待柜台处理撤单请求）             |
| `ENTRUST_STATUS_PARTSUCC_CANCEL` | 52  | 部成待撤（已报到柜台，已有部分成交，已发出对剩余部分的撤单，待柜台处理撤单请求） |
| `ENTRUST_STATUS_PART_CANCEL`     | 53  | 部撤（已报到柜台，已有部分成交，剩余部分已撤）                  |
| `ENTRUST_STATUS_CANCELED`        | 54  | 已撤                                       |
| `ENTRUST_STATUS_PART_SUCC`       | 55  | 部成（已报到柜台，已有部分成交）                         |
| `ENTRUST_STATUS_SUCCEEDED`       | 56  | 已成                                       |
| `ENTRUST_STATUS_JUNK`            | 57  | 废单（不符合报单条件，委托被打回，相关信息再委托的废单原因字段查看）       |

![委托状态流程](https://dict.thinktrader.net/assets/%E5%A7%94%E6%89%98%E7%8A%B6%E6%80%81%E5%8F%98%E5%8A%A8-c15208aa.png)

### enum_EHedge_Flag_Type - 投保类型

| 变量名称                     | 数值  | 描述  |
| ------------------------ | --- | --- |
| `HEDGE_FLAG_SPECULATION` | 49  | 投机  |
| `HEDGE_FLAG_ARBITRAGE`   | 50  | 套利  |
| `HEDGE_FLAG_HEDGE`       | 51  | 套保  |

### enum_EFutureTradeType - 成交类型

| 变量名称                                    | 数值  | 描述      |
| --------------------------------------- | --- | ------- |
| `FUTRUE_TRADE_TYPE_COMMON`              | 48  | 普通成交    |
| `FUTURE_TRADE_TYPE_OPTIONSEXECUTION`    | 49  | 期权成交    |
| `FUTURE_TRADE_TYPE_OTC`                 | 50  | OTC 成交  |
| `FUTURE_TRADE_TYPE_EFPDIRVED`           | 51  | 期转现衍生成交 |
| `FUTURE_TRADE_TYPE_COMBINATION_DERIVED` | 52  | 组合衍生成交  |

### enum_EBrokerPriceType - 价格类型

| 变量名称                                          | 数值  | 描述                                      |
| --------------------------------------------- | --- | --------------------------------------- |
| `BROKER_PRICE_ANY`                            | 49  | 市价                                      |
| `BROKER_PRICE_LIMIT`                          | 50  | 限价                                      |
| `BROKER_PRICE_BEST`                           | 51  | 最优价                                     |
| `BROKER_PRICE_PROP_ALLOTMENT`                 | 52  | 配股                                      |
| `BROKER_PRICE_PROP_REFER`                     | 53  | 转托                                      |
| `BROKER_PRICE_PROP_SUBSCRIBE`                 | 54  | 申购                                      |
| `BROKER_PRICE_PROP_BUYBACK`                   | 55  | 回购                                      |
| `BROKER_PRICE_PROP_PLACING`                   | 56  | 配售                                      |
| `BROKER_PRICE_PROP_DECIDE`                    | 57  | 指定                                      |
| `BROKER_PRICE_PROP_EQUITY`                    | 58  | 转股                                      |
| `BROKER_PRICE_PROP_SELLBACK`                  | 59  | 回售                                      |
| `BROKER_PRICE_PROP_DIVIDEND`                  | 60  | 股息                                      |
| `BROKER_PRICE_PROP_SHENZHEN_PLACING`          | 68  | 深圳配售确认                                  |
| `BROKER_PRICE_PROP_CANCEL_PLACING`            | 69  | 配售放弃                                    |
| `BROKER_PRICE_PROP_WDZY`                      | 70  | 无冻质押                                    |
| `BROKER_PRICE_PROP_DJZY`                      | 71  | 冻结质押                                    |
| `BROKER_PRICE_PROP_WDJY`                      | 72  | 无冻解押                                    |
| `BROKER_PRICE_PROP_JDJY`                      | 73  | 解冻解押                                    |
| `BROKER_PRICE_PROP_ETF`                       | 81  | ETF申购                                   |
| `BROKER_PRICE_PROP_VOTE`                      | 75  | 投票                                      |
| `BROKER_PRICE_PROP_YYSGYS`                    | 92  | 要约收购预售                                  |
| `BROKER_PRICE_PROP_YSYYJC`                    | 77  | 预售要约解除                                  |
| `BROKER_PRICE_PROP_FUND_DEVIDEND`             | 78  | 基金设红                                    |
| `BROKER_PRICE_PROP_FUND_ENTRUST`              | 79  | 基金申赎                                    |
| `BROKER_PRICE_PROP_CROSS_MARKET`              | 80  | 跨市转托                                    |
| `BROKER_PRICE_PROP_EXERCIS`                   | 83  | 权证行权                                    |
| `BROKER_PRICE_PROP_PEER_PRICE_FIRST`          | 84  | 对手方最优价格                                 |
| `BROKER_PRICE_PROP_L5_FIRST_LIMITPX`          | 85  | 最优五档即时成交剩余转限价                           |
| `BROKER_PRICE_PROP_MIME_PRICE_FIRST`          | 86  | 本方最优价格                                  |
| `BROKER_PRICE_PROP_INSTBUSI_RESTCANCEL`       | 87  | 即时成交剩余撤销                                |
| `BROKER_PRICE_PROP_L5_FIRST_CANCEL`           | 88  | 最优五档即时成交剩余撤销                            |
| `BROKER_PRICE_PROP_FULL_REAL_CANCEL`          | 89  | 全额成交并撤单                                 |
| `BROKER_PRICE_PROP_DIRECT_SECU_REPAY`         | 101 | 直接还券                                    |
| `BROKER_PRICE_PROP_FUND_CHAIHE`               | 90  | 基金拆合                                    |
| `BROKER_PRICE_PROP_DEBT_CONVERSION`           | 91  | 债转股                                     |
| `BROKER_PRICE_BID_LIMIT`                      | 92  | 港股通竞价限价                                 |
| `BROKER_PRICE_ENHANCED_LIMIT`                 | 93  | 港股通增强限价                                 |
| `BROKER_PRICE_RETAIL_LIMIT`                   | 94  | 港股通零股限价                                 |
| `BROKER_PRICE_PROP_INCREASE_SHARE`            | 'j' | 增发                                      |
| `BROKER_PRICE_PROP_COLLATERAL_TRANSFER`       | 107 | 担保品划转                                   |
| `BROKER_PRICE_PROP_NEEQ_PRICING`              | 'w' | 定价（全国股转 - 挂牌公司交易 - 协议转让）                |
| `BROKER_PRICE_PROP_NEEQ_MATCH_CONFIRM`        | 'x' | 成交确认（全国股转 - 挂牌公司交易 - 协议转让）              |
| `BROKER_PRICE_PROP_NEEQ_MUTUAL_MATCH_CONFIRM` | 'y' | 互报成交确认（全国股转 - 挂牌公司交易 - 协议转让）            |
| `BROKER_PRICE_PROP_NEEQ_LIMIT`                | 'z' | 限价（用于挂牌公司交易 - 做市转让 - 限价买卖和两网及退市交易-限价买卖） |

### enum_EOffset_Flag_Type - 操作类型

| 变量名称                                 | 数值  | 描述    |
| ------------------------------------ | --- | ----- |
| `EOFF_THOST_FTDC_OF_INVALID`         | -1  | 无效操作  |
| `EOFF_THOST_FTDC_OF_Open`            | 48  | 买入，开仓 |
| `EOFF_THOST_FTDC_OF_Close`           | 49  | 卖出，平仓 |
| `EOFF_THOST_FTDC_OF_ForceClose`      | 50  | 强平    |
| `EOFF_THOST_FTDC_OF_CloseToday`      | 51  | 平今    |
| `EOFF_THOST_FTDC_OF_CloseYesterday`  | 52  | 平昨    |
| `EOFF_THOST_FTDC_OF_ForceOff`        | 53  | 强减    |
| `EOFF_THOST_FTDC_OF_LocalForceClose` | 54  | 本地强平  |
| `EOFF_THOST_FTDC_OF_PLEDGE_IN`       | 81  | 质押入库  |
| `EOFF_THOST_FTDC_OF_PLEDGE_OUT`      | 66  | 质押出库  |
| `EOFF_THOST_FTDC_OF_ALLOTMENT`       | 67  | 股票配股  |

### enum_EXTSubjectsStatus - 融资融券状态

| 变量名称                     | 数值  | 描述  |
| ------------------------ | --- | --- |
| `SUBJECTS_STATUS_NORMAL` | 48  | 正常  |
| `SUBJECTS_STATUS_PAUSE`  | 49  | 暂停  |
| `SUBJECTS_STATUS_NOT`    | 50  | 作废  |

### enum_EXTCreditFundCtl - 融资交易控制

| 变量名称                            | 数值  | 描述               |
| ------------------------------- | --- | ---------------- |
| `FUND_CTL_ONLY_FIN_BUY`         | 48  | 只允许融资买入          |
| `FUND_CTL_ONLY_SELL_CASH_REPAY` | 49  | 只允许卖券还款          |
| `FUND_CTL_ALL`                  | 50  | 既允许融资买入又允许卖券还款   |
| `FUND_CTL_NONE`                 | 51  | 既不允许融资买入又不允许卖券还款 |

### enum_EXTCreditStkCtl - 融券交易控制

| 变量名称                          | 数值  | 描述               |
| ----------------------------- | --- | ---------------- |
| `STK_CTL_ONLY_SLO_SELL`       | 48  | 只允许融券卖出          |
| `STK_CTL_ONLY_BUY_SECU_REPAY` | 49  | 只允许买券还券          |
| `STK_CTL_ALL`                 | 50  | 既允许融券卖出又允许买券还券   |
| `STK_CTL_NONE`                | 51  | 既不允许融券卖出又不允许买券还券 |

### enum_EXTSloTypeQueryMode - 查询类型

| 变量名称                           | 数值  | 描述  |
| ------------------------------ | --- | --- |
| `XT_SLOTYPE_QUERYMODE_NOMARL`  | 48  | 普通  |
| `XT_SLOTYPE_QUERYMODE_SPECIAL` | 49  | 专项  |

### enum_EXTCompactType - 合约类型

| 变量名称               | 数值  | 描述  |
| ------------------ | --- | --- |
| `COMPACT_TYPE_ALL` | 32  | 不限制 |
| `COMPACT_TYPE_FIN` | 48  | 融资  |
| `COMPACT_TYPE_SLO` | 49  | 融券  |

### enum_EXTCompactStatus - 合约状态

| 变量名称                          | 数值  | 描述    |
| ----------------------------- | --- | ----- |
| `COMPACT_STATUS_ALL`          | 32  | 不限制   |
| `COMPACT_STATUS_UNDONE`       | 48  | 未归还   |
| `COMPACT_STATUS_PART_DONE`    | 49  | 部分归还  |
| `COMPACT_STATUS_DONE`         | 50  | 已归还   |
| `COMPACT_STATUS_DONE_BY_SELF` | 51  | 自行了结  |
| `COMPACT_STATUS_DONE_BY_HAND` | 52  | 手工了结  |
| `COMPACT_STATUS_NOT_DEBT`     | 53  | 未形成负债 |
| `COMPACT_STATUS_EXPIRY`       | 54  | 合约已过期 |

### enum_EXTCompactBrushSource - 头寸来源

| 变量名称                              | 数值  | 描述   |
| --------------------------------- | --- | ---- |
| `XT_COMPACT_BRUSH_SOURCE_ALL`     | 32  | 不限制  |
| `XT_COMPACT_BRUSH_SOURCE_NORMAL`  | 48  | 普通头寸 |
| `XT_COMPACT_BRUSH_SOURCE_SPECIAL` | 49  | 专项头寸 |

### enum_EXTSpecialAssure - 是否可以用融券资金买入

| 变量名称                          | 数值  | 描述             |
| ----------------------------- | --- | -------------- |
| `ASSURE_USE_SLO_CASH_DISABLE` | 48  | 担保品买入不允许使用融券资金 |
| `ASSURE_USE_SLO_CASH_ENABLE`  | 49  | 担保品买入允许使用融券资金  |

### enum_EOperationType - 下单操作类型/主要交易类型

| 变量名称                                                      | 数值   | 描述                         |
| --------------------------------------------------------- | ---- | -------------------------- |
| `OPT_OPEN_LONG`                                           | 0    | 开多                         |
| `OPT_CLOSE_LONG_HISTORY`                                  | 1    | 平昨多                        |
| `OPT_CLOSE_LONG_TODAY`                                    | 2    | 平今多                        |
| `OPT_OPEN_SHORT`                                          | 3    | 开空                         |
| `OPT_CLOSE_SHORT_HISTORY`                                 | 4    | 平昨空                        |
| `OPT_CLOSE_SHORT_TODAY`                                   | 5    | 平今空                        |
| `OPT_CLOSE_LONG_TODAY_FIRST`                              | 6    | 优先平今多                      |
| `OPT_CLOSE_LONG_HISTORY_FIRST`                            | 7    | 优先平昨多                      |
| `OPT_CLOSE_SHORT_TODAY_FIRST`                             | 8    | 平空优先平今                     |
| `OPT_CLOSE_SHORT_HISTORY_FIRST`                           | 9    | 平空优先平昨                     |
| `OPT_CLOSE_LONG_TODAY_HISTORY_THEN_OPEN_SHORT`            | 10   | 卖出优先平今                     |
| `OPT_CLOSE_LONG_HISTORY_TODAY_THEN_OPEN_SHORT`            | 11   | 卖出优先平昨                     |
| `OPT_CLOSE_SHORT_TODAY_HISTORY_THEN_OPEN_LONG`            | 12   | 买入优先平今                     |
| `OPT_CLOSE_SHORT_HISTORY_TODAY_THEN_OPEN_LONG`            | 13   | 买入优先平昨                     |
| `OPT_CLOSE_LONG`                                          | 14   | 平多                         |
| `OPT_CLOSE_SHORT`                                         | 15   | 平空                         |
| `OPT_OPEN`                                                | 16   | 开仓                         |
| `OPT_CLOSE`                                               | 17   | 平仓                         |
| `OPT_BUY`                                                 | 18   | 买入                         |
| `OPT_SELL`                                                | 19   | 卖出                         |
| `OPT_FIN_BUY`                                             | 20   | 融资买入                       |
| `OPT_SLO_SELL`                                            | 21   | 融券卖出                       |
| `OPT_BUY_SECU_REPAY`                                      | 22   | 买券还券                       |
| `OPT_DIRECT_SECU_REPAY`                                   | 23   | 直接还券                       |
| `OPT_SELL_CASH_REPAY`                                     | 24   | 卖券还款                       |
| `OPT_DIRECT_CASH_REPAY`                                   | 25   | 直接还款                       |
| `OPT_FUND_SUBSCRIBE`                                      | 26   | 基金申购                       |
| `OPT_FUND_REDEMPTION`                                     | 27   | 基金赎回                       |
| `OPT_FUND_MERGE`                                          | 28   | 基金合并                       |
| `OPT_FUND_SPLIT`                                          | 29   | 基金分拆                       |
| `OPT_PLEDGE_IN`                                           | 30   | 质押入库                       |
| `OPT_PLEDGE_OUT`                                          | 31   | 质押出库                       |
| `OPT_OPTION_BUY_OPEN`                                     | 32   | 买入开仓（个股期权交易）               |
| `OPT_OPTION_SELL_CLOSE`                                   | 33   | 卖出平仓（个股期权交易）               |
| `OPT_OPTION_SELL_OPEN`                                    | 34   | 卖出开仓（个股期权交易）               |
| `OPT_OPTION_BUY_CLOSE`                                    | 35   | 买入平仓（个股期权交易）               |
| `OPT_OPTION_COVERED_OPEN`                                 | 36   | 备兑开仓（个股期权交易）               |
| `OPT_OPTION_COVERED_CLOSE`                                | 37   | 备兑平仓（个股期权交易）               |
| `OPT_OPTION_CALL_EXERCISE`                                | 38   | 认购行权（个股期权交易）               |
| `OPT_OPTION_PUT_EXERCISE`                                 | 39   | 认沽行权（个股期权交易）               |
| `OPT_OPTION_SECU_LOCK`                                    | 40   | 证券锁定（个股期权交易）               |
| `OPT_OPTION_SECU_UNLOCK`                                  | 41   | 证券解锁（个股期权交易）               |
| `OPT_N3B_PRICE_BUY`                                       | 42   | 协议转让-定价买入                  |
| `OPT_N3B_PRICE_SELL`                                      | 43   | 协议转让-定价卖出                  |
| `OPT_N3B_CONFIRM_BUY`                                     | 44   | 协议转让-成交确认买入                |
| `OPT_N3B_CONFIRM_SELL`                                    | 45   | 协议转让-成交确认卖出                |
| `OPT_N3B_REPORT_CONFIRM_BUY`                              | 46   | 协议转让-互报成交确认买入              |
| `OPT_N3B_REPORT_CONFIRM_SELL`                             | 47   | 协议转让-互报成交确认卖出              |
| `OPT_N3B_LIMIT_PRICE_BUY`                                 | 48   | 全国股转-限价买入                  |
| `OPT_N3B_LIMIT_PRICE_SELL`                                | 49   | 全国股转-限价卖出                  |
| `OPT_FUTURE_OPTION_EXERCISE`                              | 50   | 期货期权行权                     |
| `OPT_CONVERT_BONDS`                                       | 51   | 可转债转股                      |
| `OPT_SELL_BACK_BONDS`                                     | 52   | 可转债回售                      |
| `OPT_STK_ALLOTMENT`                                       | 53   | 股票配股                       |
| `OPT_STK_INCREASE_SHARE`                                  | 54   | 股票增发                       |
| `OPT_COLLATERAL_TRANSFER_IN`                              | 55   | 担保品划入                      |
| `OPT_COLLATERAL_TRANSFER_OUT`                             | 56   | 担保品划出                      |
| `OPT_BLOCK_INTENTION_BUY`                                 | 57   | 意向申报买入                     |
| `OPT_BLOCK_INTENTION_SELL`                                | 58   | 意向申报卖出                     |
| `OPT_BLOCK_PRICE_BUY`                                     | 59   | 定价申报买入                     |
| `OPT_BLOCK_PRICE_SELL`                                    | 60   | 定价申报卖出                     |
| `OPT_BLOCK_CONFIRM_BUY`                                   | 61   | 成交申报买入                     |
| `OPT_BLOCK_CONFIRM_SELL`                                  | 62   | 成交申报卖出                     |
| `OPT_BLOCK_CLOSE_PRICE_BUY`                               | 63   | 盘后定价买入                     |
| `OPT_BLOCK_CLOSE_PRICE_SELL`                              | 64   | 盘后定价卖出                     |
| `OPT_GOLD_PRICE_DELIVERY_BUY`                             | 65   | 黄金交割买                      |
| `OPT_GOLD_PRICE_DELIVERY_SELL`                            | 66   | 黄金交割卖                      |
| `OPT_GOLD_PRICE_MIDDLE_BUY`                               | 67   | 黄金中立仓买                     |
| `OPT_GOLD_PRICE_MIDDLE_SELL`                              | 68   | 黄金中立仓卖                     |
| `OPT_COMPOSE_ONEKEY_BUYSELL`                              | 69   | 组合交易一键买卖                   |
| `OPT_COMPOSE_GGT_BUY`                                     | 70   | 组合交易港股通买入                  |
| `OPT_COMPOSE_GGT_SELL`                                    | 71   | 组合交易港股通卖出                  |
| `OPT_ODD_SELL`                                            | 72   | 零股卖出                       |
| `OPT_ETF_STOCK_BUY`                                       | 73   | 成份股买入                      |
| `OPT_ETF_STOCK_SELL`                                      | 74   | 成份股卖出                      |
| `OPT_OTC_FUND_SUBSCRIBE`                                  | 200  | 场外基金认购                     |
| `OPT_OTC_FUND_PURCHASE`                                   | 201  | 场外基金申购                     |
| `OPT_OTC_FUND_REDEMPTION`                                 | 202  | 场外基金赎回                     |
| `OPT_OTC_FUND_CONVERT`                                    | 203  | 场外基金转换                     |
| `OPT_OTC_FUND_BONUS_TYPE_UPDATE`                          | 204  | 场外基金分红方式变更                 |
| `OPT_OTC_CONTRACTUAL_DEPOSIT`                             | 205  | 场外协议存款                     |
| `OPT_OTC_NON_CONTRACTUAL_DEPOSIT`                         | 206  | 场外非协议存款                    |
| `OPT_OTC_CONTRACTUAL_DEPOSIT_ASK`                         | 207  | 场外协议存款询价                   |
| `OPT_OTC_NON_CONTRACTUAL_DEPOSIT_ASK`                     | 208  | 场外非协议存款询价                  |
| `OPT_OTC_NON_CONTRACTUAL_DEPOSIT_CUR`                     | 209  | 场外非协议活期存款                  |
| `OPT_OTC_DRAW_DEPOSIT`                                    | 210  | 场外存单支取                     |
| `OPT_OTC_STOCK_INQUIRY`                                   | 230  | 网下询价                       |
| `OPT_OTC_STOCK_PURCHASE`                                  | 231  | 网下申购                       |
| `OPT_OPTION_NS_DEPOSIT`                                   | 1001 | 场外转账入金                     |
| `OPT_OPTION_NS_WITHDRAW`                                  | 1002 | 场外转账出金                     |
| `OPT_OPTION_NS_INOUT`                                     | 1003 | 场外互转                       |
| `OPT_ETF_PURCHASE`                                        | 1004 | ETF申购                      |
| `OPT_ETF_REDEMPTION`                                      | 1005 | ETF赎回                      |
| `OPT_OUTER_BUY`                                           | 1006 | 外盘买入                       |
| `OPT_OUTER_SELL`                                          | 1007 | 外盘卖出                       |
| `OPT_OUTER_CAN_CLOSE_BUY`                                 | 1008 | 外盘可平买仓                     |
| `OPT_OUTER_CAN_CLOSE_SELL`                                | 1009 | 外盘可平卖仓                     |
| `OPT_SLO_SELL_SPECIAL`                                    | 1010 | 专项融券卖出                     |
| `OPT_BUY_SECU_REPAY_SPECIAL`                              | 1011 | 专项买券还券                     |
| `OPT_DIRECT_SECU_REPAY_SPECIAL`                           | 1012 | 专项直接还券                     |
| `OPT_NEEQ_O3B_LIMIT_PRICE_BUY`                            | 1013 | 全国股转-两网及退市交易-限价买入          |
| `OPT_NEEQ_O3B_LIMIT_PRICE_SELL`                           | 1014 | 全国股转-两网及退市交易-限价卖出          |
| `OPT_IBANK_BOND_BUY`                                      | 1015 | 投行债券买入                     |
| `OPT_IBANK_BOND_SELL`                                     | 1016 | 投行债券卖出                     |
| `OPT_IBANK_FUND_REPURCHASE`                               | 1017 | 质押式融资回购                    |
| `OPT_IBANK_BOND_REPURCHASE`                               | 1018 | 质押式融券回购                    |
| `OPT_IBANK_BOND_REPAY`                                    | 1019 | 质押式融资购回                    |
| `OPT_IBANK_FUND_RETRIEVE`                                 | 1020 | 质押式融券购回                    |
| `OPT_INTEREST_FEE`                                        | 1021 | 融券息费                       |
| `OPT_FIN_BUY_SPECIAL`                                     | 1022 | 专项融资买入                     |
| `OPT_SELL_CASH_REPAY_SPECIAL`                             | 1023 | 专项卖券还款                     |
| `OPT_DIRECT_CASH_REPAY_SPECIAL`                           | 1024 | 专项直接还款                     |
| `OPT_FUND_PRICE_BUY`                                      | 1025 | 货币基金申购                     |
| `OPT_FUND_PRICE_SELL`                                     | 1026 | 货币基金赎回                     |
| `OPT_N3B_CALL_AUCTION_BUY`                                | 1027 | 协议转让-集合竞价买入                |
| `OPT_N3B_CALL_AUCTION_SELL`                               | 1028 | 协议转让-集合竞价卖出                |
| `OPT_N3B_AFTER_HOURS_BUY`                                 | 1029 | 全国股转-盘后协议买入                |
| `OPT_N3B_AFTER_HOURS_SELL`                                | 1030 | 全国股转-盘后协议卖出                |
| `OPT_ETF_HEDGE`                                           | 1031 | ETF套利                      |
| `OPT_QUOTATION_REPURCHASE_BUY`                            | 1032 | 报价回购买入                     |
| `OPT_QUOTATION_REPURCHASE_STOP`                           | 1033 | 报价回购终止续做                   |
| `OPT_QUOTATION_REPURCHASE_BEFORE`                         | 1034 | 报价回购提前购回                   |
| `OPT_QUOTATION_REPURCHASE_RESERVATION`                    | 1035 | 报价回购购回预约                   |
| `OPT_QUOTATION_REPURCHASE_CANCEL`                         | 1036 | 报价回购取消预约                   |
| `OPT_BLOCK_CONFIRM_MATCH_BUY`                             | 1037 | 成交申报配对买入                   |
| `OPT_BLOCK_CONFIRM_MATCH_SELL`                            | 1038 | 成交申报配对卖出                   |
| `OPT_FUTURE_OPTION_ABANDON`                               | 1039 | 期货期权放弃行权                   |
| `OPT_ONEKEY_TRANSFER`                                     | 1040 | 一键划转                       |
| `OPT_ONEKEY_TRANSFER_IN`                                  | 1041 | 一键划入                       |
| `OPT_ONEKEY_TRANSFER_OUT`                                 | 1042 | 一键划出                       |
| `OPT_AFTER_FIX_BUY`                                       | 1043 | 盘后定价买入                     |
| `OPT_AFTER_FIX_SELL`                                      | 1044 | 盘后定价卖                      |
| `OPT_AGREEMENT_REPURCHASE_TRANSACTION_DEC_FORWARD`        | 1045 | 成交申报正回购                    |
| `OPT_AGREEMENT_REPURCHASE_TRANSACTION_DEC_REVERSE`        | 1046 | 成交申报逆回购                    |
| `OPT_AGREEMENT_REPURCHASE_EXPIRE_CONFIRM`                 | 1047 | 到期确认                       |
| `OPT_AGREEMENT_REPURCHASE_ADVANCE_REPURCHASE`             | 1048 | 提前购回正回购                    |
| `OPT_AGREEMENT_REPURCHASE_ADVANCE_REVERSE`                | 1049 | 提前购回逆回购                    |
| `OPT_AGREEMENT_REPURCHASE_EXPIRE_RENEW`                   | 1050 | 到期续做正回购                    |
| `OPT_AGREEMENT_REPURCHASE_EXPIRE_REVERSE`                 | 1051 | 到期续做逆回购                    |
| `OPT_TRANSACTION_IN_CASH_BUY`                             | 1052 | 现券买入                       |
| `OPT_TRANSACTION_IN_CASH_SELL`                            | 1053 | 现券卖出                       |
| `OPT_OUTRIGHT_REPO_FUND_REPURCHASE`                       | 1054 | 买断式融资回购                    |
| `OPT_OUTRIGHT_REPO_BOND_REPURCHASE`                       | 1055 | 买断式融券回购                    |
| `OPT_OUTRIGHT_REPO_BOND_REPAY`                            | 1056 | 买断式融资购回                    |
| `OPT_OUTRIGHT_REPO_FUND_RETRIEVE`                         | 1057 | 买断式融券购回                    |
| `OPT_DISTRIBUTION_BUYING`                                 | 1058 | 分销买入                       |
| `OPT_FIXRATE_TO_FLOATINGRATE`                             | 1059 | 固定利率换浮动利率                  |
| `OPT_FLOATINGRATE_TO_FIXRATE`                             | 1060 | 浮动利率换固定利率                  |
| `OPT_IBANK_TRANSFER_OUT`                                  | 1061 | 银行间转出托管                    |
| `OPT_IBANK_TRANSFER_IN`                                   | 1062 | 银行间转入托管                    |
| `OPT_AGREEMENT_REPURCHASE_INTENTION_BUY`                  | 1063 | 意向申报正回购买入                  |
| `OPT_AGREEMENT_REPURCHASE_INTENTION_SELL`                 | 1064 | 意向申报正回购卖出                  |
| `OPT_AGREEMENT_REPURCHASE_BIZ_APPLY_CONFIRM`              | 1065 | 协议回购成交申报确认                 |
| `OPT_AGREEMENT_REPURCHASE_BIZ_APPLY_REJECT`               | 1066 | 协议回购成交申报拒绝                 |
| `OPT_AGREEMENT_REPURCHASE_CONTINUE_CONFIRM`               | 1067 | 协议回购到期续做申报确认               |
| `OPT_AGREEMENT_REPURCHASE_CONTINUE_REJECT`                | 1068 | 协议回购到期续做申报拒绝               |
| `OPT_AGREEMENT_REPURCHASE_INTENTION_CHANGE_BONDS`         | 1069 | 协议回购换券申报                   |
| `OPT_AGREEMENT_REPURCHASE_INTENTION_CHANGE_BONDS_CONFIRM` | 1070 | 协议回购换券申报确认                 |
| `OPT_AGREEMENT_REPURCHASE_INTENTION_CHANGE_BONDS_REJECT`  | 1071 | 协议回购换券申报拒绝                 |
| `OPT_AGREEMENT_REPURCHASE_STOP_AHEAD_CONFIRM`             | 1072 | 协议回购正回购提前终止申报确认            |
| `OPT_AGREEMENT_REPURCHASE_STOP_AHEAD_REJECT`              | 1073 | 协议回购正回购提前终止申报拒绝            |
| `OPT_AGREEMENT_REPURCHASE_RELEASE_PLEDGE`                 | 1074 | 协议回购正回购方解除质押申报             |
| `OPT_AGREEMENT_REPURCHASE_RELEASE_PLEDGE_CONFIRM`         | 1075 | 协议回购正回购解除质押申报确认            |
| `OPT_AGREEMENT_REPURCHASE_RELEASE_PLEDGE_REJECT`          | 1076 | 协议回购正回购解除质押申报拒绝            |
| `OPT_AGREEMENT_REPURCHASE_EXPIRE_CONFIRM_SELL`            | 1077 | 深圳到期确认卖出                   |
| `OPT_LOAN_DISTRIBUTION_BUY`                               | 1078 | 债券分销                       |
| `OPT_PREFERENCE_SHARES_BIDDING_BUY`                       | 1079 | 优先股竞价买入                    |
| `OPT_PREFERENCE_SHARES_BIDDING_SELL`                      | 1080 | 优先股竞价卖出                    |
| `OPT_TOC_BOND`                                            | 1081 | 债券转托管                      |
| `OPT_TOC_FUND`                                            | 1082 | 基金转托管                      |
| `OPT_IBANK_BORROW`                                        | 1083 | 同业拆入                       |
| `OPT_IBANK_LOAN`                                          | 1084 | 同业拆出                       |
| `OPT_IBANK_BORROW_REPAY`                                  | 1085 | 拆入还款                       |
| `OPT_IBANK_LOAN_REPAY`                                    | 1086 | 拆出还款                       |
| `OPT_FINANCIAL_PRODUCT_BUY`                               | 1087 | 理财产品申购                     |
| `OPT_FINANCIAL_PRODUCT_SELL`                              | 1088 | 理财产品赎回                     |
| `OPT_OPTION_COMB_EXERCISE`                                | 1089 | 组合行权                       |
| `OPT_OPTION_BUILD_COMB_STRATEGY`                          | 1090 | 构建组合策略                     |
| `OPT_OPTION_RELEASE_COMB_STRATEGY`                        | 1091 | 解除组合策略                     |
| `OPT_AGREEMENT_REPURCHASE_REVERSE_STOP_AHEAD_CONFIRM`     | 1092 | 协议回购逆回购提前终止申报确认            |
| `OPT_AGREEMENT_REPURCHASE_REVERSE_STOP_AHEAD_REJECT`      | 1093 | 协议回购逆回购提前终止申报拒绝            |
| `OPT_AGREEMENT_REPURCHASE_REVERSE_RELEASE_PLEDGE`         | 1094 | 协议回购逆回购方解除质押申报             |
| `OPT_AGREEMENT_REPURCHASE_REVERSE_RELEASE_PLEDGE_CONFIRM` | 1095 | 协议回购逆回购解除质押申报确认            |
| `OPT_AGREEMENT_REPURCHASE_REVERSE_RELEASE_PLEDGE_REJECT`  | 1096 | 协议回购逆回购解除质押申报拒绝            |
| `OPT_BOND_TENDER`                                         | 1097 | 债券投标                       |
| `OPT_FINANCIAL_PRODUCT_CALL`                              | 1098 | 理财产品认购                     |
| `OPT_NEEQ_O3B_CONTINUOUS_AUCTION_BUY`                     | 1099 | 全国股转-北交所买入                 |
| `OPT_NEEQ_O3B_CONTINUOUS_AUCTION_SELL`                    | 1100 | 全国股转-北交所卖出                 |
| `OPT_NEEQ_O3B_ASK_PRICE`                                  | 1101 | 全国股转-申购-询价申报               |
| `OPT_NEEQ_O3B_PRICE_CONFIRM`                              | 1102 | 全国股转-申购-申购申报               |
| `OPT_NEEQ_O3B_BLOCKTRADING_BUY`                           | 1103 | 全国股转-大宗交易买入                |
| `OPT_NEEQ_O3B_BLOCKTRADING_SELL`                          | 1104 | 全国股转-大宗交易卖出                |
| `OPT_LMT_LOAN_SET`                                        | 1105 | 转融通非约定出借申报                 |
| `OPT_LMT_LOAN_CONVENTION`                                 | 1106 | 转融通约定出借申报                  |
| `OPT_LMT_LOAN_RENEWAL`                                    | 1107 | 转融通出借展期                    |
| `OPT_LMT_LOAN_SETTLE_EARLY`                               | 1108 | 转融通出借提前了结                  |
| `OPT_CROSS_MARKET_IN_ETF_PURCHASE`                        | 1109 | 跨市场ETF场内申购                 |
| `OPT_CROSS_MARKET_IN_ETF_REDEMPTION`                      | 1110 | 跨市场ETF场内赎回                 |
| `OPT_CROSS_MARKET_OUT_ETF_PURCHASE`                       | 1111 | 跨市场ETF场外申购                 |
| `OPT_CROSS_MARKET_OUT_ETF_REDEMPTION`                     | 1112 | 跨市场ETF场外赎回                 |
| `OPT_CREDIT_APPOINTMENT`                                  | 1113 | 券源预约                       |
| `OPT_OFF_IPO_PUB_PRICE`                                   | 1114 | 网下申购-公开发行询价                |
| `OPT_OFF_IPO_PUB_PURCHASE`                                | 1115 | 网下申购-公开发行申购                |
| `OPT_OFF_IPO_NON_PUB_PRICE`                               | 1116 | 网下申购-非公开发行询价               |
| `OPT_OFF_IPO_NON_PUB_PURCHASE`                            | 1117 | 网下申购-非公开发行申购               |
| `OPT_IBANK_PUT`                                           | 1118 | 债券回售                       |
| `OPT_IBANK_BOND_BORROW`                                   | 1119 | 债券借贷融入                     |
| `OPT_IBANK_BOND_LEND`                                     | 1120 | 债券借贷融出                     |
| `OPT_IBANK_BOND_BORROW_REPAY`                             | 1121 | 债券借贷融入购回                   |
| `OPT_IBANK_BOND_LEND_RETRIEVE`                            | 1122 | 债券借贷融出购回                   |
| `OPT_IBANK_BOND_DISPLACE`                                 | 1123 | 债券借贷-质押券置换                 |
| `OPT_LENDING_INTEGRATE_INTO`                              | 1124 | 融券通-预约融券融入                 |
| `OPT_LENDING_MELT_OUT`                                    | 1125 | 融券通-预约融券融出                 |
| `OPT_FICC_MANUAL_DECLARE_BUY`                             | 1126 | 固收业务-点击成交-报价申报买入           |
| `OPT_FICC_MANUAL_DECLARE_SELL`                            | 1127 | 固收业务-点击成交-报价申报卖出           |
| `OPT_FICC_MANUAL_CONFIRM_BUY_CONFIRM`                     | 1128 | 固收业务-点击成交-报价确认-买入-确认       |
| `OPT_FICC_MANUAL_CONFIRM_BUY_REJECT`                      | 1129 | 固收业务-点击成交-报价确认-买入-拒绝       |
| `OPT_FICC_MANUAL_CONFIRM_SELL_CONFIRM`                    | 1130 | 固收业务-点击成交-报价确认-卖出-确认       |
| `OPT_FICC_MANUAL_CONFIRM_SELL_REJECT`                     | 1131 | 固收业务-点击成交-报价确认-卖出-拒绝       |
| `OPT_FICC_CONSULT_DECLARE_BUY`                            | 1132 | 固收业务-协商成交-协商申报买入           |
| `OPT_FICC_CONSULT_DECLARE_SELL`                           | 1133 | 固收业务-协商成交-协商申报卖出           |
| `OPT_FICC_CONSULT_CONFIRM_BUY_CONFIRM`                    | 1134 | 固收业务-协商成交-协商确认-买入-确认       |
| `OPT_FICC_CONSULT_CONFIRM_BUY_REJECT`                     | 1135 | 固收业务-协商成交-协商确认-买入-拒绝       |
| `OPT_FICC_CONSULT_CONFIRM_SELL_CONFIRM`                   | 1136 | 固收业务-协商成交-协商确认-卖出-确认       |
| `OPT_FICC_CONSULT_CONFIRM_SELL_REJECT`                    | 1137 | 固收业务-协商成交-协商确认-卖出-拒绝       |
| `OPT_FICC_ENQUIRY_DECLARE_BUY`                            | 1138 | 固收业务-询价成交-询价申报买入           |
| `OPT_FICC_ENQUIRY_DECLARE_SELL`                           | 1139 | 固收业务-询价成交-询价申报卖出           |
| `OPT_FICC_ENQUIRY_REPLAY_BUY_CONFIRM`                     | 1140 | 固收业务-询价成交-报价回复-买入-确认       |
| `OPT_FICC_ENQUIRY_REPLAY_BUY_REJECT`                      | 1141 | 固收业务-询价成交-报价回复-买入-拒绝--预留字段 |
| `OPT_FICC_ENQUIRY_REPLAY_SELL_CONFIRM`                    | 1142 | 固收业务-询价成交-报价回复-卖出-确认       |
| `OPT_FICC_ENQUIRY_REPLAY_SELL_REJECT`                     | 1143 | 固收业务-询价成交-报价回复-卖出-拒绝--预留字段 |
| `OPT_FICC_ENQUIRY_INQUIRY_BUY_CONFIRM`                    | 1144 | 固收业务-询价成交-询价成交-买入-确认       |
| `OPT_FICC_ENQUIRY_INQUIRY_BUY_REJECT`                     | 1145 | 固收业务-询价成交-询价成交-买入-拒绝--预留字段 |
| `OPT_FICC_ENQUIRY_INQUIRY_SELL_CONFIRM`                   | 1146 | 固收业务-竞买成交-询价成交-卖出-确认       |
| `OPT_FICC_ENQUIRY_INQUIRY_SELL_REJECT`                    | 1147 | 固收业务-竞买成交-询价成交-卖出-拒绝--预留字段 |
| `OPT_FICC_BINDDING_RESERVE_BUY`                           | 1148 | 固收业务-竞买成交-竞买预约买入           |
| `OPT_FICC_BINDDING_RESERVE_SELL`                          | 1149 | 固收业务-竞买成交-竞买预约卖出           |
| `OPT_FICC_BINDDING_DECLARE_BUY`                           | 1150 | 固收业务-竞买成交-竞买申报买入           |
| `OPT_FICC_BINDDING_DECLARE_SELL`                          | 1151 | 固收业务-竞买成交-竞买申报卖出           |
| `OPT_FICC_BINDDING_PRICE_DECLARE_BUY`                     | 1152 | 固收业务-竞买成交-应价申报买入           |
| `OPT_FICC_BINDDING_PRICE_DECLARE_SELL`                    | 1153 | 固收业务-竞买成交-应价申报卖出           |
| `OPT_OPTION_BUY_CLOSE_THEN_OPEN`                          | 1154 | 买入优先平仓，个股期权交易业务补充类型        |
| `OPT_OPTION_SELL_CLOSE_THEN_OPEN`                         | 1155 | 卖出优先平仓                     |
| `OPT_FUND_TRANSFER_IN`                                    | 1156 | 资金划入                       |
| `OPT_FUND_TRANSFER_OUT`                                   | 1157 | 资金划出                       |

### enum_EOrderType - 算法交易、普通交易类型

| 变量名称                                        | 数值  | 描述            |
| ------------------------------------------- | --- | ------------- |
| `OTP_ORDINARY`                              | 0   | 常规            |
| `OTP_ALGORITHM`                             | 1   | 算法交易          |
| `OTP_RANDVOLUME`                            | 2   | 随机量交易         |
| `OTP_ALGORITHM3`                            | 3   | 算法交易3         |
| `OTP_ZXJT`                                  | 4   | 中信建投算法        |
| `OTP_ZSGS`                                  | 5   | 隔时交易          |
| `OTP_ORDINARY_BASKET_TRIGGER_SINGLE_ORDER`  | 6   | 普通交易的触价单笔委托方式 |
| `OTP_ALGORITHM_BASKET_TRIGGER_SINGLE_ORDER` | 7   | 算法交易的触价单笔委托方式 |
| `OTP_ZXZQ`                                  | 8   | 中信证券算法        |
| `OTP_GENUS`                                 | 9   | 金纳算法          |
| `OTP_JAZZ`                                  | 10  | 爵士算法          |
| `OTP_VWAP`                                  | 11  | 智能VWAP        |
| `OTP_TWAP`                                  | 12  | 智能TWAP        |
| `OTP_XTALGO`                                | 13  | 智能算法          |
| `OTP_HUACHUANG`                             | 14  | 华创算法          |
| `OTP_HUARUN`                                | 15  | 华润算法          |
| `OTP_CUSTOM`                                | 16  | 回转算法          |
| `OPT_EXTERN`                                | 17  | 主动算法          |
| `OTP_GUANGFA`                               | 18  | 广发算法          |

### enum_EPriceType - 价格类型

| 变量名称                                 | 数值  | 描述             |
| ------------------------------------ | --- | -------------- |
| `PRTP_SALE5`                         | 0   | 卖5             |
| `PRTP_SALE4`                         | 1   | 卖4             |
| `PRTP_SALE3`                         | 2   | 卖3             |
| `PRTP_SALE2`                         | 3   | 卖2             |
| `PRTP_SALE1`                         | 4   | 卖1             |
| `PRTP_LATEST`                        | 5   | 最新价            |
| `PRTP_BUY1`                          | 6   | 买1             |
| `PRTP_BUY2`                          | 7   | 买2             |
| `PRTP_BUY3`                          | 8   | 买3             |
| `PRTP_BUY4`                          | 9   | 买4             |
| `PRTP_BUY5`                          | 10  | 买5             |
| `PRTP_FIX`                           | 11  | 指定价            |
| `PRTP_MARKET`                        | 12  | 市价_涨跌停价        |
| `PRTP_HANG`                          | 13  | 挂单价            |
| `PRTP_COMPETE`                       | 14  | 对手价            |
| `PRTP_AUTO`                          | 15  | 自动盘口           |
| `PRTP_CLOSE`                         | 16  | 昨收价            |
| `PRTP_AVERAGE`                       | 17  | 大宗加权平均价        |
| `PRTP_MARKET_BEST`                   | 18  | 市价_最优价         |
| `PRTP_MARKET_CANCEL`                 | 19  | 市价_即成剩撤        |
| `PRTP_MARKET_CANCEL_ALL`             | 20  | 市价_全额成交或撤      |
| `PRTP_MARKET_CANCEL_1`               | 21  | 市价_最优1档即成剩撤    |
| `PRTP_MARKET_CANCEL_5`               | 22  | 市价_最优5档即成剩撤    |
| `PRTP_MARKET_CONVERT_1`              | 23  | 市价_最优1档即成剩转    |
| `PRTP_MARKET_CONVERT_5`              | 24  | 市价_最优5档即成剩转    |
| `PRTP_STK_OPTION_ASK`                | 25  | 询价             |
| `PRTP_STK_OPTION_FIX_CANCEL_ALL`     | 26  | 限价即时全部成交否则撤单   |
| `PRTP_STK_OPTION_MARKET_CACEL_LEFT`  | 27  | 市价即时成交剩余撤单     |
| `PRTP_STK_OPTION_MARKET_CANCEL_ALL`  | 28  | 市价即时全部成交否则撤单   |
| `PRTP_STK_OPTION_MARKET_CONVERT_FIX` | 29  | 市价剩余转限价        |
| `PRTP_SALE6`                         | 30  | 卖6             |
| `PRTP_SALE7`                         | 31  | 卖7             |
| `PRTP_SALE8`                         | 32  | 卖8             |
| `PRTP_SALE9`                         | 33  | 卖9             |
| `PRTP_SALE10`                        | 34  | 卖10            |
| `PRTP_BUY6`                          | 35  | 买6             |
| `PRTP_BUY7`                          | 36  | 买7             |
| `PRTP_BUY8`                          | 37  | 买8             |
| `PRTP_BUY9`                          | 38  | 买9             |
| `PRTP_BUY10`                         | 39  | 买10            |
| `PRTP_UPPER_LIMIT_PRICE`             | 40  | 涨停价            |
| `PRTP_LOWER_LIMIT_PRICE`             | 41  | 跌停价            |
| `PRTP_MARKET_SH_CONVERT_5_CANCEL`    | 42  | 最优五档即时成交剩余撤销   |
| `PRTP_MARKET_SH_CONVERT_5_LIMIT`     | 43  | 最优五档即时成交剩转限价   |
| `PRTP_MARKET_PEER_PRICE_FIRST`       | 44  | 对手方最优价格委托      |
| `PRTP_MARKET_MINE_PRICE_FIRST`       | 45  | 本方最优价格委托       |
| `PRTP_MARKET_SZ_INSTBUSI_RESTCANCEL` | 46  | 即时成交剩余撤销委托     |
| `PRTP_MARKET_SZ_CONVERT_5_CANCEL`    | 47  | 最优五档即时成交剩余撤销委托 |
| `PRTP_MARKET_SZ_FULL_REAL_CANCEL`    | 48  | 全额成交或撤销委托      |
| `PRTP_AFTER_FIX_PRICE`               | 49  | 盘后定价申报         |

### enum_ETaskStatus - 任务状态

| 变量名称                                         | 数值  | 描述               |
| -------------------------------------------- | --- | ---------------- |
| `TASK_STATUS_UNKNOWN`                        | 0   | 未知               |
| `TASK_STATUS_WAITING`                        | 1   | 等待               |
| `TASK_STATUS_COMMITING`                      | 2   | 提交中              |
| `TASK_STATUS_RUNNING`                        | 3   | 执行中              |
| `TASK_STATUS_PAUSE`                          | 4   | 暂停               |
| `TASK_STATUS_CANCELING_DEPRECATED`           | 5   | 撤销中（已弃用）         |
| `TASK_STATUS_EXCEPTION_CANCELING_DEPRECATED` | 6   | 异常撤销中（已弃用）       |
| `TASK_STATUS_COMPLETED`                      | 7   | 完成               |
| `TASK_STATUS_CANCELED`                       | 8   | 已撤               |
| `TASK_STATUS_REJECTED`                       | 9   | 打回               |
| `TASK_STATUS_EXCEPTION_CANCELED`             | 10  | 异常终止             |
| `TASK_STATUS_DROPPED`                        | 11  | 放弃（用于组合交易中，放弃补单） |
| `TASK_STATUS_FORCE_CANCELED_DEPRECATED`      | 12  | 强制终止（已弃用）        |

## 获取行情示例

### 按品种划分

#### 两融

##### 获取融资融券账户可融资买入标的

python

```
#coding:gbk
def init(C):

    r = get_assure_contract('123456789')
    if len(r) == 0:
        print('未取到担保明细')
    else:
        finable = [o.m_strInstrumentID+'.'+o.m_strExchangeID for o in r if o.m_eFinStatus==48]
        print('可融资买入标的:', finable)
```

### 按功能划分

#### 订阅K线全推

提示

1. K线全推需要[VIP权限](http://dict.thinktrader.net/dictionary/?id=null#vip-%E8%A1%8C%E6%83%85%E7%94%A8%E6%88%B7%E4%BC%98%E5%8A%BF%E5%AF%B9%E6%AF%94)，非VIP用户请勿使用此功能

订阅全市场1m周期K线

python

```
#coding:gbk

import pandas as pd
import numpy as np

def init(C):
    stock_list = C.get_stock_list_in_sector("沪深A股")
    sub_num_dict = {i:C.subscribe_quote(
        stock_code = i, 
        period = '1m',  
        dividend_type = 'none',
        result_type = 'dict',  # 回调函数的行情数据格式
        callback = call_back  # 指定一个自定义的函数接收行情，自定义的函数只能有一个位置参数
        ) for i in stock_list}

def call_back(data):
    print(data)
```

#### 获取N分钟周期K线数据

提示

1. 获取历史N分钟数据前，需要先下载历史数据
2. 1m以上，5m以下的数据，是通过1m数据合成的
3. 5m以上，1d以下的数据，是通过5m数据合成的
4. 1d以上的数据，是通过1d的数据合成的

python

```
#coding:gbk

import pandas as pd
import numpy as np

def init(C):

    # start_date = '20231001'# 格式"YYYYMMDD"，开始下载的日期，date = ""时全量下载
    start_date = '20231001'# 格式"YYYYMMDD"，开始下载的日期，date = ""时增量下载
    end_date = "" # 格式同上，下载结束时间
    period = "3m" # 数据周期
    need_download = 1 # 取数据是空值时，将need_download赋值为1，确保正确下载了历史数据
    # code_list = ["110052.SH"] # 可转债
    # code_list = ["rb2401.SF", "FG403.ZF"] # 期货列表
    # code_list = ["HO2310-P-2500.IF"] # 期权列表
    code_list = ["000001.SZ", "600519.SH"] # 股票列表

    # 判断要不要下载数据
    if need_download:
        my_download(code_list, period, start_date, end_date)
    # 取数据
    data = C.get_market_data_ex([],code_list,period = period, start_time = start_date, end_time = end_date,dividend_type = "back_ratio")

    print(data)# 行情数据查看
    print(C.get_instrumentdetail(code_list[0])) # 合约信息查看

def hanldebar(C):
    return

def my_download(stock_list,period,start_date = '', end_date = ''):
    '''
    用于显示下载进度
    '''
    if "d" in period:
        period = "1d"
    elif "m" in period:
        if int(period[0]) < 5:
            period = "1m"
        else:
            period = "5m"
    elif "tick" == period:
        pass
    else:
        raise KeyboardInterrupt("周期传入错误")


    n = 1
    num = len(stock_list)
    for i in stock_list:
        print(f"当前正在下载{n}/{num}")

        download_history_data(i,period,start_date, end_date)
        n += 1
    print("下载任务结束")
```

#### 获取 Lv1 行情数据

本示例用于说明如何通过函数获取行情数据。

python

```
#coding:gbk
# get_market_data_ex(subscribe=True)有订阅股票数量限制
# 即stock_list参数的数量不能超过500

# get_market_data_ex(subscribe=False) 该模式下（非订阅模式），接口会从本地行情文件里获取数据，不会获取动态行情数，且不受订阅数限制，但需要提前下载数据
# 下载数据在 操作/数据管理/补充数据选项卡里，按照页面提示下载数据

# get_market_data_ex(subscribe=True) 该模式下（订阅模式），受订阅数量上限限制，可以取到动态行情

# 建议每天盘后增量补充对应周期的行情

import time


def init(C):
    C.stock = C.stockcode + '.' + C.market
    # 获取指定时间的k线
    price = C.get_market_data_ex(['open','high','low','close'], [C.stock], start_time='', end_time='',period='1d', subscribe=False)
    print(price[C.stock].head())


def handlebar(C):

    bar_timetag = C.get_bar_timetag(C.barpos)
    bar_date = timetag_to_datetime(bar_timetag, '%Y%m%d%H%M%S')
    print('获取截至到%s为止前5根k线的开高低收等字段:'%(bar_date))
    # 获取截至今天为止前30根k线
    price = C.get_market_data_ex(
                    [], [C.stock], 
                    end_time=bar_date,
                    period=C.period, 
                    subscribe=True, 
                    count=5,
                    )
    print(price[C.stock].to_dict('dict'))
```

#### 获取 Lv2 数据（需要数据源支持）

##### 方法1 - 查询LV2数据

使用该函数后，会定期查询最新数据，并进行数据返回。

python返回值

```
#coding:gbk
def init(C):
    C.sub_nums  = []

    C.stock = C.stockcode+'.'+C.market
    for field in ['l2transaction', 'l2order', 'l2transactioncount', 'l2quote']:
        num = C.subscribe_quote(C.stock, period=field,
                            dividend_type='follow',
                            )

        C.sub_nums.append(num)

def handlebar(C):
    if not C.is_last_bar():
        return
    price = C.get_market_data_ex([],[C.stock],period='l2transaction',count=10)[C.stock]
    price_dict = price.to_dict('index')
    print(price_dict)
    for pos, t in enumerate(price_dict):
        print(f" 逐笔成交:{pos+1} 时间:{price_dict[t]['stime']}, 时间戳:{price_dict[t]['time']}, 成交价:{price_dict[t]['price']}, \
成交量:{price_dict[t]['volume']}, 成交额:{price_dict[t]['amount']} \
成交记录号:{price_dict[t]['tradeIndex']}, 买方委托号:{price_dict[t]['buyNo']},\
卖方委托号:{price_dict[t]['sellNo']}, 成交类型:{price_dict[t]['tradeType']}, \
成交标志:{price_dict[t]['tradeFlag']}, ")

    price = C.get_market_data_ex([],[C.stock],period='l2quote',count=10)[C.stock]
    price_dict = price.to_dict('index')
    print(price_dict)
    for pos, t in enumerate(price_dict):
        print(f" 十档快照:{pos+1} 时间:{price_dict[t]['stime']}, 时间戳:{price_dict[t]['time']}, 最新价:{price_dict[t]['lastPrice']}, \
开盘价:{price_dict[t]['open']}, 最高价:{price_dict[t]['high']} 最低价:{price_dict[t]['low']}, 成交额:{price_dict[t]['amount']},\
成交总量:{price_dict[t]['volume']}, 原始成交总量:{price_dict[t]['pvolume']}, 证券状态:{price_dict[t]['stockStatus']}, 持仓量:{price_dict[t]['openInt']},\
成交笔数:{price_dict[t]['transactionNum']},前收盘价:{price_dict[t]['lastClose']},多档委卖价:{price_dict[t]['askPrice']},多档委卖量:{price_dict[t]['askVol']},\
多档委买价:{price_dict[t]['bidPrice']},多档委买量:{price_dict[t]['bidVol']}")


    price = C.get_market_data_ex([],[C.stock],period='l2order',count=10)[C.stock]
    price_dict = price.to_dict('index')
    for pos, t in enumerate(price_dict):
        print(f" 逐笔委托:{pos+1} 时间:{price_dict[t]['stime']}, 时间戳:{price_dict[t]['time']}, 委托价:{price_dict[t]['price']}, \
委托量:{price_dict[t]['volume']}, 委托号:{price_dict[t]['entrustNo']} \
委托类型:{price_dict[t]['entrustType']}, 委托方向:{price_dict[t]['entrustDirection']},\
")
# 委托类型: 0：未知 1: 买入,2: 卖出,3: 撤单


    price = C.get_market_data_ex([],[C.stock],period='l2transactioncount',count=10)[C.stock]
    price_dict = price.to_dict('index')
    for pos, t in enumerate(price_dict):
        print('大单统计:', price_dict[t])

def stop(C):
    for num in C.sub_nums:
        C.unsubscribe_quote(num)
```

##### 方法2 - 订阅LV2数据

此方法在发起订阅后，会自动收到所订阅数据，订阅方需要记录订阅函数返回的订阅号，并在不需要订阅时调用unsubscribe_quote反订阅数据，释放资源。

python返回值

```
#coding:gbk

def l2_quote_callback(data):
    for s in data:
        print('lv2快照:',s, data[s])

def l2transaction_callback(data):
    for s in data:
        print('逐笔成交',s, data[s])


def l2order_callback(data):
    for s in data:
        print('逐笔委托',s, data[s])

def l2quoteaux_callback(data):
    for s in data:
        print('行情快照补充',s, data[s])


def l2transactioncount_callback(data):
    for s in data:
        print('大单统计',s, data[s])


def l2orderqueue_callback(data):
    for s in data:
        print('委买委卖队列',s, data[s])


def init(C):
    C.stock = C.stockcode + '.' + C.market

    # Level2 逐笔快照
    C.subscribe_quote(C.stock, 'l2quote', result_type='dict', callback=l2_quote_callback)
    # Level2 行情快照补充
    C.subscribe_quote(C.stock, 'l2quoteaux', result_type='dict', callback=l2quoteaux_callback)
    # Level2 逐笔成交
    C.subscribe_quote(C.stock, 'l2transaction', result_type='dict', callback=l2transaction_callback)
    # Level2 逐笔委托
    C.subscribe_quote(C.stock, 'l2order', result_type='dict', callback=l2order_callback)
    # Level2大单统计
    C.subscribe_quote(C.stock, 'l2transactioncount', result_type='dict', callback=l2transactioncount_callback)
    # Level2委买委卖队列
    C.subscribe_quote(C.stock, 'l2orderqueue', result_type='dict', callback=l2orderqueue_callback)

def handlebar(C):
    return
```

#### 使用 Lv1 全推数据计算全市场涨幅

python

```
#coding:gbk

import time

class a():pass

A = a()

def init(C):
    A.hsa = C.get_stock_list_in_sector('沪深A股') + C.get_stock_list_in_sector('京市A股')
    print('股票池大小', len(A.hsa))
    A.vol_dict = {}
    for stock in A.hsa:
            A.vol_dict[stock] = C.get_last_volume(stock)
    C.run_time("f","3nSecond","2019-10-14 13:20:00")


def to_zw(a):
    '''0.中文价格字符串'''
    import numpy as np
    if np.isnan(a):
            return '问题数据'
    if abs(a) < 1000:
            print(a, str(round(int(a) / 1000.0, 2)) + "千")
            return str(round(int(a) / 1000.0, 2)) + "千"
    if abs(a) < 10000:
            return str(int(a))[0] + "千"
    if abs(a) < 100000000:
            return str(int(a))[:-4] + "万" + str(int(a))[-4]
    return f"{int(a / 100000000)}亿"



def f(C):
    t0 = time.time()
    full_tick = C.get_full_tick(A.hsa)
    total_market_value = 0
    total_ratio = 0
    count = 0
    total_amount = 0
    ratio_list = []
    for stock in A.hsa:
            ratio = full_tick[stock]['lastPrice'] / full_tick[stock]['lastClose'] - 1
            amount = full_tick[stock]['amount']
            total_amount += amount
            rise_price = round(full_tick[stock]['lastClose'] *1.2,2) if stock[0] == '3' or stock[:3] == '688' else round(full_tick[stock]['lastClose'] *1.1,2)
            #如果要打印涨停品种
            #if abs(full_tick[stock]['lastPrice'] - rise_price) <0.01:
            #        print(f"涨停品种 {stock} {C.get_stock_name(stock)}")
            market_value = full_tick[stock]['lastPrice'] * A.vol_dict[stock]
            total_ratio += ratio * market_value
            total_market_value += market_value
            count += 1
            ratio_list.append(ratio)
    #print(count)
    total_ratio /= total_market_value
    total_ratio *= 100
    middle  = int(len(ratio_list) / 2)
    middle_ratio = ratio_list[middle]
    rise_num = len([i for i in ratio_list if i > 0])
    down_num = len([i for i in ratio_list if i < 0])
    print(f'A股加权涨幅 {round(total_ratio,2)}% 涨幅中位数 {round(middle_ratio,2)}% {rise_num}个上涨 {down_num}个下跌 成交金额 {to_zw(total_amount)} 耗时{round(time.time()- t0,5)}秒')
```

#### 在行情回调函数里处理动态行情

[ContextInfo.subscribe_quote - 订阅行情函数说明](https://dict.thinktrader.net/innerApi/data_function.html#contextinfosubscribe_quote---%E8%AE%A2%E9%98%85%E8%A1%8C%E6%83%85%E6%95%B0%E6%8D%AE)  
[行情回调函数字段说明](https://dict.thinktrader.net/innerApi/data_structure.html#%E4%BA%A4%E6%98%93%E7%B1%BB)

python

```
#coding:gbk

sub_nums = []


def init(C):
    global sub_nums

    # def on_quote(data1， data2): # 错误写法
    def on_quote(data):
        for s in data:
            q = data[s]
            print(type(q), q)

    stocks = [C.stockcode + '.' + C.market]  # 获取到当前主图股票代码
    for s in stocks:
        num = C.subscribe_quote(s, period='1d',  
                          dividend_type='none',
                          result_type='dict',  # 回调函数的行情数据格式
                          callback=on_quote  # 指定一个自定义的函数接收行情，自定义的函数只能有一个位置参数
                        )
        sub_nums.append(num)


def stop(C):
    # 反订阅
    for num in sub_nums:
        C.unsubscribe_quote(num)
```

#### python写入扩展数据

python

```
# coding:gbk
'''
python写扩展数据，投研接口
'''

def init(C):
    # 创建扩展数据
    extencd_name  = 'test' # 创建名为test的扩展数据
    create_extend_data# (父节点, 扩展数据名称, 是否覆盖)
    C.extencd_name = create_extend_data('扩展数据', extencd_name, True)


def handlebar(C):
    if C.is_last_bar():
        data = {'SH600177': 0.43, 'SZ000767': 0.18, 'SH600362': 0.27, 'SH600171': 0.25, 'SH600170': 0.18, 'SH600073': 0.13, 'SZ000768': 0.17, 'SH600282': 0.19, 'SH600601': 0.42, 'SH600569': 0.26, 'SZ000401': 0.21, 'SH600602': 0.17, 'SZ000806': 0.13, 'SZ000807': 0.15, 'SH600608': 0.08, 'SH600874': 0.09, 'SZ000825': 0.44, 'SZ000652': 0.19, 'SH600078': 0.11, 'SH600871': 0.1, 'SZ000573': 0.1, 'SZ000520': 0.14, 'SH600879': 0.43, 'SZ000960': 0.2, 'SH600597': 0.21, 'SZ000550': 0.1, 'SH600591': 0.14, 'SZ000059': 0.13, 'SH600215': 0.12, 'SZ000968': 0.11, 'SZ000969': 0.14, 'SZ000568': 0.22, 'SH600598': 0.22, 'SH600028': 1.85, 'SH600270': 0.27, 'SH600060': 0.22, 'SH600062': 0.15, 'SH600779': 0.14, 'SH600997': 0.2, 'SZ000707': 0.12, 'SH600068': 0.16, 'SH600770': 0.14, 'SZ000680': 0.14, 'SH600674': 0.1, 'SH600675': 0.25, 'SZ000488': 0.28, 'SZ000012': 0.11, 'SH600863': 0.17, 'SZ000429': 0.17, 'SH600027': 0.32, 'SH600866': 0.13, 'SH600001': 0.43, 'SZ000636': 0.11, 'SH600900': 2.73, 'SH600600': 0.31, 'SH600895': 0.26, 'SH600029': 0.43, 'SH600020': 0.24, 'SH600205': 0.39, 'SH600688': 0.76, 'SH600207': 0.1, 'SZ000970': 0.3, 'SZ000601': 0.19, 'SH600200': 0.35, 'SZ000975': 0.08, 'SZ000538': 0.39, 'SZ000422': 0.14, 'SZ000858': 0.88, 'SZ000651': 0.44, 'SH600780': 0.21, 'SH600007': 0.11, 'SH600016': 2.36, 'SZ000866': 0.84, 'SH600267': 0.12, 'SH600266': 0.16, 'SH600786': 0.27, 'SZ000406': 0.39, 'SH600269': 0.47, 'SZ000528': 0.19, 'SH600694': 0.51, 'SZ000786': 0.14, 'SH600004': 0.4, 'SH600663': 0.25, 'SH600662': 0.21, 'SH600548': 0.11, 'SH600383': 0.42, 'SH600357': 0.12, 'SH600705': 0.13, 'SH600812': 0.18, 'SH600707': 0.06, 'SZ000895': 0.49, 'SZ000898': 0.68, 'SZ000400': 0.17, 'SZ000607': 0.1, 'SH600894': 0.1, 'SH600418': 0.4, 'SZ000061': 0.15, 'SH600653': 0.31, 'SZ000682': 0.17, 'SZ000543': 0.07, 'SZ000541': 0.25, 'SH600377': 0.12, 'SZ000949': 0.06, 'SH600256': 0.17, 'SH600006': 0.23, 'SH600005': 1.13, 'SH600790': 0.1, 'SH600797': 0.2, 'SH600002': 0.51, 'SH600795': 0.49, 'SH600000': 1.64, 'SH600652': 0.17, 'SH600121': 0.13, 'SH600123': 0.3, 'SH600125': 0.27, 'SH600126': 0.12, 'SH600008': 0.51, 'SZ000016': 0.14, 'SH600550': 0.24, 'SH600718': 0.16, 'SH600654': 0.2, 'SZ000157': 0.2, 'SH600808': 0.42, 'SZ000666': 0.1, 'SH600805': 0.11, 'SZ000527': 0.39, 'SH600717': 0.5, 'SH600399': 0.07, 'SZ000828': 0.14, 'SZ000959': 0.17, 'SZ000729': 0.29, 'SH600098': 0.43, 'SZ000886': 0.09, 'SZ000099': 0.13, 'SZ000800': 0.29, 'SH600096': 0.29, 'SZ001872': 0.17, 'SH600091': 0.12, 'SZ000956': 0.41, 'SH600887': 0.63, 'SH600886': 0.2, 'SH600884': 0.12, 'SH600308': 0.32, 'SH600309': 0.64, 'SH600881': 0.21, 'SH600307': 0.14, 'SZ000069': 0.8, 'SZ000068': 0.1, 'SH600153': 0.19, 'SZ000792': 0.55, 'SZ000060': 0.38, 'SZ000002': 2.25, 'SZ000001': 1.25, 'SH600138': 0.1, 'SH600649': 0.44, 'SH600015': 0.99, 'SZ000533': 0.07, 'SZ000009': 0.24, 'SH600408': 0.09, 'SZ000778': 0.23, 'SH600643': 0.17, 'SH600642': 0.71, 'SH600832': 0.71, 'SZ000089': 0.36, 'SZ000088': 0.44, 'SZ000539': 0.3, 'SH600835': 0.19, 'SH600726': 0.09, 'SH600010': 0.42, 'SH600724': 0.16, 'SH600839': 0.61, 'SH600011': 0.38, 'SH600012': 0.27, 'SH600089': 0.23, 'SH600088': 0.12, 'SH600087': 0.12, 'SH600085': 0.38, 'SZ000927': 0.2, 'SZ000920': 0.11, 'SZ000962': 0.13, 'SZ000559': 0.16, 'SH600050': 2.98, 'SZ000708': 0.09, 'SZ000503': 0.36, 'SZ000839': 0.44, 'SZ000717': 0.28, 'SZ000100': 0.31, 'SZ000036': 0.18, 'SZ000878': 0.24, 'SZ000511': 0.09, 'SZ000410': 0.19, 'SH600033': 0.24, 'SH600108': 0.13, 'SZ000031': 0.21, 'SZ000507': 0.09, 'SH600104': 0.78, 'SZ002024': 0.45, 'SH600102': 0.18, 'SH600103': 0.14, 'SH600100': 0.41, 'SH600019': 2.84, 'SH600009': 1.5, 'SH600639': 0.19, 'SZ000709': 0.34, 'SH600820': 0.14, 'SZ000571': 0.1, 'SH600739': 0.13, 'SH600631': 0.25, 'SH600021': 0.24, 'SH600635': 0.22, 'SH600637': 0.15, 'SZ000733': 0.09, 'SZ000780': 0.09, 'SH600210': 0.25, 'SZ000939': 0.12, 'SZ000937': 0.26, 'SZ000875': 0.15, 'SZ000933': 0.22, 'SZ000932': 0.51, 'SZ000659': 0.15, 'SZ000930': 0.38, 'SH600320': 0.8, 'SZ000822': 0.17, 'SZ000726': 0.14, 'SZ000727': 0.1, 'SZ000725': 0.13, 'SH600380': 0.14, 'SH600030': 0.63, 'SH600031': 0.16, 'SH600036': 3.93, 'SH600037': 0.62, 'SH600035': 0.16, 'SH600058': 0.2, 'SH600110': 0.13, 'SH600508': 0.19, 'SZ000402': 0.62, 'SH600115': 0.1, 'SH600117': 0.13, 'SZ000423': 0.22, 'SH600348': 0.34, 'SZ000518': 0.14, 'SH600500': 0.26, 'SH600660': 0.37, 'SH600057': 0.09, 'SH600744': 0.12, 'SH600747': 0.11, 'SH600740': 0.09, 'SH600741': 0.17, 'SH600621': 0.11, 'SZ000698': 0.14, 'SH600851': 0.19, 'SZ000900': 0.28, 'SH600854': 0.1, 'SH600868': 0.25, 'SH600428': 0.3, 'SZ000630': 0.36, 'SH600811': 0.3, 'SZ000735': 0.1, 'SZ000737': 0.09, 'SZ000066': 0.19, 'SH600331': 0.25, 'SH600183': 0.27, 'SH600333': 0.1, 'SZ000581': 0.25, 'SZ000425': 0.11, 'SZ000983': 0.49, 'SH600236': 0.25, 'SH600026': 0.45, 'SH600339': 0.08, 'SH600231': 0.16, 'SH600188': 0.27, 'SH600022': 0.2, 'SH600166': 0.08, 'SH600350': 0.37, 'SH600519': 1.14, 'SZ000063': 1.49, 'SH600690': 0.44, 'SZ000758': 0.24, 'SH600296': 0.26, 'SH601607': 0.17, 'SHT00018': 0.8, 'SZ000039': 0.78, 'SZ000599': 0.1, 'SH600198': 0.21, 'SZ000917': 0.15, 'SZ000916': 0.22, 'SZ000912': 0.21, 'SH600190': 0.11, 'SH600196': 0.25, 'SH600583': 0.54, 'SH600581': 0.12, 'SZ000027': 0.6, 'SZ000629': 0.52, 'SH600585': 0.32, 'SZ000021': 0.26, 'SZ000625': 0.25, 'SZ000623': 0.18, 'SZ000024': 0.42, 'SH600221': 0.13, 'SH600220': 0.17}
        timetag = C.get_bar_timetag(C.barpos)
        stock_list = list(data.keys())
        print(stock_list)

        #设置需要写入的扩展数据股票列表
        #reset_extend_data_stock_list(扩展数据名称, 股票列表)
        reset_extend_data_stock_list(C.extencd_name, stock_list)
        # 执行写入扩展数据
        # set_extend_data_value（扩展数据名称，时间戳(毫秒)，数据）
        set_extend_data_value(C.extencd_name, timetag, data)  
        print('写入完成')
```

### 扩展数据展示

![扩展数据](https://dict.thinktrader.net/assets/%E5%86%85%E7%BD%AEAPI_%E6%89%A9%E5%B1%95%E6%95%B0%E6%8D%AE-58079b49.png)

#### 每1分钟统计一次市场涨跌情况

示例

```
# coding:gbk
import datetime as dt
def on_timer(ContextInfo):
    ls = globals().get("stock_list")
    now_time = dt.datetime.now().strftime("%Y%m%d %H:%M:%S")
    # 取tick数据
    ticks = ContextInfo.get_full_tick(ls)

    # 涨跌统计,并去除停牌股
    profit_ls = [i for i in ticks if ticks[i]["lastPrice"] > ticks[i]["lastClose"] and ticks[i]["openInt"] != 1]
    loss_ls = [i for i in ticks if ticks[i]["lastPrice"] < ticks[i]["lastClose"] and ticks[i]["openInt"] != 1]
    print(f"{now_time}: 涨家数{len(profit_ls)}; 跌家数{len(loss_ls)}")

def init(ContextInfo):
    globals()["stock_list"] = get_stock_list_in_sector("沪深京A股")
    # 自2023-12-31 23:59:59后每60s运行一次on_timer
    tid=ContextInfo.schedule_run(on_timer,'20231231235959',3,dt.timedelta(seconds=60),'my_timer')
    # 取消任务组为"my_timer"的任务
    # # ContextInfo.cancel_schedule_run('my_timer')

    # 关于schedule_run函数的用法请阅读文档http://dict.thinktrader.net/innerApi/system_function.html?id=7zqjlm#contextinfo-schedule-run-%E8%AE%BE%E7%BD%AE%E5%AE%9A%E6%97%B6%E5%99%A8
```

## 交易下单示例

### 按品种划分

#### 股票

python

```
#coding:gbk
def handlebar(ContextInfo):
    if not ContextInfo.is_last_bar():
        return
    # 单股单账号股票最新价买入 100 股（1 手）   
    passorder(23, 1101, 'test', '600000.SH', 5, 0, 100, '',1,'',ContextInfo)
    # 单股单账号股票最新价卖出 100 股（1 手）   
    passorder(24, 1101, 'test', '600000.SH', 5, 0, 100, '',1,'',ContextInfo)
    # 单股单账号沪市股票市价买入 100 股（1 手），沪市市价存在保护限价，Price参数为保护限价，买入为投资者能够接受的最高买价，填0会自动填为涨停价
    passorder(23, 1101, 'test', '600000.SH', 42, 0, 100, '',1,'',ContextInfo)
    # 单股单账号沪市股票市价卖出 100 股（1 手），沪市市价存在保护限价，Price参数为保护限价，卖出为投资者能够接受的最低卖价，填0会自动填为跌停价
    passorder(24, 1101, 'test', '600000.SH', 42, 0, 100, '',1,'',ContextInfo)
    # 单股单账号京市股票最新价买入 101 股（1 手零 1 股）   
    passorder(23, 1101, 'test', '430047.BJ', 5, 0, 101, '',1,'',ContextInfo)
    # 单股单账号京市股票最新价卖出 101 股（1 手零 1 股）  
    passorder(24, 1101, 'test', '430047.BJ', 5, 0, 101, '',1,'',ContextInfo)
```

#### 基金

python

```
def handlebar(ContextInfo):
    if not ContextInfo.is_last_bar():
        return

    # 申购 中证500指数ETF  
    passorder(60, 1101, 'test', '510030.SH', 5, 0, 1, 2, ContextInfo)  

    # 赎回 中证500指数ETF
    passorder(61, 1101, 'test', '510030.SH', 5, 0, 1, 2, ContextInfo)  
```

#### 两融

python

```
#coding:gbk
def handlebar(ContextInfo):
    if not ContextInfo.is_last_bar():
        return
    target = '000001.SZ'
    # 单股单账号股票指定价担保品买入 100 股（1 手）  
    passorder(33, 1101, 'test', target, 11, 7, 100, ContextInfo)
    # 单股单账号股票指定价融资买入 100 股（1 手）  
    passorder(27, 1101, 'test', target, 11, 7, 100, ContextInfo)
```

#### 期货

python

```
#coding:gbk
def handlebar(ContextInfo):
    if not ContextInfo.is_last_bar():
        return

    # 单股单账号期货最新价开多螺纹钢2401 10 手
    target = 'rb2401.SF'
    passorder(0, 1101, 'test', target, 5, -1, 10, 1, ContextInfo)


    # 单股单账号期货指定价开空甲醇2401 10 手
    target = 'MA401.ZF'
    passorder(3, 1101, 'test', target, 11, 3000, 10, 1, ContextInfo)


    #期货四键平多300股指2311,优先平今 2手
    target = 'IF2311.IF'
    passorder(6, 1101, 'test', target, 5, -1, 2, 1, ContextInfo)
```

#### 期权

python

```
#coding:gbk
def handlebar(ContextInfo):
    if not ContextInfo.is_last_bar():
        return
    target = '10005330.SHO' # 50ETF购12月2450合约

    # 单股单账号用最新价买入开仓期权合约target 2张
    passorder(50, 1101, 'test', target, 5, -1, 2, 1, ContextInfo)

    # 单股单账号用最新价卖出平仓期权合约target 2张
    passorder(51, 1101, 'test', target, 5, -1, 2, 1, ContextInfo)
```

#### 新股申购

python

```
#coding:gbk
def init(C):
    ipoStock=get_ipo_data("STOCK")#返回新股信息
    print(ipoStock)
    accont = '123456789'
    for stock in ipoStock:
        ipo_price = ipoStock[stock]['issuePrice'] # 发行价
        maxPurchaseNum = ipoStock[stock]['maxPurchaseNum'] # 可申购额度
        passorder(23,1101, accont, stock,11,ipo_price, maxPurchaseNum,'新股申购',2,stock,C)
```

#### 债券

python

```
#coding:gbk
def handlebar(ContextInfo):
    if not ContextInfo.is_last_bar():
        return
    # 单股单账号最新价可转债买入 20张
    passorder(23, 1101, 'test', '128123.SZ', 5, -1, 10, 1, ContextInfo)
```

#### ETF

python

```
#coding:gbk
def handlebar(ContextInfo):
    if not ContextInfo.is_last_bar():
        return
    # 单股单账号 最新价买入上证etf 2000份
    passorder(23, 1101, 'test', '510050.SH', 5, -1, 2000, ContextInfo)
```

#### 组合交易

一键买卖（一篮子下单）

**功能描述：** 该示例演示如何用python进行一揽子股票买卖的交易操作

**代码示例：**

python

```
#coding:gbk

def init(C):

    table=[
            {'stock':'600000.SH','weight':0.11,'quantity':100,'optType':23},
            {'stock':'600028.SH','weight':0.11,'quantity':200,'optType':24},
        ]
    basket={'name':'basket1','stocks':table}
    set_basket(basket)
    # 按篮子数量下单， 下2份 # 即下两倍篮子
    pice = 2
    passorder(35,  #一键买卖
            2101,  # 表示按股票数量下单
            account,
            'basket1',  # 篮子名称
            5, # 最新价下单
            1,  # 价格，最新价时 该参数无效，需要填任意数占位
            pice, # 篮子份数
            '',2,'strReMark',C)

    # 按篮子权重下单
    table=[
            {'stock':'600000.SH','weight':0.4,'quantity':0,'optType':23}, # 40%
            {'stock':'600028.SH','weight':0.6,'quantity':0,'optType':24}, # 60%
        ]
    basket={'name':'basket2','stocks':table}
    set_basket(basket)
    # 按组合权重 总额10000元
    money = 10000
    passorder(35,2102,account,'basket2',5,1,money,'',2,'strReMark',C)
```

#### 组合套利交易

提示

（`accountID`、`orderType` 特殊设置）

**用法**

**释义**：

**参数**

| 参数名称       | 描述                                |
| ---------- | --------------------------------- |
| accountID  | 'stockAccountID, futureAccountID' |
| orderCode  | 'basketName, futureName'          |
| hedgeRatio | 套利比例（0 ~ 2 之间值，相当于 %0 至 200% 套利）  |
| volume     | 份数 \ 资金 \ 比例                      |
| orderType  | 参考下方**orderType-下单方式（特殊设置）**      |

**orderType - 下单方式（特殊设置）**

| 编号   | 项目                          |
| ---- | --------------------------- |
| 2331 | 组合、套利、合约价值自动套利、按组合股票数量方式下单  |
| 2332 | 组合、套利、按合约价值自动套利、按组合股票权重方式下单 |
| 2333 | 组合、套利、按合约价值自动套利、按账号可用方式下单   |

**示例**

python返回值

### 按功能划分

#### passorder 下单函数

本示例用于演示K线走完下单及立即下单的参数写法差异，旨在帮助您了解如何快速实现下单操作。

python

```
#coding:gbk
c = 0
s = '000001.SZ'
def init(ContextInfo):
    # 立即下单 用最新价买入股票s 100股，且指定投资备注
    passorder(23,1101,account,s,5,0,100,'1',2,'tzbz',ContextInfo) 
    pass


def handlebar(ContextInfo):
    if not ContextInfo.is_last_bar():
        #历史k线不应该发出实盘信号 跳过
        return

    if ContextInfo.is_last_bar():
        global c
        c +=1 
        if c ==1:
            # 用14.00元限价买入股票s 100股
            passorder(23,1101,account,s,11,14.00,100,1,ContextInfo)  # 当前k线为最新k线 则立即下单
            # 用最新价限价买入股票s 100股
            passorder(23,1101,account,s,5,-1,100,0,ContextInfo)  # K线走完下单
            # 用最新价限价买入股票s 1000元
            passorder(23, 1102, account, s, 5, 0,1000, 2, ContextInfo)  # 不管是不是最新K线，立即下单
```

#### 集合竞价下单

本示例演示了利用定时器函数和passorder下单函数在集合竞价期间以指定价买入平安银行100股。

python

```
#coding:gbk

import time
c = 0
s = '000001.SZ'
def init(ContextInfo):
    # 设置定时器，历史时间表示会在一次间隔时间后开始调用回调函数 比如本例中 5秒后会后第一次触发myHandlebar调用 之后五秒触发一次
    ContextInfo.run_time("myHandlebar","5nSecond","2019-10-14 13:20:00")


def myHandlebar(ContextInfo):
    global c
    now = time.strftime('%H%M%S')
    if c ==0 and '092500' >= now >= '091500':
        c += 1
        passorder(23,1101,account,s,11,14.00,100,2,ContextInfo) # 立即下单

def handlebar(ContextInfo):
    return
```

#### 止盈止损示例

python

```
#coding:gbk

"""
1.账户内所有股票，当股价低于买入价10%止损卖出。
2.账户内所有股票，当股价高于前一天的收盘价10%时，开始监控一旦股价炸板（开板），以买三价卖出
"""

def init(C):
    C.ratio = 1
    if accountType == 'STOCK':
        C.sell_code = 24
    if accountType == 'CREDIT':
        C.sell_code = 34
    C.spare_list = C.get_stock_list_in_sector('不卖品种')

def handlebar(C):
    if not C.is_last_bar():
        return
    holdings = get_trade_detail_data(account, accountType, 'position')
    stock_list = [holding.m_strInstrumentID + '.' + holding.m_strExchangeID for holding in holdings]
    if stock_list:
        full_tick = C.get_full_tick(stock_list)
        for holding in holdings:
            stock = holding.m_strInstrumentID + '.' + holding.m_strExchangeID
            rate = holding.m_dProfitRate
            volume = holding.m_nCanUseVolume
            if not volume >= 100:
                continue
            if stock in C.spare_list:
                continue
            if rate < -0.1:
                msg = f'{stock} 盈亏比例 {rate} 小于-10% 卖出 {volume}股'
                print(msg)
                passorder(C.sell_code, 1101, account, stock, 14, -1, volume, '减仓模型', 2, msg, C)
                continue
            if stock in full_tick:
                current_price = full_tick[stock]['lastPrice']
                pre_price = full_tick[stock]['lastClose']
                high_price = full_tick[stock]['high']
                stop_price = pre_price * 1.2 if stock[:2] in ['30', '68'] else pre_price * 1.1
                stop_price = round(stop_price, 2)
                ask_price_3 = full_tick[stock]['bidPrice'][2]
                if not ask_price_3:
                    print(f"{stock} {full_tick[stock]} 未取到三档盘口价 请检查客户端右下角 行情界面 是否选择了五档行情 本次跳过卖出")
                    continue
                if high_price == stop_price and current_price < stop_price:
                    msg = f"{stock} 涨停后 开板 卖出 {volume}股"
                    print(msg)
                    passorder(C.sell_code, 1101, account, stock, 14, -1, volume, '减仓模型', 2, msg, C)
                    continue
```

#### passorder 下算法单函数

本示例由于演示如何下达算法单，具体算法参数请参考迅投投研平台客户端参数说明。

python

```
#coding:gbk
import time

def init(C):
    userparam={
    'OrderType':1,  #表示要下算法
    'PriceType':0, # 卖5价下单
    'MaxOrderCount':12, # 最大委托次数
    'SuperPriceType':0,  # 超价类型，0表示按比例
    'SuperPriceRate':0.02,  # 超价2%下单
    'VolumeRate':0.1,  # 单笔下单比率 每次拆10%
    'VolumeType': 10,  # 单笔基准量类型
    'SingleNumMax':1000000,  # 单笔拆单最大值
    'PriceRangeType':0,  # 波动区间类型
    'PriceRangeRate':1,  # 波动区间值
    'ValidTimeType':1,  # 有效时间类型 1 表示按执行时间
    'ValidTimeStart':int(time.time()),  # 算法开始时间
    'ValidTimeEnd':int(time.time()+60*60),  # 算法结束时间
    'PlaceOrderInterval':10, # 报撤间隔
    'UndealtEntrustRule':0, # 未成委托处理数值 用卖5加挂单
    }
    target_vol = 2000000  # 股， 算法目标总量
    algo_passorder(23, 1101, account, '600000.SH', -1, -1, target_vol, '', 2, '普通算法', userparam, C)
    print('finish')
def handlebar(C):
    pass
```

#### 如何使用投资备注

投资备注功能是模型下单时指定的任意字符串(长度小于24)，即passorder的userOrderId参数，可以用于匹配委托或成交。有且只有passorder， algo_passorder, smart_algo_passorder下单函数支持投资备注功能。

python

```
# encoding:gbk

note = 0

def get_new_note():
    global note
    note += 1
    return str(note)

def init(ContextInfo):
    ContextInfo.set_account(account)
    passorder(23, 1101, account, '000001.SZ', 5 ,0, 100, '', 2, get_new_note(), ContextInfo)

    orders = get_trade_detail_data(account, accountType, 'order')
    remark = [o.m_strRemark for o in orders]
    sysid_list = [o.m_strOrderSysID for o in orders]
    print(remark)



def handlebar(C):
    pass


def order_callback(C, O):
    print(O.m_strRemark, O.m_strOrderSysID)


def deal_callback(C, D):
    print(D.m_strRemark, D.m_strOrderSysID)
```

#### 如何获取委托持仓及资金数据

本示例用于演示如何通过函数获取指定账户的委托、持仓、资金数据。

python

```
#coding:gbk


def to_dict(obj):
    attr_dict = {}
    for attr in dir(obj):
        try:
            if attr[:2] == 'm_':
                attr_dict[attr] = getattr(obj, attr)
        except:
            pass
    return attr_dict


def init(C):
    pass
    #orders, deals, positions, accounts = query_info(C)


def handlebar(C):
    if not C.is_last_bar():
        return
    orders, deals, positions, accounts = query_info(C)


def query_info(C):
    orders = get_trade_detail_data('8000000213', 'stock', 'order')
    for o in orders:
        print(f'股票代码: {o.m_strInstrumentID}, 市场类型: {o.m_strExchangeID}, 证券名称: {o.m_strInstrumentName}, 买卖方向: {o.m_nOffsetFlag}',
        f'委托数量: {o.m_nVolumeTotalOriginal}, 成交均价: {o.m_dTradedPrice}, 成交数量: {o.m_nVolumeTraded}, 成交金额:{o.m_dTradeAmount}')


    deals = get_trade_detail_data('8000000213', 'stock', 'deal')
    for dt in deals:
        print(f'股票代码: {dt.m_strInstrumentID}, 市场类型: {dt.m_strExchangeID}, 证券名称: {dt.m_strInstrumentName}, 买卖方向: {dt.m_nOffsetFlag}', 
        f'成交价格: {dt.m_dPrice}, 成交数量: {dt.m_nVolume}, 成交金额: {dt.m_dTradeAmount}')

    positions = get_trade_detail_data('8000000213', 'stock', 'position')
    for dt in positions:
        print(f'股票代码: {dt.m_strInstrumentID}, 市场类型: {dt.m_strExchangeID}, 证券名称: {dt.m_strInstrumentName}, 持仓量: {dt.m_nVolume}, 可用数量: {dt.m_nCanUseVolume}',
        f'成本价: {dt.m_dOpenPrice:.2f}, 市值: {dt.m_dInstrumentValue:.2f}, 持仓成本: {dt.m_dPositionCost:.2f}, 盈亏: {dt.m_dPositionProfit:.2f}')


    accounts = get_trade_detail_data('8000000213', 'stock', 'account')
    for dt in accounts:
        print(f'总资产: {dt.m_dBalance:.2f}, 净资产: {dt.m_dAssureAsset:.2f}, 总市值: {dt.m_dInstrumentValue:.2f}', 
        f'总负债: {dt.m_dTotalDebit:.2f}, 可用金额: {dt.m_dAvailable:.2f}, 盈亏: {dt.m_dPositionProfit:.2f}')

    return orders, deals, positions, accounts
```

#### 使用快速交易参数委托

本例展示如何使用快速交易参数(quickTrade)立刻进行委托。

python

```
#coding:gbk

def after_init(C):
    #account变量是模型交易界面 添加策略时选择的资金账号 不需要手动填写
    #快速交易参数(quickTrade )填2 passorder函数执行后立刻下单 不会等待k线走完再委托。 可以在after_init函数 run_time函数注册的回调函数里进行委托 
    msg = f"投资备注字符串 用来区分不同委托"
    passorder(23, 1101, account, '600000.SH', 5, -1, 200, '测试下单', 2, msg, C)
```

#### 调整至目标持仓

本示例由于演示如何调仓。

python

```
#encoding:gbk

'''
调仓到指定篮子
'''

import pandas as pd
import numpy as np
import time
from datetime import timedelta,datetime

#自定义类 用来保存状态 
class a():pass
A=a()
A.waiting_dict =  {}
A.all_order_ref_dict = {}
#撤单间隔 单位秒 超过间隔未成交的委托撤回重报
A.withdraw_secs = 30
#定义策略开始结束时间 在两者间时进行下单判断 其他时间跳过
A.start_time = '093000'
A.end_time = '150000'

def init(C):
    '''读取目标仓位 字典格式 品种代码:持仓股数, 可以读本地文件/数据库，当前在代码里写死'''
    A.final_dict = {"600000.SH" :10000, '000001.SZ' : 20000}
    '''设置交易账号 acount accountType是界面上选的账号 账号类型'''
    A.acct = account
    A.acct_type = accountType
    #定时器 定时触发指定函数
    C.run_time("f","1nSecond","2019-10-14 13:20:00","SH")


def f(C):
    '''定义定时触发的函数 入参是ContextInfo对象'''
    #记录本次调用时间戳
    t0 = time.time()
    final_dict=A.final_dict
    #本次运行时间字符串
    now = datetime.now()
    now_timestr = now.strftime("%H%M%S")
    #跳过非交易时间
    if now_timestr < A.start_time or now_timestr > A.end_time:
        return
    #获取账号信息
    acct = get_trade_detail_data(A.acct, A.acct_type, 'account')
    if len(acct) == 0:
        print(A.acct, '账号未登录 停止委托')
        return
    acct = acct[0]
    #获取可用资金
    available_cash = acct.m_dAvailable
    print(now, '可用资金', available_cash)
    #获取持仓信息
    position_list = get_trade_detail_data(A.acct, A.acct_type, 'position')
    #持仓数据 组合为字典
    position_dict = {i.m_strInstrumentID + '.' + i.m_strExchangeID : int(i.m_nVolume) for i in position_list}
    position_dict_available = {i.m_strInstrumentID + '.' + i.m_strExchangeID : int(i.m_nCanUseVolume) for i in position_list}
    #未持有的品种填充持股数0
    not_in_position_stock_dict = {i : 0 for i in final_dict if i not in position_dict}
    position_dict.update(not_in_position_stock_dict)
    #print(position_dict)
    stock_list = list(position_dict.keys())
    # print(stock_list)
    #获取全推行情
    full_tick = C.get_full_tick(stock_list)
    #print('fulltick', full_tick)
    #更新持仓状态记录
    refresh_waiting_dict(C)
    #撤超时委托
    order_list = get_trade_detail_data(A.acct, 'stock', 'order')
    if '091500'<= now_timestr <= '093000':#指定的范围內不撤单
        pass
    else:
        for order in order_list:
            #非本策略 本次运行记录的委托 不撤
            if order.m_strRemark not in A.all_order_ref_dict:
                continue
            #委托后 时间不到撤单等待时间的 不撤
            if time.time() - A.all_order_ref_dict[order.m_strRemark] < A.withdraw_secs:
                continue
            #对所有可撤状态的委托 撤单
            if order.m_nOrderStatus in [48,49,50,51,52,55,86,255]:
                print(f"超时撤单 停止等待 {order.m_strRemark}")
                cancel(order.m_strOrderSysID,A.acct,'stock',C)
    #下单判断
    for stock in position_dict:
        #有未查到的委托的品种 跳过下单 防止超单
        if stock in A.waiting_dict:
            print(f"{stock} 未查到或存在未撤回委托 {A.waiting_dict[stock]} 暂停后续报单")
            continue
        if stock in position_dict.keys():
            #print(position_dict[stock],target_vol,'1111')
            #到达目标数量的品种 停止委托
            target_vol = final_dict[stock] if stock in final_dict else 0
            if int(abs(position_dict[stock] - target_vol)) == 0:
                print(stock, C.get_stock_name(stock), '与目标一致')
                continue
            #与目标数量差值小于100股的品种 停止委托
            if abs(position_dict[stock] - target_vol) < 100:
                print(f"{stock} {C.get_stock_name(stock)} 目标持仓{target_vol} 当前持仓{position_dict[stock]} 差额小于100 停止委托")
                continue
            #持仓大于目标持仓 卖出
            if position_dict[stock]>target_vol:
                vol = int((position_dict[stock] - target_vol)/100)*100
                if stock not in position_dict_available:
                    continue
                vol = min(vol, position_dict_available[stock])
                #获取买一价
                print(stock,'应该卖出')
                buy_one_price = full_tick[stock]['bidPrice'][0]
                #买一价无效时 跳过委托
                if not buy_one_price > 0:
                    print(f"{stock} {C.get_stock_name(stock)} 取到的价格{buy_one_price}无效，跳过此次推送")
                    continue
                print(f"{stock} {C.get_stock_name(stock)} 目标股数{target_vol} 当前股数{position_dict[stock]}")
                msg = f"{now.strftime('%Y%m%d%H%M%S')}_{stock}_sell_{vol}股"
                print(msg)
                #对手价卖出
                passorder(24,1101,A.acct,stock,14,-1,vol,'调仓策略',2,msg,C)
                A.waiting_dict[stock] = msg
                A.all_order_ref_dict[msg] = time.time()
            #持仓小于目标持仓 买入
            if position_dict[stock]<target_vol:
                vol = int((target_vol-position_dict[stock])/100)*100
                #获取卖一价
                sell_one_price = full_tick[stock]['askPrice'][0]
                #卖一价无效时 跳过委托
                if not sell_one_price > 0:
                    print(f"{stock} {C.get_stock_name(stock)} 取到的价格{sell_one_price}无效，跳过此次推送")
                    continue
                target_value = sell_one_price * vol
                if target_value > available_cash:
                    print(f"{stock} 目标市值{target_value} 大于 可用资金{available_cash} 跳过委托")
                    continue
                print(f"{stock} {C.get_stock_name(stock)} 目标股数{target_vol} 当前股数{position_dict[stock]}")
                msg = f"{now.strftime('%Y%m%d%H%M%S')}_{stock}_buy_{vol}股"
                print(msg)
                #对手价买入
                passorder(23,1101,A.acct,stock,14,-1,vol,'调仓策略',2,msg,C)
                A.waiting_dict[stock] = msg
                A.all_order_ref_dict[msg] = time.time()
                available_cash -= target_value
    #打印函数运行耗时 定时器间隔应大于该值
    print(f"下单判断函数运行完成 耗时{time.time() - t0}秒")


def refresh_waiting_dict(C):
    """更新委托状态 入参为ContextInfo对象"""
    #获取委托信息
    order_list = get_trade_detail_data(A.acct,A.acct_type,'order')
    #取出委托对象的 投资备注 : 委托状态
    ref_dict = {i.m_strRemark : int(i.m_nOrderStatus) for i in order_list}
    del_list = []
    for stock in A.waiting_dict:
        if A.waiting_dict[stock] in ref_dict and ref_dict[A.waiting_dict[stock]] in [56, 53, 54]:
            #查到对应投资备注 且状态为成交 / 已撤 / 部撤， 从等待字典中删除
            print(f'查到投资备注 {A.waiting_dict[stock]}，的委托 状态{ref_dict[A.waiting_dict[stock]]} (56已成 53部撤 54已撤)从等待等待字典中删除')
            del_list.append(stock)
        if A.waiting_dict[stock] in ref_dict and ref_dict[A.waiting_dict[stock]] == 57:
            #委托状态是废单的 也停止等待 从等待字典中删除 
            print(f"投资备注为{A.waiting_dict[stock]}的委托状态为废单 停止等待")
            del_list.append(stock)
    for stock in del_list:
        del A.waiting_dict[stock]
```

#### 获取融资融券账户可融资买入标的

python

```
#coding:gbk
def init(C):

    r = get_assure_contract('123456789')
    if len(r) == 0:
        print('未取到担保明细')
    else:
        finable = [o.m_strInstrumentID+'.'+o.m_strExchangeID for o in r if o.m_eFinStatus==48]
        print('可融资买入标的:', finable)
```

#### 获取两融账号信息示例

python

```
#coding:gbk

def init(C):
    account_str = '11800028'
    credit_account = query_credit_account(account_str, 1234, C)

def credit_account_callback(C,seq,result):
    print('可买担保品资金', result.m_dAssureEnbuyBalance)
```

#### 直接还款示例

该示例演示使用python进行融资融券账户的还款操作。

python

```
#coding:gbk  


def init(ContextInfo):
    # 用passorder函数进行融资融券账号的直接还款操作

    money = 10000  #还款金额
    #account='123456'
    s = '000001.SZ'  # 代码填任意股票，占位用
    passorder(32, 1101, account, s, 5, 0, money, 2, ContextInfo)
    # passorder(75, 1101, account, s, 5, 0, money, 2, ContextInfo) 专项直接还款
```

### 交易数据查询示例

python

```
#coding:gbk


def to_dict(obj):
    attr_dict = {}
    for attr in dir(obj):
        try:
            if attr[:2] == 'm_':
                attr_dict[attr] = getattr(obj, attr)
        except:
            pass
    return attr_dict


def init(C):
    pass
    #orders, deals, positions, accounts = query_info(C)


def handlebar(C):
    if not C.is_last_bar():
        return
    orders, deals, positions, accounts = query_info(C)


def query_info(C):
    orders = get_trade_detail_data('8000000213', 'stock', 'order')
    for o in orders:
        print(f'股票代码: {o.m_strInstrumentID}, 市场类型: {o.m_strExchangeID}, 证券名称: {o.m_strInstrumentName}, 买卖方向: {o.m_nOffsetFlag}',
        f'委托数量: {o.m_nVolumeTotalOriginal}, 成交均价: {o.m_dTradedPrice}, 成交数量: {o.m_nVolumeTraded}, 成交金额:{o.m_dTradeAmount}')


    deals = get_trade_detail_data('8000000213', 'stock', 'deal')
    for dt in deals:
        print(f'股票代码: {dt.m_strInstrumentID}, 市场类型: {dt.m_strExchangeID}, 证券名称: {dt.m_strInstrumentName}, 买卖方向: {dt.m_nOffsetFlag}', 
        f'成交价格: {dt.m_dPrice}, 成交数量: {dt.m_nVolume}, 成交金额: {dt.m_dTradeAmount}')

    positions = get_trade_detail_data('8000000213', 'stock', 'position')
    for dt in positions:
        print(f'股票代码: {dt.m_strInstrumentID}, 市场类型: {dt.m_strExchangeID}, 证券名称: {dt.m_strInstrumentName}, 持仓量: {dt.m_nVolume}, 可用数量: {dt.m_nCanUseVolume}',
        f'成本价: {dt.m_dOpenPrice:.2f}, 市值: {dt.m_dInstrumentValue:.2f}, 持仓成本: {dt.m_dPositionCost:.2f}, 盈亏: {dt.m_dPositionProfit:.2f}')


    accounts = get_trade_detail_data('8000000213', 'stock', 'account')
    for dt in accounts:
        print(f'总资产: {dt.m_dBalance:.2f}, 净资产: {dt.m_dAssureAsset:.2f}, 总市值: {dt.m_dInstrumentValue:.2f}', 
        f'总负债: {dt.m_dTotalDebit:.2f}, 可用金额: {dt.m_dAvailable:.2f}, 盈亏: {dt.m_dPositionProfit:.2f}')

    return orders, deals, positions, accountsPython环境相关
安装第三方 Python 库报错
问题描述：

"ImportError:Forbidden:Moduleopenpyxl not in whitelist!"

问题解答：

该报错是由于券商后台开启了 Python 库白名单，若您使用的是券商提供的QMT终端，请联系您的所属券商开通对应 Python 库白名单权限即可。

启动策略时pandas库报错
报错信息1 ：NameError: name 'pandas' is not defined

解答：

该报错是指当前环境下没有找到pandas库

解决方法

请在设置-模型设置中检查正确设置了路径,正确路径应指向{安装目录}\bin.x64
正确路径参考

请检查是否已经下载了python环境
下载python环境

报错信息2 ：AttributeError: module 'pandas' has no attribute 'core'

解答：

该报错是由于在pandas导入中被强行中断导致的

解决方法

重启客户端

对第三方库的支持
QMT Python API 提供基于 Python 3.6 规范的标准量化投资策略应用程序接口，本文档示例代码基于 Python 3.6 规范。我司主要通过以下两种方式对外提供：

系统自带的 Python 环境
QMT 系统的安装包默认自带 Python 运行环境。用户安装完迅投客户端后，默认可以直接使用Python。在这个打包的Python环境中，迅投除了提供标准的 Python api 带的库外，还集成了如下一些第三方库：

名称	说明
NumPy	NumPy (Numeric Python) 提供了许多高级的数值编程工具，如：矩阵数据类型、矢量处理，以及精密的运算库。专为进行严格的数字处理而产生。
Pandas	Python Data Analysis Library 或 Pandas 是基于 NumPy 的一种工具，该工具是为了解决数据分析任务而创建的。Pandas 纳入了大量库和一些标准的数据模型，提供了高效地操作大型数据集所需的工具。Pandas 提供了大量能使我们快速便捷地处理数据的函数和方法。
Patsy	一个线性模型分析和构建工具库。
SciPy	SciPy 函数库在 NumPy 库的基础上增加了众多的数学、科学以及工程计算中常用的库函数。例如线性代数、常微分方程数值求解、信号处理、图像处理、稀疏矩阵等等。
Statsmodels	Python 的统计建模和计量经济学工具包，包括一些描述统计、统计模型估计和推断。
TA_Lib	称作技术分析库，是一种广泛用在程序化交易中进行金融市场数据的技术分析的函数库。它提供了多种技术分析的函数，可以大大方便我们量化投资中编程工作，内容包括：多种指标，如 ADX, MACD, RSI, 布林轨道等；K 线形态识别，如黄昏之星，锤形线等等。
第三方库导入指引
除迅投提供的标准 Python api 和集成的部分第三方库，用户也可自己在 Python 官网下载其他所需第三方库，使用方式如下：

（1）本地安装Python环境，下载python3.6，Python官网：https://www.python.org/downloads/release/python-360/

（2）安装位置：C:\Python36

​ 新增环境变量：我的电脑--属性--高级系统设置--高级--环境变量---path：C:\Python36;C:\Python36\Scripts

​img

（3）Python环境检查

​ Win+R 打开运行,输入 cmd

img

​ 检查Python变量

image-20210318150744951

（4）安装第三方库

​ 安装前先确认客户端安装目录，根据个人电脑进行调整。

​ 安装时若遇到下面错误提示，请执行 pip 更新命令 python -m pip install --upgrade pip

image-20210318151342044

​ 安装三方库命令 pip install openpyxl -t E:\QMT交易端20962\bin.x64\Lib\site-packages

image-20210318151401916

（5）检查安装结果

​ 安装位置\bin.x64\Lib\site-packages检查安装库

image-20210318151523278

业务规则相关
交易所委托数量规则
科创板，连续交易时段限价单笔最大是10万股，市价单笔最大是5万股，盘后定价交易单笔最大量是100万股，200股起，1股递增。
创业板，连续交易时段限价单笔最大30万股，市价单笔最大15万股，100股起，100股递增。
主板，6和0开头的，连续交易时段单笔最大100万股，100股起，100股递增。
策略运行相关
在策略没有勾选终端启动后自动运行的情况下，策略自动启动运行
情况一

策略被运行于行情界面的副图上，随客户端启动被启动

解决方法

在右上角的页面布局中选择恢复默认布局，并重启客户端恢复默认布局

情况二

交易日切换/行情断线重连时，所有挂着的模型会被重新运行，这是正常的

策略回测相关
QMT在回测时如何选择复权方式
解答

回测是为了更贴近历史数据，但实际中各类配股、增发的动作，会造成价格的异常波动，为了避免这样的波动对回测的影响，我们推荐用户在回测中使用等比前复权价，这样在回测过程中，无需考虑配股、增发带来的变化，始终以统一标准的价格进行买卖，方便的同时也能得到更贴合历史数据的回测收益和表现。

交易相关
系统对象 ContextInfo 逐 k 线保存的机制
机制说明

ContextInfo是由底层维护并传递给init、handlebar等系统函数的参数，同一个 bar（不是 bar 里面的 tick，下同）内ContextInfo本质上是同一个变量且对其进行的修改只会对本次handlebar调用的下文所起作用。handlebar里对ContextInfo做的修改在该 bar 结束后才会进行保存，也就是说，对ContextInfo做的修改会在下一个 bar 体现出来。

具体来说，ContextInfo不同于一般 python 对象，做了逐 k 线更新设计，盘中主图品种每个 Level 1 分笔到达会触发handlebar函数调用，但只有 k 线结束时最后一个分笔触发的handlebar调用，对ContextInfo的修改才有效。

每次handlebar函数调用前会对ContextInfo对象进行深拷贝, 下一次分笔行情到来时，如果新的分笔不是新 k 线 bar 第一个分笔，则判断上一个分笔不是k线最后分笔，ContextInfo对象被回退为之前深拷贝的那个。

ContextInfo对象逐k线更新机制设计的目的，是为了在盘中时模拟k线的效果，只在k线结束的分笔触发的handlebar函数运行时生效一次，丢弃所有其他分笔的修改。

影响

该机制有两个影响，一是在ContextInfo对象中存数据每次分笔到达时会被深拷贝，拖慢策略运行；二是ContextInfo适用于记录逐k线生效的交易信号（quickTrade参数传0），不适宜立刻下单的情况。

如不需要模拟k线效果，希望调用交易函数后立刻下单，quickTrade参数可以传2， 下单记录可以用普通的全局变量保存, 不能存在ContextInfo对象的属性里(实现可以参考实盘示例7-调整至目标持仓Demo)。

快速交易参数 quickTrade
下单函数passorder有可选参数快速交易quickTrade， 默认为0。

传0，只在k线结束分笔时调用passorder产生有效信号，其他情况调用不产生信号。
传1，在当前k线为最新k线时调用passorder函数产生有效信号, 历史k线调用不产生信号。
传2，任何情况下调用passorder都产生有效信号，不会丢弃任何一次调用的信号。
如果在定时器注册的回调函数，行情回调函数, after_init函数中调用下单函数，需要传2，确保不会漏单。
passorder以外的下单函数不能指定快速交易参数，效果与传0的passorder一致。
下单与回报相关
为保证以尽快的速度执行交易信号, qmt 客户端提供的交易接口是异步的, 以快速交易参数填2的passorder函数为例，调用后会立刻发出委托, 然后返回。不会等待委托回报, 也不会阻塞python线程的运行。

委托/成交/持仓/账号信息的更新, 是在客户端后台进行的, python策略中无法手动控制。python提供的取账号信息接口 get_trade_detail_data， 与四种交易回调函数, 都是从客户端本地缓存中读取数据 / 触发调用，不是调用时查询柜台再返回。客户端本地缓存状态定期接收柜台推送刷新，有交易主推的柜台50ms一次，没有交易主推的柜台1-6秒一次。 不能认为get_trade_detail_data查到的状态是与柜台完全一致的, 比如卖出委托后立刻查询, 不会查到对应委托, 可用资金也不会变多。

实盘策略需要设计盘中保存/更新委托状态的机制。常见的做法是用全局变量字典保存委托状态, 给每一笔委托独立的投资备注作为字典的key，委托状态作为字典的value, 下单后默认设置为待报, 之后查到委托后更新状态。如果某品种股票存在待报状态委托, 暂停该品种后续报单, 防止发生超单的情况。(实现可以参考实盘示例7-调整至目标持仓Demo)

QMT 所有策略是在同一个线程中被调用的，任意一个策略阻塞线程(死循环 sleep 加锁等操作)会导致所有策略的执行被阻塞，所以不能在策略里写等待操作。如需要多线程 / 多进程的用法，可以使用极简模式配合 xtquant 库使用

QMT 下单失败
检查是否是在模型交易界面，实盘模式运行的策略。模拟模式只显示策略信号，不发出委托。

如运行到交易函数，未看到策略信号，检查交易函数是否使用了快速下单参数(quickTrade)，默认为0，只会在k线结束发出委托，日线及以上周期等于全天不会委托。传1时，非历史bar上执行时（ContextInfo.is_last_bar()为True），只要策略模型中调用到就触发下单交易。传2，无论是否是历史bar，运行到交易函数时立刻发出委托。

如果希望盘中出现信号立即下单，建议传1，这种情况下会有策略信号闪烁的风险，需要自己处理；如果希望K线结束下单（信号不闪烁），建议传0，通常情况下不建议传2

提示

具体到场景：

handlebar逐k线下单, 每次k线结束的分笔生效一次, 传0;
需要在handlebar盘中触发立刻下单, 传1;
定时器/init/after_init与交易回调函数, 行情回调函数内下单, 传2.
如看到实盘的策略信号，未找到对应委托，检查客户端左下角消息提示是否有报错，如有，请根据消息提示的描述修改下单参数
行情相关
QMT 行情数据基础概念
QMT行情数据主要分为三种，包括本地数据，全推数据，订阅数据。

本地数据： 指下载到本地的行情数据加密文件。包括历史数据，适合回测模式使用，对应python接口为get_market_data_ex(subscribe=False) 

全推数据： 指客户端启动后, 自动接收，更新的全市场最新数据快照， 包括日线的开高低收,成交量成交额，与五档盘口（在行情界面选择了五档行情时可用五档 具体见行情常规问题3）。支持取全市场品种, 只有最新值，没有历史值，服务器对交易所下发的数据即时转发，打包增量部分发送给下游客户端。可以用get_full_tick一次性取出当前最新值，也可以用subscribe_whole_quote注册回调函数，每次处理增量的部分。 对应python接口为get_full_tick，subscribe_whole_quote

订阅：指向行情服务器订阅指定品种行情, 共有四种周期(分笔 1分钟 5分钟 日线)，可以订阅当日数据，当天以前的需要用 down_history_data下. 订阅有最大数量限制(例如：假设最大数量限制为300个，则可以单独订阅日线300个，若同时订阅日线和五分钟 则各150个)，如需订阅超过定义上限，可以在页面右上角，选购行情vip服务。对应python接口为subscribe_quote和get_market_data_ex(subscribe=True,)其中，使用get_market_data或get_market_data_ex(subscribe=True,)时客户端会自动订阅传入的品种，不需要额外调用subscibe_quote,但这种方式订阅的品种没有订阅号，无法手动反订阅，只能通过停止策略释放可订阅数。

警告

如果超出订阅数量限制，则返回的行情数据会使用前值填充，出现重复值，非正确行情数据。

QMT 行情调用函数对比说明
down_history_data 下载指定区间的行情数据到本地，存放在硬盘上。效果和界面,点击行情数据下载一致。 开始时间不填时，为增量下载(以本地数据最后一天为开始时间), 填写的话按填写值下载。
get_local_data 取本地数据函数，盘中不会更新，速度快，回测可以用这个函数取。
get_full_tick 取客户端缓存中的最新全推数据。全推数据不包括历史，不用订阅，没有品种数量限制，盘中50ms更新一次，速度快。
subscribe_quote 向服务器订阅股票行情 盘中实时更新 初次订阅耗时长，最大订阅品种数受限. 订阅超过一定数量的品种k线行情不会更新.可订阅四种基本周期(分笔 一分钟 五分钟 日线)行情（如果有 Level-2 行情权限 也可以订 Level-2 的）, 同一品种订阅了不同周期累加计数(如订阅浦发银行 1分钟 5分钟 日线行情 算订阅3次). 复数策略订阅同一品种计数不会累加. Level-2 的订阅也会受限，但是和 Level 1 的互不影响。
unsubscribe_quote 按订阅号反订阅行情, 释放可订阅数.
get_market_data_ex 取订阅/本地数据接口。用subscribe_quote在init函数中先订阅后subscribe参数为True时，取本地数据和订阅的最新行情。subscribe参数传False时,可以用来取本地数据，不会订阅。 如股票池超过一定数量，可用 down_history_data + get_local_data + get_full_tick 拼接历史和最新数据替代get_market_data_ex。
注意

gmd系列函数在init中运行时，只能读取到本地数据，不会取到最新行情数据，因子，不建议在init中调用使用gmd系列函数

警告

不再推荐使用!

set_universe, get_history_data, get_market_data 是早期订阅股票池, 取订阅的行情数据接口. 因为set_universe订阅的品种没有订阅号 无法在策略中反订阅, 只能通过停止策略释放订阅数。

全推接口和订阅接口的分笔行情没有5档行情，只有最新价
问题描述

get_full_tick, subscribe_while_quote函数中获取分笔行情没有5档行情，只有最新价。

解决办法

修改行情源对应的全推行情级别，见下图

img

或者

img

行情中心和交易中心到底有啥区别？
行情中心控制单支订阅，例如subscribe_quote

交易中心影响全推数据，例如get_full_tick,subscribe_whole_quote

passorder使用对手价下单报错/有误
问题描述

passorder参数prType填写14(对手价下单)时，委托价格有误，或信息提示对手价无效，无法下单!

解决办法

修改行情源对应的全推行情级别，见下图

img

或者

img

为什么在handlebar中获取期货tick时，tick是3S一个而非0.5s一个？
这是由于handlebar函数是逐K线驱动的，在实时行情中，handlebar会随着主图标的tick的更新被调用。

在这个问题场景中，主图的标的通常被设置为股票，而股票的tick通常是3s一个，这就导致handlebar函数3s才被调用一次

解决方法

使用定时器(run_time)进行计算

使用订阅推送(subscribe),在回调函数中进行计算

如果需要在handlebar中进行期货策略编写，建议将主图设置为期货品种，来保证handlebar调用频率

为什么在非交易时间段handlebar也会被调用
handlebar受行情数据推送驱动，在非交易时段，行情服务会做一系列准备工作，其中可能伴随着服务重启，在重启后为了保证数据齐全，客户端会重新订阅数据，这时服务会推送最新的数据，客户端会把推送的最新数据更新至缓存，并向上层策略推送更新，也就是触发handlebar执行

这个合并数据的驱动执行只会在最新一根bar而不会在历史范围，策略可以根据需要处理这次推送或直接根据交易时间跳过这个驱动，例如判断time小于09:15则直接return

关于证券状态openint值的详细说明
沪市

时间段	状态	编码
9:15 - 9:25	盘前集合竞价	12
9:25 - 14:57	盘中连续竞价	13
14:57 - 15:00	盘后集合竞价	18
15:00	收盘状态	15
15:05 - 15:30	盘后定价	22
15:30	盘后定价结束	23
上一状态后	收盘状态	15
深市

时间段	状态	编码
9:15 - 9:25	盘前集合竞价	12
9:25 - 9:30	休市	14
9:30 - 11:30	盘中连续竞价	13
11:30 - 13:00	休市	14
13:00 - 14:57	盘中连续竞价	13
14:57 - 15:00	盘后集合竞价	18
15:00	发收盘状态	15
15:05 - 15:30	盘后定价	22
15:30	盘后定价结束	23
上一状态后	收盘状态	15
静态数据问题
报错：[系统]ERROR：******.**获取合约乘数和最小变动价位失败，跳过
点击右下角【行情】按钮，选择【智能下载】，数据选项下拉框勾选【过期合约列表】点击该面板右下角【开始】，待过期合约数据补充完毕后，即可正常获取过期合约数据。

软件运行日志相关
如何找到软件运行日志
Log文件通常在安装目录下的.\userdata\log文件夹中，在 .\userdata\log 文件夹中，你可能会看到一个或者多个 log 文件，通常以 '.log' 作为扩展名。这些文件将包含软件运行时的详细情况。

投研：{安装目录}\userdata\log

说明

XtClient_20210922.log - 客户端常规日志

XtClient_datasource_20210922.log - 行情数据日志

XtClient_Formula_20210922.log - 策略运行日志

XtClient_FormulaOutput.log - 策略输出日志

QMT：{安装目录}\userdata\log

说明

XtClient_20210922.log - 客户端常规日志

XtClient_Formula_20210922.log - 策略运行日志

XtClient_FormulaOutput.log - 策略输出日志

XtClient_PerformanceFile_20210922.log - 客户端流程节点日志

极简模式：{安装目录}\userdata_mini\log

说明

XtMiniQuote_20210917.log - 行情策略模块日志

XtMiniQmt_20210917.log - 客户端常规日志

XtMiniQmt_perform_20210917.log - 客户端流程节点日志

上次更新次
```
