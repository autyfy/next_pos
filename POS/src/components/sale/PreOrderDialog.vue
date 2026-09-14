<template>
	<Dialog v-model="show" :options="{ title: __('Enter PreOrder ID'), size: 'md' }">
		<template #body-content>
			<div class="flex flex-col gap-4">
				<div v-if="item" class="bg-blue-50 rounded-lg p-3">
					<h3 class="text-sm font-semibold text-gray-900">{{ item.item_name }}</h3>
					<p class="text-xs text-gray-600">{{ item.item_code }}</p>
				</div>
				<div>
					<label class="block text-sm font-medium text-gray-700 mb-2">
						{{ __('PreOrder ID') }}<span class="text-red-500 ms-1">*</span>
					</label>
					<input
						ref="preOrderInput"
						v-model="preOrderId"
						type="text"
						inputmode="numeric"
						pattern="[0-9]*"
						autocomplete="off"
						:placeholder="__('Enter PreOrder ID...')"
						:class="[
							'w-full px-3 py-3 border rounded-lg text-sm focus:outline-none focus:ring-2',
							hasError
								? 'border-red-500 focus:ring-red-500 focus:border-red-500'
								: 'border-gray-300 focus:ring-blue-500 focus:border-blue-500',
						]"
						@input="handleInput"
						@keyup.enter="handleConfirm"
					/>
					<p v-if="hasError" class="mt-2 text-xs text-red-500">
						{{ __('PreOrder ID is required and must contain numbers only.') }}
					</p>
					<p v-else class="mt-2 text-xs text-gray-500">
						{{ __('This PreOrder ID will be recorded with the item in the invoice.') }}
					</p>
				</div>
			</div>
		</template>
		<template #actions>
			<Button variant="solid" @click="handleConfirm">{{ __('Confirm') }}</Button>
		</template>
	</Dialog>
</template>

<script setup>
import { Button, Dialog } from "frappe-ui"
import { nextTick, ref, watch } from "vue"

const props = defineProps({
	modelValue: Boolean,
	item: Object,
	quantity: { type: Number, default: 1 },
})
const emit = defineEmits(["update:modelValue", "preorder-id-entered"])
const show = ref(props.modelValue)
const preOrderId = ref("")
const preOrderInput = ref(null)
const hasError = ref(false)

watch(() => props.modelValue, (value) => {
	show.value = value
	if (value) {
		preOrderId.value = ""
		hasError.value = false
		nextTick(() => preOrderInput.value?.focus())
	}
})
watch(show, (value) => {
	emit("update:modelValue", value)
	if (!value) preOrderId.value = ""
})
function handleInput(event) {
	preOrderId.value = event.target.value.replace(/\D/g, "")
	hasError.value = false
}
function handleConfirm() {
	if (!/^\d+$/.test(preOrderId.value)) {
		hasError.value = true
		preOrderInput.value?.focus()
		return
	}
	emit("preorder-id-entered", { pre_order_id: preOrderId.value, quantity: props.quantity })
	show.value = false
}
</script>
