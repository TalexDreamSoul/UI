
<template>
  <div class="line-button-warpper">
    <div class="line-button">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  data: function () {
    return {};
  },
  methods: {}
};
</script>

<style scoped>
/* .line-button-warpper { */
/* margin-bottom: 10px; */
/* } */

.line-button {
  text-align: center;
  background-color: #ebeef3;
  display: block;
  padding: 8px;
  line-height: 1.5;
  transition: all 0.4s;
  cursor: pointer;
  user-select: none;
}

.line-button:hover {
  color: rgb(73, 73, 73);
  background-color: #cfd1d4;
}
</style>
