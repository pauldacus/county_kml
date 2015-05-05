# county_kml
Postgres dump of KML coordinates for all US counties.
Includes id, parent, county name, state, and coordinates columns
When coding, if there are multiple {state, county(s)} combos with the same value, you will need to wrap them in a "MultiGeometry" KML tag,
this mean there are multiple Polygon tags inside of a Placename which will need to go inside separate Polygon tags.
If a row has a reference to a parent, that parent id row's coordinates should be wrapped in a "outerBoundaryIs" KML tag, and the 
row(s) referencing the parent should be wrapped in a "innerBoundaryIs" KML tag, all of which should be inside a Polygon tag.
This is how I do it for MultiGeometry tags ("c" is a county object in Rails):
```
              if c.county_kml.size > 1
                b.MultiGeometry {
                c.county_kml.each do |k|
                  b.Polygon {
                    b.outerBoundaryIs {
                      b.LinearRing {
                        b.coordinates "#{k.coordinates}"  
                      }
                    }
                    inners = c.county_kml.where(parent: k.id)
                    if inners && inners.size > 0
                      inners.each do |i|
                        b.innerBoundaryIs {
                          b.LinearRing {
                            b.coordinates "#{i.coordinates}"
                          }
                        }
                      end
                    end
                  }
                end
```
This is an easy way to preserve all the KML features for MultiGeometry, innerBoundaryIs, and outerBoundaryIs features for KML Polygons.
