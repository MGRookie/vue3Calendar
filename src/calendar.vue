<!--
 * @Description: 
 * @Version: 1.0
 * @Autor: gaoluo
 * @Date: 2022-04-29 16:43:59
 * @LastEditors: gaoluo
 * @LastEditTime: 2022-04-29 16:44:00
 * @FilePath: /CalendarForVue/src/calendar.vue
-->
<template>
  <div class="calendar-container">
    <div class="calendar-control-wrap">
      <span class="calendar-pre-btn" @click="onToPrev"></span>
      <p class="calendar-title">{{ dateTitle }}</p>
      <span class="calendar-next-btn" @click="onToNext"></span>
    </div>
    <div class="calendar-week-wrap">
      <ul class="calendar-week">
        <li class="calendar-text" v-for="text of topText" :key="text">
          {{ text }}
        </li>
      </ul>
    </div>
    <div class="calendar-content-wrap">
      <ul class="calendar-week">
        <li
          class="calendar-text"
          :class="select(item)"
          v-for="item of dateList"
          :key="'' + item.month + item.day"
          @click.stop="chooseDay(item)"
        >
          <slot :default="item">
            <span
              class="calendar-day"
              :class="[
                {
                  'calendar-today': item.isToday,
                  'calendar-curmonth': item.curMonth,
                  'calendat-select': item.select,
                },
              ]"
              >{{ item.day }}</span
            >
          </slot>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from "vue";

const props = defineProps({
  topText: {
    type: Array,
    default: () => ["日", "一", "二", "三", "四", "五", "六"],
  },
  mark: {
    type: Array,
    default: () => [],
  },
});

const emits = defineEmits(["onHandlePrev", "onHandleNext"]);
const dateList = ref([]);

const select = (date) => {
  const markObj = {};
  props.mark.forEach((item) => {
    markObj[item.mark] = item.dates.includes(
      date.year + "-" + date.month + "-" + date.day
    );
  });
  return markObj;
};

const curDate = ref(new Date());
const dateTitle = computed(() => {
  return `${curDate.value.getFullYear()}年${curDate.value.getMonth() + 1}月`;
});
dateList.value = getMonthList(curDate.value);

/**
 * 获取渲染列表数据
 * @param {*} date
 */
function getMonthList(date) {
  const dateList = [];
  const month = date.getMonth() + 1;
  const year = date.getFullYear();
  const today = new Date().toLocaleDateString().replaceAll("/", "-");
  const len = getAllDayInMonth(date);
  for (let i = 1; i <= len; i++) {
    dateList.push({
      day: ("" + i).padStart(2, "0"),
      month: ("" + month).padStart(2, "0"),
      curMonth: "cur",
      year,
      isToday: year + "-" + month + "-" + ("" + i).padStart(2, "0") === today,
    });
  }
  const preList = getPreMonth(date);
  const nextList = getNextMonth(date);
  return [...preList, ...dateList, ...nextList];
}

/**
 * 获取上一个月尾
 * @param {*} date
 */
function getPreMonth(date) {
  const preList = [];
  let weekOf1 = getOneWeekdayInMonth(date);
  let preMonth = getAllDayInMonth(
    new Date(date.getFullYear(), date.getMonth() - 1)
  );
  const month = date.getMonth() || 12;
  const year = month === 12 ? date.getFullYear() - 1 : date.getFullYear();
  while (weekOf1 > 0) {
    preList.unshift({
      day: ("" + preMonth).padStart(2, "0"),
      month: ("" + month).padStart(2, "0"),
      year,
      isToday: false,
      curMonth: "prev",
    });
    weekOf1--;
    preMonth--;
  }
  return preList;
}
/**
 * 获取下一个月初
 * @param {} date
 */
function getNextMonth(date) {
  const nextList = [];
  const nextWeek = getOneWeekdayInMonth(
    new Date(date.getFullYear(), date.getMonth() + 1)
  );
  const month = (date.getMonth() + 2) % 12 || 12;
  const year = month === 1 ? date.getFullYear() + 1 : date.getFullYear();
  for (var i = 1; i <= 7 - nextWeek; i++) {
    nextList.push({
      day: ("" + i).padStart(2, "0"),
      month: ("" + month).padStart(2, "0"),
      year,
      isToday: false,
      curMonth: "next",
    });
  }
  return nextList;
}

/**
 * 根据一个时间对象获取某个月的总天数
 * @param {*} date Date对象
 * @return days 某个月的总天数
 */
function getAllDayInMonth(date) {
  return new Date(date.getFullYear(), date.getMonth() + 1, 0).getDate();
}

/**
 * @param {*} date Date对象
 * @return week 获取某月1日是星期几
 */
function getOneWeekdayInMonth(date) {
  return new Date(date.getFullYear(), date.getMonth(), 1).getDay();
}

/**
 * 获取上一个月或下一个月date
 * @param {*} date
 * @param {*} tag
 */
function getOtherMonth(date, tag = "next") {
  if (tag === "next") {
    return new Date(date.getFullYear(), date.getMonth() + 1);
  } else {
    return new Date(date.getFullYear(), date.getMonth() - 1);
  }
}

const onToPrev = () => {
  curDate.value = getOtherMonth(curDate.value, "prev");
  dateList.value = getMonthList(curDate.value);
  emits("onHandlePrev", curDate.value);
};

const onToNext = () => {
  curDate.value = getOtherMonth(curDate.value, "next");
  dateList.value = getMonthList(curDate.value);
  emits("onHandleNext", curDate.value);
};

const chooseDay = (date) => {
  if (date.curMonth !== "cur") {
    date.curMonth === "next" ? onToNext() : onToPrev();
  }

  dateList.value.forEach((item) => {
    item.select = false;
    if (date.month + "-" + date.day === item.month + "-" + item.day) {
      item.select = !item.select;
    }
  });
};
</script>

<style lang="less">
.calendar-container {
  margin: 10px auto;
  max-width: 470px;
  border: 1px solid;
  -webkit-tap-highlight-color: transparent;
}

.calendar-control-wrap {
  width: 100%;
  display: flex;
  border-bottom: 1px solid #eee;
  align-items: center;
}

.calendar-title {
  flex: 1;
  text-align: center;
  font-size: 20px;
  margin: 0;
  padding: 10px 0;
}

.calendar-pre-btn,
.calendar-next-btn {
  margin: 0 10px;
  padding: 10px;
  cursor: pointer;
}

.calendar-pre-btn::after,
.calendar-next-btn::after {
  content: "";
  display: block;
  width: 10px;
  height: 10px;
  border: 2px solid;
  border-bottom: none;
  border-right: none;
}

.calendar-pre-btn {
  transform: rotateZ(-45deg);
}

.calendar-next-btn {
  transform: rotateZ(135deg);
}

.calendar-week-wrap {
  width: 100%;
}

.calendar-week {
  width: 100%;
  height: 40px;
  line-height: 40px;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
}

.calendar-text {
  width: 14.285%;
  text-align: center;
}

.calendar-content-wrap {
  .calendar-week {
    height: auto;
  }
}

.calendar-day {
  display: block;
  width: 40px;
  height: 40px;
  line-height: 40px;
  margin: 0 auto;
  border-radius: 50%;
  // background-color: rgb(62, 183, 249);
  color: #aaa;
  cursor: pointer;

  &.calendar-curmonth {
    color: #424242;
  }

  &.calendar-today {
    color: aqua;
    background-color: antiquewhite;
    border-radius: 50%;
  }

  &.calendat-select {
    background-color: rgb(49, 92, 232);
  }
}
</style>
