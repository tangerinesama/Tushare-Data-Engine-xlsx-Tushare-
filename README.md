# Tushare-Data-Engine-xlsx-Tushare-
An automated Tushare data collection system for A-Share quantitative trading. Features breakpoint resume, incremental updates, and smart chunking for massive datasets to prevent Excel crashes, building a clean local database with one click.高效的 A 股 Tushare 自动化数据采集系统。支持网络中断秒级续传、增量更新，以及防崩溃的百万行大表“智能分块”技术，一键为您构建干净、结构化的本地量化数据库。
这份手册将协助您快速掌握 Tushare 自动化数据采集系统 的操作逻辑与字段定义。本系统专为 A 股量化策略数据准备而设计，确保产出的 Excel 长表符合专业回测标准。
专业建议：在进行大规模同步前，请务必在 config.ipynb 中确认 OUT_DIR 的磁盘空间充足。系统会在 log 文件夹下生成与文件同名的校验日志，请在同步完成后核对行数一致性

一、	核心配置修改指南

1.所有运行参数均在 config.ipynb 中统一管理。如果您需要调整采集范围或授权信息，请直接修改该文件中的以下变量：

修改 Token
变量名称：TUSHARE_TOKEN
所在位置：config.ipynb - Cell 1
示例/建议：填入您从 Tushare 官网获取的 56 位授权码

修改起始时间
变量名称：START_DATE
所在位置：config.ipynb - Cell 3
示例/建议：START_DATE = '20240101' (YYYYMMDD 格式)

调整抓取频率
修改起始时间：SLEEP_TIME
所在位置：config.ipynb - Cell 3
示例/建议：5000 积分建议 0.12；低积分用户请调大至 0.3 以上

2. 自定义选择拉取表格 (位于 tushare.ipynb)
3. 
针对您「只想拉取特定表格」或「修复单一报错接口」的需求，系统已全面升级主控调度功能。请打开 tushare.ipynb，滑至最下方的 Cell 5: 主控调度，找到 TARGET_TABLES 列表：

	启用表格：将表格名称前面的注解符号删除，使其生效（
	停用/跳过表格：在不需要拉取的表格前方加上 # 注解掉（例如：#21_index_weight）。
	一键全选：取消注解 TARGET_TABLES = list(API_CONFIG.keys()) 即可无视列表，拉取所有设定好的表。
	极速修复优势：当某天资料缺失（如网路超时发生 Read timed out），您只需注解掉其他所有表，单独保留报错的表重新运行，系统将利用本地快取在几秒钟内为您精准补齐缺失的一天，绝对无需从头全量拉取。

二、	系统核心功能
	全量自动同步：一键拉取 A 股基础列表、交易日历、日线行情及核心财务数据。
	增量更新逻辑：系统自动扫描 data_output 文件夹，判定哪些季度或日期已存在，仅对缺失数据进行补齐，避免重复下载。
	季度自动分拆：针对海量日线行情，系统自动按季度拆分为多个 Excel（如03_daily_2024_quarter1.xlsx），彻底解决 Excel 百万行上限导致的文件崩溃问题。
	失败续跑保护：内置 data_cache 机制。若抓取中断，重启后会自动跳过已存在的单日 CSV 碎片，实现无缝续爬。

三、	文件夹与数据分级架构
1. 目录结构

文件夹名称：data_raw
功能描述：存放基础索引表
专业备注：原始接口数据，一次拉取后尽量不手改。

文件夹名称：data_cache
功能描述：存放每日 CSV 碎片	按日期或接口拆分的缓存文件，失败后可续跑。
专业备注：核心缓存区，支持“断点续传”逻辑

文件夹名称：data_output
功能描述：存放标准化成品策略使用的Excel
专业备注：最终生成的季度拆分行情长表，策略回测的唯一数据源

文件夹名称：log
功能描述：运行日志区
专业备注：记录同步量级、耗时及接口报错信息

3. 数据分级 (P0 / P1 / P2)

P0 (最小数据集)
包含内容：股票列表、日线行情、每日指标
字段策略：侧重价格、涨跌幅、估值与市值，回测必选

P1 (核心增强)
包含内容：财务利润表、业绩预告、停复牌
字段策略：侧重财报质量与公告事件，规避未来函数

P2 (后续增强)
包含内容：资金流向、两融数据、港股通持股
字段策略：侧重筹码分布与北向资金动向

四、 接口字段定义附录 (01-22 号)
四、 接口字段定义附录 (01-22 号)   

🟢 P0: 基础行情与估值数据集   

01_stock_basic (股票列表)   
包含内容：全 A 股票池   
建议字段：ts_code, symbol, name, area, industry, market, exchange, list_status, list_date, delist_date, is_hs   
字段策略：用于确定选股范围、计算上市天数、剔除已退市或暂停上市标的 。 

02_trade_cal (交易日历)   
包含内容：主要交易所的交易日历   
建议字段：exchange, cal_date, is_open, pretrade_date   
字段策略：用于确定调仓频率、对齐不同数据集的回测日期 。  

03_daily / 05_index_daily (日线行情)   
包含内容：个股及核心指数的日线价格数据   
建议字段：ts_code, trade_date, open, high, low, close, pre_close, change, pct_chg, vol, amount   
字段策略：计算动量因子、收益率、历史波动率及流动性指标 。  

04_adj_factor (复权因子)   
包含内容：股票每日的复权调节因子   
建议字段：ts_code, trade_date, adj_factor   
字段策略：用于还原股票除权后的真实价格走势，计算前复权收益率以支持真实回测 。  

06_daily_basic (每日指标)   
包含内容：每日更新的估值、市值及换手率指标   
建议字段：ts_code, trade_date, close, turnover_rate, volume_ratio, pe, pe_ttm, pb, total_mv, circ_mv   
字段策略：实现戴维斯双击策略、估值过滤（低 PE/PB）及拥挤度监控 。  

🟡 P1: 财务报表与事件数据集   

07_income (利润表)   
包含内容：上市公司利润表核心科目   
建议字段：ts_code, ann_date, f_ann_date, end_date, total_revenue, n_income, basic_eps   
字段策略：分析盈利增速及利润确认情况，必须保留 ann_date（公告日期）以规避未来函数 。  

08_fina_indicator (财务指标)   
包含内容：经过计算的各类财务比率指标   
建议字段：ts_code, ann_date, end_date, roe, roe_dt, roa, grossprofit_margin, netprofit_margin, debt_to_assets   
字段策略：构建质量因子、评估公司杠杆水平、排除基本面劣质公司 。  

09_forecast (业绩预告) / 10_express (业绩快报)   
包含内容：财报正式披露前的盈利预估及快讯数据   
建议字段：ts_code, ann_date, type, p_change_min, p_change_max, yoy_net_profit   
字段策略：开发业绩预增策略、捕捉预期差、跟踪净利润断层信号 。  

11_disclosure_date (财报披露日)   
包含内容：上市公司定期报告的预约与实际披露时间   
建议字段：ts_code, end_date, pre_date, actual_date   
字段策略：严格确定信息可获取的真实时间，规避回测中的未来函数 。  

12_suspend_d (停复牌)   
包含内容：个股每日停复牌状态及原因   
建议字段：ts_code, trade_date, suspend_timing, suspend_type   
字段策略：在回测系统中进行可交易性过滤，剔除停牌期间的无效成交 。

13_stk_limit (涨跌停价格)   
包含内容：个股每日设定的涨停价与跌停价   
建议字段：ts_code, trade_date, up_limit, down_limit   
字段策略：监控盘中极端价格波动，执行涨跌停板无法成交的过滤逻辑 。  

14_sw_member (申万行业成分)   
包含内容：个股所属的申万行业分类明细   
建议字段：index_code, index_name, con_code, con_name, in_date   
字段策略：支持行业暴露度限制、开发行业轮动策略及行业中性化处理 。  

🔵 P2: 进阶筹码与宏观数据集   
15_moneyflow (个股资金流向)   
包含内容：区分规模的个股买卖单流向数据   
建议字段：ts_code, trade_date, buy_sm_amount, buy_md_amount, buy_lg_amount, buy_elg_amount   
字段策略：监控主力资金动态，分析小单、中单、大单及特大单的成交占比 。  

16_margin (融资融券)   
包含内容：每日融资融券交易余额及变动情况   
建议字段：trade_date, ts_code, rzye, rqye, rzmre   
字段策略：衡量市场杠杆资金情绪，分析融资余额的趋势变动 。  

17_hk_hold (沪深港通持股)   
包含内容：北向资金通过沪深港通持有的个股明细   
建议字段：ts_code, trade_date, vol, ratio   
字段策略：实现“聪明资金”跟随策略，追踪北向资金持股数量及占流通盘比例 。  

18_stk_holdernumber (股东人数)   
包含内容：定期披露的上市公司最新股东户数   
建议字段：ts_code, ann_date, end_date, holder_num   
字段策略：进行筹码集中度分析，通常股东人数骤减预示筹码集中 。  

19_top10_holders (前十大股东)   
包含内容：季度更新的前十大股东持股详细信息   
建议字段：ts_code, ann_date, holder_name, hold_amount, hold_ratio   
字段策略：分析机构抱团程度、跟踪大股东及证金/汇金等国家队的持股变动 。  

20_block_trade (大宗交易)   
包含内容：当日发生的单笔大宗交易明细   
建议字段：ts_code, trade_date, price, vol, amount, buyer, seller   
字段策略：分析大宗交易的折溢价水平，追踪特定的买卖方营业部动向 。  

21_index_weight (指数权重)   
包含内容：主流指数（如沪深300）成分股的权重分配   
建议字段：index_code, con_code, trade_date, weight   
字段策略：用于开发指数增强策略，或精确计算成分股对指数的贡献度 。  

22_macro (宏观利率)   
包含内容：上海银行间同业拆放利率（Shibor）等数据   
建议字段：date, shibor, shibor_quote, shibor_ma   
字段策略：进行宏观流动性择时，分析市场无风险利率环境对股市的影响 。  
