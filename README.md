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
>>> RE(wright_plans.grid_scan_wp([sensor], motor1, -1, 1, 11, "mm", motor2, -1, 1, 11, "mm")
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

Constants provide a tool for driving auxilliary hardware with algebraic expressions relating to the position of scanning hardware.

For example, a scan involving two tunable light sources, `w1` and `w2`, might have the monochromator (`wm`) track the algebraic sum `2 * w1 + w2`.
This is an especially important capability for Coherent Multidimensional Spectroscopy as described in [Neff-Mallon and Wright 2017](https://pubs.acs.org/doi/abs/10.1021/acs.analchem.7b02917).

Constants in `wright_plans` allow for arbitray linear combinations of other hardware with compatible units.
Constants are units aware and do addition/multiplication in the specified units.

Constants are defined using the `Constant` class with `ConstantTerm` elements.

An example of the scan described above:

```python
>>> import yaqc_bluesky
>>> w1 = yaqc_bluesky.Device(39000)
>>> w2 = yaqc_bluesky.Device(39001)
>>> wm = yaqc_bluesky.Device(39002)
>>> sensor = yaqc_bluesky.Device(39003)
>>>
>>> import bluesky
>>> RE = bluesky.RunEngine()
>>>
>>> from wright_plans import Constant, ConstantTerm, gridscan_wp
>>> constants = {wm: Constant("wn", [
...     ConstantTerm(2, w1),
...     ConstantTerm(1, w2),
... ])}
>>> RE(grid_scan_wp([sensor], motor1, -1, 1, 11, "mm", motor2, -1, 1, 11, "mm", constants=constants)
```

| constant dictionary | description |
| --- | --- |
| `Constant("nm", [ConstantTerm(1300, None)])` | remain static at `1300 nm` |
| `Constant("ps", [ConstantTerm(1, d2)])}` | track `d2` |
| `Constant("wn", [ConstantTerm(3, w1)])}` | triple `w1` (in wn) |
| `Constant("wn", [ConstantTerm(3, w1), ConstantTerm(-800, None)])}` | triple `w1` and subtract 800 wn |
| `Constant("wn", [ConstantTerm(1, w1), ConstantTerm(1, w2), ConstantTerm(1, w3)])}` | Constant for Triple Sum Frequency |


Each entry in the constants dictionary describes the tracking behavior of one Movable, multiple devices can be set to track in a single scan by passing as a dictionary `{motor1: constant1, motor2: constant2}`.


### plans

Wright plans provides provides the following plans, plan signatures can be found by their docstrings:

| plan name | description |
| --- | --- |
| `scan_wp` | "Inner product" scan of multiple motors with evenly spaced points |
| `list_scan_wp` | "Inner product" scan of multiple motors with listed positions |
| `grid_scan_wp` | "Outer product" scan of multiple axes with evenly spaced points |
| `list_grid_scan_wp` | "Outer product" scan of multiple axes with listed positions |
| `scan_nd_wp` | Generic N-Dimensional scan using Cycler objects  |
| `rel_scan_wp` | "Inner product" scan of multiple motors with evenly spaced points relative to initial conditions |
| `rel_list_scan_wp` | "Inner product" scan of multiple motors with listed positions relative to initial conditions |
| `rel_grid_scan_wp` | "Outer product" scan of multiple axes with evenly spaced points relative to initial conditions |
| `rel_list_grid_scan_wp` | "Outer product" scan of multiple axes with listed positions relative to initial conditions |
| `motortune` | Scan individual motors of an OPA |
| `tune_test` | Validate the output of an OPA |
| `tune_intensity` | Tune a motor of an opa to maximize intensity |
| `tune_setpoint` | Tune a motor of an opa to optimize output color |
| `tune_holistic` | Tune two motors in an opa together to optimize color and intensity |
