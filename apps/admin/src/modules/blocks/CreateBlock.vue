<template>
  <div>
    <PageTitle :title="'Create Block'" />
    <BlockForm :block="{}" @submitted="submit"/>
  </div>
</template>
<script lang="ts">
import {defineComponent} from 'vue'
import {useRouter} from 'vue-router'
import BlockForm from './components/BlockForm.vue'
import PageTitle from '../core/components/semantics/PageTitle.vue'
import {removeUnsavedChanges} from '../drafts/compositions/unsaved-changes'
import blocksService from '../../services/blocks-service';

export default defineComponent({
  name: 'CreateBlock',
  components: {PageTitle, BlockForm},
  setup() {
    const router = useRouter()

    async function submit(data) {
      const {_id} = await blocksService.create(data)
      removeUnsavedChanges('block')
      return router.push({
        name: 'editBlock',
        params: {blockId: _id}
      })
    }

    return {
      submit
    }
  }
})
</script>
