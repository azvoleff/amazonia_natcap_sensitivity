---
title: "Test cartodbr package for working with named and anonymous maps"
---

Run setup code

```r
library(leaflet)
library(cartodbr)

# Use Resilience Atlas CartoDB server
cdb_user <- 'grp'
cdb_domain <- 'grp.global.ssl.fastly.net'
cdb_token <- scan('grp_cdb_api_key.txt', 'character')
mapbox_token <- scan('grp_mapbox_api_key.txt', 'character')
```

# Test anonymous maps

Note that anonymous maps expire - so if the below map doesn't work it is because the code 
wasn't run recently enough. For web pages "named maps" (next section) should be 
used.


```r
# Setup sensitivity map as in the publication
sensitivity_ramp <- colorRampPalette(colors=c("#006636", "#F4E337", "#FF3234"))
sensitivity_css <- '#vegetation_sensitivity_clean_stretched {raster-opacity:1; raster-scaling:near; raster-colorizer-default-mode:linear; raster-colorizer-default-color: transparent; raster-colorizer-epsilon:0.41; raster-colorizer-stops: stop(-1, transparent) stop(0, #006636) stop(50, #F4E337) stop(100, #FF3234)}'
sensitivity_sql <- 'select * from vegetation_sensitivity_clean_stretched'
sensitivity_layer <- mapconfig_raster(sensitivity_sql, sensitivity_css)

# Make anonymous map on CartoDB server
id <- create_anon_map(sensitivity_layer, user=cdb_user, domain=cdb_domain, 
                      subdomain=FALSE)

# Draw the map on screen
anon_map <- leaflet() %>%
  addTiles(urlTemplate=paste0('https://api.tiles.mapbox.com/v4/cigrp.2ad62493/{z}/{x}/{y}.png?access_token=', 
                              mapbox_token),
           attribution="<a href='http://www.resilienceatlas.org'>Resilience Atlas</a>") %>%
  addTiles(urlTemplate=paste0('https://grp.global.ssl.fastly.net/user/grp/api/v1/map/', id, '/0/{z}/{x}/{y}.png'),
           attribution="<a href='http://go.nature.com/O2vNtU'>Seddon et al. 2016</a>") %>%
  addTiles(paste0(urlTemplate='https://api.tiles.mapbox.com/v4/cigrp.829fd2d8/{z}/{x}/{y}.png?access_token=', 
                  mapbox_token)) %>%
  setView(-55, -5, zoom=4) %>%
  addLegend("bottomright", pal=colorNumeric(sensitivity_ramp(10), seq(0, 100)), 
            values=seq(0, 100), title="Sensitivity", opacity=1)
anon_map
```

<!--html_preserve--><div id="htmlwidget-3683" style="width:504px;height:504px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-3683">{"x":{"calls":[{"method":"addTiles","args":["https://api.tiles.mapbox.com/v4/cigrp.2ad62493/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoiY2lncnAiLCJhIjoiYTQ5YzVmYTk4YzM0ZWM4OTU1ZjQxMWI5ZDNiNTQ5M2IifQ.SBgo9jJftBDx4c5gX4wm3g",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"<a href='http://www.resilienceatlas.org'>Resilience Atlas\u003c/a>"}]},{"method":"addTiles","args":["https://grp.global.ssl.fastly.net/user/grp/api/v1/map/9d2c02c4cbedc80f16ac15621ed902a0:1458421243375.99/0/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"<a href='http://go.nature.com/O2vNtU'>Seddon et al. 2016\u003c/a>"}]},{"method":"addTiles","args":["https://api.tiles.mapbox.com/v4/cigrp.829fd2d8/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoiY2lncnAiLCJhIjoiYTQ5YzVmYTk4YzM0ZWM4OTU1ZjQxMWI5ZDNiNTQ5M2IifQ.SBgo9jJftBDx4c5gX4wm3g",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false}]},{"method":"addLegend","args":[{"colors":["#006636 , #006636 0%, #629736 20%, #C2CA36 40%, #F6C036 60%, #FB7935 80%, #FF3234 100%, #FF3234 "],"labels":["0","20","40","60","80","100"],"na_color":null,"na_label":"NA","opacity":1,"position":"bottomright","type":"numeric","title":"Sensitivity","extra":{"p_1":0,"p_n":1},"layerId":null,"className":"info legend"}]}],"setView":[[-5,-55],4,[]]},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

# Test named maps


```r
# Create named map on server
resp <- create_named_map(name='sensitivity_test', layers=sensitivity_layer, 
                         user=cdb_user, domain=cdb_domain, api_key=cdb_token, 
                         subdomain=FALSE, overwrite=TRUE)

# Instantiate named map so tiles can be drawn from the server
id <- inst_named_map(name='sensitivity_test', user=cdb_user, 
                     domain=cdb_domain, api_key=cdb_token,  subdomain=FALSE, 
                     overwrite=TRUE)

# Draw the map on screen
sens_map <- leaflet() %>%
  addTiles(urlTemplate=paste0('https://api.tiles.mapbox.com/v4/cigrp.2ad62493/{z}/{x}/{y}.png?access_token=', 
                              mapbox_token),
           attribution="<a href='http://www.resilienceatlas.org'>Resilience Atlas</a>") %>%
  addTiles(urlTemplate=paste0('https://grp.global.ssl.fastly.net/user/grp/api/v1/map/', id, '/all/{z}/{x}/{y}.png'),
           attribution="<a href='http://go.nature.com/O2vNtU'>Seddon et al. 2016</a>") %>%
  setView(-55, -5, zoom=4) %>%
  addLegend("bottomright", pal=colorNumeric(sensitivity_ramp(10), seq(0,100)), 
            values=seq(0,100, 10), title="Sensitivity", opacity=1)
sens_map
```

<!--html_preserve--><div id="htmlwidget-8978" style="width:504px;height:504px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-8978">{"x":{"calls":[{"method":"addTiles","args":["https://api.tiles.mapbox.com/v4/cigrp.2ad62493/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoiY2lncnAiLCJhIjoiYTQ5YzVmYTk4YzM0ZWM4OTU1ZjQxMWI5ZDNiNTQ5M2IifQ.SBgo9jJftBDx4c5gX4wm3g",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"<a href='http://www.resilienceatlas.org'>Resilience Atlas\u003c/a>"}]},{"method":"addTiles","args":["https://grp.global.ssl.fastly.net/user/grp/api/v1/map/grp@828d48ae@3e34095aa509baaeb1133955ccb12c2f:1458421243375.99/all/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"<a href='http://go.nature.com/O2vNtU'>Seddon et al. 2016\u003c/a>"}]},{"method":"addLegend","args":[{"colors":["#006636 , #006636 0%, #629736 20%, #C2CA36 40%, #F6C036 60%, #FB7935 80%, #FF3234 100%, #FF3234 "],"labels":["0","20","40","60","80","100"],"na_color":null,"na_label":"NA","opacity":1,"position":"bottomright","type":"numeric","title":"Sensitivity","extra":{"p_1":0,"p_n":1},"layerId":null,"className":"info legend"}]}],"setView":[[-5,-55],4,[]]},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

# Test map overlays

Overlay the green/yellow/red Amazon layer


```r
amazon_css <- '#green_yellow_red_amazoniay {
    raster-opacity:1;
    raster-scaling:near;
    raster-colorizer-default-mode:discrete;
    raster-colorizer-default-color: transparent;
    raster-colorizer-epsilon:0.41;
    raster-colorizer-stops: stop(-1, transparent) stop(1, #FF4D4D) stop(2, #FFFFBE) stop(3, #73B273)
}'
amazon_sql <- 'select * from green_yellow_red_amazonia'
amazon_layer <- mapconfig_raster(amazon_sql, amazon_css)
# Create amazon named map
amz_id <- create_named_map(name='amazon_green_yellow_red', layers=amazon_layer, 
                           user=cdb_user, domain=cdb_domain, 
                           api_key=cdb_token, subdomain=FALSE, overwrite=TRUE)

# Instantiate amazon named map
id <- inst_named_map(name='amazon_green_yellow_red',  user=cdb_user,  
                            domain=cdb_domain, api_key=cdb_token, 
                            subdomain=FALSE)
overlaid <- addTiles(sens_map, 
    urlTemplate=paste0('https://grp.global.ssl.fastly.net/user/grp/api/v1/map/', 
                       id, '/all/{z}/{x}/{y}.png'),
              attribution="<a href='http://www.conservation.org'>Conservation International</a>") %>%
    addLegend("topright", title='Green / yellow / red Amazon',
              opacity=1,
              colors=c("#FF4D4D", "#FFFFBE", "#73B273"), 
              labels=c("Agriculture, urban, and deforested areas",
                       "Intact habitat that is not formally protected",
                       "Protected areas and indigenous lands"))
overlaid
```

<!--html_preserve--><div id="htmlwidget-8566" style="width:504px;height:504px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-8566">{"x":{"calls":[{"method":"addTiles","args":["https://api.tiles.mapbox.com/v4/cigrp.2ad62493/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoiY2lncnAiLCJhIjoiYTQ5YzVmYTk4YzM0ZWM4OTU1ZjQxMWI5ZDNiNTQ5M2IifQ.SBgo9jJftBDx4c5gX4wm3g",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"<a href='http://www.resilienceatlas.org'>Resilience Atlas\u003c/a>"}]},{"method":"addTiles","args":["https://grp.global.ssl.fastly.net/user/grp/api/v1/map/grp@828d48ae@3e34095aa509baaeb1133955ccb12c2f:1458421243375.99/all/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"<a href='http://go.nature.com/O2vNtU'>Seddon et al. 2016\u003c/a>"}]},{"method":"addLegend","args":[{"colors":["#006636 , #006636 0%, #629736 20%, #C2CA36 40%, #F6C036 60%, #FB7935 80%, #FF3234 100%, #FF3234 "],"labels":["0","20","40","60","80","100"],"na_color":null,"na_label":"NA","opacity":1,"position":"bottomright","type":"numeric","title":"Sensitivity","extra":{"p_1":0,"p_n":1},"layerId":null,"className":"info legend"}]},{"method":"addTiles","args":["https://grp.global.ssl.fastly.net/user/grp/api/v1/map/grp@f6bf30d1@e638e355066681c66f96c4a1ee6ae026:1458771435047.94/all/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"<a href='http://www.conservation.org'>Conservation International\u003c/a>"}]},{"method":"addLegend","args":[{"colors":["#FF4D4D","#FFFFBE","#73B273"],"labels":["Agriculture, urban, and deforested areas","Intact habitat that is not formally protected","Protected areas and indigenous lands"],"na_color":null,"na_label":"NA","opacity":1,"position":"topright","type":"unknown","title":"Green / yellow / red Amazon","extra":null,"layerId":null,"className":"info legend"}]}],"setView":[[-5,-55],4,[]]},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
