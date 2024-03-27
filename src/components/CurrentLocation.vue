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
    <div class="col d-flex align-items-center p-0">

        <div class="saved-location glass-panel fade-in flex-grow-1 align-content-center rounded p-2" v-if="locationEnabled">
            <span class="ms-1">Your Location: {{ currentLocation }}</span>
        </div>

        <div class="location-container fade-in flex-grow-1 d-flex align-items-center rounded" v-else>
            <img class="no-gps" :src="getImagePath('../assets/img/nogps.png')" alt="Location disabled">
            <span>Location disabled!</span>
        </div>

        <div>
            <button class="small-btn fade-in btn mx-2" @click="$emit('startSearch')"
                v-if="locationEnabled"><i class="fa-solid fa-eye"></i></button>
        </div>

    </div>




</template>

<style scoped>
.no-gps {
    width: 18px;
    aspect-ratio: 1/1;
    filter: invert();
}
</style>
