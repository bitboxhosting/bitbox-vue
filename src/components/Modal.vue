<template>
  <Teleport to="#modals">
    <input type="checkbox" :id="props.id" class="modal-toggle" />
    <div class="modal modal-bottom sm:modal-middle">
      <div class="modal-box">
        <slot></slot>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
const props = defineProps(['id'])
</script>
