<template>
  <div class="header-version">
    <a-select ref="select" :value="version" style="width: 120px" @focus="focus" @change="handleChange">
      <a-select-option value="24A">24A</a-select-option>
      <a-select-option value="23B">23B</a-select-option>
      <a-select-option value="23A" disabled>23A</a-select-option>
      <a-select-option value="22B">22B</a-select-option>
    </a-select>
  </div>
</template>

<script>
export default {
  name: 'HeaderVersion',
  data() {
    return {
      dataSource: ['选项一', '选项二'],
      searchMode: false,
      version: '24A'
    }
  },
  methods: {
    enterSearchMode() {
      this.searchMode = true
      this.$emit('active', true)
      setTimeout(() => this.$refs.input.focus(), 300)
    },
    leaveSearchMode() {
      this.searchMode = false
      setTimeout(() => this.$emit('active', false), 300)
    }
  }
}
</script>

<style lang="less"></style>
