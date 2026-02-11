# edh-learning

This repo is a Jupyter notebook-based learning resource for the [Earth Data Hub (EDH)](https://earthdatahub.destine.eu/), a cloud platform for accessing climate and Earth observation Zarr data via Python and Xarray.

**Key Purpose**: Educational notebooks demonstrating best practices for data access, authentication, transformation, and visualization workflows for EDH climate and geospatial datasets.

**Target Users**: Data scientists and developers learning to work with the EDH Zarr datasets.

## Setup

Install the `uv` package manager, e.g. `pip install uv` or `conda install uv`.

Clone this repo, enter the repo folder and start Jupyter Lab in an isolated
environment without messing up your existing Python installations.

```bash
git clone git@github.com:bopen/edh-learning.git
cd edh-learning
uv run --frozen jupyter lab
```

### Optional interactive map support

You can start Jupyter Lab with support for interactive data exploration and map
integration (at the cost of a much more complex installation) with:

```bash
uv run --frozen --extra maps jupyter lab
```
