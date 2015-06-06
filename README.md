# Geo

A collection of GIS functions. Handles conversions to and from WKT, WKB, and GeoJSON for the following geometries:

* Point
* LineString
* Polygon
* MultiPoint
* MulitLineString
* MultiPolygon
* GeometryCollection


Also includes a Postgrex extension for PostGIS data types

```elixir
defp deps do
  [{:geo, "~> 0.12.0"}]
end
```

[Documentation](http://hexdocs.pm/geo)


## Examples


* Encode and decode WKT and EWKT

  ```elixir
  iex(1)> point = Geo.WKT.decode("POINT(30 -90)")
  %Geo.Point{ coordinates: {30, -90}, srid: nil}

  iex(2)> Geo.WKT.encode(point)
  "POINT(30 -90)"

  iex(3)> point = Geo.WKT.decode("SRID=4326;POINT(30 -90)")
  %Geo.Point{coordinates: {30, -90}, srid: 4326}
  ```


* Encode and decode WKB and EWKB

  ```elixir
  iex(1)> point = Geo.WKB.decode("0101000000000000000000F03F000000000000F03F")
  %Geo.Point{ coordinates: {1.0, 1.0}, srid: nil }

  iex(2)> Geo.WKB.encode(point)
  "00000000013FF00000000000003FF0000000000000"

  iex(3)> point = Geo.WKB.decode("0101000020E61000009EFB613A637B4240CF2C0950D3735EC0")
  %Geo.Point{ coordinates: {36.9639657, -121.8097725}, srid: 4326 }

  iex(4)> Geo.WKB.encode(point)
  "0020000001000010E640427B633A61FB9EC05E73D350092CCF"
  ```

* Encode and decode GeoJSON

  ```elixir
  iex(1)> point = Geo.JSON.decode("{ \"type\": \"Point\", \"coordinates\": [100.0, 0.0] }")
  %Geo.Point{ coordinates: {100.0, 0.0}, srid: nil }

  iex(2)> Geo.JSON.encode(point)
  "{\"type\":\"Point\",\"coordinates\":[100.0,0.0]}"
  ```

* A Postgrex Extension for PostGIS data types

  ```elixir
  opts = [hostname: "localhost", username: "postgres", database: "geo_postgrex_test",
  extensions: [{Geo.PostGIS.Extension, library: Geo}] ]

  [hostname: "localhost", username: "postgres", database: "geo_postgrex_test",
   extensions: [{Geo.PostGIS.Extension, library: Geo}]]

  {:ok, pid} = Postgrex.Connection.start_link(opts)
  {:ok, #PID<0.115.0>}

  geo = %Geo.Point{coordinates: {30, -90}, srid: 4326}
  %Geo.Point{coordinates: {30, -90}, srid: 4326}
  
  {:ok, _} = Postgrex.Connection.query(pid, "CREATE TABLE point_test (id int, geom geometry(Point, 4326))")
  {:ok, %Postgrex.Result{columns: nil, command: :create_table, num_rows: 0, rows: nil}}
  
  {:ok, _} = Postgrex.Connection.query(pid, "INSERT INTO point_test VALUES ($1, $2)", [42, geo])
  {:ok, %Postgrex.Result{columns: nil, command: :insert, num_rows: 1, rows: nil}}
  
  Postgrex.Connection.query(pid, "SELECT * FROM point_test")
  {:ok, %Postgrex.Result{columns: ["id", "geom"], command: :select, num_rows: 1,
  rows: [{42, %Geo.Point{coordinates: {30.0, -90.0}, srid: 4326 }}]}}
  ```
  
* Can now be used with Ecto as well

  ```elixir

  #Add extensions to your repo config
  config :thanks, Repo,
    database: "geo_postgrex_test",
    username: "postgres",
    password: "postgres",
    hostname: "localhost",
    adapter: Ecto.Adapters.Postgres,
    extensions: [{Geo.PostGIS, library: Geo}]


  #Create a model
  defmodule Test do
    use Ecto.Model

    schema "test" do
      field :name,           :string
      field :geom,           Geo.Point
    end
  end

  #Geometry columns can be created in migrations too
  defmodule Repo.Migrations.Init do
    use Ecto.Migration

    def up do
      create table(:test) do
        add :name,     :string
        add :geom,     :geometry
      end
    end

    def down do
      drop table(:test)
    end
  end
  ```
  
  Be sure to enable the Postgis extension if you haven't already done so:
  
  ```elixir
  
  defmodule MyApp.Repo.Migrations.EnablePostgis do
    use Ecto.Migration

    def up do
      execute "CREATE EXTENSION IF NOT EXISTS postgis"
    end

    def down do
      execute "DROP EXTENSION IF EXISTS postgis"
    end
  end
