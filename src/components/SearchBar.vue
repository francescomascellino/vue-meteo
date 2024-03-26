<script>
export default {

    name: 'Searchbar',

    props: {

        query: String,
        suggestions: Object

    },

    emits: ['update:query', 'startSearch', 'suggest'],


    data() {

        return {


        }

    },

    methods: {

    },

    mounted() {

    }

}

</script>

<template>
    <h1 class="text-light text-center">Meteo</h1>

    <div class="m-2 text-light">

        <label for="geoSearch" class="form-label">Explore Forecast</label>

        <div class="fade-in input-group">
            <input type="search" name="geoSearch" id="geoSearch" class="form-control" placeholder="Search..."
                aria-describedby="helpId" @keyup.enter="$emit('startSearch')" :value="query"
                @input="$emit('update:query', $event.target.value); $emit('suggest')" list="suggestions" />

            <button class="input-group-text btn btn-light" @click="$emit('startSearch')">
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

</template>

<style scoped></style>
