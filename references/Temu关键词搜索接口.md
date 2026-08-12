# Temu 关键词搜索接口

## 目录

- [脚本调用](#脚本调用)
- [基础参数](#基础参数)
- [站点与类目 ID](#站点与类目-id)
- [区间筛选参数](#区间筛选参数)
- [排序字段](#排序字段)
- [响应与失败处理](#成功响应)

## 脚本调用

单页数量最大为 200。

通过以下命令调用：

```bash
python3 scripts/temu_keyword_search.py \
  --param "keyword=summer dress" \
  --param "siteId=48" \
  --param "size=20" \
  --param "sort=totalSold" \
  --param "order=desc"
```

类目 ID 可重复传入：

```bash
python3 scripts/temu_keyword_search.py \
  --param "catIds=10001" \
  --param "catIds=20002" \
  --param "sort=monthSoldRate" \
  --param "order=desc"
```

服务端要求暂停查询时，按 [查询暂停与恢复流程](查询暂停与恢复流程.md) 提示用户；恢复条件满足后再次运行同一命令。

## 基础参数

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `keyword` | string | 无 | 同时匹配英文关键词和中文关键词，最长 300 个字符 |
| `catIds` | integer，可重复 | 无 | Temu 类目 ID，多个值匹配其中任一类目 |
| `siteId` | integer | `48` | 极鲸云站点 ID；美国站直接使用默认值，其他国家或地区先实时解析站点 |
| `page` | integer | `1` | 页码，从 1 开始 |
| `size` | integer | `20` | 每页数量，最大 200 |
| `sort` | string | 无 | 排序字段，取值见下方 |
| `order` | string | `desc` | `asc` 或 `desc` |

## 站点与类目 ID

- 未指定站点或指定美国站时，直接使用美国站 ID `48`，不额外查询站点列表。
- 用户指定其他国家或地区时，运行 `python3 scripts/temu_site_list.py --country "国家或地区名"`，只使用唯一精确匹配的站点 ID。
- 不使用本地静态站点表，不按语言、币种或相似名称猜测。无匹配或多个匹配时请用户确认。
- 只传入用户提供或当前上下文中已有可靠来源的类目 ID，不根据名称猜测。没有可信类目 ID 但已有关键词时，省略 `catIds` 并直接按关键词查询；必须严格限定类目时请用户提供类目 ID。

## 区间筛选参数

每项均支持 `Min` 和 `Max` 后缀：

| 维度 | 最小值 | 最大值 |
| --- | --- | --- |
| 蓝海指数 | `dsrMin` | `dsrMax` |
| 总销量 | `totalSoldMin` | `totalSoldMax` |
| 平均价格 | `avgPriceMin` | `avgPriceMax` |
| 总销售额 | `totalSalesMin` | `totalSalesMax` |
| 市场时间 | `firstOnSaleTimeMin` | `firstOnSaleTimeMax` |
| 总商品数 | `itemCountMin` | `itemCountMax` |
| 半托管商品数 | `semiManagedItemCountMin` | `semiManagedItemCountMax` |
| 总店铺数 | `mallCountMin` | `mallCountMax` |
| 半托管店铺数 | `semiManagedMallCountMin` | `semiManagedMallCountMax` |
| 日/周/月销量 | `daySoldMin`、`weekSoldMin`、`monthSoldMin` | `daySoldMax`、`weekSoldMax`、`monthSoldMax` |
| 日/周/月销量增长率 | `daySoldRateMin`、`weekSoldRateMin`、`monthSoldRateMin` | `daySoldRateMax`、`weekSoldRateMax`、`monthSoldRateMax` |
| 日/周/月销售额 | `daySalesMin`、`weekSalesMin`、`monthSalesMin` | `daySalesMax`、`weekSalesMax`、`monthSalesMax` |
| 日/周/月销售额增长率 | `daySalesRateMin`、`weekSalesRateMin`、`monthSalesRateMin` | `daySalesRateMax`、`weekSalesRateMax`、`monthSalesRateMax` |
| 日/周/月商品数变化量 | `dayItemCountMin`、`weekItemCountMin`、`monthItemCountMin` | `dayItemCountMax`、`weekItemCountMax`、`monthItemCountMax` |
| 日/周/月商品数增长率 | `dayItemCountRateMin`、`weekItemCountRateMin`、`monthItemCountRateMin` | `dayItemCountRateMax`、`weekItemCountRateMax`、`monthItemCountRateMax` |
| 日/周/月店铺数变化量 | `dayMallCountMin`、`weekMallCountMin`、`monthMallCountMin` | `dayMallCountMax`、`weekMallCountMax`、`monthMallCountMax` |
| 日/周/月店铺数增长率 | `dayMallCountRateMin`、`weekMallCountRateMin`、`monthMallCountRateMin` | `dayMallCountRateMax`、`weekMallCountRateMax`、`monthMallCountRateMax` |

市场时间使用 ISO 8601 日期时间，它代表该关键词下最早商品的上架时间，不是关键词创建时间。增长率的查询和响应均使用小数；例如用户要求增长率至少 20%，请求值使用 `0.2`。商品数和店铺数的周期值是变化量，可能为负数。平均价格和所有销售额使用当前查询站点币种。

## 排序字段

| 维度 | `sort` 可选值 |
| --- | --- |
| 蓝海指数 | `dsr` |
| 总销量/平均价格/总销售额 | `totalSold`、`avgPrice`、`totalSales` |
| 市场时间 | `firstOnSaleTime` |
| 总商品数/半托管商品数 | `itemCount`、`semiManagedItemCount` |
| 总店铺数/半托管店铺数 | `mallCount`、`semiManagedMallCount` |
| 日/周/月销量 | `daySold`、`weekSold`、`monthSold` |
| 日/周/月销量增长率 | `daySoldRate`、`weekSoldRate`、`monthSoldRate` |
| 日/周/月销售额 | `daySales`、`weekSales`、`monthSales` |
| 日/周/月销售额增长率 | `daySalesRate`、`weekSalesRate`、`monthSalesRate` |
| 日/周/月商品数变化量 | `dayItemCount`、`weekItemCount`、`monthItemCount` |
| 日/周/月商品数增长率 | `dayItemCountRate`、`weekItemCountRate`、`monthItemCountRate` |
| 日/周/月店铺数变化量 | `dayMallCount`、`weekMallCount`、`monthMallCount` |
| 日/周/月店铺数增长率 | `dayMallCountRate`、`weekMallCountRate`、`monthMallCountRate` |
| 数据创建/更新时间 | `createTime`、`updateTime` |

`order` 使用 `asc` 或 `desc`。

## 成功响应

```json
{
  "code": 0,
  "data": {
    "total": 100,
    "list": []
  }
}
```

- `data.total` 是当前条件下的命中总数，`data.list` 是当前页关键词列表。
- 基础字段包括站点 ID、英文关键词、中文关键词、缩略图、市场时间、关联类目和数据时间。
- 市场指标包括蓝海指数、总销量、平均价格、总销售额、商品数、店铺数、半托管商品/店铺数，以及对应日/周/月指标与增长率。
- `catItems` 是服务端根据关联类目 ID 补全的类目名称和层级，面向用户展示时优先使用它。
- `linkUrl` 指向该关键词在极鲸云的 Temu 关键词详情页。展示结果时必须将每个关键词名称写成 `[关键词](<linkUrl>)`；该要求覆盖列表、表格、排名、候选清单和正文，不因用户未主动要求链接、要求简洁回答或表格空间有限而省略。

## 失败处理

- 服务端要求暂停查询时，按 [查询暂停与恢复流程](查询暂停与恢复流程.md) 展示提示，有跳转地址时再展示可点击链接，不把它解释为无数据。
- 退出码 `1` 时读取 stderr 一级 `msg`，面向用户只提示该中文文案。
- `code != 0`、HTTP 非 2xx、响应不是 JSON 或缺少 `data` 时，本次查询失败。
- 请求多页时逐页累计并记录实际读取页数；未完成全部分页时不得声称结果为全量。
