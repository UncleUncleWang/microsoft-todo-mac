<template lang="pug">
div.header
  div.header__left
    slot(name="left")
  div.header__content
    slot
  div.header__right
    slot(name="right")
</template>

<script>
export default {
  name: 'Header'
}
</script>


<style lang="stylus" scoped>
.header
  display flex
  align-items center
  padding 5px 15px
  // box-shadow 0 0 5px rgba(black, .1)
  user-select none
  height 50px
  box-sizing border-box

.header__left
.header__right
  flex 1
  display flex
  align-items center
  white-space nowrap

.header__left
  justify-content flex-start
  white-space nowrap

.header__right
  justify-content flex-end

.header__content
  flex 2
  flex-grow 10
  font-size 18px
  flex-shrink 0
  text-align center
  color var(--text-main)
  outline none
</style>

