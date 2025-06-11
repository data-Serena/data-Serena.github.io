# 网约车平台完单量提升分析

⚠️ 本项目仅展示分析思路，所有数据均为虚拟数据，用于演示数据分析流程与方法。

## 📌 项目背景

随着网约车市场的竞争加剧，提高平台的司机完单量是优化运力效率、提升用户满意度和平台营收的核心目标。本项目通过数据分析方法，从多个维度拆解和评估影响完单量的关键因素，并提出优化建议。

## 📊 数据说明

本项目基于以下4张数据表：

- **orders.csv**: 订单信息（含时间、里程、金额等）
- **driver\_activity.csv**: 司机活跃度数据（按时段统计）
- **user\_requests.csv**: 用户叫车请求数据（含是否成功匹配）
- **weather\_holidays.csv**: 外部天气与节假日信息（可选）

## 🎯 分析目标

1. **指标拆解：** 将“完单量”拆解为供给、需求、匹配效率等多个维度下的细分指标。
2. **多维分析：** 从时间、空间、用户群体、司机群体等多个角度进行交叉分析。
3. **因果推断（初步）：** 通过对比分析、趋势分析、相关性分析等方法，初步推断影响因素。
4. **策略评估思维：** 运用前后对比、区域对比等方式，近似模拟A/B测试效果评估。
5. **数据可视化：** 使用折线图、柱状图、分布图等清晰展示结果。

## 🧩 分析过程与结果

### 1. 指标拆解

```python
orders['is_completed'] = orders['status'] == 'completed'
daily_summary = orders.groupby('date').agg({
    'order_id': 'count',
    'is_completed': 'sum'
}).rename(columns={'order_id': '总订单量', 'is_completed': '完单量'})

# 匹配效率 = 完单量 / 请求量
daily_requests = user_requests.groupby('date').size().rename('请求量')
daily_summary = daily_summary.join(daily_requests)
daily_summary['匹配效率'] = daily_summary['完单量'] / daily_summary['请求量']
```

### 2. 多维分析

#### ⏰ 时间维度

```python
orders['hour'] = pd.to_datetime(orders['request_time']).dt.hour
hourly_orders = orders.groupby('hour')['is_completed'].mean()
hourly_orders.plot(title='小时维度完单率')
```

#### 📍 空间维度

```python
area_completion = orders.groupby('origin_area')['is_completed'].mean()
area_completion.plot(kind='bar', title='区域完单率')
```

#### 👥 用户&司机群体特征

用户订单频次分布、司机完单率分布图使用 `sns.histplot()` 展示。

### 3. 初步因果推断

```python
orders['date'] = pd.to_datetime(orders['request_time']).dt.date
orders['is_holiday'] = orders['date'].isin(weather[weather['is_holiday'] == 1]['date'])
holiday_completion = orders.groupby('is_holiday')['is_completed'].mean()
holiday_completion.plot(kind='bar', title='节假日 vs 非节假日完单率')
```

### 4. 策略效果评估（A/B测试模拟）

```python
orders['city'] = orders['origin_area'].str.extract(r'(\D+)')
pre = orders[(orders['city'] == '城市A') & (orders['date'] < pd.to_datetime('2024-01-01'))]
post = orders[(orders['city'] == '城市A') & (orders['date'] >= pd.to_datetime('2024-01-01'))]
print("策略前完单率：", pre['is_completed'].mean())
print("策略后完单率：", post['is_completed'].mean())
```

## 🧠 策略建议

- 在高峰时段和低完单率区域加强运力调度，提升匹配效率
- 针对节假日或恶劣天气完善动态定价机制
- 对低频用户进行激励（如优惠券）提升使用率
- 强化司机激励机制，优化区域派单和接单体验

## 💼 项目贡献

- 梳理完单量构成与影响因素，支持平台精细化运营
- 提供策略模拟方法论与可视化评估方式
- 输出提升完单量的多维策略建议

## 🔁 分析思路迁移场景

本项目所采用的“指标拆解 + 多维分析 + 因果推断 + 策略模拟”的分析框架，可迁移至其他场景：

1. **外卖平台的配送单量提升分析**
    - 拆解配送成功单量为：用户下单 → 骑手接单 → 骑手配送成功。
    - 分析维度：下单时段、天气、商圈位置、骑手活跃度等。
    - 对应策略：增加高峰期骑手激励等。
2. **电商平台的下单转化分析**
    - 将订单转化拆解为：访问 → 加购 → 下单 → 成交。
    - 分析用户群体行为特征与转化漏斗表现。
    - 对应策略：提升关键页面加载速度、设置优惠引导策略等。
3. **客服中心的工单处理效率分析**
    - 拆解流程为：用户提交 → 客服接单 → 问题解决。
    - 多维分析客服班组、时间段、问题类型等。
    - 对应策略：优化排班、智能路由、处理流程优化。

## 🛠 技术栈

- 数据分析：Pandas、NumPy
- 可视化：Matplotlib、Seaborn
- 开发工具：Jupyter Notebook / VS Code

---

👉 [查看完整代码](../assets/driver-order-analysis.ipynb)

