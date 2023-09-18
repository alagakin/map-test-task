<template>
    <div class="flex space-x-4 items-center w-3/4 mx-auto mt-4">
        <div class="flex-1">
            <label for="latitude" class="block text-gray-700">Широта:</label>
            <input
                type="number"
                name="latitude"
                class="block w-full mt-1 p-2 border rounded-md focus:ring-indigo-500 focus:border-indigo-500"
                placeholder="Enter latitude"
                v-model="lat"
            />
        </div>
        <div class="flex-1">
            <label for="longitude" class="block text-gray-700">Долгота:</label>
            <input
                type="number"
                name="longitude"
                class="block w-full mt-1 p-2 border rounded-md focus:ring-indigo-500 focus:border-indigo-500"
                placeholder="Enter longitude"
                v-model="long"
            />
        </div>
        <div class="flex-1">
            <label for="zoom" class="block text-gray-700">Zoom:</label>
            <input
                type="number"
                name="zoom"
                class="block w-full mt-1 p-2 border rounded-md focus:ring-indigo-500 focus:border-indigo-500"
                placeholder="Enter zoom level"
                v-model="zoom"
            />
        </div>
        <div class="flex-1">
            <button
                @click="search"
                type="button"
                class="w-full bg-indigo-500 text-white p-2 rounded-md hover:bg-indigo-600 focus:ring-indigo-500 focus:border-indigo-500 mt-7"
            >
                Поиск
            </button>
        </div>
    </div>
    <div class="text-center border-blue-500 border w-1/3 mx-auto mt-10 h-64 rounded-2xl flex justify-center items-center">
        <span class="text-xl text-red-500" v-show="error" v-text="error"></span>
        <img :src="data" alt="" v-if="!error && data" class="mx-auto">
    </div>

</template>

<script>
import axios from "axios";

export default {
    name: 'App',
    data() {
        return {
            lat: 55.756038,
            long: 37.518560,
            zoom: 16,
            data: null,
            error: false
        }
    },
    methods: {
        checkData() {
            if (this.zoom > 30 || this.zoom < 1) {
                this.error = "Zoom не может быть больше 30 и меньше 1"
                return false
            }
            if (this.lat > 90 || this.lat < -90) {
                this.error = "Диапазон значений широты: от -90° до 90°."
                return false
            }
            if (this.long > 180 || this.long < -180) {
                this.error = "Диапазон значений долготы: от -180° до 180°."
                return false
            }
            return true
        },
        search() {
            if (!this.checkData()) {
                this.data = null
                return
            }
            const queryParams = {
                l: "carparks",
                x: this.x,
                y: this.y,
                z: this.zoom,
                lang: 'ru_RU',
            };
            axios.get('https://core-carparks-renderer-lots.maps.yandex.net/maps-rdr-carparks/tiles', {
                params: queryParams,
                responseType: "blob"
            }).then(response => {
                if (response.data.size) {
                    this.data = URL.createObjectURL(response.data)
                    this.error = false
                } else {
                    this.data = null
                    this.error = "Ничего не найдено"
                }
            }).catch(error => {
                this.error = "Ничего не найдено"
                this.data = null
                console.log(error)
            })
        },
    },
    computed: {
        x() {
            const numTiles = Math.pow(2, this.zoom);
            return Math.floor((this.long + 180) / 360 * numTiles);
        },
        y() {
            const numTiles = Math.pow(2, this.zoom);
            const latRad = this.lat * Math.PI / 180;
            return Math.floor((1 - Math.log(Math.tan(latRad) + 1 / Math.cos(latRad)) / Math.PI) / 2 * numTiles);
        },
    }
}
</script>
<style scoped>
@import "style.css";
</style>