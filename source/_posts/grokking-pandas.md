---
title: Pandas实用教程
date: 2020-03-20 16:42:30
tags: 笔记
---

最近遇到了些数据处理问题，试着用pandas进行处理，效果还不错。
下面简单做下记录，很具实用性。

<!-- more -->

# 开始之前
```
# 安装 jupyter & pandas
# 或者直接安装 anaconda —— python发行版
$ pip3 install jupyter pandas 

# 打开 jupyter-console
$ jupyter-console
import pandas as pd
```


[TODO]  未完成
```python
# 加载
df = pd.read_csv('./something.csv')
# 改列名
df.columns = ['sp_id', 'sms_type']
# 提取列
df2 = df[['mobile', 'content']]
# 过滤
df2 = df2[df2['sms_type'] == 'voice'] # 这里python运算符重载。仅使用的话，不用了解太多细节吧
# 分组统计
spid_count_series = df2[['sp_id', 'count']].groupby('sp_id').sum()
# Series转DataFrame
spid_count_df = spid_count_series.reset_index()
# 保存xls, 不带编号
spid_count_df.to_excel('./output.xls', index=None)
```
