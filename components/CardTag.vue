<script setup>
import { useStore } from '../store/index.js'

const props = defineProps({
  tag: {
    Type: String
  },
  mode: {
    Type: String
    //card, add, delete
  }
})

const store = useStore()
const router = useRouter()

const addFilter = (tag) => {
  const filter = [...store.filter]
  if (filter.includes(tag)){
    const index = filter.indexOf(tag)
    filter.splice(index)
  } else {
    filter.push(tag)
  }
  router.push({ path: '/search/' + store.searchWord, query: { filter: filter, favorite: store.favorite }})
}

const deleteFilter = (tag) => {
  const filter = [...store.filter]
  if (filter.includes(tag)){
    const index = filter.indexOf(tag)
    filter.splice(index, 1)
  }
  router.push({ path: '/search/' + store.searchWord, query: { filter: filter, favorite: store.favorite }})
}

const cardAction = () => {
  if (props.mode === 'add') {
    addFilter(props.tag)
  } else if (props.mode === 'delete') {
    deleteFilter(props.tag)
  }
}

const inFilter = computed(() => {
  return store.filter.includes(props.tag)
})
</script>

<template>
<div 
  v-if="!(mode === 'add' && inFilter)" 
  v-on:click="cardAction" 
  class="w-fit bg-red-100 rounded-md border border-black py-1 px-2 text-[0.60rem] text-black font-inter"
  v-bind:class="{'cursor-pointer': mode === 'delete' || mode === 'add' }">
  <span v-if="mode === 'delete'" class="text-[0.60rem]">x </span>{{ tag }}
</div>
</template>