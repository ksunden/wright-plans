# wright-plans 

[![PyPI](https://img.shields.io/pypi/v/wright-plans)](https://pypi.org/project/wright-plans)
[![Conda](https://img.shields.io/conda/vn/conda-forge/wright-plans)](https://anaconda.org/conda-forge/wright-plans)
[![black](https://img.shields.io/badge/code--style-black-black)](https://black.readthedocs.io/)

A set of [Bluesky Plans](https://blueskyproject.io/bluesky/plans.html) with a focus on experimental orchestration within the Wright Group.

## installation

Install the latest released version from PyPI:

```bash
$ python3 -m pip install wright-plans
```

conda-forge coming soon!

Use [flit](https://flit.readthedocs.io/) to install from source.

```
$ git clone https://github.com/wright-group/wright-plans.git
$ cd wright-plans
$ flit install -s
```

## usage

wright-plans extends Bluesky in three major ways:
- support for [WrightTools-style units](http://wright.tools/en/stable/units.html)
- support for "constants", algebraic expressions for hardware relationships as described below
- support for specialized tuning procecdures unique to Wright Group experiments

The plans defined by this package all end in `_wp` so that their names never collide with plans defined within Bluesky itself or other Bluesky plan providers.
Feel free to mix these plans into an existing environment if that makes sense.
Refer to the [Bluesky documentation](https://blueskyproject.io/bluesky/) for documentation on how to utilize these plans within a simple Python script or notebook environment.
A minimal example follows:

```python
>>> import yaqc_bluesky
>>> motor1 = yaqc_bluesky.Device(39000)
>>> motor2 = yaqc_bluesky.Device(39001)
>>> sensor = yaqc_bluesky.Device(39002)
>>> import bluesky
>>> RE = bluesky.RunEngine()
>>> import wright_plans
>>> RE(wright_plans.gridscan_wp([sensor], motor1, -1, 1, 11, "mm", motor2, -1, 1, 11, "mm")
```

For usage within a bluesky-queueserver, consider [bluesky-in-a-box](https://github.com/wright-group/bluesky-in-a-box).

Note that the runs generated by these plans are guaranteed to be compatable with WrightTools via the `from_databroker` method.

```python
>>> import databroker
>>> mongo = databroker.catalog["mongo"]
>>> run = mongo[-1]  # get most recent run from catalog
>>> wt.data.from_databroker(mongo)
<WrightTools.Data>
```

Plans from the broader Bluesky ecosystem may or may not be compatible with `from_databroker`.
Bluesky is capable of arbitrary orchestration, and some experiments violate the core assumptions of WrightTools.
Still, the rest of the Bluesky data processing ecosystem is avaliable for such plans.

### constants

### `grid_scan_wp`

documentation TODO

### `scan_nd_wp`

documentaiton TODO

### `motortune_wp`

documentation TODO

### `run_holistic_wp`

documentation TODO

### `run_intensity_wp`

documentation TODO

### `run_setpoint_wp`

documentation TODO

### `run_tune_test_wp`

documentation TODO


