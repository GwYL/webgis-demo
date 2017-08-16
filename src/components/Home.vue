<template>
	<div>
		<div id="header">WebGIS Demo</div>
		<div id="map-info"></div>
		<div id="minMap">
			<div id="extentMap"></div>
		</div>
	</div>
</template>
<script>
	import * as esriLoader from 'esri-loader'

	export default {
		name: "overall",
		data() {
			return {

			}
		},
		mounted() {    
	    if (!esriLoader.isLoaded()) {
	      esriLoader.bootstrap((err) => {
	        if (err) {
	          console.error(err);
	        }
	        this.createMap();
	      }, {
	        url: 'https://js.arcgis.com/4.4/'
	      });
	    } else {
	      this.createMap();
	    }
	  },
		methods: {
			createMap() {
				esriLoader.dojoRequire([
		      "esri/Map",
		      "esri/views/MapView",
		      "esri/widgets/Search",
		      "esri/tasks/Locator",
		      "esri/core/watchUtils",
		      "dojo/dom",
		      "dojo/promise/all",
		      "dojo/domReady!"
				], (Map, MapView, Search, Locator, watchUtils, dom, all) => {

					// 主地图
		      var map = new Map({
		        basemap: "streets-navigation-vector"
		      });

		      var view = new MapView({
		        container: "map-info",
		        map: map,
		        center: [-116.3031, 43.6088],
		        zoom: 12
		      });

		      // 鹰眼
		      var minMap = new Map({
		      	basemap: "streets-navigation-vector"
		      })

		      var minView = new MapView({
		      	container: "minMap",
		      	map: minMap
		      })

		      minView.ui.components = []

		      var extentMap = dom.byId("extentMap")

		      view.then(function() {
		      	view.goTo({
		      		center: [-116.3031, 43.6088],
		      		scale: 200000,
		      		heading: 35,
		      		tilt: 60
		      	}, {
		      		animate: false,
		      		duration: 100000
		      	})
		      })

		      minView.then(function() {
		      	view.watch("extent", updateOverviewExtent)
		      	minView.watch("extent", updateOverviewExtent)

		      	watchUtils.when(view, "stationary", updateOverview)

		      	function updateOverview() {
		      		minView.goTo({
		      			center: view.center,
		      			scale: view.scale * 2 * Math.max(view.width / minView.width, view.height / minView.height)
		      		})
		      	}

		      	function updateOverviewExtent() {
		      		var extent = view.extent

		      		var bottomLeft = minView.toScreen(extent.xmin, extent.ymin)
		      		var topRight = minView.toScreen(extent.xmax, extent.ymax)

		      		extentMap.style.top = topRight.y + "px"
		      		extentMap.style.left = bottomLeft.x + "px"

		      		extentMap.style.height = (bottomLeft.y - topRight.y) + "px"
		      		extentMap.style.width = (topRight.x - bottomLeft.x) + "px"
		      	}
		      })

		      const searchWidget = new Search({
		      	view: view
		      })

		      view.ui.add(searchWidget, {
		      	position: "top-left",
		      	index: 0
		      })

		      var locatorTask = new Locator({
		        url: "https://geocode.arcgis.com/arcgis/rest/services/World/GeocodeServer"
		      })

		      view.on("click", function(event) {
            event.stopPropagation(); 

            var lat = Math.round(event.mapPoint.latitude * 1000) / 1000;
            var lon = Math.round(event.mapPoint.longitude * 1000) / 1000;

            view.popup.open({
              title: "Reverse geocode: [" + lon + ", " + lat + "]",
              location: event.mapPoint 
            });

            locatorTask.locationToAddress(event.mapPoint).then(function(
              response) {
              view.popup.content = response.address;
            }).otherwise(function(err) {
              view.popup.content =
                "No address was found for this location";
            });
          });
				})
			}
		}
	}
</script>
<style>
	@import url('https://js.arcgis.com/4.4/esri/css/main.css');
	#header {
		width: 100%;
		height: 80px;
		background: #31363c;
		line-height: 80px;
		color: #fff;
		font-size: 40px;
	}
	#map-info {
		width: 100%;
		height: calc(100% - 80px);
	}
	#minMap {
		width: 300px;
		height: 200px;
		position: absolute;
		bottom: 0;
		left: 0;
		border: 1px solid #111;
		border-left: 0;
		border-bottom: 0;
	}
	#extentMap {
		position: absolute;
		background: #000;
		opacity: 0.5;
	}
	.esri-ui-manual-container>.esri-component {
		display: none;
	}
</style>