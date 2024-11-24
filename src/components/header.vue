<template>
	<div class="list-header">
		<div class="start">
			<v-chip v-if="selectionSync.length > 0" :label="false" color="primary" clickable
				@click="selectionSync = []">
				<v-icon name="cancel" outline></v-icon>
				<span>{{ t("n_items_selected", selectionSync.length) }}</span>
			</v-chip>
			<v-chip v-else :label="false" clickable
				@click="showSelect === 'multiple' ? $emit('select-all') : undefined">
				<v-icon name="check_circle" outline></v-icon>
				<span>{{ t(showSelect === 'multiple' ? 'select_all' : 'select_an_item') }}</span>
			</v-chip>
		</div>
		<div class="end">
			<!-- Sort Selector -->
			<v-chip :label="false">
				<v-menu show-arrow placement="bottom">
					<template #activator="{ toggle }">
						<div v-tooltip.top="t('sort_field')" class="sort-selector" @click="toggle">
							{{ sortField?.name || t('sort_field') }}
						</div>
					</template>

					<v-list>
						<v-list-item v-for="field in fieldsWithoutFake" :key="field.field"
							:active="field.field === sortKey" clickable @click="sortSync = [field.field]">
							<v-list-item-content>{{ field.name }}</v-list-item-content>
						</v-list-item>
					</v-list>
				</v-menu>
				<v-icon v-tooltip.top="t('sort_direction')" class="sort-direction" :class="{ descending }" name="sort"
					clickable @click="toggleDescending"></v-icon>
			</v-chip>
			<v-list>
				<v-list-item v-for="field in selectedFields" :key="field.field">
					{{ field.name }}
				</v-list-item>
			</v-list>

			<!-- Field selector-->
			<div class="field-selector">
				<v-menu placement="bottom-end" show-arrow :close-on-content-click="false">
					<template #activator="{ toggle, active }">
						<v-icon v-tooltip="t('add_field')" class="add-field" name="add" :class="{ active }" clickable
							@click.stop="toggle"></v-icon>
					</template>

					<v-field-list :collection="collection" :disabled-fields="disabledFields" :allow-select-all="false"
						@add="addField">
					</v-field-list>
				</v-menu>
			</div>
		</div>
	</div>
</template>

<script lang="ts">
import { useI18n } from "vue-i18n";
import { defineComponent, PropType, computed, ref } from "vue";
import { Field, ShowSelect } from "@directus/types";
import { useSync } from "@directus/composables";

export default defineComponent({
	props: {
		collection: {
			type: String,
			required: true,
		},
		fields: {
			type: Array as PropType<Field[]>,
			default: () => [],  // Provide an empty array as default
		},
		fieldsInCollection: {  // Add this prop to get all available fields
			type: Array as PropType<Field[]>,
			required: true,
		},
		size: {
			type: Number,
			required: true,
		},
		sort: {
			type: Array as PropType<string[]>,
			required: true,
		},
		showSelect: {
			type: String as PropType<ShowSelect>,
			default: "multiple",
		},
		selection: {
			type: Array as PropType<Record<string, any>>,
			default: () => [],
		},
		selectedField: {
			type: String,
			default: null,
		},
	},

	emits: [
		"select-all",
		"update:size",
		"update:sort",
		"update:selection",
		"update:selectedField",
		"update:fields"
	],

	setup(props, { emit }) {
		const { t } = useI18n();

		// Sync handlers
		const sizeSync = useSync(props, "size", emit);
		const sortSync = useSync(props, "sort", emit);
		const selectionSync = useSync(props, "selection", emit);
		const selectedFields = ref<Field[]>([]); // List of selected fields


		// Track selected fields
		const selectedFieldNames = computed(() =>
			props.fields.map(field => field.field)
		);

		// Get disabled fields that shouldn't be selectable
		const disabledFields = computed(() => {
			// Example logic: Disable only "chips" field
			return props.fields.filter(field => field.type === 'json').map(field => field.field);
		});



		// Handle adding new fields
		function addField(fieldKey: string | string[]) {
			const key = Array.isArray(fieldKey) ? fieldKey[0] : fieldKey;
			console.log("Resolved field key:", key);

			const fieldToAdd = props.fields.find(f => f.field === key);
			console.log("Field found in props.fields:", fieldToAdd);

			if (fieldToAdd && !selectedFields.value.some(field => field.field === key)) {
				// Debug before updating the selectedFields array
				console.log("Before adding field:", selectedFields.value);

				// Reactively update the array
				selectedFields.value = [...selectedFields.value, fieldToAdd];

				// Debug after updating the selectedFields array
				console.log("After adding field:", selectedFields.value);

				// Emit the updated fields list to the parent
				emit('update:fields', [...props.fields, fieldToAdd]);
			}
			console.log("Selected fields list:", selectedFields.value);
		}





		// Sort handling 
		const descending = computed(() => props.sort[0]?.startsWith("-"));
		const sortKey = computed(() =>
			props.sort[0]?.startsWith("-") ? props.sort[0].substring(1) : props.sort[0]
		);

		const sortField = computed(() =>
			props.fields.find(field => field.field === sortKey.value)
		);

		function toggleDescending() {
			if (!sortSync.value?.[0]) return;
			sortSync.value = descending.value
				? [sortSync.value[0].substring(1)]
				: [`-${sortSync.value[0]}`];
		}

		const fieldsWithoutFake = computed(() => {
			return props.fields
				.filter(field => !field.field.startsWith("$"))
				.map(field => ({
					field: field.field,
					name: field.name,
					type: field.type,
					disabled: ["json", "file", "files", "alias", "presentation"].includes(field.type),
				}));
		});



		return {
			t,
			addField,
			selectedFields,
			descending,
			sortField,
			sortKey,
			fieldsWithoutFake,
			sizeSync,
			sortSync,
			selectionSync,
			toggleDescending,
			disabledFields
		};
	}
});
</script>

<style lang="scss" scoped>
.list-header {
	position: sticky;
	top: var(--layout-offset-top);
	z-index: 4;
	display: flex;
	align-items: center;
	justify-content: space-between;
	width: 100%;
	height: 52px;
	margin-bottom: 36px;
	background-color: var(--background-page);
	border-top: var(--border-width) solid var(--border-subdued);
	border-bottom: var(--border-width) solid var(--border-subdued);
	box-shadow: 0 0 0 2px var(--background-page);
	padding: var(--content-padding);
	padding-bottom: 0px;
	padding-top: 0px;
}

.v-chip {
	gap: 4px;
}

.start {

	.select-all,
	.selected {
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 4px;
		height: 100%;
		line-height: normal;
		padding: 0;
	}
}

.end {
	display: flex;
	align-items: center;
	gap: 8px;
	color: var(--foreground-subdued);

	.field-selector,
	.sort-selector {
		display: flex;
		align-items: center;
		gap: 4px;
		margin-right: 8px;
		transition: color var(--fast) var(--transition);

		&:hover {
			color: var(--foreground-normal);
			cursor: pointer;
		}
	}

	.sort-direction {
		transition: color var(--fast) var(--transition);

		&.descending {
			transform: scaleY(-1);
		}

		&:hover {
			color: var(--foreground-normal);
			cursor: pointer;
		}
	}
}
</style>