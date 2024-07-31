<template>
  <Vueform :key="updateKey" v-bind="vueform" />

  <div id="widget-content" style="width: 800px; max-width: 100%; margin: 0 auto; background-color: #f0f0f0; padding: 10px; border-radius: 5px;">
    <table style="width: 100%; border-collapse: collapse;">
      <tbody>
        <tr>
          <!-- Loop through columns to create table cells -->
          <template v-for="(column, index) in columns" :key="index">
            <td v-if="column && index < formValues.numColumns" :style="columnStyle">
              <!-- Display image if URL is defined and valid -->
              <template v-if="column.image">
                <div style="text-align: center; padding: 10px;">
                  <img :src="column.image" alt="Image" style="max-width: 100%; height: auto; object-fit: cover;" />
                </div>
              </template>
              <p v-if="column.text" style="margin: 10px 0; text-align: center;">{{ column.text }}</p>
              <template v-if="column.button">
                <a :href="column.buttonLink" style="text-decoration: none;">
                  <button style="display: block; width: 100%; padding: 10px; background-color: #007bff; color: #fff; border: none; border-radius: 5px; cursor: pointer;">
                    {{ column.button }}
                  </button>
                </a>
              </template>
            </td>
          </template>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { onMounted, onUpdated, ref, computed, unref } from 'vue';
import { cloneDeep } from 'lodash';
import sdkclass from 'blocksdk';

const sdk = new sdkclass();

const formValues = ref({});

const vueform = ref({
  size: 'md',
  displayErrors: false,
  onChange: (Values) => { formValues.value = Values },
  schema: {
    title: {
      type: 'static',
      content: 'Custom Widget',
      tag: 'h1'
    },
    numColumns: {
      type: 'select',
      items: [
        { label: '1 Column', value: 1 },
        { label: '2 Columns', value: 2 },
        { label: '3 Columns', value: 3 },
      ],
      label: 'Number of Columns',
    },
    columns: {
      type: 'list',
      element: {
        type: 'object',
        schema: {
          image: {
            type: 'text',
            placeholder: 'Image URL',
            columns: {
              container: 12,
              label: 12,
              wrapper: 12,
            }
          },
          text: {
            type: 'text',
            placeholder: 'Text',
            columns: {
              container: 12,
              label: 12,
              wrapper: 12,
            }
          },
          button: {
            type: 'text',
            placeholder: 'Button Name',
            columns: {
              container: 12,
              label: 12,
              wrapper: 12,
            }
          },
          buttonLink: {
            type: 'text',
            placeholder: 'Button Link',
            columns: {
              container: 12,
              label: 12,
              wrapper: 12,
            }
          }
        },
        columns: {
          container: 12,
          label: 12,
          wrapper: 12,
        }
      },
      addText: '+ Add another column',
      initial: 1,
    },
  }
});

const columns = computed(() => {
  return (formValues.value.columns || []).filter(column => column !== undefined);
});

const columnStyle = computed(() => {
  const columnCount = formValues.value.numColumns || 1;
  const totalWidth = 100; // Total width of the container in %

  // Calculate the width for each column
  const columnWidth = totalWidth / columnCount; // Width for each column

  return {
    width: `${columnWidth}%`,
    boxSizing: 'border-box',
    padding: '10px',
    textAlign: 'center'
  };
});

onMounted(() => {
  sdk.getData((data) => {
    formValues.value = data;
    vueform.value.default = data;
    updateKey.value++;
  });
});

onUpdated(() => {
  const content = document.querySelector("#widget-content").innerHTML;
  sdk.setData(cloneDeep(unref(formValues)));
  sdk.setContent(content);
});

const updateKey = ref(0);
</script>
