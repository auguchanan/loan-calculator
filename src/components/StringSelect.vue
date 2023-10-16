<script>
import {v4} from 'uuid'

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
    options: {
      type: Array,
      required: true
    },
    value: {
      type: String,
      required: true
    }
  }
}
</script>

<template>
  <div class="mb-3">
    <label v-if="description" class="form-label" :for="id">{{description}}</label>
    <select class="form-control" :id="id" :value="value" @input="event => $emit('update', event.target.value)">
      <option v-for="option in options" :value="option.value">{{option.description}}</option>
    </select>
  </div>
</template>
