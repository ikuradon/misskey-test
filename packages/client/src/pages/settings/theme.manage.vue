<template>
<div class="_formRoot">
	<FormSelect v-model="selectedThemeId" class="_formBlock">
		<template #label>{{ i18n.ts.theme }}</template>
		<optgroup :label="i18n.ts._theme.installedThemes">
			<option v-for="x in installedThemes" :key="x.id" :value="x.id">{{ x.name }}</option>
		</optgroup>
		<optgroup :label="i18n.ts._theme.builtinThemes">
			<option v-for="x in builtinThemes" :key="x.id" :value="x.id">{{ x.name }}</option>
		</optgroup>
	</FormSelect>
	<template v-if="selectedTheme">
		<FormInput readonly :modelValue="selectedTheme.author" class="_formBlock">
			<template #label>{{ i18n.ts.author }}</template>
		</FormInput>
		<FormTextarea v-if="selectedTheme.desc" readonly :modelValue="selectedTheme.desc" class="_formBlock">
			<template #label>{{ i18n.ts._theme.description }}</template>
		</FormTextarea>
		<FormTextarea readonly tall :modelValue="selectedThemeCode" class="_formBlock">
			<template #label>{{ i18n.ts._theme.code }}</template>
			<template #caption><button class="_textButton" @click="copyThemeCode()">{{ i18n.ts.copy }}</button></template>
		</FormTextarea>
		<FormButton v-if="!builtinThemes.some(t => t.id == selectedTheme.id)" class="_formBlock" danger @click="uninstall()"><i class="fas fa-trash-alt"></i> {{ i18n.ts.uninstall }}</FormButton>
	</template>
</div>
</template>

<script lang="ts" setup>
import { computed, defineExpose, ref } from 'vue';
import JSON5 from 'json5';
import FormTextarea from '@/components/form/textarea.vue';
import FormSelect from '@/components/form/select.vue';
import FormInput from '@/components/form/input.vue';
import FormButton from '@/components/ui/button.vue';
import { Theme, builtinThemes } from '@/scripts/theme';
import copyToClipboard from '@/scripts/copy-to-clipboard';
import * as os from '@/os';
import { getThemes, removeTheme } from '@/theme-store';
import * as symbols from '@/symbols';
import { i18n } from '@/i18n';

const installedThemes = ref(getThemes());
const selectedThemeId = ref(null);

const themes = computed(() => builtinThemes.concat(installedThemes.value));

const selectedTheme = computed(() => {
	if (selectedThemeId.value == null) return null;
	return themes.value.find(x => x.id === selectedThemeId.value);
});

const selectedThemeCode = computed(() => {
	if (selectedTheme.value == null) return null;
	return JSON5.stringify(selectedTheme.value, null, '\t');
});

function copyThemeCode() {
	copyToClipboard(selectedThemeCode.value);
	os.success();
}

function uninstall() {
	removeTheme(selectedTheme.value as Theme);
	installedThemes.value = installedThemes.value.filter(t => t.id !== selectedThemeId.value);
	selectedThemeId.value = null;
	os.success();
}

defineExpose({
	[symbols.PAGE_INFO]: {
		title: i18n.ts._theme.manage,
		icon: 'fas fa-folder-open',
		bg: 'var(--bg)',
	}
});
</script>
