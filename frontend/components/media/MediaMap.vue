<template>
  <div
    id="map"
    class="card-style-base dark:brightness-95 dark:contrast-[90%] dark:hue-rotate-180 dark:invert"
  ></div>
</template>

<script setup lang="ts">
import MapLibreGlDirections, {
  layersFactory,
} from "@maplibre/maplibre-gl-directions";
import maplibregl from "maplibre-gl";
import "maplibre-gl/dist/maplibre-gl.css";

const i18n = useI18n();
const props = defineProps<{ markerColors: string[] }>();
const colorMode = useColorMode();

const isTouchDevice =
  // `maxTouchPoints` isn't recognized by TS. Safe to ignore.
  // eslint-disable-next-line @typescript-eslint/ban-ts-comment
  // @ts-ignore
  navigator.msMaxTouchPoints > 0 ||
  "ontouchstart" in window ||
  navigator.maxTouchPoints > 0;

function isWebglSupported() {
  if (window.WebGLRenderingContext) {
    const canvas = document.createElement("canvas");
    try {
      const context = canvas.getContext("webgl2") || canvas.getContext("webgl");
      if (context && typeof context.getParameter == "function") {
        return true;
      }
    } catch (e) {
      // WebGL is supported, but disabled.
    }
    return false;
  }
  // WebGL not supported.
  return false;
}

onMounted(() => {
  if (!isWebglSupported()) {
    alert(i18n.t("components.media-map.maplibre-gl-alert"));
  } else {
    const map = new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: {
          "raster-tiles": {
            type: "raster",
            tiles: ["https://tile.openstreetmap.org/{z}/{x}/{y}.png"],
            tileSize: 256,
            attribution:
              '<a href="https://www.openstreetmap.org/about" target="_blank">Data &copy; OpenStreetMap contributors</a>',
          },
        },
        layers: [
          {
            id: "background",
            type: "background",
            paint: {
              "background-color":
                colorMode.preference == "dark" ? "#131316" : "#F6F8FA",
            },
          },
          {
            id: "simple-tiles",
            type: "raster",
            source: "raster-tiles",
            minzoom: 0,
            maxzoom: 24,
          },
        ],
      },
      center: [13.38, 52.5168],
      zoom: 15,
      pitch: 20,
    });

    map.addControl(
      new maplibregl.NavigationControl({
        visualizePitch: true,
      }),
      "top-left"
    );
    map.addControl(new maplibregl.FullscreenControl());
    map.addControl(
      new maplibregl.GeolocateControl({
        positionOptions: {
          enableHighAccuracy: true,
        },
        trackUserLocation: true,
      })
    );

    new maplibregl.Marker({ color: `${props.markerColors[0]}` })
      .setLngLat([13.3778, 52.5163])
      .addTo(map);

    map.on("load", () => {
      const layers = layersFactory(
        isTouchDevice ? 1.5 : 1,
        isTouchDevice ? 2 : 1
      );

      const directions = new MapLibreGlDirections(map, {
        requestOptions: {
          alternatives: "true",
        },
        layers,
      });

      directions.interactive = true;

      document.addEventListener("keydown", (event) => {
        if (event.key === "x") {
          directions.clear();
        }
      });

      const clearDirectionsTextColor =
        colorMode.preference == "dark" ? "white" : "black";

      // const clearDirectionsControl = `
      //   <div style="
      //     background-color: rgba(255, 255, 255, 1);
      //     padding: 1px 5px;
      //     margin: 10px;
      //     border-radius: 5px;
      //     box-shadow: 0 0 1px 2px rgba(0, 0, 0, 0.15);
      //     color: ${clearDirectionsTextColor};
      //   ">
      //     Click to clear directions
      //   </div>
      // `;

      const clearDirectionsHotkeyControl = `
        <div style="
          background-color: rgba(255, 255, 255, 1);
          padding: 1px 5px;
          margin: 10px;
          border-radius: 5px;
          box-shadow: 0 0 1px 2px rgba(0, 0, 0, 0.15);
          color: ${clearDirectionsTextColor};
        ">
          Press x to clear directions
        </div>
      `;
      map.addControl(
        {
          onAdd: function () {
            const div = document.createElement("div");
            if (window.innerWidth >= 768) {
              div.innerHTML = clearDirectionsHotkeyControl;
            }
            // if (window.innerWidth < 768) {
            //   div.innerHTML = clearDirectionsControl;
            // } else {
            //   div.innerHTML = clearDirectionsHotkeyControl;
            // }
            return div;
          },
          onRemove: function () {},
        },
        "bottom-left"
      );
    });
  }
});
</script>
