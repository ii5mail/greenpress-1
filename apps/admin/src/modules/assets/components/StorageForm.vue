<template>
	<el-form class="storage-form" @submit.native.prevent="submit">
		<FormInput
				title="Name"
				:model-value="editedStorage.name"
				@input="editedStorage.name = $event"
		/>
		<FormInput
				title="Public URL"
				:model-value="editedStorage.metadata.publicUrl"
				@input="editedStorage.metadata.publicUrl = $event"
		/>
		<FormInput
				title="Base Storage Path"
				:model-value="editedStorage.metadata.basePath"
				@input="editedStorage.metadata.basePath = $event"
		/>
		<p>
			<label>
				{{ $t('Kind') }}:
				<el-select
						:model-value="editedStorage.kind"
						@change="editedStorage.kind = $event"
				>
					<el-option value="s3" label="Amazon S3"/>
					<el-option value="gcs" label="Google Cloud"/>
					<el-option value="ftp" label="FTP"/>
				</el-select>
			</label>
		</p>
		<FormInput
				v-if="editedStorage.kind !== 'ftp'"
				title="Bucket Name"
				:model-value="editedStorage.metadata.bucketName"
				@input="editedStorage.metadata.bucketName = $event"
		/>
		<div class="authentication">
			<h3 @click="showAuth = !showAuth">
				<i :class="showAuth ? 'el-icon-arrow-down' : 'el-icon-arrow-right'"/>
				{{ $t('Authentication Secrets') }}
			</h3>
			<template v-if="showAuth">
				<StorageFtpAuth
						v-if="editedStorage.kind === 'ftp'"
						v-model="editedStorage.authentication"
				/>
				<StorageGcsAuth
						v-else-if="editedStorage.kind === 'gcs'"
						v-model="editedStorage.authentication"
				/>
				<StorageS3Auth
						v-else-if="editedStorage.kind === 's3'"
						v-model="editedStorage.authentication"
				/>
			</template>
		</div>
		<el-button native-type="submit" :loading="submitting">{{ $t('SAVE') }}</el-button>
	</el-form>
</template>
<script>
import StorageFtpAuth from './StorageFtpAuth.vue'
import StorageGcsAuth from './StorageGcsAuth.vue'
import StorageS3Auth from './StorageS3Auth.vue'
import FormInput from '../../core/components/forms/FormInput.vue'
import { useStorageForm } from '../compositions/storages'

export default {
	name: 'StorageForm',
	components: { FormInput, StorageFtpAuth, StorageGcsAuth, StorageS3Auth },
	props: {
		value: Object,
		submitting: Boolean
	},
	setup(props, { emit }) {
		const data = useStorageForm(props)
		return {
			...data,
			submit() {
				return emit('submitted', data.editedStorage)
			}
		}
	}
}
</script>
<style scoped lang="scss">
@import '../../../style/colors';

.authentication {
  border: 1px solid #eee;
  padding: 10px;
  margin: 10px;

  h3 {
	color: $main-color;
	cursor: pointer;
  }
}
</style>
