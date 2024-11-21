---
title: "Concurrent cdsapi requests using Python asynio"
date: 2024-10-10
categories: 
    - Blog
tags:
    - python
    - data
    - ECMWF
    - asyncio
    - parallel
---

The code below demonstrates how to make concurrent requests to the ECMWF API using the `cdsapi` library and Python's `asyncio` module. It defines an asynchronous function `year_data` to fetch data for a specific year and a `main` function to create and run tasks for multiple years concurrently. The `client.retrieve` method is run in an executor to avoid blocking the event loop, allowing multiple requests to be processed in parallel. The `asyncio.run(main())` call at the end executes the `main` function, initiating the concurrent data retrieval. 

```python
import asyncio
from functools import partial

import cdsapi

client = cdsapi.Client()


async def year_data(year):
    dataset = "reanalysis-era5-single-levels"
    request = {
        "product_type": ["reanalysis"],
        "year": year,
        "month": ["01"],
        "day": ["01"],
        "time": ["00:00"],
        "data_format": "netcdf",
        "download_format": "unarchived",
        "variable": ["100m_u_component_of_wind"],
        "area": [31, 34, 27, 38],
    }
    target = f"data_{year}.nc"

    #    client.retrieve(dataset, request, target)
    loop = asyncio.get_running_loop()
    return await loop.run_in_executor(
        None,
        partial(client.retrieve, dataset, request, target),
    )


async def main():
    tasks = []

    for yr in range(2010, 2025):
        tasks.append(asyncio.create_task(year_data(str(yr))))

    return await asyncio.gather(*tasks)


asyncio.run(main())
```