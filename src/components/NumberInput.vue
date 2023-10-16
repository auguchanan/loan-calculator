<script>
import {v4} from 'uuid';

export default {
  data() {
    return {
      id: v4()
    }
  },
  emits: [
    'update'
  ],
  props: {
    description: String,
    min: Number,
    max: Number,
    step: Number,
    value: {
      type: Number,
      required: true
    },
    unit: String
  }
}
</script>

<template>
  <div class="mb-3">
    <label v-if="description" class="form-label" :for="id">{{description}}</label>
    <div class="input-group">
      <input class="form-control" :id="id" type="number" :min="min" :max="max" :step="step" :value="value" @input="event => {const number = parseFloat(event.target.value); $emit('update', isNaN(number) ? 0 : number) }"/>
      <span v-if="unit" class="input-group-text">{{ unit }}</span>
    </div>
  </div>
</template>
