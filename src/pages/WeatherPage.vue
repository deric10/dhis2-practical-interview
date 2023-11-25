<template>
    <section class="page__container">
        <div class="page">
            <div class="page__top-container" :class="determineTime">
                <div class="page__top">
                    <div class="page__actions">
                        <div class="page__location" @click="toggleModal">
                            <font-awesome-icon :icon="'fa-solid fa-location-dot'"
                                class="location__icon"></font-awesome-icon>
                            <p class="location__text">{{ information.name }}</p>
                        </div>
                        <font-awesome-icon :icon="'fa-solid fa-rotate'" :class="['page__reload', dataRefetchingAnimation]"
                            @click="refetchData"></font-awesome-icon>
                    </div>
                    <div class="page__summary">
                        <div class="page__info">
                            <div class="summary__info">
                                <h2 class="summary__temperature">{{ roundTemp(information.main.temp) }}</h2>
                                <p class="summary__feels">Feels like {{ roundTemp(information.main.feels_like) }}</p>
                                <p class="summary__description">{{ information.weather[0].description }}</p>
                            </div>
                            <img class="summary__icon" :src="iconUrl" alt="Weather icon">
                        </div>
                        <p class="page__last-updated">{{ lastUpdated }}</p>
                    </div>

                </div>
            </div>
            <div :class="['page__bottom-container', determineBackgroundMobile]">
                <div class="page__bottom">
                    <div class="bottom__header">
                        <h3 class="bottom__title">today</h3>
                    </div>
                    <div class="bottom__body">
                        <div class="body__item">
                            <p class="item__title">Max</p>
                            <p class="item__value">{{ roundTemp(information.main.temp_max) }}</p>
                        </div>
                        <div class="body__item">
                            <p class="item__title">Min</p>
                            <p class="item__value">{{ roundTemp(information.main.temp_min) }}</p>
                        </div>
                        <div class="body__item">
                            <p class="item__title">Humidity</p>
                            <p class="item__value">{{ `${information.main.humidity}%` }}</p>
                        </div>
                        <div class="body__item">
                            <p class="item__title">Sunrise</p>
                            <p class="item__value">{{ convertTime(information.sys.sunrise) }} AM</p>
                        </div>
                        <div class="body__item">
                            <p class="item__title">Sunset</p>
                            <p class="item__value">{{ convertTime(information.sys.sunset) }} PM</p>
                        </div>
                        <div class="body__item">
                            <p class="item__title">Pressure</p>
                            <p class="item__value">{{ `${information.main.pressure}hPa` }}</p>
                        </div>
                        <div class="body__item">
                            <p class="item__title">Visibility</p>
                            <p class="item__value">{{ information.visibility }}m</p>
                        </div>
                        <div class="body__item">
                            <p class="item__title">Wind</p>
                            <p class="item__value">{{ convertWind(information.wind.speed) }}</p>
                        </div>
                        <div class="body__item">
                            <p class="item__title">Weather</p>
                            <p class="item__value">{{ information.weather[0].main }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <transition name="modal">
            <base-modal v-if="isModalOpen" @close-modal="toggleModal" @refetch-data="updatePage"></base-modal>
        </transition>
    </section>
</template>

<script>
import axios from 'axios'

import BaseModal from '../components/BaseModal.vue'

export default {
    components: {
        BaseModal
    },
    created() {
        this.information = JSON.parse(localStorage.getItem('weather-app-data'))
        this.appSettings = JSON.parse(localStorage.getItem('weather-app-settings'))
        this.checkLastUpdated()
        setInterval(() => {
            this.checkLastUpdated()
        }, 10000)
    },
    mounted() {
        this.updateScreenWidth()
        this.onScreenResize()
    },
    data() {
        return {
            appSettings: null,
            information: null,
            isModalOpen: false,
            isDataRefetching: false,
            screenWidth: 0,
            lastUpdated: ''
        }
    },
    methods: {
        roundTemp(temp) {
            const settings = JSON.parse(localStorage.getItem('weather-app-settings'))
            if (settings.units_of_measure === 'metric') {
                return `${Math.round(Number(temp))}°C`
            } else if (settings.units_of_measure === 'imperial') {
                return `${Math.round(Number(temp))}℉`
            } else if (settings.units_of_measure === 'standard') {
                return `${Math.round(Number(temp))}K`
            }
        },
        convertTime(time) {
            const convertedTime = new Date(time)
            return `${convertedTime.getHours()}:${convertedTime.getMinutes()}`
        },
        convertWind(windSpeed) {
            const settings = JSON.parse(localStorage.getItem('weather-app-settings'))
            if (settings.units_of_measure === 'metric' || settings.units_of_measure === 'standard') {
                return `${windSpeed}m/s`
            } else {
                return `${windSpeed}miles/h`
            }
        },
        toggleModal(clickedComponent) {
            this.isModalOpen = true

            if (this.isModalOpen === true && clickedComponent === 'modal') {
                this.isModalOpen = false
            }
        },
        updatePage() {
            this.$router.go()
        },
        async refetchData() {
            this.isDataRefetching = true
            try {
                const weatherUrl = import.meta.env.VITE_WEATHER_API_URL
                const apiKey = import.meta.env.VITE_OPEN_WEATHER_API_KEY
                const response = await axios.get(`${weatherUrl}/?appid=${apiKey}&query= New York`)
                localStorage.setItem('weather-app-data', JSON.stringify({ ...response.data, lastUpdated: new Date() }))
                this.isDataRefetching = false
                this.information = JSON.parse(localStorage.getItem('weather-app-data'))
                this.checkLastUpdated()
            } catch (error) {
                console.warn(error)
                this.isDataRefetching = false
            }
        },
        onScreenResize() {
            window.addEventListener('resize', () => {
                this.updateScreenWidth()
            })
        },
        updateScreenWidth() {
            this.screenWidth = window.innerWidth
        },
        checkLastUpdated() {
            const currentDate = new Date()
            const lastUpdated = new Date(this.information.lastUpdated)
            const difference = currentDate - lastUpdated
            const differenceInMinutes = Math.round(((difference / 1000) / 60))
            if (differenceInMinutes === 0 || differenceInMinutes === 1) {
                this.lastUpdated = 'Updated 1 minute ago'
            } else if (differenceInMinutes > 0 && differenceInMinutes < 60) {
                this.lastUpdated = `Updated ${differenceInMinutes} minutes ago`
            } else if (differenceInMinutes >= 60) {
                const differenceInHours = Math.round(differenceInMinutes / 60)
                differenceInHours === 1 ? this.lastUpdated = 'updated 1 hour ago' : this.lastUpdated = `Updated ${differenceInHours} hours ago`
            }
        }
    },
    computed: {
        iconUrl() {
            const iconsUrl = import.meta.env.VITE_WEATHER_ICONS_URL
            return `${iconsUrl}/${this.information.weather[0].icon}@2x.png`
        },
        determineTime() {
            const weatherIconCode = this.information.weather[0].icon
            const weatherIconCodeToArray = weatherIconCode.split('')
            if (weatherIconCodeToArray[weatherIconCodeToArray.length - 1] === 'd') {
                return 'top__scheme--day'
            } else {
                return 'top__scheme--night'
            }
        },
        dataRefetchingAnimation() {
            if (this.isDataRefetching) {
                return 'data-refetching'
            }
        },
        determineBackgroundMobile() {
            const weatherIconCode = this.information.weather[0].icon
            const weatherIconCodeToArray = weatherIconCode.split('')
            if (this.screenWidth < 1024 && weatherIconCodeToArray[weatherIconCodeToArray.length - 1] === 'd') {
                return 'top__scheme--day'
            } else if (this.screenWidth < 1024 && weatherIconCodeToArray[weatherIconCodeToArray.length - 1] === 'n') {
                return 'top__scheme--night'
            } else if (this.screenWidth >= 1024) {
                return 'default'
            }
        }
    }
}
</script>

<style scoped>
.page {
    display: flex;
    flex-direction: column;
    height: 100%;
}


.page__top-container {
    flex: 1;
}

.page__top {
    height: 100%;
    max-width: 1200px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}


.page__bottom-container {
    flex: 1;
}

.page__bottom {
    height: 100%;
    max-width: 1200px;
    margin: 0 auto;
    padding-block-start: 16px;
    border-top-left-radius: 16px;
    border-top-right-radius: 16px;
    background-color: var(--white);
}

.page__actions {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16px 24px;
}

.page__location {
    display: flex;
    align-items: center;
    padding: 8px;
}

.location__icon,
.page__reload {
    width: 21px;
    height: 21px;
}

.location__text {
    font-size: 14px;
    margin-inline-start: 8px;
}

.page__reload {
    padding: 8px;
}

.page__info {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;

}

.page__summary {
    padding: 0px 32px 32px;
    height: 100%;
}

.page__last-updated {
    opacity: 0.5;
    margin-block-start: 32px;
    font-size: 12.25px;
}

.summary__temperature {
    font-size: 64px;
    font-weight: 500;
    margin-block-end: 16px;
}

.summary__feels,
.summary__description {
    font-size: 12.25px;
    text-transform: capitalize;
}

.summary__feels {
    margin-block-end: 5px;
}


.top__scheme--day {
    background-color: var(--primary);
    color: var(--white);
}

.top__scheme--night {
    background-color: #44296a;
    color: var(--white)
}

.bottom__header {
    padding: 16px 32px;
}

.bottom__title {
    text-transform: uppercase;
    font-size: 12.25px;
    color: var(--black)
}

.bottom__body {
    height: calc(100% - 46px);
    padding-inline: 32px;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
    gap: 16px;
}

.body__item {
    font-size: 12.25px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.item__title {
    color: var(--font-color-meta);
    margin-block-end: 5px;
    text-transform: capitalize;
}

.item__value {
    color: var(--font-color);
}

.modal-enter-active {
    animation: fade .3s ease;
}

.modal-leave-active {
    animation: fade .3s ease reverse;
}

.data-refetching {
    animation: rotation 1s linear infinite;
}

.default {
    background-color: var(--white);
}

@keyframes fade {
    from {
        opacity: 0;
    }

    to {
        opacity: 1;
    }
}

@keyframes rotation {
    0% {
        transform: rotate(0deg);
    }

    100% {
        transform: rotate(360deg);
    }
}

@media screen and (min-width: 1024px) {

    .page__location:hover,
    .page__reload:hover {
        background-color: var(--hover);
        color: var(--black);
        border-radius: 5px;
        cursor: pointer;
    }

    .summary__temperature {
        font-size: 82px;
    }

    .location__text,
    .summary__feels,
    .summary__description,
    .page__last-updated,
    .bottom__title,
    .body__item {
        font-size: 16px;
    }

    .page__bottom {
        border-top-right-radius: 0px;
        border-top-left-radius: 0px;
    }
}
</style>