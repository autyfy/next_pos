<template>
	<Dialog v-model="show" :options="{ title: __('Select Insurance Supplier'), size: 'md' }">
		<template #body-content>
			<div class="flex flex-col gap-4">
				<div v-if="item" class="rounded-lg bg-blue-50 p-3">
					<p class="text-sm font-semibold text-gray-900">{{ item.item_name }}</p>
					<p class="text-xs text-gray-600">{{ item.item_code }}</p>
				</div>
				<div v-if="loading" class="py-6 text-center text-sm text-gray-500">{{ __('Loading suppliers...') }}</div>
				<div v-else-if="errorMessage" class="rounded-lg bg-red-50 p-3 text-sm text-red-700">{{ errorMessage }}</div>
				<div v-else class="flex flex-col gap-2">
					<button
						v-for="row in suppliers"
						:key="`${row.supplier}-${row.supplier_address || ''}`"
						type="button"
						:class="['w-full rounded-lg border p-4 text-start transition-colors', selectedSupplier === row.supplier ? 'border-blue-600 bg-blue-50 ring-1 ring-blue-600' : 'border-gray-200 bg-white hover:border-blue-400 hover:bg-blue-50']"
						@click="selectedSupplier = row.supplier"
					>
						<p class="text-sm font-semibold text-gray-900">{{ row.supplier_name || row.supplier }}</p>
						<p v-if="row.supplier_address" class="mt-1 text-xs text-gray-500">{{ row.supplier_address }}</p>
					</button>
				</div>
			</div>
		</template>
		<template #actions>
			<Button variant="solid" :disabled="!selectedRow" @click="handleConfirm">{{ __('Confirm') }}</Button>
		</template>
	</Dialog>
</template>

<script setup>
import { Button, Dialog, call } from "frappe-ui"
import { computed, ref, watch } from "vue"

const props = defineProps({ modelValue: Boolean, item: Object, company: String })
const emit = defineEmits(["update:modelValue", "supplier-selected"])
const show = ref(props.modelValue)
const suppliers = ref([])
const selectedSupplier = ref("")
const loading = ref(false)
const errorMessage = ref("")
const selectedRow = computed(() => suppliers.value.find((row) => row.supplier === selectedSupplier.value))

watch(() => props.modelValue, (value) => {
	show.value = value
	if (value) loadSuppliers()
})
watch(show, (value) => emit("update:modelValue", value))

async function loadSuppliers() {
	suppliers.value = []
	selectedSupplier.value = ""
	errorMessage.value = ""
	loading.value = true
	try {
		const rows = await call("auto_insurance.insurance_v2.api.get_category_suppliers", {
			item_code: props.item?.item_code,
			company: props.company,
		})
		suppliers.value = rows || []
		if (suppliers.value.length === 1) selectedSupplier.value = suppliers.value[0].supplier
		if (!suppliers.value.length) errorMessage.value = __("No Insurance Supplier is configured for this item and company.")
	} catch (error) {
		console.error("[InsuranceSupplierDialog] Unable to load suppliers:", error)
		errorMessage.value = __("Unable to load Insurance Suppliers.")
	} finally {
		loading.value = false
	}
}

function handleConfirm() {
	if (!selectedRow.value) return
	emit("supplier-selected", {
		insurance_supplier: selectedRow.value.supplier,
		insurance_supplier_address: selectedRow.value.supplier_address || null,
	})
	show.value = false
}
</script>
