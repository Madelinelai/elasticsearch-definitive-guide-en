[[geohash-cell-filter]]
=== geohash_cell Filter

The `geohash_cell` filter simply translates a `lat/lon` location((("geohash_cell filter")))((("filters", "geohash_cell"))) into a
geohash with the specified precision and finds all locations that contain
that geohash--a very efficient filter indeed.

[source,json]
----------------------------
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geohash_cell": {
          "location": {
            "lat":  40.718,
            "lon": -73.983
          },
          "precision": "2km" <1>
        }
      }
    }
  }
}
----------------------------
<1> The `precision` cannot be more precise than that specified in the
    `geohash_precision` mapping.

This filter translates the `lat/lon` point into a geohash of the appropriate
length--in this example `dr5rsk`&#x2014;and looks for all locations that contain
that exact term.

However, the filter as written in the preceding example may not return all restaurants within 5km
of the specified point.  Remember that a geohash is just a rectangle, and the
point may fall anywhere within that rectangle.  If the point happens to fall
near the edge of a geohash cell, the filter may well exclude any
restaurants in the adjacent cell.

To fix that, we can tell the filter to include the neigboring cells, by
setting `neighbors` to((("neighbors setting (geohash_cell)"))) `true`:

[source,json]
----------------------------
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geohash_cell": {
          "location": {
            "lat":  40.718,
            "lon": -73.983
          },
          "neighbors": true, <1>
          "precision": "2km"
        }
      }
    }
  }
}
----------------------------

<1> This filter will look for the resolved geohash and all surrounding
    geohashes.

Clearly, looking for a geohash with precision `2km` plus all the neighboring
cells results in quite a large search area.  This filter is not built for
accuracy, but it is very efficient and can be used as a prefiltering step
before applying a more accurate geo-filter.

TIP: Specifying the `precision` as a distance can be misleading. A `precision`
of `2km` is converted to a geohash of length 6, which actually has dimensions
of about 1.2km x 0.6km.  You may find it more understandable to specify an
actual length such as `5` or `6`.

The other advantage that this filter has over a `geo_bounding_box` filter is
that it supports multiple locations per field.((("latitude/longitude pairs", "multiple lat/lon points per field, geohash_cell")))  The `lat_lon` option that we
discussed in <<optimize-bounding-box>> is efficient, but only when there
is a single `lat/lon` point per field.


