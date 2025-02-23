# Narwhals

<h1 align="center">
	<img
		width="400"
		alt="narwhals_small"
		src="https://github.com/MarcoGorelli/narwhals/assets/33491632/26be901e-5383-49f2-9fbd-5c97b7696f27">
</h1>

[![PyPI version](https://badge.fury.io/py/narwhals.svg)](https://badge.fury.io/py/narwhals)
[![Docs](https://img.shields.io/badge/Docs-coolgreen?style=flat&link=https://marcogorelli.github.io/narwhals/)](https://marcogorelli.github.io/narwhals/)
[![Chat with us! - Join Discord](https://img.shields.io/badge/Chat_with_us!-Join_Discord-coolgreen)](https://discord.gg/V3PqtB4VA4)

Extremely lightweight and extensible compatibility layer between Polars, pandas, Modin, and cuDF (and more!).

Seamlessly support all, without depending on any!

- ✅ **Just use** a subset of **the Polars API**, no need to learn anything new
- ✅ **No dependencies** (not even Polars), keep your library lightweight
- ✅ Separate **lazy** and eager APIs
- ✅ Use **Expressions**
- ✅ 100% branch coverage, tested against pandas and Polars nightly builds!

## Used by

Join the party!

- https://github.com/FBruzzesi/timebasedcv

## Installation

```
pip install narwhals
```
Or just vendor it, it's only a bunch of pure-Python files.

## Usage

There are three steps to writing dataframe-agnostic code using Narwhals:

1. use `narwhals.from_native` to wrap a pandas/Polars/Modin/cuDF
   DataFrame/LazyFrame in a Narwhals class
2. use the [subset of the Polars API supported by Narwhals](https://marcogorelli.github.io/narwhals/api-reference/narwhals/)
3. use `narwhals.to_native` to return an object to the user in its original
   dataframe flavour. For example:

   - if you started with pandas, you'll get pandas back
   - if you started with Polars, you'll get Polars back
   - if you started with Modin, you'll get Modin back (and compute will be distributed)
   - if you started with cuDF, you'll get cuDF back (and compute will happen on GPU)

## What about Ibis?

Like Ibis, Narwhals aims to enable dataframe-agnostic code. However, Narwhals comes with **zero** dependencies,
is about as lightweight as it gets, and is aimed at library developers rather than at end users. It also does
not aim to support as many backends, preferring to instead focus on dataframes. So, which should you use?

- If you need to run complicated analyses and aren't too bothered about package size: Ibis!
- If you're a library maintainer and want the thinnest-possible layer to get cross-dataframe library support: Narwhals!

Here is the package size increase which would result from installing each tool in a non-pandas
environment:

![image](https://github.com/MarcoGorelli/narwhals/assets/33491632/a8dfba78-feb1-48c1-960a-5b9b03585fa5)
   
## Example

Here's an example of a dataframe agnostic function:

```python
from typing import Any
import pandas as pd
import polars as pl

import narwhals as nw


def my_agnostic_function(
    suppliers_native,
    parts_native,
):
    suppliers = nw.from_native(suppliers_native)
    parts = nw.from_native(parts_native)

    result = (
        suppliers.join(parts, left_on="city", right_on="city")
        .filter(nw.col("weight") > 10)
        .group_by("s")
        .agg(
            weight_mean=nw.col("weight").mean(),
            weight_max=nw.col("weight").max(),
        )
        .sort("s")
    )

    return nw.to_native(result)
```
You can pass in a pandas or Polars dataframe, the output will be the same!
Let's try it out:

```python
suppliers = {
    "s": ["S1", "S2", "S3", "S4", "S5"],
    "sname": ["Smith", "Jones", "Blake", "Clark", "Adams"],
    "status": [20, 10, 30, 20, 30],
    "city": ["London", "Paris", "Paris", "London", "Athens"],
}
parts = {
    "p": ["P1", "P2", "P3", "P4", "P5", "P6"],
    "pname": ["Nut", "Bolt", "Screw", "Screw", "Cam", "Cog"],
    "color": ["Red", "Green", "Blue", "Red", "Blue", "Red"],
    "weight": [12.0, 17.0, 17.0, 14.0, 12.0, 19.0],
    "city": ["London", "Paris", "Oslo", "London", "Paris", "London"],
}

print("pandas output:")
print(
    my_agnostic_function(
        pd.DataFrame(suppliers),
        pd.DataFrame(parts),
    )
)
print("\nPolars output:")
print(
    my_agnostic_function(
        pl.LazyFrame(suppliers),
        pl.LazyFrame(parts),
    ).collect()
)
```

```
pandas output:
    s  weight_mean  weight_max
0  S1         15.0        19.0
1  S2         14.5        17.0
2  S3         14.5        17.0
3  S4         15.0        19.0

Polars output:
shape: (4, 3)
┌─────┬─────────────┬────────────┐
│ s   ┆ weight_mean ┆ weight_max │
│ --- ┆ ---         ┆ ---        │
│ str ┆ f64         ┆ f64        │
╞═════╪═════════════╪════════════╡
│ S1  ┆ 15.0        ┆ 19.0       │
│ S2  ┆ 14.5        ┆ 17.0       │
│ S3  ┆ 14.5        ┆ 17.0       │
│ S4  ┆ 15.0        ┆ 19.0       │
└─────┴─────────────┴────────────┘
```
Magic! 🪄 

## Scope

- Do you maintain a dataframe-consuming library?
- Is there a Polars function which you'd like Narwhals to have, which would make your work easier?

If, I'd love to hear from you!

**Note**: You might suspect that this is a secret ploy to infiltrate the Polars API everywhere.
Indeed, you may suspect that.

## Why "Narwhals"?

[Coz they are so awesome](https://youtu.be/ykwqXuMPsoc?si=A-i8LdR38teYsos4).

Thanks to [Olha Urdeichuk](https://www.fiverr.com/olhaurdeichuk) for the illustration!
