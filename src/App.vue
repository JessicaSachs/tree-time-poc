<script setup lang="ts">
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import { computed } from 'vue'
import faker from 'faker'
import _, { omit, uniq } from 'lodash'
import { useVirtualList, useToggle } from '@vueuse/core'

const makeTestDataNode = (data = {}, children = null) => ({
  id: faker.datatype.uuid(),
  data,
  children
})

const arr = Array.from(new Array(20).keys())
const mammalsId = faker.datatype.uuid()
const insectsId = faker.datatype.uuid()
const otherIds = faker.datatype.uuid()
const animalsId = faker.datatype.uuid()

const insects = {
  id: insectsId,
  // parentId: animalsId,
  data: { name: 'Bugs' },
  children: uniq(arr.map(() => makeTestDataNode({ name: faker.animal.insect() })))
}

const other = {
  id: otherIds,
  // parentId: animalsId,
  data: { name: 'Other Animals' },
  children: [
    {
      id: faker.datatype.uuid(),
      // parentId: otherIds,
      data: { name: 'Whales and Dolphins' },
      children: uniq(arr.map(() => makeTestDataNode({ name: faker.animal.cetacean() }))),
    },
    {
      id: faker.datatype.uuid(),
      // parentId: otherIds,
      data: { name: 'Crocodiles and Stuff' },
      children: uniq(arr.map(() => makeTestDataNode({ name: faker.animal.crocodilia() }))),
    }
  ]
}

const mammals = {
  id: mammalsId,
  // parentId: animalsId,
  data: { name: 'Mammals' },
  children: [{
    id: faker.datatype.uuid(),
    // parentId: mammalsId,
    data: { name: 'Bear' },
    children: uniq(arr.map(() => makeTestDataNode({ name: faker.animal.bear() })))
  },
  {
    id: faker.datatype.uuid(),
    // parentId: mammalsId,
    data: { name: 'Cow' },
    children: uniq(arr.map(() => makeTestDataNode({ name: faker.animal.cat() })))
  },
  {
    id: faker.datatype.uuid(),
    // parentId: mammalsId,
    data: { name: 'Dogz' },
    children: uniq(arr.map(() => makeTestDataNode({ name: faker.animal.dog() })))
  },
  {
    id: faker.datatype.uuid(),
    // parentId: mammalsId,
    data: { name: 'Cats' },
    children: uniq(arr.map(() => makeTestDataNode({ name: faker.animal.cat() })))
  }
  ]
}

const animals = {
  // parentId: null,
  id: animalsId,
  data: {},
  children: [mammals, insects, other]
}

const makeNode = (node, depth, parent) => {
  const [closed, toggleOpen] = useToggle(false)
  const hidden = computed(() => {
    node;
    debugger
    return parent?.hidden.value || closed.value
  })

  return {
    ...node,
    depth,
    parent,
    hidden,
    closed,
    toggleOpen
  }
}

const flatTree = (rawNode, acc = [], depth = 0, parent = null) => {
  const node = makeNode(rawNode, depth, parent)
  acc.push(node)

  if (node.children?.length) {
    node.children.forEach((child) => {
      const gonnaBeAThing = flatTree(child, [], depth + 1, node)
      acc = acc.concat(gonnaBeAThing)
    })
  }

  // IDK lgtm, push it
  return acc
}

const allAnimals = flatTree(animals).slice(1)
const { list: flatAnimals, containerProps, wrapperProps } = useVirtualList(allAnimals, { itemHeight: 32 })

</script>

<template>
  <div class="prose mx-24 my-12">
    <h3>Items</h3>
    <div v-bind="containerProps" style="height:300px">
      <div v-bind="wrapperProps">
        <div
          @click="data.toggleOpen"
          :class="{ 'text-indigo-500': data.hidden }"
          :style="{ height: '32px', marginLeft: `${20 * data.depth}px` }"
          v-for="{ data } in flatAnimals"
        >
        {{ data.data.name }}
        </div>
      </div>
    </div>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
