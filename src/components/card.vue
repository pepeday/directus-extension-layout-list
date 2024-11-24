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
				<div v-if="title" class="field">
					<render-template :collection="collection" :item="item" :template="title" />
				</div>
				<div v-if="subtitle" class="field">
					<render-template :collection="collection" :item="item" :template="subtitle" />
				</div>
				<div class="field">
					<span>Status: {{ item?.status }}</span>
				</div>

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


		function handleFieldSelection(field: string) {
			console.log('Selected field:', field);
			// We'll implement the field handling later
		}

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
.list-item {
	padding: 12px;
	display: flex;
	gap: 12px;
	width: 100%;

	&.selected {
		background-color: var(--theme--background-accent);
	}

	&.readonly {
		opacity: 0.75;
		pointer-events: none;
	}
}

.content-wrapper {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 12px;
}

.fields-row {
	display: flex;
	align-items: center;
	gap: 24px;
	width: 100%;

	@media (max-width: 768px) {
		flex-direction: column;
		align-items: flex-start;
		gap: 8px;
	}
}

.field {
	flex: 1;
	min-width: 0;

	&:deep(.render-template) {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}
}

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

	// List item states
	&.selected {
		background-color: var(--theme--background-accent);
	}

	&.readonly {
		pointer-events: none;
		opacity: 0.75;
	}
}

// Selection buttons
.selected-button,
.unselected-button {
	align-self: flex-start;
}

// Main content layout
.fields-container {
	display: grid;
	grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
	gap: 12px;
	width: 100%;

	@media (min-width: 1201px) {
		grid-template-columns: repeat(4, 1fr); // Show 4 columns on large screens
	}

	@media (max-width: 1200px) {
		grid-template-columns: repeat(3, 1fr); // Show 3 columns on medium screens
	}

	@media (max-width: 900px) {
		grid-template-columns: repeat(2, 1fr); // Show 2 columns on smaller screens
	}

	@media (max-width: 600px) {
		grid-template-columns: 1fr; // Stack all fields on mobile
	}
}

.field-wrapper {
	min-width: 0;
	width: 100%;
	overflow-wrap: break-word;
	word-break: break-word;

	// Ensure consistent wrapping
	&:deep(.render-template) {
		overflow: visible;
		white-space: normal;
	}
}



// Subtitle toggle
.toggle-subtitle-button {
	align-self: flex-start;
}

.tags {
	align-content: flex-start;
}

.field-selector {
	flex: 0; // Don't grow
	margin-left: auto; // Push to the right
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