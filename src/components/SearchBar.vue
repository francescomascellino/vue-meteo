<script>
export default {

    name: 'Searchbar',

    props: {

        query: String,
        suggestions: Object

    },

    emits: ['update:query', 'startSearch', 'suggest']
}

</script>

<template>
    <div class="m-2">

        <label for="geoSearch" class="form-label">Explore Forecast</label>

        <div class="fade-in input-group">
            <input type="search" name="geoSearch" id="geoSearch" class="glass-panel form-control rounded" placeholder="Search..."
                aria-describedby="helpId" @keyup.enter="$emit('startSearch')" :value="query"
                @input="$emit('update:query', $event.target.value); $emit('suggest')" list="suggestions" />

            <button class="small-btn input-group-text btn mx-2" @click="$emit('startSearch')" title="Search">
                <i class="fa-solid fa-magnifying-glass"></i>
            </button>
        </div>

        <datalist id="suggestions">
            <option v-for="suggestion in suggestions" :value="suggestion">
                {{ suggestion }}
            </option>
        </datalist>

        <small id="helpId">What the small will be like today?</small>

    </div>
</template>

<style scoped></style>
