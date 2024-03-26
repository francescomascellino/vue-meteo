<script>
import SearchBar from './components/SearchBar.vue';

import MeteoPanel from './components/MeteoPanel.vue';

import SavedLocations from './components/SavedLocations.vue';

import CurrentLocation from './components/CurrentLocation.vue';

export default {
    name: 'App',

    components: {
        SearchBar,
        CurrentLocation,
        MeteoPanel,
        SavedLocations
    },

    created() {

    },

    data() {
        return {

            loading: false,
            locationEnabled: false,
            inputTimer: null,
            savedLocations: [],
            geoSearchInput: null,
            currentLocation: '',

            // TOMTOM API DATA
            TomTomAPI_KEY: 'YOUR API KEY HERE',
            latitude: '',
            longitude: '',
            suggestions: [],

            // OPENWATHER API DATA
            API_KEY: 'YOUR API KEY HERE',
            units: 'metric',
            lang: 'it',
            location: '',
            description: '',
            temperature: '',
            icon: '',

        }
    },

    methods: {

        // CAPITALIZER
        capitalize(string) {
            const words = string.split(" ");
            return words.map((word) => {
                return word.charAt(0).toUpperCase() + word.slice(1);
            }).join(" ");
        },

        // LOCAL WEATHER
        async onSuccess(position) {
            // GETS LOCAL COORDINATES FROM NAVIGATOR
            this.latitude = position.coords.latitude;
            this.longitude = position.coords.longitude;

            // TOM TOM GETS THE LOCATION NAME USED AS STANDARD USING REVERSE GEOCODING TO MATCH THE AUTOCOMPLETED SUGGESTION AND BETTER PRECISION
            const reverseGeocode = await fetch(`https://api.tomtom.com/search/2/reverseGeocode/crossStreet/${this.latitude}%2C${this.longitude}.json?limit=1&radius=10000&allowFreeformNewLine=false&view=Unified&key=${this.TomTomAPI_KEY}`);

            const reverseGeocodeData = await reverseGeocode.json();

            // SAVES THE CURRENT LOCATION USING AS STANDARD MUNICIPALITY, COUNTRY
            this.currentLocation = reverseGeocodeData.addresses[0].address.municipality + ', ' + reverseGeocodeData.addresses[0].address.country;
            console.log(this.currentLocation);

            this.location = this.currentLocation;

            const owURL = `https://api.openweathermap.org/data/2.5/weather?lat=${this.latitude}&lon=${this.longitude}&appid=${this.API_KEY}&units=${this.units}&lang=${this.lang}`;

            const response = await fetch(owURL);

            const data = await response.json();

            // WE TAKE FROM THE OW API ONLY METEO DATA NEEDED. (WE USE TOM TOM TO GET THE LOCATION NAME)
            this.temperature = `${Math.floor(data.main.temp)}°`

            this.description = this.capitalize(data.weather[0].description);

            this.icon = data.weather[0].icon;

            console.log('LOCATION =', this.location, '- WEATHER =', this.description, '- TEMP =', this.temperature);

            this.loading = false;
        },

        onError() {
            this.locationEnabled = false;
            console.log('Please enable location sharing in your browser settings to use the weather app');
        },

        //SEARCHS WITH GEOCODE
        async geoSearch(location) {
            console.log('Geocoding...', location);

            this.loading = true;

            const [municipality, country] = location.split(', ');
            console.log('Municipality: ', municipality);

            console.log('Country', country);

            const geoResponse = await fetch(`https://api.tomtom.com/search/2/geocode/${location}.json?key=${this.TomTomAPI_KEY}&limit=100`);

            const data = await geoResponse.json();

            console.log('Geocoding results: ', data.results);

            let found = false;

            data.results.forEach(item => {
                if (item.address.municipality === municipality && item.address.country === country) {
                    this.latitude = item.position.lat;
                    this.longitude = item.position.lon;
                    this.location = item.address.municipality + ', ' + item.address.country;
                    found = true;
                    console.log('FOUND: ', item.address.municipality, item.address.country, 'Lat: ', this.latitude, 'Long: ', this.longitude);
                }
            });

            if (!found) {
                this.latitude = data.results[0].position.lat;
                this.longitude = data.results[0].position.lon;
                this.location = data.results[0].address.municipality + ", " + data.results[0].address.country;
                console.log('USING NEAREST MATCH: ', data.results[0].address.municipality, data.results[0].address.country);
            }

            const weatherResponse = await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${this.latitude}&lon=${this.longitude}&appid=${this.API_KEY}&units=${this.units}&lang=${this.lang}`);

            const weatherData = await weatherResponse.json();

            this.temperature = `${Math.floor(weatherData.main.temp)}°`;

            this.description = this.capitalize(weatherData.weather[0].description);

            this.icon = weatherData.weather[0].icon;

            console.log('GEOCODED LOCATION =', this.location, '- TEMP =', this.temperature, '- WEATHER =', this.capitalize(this.description));

            console.log('This location is bookmarked? ', this.bookmarked);

            this.loading = false;
            this.geoSearchInput = null;
        },

        // SUGGESTS LOCATIONS
        async suggestLocations() {
            if (this.geoSearchInput.trim().length > 0) {

                const suggestionsResponse = await fetch(`https://api.tomtom.com/search/2/search/${this.geoSearchInput}.json?key=${this.TomTomAPI_KEY}`);

                const suggestionsData = await suggestionsResponse.json();

                const suggestionsResults = suggestionsData.results;

                const suggestionsSet = new Set();

                suggestionsResults.forEach(suggestion => {

                    if (suggestion.address.municipality && suggestion.address.country) {

                        console.log('Suggestion: ', suggestion.address.municipality + ', ' + suggestion.address.country);

                        suggestionsSet.add(suggestion.address.municipality + ', ' + suggestion.address.country);

                    }

                });

                this.suggestions = Array.from(suggestionsSet);

            }
        },

        async handleInputDebounced() {

            clearTimeout(this.inputTimer);

            this.inputTimer = setTimeout(() => {

                this.suggestLocations();

            }, 250);
        },

        async suggestLocationsDebounced() {

            if (this.geoSearchInput.trim().length > 0) {

                this.handleInputDebounced();

            }
        },

        // SAVES A LOCATION
        saveLocation() {

            if (this.savedLocations.length < 4) {

                // GET A BOOLEAN VALUE TO CHECK IF THE LOCATION IS ALREADY SAVED
                const isSaved = this.savedLocations.some((location) => location.name === this.location);

                // IF THE LOCATION HAS NOT ALREADY BEEN SAVED
                if (!isSaved) {
                    console.log('Saving Geocoded Location ', this.location, ' in local storage');

                    this.savedLocations.push(
                        {
                            name: this.location,
                            lat: this.latitude,
                            lon: this.longitude
                        }
                    )

                    console.log(this.savedLocations);

                    localStorage.setItem("savedLocations", JSON.stringify(this.savedLocations));

                } else {
                    console.log("Location already saved");
                }

            } else {
                console.log("You can save only 4 locations");
            }

        },

        // DELETES A LOCATION
        deleteLocation(location) {
            console.log("deleting ", location);

            // REMOVES THE LOCATION FROM THE DAVED LOCATIONS ARRAY
            this.savedLocations.splice(this.savedLocations.indexOf(location), 1);

            // UPDATES LOCAL STORAGE DATA WITH LOCAL DATA
            localStorage.setItem("savedLocations", JSON.stringify(this.savedLocations));

            // IF SAVED LOCATIONS ARRAY IS EMPTY, DELETES THE LOCAL STORAGE ITEM
            if (this.savedLocations.length == 0) {
                localStorage.removeItem("savedLocations");
            }
        },

    },

    mounted() {

        // NAVIGATOR INSTANCE
        navigator.geolocation.getCurrentPosition(this.onSuccess, this.onError);

        // CHECKS GEOLOCATION PERMISSIONS
        if ("geolocation" in navigator) {
            navigator.permissions.query({ name: 'geolocation' }).then(permissionStatus => {
                console.log("Geolocation: ", permissionStatus);

                permissionStatus.onchange = () => {
                    console.log("Permission changed to " + permissionStatus.state);
                    this.locationEnabled = permissionStatus.state === "granted" ? true : false;
                }

                this.locationEnabled = permissionStatus.state === "granted" ? true : false;
                console.log("Localization enabled? ", this.locationEnabled);

            });
        } else {
            console.log('Geolocation is not supported by this browser.');
        }

        // PARSE LOCATIONS FROM LOCALSTORAGE
        if (localStorage.getItem("savedLocations")) {
            this.savedLocations = JSON.parse(localStorage.getItem("savedLocations"));
        } else {
            // EMPTIES THE SAVED LOCATIONS ARRAY IF THERE ARE NO LOCATIONS SAVED IN THE STORAGE
            this.savedLocations = [];
        }
    },

    computed: {
        // DINAMICALLY CHECKS IF BOOKMARKS CONTAIN A LOCATION
        bookmarked() {
            return this.savedLocations.some(location => location.name === this.location);
        }
    },

}
</script>

<template>
    <!-- SEARCHBAR -->
    <!-- Per impostazione predefinita, v-model su un componente utilizza modelValue come prop e update:modelValue come evento. Possiamo modificare questi nomi passando un argomento a v-model: -->
    <SearchBar v-model:query="geoSearchInput" @startSearch="geoSearch(geoSearchInput)"
        @suggest="suggestLocationsDebounced()" :suggestions="suggestions" />
    <!-- In questo caso, il componente figlio dovrebbe aspettarsi una prop query ed emettere un evento update:query per aggiornare il valore nel componente genitore -->

    <div class="container-fluid text-light">

        <div class="row flex-column p-2 gy-2">

            <!-- CURRENT LOCATION -->
            <CurrentLocation :locationEnabled="locationEnabled" :currentLocation="currentLocation"
                @startSearch="geoSearch(currentLocation)" />

            <!-- METEO PANEL -->
            <MeteoPanel :loading="this.loading" :temperature="this.temperature" :icon="icon" :description="description"
                :location="location" :bookmarked="bookmarked" @saveLocation="saveLocation" />

            <!-- SAVED LOCATIONS LIST -->
            <SavedLocations :savedLocations="savedLocations" @deleteLocation="deleteLocation(location)"
                @startSearch="geoSearch($event)" />

        </div>
    </div>

    <footer class="text-light p-2">

        <p>
            Gps navigation icons created by <a href="https://www.flaticon.com/free-icons/gps-navigation"
                title="gps navigation icons"> Didin jpr - Flaticon</a>
        </p>

        <p>
            Geolocation provvided by <a href="https://developer.tomtom.com/">TomTom API</a>
        </p>

        <p>
            Weather data provided by <a href="https://openweathermap.org/api">Open Weather API</a>.
        </p>

    </footer>

</template>

<style></style>

<!-- ORIGINAL CODE -->

<!-- SEARCHBAR  -->
<!--     
    <div class="m-2 text-light">

        <label for="geoSearch" class="form-label">Explore Forecast</label>

        <div class="fade-in input-group">
            <input type="search" name="geoSearch" id="geoSearch" class="form-control" placeholder="Search..."
                aria-describedby="helpId" @keyup.enter="geoSearch(geoSearchInput)" @input="suggestLocationsDebounced()"
                v-model="geoSearchInput" list="suggestions" />

            <button class="input-group-text btn btn-light" @click="geoSearch(geoSearchInput)">
                <i class="fa-solid fa-magnifying-glass"></i>
            </button>
        </div>

        <datalist id="suggestions">
            <option v-for="suggestion in suggestions" :value="suggestion">
                {{ suggestion }}
            </option>
        </datalist>

        <small id="helpId" class="text-light">What the weather will be like today?</small>

    </div> 
-->

<!-- CURRENT LOCATION -->
<!-- 
<div class="col d-flex align-items-center border rounded p-0">

    <div class="saved-location fade-in flex-grow-1 align-content-center ms-2" v-if="locationEnabled">
        <span>Your Location: {{ currentLocation }}</span>
    </div>

    <div class="fade-in flex-grow-1 ms-2 location-container" v-else>
        <img class="no-gps" :src="getImagePath('./assets/img/nogps.png')" alt="Location Unavaiable">
        <span>Location disabled!</span>
    </div>

    <button class="fade-in btn btn-light ms-auto" @click="geoSearch(currentLocation)"
        v-if="locationEnabled">View</button>

</div>
-->

<!-- METEO PANEL -->
<!--             
    <div class="weather-card col d-flex justify-content-center align-items-center flex-column border rounded ">

                <template v-if="!loading">

                    <h1 class="fade-in">{{ this.temperature }}</h1>

                    <img class="weather-ico fade-in" :src="`https://openweathermap.org/img/wn/${this.icon}@2x.png`"
                        alt="{{ this.description }}">

                    <h5 class="fade-in">{{ this.location }}</h5>

                    <h5 class="fade-in">{{ this.description }}</h5>

                    <button class="fade-in btn btn-success m-2" @click="saveLocation()"><i
                            class="fa-regular fa-bookmark" v-if="!bookmarked"></i> <i class="fa-solid fa-bookmark"
                            v-else></i></button>

                </template>

                <template v-else>
                    <div class="loader"></div>
                </template>

            </div> 
-->

<!-- SAVED LOCATIONS LIST -->
<!-- 
            <h3>Your Bookmarks:</h3>
            <div class="col d-flex align-items-center border rounded p-0" v-if="savedLocations.length > 0"
                v-for="location in savedLocations">

                <div class="saved-location fade-in flex-grow-1 align-content-center ms-2"
                    @click="geoSearch(location.name)">
                    <span>{{ location.name }}</span>
                </div>

                <button class="fade-in btn btn-danger ms-auto" @click="deleteLocation(location)"><i
                        class="fa-solid fa-trash"></i></button>

            </div>

            <div class="col text-center border rounded p-2" v-if="savedLocations.length == 0">
                <span class="location-container">You have not saved any locations.</span>
            </div>

            <div class="saved-locations-counter col d-flex align-items-center rounded p-0">

                <template v-if="savedLocations.length === 0">
                    <span class="fade-in text-success">You can save up to 4 locations</span>
                </template>

                <template v-else-if="savedLocations.length < 4">
                    <span class="fade-in text-warning">You can save {{ 4 - savedLocations.length }} more location{{
                    avedLocations.length < 3 ? 's' : '' }}</span>
                </template>

                <template v-else>
                    <span class="fade-in text-danger">Your saved locations slots are full</span>
                </template>

            </div>
-->
