<template>
    <div id="mapContainer" class="basemap"></div>
</template>

<script>
import mapboxgl from "mapbox-gl";

export default {
    name: "BaseMap",
    props: {
        active_filters: Array,
    },
    data() {
        return {
            filter_state: [
                { category: "entertainment", color: "#f34c46", filter: 0 },
                { category: "politics", color: "#fa8d4f", filter: 0 },
                { category: "business", color: "#fdd742", filter: 0 },
                { category: "sports", color: "#a3e048", filter: 0 },
                { category: "technology", color: "#49da9a", filter: 0 },
                { category: "Positive", color: "#A71EF7", filter: 0},
                { category: "Negative", color: "#CA3B3B", filter: 0},
                { category: "Neutral", color: "#909681", filter: 0}
            ],
            map: null,
            circleColor: ["match", ["get", "category"], "#ccc"],
            // datafile: require("./articles_with_location.json")
        };
    },
    watch: {
        active_filters: {
            handler() {
                this.chooseJSON();
            },
            deep: true,
        },
    },
    methods: {
        resetMap(){
            if (this.map.getSource("mypoints") !== undefined){
                if (this.map.getLayer("initial-layer")) {
                        this.map.removeLayer("initial-layer");
                }
                this.map.removeSource("mypoints");
                this.map.remove();
                mapboxgl.accessToken = "pk.eyJ1Ijoic2hheWdyb3NlIiwiYSI6ImNraThraG92bDA2cGMyeHBobnlvbjJubmQifQ.IwOEDSUCw2clHdFk39XPWA";
                this.map = new mapboxgl.Map({
                    container: "mapContainer",
                    style: "mapbox://styles/shaygrose/cki8ki5sm96xn1ao1lk33h7e0",
                    center: [-10.5, 30],
                    minZoom: 1.8,
                });
            }
        },

        async chooseJSON(){
            if (this.active_filters[5].include){
                let datafile = require("./articles_with_location_positive.json");
                await this.resetMap();
                await this.load(this.map, datafile);
                setTimeout(() => {this.updateMap();}, 1000);
            }
            else if (this.active_filters[6].include){
                let datafile = require("./articles_with_location_negative.json");
                await this.resetMap();
                await this.load(this.map, datafile);
                setTimeout(() => {this.updateMap();}, 1000);
            }
            else if (this.active_filters[7].include){
                let datafile = require("./articles_with_location_neutral.json");
                await this.resetMap();
                await this.load(this.map, datafile);
                setTimeout(() => {this.updateMap();}, 1000);
            }
            else{
                let datafile = require("./articles_with_location.json");
                await this.resetMap();
                await this.load(this.map, datafile);
                setTimeout(() => {this.updateMap();}, 1000);
            }
        },
        updateMap() {
            for (let i in this.active_filters) {
                var temp = JSON.parse(JSON.stringify(this.circleColor));
                if (this.active_filters[i].include) {
                    temp.pop();
                    temp.push(this.active_filters[i].realCategory);
                    temp.push(this.active_filters[i].color);
                    temp.push("#ccc");

                    let newLayer = {
                        id: this.active_filters[i].realCategory,
                        type: "circle",
                        source: "mypoints",
                        paint: {
                            "circle-color": this.active_filters[i].bgColor,
                            "circle-radius": [
                                "interpolate",
                                ["linear"],
                                ["zoom"],
                                2,
                                ["ln", ["get", "int_sc"]],
                                6,
                                ["/", ["get", "nat_sc"], 5],
                                12,
                                ["/", ["get", "prov_sc"], 5],
                            ],
                        },
                        filter: 
                        [
                            "==",
                            "category",
                            this.active_filters[i].realCategory,
                        ]
                    };
                    if (this.map.getLayer("initial-layer")) {
                        this.map.removeLayer("initial-layer");
                    }
                    if (
                        this.map.getLayer(this.active_filters[i].realCategory)
                    ) {
                        this.map.removeLayer(
                            this.active_filters[i].realCategory
                        );
                    }
                    this.map.addLayer(newLayer);
                } else {
                    if (
                        this.map.getLayer(this.active_filters[i].realCategory)
                    ) {
                        this.map.removeLayer(
                            this.active_filters[i].realCategory
                        );
                    }
                }
            }
            
            var actives = false;
            for (let j in this.active_filters) {
                if (this.active_filters[j].include && this.active_filters[j].realCategory != "Negative"
                && this.active_filters[j].realCategory != "Positive"
                && this.active_filters[j].realCategory != "Neutral") {
                    actives = true;
                }
            }
            if (this.map.getLayer("initial-layer")) {
                this.map.removeLayer("initial-layer");
            }
            if (!actives) {
                    this.map.addLayer({
                        id: "initial-layer",
                        type: "circle",
                        source: "mypoints",
                        paint: {
                            "circle-color": 
                            [
                                "match",
                                ["get", "category"],
                                "entertainment",
                                "#f34c46",
                                "politics",
                                "#fa8d4f",
                                "business",
                                "#fdd742",
                                "sports",
                                "#a3e048",
                                "technology",
                                "#50d4fe",
                                "Positive",
                                "#A71EF7",
                                "Negative",
                                "#CA3B3B",
                                "Neutral",
                                "#0064FA",
                                /* other */ "#ccc",
                            ],
                            "circle-radius": [
                                "interpolate",
                                ["linear"],
                                ["zoom"],
                                2,
                                ["ln", ["get", "int_sc"]],
                                6,
                                ["/", ["get", "nat_sc"], 5],
                                12,
                                ["/", ["get", "prov_sc"], 3],
                            ],
                        },
                    });
            }
        },

        load(map, datafile) {
            var nav = new mapboxgl.NavigationControl();
            map.addControl(nav, "top-right");

            map.on("load", function () {
                map.addSource("mypoints", {
                    type: "geojson",
                    //input the file name of the data you want to display
                    data: datafile,
                    // data: require("./articles_with_location.json")
                });

                map.addLayer({
                    id: "initial-layer",
                    type: "circle",
                    source: "mypoints",
                    paint: {
                        "circle-color": 
                        [
                            "match",
                            ["get", "category"],
                            "entertainment",
                            "#f34c46",
                            "politics",
                            "#fa8d4f",
                            "business",
                            "#fdd742",
                            "sports",
                            "#a3e048",
                            "technology",
                            "#50d4fe",
                            "Positive",
                            "#A71EF7",
                            "Negative",
                            "#CA3B3B",
                            "Neutral",
                            "#0064FA",
                            /* other */ "#ccc",
                        ],
                        "circle-radius": [
                            "interpolate",
                            ["linear"],
                            ["zoom"],
                            2,
                            ["ln", ["get", "int_sc"]],
                            6,
                            ["/", ["get", "nat_sc"], 5],
                            12,
                            ["/", ["get", "prov_sc"], 3],
                        ],
                    },
                });

                var popup = new mapboxgl.Popup({
                    className: "hover-popup",
                    closeButton: false,
                    closeOnClick: false,
                });

                // right now if multiple layers (categories) have a data point in the same location they get stacked, and if you click on the data point it opens up a pop up for every layer that has data point in that location and they stack on top of each other - this should be fixed when we're pulling different data where data points won't be overlapping (hopefully)
                map.on("click", "Neutral", function (e) {
                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;
                    var titles = e.features[0].properties.title;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }
                });

                map.on("mouseenter", "Neutral", function (e) {
                    // Change the cursor style as a UI indicator.
                    map.getCanvas().style.cursor = "pointer";

                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }

                    // Populate the popup and set its coordinates
                    // based on the feature found.
                    popup
                        .setLngLat(coordinates)
                        .setHTML("<div>" + topic + "</div>")
                        .addTo(map);
                });

                // Change it back to a pointer when it leaves.
                map.on("mouseleave", "Neutral", function () {
                    map.getCanvas().style.cursor = "";
                    popup.remove();
                });

                // right now if multiple layers (categories) have a data point in the same location they get stacked, and if you click on the data point it opens up a pop up for every layer that has data point in that location and they stack on top of each other - this should be fixed when we're pulling different data where data points won't be overlapping (hopefully)
                map.on("click", "Negative", function (e) {
                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;
                    var titles = e.features[0].properties.title;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }
                });

                map.on("mouseenter", "Negative", function (e) {
                    // Change the cursor style as a UI indicator.
                    map.getCanvas().style.cursor = "pointer";

                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }

                    // Populate the popup and set its coordinates
                    // based on the feature found.
                    popup
                        .setLngLat(coordinates)
                        .setHTML("<div>" + topic + "</div>")
                        .addTo(map);
                });

                // Change it back to a pointer when it leaves.
                map.on("mouseleave", "Negative", function () {
                    map.getCanvas().style.cursor = "";
                    popup.remove();
                });

                // right now if multiple layers (categories) have a data point in the same location they get stacked, and if you click on the data point it opens up a pop up for every layer that has data point in that location and they stack on top of each other - this should be fixed when we're pulling different data where data points won't be overlapping (hopefully)
                map.on("click", "Positive", function (e) {
                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;
                    var titles = e.features[0].properties.title;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }
                });

                map.on("mouseenter", "Positive", function (e) {
                    // Change the cursor style as a UI indicator.
                    map.getCanvas().style.cursor = "pointer";

                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }

                    // Populate the popup and set its coordinates
                    // based on the feature found.
                    popup
                        .setLngLat(coordinates)
                        .setHTML("<div>" + topic + "</div>")
                        .addTo(map);
                });

                // Change it back to a pointer when it leaves.
                map.on("mouseleave", "Positive", function () {
                    map.getCanvas().style.cursor = "";
                    popup.remove();
                });

                // right now if multiple layers (categories) have a data point in the same location they get stacked, and if you click on the data point it opens up a pop up for every layer that has data point in that location and they stack on top of each other - this should be fixed when we're pulling different data where data points won't be overlapping (hopefully)
                map.on("click", "entertainment", function (e) {
                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;
                    var titles = e.features[0].properties.title;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }
                });

                map.on("mouseenter", "entertainment", function (e) {
                    // Change the cursor style as a UI indicator.
                    map.getCanvas().style.cursor = "pointer";

                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }

                    // Populate the popup and set its coordinates
                    // based on the feature found.
                    popup
                        .setLngLat(coordinates)
                        .setHTML("<div>" + topic + "</div>")
                        .addTo(map);
                });

                // Change it back to a pointer when it leaves.
                map.on("mouseleave", "entertainment", function () {
                    map.getCanvas().style.cursor = "";
                    popup.remove();
                });

                map.on("click", "politics", function (e) {
                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;
                    var titles = e.features[0].properties.title;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }
                });

                map.on("mouseenter", "politics", function (e) {
                    // Change the cursor style as a UI indicator.
                    map.getCanvas().style.cursor = "pointer";

                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }

                    // Populate the popup and set its coordinates
                    // based on the feature found.
                    popup
                        .setLngLat(coordinates)
                        .setHTML("<div>" + topic + "</div>")
                        .addTo(map);
                });

                // Change it back to a pointer when it leaves.
                map.on("mouseleave", "politics", function () {
                    map.getCanvas().style.cursor = "";
                    popup.remove();
                });

                map.on("click", "business", function (e) {
                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;
                    var titles = e.features[0].properties.title;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }
                });

                map.on("mouseenter", "business", function (e) {
                    // Change the cursor style as a UI indicator.
                    map.getCanvas().style.cursor = "pointer";

                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }

                    // Populate the popup and set its coordinates
                    // based on the feature found.
                    popup
                        .setLngLat(coordinates)
                        .setHTML("<div>" + topic + "</div>")
                        .addTo(map);
                });

                // Change it back to a pointer when it leaves.
                map.on("mouseleave", "business", function () {
                    map.getCanvas().style.cursor = "";
                    popup.remove();
                });

                map.on("click", "sports", function (e) {
                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;
                    var titles = e.features[0].properties.title;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }
                });

                map.on("mouseenter", "sports", function (e) {
                    // Change the cursor style as a UI indicator.
                    map.getCanvas().style.cursor = "pointer";

                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }

                    // Populate the popup and set its coordinates
                    // based on the feature found.
                    popup
                        .setLngLat(coordinates)
                        .setHTML("<div>" + topic + "</div>")
                        .addTo(map);
                });

                // Change it back to a pointer when it leaves.
                map.on("mouseleave", "sports", function () {
                    map.getCanvas().style.cursor = "";
                    popup.remove();
                });

                map.on("click", "technology", function (e) {
                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;
                    var titles = e.features[0].properties.title;
                });

                map.on("mouseenter", "technology", function (e) {
                    // Change the cursor style as a UI indicator.
                    map.getCanvas().style.cursor = "pointer";

                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }

                    // Populate the popup and set its coordinates
                    // based on the feature found.
                    popup
                        .setLngLat(coordinates)
                        .setHTML("<div>" + topic + "</div>")
                        .addTo(map);
                });

                // Change it back to a pointer when it leaves.
                map.on("mouseleave", "technology", function () {
                    map.getCanvas().style.cursor = "";
                    popup.remove();
                });

                map.on("click", "initial-layer", function (e) {
                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;
                    var titles = e.features[0].properties.title;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }
                });

                // // Change the cursor to a pointer when the mouse is over the places layer.
                // map.on("mouseenter", "covid-19", function () {
                //   map.getCanvas().style.cursor = "pointer";
                // });
                // Create a popup, but don't add it to the map yet.

                map.on("mouseenter", "initial-layer", function (e) {
                    // Change the cursor style as a UI indicator.
                    map.getCanvas().style.cursor = "pointer";

                    var coordinates = e.features[0].geometry.coordinates.slice();
                    var topic = e.features[0].properties.topic;

                    // Ensure that if the map is zoomed out such that multiple
                    // copies of the feature are visible, the popup appears
                    // over the copy being pointed to.
                    while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                        coordinates[0] +=
                            e.lngLat.lng > coordinates[0] ? 360 : -360;
                    }

                    // Populate the popup and set its coordinates
                    // based on the feature found.
                    popup
                        .setLngLat(coordinates)
                        .setHTML("<div>" + topic + "</div>")
                        .addTo(map);
                });

                // Change it back to a pointer when it leaves.
                map.on("mouseleave", "initial-layer", function () {
                    map.getCanvas().style.cursor = "";
                    popup.remove();
                });
            });
            return true;
        },
    },

    mounted() {
        mapboxgl.accessToken =
            "pk.eyJ1Ijoic2hheWdyb3NlIiwiYSI6ImNraThraG92bDA2cGMyeHBobnlvbjJubmQifQ.IwOEDSUCw2clHdFk39XPWA";
        this.map = new mapboxgl.Map({
            container: "mapContainer",
            style: "mapbox://styles/shaygrose/cki8ki5sm96xn1ao1lk33h7e0",
            center: [-10.5, 30],
            minZoom: 1.8,
        });
        let datafile = require("./articles_with_location.json")
        this.load(this.map, datafile);
    },
};
</script>

<style>
@import url("https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.css");
.basemap {
    position: absolute;
    width: 100%;
    height: 100%;
}
.headline-wrapper {
    margin: 10px;
    float: left;
}
.article {
    display: flex;
    justify-content: flex-start;
    align-items: center;
}
.image {
    border: 0.1px solid #8da9bf;
    width: 60px;
    height: 40px;
}
.hover-popup .mapboxgl-popup-content {
    font-size: 13px;
    font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
    background-color: #cbd5dc;
    color: #3f4b55;
    padding: 6px 8px;
    font-weight: bold;
    text-align: center;
    border: 0.1px solid #8da9bf;
}
.hover-popup .mapboxgl-popup-tip {
    border-top-color: transparent;
    color: transparent;
}

.click-popup .mapboxgl-popup-content {
    font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
    background-color: #cbd5dc;
    color: #3f4b55;
    padding: 10px 15px;
    border-radius: 5px;
    border: 0.1px solid #3f4b55;
    width: 300px;
}
.click-popup .mapboxgl-popup-content .title {
    font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
    font-size: 24px;
    /*margin: 5px 0px 15px 0px;*/
}
.click-popup .mapboxgl-popup-tip {
    border-top-color: #cbd5dc;
    color: #cbd5dc;
}

.mapboxgl-popup-content a {
    font: "Avenir";
    text-decoration: none;
    color: #3f4b55;
    margin: 5px 0px 5px 10px;
}

.mapboxgl-popup-content a:hover {
    text-decoration: underline;
    /* color: #d0d0d9; */
    /* color: #cbd5dc; */
}

.mapboxgl-popup-close-button {
    font-size: 16px;
    color: white;
}
.mapboxgl-ctrl-group:not(:empty) {
    background-color: #cbd5dc;
    border: 1px solid #8da9bf;
    border-radius: 5px;
}
.mapboxgl-ctrl button {
    /*background-color: white; */
    color: white;
    transition: all;
    transition-duration: 150ms;
}
.topic {
    font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
    color: #3f4b55;
    text-align: center;
    font-size: 15px;
    font-weight: bold;
}
</style>


<!--template>
//     <div id="mapContainer" class="basemap"></div>
// </template>

// <script>
// import mapboxgl from "mapbox-gl";

// export default {
//     name: "BaseMap",
//     props: {
//         active_filters: Array
//     },
//     data() {
//         return {
//             filter_state: [
//                 { category: "entertainment", color: "#f34c46", filter: 0 },
//                 { category: "politics", color: "#fa8d4f", filter: 0 },
//                 { category: "business", color: "#fdd742", filter: 0 },
//                 { category: "sports", color: "#a3e048", filter: 0 },
//                 { category: "technology", color: "#49da9a", filter: 0 },
//                 { category: "Positive", color: "#A71EF7", filter: 0},
//                 { category: "Negative", color: "#CA3B3B", filter: 0},
//                 { category: "Neutral", color: "#909681", filter: 0}
//             ],
//             map: null,
//             circleColor: ["match", ["get", "category"], "#ccc"],
//         };
//     },
//     watch: {
//         active_filters: {
//             handler() {
//                 this.updateMap();
//                 // this.chooseJSON();
//             },
//             deep: true,
//         },
//     },
//     methods: {
//         resetMap(){
//             if (this.map.getSource("mypoints") !== undefined){
//                 if (this.map.getLayer("initial-layer")) {
//                         this.map.removeLayer("initial-layer");
//                     }
//                 this.map.removeSource("mypoints");
//                 this.map.remove();
//                 mapboxgl.accessToken = "pk.eyJ1Ijoic2hheWdyb3NlIiwiYSI6ImNraThraG92bDA2cGMyeHBobnlvbjJubmQifQ.IwOEDSUCw2clHdFk39XPWA";
//                 this.map = new mapboxgl.Map({
//                     container: "mapContainer",
//                     style: "mapbox://styles/shaygrose/cki8ki5sm96xn1ao1lk33h7e0",
//                     center: [-10.5, 30],
//                     minZoom: 1.8,
//                 });
//             }
//         },
//         chooseJSON(){
//             if (this.active_filters[5].include){
//                 let datafile = require("./articles_with_location_positive.json");
//                 this.resetMap();
//                 this.load(this.map, datafile);
//                 this.updateMap();
//             }
//             else if (this.active_filters[6].include){
//                 let datafile = require("./articles_with_location_negative.json");
//                 this.resetMap();
//                 this.load(this.map, datafile);
//                 this.updateMap();
//             }
//             else if (this.active_filters[7].include){
//                 let datafile = require("./articles_with_location_neutral.json");
//                 this.resetMap();
//                 this.load(this.map, datafile);
//                 this.updateMap();
//             }
//             else{
//                 let datafile = require("./articles_with_location.json");
//                 this.resetMap();
//                 this.load(this.map, datafile);
//                 this.updateMap();
//             }
//         },
//         updateMap() {
//             for (let i in this.active_filters) {
//                 var temp = JSON.parse(JSON.stringify(this.circleColor));
//                 if (this.active_filters[i].include) {
//                     temp.pop();
//                     temp.push(this.active_filters[i].realCategory);
//                     temp.push(this.active_filters[i].color);
//                     temp.push("#ccc");

//                     let newLayer = {
//                         id: this.active_filters[i].realCategory,
//                         type: "circle",
//                         source: "mypoints",
//                         paint: {
//                             "circle-color": temp,
//                             "circle-radius": [
//                                 "interpolate",
//                                 ["linear"],
//                                 ["zoom"],
//                                 2,
//                                 ["ln", ["get", "int_sc"]],
//                                 6,
//                                 ["/", ["get", "nat_sc"], 5],
//                                 12,
//                                 ["/", ["get", "prov_sc"], 5],
//                             ],
//                         },
//                         filter: [
//                             [
//                                 "==",
//                                 "category",
//                                 this.active_filters[i].realCategory,
//                             ]
//                         ]
//                     };
//                     if (this.map.getLayer("initial-layer")) {
//                         this.map.removeLayer("initial-layer");
//                     }
//                     if (
//                         this.map.getLayer(this.active_filters[i].realCategory)
//                     ) {
//                         this.map.removeLayer(
//                             this.active_filters[i].realCategory
//                         );
//                     }
//                     this.map.addLayer(newLayer);
//                 } else {
//                     if (
//                         this.map.getLayer(this.active_filters[i].realCategory)
//                     ) {
//                         this.map.removeLayer(
//                             this.active_filters[i].realCategory
//                         );
//                     }
//                 }
//             }
            
//             var actives = false;
//             for (let j in this.active_filters) {
//                 if (this.active_filters[j].include) {
//                     actives = true;
//                 }
//             }
//             if (!actives) {
//                 if (this.map.isStyleLoaded()) {
//                     this.map.addLayer({
//                         id: "initial-layer",
//                         type: "circle",
//                         source: "mypoints",
//                         paint: {
//                             "circle-color": [
//                                 "match",
//                                 ["get", "category"],
//                                 "entertainment",
//                                 "#f34c46",
//                                 "politics",
//                                 "#fa8d4f",
//                                 "business",
//                                 "#fdd742",
//                                 "sports",
//                                 "#a3e048",
//                                 "technology",
//                                 "#50d4fe",
//                                 "Positive",
//                                 "#A71EF7",
//                                 "Negative",
//                                 "#CA3B3B",
//                                 "Neutral",
//                                 "#909681",
//                                 /* other */ "#ccc",
//                             ],
//                             "circle-radius": [
//                                 "interpolate",
//                                 ["linear"],
//                                 ["zoom"],
//                                 2,
//                                 ["ln", ["get", "int_sc"]],
//                                 6,
//                                 ["/", ["get", "nat_sc"], 5],
//                                 12,
//                                 ["/", ["get", "prov_sc"], 3],
//                             ],
//                         },
//                     });
//                 }
//             }
//         },

//         load(map, datafile) {
//         var nav = new mapboxgl.NavigationControl();
//             map.addControl(nav, "top-right");

//             map.on("load", function () {
//                 map.addSource("mypoints", {
//                     type: "geojson",
//                     //input the file name of the data you want to display
//                     data: datafile,
//                 });

//                 map.addLayer({
//                     id: "initial-layer",
//                     type: "circle",
//                     source: "mypoints",
//                     paint: {
//                         "circle-color": [
//                             "match",
//                             ["get", "category"],
//                             "entertainment",
//                             "#f34c46",
//                             "politics",
//                             "#fa8d4f",
//                             "business",
//                             "#fdd742",
//                             "sports",
//                             "#a3e048",
//                             "technology",
//                             "#50d4fe",
//                             "Positive",
//                             "#A71EF7",
//                             "Negative",
//                             "#CA3B3B",
//                             "Neutral",
//                             "#909681",
//                             /* other */ "#ccc",
//                         ],

//                         "circle-radius": [
//                             "interpolate",
//                             ["linear"],
//                             ["zoom"],
//                             2,
//                             ["ln", ["get", "int_sc"]],
//                             6,
//                             ["/", ["get", "nat_sc"], 5],
//                             12,
//                             ["/", ["get", "prov_sc"], 3],
//                         ],
//                     },
//                 });

//                 var popup = new mapboxgl.Popup({
//                     className: "hover-popup",
//                     closeButton: false,
//                     closeOnClick: false,
//                 });

//                 // right now if multiple layers (categories) have a data point in the same location they get stacked, and if you click on the data point it opens up a pop up for every layer that has data point in that location and they stack on top of each other - this should be fixed when we're pulling different data where data points won't be overlapping (hopefully)
//                 map.on("click", "Positive", function (e) {
//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;
//                     var titles = e.features[0].properties.title;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }
//                 });

//                 map.on("mouseenter", "Positive", function (e) {
//                     // Change the cursor style as a UI indicator.
//                     map.getCanvas().style.cursor = "pointer";

//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }

//                     // Populate the popup and set its coordinates
//                     // based on the feature found.
//                     popup
//                         .setLngLat(coordinates)
//                         .setHTML("<div>" + topic + "</div>")
//                         .addTo(map);
//                 });

//                 // Change it back to a pointer when it leaves.
//                 map.on("mouseleave", "Positive", function () {
//                     map.getCanvas().style.cursor = "";
//                     popup.remove();
//                 });

//                 // right now if multiple layers (categories) have a data point in the same location they get stacked, and if you click on the data point it opens up a pop up for every layer that has data point in that location and they stack on top of each other - this should be fixed when we're pulling different data where data points won't be overlapping (hopefully)
//                 map.on("click", "entertainment", function (e) {
//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;
//                     var titles = e.features[0].properties.title;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }
//                 });

//                 map.on("mouseenter", "entertainment", function (e) {
//                     // Change the cursor style as a UI indicator.
//                     map.getCanvas().style.cursor = "pointer";

//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }

//                     // Populate the popup and set its coordinates
//                     // based on the feature found.
//                     popup
//                         .setLngLat(coordinates)
//                         .setHTML("<div>" + topic + "</div>")
//                         .addTo(map);
//                 });

//                 // Change it back to a pointer when it leaves.
//                 map.on("mouseleave", "entertainment", function () {
//                     map.getCanvas().style.cursor = "";
//                     popup.remove();
//                 });

//                 map.on("click", "politics", function (e) {
//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;
//                     var titles = e.features[0].properties.title;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }
//                 });

//                 map.on("mouseenter", "politics", function (e) {
//                     // Change the cursor style as a UI indicator.
//                     map.getCanvas().style.cursor = "pointer";

//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }

//                     // Populate the popup and set its coordinates
//                     // based on the feature found.
//                     popup
//                         .setLngLat(coordinates)
//                         .setHTML("<div>" + topic + "</div>")
//                         .addTo(map);
//                 });

//                 // Change it back to a pointer when it leaves.
//                 map.on("mouseleave", "politics", function () {
//                     map.getCanvas().style.cursor = "";
//                     popup.remove();
//                 });

//                 map.on("click", "business", function (e) {
//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;
//                     var titles = e.features[0].properties.title;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }
//                 });

//                 map.on("mouseenter", "business", function (e) {
//                     // Change the cursor style as a UI indicator.
//                     map.getCanvas().style.cursor = "pointer";

//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }

//                     // Populate the popup and set its coordinates
//                     // based on the feature found.
//                     popup
//                         .setLngLat(coordinates)
//                         .setHTML("<div>" + topic + "</div>")
//                         .addTo(map);
//                 });

//                 // Change it back to a pointer when it leaves.
//                 map.on("mouseleave", "business", function () {
//                     map.getCanvas().style.cursor = "";
//                     popup.remove();
//                 });

//                 map.on("click", "sports", function (e) {
//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;
//                     var titles = e.features[0].properties.title;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }
//                 });

//                 map.on("mouseenter", "sports", function (e) {
//                     // Change the cursor style as a UI indicator.
//                     map.getCanvas().style.cursor = "pointer";

//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }

//                     // Populate the popup and set its coordinates
//                     // based on the feature found.
//                     popup
//                         .setLngLat(coordinates)
//                         .setHTML("<div>" + topic + "</div>")
//                         .addTo(map);
//                 });

//                 // Change it back to a pointer when it leaves.
//                 map.on("mouseleave", "sports", function () {
//                     map.getCanvas().style.cursor = "";
//                     popup.remove();
//                 });

//                 map.on("click", "technology", function (e) {
//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;
//                     var titles = e.features[0].properties.title;
//                 });

//                 map.on("mouseenter", "technology", function (e) {
//                     // Change the cursor style as a UI indicator.
//                     map.getCanvas().style.cursor = "pointer";

//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }

//                     // Populate the popup and set its coordinates
//                     // based on the feature found.
//                     popup
//                         .setLngLat(coordinates)
//                         .setHTML("<div>" + topic + "</div>")
//                         .addTo(map);
//                 });

//                 // Change it back to a pointer when it leaves.
//                 map.on("mouseleave", "technology", function () {
//                     map.getCanvas().style.cursor = "";
//                     popup.remove();
//                 });

//                 map.on("click", "initial-layer", function (e) {
//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;
//                     var titles = e.features[0].properties.title;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }
//                 });

//                 // // Change the cursor to a pointer when the mouse is over the places layer.
//                 // map.on("mouseenter", "covid-19", function () {
//                 //   map.getCanvas().style.cursor = "pointer";
//                 // });
//                 // Create a popup, but don't add it to the map yet.

//                 map.on("mouseenter", "initial-layer", function (e) {
//                     // Change the cursor style as a UI indicator.
//                     map.getCanvas().style.cursor = "pointer";

//                     var coordinates = e.features[0].geometry.coordinates.slice();
//                     var topic = e.features[0].properties.topic;

//                     // Ensure that if the map is zoomed out such that multiple
//                     // copies of the feature are visible, the popup appears
//                     // over the copy being pointed to.
//                     while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//                         coordinates[0] +=
//                             e.lngLat.lng > coordinates[0] ? 360 : -360;
//                     }

//                     // Populate the popup and set its coordinates
//                     // based on the feature found.
//                     popup
//                         .setLngLat(coordinates)
//                         .setHTML("<div>" + topic + "</div>")
//                         .addTo(map);
//                 });

//                 // Change it back to a pointer when it leaves.
//                 map.on("mouseleave", "initial-layer", function () {
//                     map.getCanvas().style.cursor = "";
//                     popup.remove();
//                 });
//             });
//         },

//     },

//     mounted() {
//         mapboxgl.accessToken =
//             "pk.eyJ1Ijoic2hheWdyb3NlIiwiYSI6ImNraThraG92bDA2cGMyeHBobnlvbjJubmQifQ.IwOEDSUCw2clHdFk39XPWA";
//         this.map = new mapboxgl.Map({
//             container: "mapContainer",
//             style: "mapbox://styles/shaygrose/cki8ki5sm96xn1ao1lk33h7e0",
//             center: [-10.5, 30],
//             minZoom: 1.8,
//         });
//         let datafile = require("./articles_with_location.json")
//         this.load(this.map, datafile);
//     },
// };
// </script>

// <style>
// @import url("https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.css");
// .basemap {
//     position: absolute;
//     width: 100%;
//     height: 100%;
// }
// .headline-wrapper {
//     margin: 10px;
//     float: left;
// }
// .article {
//     display: flex;
//     justify-content: flex-start;
//     align-items: center;
// }
// .image {
//     border: 0.1px solid #8da9bf;
//     width: 60px;
//     height: 40px;
// }
// .hover-popup .mapboxgl-popup-content {
//     font-size: 13px;
//     font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
//     background-color: #cbd5dc;
//     color: #3f4b55;
//     padding: 6px 8px;
//     font-weight: bold;
//     text-align: center;
//     border: 0.1px solid #8da9bf;
// }
// .hover-popup .mapboxgl-popup-tip {
//     border-top-color: transparent;
//     color: transparent;
// }

// .click-popup .mapboxgl-popup-content {
//     font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
//     background-color: #cbd5dc;
//     color: #3f4b55;
//     padding: 10px 15px;
//     border-radius: 5px;
//     border: 0.1px solid #3f4b55;
//     width: 300px;
// }
// .click-popup .mapboxgl-popup-content .title {
//     font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
//     font-size: 24px;
//     /*margin: 5px 0px 15px 0px;*/
// }
// .click-popup .mapboxgl-popup-tip {
//     border-top-color: #cbd5dc;
//     color: #cbd5dc;
// }

// .mapboxgl-popup-content a {
//     font: "Avenir";
//     text-decoration: none;
//     color: #3f4b55;
//     margin: 5px 0px 5px 10px;
// }

// .mapboxgl-popup-content a:hover {
//     text-decoration: underline;
//     /* color: #d0d0d9; */
//     /* color: #cbd5dc; */
// }

// .mapboxgl-popup-close-button {
//     font-size: 16px;
//     color: white;
// }
// .mapboxgl-ctrl-group:not(:empty) {
//     background-color: #cbd5dc;
//     border: 1px solid #8da9bf;
//     border-radius: 5px;
// }
// .mapboxgl-ctrl button {
//     /*background-color: white; */
//     color: white;
//     transition: all;
//     transition-duration: 150ms;
// }
// .topic {
//     font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
//     color: #3f4b55;
//     text-align: center;
//     font-size: 15px;
//     font-weight: bold;
// }
// </style>
