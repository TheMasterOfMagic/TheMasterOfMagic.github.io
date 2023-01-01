<style src="@vueform/slider/themes/default.css">

</style>
<style>
ul li {
    font-family: Fira Code;
}
</style>

<template>
    <div>
        当前日期：{{ date.solar }}（{{ date.lunar }}）
        <br />

        <ul>
            <li>
                宜：{{ date.yi }}
            </li>
            <li>
                忌：{{ date.ji }}
            </li>
        </ul>
    </div>
    <br />
    <div>
        当前时间：{{ time.solar }}（{{ time.lunar }}）
        <br />

        <ul>
            <li>
                宜：{{ time.yi }}
            </li>
            <li>
                忌：{{ time.ji }}
            </li>
        </ul>
    </div>
    <br />
    <div>
        <Slider v-model="scoreRange" :min=0 :max=6 />
    </div>
    <br />
    <div>
        <ul>
            <li v-for="item in okYiJiTimes">
                {{ `${item.date} 星期${item.weekday} ${item.time} ${item.score}分` }}
            </li>
        </ul>
    </div>
</template>

<script>
import { defineComponent } from 'vue'
import { Lunar } from 'lunar-javascript';
import Slider from '@vueform/slider'

export default defineComponent({
    components: {
        Slider,
    },
    data() {
        return {
            date: {
                solar: '',
                lunar: '',
                yi: '',
                ji: ''
            },
            time: {
                solar: '',
                lunar: '',
                yi: '',
                ji: ''
            },
            yiJiTimes: [{ date: '', weekday: '', time: '', score: 0 }],
            scoreRange: [0, 6]
        }
    },
    created() {
        setInterval(() => {
            this.updateBasic()
        }, 100)
        this.updateBasic()
    },
    computed: {
        okYiJiTimes() {
            return this.yiJiTimes.filter(item => item.score >= this.scoreRange[0] && item.score <= this.scoreRange[1])
        }
    },
    methods: {
        updateBasic() {
            let now = {
                solar: null,
                lunar: null
            }
            now.solar = new Date()
            now.solar.setSeconds(now.solar.getSeconds())
            now.lunar = Lunar.fromDate(now.solar)

            this.date = {
                solar: now.solar.toLocaleString('zh-CN', {
                    year: 'numeric',
                    month: '2-digit',
                    day: '2-digit'
                }),
                lunar: now.lunar.toString(),
                yi: now.lunar.getDayYi().filterConcernedYiJi().join(' '),
                ji: now.lunar.getDayJi().filterConcernedYiJi().join(' ')
            }
            let midLunarHour = now.lunar.getTimeZhiIndex() * 2
            let startLunarHourStr = `${(midLunarHour + 23) % 24}`.padStart(2, '0')
            let endLunarHourStr = `${(midLunarHour + 1) % 24}`.padStart(2, '0')
            this.time = {
                solar: now.solar.toLocaleString('zh-CN', {
                    hour: '2-digit',
                    minute: '2-digit',
                    second: '2-digit'
                }),
                lunar: `${startLunarHourStr}:00-${endLunarHourStr}:00 ${now.lunar.getTimeInGanZhi()}时`,
                yi: now.lunar.getTimeYi().filterConcernedYiJi().join(' '),
                ji: now.lunar.getTimeJi().filterConcernedYiJi().join(' ')
            }

            if (this.yiJiTimes.length === 1 || now.solar.getMinutes() === 0 && now.solar.getSeconds() === 0) {
                this.updateYiJiTimes()
            }
        },
        updateYiJiTimes() {
            let currentSolar = new Date()
            let currentLunar = Lunar.fromDate(currentSolar)
            let midLunarHour = currentLunar.getTimeZhiIndex() * 2

            currentSolar.setHours(midLunarHour)
            currentSolar.setMinutes(0)
            currentSolar.setSeconds(0)

            this.yiJiTimes.clear()
            for (let i = 0; i < 21 * 12; i++) {
                let currentLunar = Lunar.fromDate(currentSolar)
                let yi = currentLunar.getDayYi().concat(currentLunar.getTimeYi()).filterConcernedYiJi()
                let ji = currentLunar.getDayJi().concat(currentLunar.getTimeJi()).filterConcernedYiJi()
                let score = yi.length - ji.length
                if (score >= this.scoreRange[0] && score <= this.scoreRange[1]) {
                    let date = currentSolar.toLocaleString('zh-CN', {
                        year: 'numeric',
                        month: '2-digit',
                        day: '2-digit'
                    })
                    let weekday = currentSolar.toLocaleString('zh-CN', {
                        weekday: 'narrow'
                    })
                    let currentMidLunarHour = currentLunar.getTimeZhiIndex() * 2
                    let currentStartLunarHourStr = `${(currentMidLunarHour + 23) % 24}`.padStart(2, '0')
                    let currentEndLunarHourStr = `${(currentMidLunarHour + 1) % 24}`.padStart(2, '0')
                    let time = `${currentStartLunarHourStr}:00-${currentEndLunarHourStr}:00 ${currentLunar.getTimeInGanZhi()}时`

                    this.yiJiTimes.push({
                        date: date,
                        weekday: weekday,
                        time: time,
                        score: score
                    })
                }
                currentSolar.setHours(currentSolar.getHours() + 2)
            }
        }
    }
})

let concernedYiJi = ['祭祀', '祈福', '交易']

Array.prototype.filterConcernedYiJi = function () {
    let temp = this.filter((item) => concernedYiJi.includes(item))
    if (temp.length == 0) {
        temp.push('（暂无）')
    }
    return temp
}
Array.prototype.clear = function () {
    this.splice(0, this.length)
}

</script>
  