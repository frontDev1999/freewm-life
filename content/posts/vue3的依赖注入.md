+++
date = '2026-07-01T09:00:15+08:00'
draft = false
title = 'Vue3的依赖注入'
+++


### provider 组件

``` javascript
<script setup>
  import {provide, ref} from 'vue'
  
  const location = ref('North Pole')
  
  function updateLocation = (value) => {
    location.value = value
  }
  
  provide('location', {
    location,
    updateLocation
  })
</script>

```

### inject 组件

``` javascript

<script setup>
    import {inject} from 'vue'

    const {location, updateLocation} = inject('location')
</script>

<template>

    <button @click="updateLocation">{{location}}</button>
</template>

```

### 使用 Symbol 作为 注入名

``` javascript

export const myInjectionKey = Symbol()
```