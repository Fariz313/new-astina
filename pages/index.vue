<template>
  <div>
    <div
      style="height: 100vh; background-color: black"
      id="map-conatiner"
    ></div>
    <SideChart
      v-if="hover"
      :wilayah="dapil"
      :dataChart="dataChart"
      :left="left"
    />
  </div>
</template>
<script>
import { party } from "../assets/party";
import * as turf from "@turf/turf";
export default {
  data() {
    return {
      paintData: [],
      geoJson: {
        type: "FeatureCollection",
        features: [],
      },
      geoJsonTurfted: {
        type: "FeatureCollection",
        features: [],
      },
      hover: false,
      dapil: "",
      left: true,
      dataChart: "[]",
      dapilRes: null,
    };
  },
  methods: {
    getRndInteger(min, max) {
      return Math.floor(Math.random() * (max - min)) + min;
    },
    generatePaint() {
      let dapilRes = this.dapilRes;
      this.paintData = {
        "fill-color": ["match", ["get", "id"]],
        "fill-opacity": 1,
      };
      this.geoJsonTurfted.features.forEach((element) => {
        const idwil = dapilRes.table[element.properties.id];
        if (idwil) {
          let highest = 0;
          let highestkey = "";
          Object.keys(idwil).forEach((key) => {
            if (key !== "persen") {
              if (highest < idwil[key]) {
                highest = idwil[key];
                highestkey = key;
              }
            }
          });
          if (highestkey && element.properties.id) {
            if (party[highestkey]) {
              this.paintData["fill-color"].push(element.properties.id);
              this.paintData["fill-color"].push(party[highestkey].warna);
            }
          }
        }
      });
      this.paintData["fill-color"].push("#000000");
    },
    search(value, myArray) {
      if (myArray.hasOwnProperty(value)) {
        return myArray[value];
      } else {
        return null;
      }
      //   return myArray.table.find((e) => e.id === value);

      //   for (let i = 0; i < myArray.length; i++) {
      //     if (myArray[i].dapil === value) {
      //       return myArray[i];
      //     }
      //   }
    },
  },
  mounted() {
    $fetch("https://api.ardaworks.id/election/v1/dapil/dprri?border=true", {
      method: "GET",
    }).then((data) => {
      data.data.forEach((element) => {
        if (element.border) {
          this.geoJson.features.push({
            ...element.border,
            properties: {
              _id: element._id,
              id: element.id,
              kodeWilayah: element.kodeWilayah,
              dapil: element.dapil,
              name: element.name,
              parent: element.parent,
              tingkat: element.tingkat,
              type: element.type,
              urut: element.urut,
            },
          });
        }
      });
      $fetch("https://sirekap-obj-data.kpu.go.id/pemilu/hhcd/pdpr/0.json", {
        method: "GET",
      }).then((dapilRes) => {
        this.dapilRes = dapilRes;
        this.geoJson.features.sort((a, b) => b.properties.id - a.properties.id);
        let tempData = null;
        this.geoJson.features.forEach((element) => {
          if (tempData?.properties.id == element.properties.id) {
            const tempProperties = element.properties;
            tempData = turf.union(tempData, element);
            tempData.properties = {
              id: tempProperties.id,
              dapil: tempProperties.dapil,
            };
          } else if (tempData) {
            this.geoJsonTurfted.features.push({
              type: tempData.type,
              geometry: tempData.geometry,
              properties: {
                id: tempData.properties.id,
                dapil: tempData.properties.dapil,
              },
            });
            tempData = element;
          } else {
            tempData = element;
          }
        });
        this.geoJsonTurfted.features.push({
          type: tempData.type,
          geometry: tempData.geometry,
          properties: {
            id: tempData.properties.id,
            dapil: tempData.properties.dapil,
          },
        });
        this.generatePaint();
        const map = this.$generateMap(this.geoJsonTurfted, this.paintData);
        map.on("mousemove", "area-boundary", (e) => {
          const f = map.queryRenderedFeatures(e.point)[0];
          if (
            window.event.screenX <
            window.screen.width / 2 - 0.05 * window.screen.width
          ) {
            this.left = false;
          }
          if (
            window.event.screenX >
            window.screen.width / 2 + 0.05 * window.screen.width
          ) {
            this.left = true;
          }
          if (this.dapil !== e.features[0].properties.dapil) {
            this.hover = false;
          }
          this.dapil = e.features[0].properties.dapil;
          const provinfo = dapilRes.table[e.features[0].properties.id];
          if (provinfo) {
            this.dataChart = "[]";
            let dataChartArray = [];
            if (provinfo) {
              for (let index = 0; index < 20; index++) {
                if (provinfo[index + 1]) {
                  dataChartArray.push(provinfo[index + 1]);
                } else {
                  dataChartArray.push(0);
                }
              }
              this.dataChart = JSON.stringify(dataChartArray);
            }
          }

          this.hover = true;
          this.lastFeature = f.properties.province;
        });
        map.on("mouseleave", "area-boundary", (e) => {
          this.prov = "";
          this.dataChart = {};
          this.hover = false;
        });
      });
    });
  },
};
</script>
