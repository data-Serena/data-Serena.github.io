# 网约车平台司机留存率提升分析

 ⚠️ 本项目仅展示分析思路，所有数据均为虚拟数据，用于演示数据分析流程与方法。

## 📌 项目背景

随着运力竞争的加剧，司机留存率已成为网约车平台核心关注指标之一。本项目基于模拟数据，构建司机留存分析全流程，识别关键影响因素并提供可执行建议。

## 📊 数据说明

本项目基于以下4张数据表：

- **driver_info.csv**: 司机注册信息（注册城市、注册时间、车辆类型等）  
- **driver_activity.csv**: 每日活跃行为（在线时长、完单数、收入）  
- **driver_orders.csv**: 每笔订单详细数据（时间、评分、取消情况）  
- **driver_churn_status.csv**: 系统标记是否流失

## 🎯 分析目标

1. 整合多源数据形成司机全景行为画像  
2. 根据平台定义识别流失司机群体  
3. 通过留存曲线和同期群对比分析关键影响因素  
4. 提出精准运营策略

## 🧩 分析过程与结果

### 1. 数据清洗与整合
```python
# 合并基本信息、行为数据与标签
import pandas as pd
info = pd.read_csv('driver_info.csv')
activity = pd.read_csv('driver_activity.csv')
churn = pd.read_csv('driver_churn_status.csv')

# 汇总日均收入、在线时长等
agg_df = activity.groupby('driver_id').agg({
    'daily_income': 'mean',
    'online_hours': 'mean',
    'order_count': 'sum',
    'completed_order_count': 'sum'
}).reset_index()

agg_df['completion_rate'] = agg_df['completed_order_count'] / agg_df['order_count']

# 合并
driver_df = info.merge(agg_df, on='driver_id').merge(churn, on='driver_id')
```

### 2. 流失司机定义
```python
last_active = driver_activity.groupby('driver_id')['date'].max().reset_index()
last_active['is_churned'] = last_active['date'] < (driver_activity['date'].max() - pd.Timedelta(days=30))
driver_df = driver_df.merge(last_active[['driver_id', 'is_churned']], on='driver_id', how='left')
```

### 3. 探索性数据分析
```python
agg = driver_activity.groupby('driver_id').agg({
    'online_hours': 'mean',
    'daily_income': 'mean',
    'order_count': 'mean'
}).reset_index()
agg = agg.merge(last_active[['driver_id', 'is_churned']], on='driver_id')
```

### 4. 同期群分析
```python
driver_df['reg_month'] = driver_df['registration_date'].dt.to_period('M')
driver_df['active_month'] = driver_df['date'].dt.to_period('M')
cohort = driver_df.groupby(['reg_month', 'active_month'])['driver_id'].nunique().reset_index()
cohort_pivot = cohort.pivot(index='reg_month', columns='active_month', values='driver_id')
cohort_size = cohort_pivot.iloc[:, 0]
retention = cohort_pivot.divide(cohort_size, axis=0)
```

### 5. 关键流失因素分析
```python
driver_df['order_completion_rate'] = driver_df['completed_order_count'] / driver_df['order_count'].replace(0, 1)
driver_df['online_hours_bin'] = pd.cut(driver_df['online_hours'], bins=[0,2,4,6,8,12,24])
print(driver_df.groupby('is_churned_y')[['daily_income', 'online_hours', 'order_completion_rate']].mean())
```

## 🧠 策略建议

- 新司机保护机制： 提供前30天“保底收入”
- 智能派单优化： 减少司机长时间空等现象
- 精准激励机制： 针对潜在流失司机进行分层触达

## 💼 项目贡献

- 构建了平台侧司机生命周期分析模板
- 明确关键影响因素并输出可落地建议
- 提供平台级司机分层运营策略参考

## 🔁 分析思路迁移场景

本项目采用的数据整合 + 流失识别 + 同期群分析 + 流失因素挖掘 + 策略建议的方法论，可迁移至以“用户/个体留存”为核心目标的行业场景：

1. **SaaS企业的客户续约分析**
    - 定义续约 vs 流失用户群体，基于使用频率、功能偏好等指标分析。
    - 同期注册客户的留存对比，识别关键使用行为与付费意愿。
2. **在线教育平台的学员留存分析**
    - 分析课程学习频率、完课率、作业提交情况与流失关系。
    - 评估教学内容、班型、学习激励机制对留存的影响。
3. **健身房会员的到店活跃分析**
    - 根据会员注册时间、课程预约、打卡频率等数据构建同期群。
    - 挖掘促销活动、教练互动、设施利用率对会员活跃的作用。

## 🛠 技术栈

- 数据分析：Pandas、NumPy
- 可视化：Matplotlib、Seaborn
- 开发工具：Jupyter Notebook / VS Code

---

👉 [查看完整代码](../assets/driver-retention-analysis.ipynb)

