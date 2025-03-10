<template>
  <el-form class="post-form" @submit.native.prevent="submit">
    <EditPostHeader :post="post" :submitting="submitting"/>
    <div class="form-content">
      <div class="details" v-if="$route.query.tab === 'details'">
        <el-checkbox :model-value="isPublic" @change="editedPost.isPublic = $event">{{ $t('Public Post') }}
        </el-checkbox>
        <el-checkbox :model-value="isPinned" @change="editedPost.isPinned = $event">{{ $t('Pinned Post') }}
        </el-checkbox>

        <FormInput title="Title" :model-value="title" @input="editedPost.title = $event"/>

        <el-form-item label="Thumbnail">
          <a @click="toggleUpload">Upload</a>
          <AssetUploader v-if="uploadThumbnailOpen" @change="uploadComplete"/>
          <FormInput v-else :model-value="thumbnail" placeholder="https://" @input="thumbnail = $event"/>
          <div v-if="editedPost.thumbnail || post.thumbnail">
            <img v-if="!uploadThumbnailOpen" class="thumbnail-image"
                 :src="editedPost.thumbnail || post.thumbnail">
          </div>
        </el-form-item>

        <PostFormMetadata :category-path="categoryPath"
                          :path="path"
                          :is-new="!post._id"
                          @changed:path="editedPost.path = $event"
                          @changed:category="editedPost.category = $event"/>

        <FormInput title="Tags" v-model="currentTagText" @keydown.enter.prevent="addTag"
                   placeholder="ADD NEW TAG">
          <ul class="tags-list">
            <li v-for="tag in tags" :key="tag" class="tag">
              {{ tag }}
              <i @click="removeTag(tag)" class="el-icon-delete"/>
            </li>
          </ul>
        </FormInput>
      </div>

      <el-form-item v-else-if="$route.query.tab === 'short'" label="Short">
        <gp-editor :model-value="post.short || editedPost.short" @input="editedPost.short = $event"
                   large-editor
                   :config="editorConfig"/>
      </el-form-item>

      <el-form-item v-else-if="$route.query.tab === 'content'" class="form-item-flex">
        <div class="post-contents">
          <PostContentEditor v-for="item in contents"
                             :key="item.index"
                             :value="item.content"
                             :state="item.state"
                             @remove="removeContent(item.index)"
                             @contentChange="setContent(item.index, $event)"
                             @typeChange="setContentsStates(item.index, $event)"/>
          <AddMore @click="addContent"/>
        </div>
      </el-form-item>
    </div>
  </el-form>
</template>
<script lang="ts">
import FormInput from '../../core/components/forms/FormInput.vue'
import {clearNulls} from '../../core/utils/clear-nulls'
import PostContentEditor from './PostContentEditor.vue'
import {computed, onBeforeMount} from 'vue'
import {usePostTags} from '../compositions/post-tags'
import {usePostContents} from '../compositions/post-contents'
import {useNewPost} from '../compositions/posts'
import {useEditedInputs} from '../../core/compositions/edited-inputs'
import {usePostThumbnail} from '../compositions/post-thumbnail'
import {useUnsavedChanges} from '../../drafts/compositions/unsaved-changes'
import {useEditorConfig} from '../compositions/gp-editor'
import AssetUploader from '../../assets/components/AssetUploader.vue'
import PostFormMetadata from './PostFormMetadata.vue';
import EditPostHeader from './EditPostHeader.vue';
import AddMore from '../../core/components/forms/AddMore.vue';

export default {
  components: {AddMore, EditPostHeader, PostFormMetadata, AssetUploader, PostContentEditor, FormInput},
  props: {
    post: Object as () => any,
    submitting: Boolean
  },
  emits: ['submit'],
  setup(props, {emit}) {
    const editedPost: any = useNewPost().post
    const tagsContext = usePostTags(editedPost, props.post)
    const contentsContext = usePostContents(editedPost, props.post)

    useUnsavedChanges('post', props.post._id, computed(() => props.post.title), editedPost)

    onBeforeMount(() => {
      if (!props.post._id) {
        editedPost.isPublic = true
      }
    })

    return {
      ...tagsContext,
      ...contentsContext,
      ...useEditorConfig(),
      ...usePostThumbnail(editedPost, props.post),
      ...useEditedInputs(editedPost, props.post, ['title', 'path']),
      editedPost,
      categoryPath: computed(() => editedPost.category || (props.post.category && props.post.category.path)),
      isPublic: computed(() => {
        const isBool = typeof editedPost.isPublic === 'boolean'
        return isBool ? editedPost.isPublic : props.post.isPublic
      }),
      isPinned: computed(() => {
        const isBool = typeof editedPost.isPinned === 'boolean'
        return isBool ? editedPost.isPinned : props.post.isPinned
      }),
      submit() {
        const submittedPost = clearNulls(editedPost)
        emit('submitted', submittedPost)
      }
    }
  }
}
</script>
<style scoped lang="scss">
@import "../../../style/colors";

.post-form {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 0;
}

.form-content {
  flex: 1;
  overflow: auto;
}

.details {
  padding: 10px;
}

.thumbnail-image {
  max-width: 100px;
}

.post-contents {
  display: flex;
  flex-direction: column;
  align-items: center;

  .content-editor {
    width: 100%;
    margin-bottom: 5px;
  }
}

.tags-list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
}

.tag {
  margin: 5px;
  font-size: 80%;
  padding: 6px;
  border-radius: 3px;
  background-color: #c4ecd0;

  > i {
    cursor: pointer;
  }
}
</style>
