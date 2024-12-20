<template>
	<div ref="layoutElement" :style="{
		'--size': width >= 600 ? size : 1,
	}">
		<template v-if="loading || itemCount > 0">
			<cards-header :collection="collection" v-model:size="sizeWritable" v-model:selection="selectionWritable"
				v-model:sort="sortWritable" v-model:selectedFields="selectedFieldsWritable" :fields="fieldsInCollection"
				:show-select="showSelect" @select-all="selectAll" />

			<div class="grid" :class="{ 'single-row': isSingleRow }">
				<v-list>
					<card v-for="item in items" :key="item[primaryKeyField.field]" v-model="selectionWritable"
						:fields="selectedFieldsWritable" :item-key="primaryKeyField.field" :collection="collection" :item="item"
						:imageFit="imageFit" :icon="icon" :tag="tag" :idShow="idShow" :title="title"
						:subtitle="subtitle" :imageSource="imageSource ? item[imageSource] : null"
						:select-mode="selectMode || (selection && selection.length > 0)" :to="getLinkForItem(item)"
						:readonly="readonly" :size="size" />
				</v-list>
			</div>
			<div class="footer">
				<div class="pagination">
					<v-pagination v-if="totalPages > 1" :length="totalPages" :total-visible="7" show-first-last
						:model-value="page" @update:model-value="toPage" />
				</div>

				<div v-if="loading === false && items.length >= 25" class="per-page">
					<span>{{ t("per_page") }}</span>
					<v-select :model-value="`${limit}`" :items="['25', '50', '100', '250', '500', '1000']" inline
						@update:model-value="limitWritable = +$event" />
				</div>
			</div>
		</template>

		<v-info v-else-if="error" type="danger" :title="t('unexpected_error')" icon="error" center>
			{{ t("unexpected_error_copy") }}
			<template #append>
				<v-error :error="error" />
				<v-button small class="reset-preset" @click="resetPresetAndRefresh">
					{{ t("reset_page_preferences") }}
				</v-button>
			</template>
		</v-info>
		<slot v-else-if="itemCount === 0 && (filter || search)" name="no-results" />
		<slot v-else-if="itemCount === 0" name="no-items" />
	</div>
</template>


<script lang="ts">
import { useI18n } from "vue-i18n";
import {
	defineComponent,
	watch,
	PropType,
	ref,
	inject,
	Ref,
	toRefs,
} from "vue";

import Card from "./components/card.vue";
import CardsHeader from "./components/header.vue";
import {
	Field,
	Item,
} from "@directus/types";
import {
	useSync,
	useElementSize,
} from "@directus/composables";
import {
	Filter,
	ShowSelect,
} from "@directus/types";

export default defineComponent({
	components: { Card, CardsHeader },
	inheritAttrs: false,
	props: {
		modelValue: {
			type: Array as PropType<(string | number)[] | Item[]>, // Accepts both
			default: () => [],
		},
		size: {
			type: Number,
			default: 1
		},
		collection: {
			type: String,
			required: true,
		},
		selection: {
			type: Array as PropType<
				Item[]
			>,
			required: true,
		},
		selectedFields: {
			type: Array as PropType<Field[]>,
			default: () => [],
		},
		showSelect: {
			type: String as PropType<ShowSelect>,
			default: "multiple",
		},
		selectMode: {
			type: Boolean,
			required: true,
		},
		readonly: {
			type: Boolean,
			required: true,
		},
		items: {
			type: Array as PropType<
				Item[]
			>,
			required: true,
		},
		loading: {
			type: Boolean,
			required: true,
		},
		error: {
			type: Object as PropType<any>,
			default: null,
		},
		totalPages: {
			type: Number,
			required: true,
		},
		page: {
			type: Number,
			required: true,
		},
		toPage: {
			type: Function as PropType<
				(
					newPage: number
				) => void
			>,
			required: true,
		},
		itemCount: {
			type: Number,
			default: null,
		},
		fieldsInCollection: {
			type: Array as PropType<Field[]>,
			required: true,
		},

		limit: {
			type: Number,
			required: true,
		},
		primaryKeyField: {
			type: Object as PropType<Field>,
			default: null,
		},
		icon: {
			type: String,
			required: true,
		},
		imageSource: {
			type: String,
			default: null,
		},
		title: {
			type: String,
			default: null,
		},
		subtitle: {
			type: String,
			default: null,
		},
		getLinkForItem: {
			type: Function as PropType<
				(
					item: Record<
						string,
						any
					>
				) => string | undefined
			>,
			required: true,
		},
		imageFit: {
			type: String,
		},
		sort: {
			type: Array as PropType<
				string[]
			>,
			required: true,
		},
		info: {
			type: Object,
			default: null,
		},
		isSingleRow: {
			type: Boolean,
			required: true,
		},
		width: {
			type: Number,
			required: true,
		},
		selectAll: {
			type: Function as PropType<
				() => void
			>,
			required: true,
		},
		resetPresetAndRefresh: {
			type: Function as PropType<
				() => Promise<void>
			>,
			required: true,
		},
		filter: {
			type: Object as PropType<Filter>,
			default: null,
		},
		search: {
			type: String,
			default: null,
		},
		tag: {
			type: String,
			default: null,
		},
		idShow: {
			type: Boolean,
		},
	},
	emits: [
		"update:selection",
		"update:limit",
		"update:size",
		"update:sort",
		"update:width",
		"update:selectedFields"
	],
	setup(props, { emit }) {
		const { t } = useI18n();
		const selectedFields = ref([]);
		const selectionWritable =
			useSync(
				props,
				"selection",
				emit
			);

		const limitWritable = useSync(
			props,
			"limit",
			emit
		);

		const sizeWritable = useSync(
			props,
			"size",
			emit
		);
		const sortWritable = useSync(
			props,
			"sort",
			emit
		);

		const selectedFieldsWritable = useSync(
			props,
			"selectedFields", // Sync with the selectedFields prop
			emit
		);




		const mainElement = inject<
			Ref<Element | undefined>
		>("main-element");

		const layoutElement =
			ref<HTMLElement>();

		const { width } =
			useElementSize(
				layoutElement
			);

		watch(() => props.page, () => {
			if (mainElement?.value) { // Add this conditional check
				mainElement.value.scrollTo({
					top: 0,
					behavior: "smooth",
				});
			}
		});

		watch(width, () => {
			emit(
				"update:width",
				width.value
			);
		});

		return {
			t,
			selectionWritable,
			limitWritable,
			sizeWritable,
			sortWritable,
			layoutElement,
			width,
			selectedFields,
			selectedFieldsWritable
		};
	},
});
</script>

<style lang="scss" scoped>
.grid {
	display: grid;
	gap: 8px;
	/* Keeps spacing between grid items */
	grid-template-columns: 1fr;
	/* Single-column layout */
	padding: var(--content-padding) !important;
	/* Ensures proper padding */
	padding-bottom: 0;
	/* Optional: Adjust as needed */
	padding-top: 0;
	/* Optional: Adjust as needed */
}

.single-row {
	max-width: 100%;
	/* Ensures the layout spans the full width */
}

.single-row>.grid {
	grid-template-columns: 1fr;
	padding: var(--content-padding);
	/* Standard padding */
	padding-bottom: 0;
	padding-top: 0;
	/* Explicit single-column layout for nested grids */
}




.footer {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding-top: 40px;

	.pagination {
		display: inline-block;
	}

	.per-page {
		display: flex;
		align-items: center;
		justify-content: flex-end;
		width: 240px;
		color: var(--foreground-subdued);

		span {
			width: auto;
			margin-right: 4px;
		}

		.v-select {
			color: var(--foreground-normal);
		}
	}
}

.reset-preset {
	margin-top: 24px;
}
</style>
