<template>
  <b-form-group :label="$t(field.label) || field.name" >
    <b-input-group :prepend="field.options.prefix" :append="field.options.suffix">
      <b-form-input type="number" v-model="value"></b-form-input>
    </b-input-group>
    <b-form-text v-if="validate && errors">
      <div v-for="(error, i) in errors" :key="i">{{ error }}</div>
    </b-form-text>
  </b-form-group>
</template>
<script>
import base from './base'

export default {
  extends: base,
}
</script>
