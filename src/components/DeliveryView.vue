<template>
    <GoogleMapLoader
        :mapConfig="mapConfig"
        apiKey="AIzaSyAXPsC6VZy82P_LaXTx4_ufWT9NWdS5SMU"
    >
        <template v-slot="{ google, map }">
            <GoogleMapMarker
                v-for="marker in markers"
                :key="marker.id"
                :marker="marker"
                :google="google"
                :map="map"
            />
            <GoogleMapLine
                v-for="line in lines"
                :key="line.id"
                :path.sync="line.path"
                :google="google"
                :map="map"
            />
        </template>
    </GoogleMapLoader>
</template>
<script>
import { mapSettings } from '@/constants/mapSettings';
import GoogleMapLoader from './GoogleMapLoader';
import GoogleMapLine from './GoogleMapLine';
import GoogleMapMarker from './GoogleMapMarker';

export default {
    components: {
        GoogleMapLoader, GoogleMapLine, GoogleMapMarker,
    },

    data() {
        return {
            markers: [],
            lines: [],
            apiCaller: null,
            driver: 'Driver A',
        };
    },

    computed: {
        mapConfig() {
            return {
                ...mapSettings,
                center: { lat: -1.298982, lng: 36.776811 },
            };
        },

        locationCoordinates() {
            return [
                [-1.300355, 36.773850],
                [-1.298982, 36.776811],
                [-1.297459, 36.776747],
                [-1.296193, 36.776726],
                [-1.296097, 36.779236],
                [-1.296151, 36.777637],
                [-1.296215, 36.776693],
                [-1.294252, 36.776586],
                [-1.294048, 36.776790],
                [-1.293973, 36.779118],
                [-1.292622, 36.779075],
                [-1.291844, 36.779049],
            ];
        },

        mapCenter() {
            return this.markers[1].position;
        },
    },

    methods: {
        fetchLocations() {
            let currentLocationIndex = 0;

            const self = this;

            self.apicaller = setInterval(() => {
                if (self.locationCoordinates && (currentLocationIndex < self.locationCoordinates.length)) {
                    // hack the driver switch after 25 seconds
                    // by this time driver A will have moved for about 5 seconds
                    // thus only covered 3 co-ordinates
                    if (currentLocationIndex === 3) {
                        // do nothing for the 10 seconds for driver A
                        // add driver B co-ordinates now
                        self.locationCoordinates.splice(2, 0, [
                            -1.291879, 36.778389,
                        ]);
                        self.locationCoordinates.join();

                        self.driver = 'Driver B'; // driver b takes over from there
                    }

                    const currentLat = self.locationCoordinates[currentLocationIndex][0];
                    const currentLong = self.locationCoordinates[currentLocationIndex][1];

                    self.addMarker(currentLocationIndex, currentLat, currentLong, self.driver);

                    // get path
                    if (self.locationCoordinates.length > 1 && currentLocationIndex > 0) {
                        self.addPath(currentLocationIndex, currentLat, currentLong);
                    }

                    // inactivate the previous marker, (not active anymore)
                    if (currentLocationIndex > 0) {
                        self.markers[currentLocationIndex - 1].current_state = null;
                    }

                    self.mapConfig.center.lat = currentLat;
                    self.mapConfig.center.long = currentLong;

                    currentLocationIndex += 1;
                } else {
                    clearInterval(self.apicaller);
                }
            }, 5000);
        },

        addMarker(currentLocationIndex, currentLat, currentLong, driver) {
            const previousMarker = currentLocationIndex > 0 ? this.markers[currentLocationIndex - 1] : null;

            // markers
            this.markers.push({
                id: currentLocationIndex,
                current_state: 'active',
                driver,
                position:
                            {
                                lat: currentLat,
                                lng: currentLong,
                            },
                previous_position: {
                    lat: previousMarker ? previousMarker.position.lat : null,
                    lng: previousMarker ? previousMarker.position.lng : null,
                },
            });
        },

        addPath(currentLocationIndex, currentLat, currentLong) {
            this.lines.push(
                {
                    id: `${currentLocationIndex}l`,
                    path: [
                        {
                            lat: this.locationCoordinates[currentLocationIndex - 1][0],
                            lng: this.locationCoordinates[currentLocationIndex - 1][1],
                        },
                        {
                            lat: currentLat,
                            lng: currentLong,
                        },
                    ],
                },
            );
        },
    },

    mounted() {
        this.fetchLocations();
    },
};
</script>
