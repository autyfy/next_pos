<template>
	<Dialog
		v-model="show"
		:options="{ title: __('Enter Insurance Serial Number'), size: 'md' }"
	>
		<template #body-content>
			<div class="flex flex-col gap-4">
				<!-- Item Info -->
				<div v-if="item" class="bg-blue-50 rounded-lg p-3">
					<div class="flex items-center gap-3">
						<div class="w-12 h-12 bg-gray-100 rounded-md flex-shrink-0 flex items-center justify-center overflow-hidden">
							<img
								v-if="item.image"
								:src="item.image"
								:alt="item.item_name"
								loading="lazy"
								class="w-full h-full object-cover"
							/>
							<svg v-else class="h-6 w-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
								<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"/>
							</svg>
						</div>
						<div class="flex-1">
							<h3 class="text-sm font-semibold text-gray-900">{{ item.item_name }}</h3>
							<p class="text-xs text-gray-600">{{ item.item_code }}</p>
							<p v-if="item.custom_item_category" class="text-xs text-blue-600 mt-1">
								{{ item.custom_item_category }}
							</p>
						</div>
					</div>
				</div>

				<!-- Insurance Serial Number Input -->
				<div>
					<label class="block text-sm font-medium text-gray-700 mb-2">
						{{ __('Insurance Serial Number') }}
						<span class="text-red-500 ms-1">*</span>
					</label>
					<div class="relative">
						<input
							ref="serialInput"
							v-model="insuranceSerialNo"
							type="text"
							:placeholder="__('Enter or scan insurance serial number...')"
							:class="[
								'w-full px-3 py-3 ps-10 border rounded-lg text-sm focus:outline-none focus:ring-2',
								hasError
									? 'border-red-500 focus:ring-red-500 focus:border-red-500'
									: 'border-gray-300 focus:ring-blue-500 focus:border-blue-500'
							]"
							@keyup.enter="handleConfirm"
							@input="hasError = false"
						/>
						<svg class="absolute start-3 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"/>
						</svg>
					</div>
					<p v-if="hasError" class="mt-2 text-xs text-red-500">
						{{ __('Insurance Serial Number is required.') }}
					</p>
					<p v-else class="mt-2 text-xs text-gray-500">
						{{ __('This serial number will be recorded with the item in the invoice.') }}
					</p>
				</div>
			</div>
		</template>
		<template #actions>
			<Button
				variant="solid"
				@click="handleConfirm"
			>
				{{ __('Confirm') }}
			</Button>
		</template>
	</Dialog>
</template>

<script setup>
import { Button, Dialog } from "frappe-ui"
import { ref, watch, nextTick } from "vue"

const props = defineProps({
	modelValue: Boolean,
	item: Object,
	quantity: {
		type: Number,
		default: 1,
	},
})

const emit = defineEmits(["update:modelValue", "insurance-serial-entered"])

const show = ref(props.modelValue)
const insuranceSerialNo = ref("")
const serialInput = ref(null)
const hasError = ref(false)

watch(
	() => props.modelValue,
	(val) => {
		show.value = val
		if (val) {
			// Reset and focus input when dialog opens
			insuranceSerialNo.value = ""
			hasError.value = false
			nextTick(() => {
				serialInput.value?.focus()
			})
		}
	},
)

watch(show, (val) => {
	emit("update:modelValue", val)
	if (!val) {
		insuranceSerialNo.value = ""
	}
})

function handleConfirm() {
	if (!insuranceSerialNo.value.trim()) {
		hasError.value = true
		serialInput.value?.focus()
		return
	}
	hasError.value = false
	emit("insurance-serial-entered", {
		insurance_sr_no: insuranceSerialNo.value.trim(),
		quantity: props.quantity,
	})
	show.value = false
}
</script>
