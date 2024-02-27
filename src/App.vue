<script setup>
import { ref, onMounted } from "vue";
import {
  curriculums,
  numberMap,
  getCurrentDate,
  obtainTheTotalNumberOfWeeks,
  modifyCourseData,
  getCurrentDateTime,
  firstWeekStartDate,
  lastWeekEndDate,
  curriculumTimes,
  getDatesForWeekNumber,
} from "./utils";
import { showNotify } from "vant";
import "vant/es/notify/style";

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
// 显示日历
const showCalendar = ref(false);
// 展示弹框
const show = ref(false);
// 弹框内容
const dialogInfo = ref({});
// 背景图片
const backImage = ref(null);
// 修改背景图片弹框
const showImg = ref(false);
// 待设置的图片 url
const uploadUrl = ref(null);

// 周的切换
const onConfirm = async ({ selectedOptions }) => {
  result.value = selectedOptions[0]?.value;
  showPicker.value = false;

  emptyArray.value = new Array(8).fill().map(() => new Array(7));
  // 修改课程数据
  modifyCourseData(curriculums, emptyArray, result.value);

  // 展示日期
  const dateArray = getDatesForWeekNumber(result.value);
  weekInfo.value = ["", ...dateArray];
};

// 日历(确认)
const onCalendarConfirm = async (date) => {
  // 检查选择的日期是否在教学周之间
  if (date >= firstWeekStartDate && date <= lastWeekEndDate) {
    showCalendar.value = false;
    const { weekNumber, dateArray } = await getCurrentDate(date);
    // 修改课程数据
    modifyCourseData(curriculums, emptyArray, weekNumber);

    // 展示日期
    weekInfo.value = ["", ...dateArray];

    result.value = weekNumber;
  } else {
    // 如果不在范围内，则弹出警告通知
    showNotify({
      type: "warning",
      message: "选择的日期不在教学周",
      className: ["h-10", "flex", "justify-center", "items-center"],
      color: "#fff",
      background: "rgb(239 68 68)",
    });
  }
};

// 点击课程
const clickCouse = (info) => {
  if (info) {
    show.value = true;
    dialogInfo.value = info;
    dialogInfo.value.time = ["未知"];

    const regex = /教(\d+)/;
    const match = info.classroom.match(regex);

    if (match) {
      const times = match[1] % 2 === 0 ? "double" : "single";
      const index = info.arrayIndex[1];
      dialogInfo.value.time = [
        curriculumTimes[times][index],
        curriculumTimes[times][index + 1],
      ];
    }
  }
};

// 取消 与 确认背景图片
const imgCancel = () => {
  uploadUrl.value = null;
};
const imgConfirm = () => {
  if (uploadUrl.value) {
    localStorage.setItem("bg-image", uploadUrl.value);
    backImage.value = uploadUrl.value;
    uploadUrl.value = null;
  } else {
    localStorage.removeItem("bg-image");
    backImage.value = null;
    uploadUrl.value = null;
  }
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

  // localStorage.setItem(
  //   "bg-image",
  //   "https://dogefs.s3.ladydaily.com/~/source/wallhaven/full/gp/wallhaven-gp17xe.jpg?w=2560&h=1440&fmt=webp"
  // );

  backImage.value = localStorage.getItem("bg-image");
});
</script>

<template>
  <div
    class="home w-full h-full flex flex-col text-xs pt-2.5 relative z-10"
    :class="backImage ? 'bg-no-repeat bg-cover' : 'bg-stone-200'"
  >
    <div class="flex px-2.5">
      <div class="select-col">
        <span> 当前为： </span>
        <span class="text-blue-400 cursor-pointer" @click="showPicker = true">
          第{{ numberMap[result] }}周
        </span>
      </div>

      <div class="flex-1 text-center cursor-pointer" @click="showImg = true">
        课程表
      </div>

      <div class="select-col cursor-pointer" @click="showCalendar = true">
        {{ currentDay }}
      </div>

      <van-popup v-model:show="showPicker" position="bottom">
        <van-picker
          :columns="columns"
          @confirm="onConfirm"
          @cancel="showPicker = false"
        />
      </van-popup>

      <van-calendar v-model:show="showCalendar" @confirm="onCalendarConfirm" />
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
          <table
            class="w-full h-full border-collapse"
            :class="backImage ? '' : 'bg-white'"
          >
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
                    'text-center w-12 h-12 border border-gray-200 cursor-pointer',
                    col?.title ? 'text-white bg-sky-600 hover:bg-blue-400' : '',
                  ]"
                  @click="clickCouse(col)"
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

    <van-dialog
      v-model:show="show"
      :title="dialogInfo.title"
      theme="round-button"
      confirm-button-text="知道了"
    >
      <div class="flex flex-col text-sm p-4 text-gray-400">
        <div class="flex">
          <div class="w-20 text-right">教室：</div>
          <span class="text-black">{{ dialogInfo.classroom }}</span>
        </div>
        <div class="flex">
          <div class="w-20 text-right">课程编号：</div>
          <span class="text-black">{{ dialogInfo.class }}</span>
        </div>
        <div class="flex">
          <div class="w-20 text-right">班级：</div>
          <div class="flex-1 flex flex-col text-black">
            <div
              v-for="classstr in dialogInfo.classString.split(' ')"
              :key="classstr"
            >
              {{ classstr }}
            </div>
          </div>
        </div>
        <div class="flex">
          <div class="w-20 text-right">起止周：</div>
          <span class="text-black">
            {{ dialogInfo.startWeek }} - {{ dialogInfo.endWeek }}
          </span>
        </div>
        <div class="flex">
          <div class="w-20 text-right">上课时间：</div>
          <span class="text-black">{{ dialogInfo.time.join(" ") }}</span>
        </div>
        <div class="flex">
          <div class="w-20 text-right">上课人数：</div>
          <a
            class="text-blue-400"
            target="_blank"
            :href="'/excel/' + dialogInfo.class + '.xls'"
          >
            {{ dialogInfo.peopleNums }}
          </a>
        </div>
        <div class="flex">
          <div class="w-20 text-right">周课时：</div>
          <span class="text-black">{{ dialogInfo.weeklyClassHours }}</span>
        </div>
        <div class="flex">
          <div class="w-20 text-right">学分：</div>
          <span class="text-black">{{ dialogInfo.credit }}</span>
        </div>
        <div class="flex">
          <div class="w-20 text-right">总课时：</div>
          <span class="text-black">{{ dialogInfo.totalClassHours }}</span>
        </div>
      </div>
    </van-dialog>

    <van-dialog
      v-model:show="showImg"
      title="修改背景图片"
      show-cancel-button
      @cancel="imgCancel"
      @confirm="imgConfirm"
      :confirmButtonText="uploadUrl ? '确认' : '重置'"
    >
      <van-cell-group inset>
        <van-field v-model="uploadUrl" label="" placeholder="请输入图片地址" />
      </van-cell-group>

      <img v-if="uploadUrl" :src="uploadUrl" class="w-full p-3" />
    </van-dialog>
  </div>

  <div
    class="absolute w-full h-full inset-0 z-0"
    :class="backImage ? 'bg-no-repeat bg-cover blur-sm backdrop-blur-sm' : ''"
    :style="backImage ? { backgroundImage: `url(${backImage})` } : {}"
  ></div>
</template>

<style scoped>
.home {
  box-sizing: border-box;
}

.right {
  flex: 7 0 0;
}
</style>
