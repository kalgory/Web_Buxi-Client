<template>
  <div id="map" />
</template>

<script>

export default {
  name: 'NaverMap',

  props: {
    zoom: {
      type: Number,
      required: false,
      default: 18,
    },
    centerPosition: {
      type: Object,
      required: true,
    },
  },

  data: () => ({
    map: null,
    markers: [],
    busMarker: null,
    hasClickEvent: true,
    isClickLoading: false,
    busMarkerIcon: {
      content: '<svg style="width:22px;height:22px; color: white; background-color: #286955; border-radius: 50%; padding: 3px" viewBox="0 0 24 24">'
        + '   <path fill="currentColor" d="M18,11H6V6H18M16.5,17A1.5,1.5 0 0,1 15,15.5A1.5,1.5 0 0,1 16.5,14A1.5,1.5 0 0,1 18,15.5A1.5,1.5 0 0,1 16.5,17M7.5,17A1.5,1.5 0 0,1 6,15.5A1.5,1.5 0 0,1 7.5,14A1.5,1.5 0 0,1 9,15.5A1.5,1.5 0 0,1 7.5,17M4,16C4,16.88 4.39,17.67 5,18.22V20A1,1 0 0,0 6,21H7A1,1 0 0,0 8,20V19H16V20A1,1 0 0,0 17,21H18A1,1 0 0,0 19,20V18.22C19.61,17.67 20,16.88 20,16V6C20,2.5 16.42,2 12,2C7.58,2 4,2.5 4,6V16Z" />'
        + '</svg>',
    },
    departureStopMarkerIcon: {
      content: '<div class="container" style="'
        + 'position: absolute;'
        + 'background-color: white;'
        + 'line-height: 10px;'
        + 'width: 40px;'
        + 'height: 10px;'
        + 'border-radius: 10px 10px 10px 10px;'
        + 'text-align: center;'
        + 'border: 2px solid black;'
        + 'color: black;'
        + 'font-weight: bold;'
        + 'font-size: medium">'
        + '출발'
        + '<div style="background-color: black; width: 2px; height: 10px; margin-left: 50%; margin-top: 12px; align-content: center"></div>'
        + '</div>',
    },
    arrivalStopMarkerIcon: {
      content: '<div class="container" style="position: absolute;'
        + 'background-color: white;'
        + 'line-height: 10px;'
        + 'width: 40px;'
        + 'height: 10px;'
        + 'border-radius: 10px 10px 10px 10px;'
        + 'text-align: center;'
        + 'border: 2px solid #ef2c2c;'
        + 'color: #ef2c2c;'
        + 'font-weight: bold;'
        + 'font-size: medium">'
        + '도착'
        + '<div style="background-color: #ef2c2c; width: 2px; height: 10px; margin-left: 50%; margin-top: 12px; align-content: center"></div>'
        + '</div>',
    },
  }),

  watch: {
    centerPosition(position) {
      this.map.setCenter(position);
    },
  },

  mounted() {
    // eslint-disable-next-line no-unused-expressions
    window.naver ? this.initMap() : this.addNaverMapApiScript();
    window.clickSelectStationButton = this.clickSelectStationButton;
  },

  methods: {
    addNaverMapApiScript() {
      /* global naver */
      const script = document.createElement('script');
      const type = 'text/javascript';
      const src = `https://openapi.map.naver.com/openapi/v3/maps.js?ncpClientId=${this.$clientID}`;
      script.setAttribute('async', '');
      script.setAttribute('defer', '');
      script.setAttribute('type', type);
      script.setAttribute('src', src);
      script.id = 'map';
      script.onload = () => this.initMap();
      document.body.appendChild(script);
    },
    initMap() {
      const container = document.getElementById('map');
      this.map = new naver.maps.Map(container, {
        center: new naver.maps.LatLng(
          this.centerPosition.lat,
          this.centerPosition.lng,
        ),
        zoom: this.zoom,
      });
      this.$emit('mapLoaded');
      this.addClickMapEventListener();
      this.addDragendEventListener();
      this.departureStopMarkerIcon.anchor = new naver.maps.Point(33.4, 46);
      this.arrivalStopMarkerIcon.anchor = new naver.maps.Point(33.4, 46);
      this.busMarkerIcon.anchor = new naver.maps.Point(13.2, 18);
    },
    addDragendEventListener() {
      window.naver.maps.Event.addListener(this.map, 'dragend', () => {
        this.$emit('dragend', {
          lat: this.map.getCenter().y,
          lng: this.map.getCenter().x,
        });
      });
    },
    addClickMapEventListener() {
      window.naver.maps.Event.addListener(this.map, 'click', (eventArgument) => {
        if (this.hasClickEvent) {
          this.isClickLoading = true;
          this.$emit('click', eventArgument.coord);
          this.hasClickEvent = false;
          this.$emit('clickMapEventComplete');
          this.isClickLoading = false;
        }
      });
    },
    addClickMarkerEventListener(marker, stationInformation) {
      const contentString = [
        '<div class="iw_inner" style="width: 240px;height: 80px; border-radius: 10px 10px 10px 10px; background-color: #ffffff; padding-top: 10px; border: 3px solid #286955">',
        '   <h3 class="text-center">', stationInformation.name, '</h1>',
        '   <p class="center" style="padding-top: 5px">',
        `       <button class="button"
                        onclick="clickSelectStationButton('Departure', '${encodeURIComponent(JSON.stringify(stationInformation))}')"
                        style="background-color: white; margin-right: 3px; border: 2px solid black; color: black"
                        >
                        출발지 선택
                </button>`,
        `       <button class="button"
                        onclick="clickSelectStationButton('Arrival', '${encodeURIComponent(JSON.stringify(stationInformation))}')"
                        style="background-color: white; margin-left: 3px; border: 2px solid #ef2c2c; color: #ef2c2c"
                        >
                        도착지 선택
                </button>`,
        '   </p>',
        '</div>'].join('');
      const infoWindow = new naver.maps.InfoWindow({
        content: contentString,
        backgroundColor: 'transparent',
        maxWidth: 240,
        maxHeight: 'auto',
        borderColor: 'transparent',
        anchorSize: new naver.maps.Size(20, 35),
        anchorSkew: true,
        anchorColor: 'white',
      });
      window.naver.maps.Event.addListener(marker, 'click', () => {
        if (infoWindow.getMap()) {
          infoWindow.close();
        } else {
          infoWindow.open(this.map, marker);
        }
      });
      window.naver.maps.Event.addListener(this.map, 'click', () => {
        infoWindow.close();
      });
    },
    setHasClickEvent() {
      if (!this.isClickLoading) {
        this.hasClickEvent = true;
      } else {
        this.$on('clickMapEventComplete', () => {
          this.hasClickEvent = true;
        });
      }
    },
    addMarker(stationInformation) {
      const marker = new window.naver.maps.Marker({
        map: this.map,
        position: new window.naver.maps.LatLng(stationInformation.lat, stationInformation.lng),
        clickable: true,
      });
      marker.setMap(this.map);
      this.markers.push(marker);
      this.addClickMarkerEventListener(marker, stationInformation);
    },
    setMarker(stationInformation) {
      // eslint-disable-next-line no-unused-expressions
      window.naver ? this.addMarker(stationInformation) : this.$on('mapLoaded', () => this.addMarker(stationInformation));
    },
    removeMarkers() {
      this.markers.forEach((marker) => {
        marker.setMap(null);
      });
      this.markers = null;
      this.setHasClickEvent();
    },
    clickSelectStationButton(selectOptionString, stationInformation) {
      const object = JSON.parse(decodeURIComponent(stationInformation));
      if (selectOptionString === 'Departure') {
        this.$emit('setDepartureStop', object);
      } else if (selectOptionString === 'Arrival') {
        this.$emit('setArrivalStop', object);
      }
    },
    getCustomMarkerIcon(purposeOfMarker) {
      if (purposeOfMarker === 'bus') return this.busMarkerIcon;
      if (purposeOfMarker === 'departureStop') return this.departureStopMarkerIcon;
      if (purposeOfMarker === 'arrivalStop') return this.arrivalStopMarkerIcon;
      return null;
    },
    setCustomMarker(markerPosition, purposeOfMarker) {
      if (window.naver && this.map) {
        const markerIcon = this.getCustomMarkerIcon(purposeOfMarker);
        if (this.busMarker !== null && purposeOfMarker === 'bus') {
          this.busMarker.setMap(null);
          this.busMarker = null;
        }
        const marker = new window.naver.maps.Marker({
          map: this.map,
          icon: markerIcon,
          position: new window.naver.maps.LatLng(markerPosition.lat, markerPosition.lng),
        });
        if (purposeOfMarker === 'bus') this.busMarker = marker;
        marker.setMap(this.map);
      } else {
        this.$on('mapLoaded', () => {
          const markerIcon = this.getCustomMarkerIcon(purposeOfMarker);
          if (this.busMarker !== null && purposeOfMarker === 'bus') this.busMarker.setMap(null);
          const marker = new window.naver.maps.Marker({
            map: this.map,
            icon: markerIcon,
            position: new window.naver.maps.LatLng(markerPosition.lat, markerPosition.lng),
          });
          if (purposeOfMarker === 'bus') this.busMarker = marker;
          marker.setMap(this.map);
        });
      }
    },
    getRadius() {
      const mapZoomLevel = this.map.getZoom();
      switch (mapZoomLevel) {
        case 21:
          return 50;
        case 20:
          return 70;
        case 19:
          return 130;
        case 18:
          return 190;
        case 17:
          return 300;
        case 16:
          return 500;
        default:
          break;
      }
      return null;
    },
  },
};
</script>

<style>
.button {
  width: 90px;
  height: 30px;
  border-radius: 12px;
  z-index: 1;
}

.center {
  display: flex;
  justify-content: center;
}
</style>
