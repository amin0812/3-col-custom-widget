<template>
  <Vueform :key="updateKey" v-bind="vueform" />

  <div id="widget-content"
    style="max-width: 100%; margin: 0 auto; background-color: #f0f0f0; padding: 10px; border-radius: 5px;">
    <table style="width: 100%; border-collapse: collapse; table-layout: fixed;">
      <tbody>
        <tr>
          <td class="regular-text" style="text-align: left; padding: 10px 40px;">
            <h2 class="h3-purple" style="color:#6D5BA3;">
              {{ formValues.headline }}
            </h2>
            <h4 class="h3-black"
              style="color:#000;text-align: left; ">
              {{ formValues.subheadline }}
            </h4>
          </td>
        </tr>
        <tr>
          <!-- Loop through columns to create table cells -->
          <template v-for="(column, index) in columns" :key="index">
            <td v-if="column && index < formValues.numColumns" :style="columnStyle">
              <!-- Display image if URL is defined and valid -->
              <template v-if="column.image">
                <div style="text-align: center; padding: 10px;">
                  <img :src="column.image" alt="Image" style="max-width: 100%; height: auto; object-fit: cover;" />
                  <p v-if="column.text" class="regular-text"
                    style="margin: 10px auto; text-align: left; word-wrap: break-word; width: 100%;">
                    {{ column.text }}
                  </p>
                </div>
              </template>

              <!-- Custom button HTML structure -->
              <template v-if="column.button">
                <div style="padding: 10px;">
                  <table border="0" cellpadding="0" cellspacing="0" style="background:#ffffff; width: 100%;">
                    <tr>
                      <td alt="left-button" width="28px !important;">
                        <img alt="" height="40"
                          src="https://image.s50.sfmc-content.com/lib/fe3211717564047b7c1272/m/1/2cff3a47-c908-46e5-bf25-fcc44541a4d3.png"
                          style="display: block; height: 40px !important; max-height: 40px !important;" width="28">
                      </td>
                      <td style="max-width:290px; background-color: #007d85;">
                        <a :href="column.buttonLink"
                          style="color:#ffffff;text-decoration:none;font-family: DWS Slab TT, Rockwell, Calibri, Arial, Helvetica, sans-serif;height:40px !important;max-height:40px !important;font-size: 14px;line-height: 45px;mso-line-height-rule: exactly;display: block;background: #007d85;background-color: #007d85;text-align: center;vertical-align: middle;"
                          :title="column.button">{{ column.button }}</a>
                      </td>
                      <td alt="right-button" width="28px !important;">
                        <img alt="" height="40"
                          src="https://image.s50.sfmc-content.com/lib/fe3211717564047b7c1272/m/1/1d91c65d-47fb-4c8a-98e1-612c093749f7.png"
                          style="display: block; height: 40px !important; max-height: 40px !important;" width="28">
                      </td>
                    </tr>
                  </table>
                </div>
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
    headline: {
      type: 'text',
      placeholder: 'Headline'
    },
    subheadline: {
      type: 'text',
      placeholder: 'Subheadline'
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
    textAlign: 'center',
    overflow: 'hidden', // Ensures content stays within the boundaries
    wordWrap: 'break-word', // Ensures long words wrap within the column
    verticalAlign: 'top', // Align content to the top of each column
    height: '300px', // Set a fixed height for all columns (adjust as needed)
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
