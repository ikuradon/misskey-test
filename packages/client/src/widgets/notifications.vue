<template>
<MkContainer :style="`height: ${widgetProps.height}px;`" :show-header="widgetProps.showHeader" :scrollable="true">
	<template #header><i class="fas fa-bell"></i>{{ $ts.notifications }}</template>
	<template #func><button class="_button" @click="configureNotification()"><i class="fas fa-cog"></i></button></template>

	<div>
		<XNotifications :include-types="widgetProps.includingTypes"/>
	</div>
</MkContainer>
</template>

<script lang="ts" setup>
import { GetFormResultType } from '@/scripts/form';
import { useWidgetPropsManager, Widget, WidgetComponentEmits, WidgetComponentExpose, WidgetComponentProps } from './widget';
import MkContainer from '@/components/ui/container.vue';
import XNotifications from '@/components/notifications.vue';
import * as os from '@/os';
import { defineAsyncComponent } from 'vue';

const name = 'notifications';

const widgetPropsDef = {
	showHeader: {
		type: 'boolean' as const,
		default: true,
	},
	height: {
		type: 'number' as const,
		default: 300,
	},
	includingTypes: {
		type: 'array' as const,
		hidden: true,
		default: null,
	},
};

type WidgetProps = GetFormResultType<typeof widgetPropsDef>;

// 現時点ではvueの制限によりimportしたtypeをジェネリックに渡せない
//const props = defineProps<WidgetComponentProps<WidgetProps>>();
//const emit = defineEmits<WidgetComponentEmits<WidgetProps>>();
const props = defineProps<{ widget?: Widget<WidgetProps>; }>();
const emit = defineEmits<{ (e: 'updateProps', props: WidgetProps); }>();

const { widgetProps, configure, save } = useWidgetPropsManager(name,
	widgetPropsDef,
	props,
	emit,
);

const configureNotification = () => {
	os.popup(defineAsyncComponent(() => import('@/components/notification-setting-window.vue')), {
		includingTypes: widgetProps.includingTypes,
	}, {
		done: async (res) => {
			const { includingTypes } = res;
			widgetProps.includingTypes = includingTypes;
			save();
		}
	}, 'closed');
};

defineExpose<WidgetComponentExpose>({
	name,
	configure,
	id: props.widget ? props.widget.id : null,
});
</script>
