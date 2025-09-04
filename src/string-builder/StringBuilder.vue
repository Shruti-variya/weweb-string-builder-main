<script setup>
import { ref, computed } from 'vue'
import { Copy, Check } from 'lucide-vue-next'

const props = defineProps({
  data: [Object, Array]
})

// Sample input - can be an object or an array of objects
const attributes = ref(props.data)

// Function to get available items based on the first item in attributes
const getAvailableItems = computed(() => {
  const firstItem = Array.isArray(attributes.value) ? attributes.value[0] : attributes.value
  return Object.entries(firstItem).map(([key, value]) => ({
    id: key,
    label: key.replace(/_/g, ' ').replace(/\b\w/g, (l) => l.toUpperCase()),
    value: value
  }))
})

// Builder content
const builderContent = ref('')

// Function to insert field at cursor position
const insertField = (field) => {
  const textarea = document.getElementById('builder-textarea')
  const cursorPos = textarea.selectionStart
  const textBefore = builderContent.value.substring(0, cursorPos)
  const textAfter = builderContent.value.substring(cursorPos)
  builderContent.value = `${textBefore}{${field.id}}${textAfter}`

  // Set cursor position after inserted field
  setTimeout(() => {
    textarea.selectionStart = textarea.selectionEnd = cursorPos + field.id.length + 2
    textarea.focus()
  }, 0)
}

// Computed property for the final string(s)
const finalStrings = computed(() => {
  const replaceFields = (str, item) => {
    return str.replace(/\{(\w+)\}/g, (match, key) => {
      return item[key] !== undefined ? item[key] : match
    })
  }

  if (Array.isArray(attributes.value)) {
    return attributes.value.map((item) => replaceFields(builderContent.value, item))
  } else {
    return [replaceFields(builderContent.value, attributes.value)]
  }
})

// Copy functionality
const copyStatus = ref(finalStrings.value.map(() => false))

const copyToClipboard = async (text, index) => {
  try {
    await navigator.clipboard.writeText(text)
    copyStatus.value[index] = true
    setTimeout(() => {
      copyStatus.value[index] = false
    }, 2000)
  } catch (err) {
    console.error('Failed to copy: ', err)
  }
}
</script>

<template>
  <div class="string-builder">
    <div class="attributes-wrapper">
      <div
        v-for="item in getAvailableItems"
        :key="item.id"
        @click="insertField(item)"
        class="attributes"
      >
        {{ item.id }}
      </div>
    </div>
    <textarea
      id="builder-textarea"
      v-model="builderContent"
      placeholder="Click on fields to insert, or type freely here..."
    ></textarea>

    <!-- Preview -->
    <div v-for="(finalString, index) in finalStrings" :key="index" class="final-strings-wrapper">
      <div v-if="finalString" class="final-strings-inner">
        <pre class="final-strings">{{ finalString }}</pre>
        <button
          v-if="finalString.length"
          @click="copyToClipboard(finalString, index)"
          class="button-action"
          :title="copyStatus[index] ? 'Copied!' : 'Copy to clipboard'"
        >
          <Copy v-if="!copyStatus[index]" class="icon-copy" />
          <Check v-else class="icon-check" />
        </button>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.string-builder {
  display: flex;
  padding: 1rem;
  margin: 1rem;
  flex-direction: column;
  gap: 1rem;
  border-radius: 0.5rem;
  border-width: 1px;
  background-color: #f9fafb;
  .attributes-wrapper {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    .attributes {
      padding: 0.5rem;
      border-radius: 0.25rem;
      color: #ffffff;
      background-color: #3b82f6;
      transition-property:
        color, background-color, border-color, text-decoration-color, fill, stroke;
      transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
      transition-duration: 300ms;
      cursor: pointer;

      &:hover {
        background-color: #2563eb;
      }
    }
  }
  textarea {
    padding: 0.5rem;
    border-radius: 0.5rem;
    border-width: 1px;
    width: 100%;
    height: 10rem;
    resize: none;
  }

  .final-strings-wrapper {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    .final-strings-inner {
      display: relative;
    }
    .final-strings {
      padding: 0.75rem;
      padding-right: 3rem;
      border-radius: 0.25rem;
      white-space: pre-wrap;
      word-break: break-all;
      background-color: #f3f4f6;
    }
    .button-action {
      position: absolute;
      top: 0.5rem;
      right: 0.5rem;
      padding: 0.5rem;
      border-radius: 9999px;
      transition-property:
        color, background-color, border-color, text-decoration-color, fill, stroke;
      transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
      transition-duration: 300ms;

      &:hover {
        background-color: #e5e7eb;
      }

      .icon-copy {
        width: 1.25rem;
        height: 1.25rem;
        color: #4b5563;
      }
      .icon-check {
        width: 1.25rem;
        height: 1.25rem;
        color: #059669;
      }
    }
  }
}
</style>
