# SQL 速查表

本页包含一些帮助信息，供你查询 sqlite 数据库时使用。

!!! Tip "其他数据库系统"
    要使用其他数据库系统如 PostgreSQL 或 MariaDB，你可以使用相同的查询，但需要使用对应数据库系统的客户端。[点击此处](advanced-setup.md#use-a-different-database-system)了解如何在 freqtrade 中设置不同的数据库系统。

!!! Warning
    如果你不熟悉 SQL，在数据库上运行查询时应非常小心。
    在运行任何查询之前，请始终确保已备份数据库。

## 安装 sqlite3

Sqlite3 是一个基于终端的 sqlite 应用程序。
如果你觉得更舒服，也可以使用像 SqliteBrowser 这样的可视化数据库编辑器。

### Ubuntu/Debian 安装

```bash
sudo apt-get install sqlite3
```

### 通过 docker 使用 sqlite3

freqtrade docker 镜像包含 sqlite3，因此你无需在宿主机系统上安装任何东西即可编辑数据库。

``` bash
docker compose exec freqtrade /bin/bash
sqlite3 <database-file>.sqlite
```

## 打开数据库

```bash
sqlite3
.open <filepath>
```

## 表结构

### 列出表

```bash
.tables
```

### 显示表结构

```bash
.schema <table_name>
```

### 获取表中的所有交易

```sql
SELECT * FROM trades;
```

## 破坏性查询

写入数据库的查询。
这些查询通常不应需要，因为 freqtrade 会尝试自行处理所有数据库操作——或通过 API 或 Telegram 命令暴露它们。

!!! Warning
    在运行以下任何查询之前，请确保你已备份数据库。

!!! Danger
    你也不应在机器人连接到数据库时运行任何写入查询（`update`、`insert`、`delete`）。
    这会导致数据损坏——很可能无法恢复。

### 修复在交易所手动平仓后仍显示为未平仓的交易

!!! Warning
    在交易所手动卖出一个交易对不会被机器人检测到，它仍会尝试卖出。尽可能应使用 /forceexit <tradeid> 来完成同样的操作。
    在进行任何手动更改之前，强烈建议备份你的数据库文件。

!!! Note
    在使用 /forceexit 后这应该不是必需的，因为 force_exit 订单会在下一次迭代中由机器人自动关闭。

```sql
UPDATE trades
SET is_open=0,
  close_date=<close_date>,
  close_rate=<close_rate>,
  close_profit = close_rate / open_rate - 1,
  close_profit_abs = (amount * <close_rate> * (1 - fee_close) - (amount * (open_rate * (1 - fee_open)))),
  exit_reason=<exit_reason>
WHERE id=<trade_ID_to_update>;
```

#### 示例

```sql
UPDATE trades
SET is_open=0,
  close_date='2020-06-20 03:08:45.103418',
  close_rate=0.19638016,
  close_profit=0.0496,
  close_profit_abs = (amount * 0.19638016 * (1 - fee_close) - (amount * (open_rate * (1 - fee_open)))),
  exit_reason='force_exit'  
WHERE id=31;
```

### 从数据库中删除交易

!!! Tip "使用 RPC 方法删除交易"
    请考虑通过 Telegram 或 REST API 使用 `/delete <tradeid>`。这是删除交易的推荐方式，因为它也会移除相应的订单和自定义数据，并触发机器人中必要的事件以保持一切同步。

如果你仍想直接从数据库中删除交易，可以使用以下查询。

!!! Danger
    某些系统（如 Ubuntu）在其 sqlite3 打包中禁用了外键。使用 sqlite 时，请确保在运行上述查询前通过运行 `PRAGMA foreign_keys = ON` 启用外键。

```sql
DELETE FROM trades WHERE id = <tradeid>;
DELETE FROM orders WHERE ft_trade_id = <tradeid>;
DELETE FROM trade_custom_data WHERE ft_trade_id = <tradeid>;

DELETE FROM trades WHERE id = 31;
DELETE FROM orders WHERE ft_trade_id = 31;
DELETE FROM trade_custom_data WHERE ft_trade_id = 31;
```

!!! Warning
    这将从数据库中删除指定的交易。请确保你使用了正确的 ID，**绝不**在不带 `where` 子句的情况下运行此查询。