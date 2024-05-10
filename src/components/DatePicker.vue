<template>
  <div class="datepicker v-theme--light">
    <v-text-field
      label="Picker in menu"
      :modelValue="props.modelValue"
      readonly
      @focus="datePickerVisible = true"
    ></v-text-field>
    <div
      v-if="datePickerVisible"
      class="datepicker-modal"
    >
      <div class="datepicker-header">
        <span>
          {{ formattedValue }}
        </span>
        <button @click="favoriteDateVisible = true">
          <svg width="48" height="44" viewBox="0 0 48 44" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M38 38H10V16H38M32 2V6H16V2H12V6H10C7.8 6 6 7.8 6 10V38C6 40.22 7.8 42 10 42H38C40.22 42 42 40.22 42 38V10C42 7.8 40.22 6 38 6H36V2M21.76 24H14.54L20.38 28.22L18.16 35.12L24 30.86L29.84 35.12L27.6 28.24L33.44 24H26.24L24 17.12L21.76 24Z" fill="white"/>
          </svg>
        </button>
        <div v-if="favoriteDateVisible" class="favorite-dates">
          <button
            class="favorite-date"
            v-for="(date, index) in favoriteDates"
            :key="index"
            @click="favoriteDateClick(date.shift)"
          >
            {{ date.text }}
          </button>
        </div>
      </div>

      <div class="datepicker-controls">
        <v-btn variant="elevated" flat icon="$prev" @click="prevMonth"></v-btn>
        <v-btn variant="elevated" flat @click="datePickerComponentActive = 'month'">{{ activeMonthName }}</v-btn>
        <v-btn variant="elevated" flat @click="datePickerComponentActive = 'year'">{{ year }}</v-btn>
        <v-btn variant="elevated" flat icon="$next" @click="nextMonth"></v-btn>
      </div>

      <div class="v-picker__body">
        <v-locale-provider locale="ua">
          <v-date-picker-months
            v-show="datePickerComponentActive === 'month'"
            color="primary"
            :modelValue="month"
            @update:modelValue="updateMonth"
          >
          </v-date-picker-months>
        </v-locale-provider>

        <v-date-picker-years
          color="primary"
          v-show="datePickerComponentActive === 'year'"
          :modelValue="year"
          @update:modelValue="updateYear"
        >
        </v-date-picker-years>

        <div v-show="datePickerComponentActive === 'calendar'">
          <div class="v-date-picker-month">
            <div class="v-date-picker-month__days">
              <div
                class="v-date-picker-month__day v-date-picker-month__weekday"
                v-for="(day, index) in weekdays"
                :key="index"
              >{{ day }}</div>
            </div>
          </div>
          <v-date-picker-month
            :modelValue="[value]"
            @update:modelValue="value = $event[0]"
            :hideWeekdays="true"
            :month="month"
            :year="year"
            color="primary"
          ></v-date-picker-month>
        </div>
      </div>

      <button class="datepicker-footer" @click="updateValue()">
        <svg width="48" height="48" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
          <g opacity="0.6">
          <path d="M38 38H10V16H38M38 6H36V2H32V6H16V2H12V6H10C7.78 6 6 7.8 6 10V38C6 39.0609 6.42143 40.0783 7.17157 40.8284C7.92172 41.5786 8.93913 42 10 42H38C39.0609 42 40.0783 41.5786 40.8284 40.8284C41.5786 40.0783 42 39.0609 42 38V10C42 8.93913 41.5786 7.92172 40.8284 7.17157C40.0783 6.42143 39.0609 6 38 6ZM33.06 22.12L30.94 20L21.18 29.76L16.94 25.52L14.82 27.64L21.18 34L33.06 22.12Z" fill="black"/>
          </g>
        </svg>
        ОБРАТИ
      </button>
    </div>
    <div class="datepicker-overlay" @click="datePickerVisible = false" v-if="datePickerVisible"></div>
  </div>
</template>

<script setup>
  import { ref, defineProps, computed } from 'vue';

  const weekdays = ['ПН', 'ВТ', 'СР', 'ЧТ', 'ПТ', 'СБ', 'НД'];
  const formatter = new Intl.DateTimeFormat('uk-UA', { month: 'long', locale: 'uk-UA', day: 'numeric', year: 'numeric' });
  const favoriteDates = [
    {
      text: 'Позавчора',
      shift: -2,
    },
    {
      text: 'Вчора',
      shift: -1,
    },
    {
      text: 'Сьогодні',
      shift: 0,
    },
    {
      text: 'Завтра',
      shift: 1,
    },
    {
      text: 'Післязавтра',
      shift: 2,
    },
  ];
  const props = defineProps({
    modelValue: {
      type: Date,
      default: new Date(),
    },
  });

  const emit = defineEmits(['update:modelValue']);

  const value = ref(props.modelValue);
  const month = ref(value.value.getMonth());
  const year = ref(value.value.getFullYear());
  const datePickerVisible = ref(false);
  const datePickerComponentActive = ref('calendar');
  const favoriteDateVisible = ref(false);

  const formattedValue = computed(() => 
    `${formatter.format(value.value).slice(0, -3)}, ${new Intl.DateTimeFormat('uk-UA', {weekday: 'short'}).format(value.value).toUpperCase()}`
  );

  const activeMonthName = computed(() => 
    new Intl.DateTimeFormat('uk-UA', { month: 'long' }).format(new Date().setMonth(month.value))
  );

  function updateValue() {
    emit('update:modelValue', value.value);
    datePickerVisible.value = false;
  }

  function updateMonth(value) {
    month.value = value;
    datePickerComponentActive.value = 'calendar';
  }

  function updateYear(value) {
    year.value = value;
    datePickerComponentActive.value = 'calendar';
  }

  function prevMonth() {
    if (month.value === 0) {
      month.value = 11;
      year.value--;
    } else {
      month.value--;
    }
  }

  function nextMonth() {
    if (month.value === 11) {
      month.value = 0;
      year.value++;
    } else {
      month.value++;
    }
  }

  function favoriteDateClick(shift) {
    const date = new Date();
    date.setDate(date.getDate() + shift);
    year.value = date.getFullYear();
    month.value = date.getMonth();
    value.value = date;
    favoriteDateVisible.value = false;
    datePickerComponentActive.value = 'calendar';
  }
</script>

<style lang="scss">
  .datepicker.v-theme--light *{
    --v-theme-primary: 101 156 192;
  }

  .datepicker {
    position: relative;
    width: 100%;

    &-modal {
      position: absolute;
      width: 100%;
      z-index: 1;
      background: #fff;
    }

    &-overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100vh;
      background: rgba(0, 0, 0, 0.2);
      z-index: 0;
    }

    .v-picker-title {
      display: none;
    }

    &-controls {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    &-header {
      position: relative;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 12px;
      font-size: 25px;
      color: #fff;
      background: rgb(var(--v-theme-primary));
    }

    &-footer {
      height: 65px;
      display: flex;
      align-items: center;
      justify-content: center;
      width: 100%;
      background: #e5e5e5;
      font-size: 24px;

      svg {
        margin-right: 7px;
      }
    }

    .v-btn--icon {
      border-radius: unset;
    }

    .v-date-picker-month__weekday {
      color: rgba(0, 0, 0, 0.7);
    }

    .favorite-dates {
      position: absolute;
      left: 100%;
      top: 0;
      background: #fff;
      color: rgba(0, 0, 0, 0.7);
      font-size: 16px;
      min-width: 200px;
    
      .favorite-date {
        text-align: left;
        width: 100%;
        padding: 18px 20px;
        border-bottom: 1px solid #C9C9C9;
        
        &:last-child {
          border: unset;
        }
      }
    }
  }
</style>
