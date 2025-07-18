<ComposableHeader path="useBreadcrumb/index.ts" title="useBreadcrumb" />

<div class="lead mb-5">

`useBreadcrumb` is a helper utility for the `BBreadcrumb` component. It provides a **globally** changable context so you can modify a breadcrumb. It should be noted that the breadcrumb component will automatically use the global context by default. `useBreadcrumb` is shared globally, one modification to the state will be recognized throughout the app. As noted in the BBreadcrumb documentation, the items prop for the component takes precedence over `useBreadcrumb`

</div>

<UsePluginAlert />

## Demo

<HighlightCard>
  <BBreadcrumb />
  <BFormInput class="my-3" v-model="inputValue" />
  <BButton @click="addItem" class="me-2">Add</BButton>
  <BButton variant="danger" @click="breadcrumb.reset">Clear</BButton>
  <template #html>

```vue
<template>
  <BBreadcrumb />

  <BFormInput v-model="inputValue" />

  <BButton @click="addItem">Add</BButton>
  <BButton variant="danger" @click="breadcrumb.reset">Clear</BButton>
</template>

<script setup lang="ts">
const breadcrumb = useBreadcrumb()

const inputValue = ref('')

const addItem = () => {
  breadcrumb.items?.value.push(inputValue.value)
  inputValue.value = ''
}
</script>
```

  </template>
</HighlightCard>

You can also pass in an id to the component and composable to create a unique breadcrumb trail.

<HighlightCard>
  <BBreadcrumb id="foobar" />
  <BFormInput class="my-3" v-model="foobarInputValue" />
  <BButton @click="foobarAddItem" class="me-2">Add</BButton>
  <BButton variant="danger" @click="foobarBreadcrumb.reset">Clear</BButton>
  <template #html>

```vue
<template>
  <BBreadcrumb id="foobar" />

  <BFormInput v-model="inputValue" />

  <BButton @click="addItem">Add</BButton>
  <BButton variant="danger" @click="breadcrumb.reset">Clear</BButton>
</template>

<script setup lang="ts">
// Input matches the id passed to the component
const breadcrumb = useBreadcrumb('foobar')

const inputValue = ref('')

const addItem = () => {
  breadcrumb.items.value.push(inputValue.value)
  inputValue.value = ''
}
</script>
```

  </template>
</HighlightCard>

<script setup lang="ts">
import {ref} from 'vue'
import HighlightCard from '../../components/HighlightCard.vue'
import UsePluginAlert from '../../components/UsePluginAlert.vue'
import {BBreadcrumb, BButton, BFormInput, BFormGroup, BCard, BCardBody, useBreadcrumb} from 'bootstrap-vue-next'
import ComposableHeader from './ComposableHeader.vue'

const breadcrumb = useBreadcrumb()
const inputValue = ref('')
const addItem = () => {
    breadcrumb.items?.value.push(inputValue.value)
    inputValue.value = ''
}

const foobarBreadcrumb = useBreadcrumb('foobar')
const foobarInputValue = ref('')
const foobarAddItem = () => {
    foobarBreadcrumb.items?.value.push(foobarInputValue.value)
    foobarInputValue.value = ''
}
</script>
