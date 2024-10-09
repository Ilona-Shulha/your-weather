<template>
  <div>
    <div class="weather-timeline-wrapper">
      <div class="weather-timeline">
        <WeatherTimelineItem v-for="timeWeather in weatherData" :weatherData="timeWeather" :key="timeWeather.time" class="weather-timeline-item"/>
      </div>
    </div>
    <TempChart :chartTime="chartTime" :chartTemp="chartTemp" class="weather-chart" />
  </div>
</template>
<script setup>
  import { ref, onMounted, computed, watch } from 'vue';
  import WeatherTimelineItem from '@/components/WeatherTimelineItem.vue';
  import TempChart from '@/components/TempChart.vue';

  const props = defineProps(['weatherDataList', 'forecastType'])
  const weatherData= ref([]);

  onMounted(() => {
    initWeatherData();
  });

  watch(()=> props.forecastType, () => {
    initWeatherData();
  })

  const chartTime = computed(() => {
    return weatherData.value?.map(data => data.time)
  });
  const chartTemp = computed(() => {
    return weatherData.value?.map(data => data.temp);
  });

  const initWeatherData = () => {
    props.forecastType === 'daily'
      ? formatedDayWeather(props.weatherDataList)
      : formatedHourlyWeather(props.weatherDataList)
  };

  const formatedHourlyWeather = (data) => {
    const currentDate = new Date().getDate();
    weatherData.value = [];
    data.slice(0, 9).forEach(hourWeather => {
      if (+hourWeather.dt_txt.slice(8, 11) !== +currentDate) return;
      
      weatherData.value.push({
      time: hourWeather.dt_txt.slice(-8,-3),
      weatherState: hourWeather.weather[0].description,
      weatherDescription: hourWeather.weather[0].main,
      wind: hourWeather.wind.speed,
      temp: hourWeather.main.temp,
    })
  })
  };

  const formatedDayWeather = (data) => {
    const weatherArr = [];
    let weatherEl = {};
    let date = '';
    data.forEach(weather => {
      if (!date || date === weather.dt_txt.split(" ")[0]) {
          date = weather.dt_txt.split(" ")[0];
          weatherEl.weatherState = weatherEl.weatherState ? [...weatherEl.weatherState, weather.weather[0].description] : [weather.weather[0].description];
          weatherEl.weatherDescription = weatherEl.weatherDescription ? [...weatherEl.weatherDescription, weather.weather[0].main] : [ weather.weather[0].main];
          weatherEl.wind = weatherEl.wind ? [...weatherEl.wind, weather.wind.speed] : [weather.wind.speed];
          weatherEl.temp = weatherEl.temp ? [...weatherEl.temp , +weather.main.temp] : [ +weather.main.temp];
        } else {
          
          weatherEl.time = date.slice(5,11).replace('-', '.')
          weatherEl.weatherState = weatherEl.weatherState[Math.floor(weatherEl.weatherState.length/2)];
          weatherEl.weatherDescription = weatherEl.weatherDescription[Math.floor(weatherEl.weatherDescription.length/2)];
          console.log(weatherEl.weatherDescription, Math.floor(weatherEl.weatherDescription.length/2))
          weatherEl.wind = weatherEl.wind.reduce((sum, el) => sum + el, 0) / weatherEl.wind.length;
          weatherEl.temp = weatherEl.temp.reduce((sum, el) => sum + el, 0) / weatherEl.temp.length;

          weatherArr.push(weatherEl);
          weatherEl = [];
          date = weather.dt_txt.split(" ")[0];
          weatherEl.weatherState = weatherEl.weatherState ? [...weatherEl.weatherState, weather.weather[0].description] : [weather.weather[0].description];
          weatherEl.weatherDescription = weatherEl.weatherDescription ? [...weatherEl.weatherDescription, weather.weather[0].main] : [ weather.weather[0].main];
          weatherEl.wind = weatherEl.wind ? [...weatherEl.wind, weather.wind.speed] : [weather.wind.speed];
          weatherEl.temp = weatherEl.temp ? [...weatherEl.temp , +weather.main.temp] : [ +weather.main.temp];
        }
        console.log(data[data.length-1].dt_txt.split(" ")[0], data[data.length-2].dt_txt.split(" ")[0])
        if(data[data.length-1].dt_txt.split(" ")[0] !== data[data.length-2].dt_txt.split(" ")[0]) {
          weatherEl.time = date.slice(5,11).replace('-', '.')
          weatherEl.weatherState = weatherEl.weatherState[Math.floor(weatherEl.weatherState.length/2)];
          weatherEl.weatherDescription = weatherEl.weatherDescription[Math.floor(weatherEl.weatherDescription.length/2)];
          weatherEl.wind = weatherEl.wind.reduce((sum, el) => sum + el, 0) / weatherEl.wind.length;
          weatherEl.temp = weatherEl.temp.reduce((sum, el) => sum + el, 0) / weatherEl.temp.length;
    
          weatherArr.push();
        }
        weatherData.value = weatherArr;
      })
  };
</script>
<style>
.weather-timeline-wrapper {
  max-width: 1065px;
  overflow-x: auto;
}
.weather-timeline {
  display: flex;
  margin: 0 auto 20px;
  width: fit-content;
}
.weather-timeline-item {
  text-align: center;
  background-color: #f7f5f5;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); 
  padding: 20px 20px 0 20px; 
  margin: 5px; 
}
.weather-chart {
  width: 100%;
}
</style>