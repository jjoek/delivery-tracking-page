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
    import {ACTIVE_POINT_MARKER_ICON_CONFIG, mapSettings, POINT_MARKER_ICON_CONFIG} from '@/constants/mapSettings';
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
                driver:'Driver A',
            };
        },

        computed: {
            mapConfig() {
                return {
                    ...mapSettings,
                    center: { lat: -1.298982, lng: 36.776811 },
                };
            },

            locationCoordinates () {
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
                ]
            },

            mapCenter() {
                return this.markers[1].position;
            },
        },

        methods: {
            fetchLocations () {
                let current_location_index = 0

                let self = this;

                self.apicaller = setInterval(function () {
                    if(self.locationCoordinates && (current_location_index < self.locationCoordinates.length)) {
                        // hack the driver switch after 25 seconds
                        // by this time driver A will have moved for about 5 seconds
                        // thus only covered 3 co-ordinates
                        if(current_location_index === 3) {
                            // do nothing for the 10 seconds for driver A
                            // add driver B co-ordinates now
                            self.locationCoordinates.splice(2, 0, [
                                -1.291879, 36.778389
                            ]);
                            self.locationCoordinates.join()

                            self.driver = 'Driver B' // driver b takes over from there
                        }

                        let current_lat = self.locationCoordinates[current_location_index][0];
                        let current_long = self.locationCoordinates[current_location_index][1];

                        self.addMarker(current_location_index, current_lat, current_long, self.driver);

                        // get path
                        if(self.locationCoordinates.length > 1 && current_location_index > 0) {
                            self.addPath(current_location_index, current_lat, current_long);
                        }

                        // inactivate the previous marker, (not active anymore)
                        if(current_location_index > 0) {
                            self.markers[current_location_index - 1].current_state = null;
                        }

                        self.mapConfig.center.lat = current_lat;
                        self.mapConfig.center.long = current_long;

                        current_location_index++;
                    } else {
                        clearInterval(self.apicaller);
                    }
                }, 5000)
            },

            addMarker (current_location_index, current_lat, current_long, driver) {

                let previous_marker = current_location_index > 0 ? this.markers[current_location_index - 1] : null;

                // markers
                this.markers.push({
                        id: current_location_index,
                        current_state: 'active',
                        driver: driver,
                        position:
                            {
                                lat: current_lat,
                                lng: current_long,
                            },
                        previous_position: {
                            lat: previous_marker ? previous_marker.position.lat : null,
                            lng: previous_marker ? previous_marker.position.lng : null
                        }
                    },
                );
            },

            addPath (current_location_index, current_lat, current_long) {
                this.lines.push(
                    {
                        id: current_location_index + 'l',
                        path: [
                            {
                                lat: this.locationCoordinates[current_location_index - 1][0],
                                lng: this.locationCoordinates[current_location_index - 1][1]
                            },
                            {
                                lat: current_lat,
                                lng: current_long
                            }
                        ]
                    },
                )
            }
        },

        mounted () {
            this.fetchLocations();
        }
    };
</script>
