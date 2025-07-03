
<!-- README.md is generated from README.Rmd. Please edit that file -->

# lehdr

<!-- badges: start -->

[![CRAN_Status_Badge](https://www.r-pkg.org/badges/version/lehdr)](https://cran.r-project.org/package=lehdr)
[![metacran
downloads](https://cranlogs.r-pkg.org/badges/lehdr)](https://cran.r-project.org/package=lehdr)
[![R-CMD-check](https://github.com/jamgreen/lehdr/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/jamgreen/lehdr/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

**lehdr** (pronounced: *lee dur* like a metric *litre*) is an R package
that allows users to interface with the [Longitudinal and
Employer-Household Dynamics (LEHD)](https://lehd.ces.census.gov/)
Origin-Destination Employment Statistics (LODES) dataset returned as
dataframes. The package is continually in development and can be
installed via CRAN.

## Installation

You can install the released version of lehdr from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("lehdr")
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("pak")
pak::pak("jamgreen/lehdr")
```

## Usage

Load the library and optionally set the `lehdr_use_cache` option to save
downloaded files for reuse:

``` r
library(lehdr)

options(lehdr_use_cache = TRUE)
```

The `grab_lodes()` function can be used to download data for a specific
state, year, and LODES version. The LODES table type can be set to the
default origin-destination (“od”),residential association (“rac”), or
workplace association (“wac”).

For example, Oregon (`state = "or"`) 2020 (`year = 2020`) from LODES
version 8 (`version="LODES8"`, default), origin-destination
(`lodes_type = "od"`), primary jobs including private primary,
secondary, and Federal (`job_type = "JT01"`, default), as well as,
primary jobs across ages, earnings, and industry (`segment = "S000"`,
default), aggregated at the Census Tract level rather than the default
Census Block (`agg_geo = "tract"`).

``` r
or_od <- grab_lodes(
  state = "or",
  year = 2020,
  version = "LODES8",
  lodes_type = "od",
  job_type = "JT01",
  segment = "S000",
  state_part = "main",
  agg_geo = "tract"
)

head(or_od)
```

The package can be used to retrieve multiple states and years at the
same time by creating a vector or list. This second example pulls the
Oregon AND Rhode Island (`state = c("or", "ri")`) for 2013 and 2014
(`year = c(2013, 2014)` or `year = 2013:2014`).

``` r
or_ri_od <- grab_lodes(
  state = c("or", "ri"),
  year = c(2013, 2014),
  lodes_type = "od",
  job_type = "JT01",
  segment = "S000",
  state_part = "main",
  agg_geo = "tract"
)

head(or_ri_od)
```

Not all years are available for each state. To see all options for
`lodes_type`, `job_type`, and `segment` and the availability for each
state/year, please see the most recent LEHD Technical Document at
<https://lehd.ces.census.gov/data/lodes/LODES8>.

Set `geometry = TRUE` to download spatial data using the `{tigris}`
package and join to your output data. When `lodes_type = "rac"` or
`lodes_type = "wac"`, `grab_lodes()` now returns an `sf` data frame.

``` r
ri_rac_geo <- grab_lodes(
  state = "ri",
  year = 2020,
  lodes_type = "rac",
  agg_geo = "county",
  geometry = TRUE
)

plot(ri_rac_geo["C000"])
```

If `lodes_type = "od"` the returned data frame contains the geometry for
both the origin and destination in `sfc` class columns. These features
can be combined together as the following example shows:

``` r
or_od_geo <- grab_lodes(
  state = "or",
  year = 2020,
  lodes_type = "od",
  agg_geo = "county",
  geometry = TRUE,
  state_part = "main"
)

h_to_w_geometry <- lapply(
  seq(nrow(or_od_geo)),
  function(i) {
    sf::st_linestring(
      c(
        sf::st_centroid(or_od_geo[["h_geometry"]][[i]]),
        sf::st_centroid(or_od_geo[["w_geometry"]][[i]])
      )
    )
  }
)

h_to_w_lines <- sf::st_as_sfc(h_to_w_geometry, crs = 4269)

or_od_lines <- sf::st_set_geometry(
  or_od_geo[, c("w_county", "h_county", "h_geometry", "S000")],
  h_to_w_lines
)

multnomah_od_lines <- dplyr::filter(
  or_od_lines,
  w_county == "41051"
)

plot(multnomah_od_lines["S000"], reset = FALSE)
plot(or_od_geo["h_geometry"], lwd = 0.25, add = TRUE)
```

Using the optional `version` paramater, users can specify which LODES
version to use. Version 8 is default (`version="LODES8"`) is enumerated
at 2020 Census blocks. LODES7 (`version="LODES7"`) is enumerated at 2010
Census blocks, but ends in 2019. LODES5 (`version="LODES5"`) is
enumerated at 2000 Census blocks, but ends in 2009.

Other common uses might include retrieving Residential or Work Area
Characteristics (`lodes_type = "rac"` or `lodes_type = "wac"`
respectively), low income jobs (`segment = "SE01"`) or good producing
jobs (`segment = "SI01"`). Other common geographies might include
retrieving data at the Census Block level (`agg_geo = "block"`, not
necessary as it is default).

## Why lehdr?

The LODES dataset is frequently used by transportation and economic
development planners, regional economists, disaster managers and other
public servants in order to have a fine grained understanding of the
distribution of employment.

Such data is integral for regional travel demand models that help to
dictate transportation policy options, regional economists and economic
development planners interested in the spatial distribution of
particular kinds of work use the data to weigh different industrial or
workforce policy options.

Finally, as a Census product, the LODES data can be joined to Census
Decennial or American Community Survey data to help visualize the
interactions between different population groups and work.

In short, the LODES dataset is the only source of detailed geographic
information on employment for the country and should be more widely
available for researchers and analysts who work on regional development
issues.

## Future development

Currently, **lehdr** is designed to grab the LODES flat files
(origin-destination, workplace, and residential association files) and
includes an option to aggregate results to the Census tract level for
analysts who find the fuzzing at the block level too great.

## Acknowledgements

This package was developed by Jamaal Green, University of Pennsylvania;
Dillon Mahmoudi, University of Maryland Baltimore County; and Liming
Wang, Portland State University.

This package would not exist in its current format without the
inspiration of [Bob Rudis’s](https://rud.is/b/) [lodes
package](https://github.com/hrbrmstr/lodes)
