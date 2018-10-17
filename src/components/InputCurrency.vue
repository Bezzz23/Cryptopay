<template>
  <div class="select-field">
    <input
      type="number"
      v-model="price"
      v-on:keyup="$emit('price-change', getInfo)"
      class="select-field__input"
    >
    <div class="select-field__select-wrap">
      <select
        v-model="mutatedCurrency"
        v-on:change="$emit('currency-change', getInfo)"
        class="select-field__select"
      >
        <option
          v-for="option in options"
          v-bind:value="option.value"
          v-bind:key="option.value"
        >
          {{option.text}}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
export default {
  name: 'InputCurrency',
  data: function () {
    return {
      mutatedCurrency: this.currency,
      price: this.text
    }
  },
  watch: {
    text: function (newVal, oldVal) {
      this.price = newVal
    },
    currency: function  (newVal, oldVal) {
      this.mutatedCurrency = newVal
    }
  },
  computed: {
    getInfo: function () {
      return {
        text: this.price,
        currency: this.mutatedCurrency
      }
    }
  },
  props: {
    options: Array,
    text: String,
    currency: String
  }
}
</script>

<style scoped lang="scss">
  .select-field {
    display: flex;
    align-items: center;
    &__input {
      width: 200px;
      height: 40px;
      line-height: 40px;
      background: #DEEEFE;
      padding: 0;
      padding-left: 15px;
      font-size: 16px;
      border: none;
      border-top-left-radius: 4px;
      border-bottom-left-radius: 4px;

      &:focus, &:active {
        outline: none;
      }
    }

    &__select-wrap {
      background-color: #27DC8D;
      background-image: url('../assets/arrow_down.png');
      background-size: 10px;
      background-repeat: no-repeat;
      background-position: 90% 50%;
      height: 40px;
      overflow: hidden;
      width: 70px;
      
      border-top-right-radius: 4px;
      border-bottom-right-radius: 4px;
      select {
        background: transparent;
        border: none;
        font-size: 16px;
        height: 40px;
        cursor: pointer;
        padding-left: 5px; /* If you add too much padding here, the options won't show in IE */
        width: 100px;
        color: white;
        font-weight: 600;

        &:focus, &:active {
          outline: none;
        }
      }
    }
  }
</style>
