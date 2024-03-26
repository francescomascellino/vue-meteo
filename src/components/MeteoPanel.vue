<script>

export default {

    name: 'MeteoPanel',

    props: {

        loading: Boolean,
        locationEnabled: Boolean,
        temperature: String,
        icon: String,
        description: String,
        location: String,
        bookmarked: Boolean

    },

    emits: ['saveLocation'],


    data() {

        return {


        }

    },

    methods: {
        // GET IMAGES RELATIVE PATH
        getImagePath(url) {
            return new URL(`${url}`, import.meta.url).href
        },
    },

    mounted() {

    }

}
</script>

<template>
    <div
        class="weather-card col d-flex justify-content-center align-items-center flex-column position-relative border rounded">

        <template v-if="!loading && location">

            <h1 class="fade-in position-absolute top-0 start-0 ms-2" v-if="bookmarked"><i
                    class="fa-solid fa-bookmark"></i></h1>

            <h1 class="fade-in">{{ temperature }}</h1>

            <img class="weather-ico fade-in" :src="`https://openweathermap.org/img/wn/${icon}@2x.png`"
                alt="{{ description }}">

            <h5 class="fade-in">{{ location }}</h5>

            <h5 class="fade-in">{{ description }}</h5>

            <button class="fade-in btn btn-success m-2" @click="$emit('saveLocation')" v-if="!bookmarked"><i
                    class="fa-regular fa-bookmark"></i></button>



        </template>

        <template v-else-if="!locationEnabled && !loading">
            <img class="no-gps" :src="getImagePath('../assets/img/nogps.png')" alt="Location disabled">
            <h5 class="text-center">Please enable location sharing to automatically retrieve weather data for your area.
            </h5>
        </template>

        <template v-else>
            <div class="loader"></div>
        </template>

    </div>
</template>

<style scoped>
.weather-card {
    min-width: 304px;
    min-height: 304px;

    h1,
    h2,
    h3,
    h4,
    h5 {
        margin: 5px 0;
    }
}

.w {
    min-height: 304px;
}

.no-gps {
    width: 100px;
    aspect-ratio: 1/1;
    filter: invert();
}

.loader {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    border: 16px solid;
    border-color: #E4E4ED;
    border-right-color: #766DF4;
    animation: loading 1.5s infinite linear;
}

@keyframes loading {
    to {
        transform: rotate(1turn)
    }
}
</style>
