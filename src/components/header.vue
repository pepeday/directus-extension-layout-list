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
			<!-- Fields Selector -->
			<v-chip :label="false" color="primary" clickable>
				<v-menu placement="bottom-end" show-arrow>
					<template #activator="{ toggle }">
						<div class="selected-fields-chip" @click="toggle">
							<v-icon name="list" outline></v-icon>
							<span>{{ selectedFieldsSync.length }} fields</span>
						</div>
					</template>

					<!-- Popup List -->
					<v-list>
						<v-list-item v-for="(field, index) in selectedFieldsSync" :key="field.field">
							<v-list-item-action>
								<!-- Remove icon -->
								<v-icon name="close" clickable @click.stop="removeField(field.field)"
									title="Remove"></v-icon>
							</v-list-item-action>
							<v-list-item-content>{{ field.name }}</v-list-item-content>
							<v-list-item-action>
								<!-- Up Arrow -->
								<v-icon v-if="index > 0" name="arrow_upward" clickable
									@click.stop="moveFieldUp(index)"></v-icon>
								<!-- Down Arrow -->
								<v-icon v-if="index < selectedFieldsSync.length - 1" name="arrow_downward" clickable
									@click.stop="moveFieldDown(index)"></v-icon>
							</v-list-item-action>
						</v-list-item>
					</v-list>
				</v-menu>
				<v-menu placement="bottom-end" show-arrow :close-on-content-click="false">
					<template #activator="{ toggle, active }">
						<v-icon v-tooltip="t('add_field')" class="add-field" name="add" :class="{ active }" clickable
							@click.stop="toggle"></v-icon>
					</template>

					<v-field-list :collection="collection" :disabled-fields="disabledFields" :allow-select-all="false"
						@add="addField">
					</v-field-list>

				</v-menu>
			</v-chip>

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
		selectedFields: {
			type: Array as PropType<Field[]>,
			default: () => [], // Provide an empty array as the default value
		},
	},

	emits: [
		"select-all",
		"update:size",
		"update:sort",
		"update:selection",
		"update:selectedField",
		"update:fields",
		"update:selectedFields"
	],

	setup(props, { emit }) {
		const { t } = useI18n();

		// Sync handlers
		const sizeSync = useSync(props, "size", emit);
		const sortSync = useSync(props, "sort", emit);
		const selectionSync = useSync(props, "selection", emit);
		const selectedFieldsSync = useSync(props, "selectedFields", emit);
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



		// Handle adding and removing fields
		function addField(fieldKey: string | string[]) {
			const key = Array.isArray(fieldKey) ? fieldKey[0] : fieldKey;

			const fieldToAdd = props.fields.find(f => f.field === key);

			if (fieldToAdd && !selectedFieldsSync.value.some(field => field.field === key)) {
				// Reactively update the array
				selectedFieldsSync.value = [...selectedFieldsSync.value, fieldToAdd];

				// Emit the updated fields list to the parent
				emit('update:selectedFields', [...selectedFieldsSync.value, fieldToAdd]);

			}
		}

		function removeField(fieldKey) {
			selectedFieldsSync.value = selectedFieldsSync.value.filter(f => f.field !== fieldKey);

		}

		function moveFieldUp(index: number) {
			if (index === 0) return; // Already at the top
			const fields = [...selectedFieldsSync.value];
			[fields[index - 1], fields[index]] = [fields[index], fields[index - 1]];
			selectedFieldsSync.value = fields;
			emit('update:selectedFields', fields); // Emit the updated order
		}

		function moveFieldDown(index: number) {
			if (index === selectedFieldsSync.value.length - 1) return; // Already at the bottom
			const fields = [...selectedFieldsSync.value];
			[fields[index], fields[index + 1]] = [fields[index + 1], fields[index]];
			selectedFields.value = fields;
			emit('update:selectedFields', fields); // Emit the updated order
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
			removeField,
			selectedFields,
			selectedFieldsSync,
			moveFieldUp,
			moveFieldDown,
			descending,
			sortField,
			sortKey,
			fieldsWithoutFake,
			sizeSync,
			sortSync,
			selectionSync,
			toggleDescending,
			disabledFields,


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