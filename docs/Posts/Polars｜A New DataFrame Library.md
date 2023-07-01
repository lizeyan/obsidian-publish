---
share: true
tags:
- Notes
---

`Polars`可理解为`Pandas`的竞品，相比之下，主要的优势有几点：
- 速度快。通过一个简单的例子（读取一个272M的CSV文件）就能看出来：
```python
In [1]: import pandas as pd; import polars as pl
time
In [2]: %timeit pd.read_csv("37.csv")
1.78 s ± 21 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

In [3]: %timeit pl.read_csv("37.csv")
174 ms ± 16.5 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```
- API的类型是稳定的。`Pandas`的很多API，返回值的类型和数据的具体情况是有关的，比如如果查询出来的数据只有一行就返回`Series`，是多行就返回`DataFrame`，这非常不利于脚本的编写，只适合交互式的使用。

## 基本的数据结构
`Polars`的基本数据结构也是`Series`和`DataFrame`。`Series`指的是一列，其中的所有元素都有相同的数据类型。
```python
In [4]: import polars as pl
   ...:
   ...: s = pl.Series("a", [1, 2, 3, 4, 5])
   ...: print(s)
shape: (5,)
Series: 'a' [i64]
[
	1
	2
	3
	4
	5
]
```
`Series`中的基本数据类型：
- 数值
- 结构，包括`Struct`和`List`
- 时间，包括`Date, Datetime`, `Duration`, `Time`
- 其他，包括`Categorical`, `Object`, `Utf8`等等
```
`DataFrame`是多列`Series`组成的二维结构：
```python
In [7]: df = pl.DataFrame(
   ...:     {
   ...:         "integer": [1, 2, 3, 4, 5],
   ...:         "date": [
   ...:             datetime(2022, 1, 1),
   ...:             datetime(2022, 1, 2),
   ...:             datetime(2022, 1, 3),
   ...:             datetime(2022, 1, 4),
   ...:             datetime(2022, 1, 5),
   ...:         ],
   ...:         "float": [4.0, 5.0, 6.0, 7.0, 8.0],
   ...:     }
   ...: )
   ...:
   ...: print(df)
shape: (5, 3)
┌─────────┬─────────────────────┬───────┐
│ integer ┆ date                ┆ float │
│ ---     ┆ ---                 ┆ ---   │
│ i64     ┆ datetime[μs]        ┆ f64   │
╞═════════╪═════════════════════╪═══════╡
│ 1       ┆ 2022-01-01 00:00:00 ┆ 4.0   │
│ 2       ┆ 2022-01-02 00:00:00 ┆ 5.0   │
│ 3       ┆ 2022-01-03 00:00:00 ┆ 6.0   │
│ 4       ┆ 2022-01-04 00:00:00 ┆ 7.0   │
│ 5       ┆ 2022-01-05 00:00:00 ┆ 8.0   │
└─────────┴─────────────────────┴───────┘
```

`dataframe.describe`可以描述一个`DataFrame`的整体情况：
```python
In [8]: df.describe()
Out[8]:
shape: (9, 4)
┌────────────┬──────────┬─────────────────────┬──────────┐
│ describe   ┆ integer  ┆ date                ┆ float    │
│ ---        ┆ ---      ┆ ---                 ┆ ---      │
│ str        ┆ f64      ┆ str                 ┆ f64      │
╞════════════╪══════════╪═════════════════════╪══════════╡
│ count      ┆ 5.0      ┆ 5                   ┆ 5.0      │
│ null_count ┆ 0.0      ┆ 0                   ┆ 0.0      │
│ mean       ┆ 3.0      ┆ null                ┆ 6.0      │
│ std        ┆ 1.581139 ┆ null                ┆ 1.581139 │
│ min        ┆ 1.0      ┆ 2022-01-01 00:00:00 ┆ 4.0      │
│ max        ┆ 5.0      ┆ 2022-01-05 00:00:00 ┆ 8.0      │
│ median     ┆ 3.0      ┆ null                ┆ 6.0      │
│ 25%        ┆ 2.0      ┆ null                ┆ 5.0      │
│ 75%        ┆ 4.0      ┆ null                ┆ 7.0      │
└────────────┴──────────┴─────────────────────┴──────────┘
```
`dataframe.shape`返回一个`Tuple`，分别是`dataframe`的行和列


## Context 和 Expression
`Polars` 中进行数据的变换依赖两个概念，分别是 Context 和 Expression。
Context有三种：
- Selection: `dataframe.select`, `dataframe.with_columns`
- Filter: `dataframe.filter`
- Groupby: `dataframe.groupby().agg()`

### Selection
Selection 中的每一个 expression，都必须生成一个 Series，这些 Series必须要有相同的长度，或者长度为1（以进行broadcast）。
``` python
In [16]: df = pl.DataFrame(
    ...:     {
    ...:         "nrs": [1, 2, 3, None, 5],
    ...:         "names": ["foo", "ham", "spam", "egg", None],
    ...:         "random": np.random.rand(5),
    ...:         "groups": ["A", "A", "B", "C", "B"],
    ...:     }
    ...: )
    ...: print(df)
shape: (5, 4)
┌──────┬───────┬──────────┬────────┐
│ nrs  ┆ names ┆ random   ┆ groups │
│ ---  ┆ ---   ┆ ---      ┆ ---    │
│ i64  ┆ str   ┆ f64      ┆ str    │
╞══════╪═══════╪══════════╪════════╡
│ 1    ┆ foo   ┆ 0.136646 ┆ A      │
│ 2    ┆ ham   ┆ 0.124722 ┆ A      │
│ 3    ┆ spam  ┆ 0.993891 ┆ B      │
│ null ┆ egg   ┆ 0.864888 ┆ C      │
│ 5    ┆ null  ┆ 0.585665 ┆ B      │
└──────┴───────┴──────────┴────────┘
```
```python
In [17]: out = df.select(
    ...:     [
    ...:         pl.sum("nrs"),
    ...:         pl.col("names").sort(),
    ...:         pl.col("names").first().alias("first name"),
    ...:         (pl.mean("nrs") * 10).alias("10xnrs"),
    ...:     ]
    ...: )
    ...: print(out)
shape: (5, 4)
┌─────┬───────┬────────────┬────────┐
│ nrs ┆ names ┆ first name ┆ 10xnrs │
│ --- ┆ ---   ┆ ---        ┆ ---    │
│ i64 ┆ str   ┆ str        ┆ f64    │
╞═════╪═══════╪════════════╪════════╡
│ 11  ┆ null  ┆ foo        ┆ 27.5   │
│ 11  ┆ egg   ┆ foo        ┆ 27.5   │
│ 11  ┆ foo   ┆ foo        ┆ 27.5   │
│ 11  ┆ ham   ┆ foo        ┆ 27.5   │
│ 11  ┆ spam  ┆ foo        ┆ 27.5   │
└────
```
### Filter
Filter 接受一个 expression 作为输入，该 expression 的输出将被 evaluate 为 bool 变量，以对 row 进行筛选。
```python
In [18]: out = df.filter(pl.col("nrs") > 2)
    ...: print(out)
shape: (2, 4)
┌─────┬───────┬──────────┬────────┐
│ nrs ┆ names ┆ random   ┆ groups │
│ --- ┆ ---   ┆ ---      ┆ ---    │
│ i64 ┆ str   ┆ f64      ┆ str    │
╞═════╪═══════╪══════════╪════════╡
│ 3   ┆ spam  ┆ 0.993891 ┆ B      │
│ 5   ┆ null  ┆ 0.585665 ┆ B      │
└─────┴───────┴──────────┴────────┘
```
### Groupby
Groupby之后，在`agg`中，每一个 expression 将作用在每一个group上
```python
In [21]: df.groupby("groups").agg(pl.col("nrs"))
Out[21]:
shape: (3, 2)
┌────────┬───────────┐
│ groups ┆ nrs       │
│ ---    ┆ ---       │
│ str    ┆ list[i64] │
╞════════╪═══════════╡
│ C      ┆ [null]    │
│ A      ┆ [1, 2]    │
│ B      ┆ [3, 5]    │
└────────┴───────────┘
```
```python
In [19]: out = df.groupby("groups").agg(
    ...:     [
    ...:         pl.sum("nrs"),  # sum nrs by groups
    ...:         pl.col("random").count().alias("count"),  # count group members
    ...:         # sum random where name != null
    ...:         pl.col("random").filter(pl.col("names").is_not_null()).sum().suffix("_sum"),
    ...:         pl.col("names").reverse().alias(("reversed names")),
    ...:     ]
    ...: )
    ...: print(out)
shape: (3, 5)
┌────────┬──────┬───────┬────────────┬────────────────┐
│ groups ┆ nrs  ┆ count ┆ random_sum ┆ reversed names │
│ ---    ┆ ---  ┆ ---   ┆ ---        ┆ ---            │
│ str    ┆ i64  ┆ u32   ┆ f64        ┆ list[str]      │
╞════════╪══════╪═══════╪════════════╪════════════════╡
│ A      ┆ 3    ┆ 2     ┆ 0.261368   ┆ ["ham", "foo"] │
│ B      ┆ 8    ┆ 2     ┆ 0.993891   ┆ [null, "spam"] │
│ C      ┆ null ┆ 1     ┆ 0.864888   ┆ ["egg"]        │
└────────┴──────┴───────┴────────────┴────────────────┘
```

`groupby_dynamic`和`groupby_rolling`是用于时间或者整数类型的列上滑动窗口。
`groupby_dynamic`的间隔是固定的，`groupby_rolling`的时间间隔取决于实际有的值

在 expression 中也能进行类似 groupby的操作:
```python
In [72]: df = pl.DataFrame(
    ...:     {
    ...:         "a": ["a", "a", "b", "b", "b"],
    ...:         "b": [1, 2, 3, 5, 3],
    ...:         "c": [5, 4, 3, 2, 1],
    ...:     }
    ...: )

In [73]: df.with_columns(pl.col("c").max().over("a").suffix("_max"))
Out[73]:
shape: (5, 4)
┌─────┬─────┬─────┬───────┐
│ a   ┆ b   ┆ c   ┆ c_max │
│ --- ┆ --- ┆ --- ┆ ---   │
│ str ┆ i64 ┆ i64 ┆ i64   │
╞═════╪═════╪═════╪═══════╡
│ a   ┆ 1   ┆ 5   ┆ 5     │
│ a   ┆ 2   ┆ 4   ┆ 5     │
│ b   ┆ 3   ┆ 3   ┆ 3     │
│ b   ┆ 5   ┆ 2   ┆ 3     │
│ b   ┆ 3   ┆ 1   ┆ 3     │
└─────┴─────┴─────┴───────┘
```
### Expression
#### 基本的操作符
`pl.col()`表示提取一列，列与列，列与 literal 之间支持四则运算和逻辑运算
#### 选择多列
列可以同时选择多列
```python
In [66]: df, df.select(pl.all())
Out[66]:
(shape: (2, 3)
 ┌──────┬───────┬───────┐
 │ TT   ┆ TF    ┆ FF    │
 │ ---  ┆ ---   ┆ ---   │
 │ bool ┆ bool  ┆ bool  │
 ╞══════╪═══════╪═══════╡
 │ true ┆ true  ┆ false │
 │ true ┆ false ┆ false │
 └──────┴───────┴───────┘,
 shape: (2, 3)
 ┌──────┬───────┬───────┐
 │ TT   ┆ TF    ┆ FF    │
 │ ---  ┆ ---   ┆ ---   │
 │ bool ┆ bool  ┆ bool  │
 ╞══════╪═══════╪═══════╡
 │ true ┆ true  ┆ false │
 │ true ┆ false ┆ false │
 └──────┴───────┴───────┘)
```
```python
In [67]: df.select(pl.concat_list(pl.all()))
Out[67]:
shape: (2, 1)
┌──────────────────────┐
│ TT                   │
│ ---                  │
│ list[bool]           │
╞══════════════════════╡
│ [true, true, false]  │
│ [true, false, false] │
└──────────────────────┘
```
多列可以使用`prefix`或者`suffix`进行批量的重命名
#### NumPy
Expression 支持使用 NumPy 中的 ufuncs
#### 类型转换
```python
In [68]: df = pl.DataFrame(
    ...:     {
    ...:         "integers": [1, 2, 3, 4, 5],
    ...:         "big_integers": [1, 10000002, 3, 10000004, 10000005],
    ...:         "floats": [4.0, 5.0, 6.0, 7.0, 8.0],
    ...:         "floats_with_decimal": [4.532, 5.5, 6.5, 7.5, 8.5],
    ...:     }
    ...: )
    ...:
    ...: print(df)
shape: (5, 4)
┌──────────┬──────────────┬────────┬─────────────────────┐
│ integers ┆ big_integers ┆ floats ┆ floats_with_decimal │
│ ---      ┆ ---          ┆ ---    ┆ ---                 │
│ i64      ┆ i64          ┆ f64    ┆ f64                 │
╞══════════╪══════════════╪════════╪═════════════════════╡
│ 1        ┆ 1            ┆ 4.0    ┆ 4.532               │
│ 2        ┆ 10000002     ┆ 5.0    ┆ 5.5                 │
│ 3        ┆ 3            ┆ 6.0    ┆ 6.5                 │
│ 4        ┆ 10000004     ┆ 7.0    ┆ 7.5                 │
│ 5        ┆ 10000005     ┆ 8.0    ┆ 8.5                 │
└──────────┴──────────────┴────────┴─────────────────────┘

In [69]: out = df.select(
    ...:     [
    ...:         pl.col("integers").cast(pl.Float32).alias("integers_as_floats"),
    ...:         pl.col("floats").cast(pl.Int32).alias("floats_as_integers"),
    ...:         pl.col("floats_with_decimal")
    ...:         .cast(pl.Int32)
    ...:         .alias("floats_with_decimal_as_integers"),
    ...:     ]
    ...: )
    ...: print(out)
shape: (5, 3)
┌────────────────────┬────────────────────┬─────────────────────────────────┐
│ integers_as_floats ┆ floats_as_integers ┆ floats_with_decimal_as_integers │
│ ---                ┆ ---                ┆ ---                             │
│ f32                ┆ i32                ┆ i32                             │
╞════════════════════╪════════════════════╪═════════════════════════════════╡
│ 1.0                ┆ 4                  ┆ 4                               │
│ 2.0                ┆ 5                  ┆ 5                               │
│ 3.0                ┆ 6                  ┆ 6                               │
│ 4.0                ┆ 7                  ┆ 7                               │
│ 5.0                ┆ 8                  ┆ 8                               │
└────────────────────┴────────────────────┴─────────────────────────────────┘
```
#### str 的API
- `polars.Expr.str.concat
- `polars.Expr.str.contains`
- `polars.Expr.str.count_match`
- `polars.Expr.str.decode`
- `polars.Expr.str.encode`
- `polars.Expr.str.ends_with`
- `polars.Expr.str.explode`
- `polars.Expr.str.extract`
- `polars.Expr.str.extract_all`
- `polars.Expr.str.json_extract`
- `polars.Expr.str.json_path_match`
- `polars.Expr.str.lengths`
- `polars.Expr.str.ljust`
- `polars.Expr.str.lstrip`
- `polars.Expr.str.n_chars`
- `polars.Expr.str.replace`
- `polars.Expr.str.replace_all`
- `polars.Expr.str.rjust`
- `polars.Expr.str.rstrip`
- `polars.Expr.str.slice`
- `polars.Expr.str.split`
- `polars.Expr.str.split_exact`
- `polars.Expr.str.splitn`
- `polars.Expr.str.starts_with`
- `polars.Expr.str.strip`
- `polars.Expr.str.strptime`
- `polars.Expr.str.to_date`
- `polars.Expr.str.to_datetime`
- `polars.Expr.str.to_decimal`
- `polars.Expr.str.to_lowercase`
- `polars.Expr.str.to_titlecase`
- `polars.Expr.str.to_time`
- `polars.Expr.str.to_uppercase`
- `polars.Expr.str.zfill`
- `polars.Expr.str.parse_int`
#### Aggregations
和Pandas差不多
#### Window Functions
`Window function`的作用是在 selection context 中进行 groupby
```python
In [76]: filtered
Out[76]:
shape: (7, 3)
┌─────────────────────┬────────┬───────┐
│ Name                ┆ Type 1 ┆ Speed │
│ ---                 ┆ ---    ┆ ---   │
│ str                 ┆ str    ┆ i64   │
╞═════════════════════╪════════╪═══════╡
│ Slowpoke            ┆ Water  ┆ 15    │
│ Slowbro             ┆ Water  ┆ 30    │
│ SlowbroMega Slowbro ┆ Water  ┆ 30    │
│ Exeggcute           ┆ Grass  ┆ 40    │
│ Exeggutor           ┆ Grass  ┆ 55    │
│ Starmie             ┆ Water  ┆ 115   │
│ Jynx                ┆ Ice    ┆ 95    │
└─────────────────────┴────────┴───────┘

In [77]: out = filtered.with_columns(
    ...:     [
    ...:         pl.col(["Name", "Speed"]).sort_by("Speed", descending=True).over("Type 1"),
    ...:     ]
    ...: )
    ...: print(out)
shape: (7, 3)
┌─────────────────────┬────────┬───────┐
│ Name                ┆ Type 1 ┆ Speed │
│ ---                 ┆ ---    ┆ ---   │
│ str                 ┆ str    ┆ i64   │
╞═════════════════════╪════════╪═══════╡
│ Starmie             ┆ Water  ┆ 115   │
│ Slowbro             ┆ Water  ┆ 30    │
│ SlowbroMega Slowbro ┆ Water  ┆ 30    │
│ Exeggutor           ┆ Grass  ┆ 55    │
│ Exeggcute           ┆ Grass  ┆ 40    │
│ Slowpoke            ┆ Water  ┆ 15    │
│ Jynx                ┆ Ice    ┆ 95    │
└─────────────────────┴────────┴───────┘
```
`over`中的aggregation可以返回不止一个值：
```python
# aggregate and broadcast within a group # output type: -> Int32 
pl.sum("foo").over("groups") 
# sum within a group and multiply with group elements # output type: -> Int32 
(pl.col("x").sum() * pl.col("y")).over("groups") 
# sum within a group and multiply with group elements # and aggregate the group to a list # output type: -> List(Int32) 
(pl.col("x").sum() * pl.col("y")).over("groups", mapping_strategy="join") 
# sum within a group and multiply with group elements # and aggregate the group to a list # then explode the list to multiple rows 
# This is the fastest method to do things over groups when the groups are sorted 
(pl.col("x").sum() * pl.col("y")).over("groups", mapping_strategy="explode")
```

```python
In [78]: # then let's load some csv data with information about pokemon
    ...: df = pl.read_csv(
    ...:     "https://gist.githubusercontent.com/ritchie46/cac6b337ea52281aa23c049250a4ff03/raw/89a957ff3919d90e6ef2d34235e6bf22304f3366/pokemon.csv"
    ...: )
    ...: print(df.head())
shape: (5, 13)
┌─────┬───────────────────────┬────────┬────────┬───┬─────────┬───────┬────────────┬───────────┐
│ #   ┆ Name                  ┆ Type 1 ┆ Type 2 ┆ … ┆ Sp. Def ┆ Speed ┆ Generation ┆ Legendary │
│ --- ┆ ---                   ┆ ---    ┆ ---    ┆   ┆ ---     ┆ ---   ┆ ---        ┆ ---       │
│ i64 ┆ str                   ┆ str    ┆ str    ┆   ┆ i64     ┆ i64   ┆ i64        ┆ bool      │
╞═════╪═══════════════════════╪════════╪════════╪═══╪═════════╪═══════╪════════════╪═══════════╡
│ 1   ┆ Bulbasaur             ┆ Grass  ┆ Poison ┆ … ┆ 65      ┆ 45    ┆ 1          ┆ false     │
│ 2   ┆ Ivysaur               ┆ Grass  ┆ Poison ┆ … ┆ 80      ┆ 60    ┆ 1          ┆ false     │
│ 3   ┆ Venusaur              ┆ Grass  ┆ Poison ┆ … ┆ 100     ┆ 80    ┆ 1          ┆ false     │
│ 3   ┆ VenusaurMega Venusaur ┆ Grass  ┆ Poison ┆ … ┆ 120     ┆ 80    ┆ 1          ┆ false     │
│ 4   ┆ Charmander            ┆ Fire   ┆ null   ┆ … ┆ 50      ┆ 65    ┆ 1          ┆ false     │
└─────┴───────────────────────┴────────┴────────┴───┴─────────┴───────┴────────────┴───────────┘

In [79]: out = df.sort("Type 1").select(
    ...:     pl.col("Type 1").head(3).over("Type 1", mapping_strategy="explode"),
    ...:     pl.col("Name")
    ...:     .sort_by(pl.col("Speed"))
    ...:     .head(3)
    ...:     .over("Type 1", mapping_strategy="explode")
    ...:     .alias("fastest/group"),
    ...:     pl.col("Name")
    ...:     .sort_by(pl.col("Attack"))
    ...:     .head(3)
    ...:     .over("Type 1", mapping_strategy="explode")
    ...:     .alias("strongest/group"),
    ...:     pl.col("Name")
    ...:     .sort()
    ...:     .head(3)
    ...:     .over("Type 1", mapping_strategy="explode")
    ...:     .alias("sorted_by_alphabet"),
    ...: )
    ...: print(out)
shape: (43, 4)
┌────────┬─────────────────────┬─────────────────┬─────────────────────────┐
│ Type 1 ┆ fastest/group       ┆ strongest/group ┆ sorted_by_alphabet      │
│ ---    ┆ ---                 ┆ ---             ┆ ---                     │
│ str    ┆ str                 ┆ str             ┆ str                     │
╞════════╪═════════════════════╪═════════════════╪═════════════════════════╡
│ Bug    ┆ Paras               ┆ Metapod         ┆ Beedrill                │
│ Bug    ┆ Metapod             ┆ Kakuna          ┆ BeedrillMega Beedrill   │
│ Bug    ┆ Parasect            ┆ Caterpie        ┆ Butterfree              │
│ Dragon ┆ Dratini             ┆ Dratini         ┆ Dragonair               │
│ …      ┆ …                   ┆ …               ┆ …                       │
│ Rock   ┆ Omanyte             ┆ Omastar         ┆ Geodude                 │
│ Water  ┆ Slowpoke            ┆ Magikarp        ┆ Blastoise               │
│ Water  ┆ Slowbro             ┆ Tentacool       ┆ BlastoiseMega Blastoise │
│ Water  ┆ SlowbroMega Slowbro ┆ Horsea          ┆ Cloyster                │
└────────┴─────────────────────┴─────────────────┴─────────────────────────┘
```
Join会把列表mapping到每一个原来的位置上，而groupby是不会的
```python
In [103]: df.sort("Type 1").select(pl.col("Name").sort_by(pl.col("Speed")).head(1).over("Type 1", mapping_strategy="join"))
Out[103]:
shape: (163, 1)
┌──────────────┐
│ Name         │
│ ---          │
│ list[str]    │
╞══════════════╡
│ ["Paras"]    │
│ ["Paras"]    │
│ ["Paras"]    │
│ ["Paras"]    │
│ …            │
│ ["Slowpoke"] │
│ ["Slowpoke"] │
│ ["Slowpoke"] │
│ ["Slowpoke"] │
└──────────────┘
```
```python
In [115]: df.groupby("Type 1").agg(pl.col("Name").sort_by(pl.col("Speed")).head(3).explode())
Out[115]:
shape: (15, 2)
┌──────────┬───────────────────────────────────┐
│ Type 1   ┆ Name                              │
│ ---      ┆ ---                               │
│ str      ┆ list[str]                         │
╞══════════╪═══════════════════════════════════╡
│ Bug      ┆ ["Paras", "Metapod", "Parasect"]  │
│ Water    ┆ ["Slowpoke", "Slowbro", "Slowbro… │
│ Normal   ┆ ["Jigglypuff", "Lickitung", "Sno… │
│ Grass    ┆ ["Oddish", "Gloom", "Bellsprout"… │
│ …        ┆ …                                 │
│ Ice      ┆ ["Articuno", "Jynx"]              │
│ Fighting ┆ ["Machop", "Machoke", "Machamp"]  │
│ Fairy    ┆ ["Clefairy", "Clefable"]          │
│ Ghost    ┆ ["Gastly", "Haunter", "Gengar"]   │
└──────────┴───────────────────────────────────┘

In [116]: df.groupby("Type 1").agg(pl.col("Name").sort_by(pl.col("Speed")).head(3)).select(pl.col("Name").explode())
Out[116]:
shape: (43, 1)
┌──────────┐
│ Name     │
│ ---      │
│ str      │
╞══════════╡
│ Geodude  │
│ Graveler │
│ Omanyte  │
│ Articuno │
│ …        │
│ Pikachu  │
│ Grimer   │
│ Koffing  │
│ Nidoran♀ │
└──────────┘
```
#### Folds
Fold 用来进行 columns 之间的聚合操作，即 horizontal reduction

```python
# 求每一列的和
In [117]: df = pl.DataFrame(
     ...:     {
     ...:         "a": [1, 2, 3],
     ...:         "b": [10, 20, 30],
     ...:     }
     ...: )
     ...:
     ...: out = df.select(
     ...:     pl.fold(acc=pl.lit(0), function=lambda acc, x: acc + x, exprs=pl.all()).alias(
     ...:         "sum"
     ...:     ),
     ...: )
     ...: print(out)
shape: (3, 1)
┌─────┐
│ sum │
│ --- │
│ i64 │
╞═════╡
│ 11  │
│ 22  │
│ 33  │
└─────┘
```
```python
# 筛选每列都大于1的行
In [118]: df = pl.DataFrame(
     ...:     {
     ...:         "a": [1, 2, 3],
     ...:         "b": [0, 1, 2],
     ...:     }
     ...: )
     ...:
     ...: out = df.filter(
     ...:     pl.fold(
     ...:         acc=pl.lit(True),
     ...:         function=lambda acc, x: acc & x,
     ...:         exprs=pl.col("*") > 1,
     ...:     )
     ...: )
     ...: print(out)
shape: (1, 2)
┌─────┬─────┐
│ a   ┆ b   │
│ --- ┆ --- │
│ i64 ┆ i64 │
╞═════╪═════╡
│ 3   ┆ 2   │
└─────┴─────┘
```
为了提高性能，在使用fold之前，最好先想想能不能用内置方法替代：
```python
df = pl.DataFrame(
     ...:     {
     ...:         "a": ["a", "b", "c"],
     ...:         "b": [1, 2, 3],
     ...:     }
     ...: )
     ...:
     ...: out = df.select(
     ...:     [
     ...:         pl.concat_str(["a", "b"]),
     ...:     ]
     ...: )
     ...: print(out)
shape: (3, 1)
┌─────┐
│ a   │
│ --- │
│ str │
╞═════╡
│ a1  │
│ b2  │
│ c3  │
└─────┘
```
需要注意，字符串表示列名，想要用literal需要用`pl.lit`函数：
```python
In [120]: df = pl.DataFrame(
     ...:     {
     ...:         "a": ["a", "b", "c"],
     ...:         "b": [1, 2, 3],
     ...:     }
     ...: )
     ...:
     ...: out = df.select(
     ...:     [
     ...:         pl.concat_str(["a", "b", pl.lit("123")]),
     ...:     ]
     ...: )
     ...: print(out)
shape: (3, 1)
┌───────┐
│ a     │
│ ---   │
│ str   │
╞═══════╡
│ a1123 │
│ b2123 │
│ c3123 │
└───────┘
```
#### List and array

```python
# 通过split等API可以创建lists
In [121]: weather = pl.DataFrame(
     ...:     {
     ...:         "station": ["Station " + str(x) for x in range(1, 6)],
     ...:         "temperatures": [
     ...:             "20 5 5 E1 7 13 19 9 6 20",
     ...:             "18 8 16 11 23 E2 8 E2 E2 E2 90 70 40",
     ...:             "19 24 E9 16 6 12 10 22",
     ...:             "E2 E0 15 7 8 10 E1 24 17 13 6",
     ...:             "14 8 E0 16 22 24 E1",
     ...:         ],
     ...:     }
     ...: )
     ...: print(weather)
shape: (5, 2)
┌───────────┬───────────────────────────────────┐
│ station   ┆ temperatures                      │
│ ---       ┆ ---                               │
│ str       ┆ str                               │
╞═══════════╪═══════════════════════════════════╡
│ Station 1 ┆ 20 5 5 E1 7 13 19 9 6 20          │
│ Station 2 ┆ 18 8 16 11 23 E2 8 E2 E2 E2 90 7… │
│ Station 3 ┆ 19 24 E9 16 6 12 10 22            │
│ Station 4 ┆ E2 E0 15 7 8 10 E1 24 17 13 6     │
│ Station 5 ┆ 14 8 E0 16 22 24 E1               │
└───────────┴───────────────────────────────────┘

In [122]: weather.with_columns(pl.col("temperatures").str.split(" "))
Out[122]:
shape: (5, 2)
┌───────────┬──────────────────────┐
│ station   ┆ temperatures         │
│ ---       ┆ ---                  │
│ str       ┆ list[str]            │
╞═══════════╪══════════════════════╡
│ Station 1 ┆ ["20", "5", … "20"]  │
│ Station 2 ┆ ["18", "8", … "40"]  │
│ Station 3 ┆ ["19", "24", … "22"] │
│ Station 4 ┆ ["E2", "E0", … "6"]  │
│ Station 5 ┆ ["14", "8", … "E1"]  │
└───────────┴──────────────────────┘
```

List 的 element-wise 操作，其中`pl.element()`指代要计算的元素：
```python
In [132]: weather.with_columns(pl.col("temperatures").str.split(" ").list.eval(pl.element() + "1"))
Out[132]:
shape: (5, 2)
┌───────────┬─────────────────────────┐
│ station   ┆ temperatures            │
│ ---       ┆ ---                     │
│ str       ┆ list[str]               │
╞═══════════╪═════════════════════════╡
│ Station 1 ┆ ["201", "51", … "201"]  │
│ Station 2 ┆ ["181", "81", … "401"]  │
│ Station 3 ┆ ["191", "241", … "221"] │
│ Station 4 ┆ ["E21", "E01", … "61"]  │
│ Station 5 ┆ ["141", "81", … "E11"]  │
└───────────┴─────────────────────────┘
```
```python
In [133]: out = weather.with_columns(
     ...:     pl.col("temperatures")
     ...:     .str.split(" ")
     ...:     .list.eval(pl.element().cast(pl.Int64, strict=False).is_null())
     ...:     .list.sum()
     ...:     .alias("errors")
     ...: )
     ...: print(out)
shape: (5, 3)
┌───────────┬───────────────────────────────────┬────────┐
│ station   ┆ temperatures                      ┆ errors │
│ ---       ┆ ---                               ┆ ---    │
│ str       ┆ str                               ┆ u32    │
╞═══════════╪═══════════════════════════════════╪════════╡
│ Station 1 ┆ 20 5 5 E1 7 13 19 9 6 20          ┆ 1      │
│ Station 2 ┆ 18 8 16 11 23 E2 8 E2 E2 E2 90 7… ┆ 4      │
│ Station 3 ┆ 19 24 E9 16 6 12 10 22            ┆ 1      │
│ Station 4 ┆ E2 E0 15 7 8 10 E1 24 17 13 6     ┆ 3      │
│ Station 5 ┆ 14 8 E0 16 22 24 E1               ┆ 2      │
└───────────┴───────────────────────────────────┴────────┘
```
```python
In [135]: weather_by_day = pl.DataFrame(
     ...:     {
     ...:         "station": ["Station " + str(x) for x in range(1, 11)],
     ...:         "day_1": [17, 11, 8, 22, 9, 21, 20, 8, 8, 17],
     ...:         "day_2": [15, 11, 10, 8, 7, 14, 18, 21, 15, 13],
     ...:         "day_3": [16, 15, 24, 24, 8, 23, 19, 23, 16, 10],
     ...:     }
     ...: )
     ...: print(weather_by_day)
shape: (10, 4)
┌────────────┬───────┬───────┬───────┐
│ station    ┆ day_1 ┆ day_2 ┆ day_3 │
│ ---        ┆ ---   ┆ ---   ┆ ---   │
│ str        ┆ i64   ┆ i64   ┆ i64   │
╞════════════╪═══════╪═══════╪═══════╡
│ Station 1  ┆ 17    ┆ 15    ┆ 16    │
│ Station 2  ┆ 11    ┆ 11    ┆ 15    │
│ Station 3  ┆ 8     ┆ 10    ┆ 24    │
│ Station 4  ┆ 22    ┆ 8     ┆ 24    │
│ …          ┆ …     ┆ …     ┆ …     │
│ Station 7  ┆ 20    ┆ 18    ┆ 19    │
│ Station 8  ┆ 8     ┆ 21    ┆ 23    │
│ Station 9  ┆ 8     ┆ 15    ┆ 16    │
│ Station 10 ┆ 17    ┆ 13    ┆ 10    │
└────────────┴───────┴───────┴───────┘

In [136]: rank_pct = (pl.element().rank(descending=True) / pl.col("*").count()).round(2)
     ...:
     ...: out = weather_by_day.with_columns(
     ...:     # create the list of homogeneous data
     ...:     pl.concat_list(pl.all().exclude("station")).alias("all_temps")
     ...: ).select(
     ...:     # select all columns except the intermediate list
     ...:     pl.all().exclude("all_temps"),
     ...:     # compute the rank by calling `list.eval`
     ...:     pl.col("all_temps").list.eval(rank_pct, parallel=True).alias("temps_rank"),
     ...: )
     ...:
     ...: print(out)
shape: (10, 5)
┌────────────┬───────┬───────┬───────┬────────────────────┐
│ station    ┆ day_1 ┆ day_2 ┆ day_3 ┆ temps_rank         │
│ ---        ┆ ---   ┆ ---   ┆ ---   ┆ ---                │
│ str        ┆ i64   ┆ i64   ┆ i64   ┆ list[f64]          │
╞════════════╪═══════╪═══════╪═══════╪════════════════════╡
│ Station 1  ┆ 17    ┆ 15    ┆ 16    ┆ [0.33, 1.0, 0.67]  │
│ Station 2  ┆ 11    ┆ 11    ┆ 15    ┆ [0.83, 0.83, 0.33] │
│ Station 3  ┆ 8     ┆ 10    ┆ 24    ┆ [1.0, 0.67, 0.33]  │
│ Station 4  ┆ 22    ┆ 8     ┆ 24    ┆ [0.67, 1.0, 0.33]  │
│ …          ┆ …     ┆ …     ┆ …     ┆ …                  │
│ Station 7  ┆ 20    ┆ 18    ┆ 19    ┆ [0.33, 1.0, 0.67]  │
│ Station 8  ┆ 8     ┆ 21    ┆ 23    ┆ [1.0, 0.67, 0.33]  │
│ Station 9  ┆ 8     ┆ 15    ┆ 16    ┆ [1.0, 0.67, 0.33]  │
│ Station 10 ┆ 17    ┆ 13    ┆ 10    ┆ [0.33, 0.67, 1.0]  │
└────────────┴───────┴───────┴───────┴────────────────────┘
```
#### Map and APPLy
Map 作用于列，但是在 groupby context 中，map 会先作用于列，再进行aggregation。所以永远不要在 groupby context 中使用 map。
```python
In [137]: df = pl.DataFrame(
     ...:     {
     ...:         "keys": ["a", "a", "b"],
     ...:         "values": [10, 7, 1],
     ...:     }
     ...: )
     ...:
     ...: out = df.groupby("keys", maintain_order=True).agg(
     ...:     [
     ...:         pl.col("values").map(lambda s: s.shift()).alias("shift_map"),
     ...:         pl.col("values").shift().alias("shift_expression"),
     ...:     ]
     ...: )
     ...: print(df)
shape: (3, 2)
┌──────┬────────┐
│ keys ┆ values │
│ ---  ┆ ---    │
│ str  ┆ i64    │
╞══════╪════════╡
│ a    ┆ 10     │
│ a    ┆ 7      │
│ b    ┆ 1      │
└──────┴────────┘
```
Apple 在 selection context 中作用于 element，在 groupby context 中作用于 group。
```python
In [138]: out = df.groupby("keys", maintain_order=True).agg(
     ...:     [
     ...:         pl.col("values").apply(lambda s: s.shift()).alias("shift_map"),
     ...:         pl.col("values").shift().alias("shift_expression"),
     ...:     ]
     ...: )
     ...: print(out)
shape: (2, 3)
┌──────┬────────────┬──────────────────┐
│ keys ┆ shift_map  ┆ shift_expression │
│ ---  ┆ ---        ┆ ---              │
│ str  ┆ list[i64]  ┆ list[i64]        │
╞══════╪════════════╪══════════════════╡
│ a    ┆ [null, 10] ┆ [null, 10]       │
│ b    ┆ [null]     ┆ [null]           │
└──────┴────────────┴──────────────────┘
```
## Transformation
和`Pandas`中类似，包括`concat`， `melt`, `pivot`, `join`
## Polars中没有 Index 的概念
[Understand Polars’ Lack of Indexes | by Carl M. Kadie | Towards Data Science](https://towardsdatascience.com/understand-polars-lack-of-indexes-526ea75e413)
## 缺失值
在Polars中，只有`None`才是缺失值，而`NaN`是`float`类型中一个普通的值
```python
In [29]: df = pl.DataFrame(
    ...:     {
    ...:         "value": [1., float('NaN'), None],
    ...:     },
    ...: )
    ...: print(df)
shape: (3, 1)
┌───────┐
│ value │
│ ---   │
│ f64   │
╞═══════╡
│ 1.0   │
│ NaN   │
│ null  │
└───────┘

In [30]: df.null_count()
Out[30]:
shape: (1, 1)
┌───────┐
│ value │
│ ---   │
│ u32   │
╞═══════╡
│ 1     │
└───────┘
```
在`Pandas`中，`NaN`也是缺失值（如果列的类型是`float`的话）

`DataFrame.interpolate`可以进行线性插值
## 各种特殊操作的API
####  Explode: 把某一列 list 展开
```python
In [26]: out = df.groupby("groups").agg(pl.col("nrs"))

In [27]: out
Out[27]:
shape: (3, 2)
┌────────┬───────────┐
│ groups ┆ nrs       │
│ ---    ┆ ---       │
│ str    ┆ list[i64] │
╞════════╪═══════════╡
│ B      ┆ [3, 5]    │
│ A      ┆ [1, 2]    │
│ C      ┆ [null]    │
└────────┴───────────┘

In [28]: out.explode("nrs")
Out[28]:
shape: (5, 2)
┌────────┬──────┐
│ groups ┆ nrs  │
│ ---    ┆ ---  │
│ str    ┆ i64  │
╞════════╪══════╡
│ B      ┆ 3    │
│ B      ┆ 5    │
│ A      ┆ 1    │
│ A      ┆ 2    │
│ C      ┆ null │
└────────┴──────┘
```

## 实际问题
