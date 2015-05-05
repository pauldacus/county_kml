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

Example data:
```
State  | County                 | id   | parent
-------|------------------------|------|-------
 CO    | Denver                 |  336 |       
 CO    | Denver                 |  337 |       
 CO    | Denver                 |  338 |    337
 CO    | Denver                 |  339 |    337
 CO    | Denver                 |  340 |    337
 CO    | Denver                 |  341 |    337
 CO    | Denver                 |  342 |    337
 CO    | Denver                 |  343 |    337
 CO    | Denver                 |  344 |    337
 CO    | Denver                 |  345 |    337
 CO    | Denver                 |  346 |    337
 CO    | Denver                 |  347 |    337
 CO    | Denver                 |  348 |    337
 CO    | Denver                 |  349 |    337
```
This is for ~surprise~, Denver county, Colorado. The columns are state, county, id, parent (coordinates column omitted for readability).
You can see that rows with id's 336 & 337 are separate Polygons (eg: they have more than one repeating {state,county} combination, {'CO','Denver'} in this case), 
which means the Placemark for Denver county will have a MultiGeometry tag which contains 2 Polygon tags.
You can also see that all the rows with 337 in the parent column will have their coordinates in innerBoundaryIs tags.

The tags for the above data would look roughly like this:

```
    <Placemark>
      <name>Denver, CO</name>
      <description><![CDATA[<div>Denver, CO</div>
]]></description>
      <styleUrl>Red</styleUrl>
      <MultiGeometry>
        <Polygon>
          <outerBoundaryIs>
            <LinearRing>
              <coordinates>-104.93413,39.70018 -104.93288,39.70021 -104.93283,39.70021 -104.93171,39.70021 -104.93167,39.69917 -104.93165,39.6984 -104.93261,39.69844 -104.93276,39.69842 -104.93288,39.69841 -104.9339,39.69848 -104.9339,39.69851 -104.93391,39.6986 -104.93392,39.69865 -104.93405,39.69966 -104.9341,39.69994 -104.93413,39.70011 -104.93413,39.70014</coordinates>
            </LinearRing>
          </outerBoundaryIs>
        </Polygon>
        <Polygon>
          <outerBoundaryIs>
            <LinearRing>
              <coordinates>-104.75528,39.84534 -104.69293,39.91418 -104.60578,39.87514 -104.65912,39.81403 -104.76268,39.80378 -104.73447,39.79844 -104.7534,39.76949 -104.78183,39.77616 -104.82822,39.7698 -104.85388,39.76849 -104.86457,39.7549 -104.88464,39.74021 -104.88465,39.74016 -104.88476,39.73213 -104.87534,39.72013 -104.86898,39.7029 -104.8742,39.69647 -104.89737,39.6963 -104.9,39.68933 -104.89363,39.67869 -104.90753,39.68037 -104.89891,39.66762 -104.88413,39.66749 -104.85615,39.66039 -104.85675,39.65346 -104.88512,39.63896 -104.91334,39.62421 -104.91333,39.62588 -104.93189,39.6503 -104.9559,39.653 -104.96762,39.66041 -104.97958,39.6677 -105.00437,39.66631 -105.00864,39.67767 -105.0158,39.66213 -105.03463,39.65143 -105.03344,39.63356 -105.02542,39.62955 -105.04014,39.62813 -105.05341,39.63159 -105.05341,39.63164 -105.05341,39.63202 -105.0533,39.63483 -105.05333,39.63493 -105.05322,39.63568 -105.05337,39.63668 -105.05336,39.63671 -105.05324,39.63699 -105.05333,39.63889 -105.05539,39.63889 -105.05649,39.63889 -105.05884,39.63889 -105.05896,39.63889 -105.05943,39.63881 -105.059,39.63818 -105.0592,39.6371 -105.05929,39.63668 -105.05952,39.63549 -105.05954,39.63508 -105.05946,39.63466 -105.0593,39.63433 -105.05876,39.63365 -105.05711,39.63178 -105.05654,39.63135 -105.0555,39.6307 -105.05432,39.63001 -105.05346,39.62953 -105.05333,39.62798 -105.05344,39.6279 -105.05378,39.62763 -105.0539,39.62754 -105.05401,39.62745 -105.05433,39.62719 -105.05444,39.6271 -105.05465,39.62694 -105.05526,39.62645 -105.05547,39.62629 -105.05552,39.62624 -105.05567,39.62612 -105.05573,39.62608 -105.05584,39.62599 -105.05584,39.62526 -105.05584,39.62419 -105.05431,39.62432 -105.05344,39.62439 -105.05344,39.62142 -105.056,39.62144 -105.0914,39.61445 -105.08264,39.61857 -105.06574,39.62059 -105.08174,39.62758 -105.09446,39.62419 -105.10754,39.62422 -105.09555,39.63512 -105.08169,39.63706 -105.07429,39.64427 -105.05488,39.65189 -105.06392,39.64957 -105.07443,39.65951 -105.07916,39.66936 -105.06648,39.66778 -105.05487,39.66778 -105.06273,39.67498 -105.05319,39.68635 -105.05325,39.70454 -105.05256,39.72552 -105.05498,39.73373 -105.05321,39.74358 -105.05323,39.7657 -105.0532,39.78285 -105.06245,39.7876 -105.05508,39.79106 -105.05325,39.79106 -105.05322,39.79106 -105.02519,39.79111 -105.02184,39.79299 -104.99905,39.79113 -104.99319,39.79208 -104.97954,39.79109 -104.97219,39.79828 -104.95945,39.79178 -104.94051,39.79108 -104.9125,39.78382 -104.90339,39.79197 -104.88326,39.81294 -104.84231,39.79816 -104.79098,39.80587</coordinates>
            </LinearRing>
          </outerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-104.90403,39.62412 -104.90276,39.62411 -104.90311,39.62579 -104.90315,39.62584 -104.90375,39.62669 -104.90401,39.62609 -104.90405,39.6255 -104.90405,39.62544</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-104.89916,39.62451 -104.89915,39.62413 -104.89894,39.62412 -104.89868,39.62414 -104.89869,39.62452 -104.89893,39.62449</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-104.89946,39.62444 -104.89946,39.62452 -104.90014,39.62452 -104.90014,39.62413 -104.89973,39.62413 -104.89947,39.62413</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-105.08113,39.6348 -105.08103,39.63509 -105.0811,39.63513 -105.08091,39.6356 -105.0816,39.63561 -105.08153,39.63553 -105.08161,39.63548 -105.08177,39.63535 -105.08188,39.63525 -105.08196,39.63517 -105.08179,39.63512</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-105.06256,39.65298 -105.06256,39.65299 -105.06275,39.65299 -105.06335,39.65299 -105.06354,39.65299 -105.06354,39.65298 -105.06354,39.65267 -105.06354,39.65239 -105.06288,39.65239 -105.0628,39.65239 -105.06257,39.65238 -105.06257,39.6525 -105.06256,39.65286</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-104.92205,39.6633 -104.92182,39.66249 -104.91905,39.66032 -104.91814,39.65989 -104.91707,39.65904 -104.91662,39.65838 -104.91565,39.65688 -104.91593,39.65649 -104.91583,39.65641 -104.91282,39.65875 -104.91281,39.66283 -104.91628,39.66326 -104.91659,39.66759 -104.91756,39.67246 -104.91978,39.674 -104.92226,39.67557 -104.92312,39.67664 -104.92449,39.67611 -104.9253,39.67665 -104.92781,39.67495 -104.92851,39.67267 -104.92486,39.66885 -104.92263,39.66805 -104.92071,39.66758 -104.92004,39.66746 -104.92214,39.66562</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-104.93169,39.66939 -104.93163,39.66757 -104.93162,39.66747 -104.93044,39.66747 -104.93043,39.66756 -104.9305,39.66848 -104.92943,39.66848 -104.92954,39.66938 -104.92931,39.66938 -104.92856,39.66938 -104.92804,39.66909 -104.92704,39.66899 -104.92721,39.66928 -104.92733,39.66948 -104.92793,39.67033 -104.92811,39.67056 -104.92943,39.67239 -104.93003,39.67244 -104.93023,39.67246 -104.93032,39.67247 -104.93166,39.67301 -104.93422,39.67339 -104.93418,39.67109 -104.93291,39.67089 -104.9329,39.67055 -104.93169,39.67056</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-104.9082,39.68207 -104.90961,39.68207 -104.90973,39.68207 -104.91103,39.68207 -104.91104,39.67989 -104.90972,39.67992 -104.90846,39.67992 -104.90832,39.6799 -104.90829,39.67988 -104.90831,39.68012 -104.90822,39.68031 -104.90831,39.68031 -104.90821,39.68206</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-104.89813,39.68279 -104.89815,39.68307 -104.89819,39.68354 -104.89833,39.68539 -104.89833,39.68559 -104.89873,39.68559 -104.90173,39.68559 -104.89858,39.68314</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-104.93621,39.69478 -104.93621,39.69482 -104.93643,39.69839 -104.93391,39.69792 -104.9324,39.6966 -104.92858,39.69663 -104.9279,39.69843 -104.93036,39.70022 -104.93172,39.70029 -104.93397,39.703 -104.93284,39.7034 -104.93026,39.70384 -104.92692,39.7036 -104.92696,39.70656 -104.9296,39.70672 -104.92699,39.70805 -104.92953,39.70858 -104.93167,39.7111 -104.93516,39.71104 -104.94074,39.70734 -104.94069,39.7049 -104.94073,39.70185 -104.94075,39.69796 -104.93863,39.6966 -104.94062,39.69549 -104.93963,39.69559</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-104.92925,39.70243 -104.92947,39.70211 -104.92953,39.70199 -104.92953,39.70129 -104.92877,39.70129 -104.92813,39.70129 -104.92807,39.70141 -104.92855,39.70203 -104.92907,39.7019</coordinates>
            </LinearRing>
          </innerBoundaryIs>
          <innerBoundaryIs>
            <LinearRing>
              <coordinates>-105.06446,39.65333 -105.06437,39.65333 -105.06434,39.65333 -105.06427,39.65333 -105.06406,39.65333 -105.064,39.65333 -105.06399,39.65334 -105.06399,39.65338 -105.06399,39.65339 -105.06399,39.65341 -105.06399,39.65346 -105.06399,39.65348 -105.06399,39.6535 -105.06408,39.6535 -105.06441,39.6535 -105.0645,39.6535 -105.0645,39.65348 -105.0645,39.65345 -105.06449,39.65336 -105.06449,39.65333</coordinates>
            </LinearRing>
          </innerBoundaryIs>
        </Polygon>
      </MultiGeometry>
    </Placemark>
```
