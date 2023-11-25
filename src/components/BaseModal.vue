<template>
    <div class="modal" @click="((event) => closeModal(event))">
        <section class="modal__container">
            <div class="input__container">
                <font-awesome-icon :icon="'fa-solid fa-magnifying-glass'" class="input__icon"></font-awesome-icon>
                <input type="text" placeholder="Search for a Country..." class="input__text" v-model.trim="country"
                    v-debounce:1000ms="fetchData" />
                    <br/>
                    <input type="text" placeholder="Search for a city..." class="input__text" v-model.trim="location"
                    v-debounce:1000ms="fetchData" />
            </div>
            <div class="modal__menu">
                <div class="modal__item" @click="setCurrentLocation">
                    <font-awesome-icon :icon="'fa-solid fa-location-crosshairs'" class="item__icon"></font-awesome-icon>
                    <p class="item__text">Current Location</p>
                </div>
                <div class="modal__spinner" v-if="isSpinnerLoading">
                    <the-spinner></the-spinner>
                </div>
                <div class="modal__item" v-if="!isSpinnerLoading && results && results.length > 0" v-for="result in results"
                    :key="result.lat" @click="chooseLocation(result)">
                    <font-awesome-icon :icon="'fa-solid fa-location-dot'" class="item__icon"></font-awesome-icon>
                    <p class="item__text">{{ result.name }}, {{ result.state }}, {{ result.country }}</p>
                </div>
            </div>
        </section>
    </div>
</template>

<script>
import { vue3Debounce } from "vue-debounce"
import { getCurrentPosition } from "../utils/utils"
import TheSpinner from "./TheSpinner.vue"
import axios from 'axios'

export default {
    created() {
        this.appSettings = JSON.parse(localStorage.getItem('weather-app-settings'))
    },
    directives: {
        debounce: vue3Debounce({ lock: true })
    },
    emits: ['close-modal', 'refetch-data'],
    components: {
        TheSpinner
    },
    data() {
        return {
            isSpinnerLoading: false,
            location: null,
            country:null,
            results: [],
            appSettings: null
        }
    },
    methods: {
        closeModal(event) {
            const clickedComponent = event.target.classList[0]
            this.$emit('close-modal', clickedComponent)
        },
        async fetchData() {
            this.isSpinnerLoading = true;
            try {
                const locationsUrl = import.meta.env.VITE_GEOCODING_API_URL
                const apiKey = import.meta.env.VITE_OPEN_WEATHER_API_KEY
                if (this.location !== '') {
                    
                    const response = await axios.get(`${locationsUrl}/?appid=${apiKey}&query= New York`)
                    this.results = response.data
                } else {
                    this.results = []
                }
                this.isSpinnerLoading = false;
            } catch (error) {
                console.warn(error)
                this.isSpinnerLoading = false
            }
        },
        async chooseLocation(result) {
            this.isSpinnerLoading = true
            try {
                const weatherUrl = import.meta.env.VITE_WEATHER_API_URL
                const apiKey = import.meta.env.VITE_OPEN_WEATHER_API_KEY
                const response = await axios.get(`${weatherUrl}/current?access_key=${apiKey}&query= New York`)
                localStorage.setItem('weather', JSON.stringify({ ...response.data, lastUpdated: new Date() }))
                this.isSpinnerLoading = false
                this.$emit('refetch-data')
                this.$router.push('/weather')
            } catch (error) {
                console.warn(error)
                this.isSpinnerLoading = false
            }
        },
        async setCurrentLocation() {
            this.isSpinnerLoading = true
            try {
                const { coords } = await getCurrentPosition()
                const result = {
                    lat: coords.latitude,
                    lon: coords.longitude
                }
                this.isSpinnerLoading = false
                this.chooseLocation(result)
            } catch (error) {
                console.warn(error)
                this.isSpinnerLoading = false
            }
        }
    }
}
</script>

<style scoped>
.modal {
    width: 100%;
    height: 100%;
    position: fixed;
    background-color: rgba(0, 0, 0, .5);
    top: 0;
    left: 0;
    padding: 24px;
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal__container {
    flex: 1;
    max-width: 480px;
    background-color: var(--white);
    padding: 16px;
    border-radius: 16px;
    z-index: 9999;
}

.input__container {
    padding: 8px 16px;
    display: flex;
}

.input__icon {
    margin-inline-end: 16px;
}

.input__text {
    border: none;
    font-size: 14px;
    width: 100%;
}

.input__text:focus {
    outline: none;
}

.modal__menu {
    margin-block-start: 16px;
}

.modal__item {
    padding: 16px;
    color: var(--font-color);
    font-size: 14px;
    display: flex;
    cursor: pointer;
}

.modal__item:hover {
    background-color: var(--hover);
}

.item__icon {
    margin-inline-end: 16px;
}

.item__text {
    flex: 1;
}

.modal__spinner {
    display: flex;
    justify-content: center;
    padding: 16px;
}
</style>