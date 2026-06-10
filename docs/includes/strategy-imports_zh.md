## 策略必需的导入

创建策略时需要导入必要的模块和类。以下推荐作为策略的基线导入：

``` python
# flake8: noqa: F401
# isort: skip_file
# --- 不要移除这些导入 ---
import numpy as np
import pandas as pd
from datetime import datetime, timedelta, timezone
from pandas import DataFrame
from typing import Dict, Optional, Union, Tuple

from freqtrade.strategy import (
    IStrategy,
    Trade, 
    Order,
    PairLocks,
    informative,  # @informative 装饰器
    # 超参优化参数
    BooleanParameter,
    CategoricalParameter,
    DecimalParameter,
    IntParameter,
    RealParameter,
    # 时间周期辅助函数
    timeframe_to_minutes,
    timeframe_to_next_date,
    timeframe_to_prev_date,
    # 策略辅助函数
    merge_informative_pair,
    stoploss_from_absolute,
    stoploss_from_open,
)

# --------------------------------
# 在此处添加你要导入的库
import talib.abstract as ta
from technical import qtpylib
```