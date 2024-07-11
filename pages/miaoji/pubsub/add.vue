<template>
	<view class="uni-container">
		<uni-forms ref="form" :value="formData" validateTrigger="bind" label-position="top">
			<uni-forms-item name="name" label="模板名称" required>
				<input placeholder="模板名称" @input="binddata('name', $event.detail.value)" class="uni-input-border"
					v-model="formData.name" trim="both" />
			</uni-forms-item>
			<uni-forms-item name="msg_temp_id" label="模板id" required>
				<input placeholder="模板id" @input="binddata('msg_temp_id', $event.detail.value)" class="uni-input-border"
					v-model="formData.msg_temp_id" trim="both" />
			</uni-forms-item>
			<uni-forms-item name="msg_data" label="模板消息(填入JSON序列化后的数据)">
				<textarea placeholder="模板消息" @input="binddata('msg_data', $event.detail.value)"
					class="uni-textarea-border" v-model="formData.msg_data" trim="both" />
			</uni-forms-item>
			<view class="uni-button-group">
				<button type="primary" class="uni-button" style="width: 100px;"
					@click="submit">{{$t('common.button.submit')}}</button>
				<navigator open-type="navigateBack" style="margin-left: 15px;">
					<button class="uni-button" style="width: 100px;">{{$t('common.button.back')}}</button>
				</navigator>
			</view>
		</uni-forms>
	</view>
</template>

<script>
	// import { validator } from '@/js_sdk/validator/uni-id-permissions.js';

	const db = uniCloud.database();
	const dbCmd = db.command;
	const dbCollectionName = 'mj-pubmsg';

	// function getValidator(fields) {
	//   let result = {}
	//   for (let key in validator) {
	//     if (fields.includes(key)) {
	//       result[key] = validator[key]
	//     }
	//   }
	//   return result
	// }

	export default {
		data() {
			let formData = {
				"name": "",
				"msg_temp_id": "",
				"msg_data": "{}"
			}
			return {
				formData,
				formOptions: {},
				// rules: {
				//   ...getValidator(Object.keys(formData))
				// }
			}
		},
		onReady() {
			// this.$refs.form.setRules(this.rules)
		},
		methods: {
			/**
			 * 触发表单提交
			 */
			submit() {
				uni.showLoading({
					mask: true
				})
				this.$refs.form.validate().then((res) => {
					// 解析msg_data
					res.msg_data = JSON.parse(res.msg_data)
					this.submitForm(res)
				}).catch(() => {
					uni.hideLoading()
				})
			},

			submitForm(value) {
				// 使用 clientDB 提交数据
				db.collection(dbCollectionName).add(value).then((res) => {
					uni.showToast({
						title: '新增成功'
					})
					this.getOpenerEventChannel().emit('refreshData')
					setTimeout(() => uni.navigateBack(), 500)
				}).catch((err) => {
					uni.showModal({
						content: err.message || '请求服务失败',
						showCancel: false
					})
				}).finally(() => {
					uni.hideLoading()
				})
			}
		}
	}
</script>

<style>
	::v-deep .uni-forms-item__label {
		width: 90px !important;
	}
</style>