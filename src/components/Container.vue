<template>
    <div id="app">
        <!-- city -->
        <input type="text" v-model="weather.city" placeholder="city" v-on:keyup.enter="request" :class="{isSunny: isSunny}">
        <div id="demo">
            <!-- WEATHER DATA -->

            <div class="sidebar partition">
                <div class="tooltip">
                <img :src="weather.img" alt="">
<span class="tooltip-text">{{weather.description}}</span>
                </div>
                <h1>{{weather.city}}<span v-if="weather.country != ''">, </span><span v-else><br></span>{{weather.country}}</h1>
                <h2>{{weather.temp}}°C</h2>
                <h3>{{weather.description}}</h3>
                <h3>low {{weather.low}}°C</h3>
                <h3>high {{weather.high}}°C</h3>
            </div>
            
            <div class="right-bar">
                <div class="five-day partition">
                    <div class="day" v-for="day in nextFiveDays" :key="day.index">
                        <p class="white">{{day.date}}</p>
                        <!-- <p>weather: {{day.weather}}</p> -->
                        <div class="tooltip">
                            <!-- hover over me -->
                            <img :src="day.img" :alt="day.description">
                            <span class="tooltip-text">{{day.description}}</span>
                        </div>
                        <p class="blue">{{day.low}}°C</p>
                        <p class="red">{{day.high}}°C</p>
                    </div>
                </div>

                <div class="bottom-bar partition">
                    <div class="humidity">
                        <h3>humidity</h3>
                        <img src="timer.png" alt="">
                        <p>{{weather.humidity}}%</p>
                    </div>
                    <div class="visibility">
                        <h3>precipitation</h3>
                        <img src="drop.png" alt="">
                        <p>{{weather.precip}}mm</p>
                    </div>
                    <div class="wind">
                        <h3>wind</h3>
                        <!-- COMPASS -->
                        <svg width="100" height="100">
                            <circle cx="50" cy="50" r="40" stroke="black" stroke-width="4" fill="none" />
                            <circle cx="50" cy="50" r="6" stroke="black" stroke-width="4" fill="none" />
                            <polygon class="arrow" points="50,50 50,40 45,40 45,25 40,25 50,15 60,25 55,25 55,40 50,40" :style="{transform: 'rotate(' + weather.wind.deg + 'deg)'}" :class="{sunnyCompass: isSunny}" />
                        </svg>

                        <p>{{weather.wind.speed}}mp/h <span v-show="weather.wind.deg">and {{weather.wind.deg}}° </span> 
                            <span>{{getWindDirection()}}</span>
                        </p>
                    </div>
                </div>
            </div>

            


        </div>
    </div>
</template>

<script>

export default {
    components: {
    },
    data(){
        return{
            // default weather object
            weather: {temp: "", city: "Christchurch", country: "", description: "", wind: {
                speed: 0,
                deg: 0
            }},

            nextFiveDays: [],

            // is sunny, by default it is raining
            isSunny: false,
        }
    },
    mounted(){
        this.request()
    },
    methods: {

        whichImage: function(description){
            let img = ""

            description = description.toLowerCase();

            if(description.includes('cloud')) img = "cloud.png"
            if(description.includes('few')) img = "cloudy.png"
            if(description.includes('broken')) img = "cloudy.png"
            if(description.includes('partly')) img = "cloudy.png"
            if(description.includes('rain')) img = "rain.png"
            if(description.includes('heavy') || img.includes('thunder')) img = "heavy-rain.png"
            if(description.includes('sun')) img = "sun.png"
            if(description.includes('wind')) img = "wind.png"
            if(description.includes('snow')) img = "snowflake.png"
            if(img === "") img = "cloudy.png"

            return img
        },

        // requests openweathermap and stores data in weather object
        request: function(){
            var xhttp = new XMLHttpRequest();
            var comp = this;

            xhttp.open("GET",'https://api.weatherbit.io/v2.0/forecast/daily?city=' + comp.weather.city + '&key=e0f56bbe0f43478c8e16856ccae1c0c7',true);
            xhttp.send();
            let json2;
            xhttp.onload=function(){
                json2=JSON.parse(xhttp.responseText);
                comp.weather.low = json2.data[0].app_min_temp
                comp.weather.high = json2.data[0].app_max_temp
                comp.weather.precip = json2.data[0].precip
                comp.nextFiveDays = [];

                for (let index = 1; index < 6; index++) {
                    let dataForThisDay = json2.data[index]

                    let date = dataForThisDay.datetime.slice(8,10);

                    if(date[1] === '1') date += "st"
                    else if(date[1] === '2') date += "nd"
                    else if(date[1] === '3') date += "rd"
                    else date += "th"

                    const description = dataForThisDay.weather.description
                    let img = comp.whichImage(description)

                    let day = {
                        low: dataForThisDay.app_min_temp,
                        high: dataForThisDay.app_max_temp,
                        description: description,
                        img: img,
                        date: date,
                        key: index
                    }

                    comp.nextFiveDays.push(day)
                }
            }

            var xhttp2 = new XMLHttpRequest();

            xhttp2.open("GET",'https://api.openweathermap.org/data/2.5/weather?q=' + comp.weather.city + '&appid=58e70e2fb15041870d65b59cc4f4e856',true);
            xhttp2.send();
            var json;
            xhttp2.onload=function(){
                json=JSON.parse(xhttp2.responseText);
                comp.weather.temp = Math.round(json.main.temp - 273.15)
                comp.weather.city = json.name
                comp.weather.country = json.sys.country
                comp.weather.description = json.weather[0].description
                comp.weather.img = comp.whichImage(json.weather[0].description)
                comp.weather.humidity = json.main.humidity
                comp.weather.visibility = json.visibility + "m"

                if(comp.weather.description.includes("clear"))
                    comp.picked = 'sunny';

                else if(comp.weather.description.includes('rain'))
                    comp.picked = 'rain';
                
                else if(comp.weather.description.includes('snow'))
                    comp.picked = 'snow';
                
                else
                    comp.picked = 'cloudy';
                comp.weather.wind = json.wind

                if(comp.picked != 'sunny'){
                    comp.rainy();
                    comp.$emit('changeToRain', 'rain');
                }
                else{
                    comp.$emit('changeToSunny', 'sun')
                    comp.sunny();
                }
                    
            }

        },

        // returns NESW direction of wind
        getWindDirection: function(){
            var deg = this.weather.wind.deg;

            if(deg > 22 && deg <= 67)
                return "NE";
            else if(deg > 67 && deg <= 112)
                return "E"
            else if(deg > 112 && deg <= 157)
                return "SE"
            else if(deg > 157 && deg <= 202)
                return "S"
            else if(deg > 202 && deg <= 247)
                return "SW"
            else if(deg > 247 && deg <= 292)
                return "W"
            else if(deg > 292 && deg <= 338)
                return "NW"
            else
                return "N";
        },

        rainy: function(){
            this.isSunny = false;
        },

        sunny: function(){
            this.isSunny = true;

            document.getElementById("sunRaysAni2").beginElement();

            document.getElementById("rays").style.opacity = "1";
        }
    }
}
</script>

<style scoped>

#app{
    margin-top: 20px;
    color: snow;
}

.white{
    color: snow;
}

p{
    color: rgba(200,200,200,1);
}

/* INPUT */
input{
    background: transparent;
    color: snow;
    border-bottom: 2px solid snow;
    border-left: 0px;
    border-top: 0px;
    border-right: 0px;
    font-size: 2em;
    transition: border-color 0.2s;
    outline: none;
}

input:focus{
    border-color: #EBBAB9;
    border-left: 0px;
    border-top: 0px;
}

/* COMPASS */
#demo svg{
    /* margin-top: 40px; */
    /* margin-bottom: 40px; */
    /* transform: scale(1.8); */
    fill: #11998e;
    fill: snow;
}

#demo{
    display: flex;
    border: 1px solid rgba(255,200,180,0.5);
    /* margin-left: 10%; */
    margin-top: 38px;
    flex-direction: column;
    /* margin-right: 10%; */
}

@media screen and (min-width: 780px) {
    #demo{
        margin-left: 10%;
        margin-right: 10%;
        flex-direction: row;
    }
}

.arrow{
    transform-origin: center;
    transform:rotate(0deg);
}

.five-day{
    display: flex;
    justify-content: space-evenly;
}

.bottom-bar{
    display: flex;
    justify-content: space-evenly;
    align-items: center;
    border-top: 1px solid rgba(255,200,180,0.5);
}

.partition{
    padding: 24px;
}

.tooltip {
    position: relative;
    display: inline-block;
}

.tooltip img{
    width: 64px;
}

.tooltip-text{
    visibility: hidden;
    width: 120px;
    background-color: rgba(50,50,100,0.9);
    color: #fff;
    text-align: center;
    padding: 5px 0;
    border-radius: 6px;
    
    position: absolute;
    z-index: 1;
    bottom: -48px;
    left: -20px;
}
.sidebar{
    border-right: 1px solid rgba(255,200,180,0.5);
}
.sidebar .tooltip-text{
    left: 5px;
}

.tooltip:hover .tooltip-text {
  visibility: visible;
}

.tooltip-text:hover{
    visibility: hidden;
}

.right-bar{
    width: 100%;
}

.sidebar img{
    width: 128px;
}

.bottom-bar img{
    width: 100px;
    height: 100px;
}

.blue{
    /* color: rgb(100,150,255); */
    color: snow;
    background-color: rgba(0,0,255,0.1);
    padding: 4px;
    border-radius: 4px;
}
.red{
    /* color: rgb(255,100,100); */
    color: white;
    background-color: rgba(255,0,0,0.1);
    padding: 4px;
    border-radius: 4px;
}
</style>