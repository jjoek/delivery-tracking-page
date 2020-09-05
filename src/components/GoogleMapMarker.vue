<script>
import { POINT_MARKER_ICON_CONFIG, ACTIVE_POINT_MARKER_ICON_CONFIG } from '@/constants/mapSettings';

export default {
    props: {
        google: {
            type: Object,
            required: true,
        },
        map: {
            type: Object,
            required: true,
        },
        marker: {
            type: Object,
            required: true,
        },
    },

    data: () => ({
        markerObj: null,
        numDeltas: 100,
        delay: 10,
        deltaLat: null,
        deltaLng: null,
        iterator: 0,

    }),

    computed: {
        active() {
            return this.marker.current_state;
        },

        contentStr() {
            return `${'<div id="content">'
            + '<div id="siteNotice">'
            + '</div>'
            + '<h1 id="firstHeading" class="firstHeading">'}${this.marker.driver}</h1>`
            + '<div id="bodyContent">'
            + `<p>Currently at: ${this.marker.position.lat},  ${this.marker.position.lng}</p>`
            + '</div>'
            + '</div>';
        },
    },

    watch: {
        active() {
            this.markerObj.setMap(null);

            this.initialize();
        },
    },

    methods: {

        transition() {
            if (this.marker && this.marker.previous_position.lat) {
                this.deltaLat = (this.marker.position.lat - this.marker.previous_position.lat) / this.numDeltas;
                this.deltaLng = (this.marker.position.lng - this.marker.previous_position.lng) / this.numDeltas;

                this.moveMarker();
            }
        },

        moveMarker() {
            this.marker.previous_position.lat += this.deltaLat;
            this.marker.previous_position.lng += this.deltaLng;

            const latlng = new this.google.maps.LatLng(this.marker.previous_position.lat, this.marker.previous_position.lng);

            this.markerObj.setPosition(latlng);

            if (this.iterator !== this.numDeltas) {
                this.iterator += 1;
                setTimeout(this.moveMarker, this.delay);
            }
        },

        initialize() {
            const locationCoordinates = `@ ${this.marker.position.lat}, ${this.marker.position.lng}`;

            this.markerObj = new this.google.maps.Marker({
                position: this.marker.position,
                marker: this.marker,
                label: {
                    color: '#00aaff',
                    fontWeight: 'bold',
                    fontSize: '14px',
                    text: this.marker.current_state === 'active' ? this.marker.driver : '.',
                },
                map: this.map,
                title: locationCoordinates,
                icon: this.marker.current_state === 'active' ? ACTIVE_POINT_MARKER_ICON_CONFIG : POINT_MARKER_ICON_CONFIG,
            });

            this.transition();

            if (this.marker.current_state === 'active') {
                const self = this;

                const infowindow = new this.google.maps.InfoWindow({
                    content: self.contentStr,
                });

                this.markerObj.addListener('click', () => {
                    infowindow.open(self.map, self.markerObj);
                });
            }
        },
    },

    mounted() {
        this.initialize();
    },

    render() {
        return null;
    },
};
</script>
