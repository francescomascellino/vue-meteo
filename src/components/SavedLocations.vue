<script>

export default {

    name: 'SavedLocations',

    props: {

        savedLocations: Array

    },

    emits: ['deleteLocation', 'startSearch']
}
</script>

<template>
    <h3>Your Bookmarks:</h3>
    <div class="col d-flex align-items-center p-0" v-if="savedLocations.length > 0"
        v-for="location in savedLocations">

        <div class="saved-location glass-panel fade-in flex-grow-1 align-content-center rounded"
            @click="$emit('startSearch', location.name)" title="View">
            <span class="ms-2">{{ location.name }}</span>
        </div>

        <button class="small-btn fade-in btn mx-2" @click="$emit('deleteLocation', location.name)" title="Delete"><i
                class="fa-solid fa-trash"></i></button>

    </div>

    <div class="glass-panel col text-center rounded p-2" v-if="savedLocations.length == 0">
        <span class="location-container">You have not saved any locations.</span>
    </div>

    <div class="saved-locations-counter col d-flex align-items-center rounded p-0">

        <template v-if="savedLocations.length === 0">
            <span class="fade-in text-success">You can save up to 4 locations</span>
        </template>

        <template v-else-if="savedLocations.length < 4">
            <span class="fade-in">You can save {{ 4 - savedLocations.length }} more location{{
        savedLocations.length < 3 ? 's' : '' }}</span>
        </template>

        <template v-else>
            <span class="fade-in text-danger">Your saved locations slots are full</span>
        </template>

    </div>
</template>

<style scoped>
.saved-location {
    cursor: pointer;
    height: 38px;
}

.saved-locations-counter {
    font-size: smaller;
}
</style>
