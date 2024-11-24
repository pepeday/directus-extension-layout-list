<template>
	<div class="content">
		<!-- Title and Subtitle -->
		<div class="title-subtitle" :class="{ 'subtitle-hidden': !(isExpanded || isLargeScreen) }">
			<render-template v-if="title" class="title" :collection="collection" :item="item" :template="title" />
			<render-template v-if="(isExpanded || isLargeScreen) && subtitle" class="subtitle" :collection="collection"
				:item="item" :template="subtitle" />
		</div>

		<!-- Tags -->
	</div>
</template>


<script lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from "vue";
import { useI18n } from "vue-i18n";
import { defineComponent, PropType } from "vue";
import { LayoutOptions } from "../types";

export default defineComponent({
	props: {
		isSubtitleExpanded: {
			type: Boolean,
			default: false,
		},
		layoutOptions: {
			type: Object as PropType<LayoutOptions>,
			required: false,
		},
		collection: {
			type: Object as PropType<any>,
			default: null,
		},
		title: {
			type: String,
			default: "Title",
		},
		subtitle: {
			type: String,
			default: "Sub title",
		},
		tag: {
			type: String,
			default: null,
		},
		status: {
			type: String,
			default: null,
		},
		id: {
			type: [String, Number],
			default: null,
		},
		selectMode: {
			type: String,
			default: false,
		},
		statusClass: {
			type: String,
			default: "color_primary",
		},
		item: {
			type: Object,
			default: null,
		},
		idShow: {
			type: Boolean,
		},
	},
	setup(props, { emit }) {
		const isExpanded = ref(props.isSubtitleExpanded);
		const isLargeScreen = ref(false);
		watch(() => props.isSubtitleExpanded, (newValue) => {
			if (!isLargeScreen.value) {
				isExpanded.value = newValue;
			}
		});
		const checkScreenSize = () => {
			if (typeof window !== "undefined") {
				const wasLargeScreen = isLargeScreen.value;
				isLargeScreen.value = window.innerWidth >= 900;

				// Only update expanded state if screen size category changed
				if (wasLargeScreen !== isLargeScreen.value) {
					isExpanded.value = isLargeScreen.value;
					emit("update:isSubtitleExpanded", isExpanded.value);
				}
			}
		};


		onMounted(() => {
			checkScreenSize();
			window.addEventListener("resize", checkScreenSize);
		});

		onBeforeUnmount(() => {
			window.removeEventListener("resize", checkScreenSize);
		});

		const toggleSubtitle = () => {
			if (!isLargeScreen.value) {
				isExpanded.value = !isExpanded.value; // Allow toggling on small screens
				emit("update:isSubtitleExpanded", isExpanded.value);
			}
		};


		return {
			isExpanded,
			toggleSubtitle,
			isLargeScreen,
		};
	},
});
</script>

<style lang="scss" scoped>
.content {
	display: block;
	width: 100%;

	.title-subtitle {
		display: grid;
		width: 100%;
		gap: var(--theme-form-column-gap);

		// Small screens: default to one column
		grid-template-columns: 1fr;

		// Large screens
		@media (min-width: 900px) {
			// When expanded (default on large screens)
			grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);

			// When hidden (should never happen on large screens due to our JS)
			&.subtitle-hidden {
				grid-template-columns: 1fr;
			}
		}
	}

	.title {
		font-size: 1rem;
		font-weight: bold;
		color: var(--text);
		overflow: hidden;
		text-overflow: ellipsis;
		display: -webkit-box;
		-webkit-box-orient: vertical;
		-webkit-line-clamp: 3;
		line-clamp: 3;
	}

	.subtitle {
		font-size: 1rem;
		font-weight: normal;
		color: var(--text);
		overflow: hidden;
		text-overflow: ellipsis;
		display: -webkit-box;
		-webkit-box-orient: vertical;
		-webkit-line-clamp: 5;
		line-clamp: 5;

		@media (min-width: 900px) {
			-webkit-line-clamp: unset;
			overflow: visible;
		}
	}
}
</style>
