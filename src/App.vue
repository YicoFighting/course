<script setup>
import { ref, onMounted } from "vue";
import {
  curriculums,
  numberMap,
  getCurrentDate,
  obtainTheTotalNumberOfWeeks,
  modifyCourseData,
  getCurrentDateTime,
} from "./utils";

// 用于展示的七个日期
const weekInfo = ref(new Array(8).fill());
// 计算出来的课程数据
let emptyArray = ref(new Array(8).fill().map(() => new Array(7)));
// 展示周
const weeks = ["", "周一", "周二", "周三", "周四", "周五", "周六", "周日"];

// 当前日期
const currentDay = ref("");
// 周(显示文字)
const result = ref(1);
// 显示选择器
const showPicker = ref(false);
// 选择器数据
const columns = ref([]);

// 周的切换
const onConfirm = ({ selectedOptions }) => {
  result.value = selectedOptions[0]?.value;
  showPicker.value = false;

  emptyArray.value = new Array(8).fill().map(() => new Array(7));
  // 修改课程数据
  modifyCourseData(curriculums, emptyArray, result.value);
};

onMounted(async () => {
  // 当前周数 及 本周日期     new Date("2024-06-15")
  const { weekNumber, dateArray } = await getCurrentDate();

  // 展示日期
  weekInfo.value = ["", ...dateArray];

  // 总周数
  const weeks = obtainTheTotalNumberOfWeeks();

  // 修改课程数据
  modifyCourseData(curriculums, emptyArray, weekNumber);

  for (let i = 1; i <= weeks; i++) {
    columns.value.push({ text: `第${i}周`, value: i });
  }

  result.value = weekNumber;

  setInterval(() => {
    currentDay.value = getCurrentDateTime();
  }, 1000);
});
</script>

<template>
  <div class="home w-full h-full flex flex-col text-xs bg-stone-200 pt-2.5">
    <div class="flex px-2.5">
      <div class="select-col">
        <span> 当前为： </span>
        <span class="text-blue-400" @click="showPicker = true">
          第{{ numberMap[result] }}周
        </span>
      </div>

      <div class="flex-1 text-center">课程表</div>

      <div class="select-col">
        {{ currentDay }}
      </div>

      <van-popup v-model:show="showPicker" position="bottom">
        <van-picker
          :columns="columns"
          @confirm="onConfirm"
          @cancel="showPicker = false"
        />
      </van-popup>
    </div>

    <div class="flex-1 flex flex-col">
      <div class="flex w-full text-center">
        <div class="flex-1" v-for="(week, index) in weeks" :key="week">
          <div class="week">{{ week }}</div>
          <div class="date">{{ weekInfo[index] }}</div>
        </div>
      </div>
      <div class="flex-1 flex">
        <div class="flex-1 flex flex-col">
          <div
            class="flex-1 text-center"
            v-for="num in 8"
            :key="num + ' - section'"
          >
            第{{ numberMap[num] }}节
          </div>
        </div>
        <div class="right">
          <table class="w-full h-full bg-white border-collapse">
            <tr
              class="row"
              v-for="(row, index) in emptyArray"
              :key="index + ' - row'"
            >
              <template v-for="(col, index) in row" :key="index + ' - col'">
                <td
                  v-if="!col?.del"
                  :rowspan="col?.rowspan ?? 1"
                  :class="[
                    'text-center w-12 h-12 border border-gray-200',
                    col?.title ? 'text-white bg-sky-600' : '',
                  ]"
                >
                  <template v-if="col?.title">
                    {{ col?.title }}({{ col?.classroom }})
                  </template>
                </td>
              </template>
            </tr>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.home {
  box-sizing: border-box;
}

.right {
  flex: 7 0 0;
}
</style>
