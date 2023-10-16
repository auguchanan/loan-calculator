<script>
export default {
  props: {
    includeIndex: {
      type: Boolean,
      default: false
    },
    headIndex: {
      type: String,
      default: ''
    },
    head: Array,
    formatHead: {
      type: Function,
      default: value => value
    },
    formatBodyIndex: {
      type: Function,
      default: value => value
    },
    body: {
      type: Array,
      required: true
    },
    formatBody: {
      type: Function,
      default: value => value
    },
    footIndex: {
      type: String,
      default: ''
    },
    foot: Array,
    formatFoot: {
      type: Function,
      default: value => value
    }
  }
}
</script>

<template>
  <table class="table table-striped text-end">
    <thead v-if="head">
      <tr>
        <th v-if="includeIndex" class="col-1" scope="col">{{headIndex}}</th>
        <th v-for="column in head" class="col-2" scope="col">{{formatHead(column)}}</th>
      </tr>
    </thead>
    <tbody :class="{'table-group-divider': head}">
      <tr v-for="(row, index) in body">
        <td v-if="includeIndex">{{formatBodyIndex(index)}}</td>
        <td v-for="column in row">{{formatBody(column)}}</td>
      </tr>
    </tbody>
    <tfoot v-if="foot" class="table-group-divider">
      <tr>
        <th v-if="includeIndex" scope="row">{{footIndex}}</th>
        <td v-for="column in foot">{{formatFoot(column)}}</td>
      </tr>
    </tfoot>
  </table>
</template>
