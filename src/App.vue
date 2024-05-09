<template>
  <div class="date-wrap">
    <DatePicker v-model="date"/>
  </div>
</template>

<script setup>
import DatePicker from './components/DatePicker.vue';
import { ref } from 'vue';

const date = ref(new Date());

</script>

<style>
.date-wrap {
  width: 400px;
}
</style>