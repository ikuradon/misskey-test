<template>
<div class="_formRoot">
	<FormInfo warn class="_formBlock">{{ i18n.ts._plugin.installWarn }}</FormInfo>

	<FormTextarea v-model="code" tall class="_formBlock">
		<template #label>{{ i18n.ts.code }}</template>
	</FormTextarea>

	<div class="_formBlock">
		<FormButton :disabled="code == null" primary inline @click="install"><i class="fas fa-check"></i> {{ i18n.ts.install }}</FormButton>
	</div>
</div>
</template>

<script lang="ts" setup>
import { defineExpose, defineAsyncComponent, nextTick, ref } from 'vue';
import { AiScript, parse } from '@syuilo/aiscript';
import { serialize } from '@syuilo/aiscript/built/serializer';
import { v4 as uuid } from 'uuid';
import FormTextarea from '@/components/form/textarea.vue';
import FormButton from '@/components/ui/button.vue';
import FormInfo from '@/components/ui/info.vue';
import * as os from '@/os';
import { ColdDeviceStorage } from '@/store';
import { unisonReload } from '@/scripts/unison-reload';
import { i18n } from '@/i18n';
import * as symbols from '@/symbols';

const code = ref(null);

function installPlugin({ id, meta, ast, token }) {
	ColdDeviceStorage.set('plugins', ColdDeviceStorage.get('plugins').concat({
		...meta,
		id,
		active: true,
		configData: {},
		token: token,
		ast: ast
	}));
}

async function install() {
	let ast;
	try {
		ast = parse(code.value);
	} catch (e) {
		os.alert({
			type: 'error',
			text: 'Syntax error :('
		});
		return;
	}

	const meta = AiScript.collectMetadata(ast);
	if (meta == null) {
		os.alert({
			type: 'error',
			text: 'No metadata found :('
		});
		return;
	}

	const data = meta.get(null);
	if (data == null) {
		os.alert({
			type: 'error',
			text: 'No metadata found :('
		});
		return;
	}

	const { name, version, author, description, permissions, config } = data;
	if (name == null || version == null || author == null) {
		os.alert({
			type: 'error',
			text: 'Required property not found :('
		});
		return;
	}

	const token = permissions == null || permissions.length === 0 ? null : await new Promise((res, rej) => {
		os.popup(defineAsyncComponent(() => import('@/components/token-generate-window.vue')), {
			title: i18n.ts.tokenRequested,
			information: i18n.ts.pluginTokenRequestedDescription,
			initialName: name,
			initialPermissions: permissions
		}, {
			done: async result => {
				const { name, permissions } = result;
				const { token } = await os.api('miauth/gen-token', {
					session: null,
					name: name,
					permission: permissions,
				});
				res(token);
			}
		}, 'closed');
	});

	installPlugin({
		id: uuid(),
		meta: {
			name, version, author, description, permissions, config
		},
		token,
		ast: serialize(ast)
	});

	os.success();

	nextTick(() => {
		unisonReload();
	});
}

defineExpose({
	[symbols.PAGE_INFO]: {
		title: i18n.ts._plugin.install,
		icon: 'fas fa-download',
		bg: 'var(--bg)',
	}
});
</script>
