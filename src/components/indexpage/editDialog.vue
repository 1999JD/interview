<template>
  <q-dialog ref="dialogRef">
    <q-card class="q-dialog-plugin">
      <div class="q-pa-md">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
      </div>
      <!-- buttons example -->
      <q-card-actions align="right">
        <q-btn color="primary" label="更新" @click="onDialogOK(tempData)" />
        <q-btn color="primary" label="取消" @click="onDialogCancel" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>
<script setup lang="ts">
import { ref, defineProps } from 'vue';
import { useDialogPluginComponent } from 'quasar';
import { Axios } from 'axios';

const { dialogRef, onDialogOK, onDialogCancel } = useDialogPluginComponent();

const props = defineProps<{
  name: string;
  age: string;
  id: string;
  api: Axios;
}>();

const tempData = ref({
  name: props.name,
  age: props.age,
  id: props.id,
});
</script>
