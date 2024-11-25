<template>
	<v-list-item class="list-item" :clickable="true" @click="handleClick" :class="{
		loading,
		readonly,
		selected: item && modelValue.includes(item[itemKey]),
		'select-mode': selectMode,
	}">
		<!-- Selection button -->
		<v-button icon rounded @click.stop="toggleSelection" :secondary="!isSelected" class="selection-button">
			<v-icon :name="selectionIcon" />
		</v-button>

		<div class="content-wrapper">
			<!-- Main content area with fields -->
			<div class="fields-row">
				<render-template v-for="(field, index) in fields" :key="field.field" class="field"
					:class="{ 'bold-field': index === 0 }" :collection="collection" :item="item"
					:template="`{{${field.field}}}`" />
			</div>

			<!-- Tags section -->
			<div v-if="tag" class="tags-section">
				<render-template :collection="collection" :item="item" :template="tag" />
			</div>
		</div>
	</v-list-item>
</template>

<script lang="ts">
import { computed, defineComponent, PropType, ref } from "vue";
import { useI18n } from "vue-i18n";
import { useRouter } from "vue-router";

type File = {
	id: string;
	type: string;
	modified_on: Date;
	[key: string]: any;
};

export default defineComponent({
	props: {
		collection: {
			type: String,
			default: null,
		},
		fields: {
			type: Array as PropType<Field[]>,
			default: () => [],  // Provide empty array as default
		},
		icon: {
			type: String,
			default: "box",
		},
		file: {
			type: Object as PropType<File>,
			default: null,
		},
		loading: {
			type: Boolean,
			default: false,
		},
		item: {
			type: Object as PropType<Record<string, any>>,
			default: null,
		},
		modelValue: {
			type: Array as PropType<(string | number)[]>,
			default: () => [],
		},
		selectMode: {
			type: Boolean,
			default: false,
		},
		to: {
			type: String,
			default: "",
		},
		readonly: {
			type: Boolean,
			default: false,
		},
		itemKey: {
			type: String,
			required: true,
		},
		imageSource: {
			type: String,
			default: null,
		},
		title: {
			type: String,
			default: "",
		},
		subtitle: {
			type: String,
			default: "",
		},
		imageFit: {
			type: String,
			default: "contain",
		},
		tag: {
			type: [String, Boolean],
			default: false,
		},
		size: {
			type: Number,
			default: 1,
		},
		idShow: {
			type: Boolean,
			default: false,
		},
	},

	emits: ["update:modelValue"],

	setup(props, { emit }) {
		const router = useRouter();
		const { t } = useI18n();
		const imgError = ref(false);
		const isSubtitleExpanded = ref(false);

		const isSelected = computed(() =>
			props.modelValue.includes(props.item?.[props.itemKey])
		);



		const selectionIcon = computed(() => {
			if (!props.item) return "radio_button_unchecked";
			return isSelected.value ? "check_circle" : "radio_button_unchecked";
		});

		function toggleSubtitle() {
			isSubtitleExpanded.value = !isSubtitleExpanded.value;
		}

		function toggleSelection() {
			if (!props.item) return null;

			if (props.modelValue.includes(props.item[props.itemKey])) {
				emit(
					"update:modelValue",
					props.modelValue.filter(
						(key) => key !== props.item[props.itemKey]
					)
				);
			} else {
				emit(
					"update:modelValue",
					[...props.modelValue, props.item[props.itemKey]]
				);
			}
		}

		function handleClick(): void {
			if (props.selectMode === true) {
				toggleSelection();
			} else {
				router.push(props.to);
			}
		}

		function imgFit(): string {
			return props.imageFit === "crop" ? "object-cover" : "object-contain";
		}

		function statusClass(status: string): string {
			return status === "published"
				? "color_primary"
				: status === "archived"
					? "color_archire"
					: "color_sub";
		}

		return {
			t,
			selectionIcon,
			toggleSelection,
			handleClick,
			imgError,
			imgFit,
			statusClass,
			isSelected,
			isSubtitleExpanded,
			toggleSubtitle,
		};
	},
});
</script>

<style lang="scss" scoped>
.content-wrapper {
	width: 100%;
	/* Ensure it takes up the full width of the parent container */
	box-sizing: border-box;
	/* Ensures padding/borders do not affect width */
}

.fields-row {
	display: flex;
	gap: 24px;
	width: 100%;
}

.field {
	min-width: 0;
	width: 100%;
	/* Ensures each field takes up equal width */
	display: flex;
	flex-direction: column;
	align-items: flex-start;
	box-sizing: border-box;
	border-right: 2px solid var(--theme--primary-background);
	word-wrap: break-word;
	white-space: normal;
	overflow-wrap: break-word;
	word-break: break-word;
}

.bold-field {
	font-weight: bold;
}

/* Apply word wrapping and ensure fields display correctly */
::v-deep .field {
	word-wrap: break-word;
	white-space: normal;
	overflow-wrap: break-word;
	word-break: break-word;
}



.field:not(:last-child) {
	padding-right: 16px;
	/* Space between fields */
}

/* Ensuring equal column widths and making sure content inside doesn't break the layout */
.selection-button {
	flex-shrink: 0;
}

.tags-section {
	width: 100%;
	padding-top: 8px;
	border-top: 1px solid var(--theme--border);
}

.v-list-item {
	padding: 12px;
	gap: 8px;

	&-content {
		display: flex;
		flex-direction: column;
		gap: 8px;
		width: 100%;
	}

	/* List item states */
	&.selected {
		background-color: var(--theme--background-accent);
	}

	&.readonly {
		pointer-events: none;
		opacity: 0.75;
	}
}

.selected-button,
.unselected-button {
	align-self: flex-start;
}

.tags {
	align-content: flex-start;
}

.field-selector {
	flex: 0;
	/* Don't grow */
	margin-left: auto;
	/* Push to the right */
}

.selector-trigger {
	display: flex;
	align-items: center;
	gap: 8px;
	padding: 4px 8px;
	cursor: pointer;

	&:hover {
		color: var(--theme--foreground-normal);
	}

	.v-icon.active {
		color: var(--theme--primary);
	}
}
</style>
