<script>

export default {

    name: 'CurrentLocation',

    props: {

        locationEnabled: Boolean,
        currentLocation: String

    },

    emits: ['startSearch'],


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
    <div class="col d-flex align-items-center border rounded p-0">

        <div class="saved-location fade-in flex-grow-1 align-content-center ms-2" v-if="locationEnabled">
            <span>Your Location: {{ currentLocation }}</span>
        </div>

        <div class="location-container fade-in flex-grow-1 d-flex align-items-center ms-2 " v-else>
            <img class="no-gps" :src="getImagePath('../assets/img/nogps.png')" alt="Location disabled">
            <span>Location disabled!</span>
        </div>

        <button class="fade-in btn btn-light ms-auto" @click="$emit('startSearch')" v-if="locationEnabled">View</button>

    </div>
</template>

<style scoped>
.no-gps {
    width: 18px;
    aspect-ratio: 1/1;
    filter: invert();
}
</style>
