<template>
	<span v-if="status" style="color:green;">
		Good
	</span>
	<span v-else style="color:red;">
		Bad
	</span>:
	{{ local < 0 || remote < 0 ? '???' : (local - remote).toLocaleString() }}
</template>

<script lang="ts">

import { defineComponent, computed } from 'vue';

export interface Props {
	local: number;
	remote: number;
	tolerance: number;
	status: boolean;
};

export default defineComponent({
	props: {
		local: Number,
		remote: Number,
		tolerance: Number,
	},
	setup(props: Props) {
		const status = computed(() => {
			if(props.local < 0) return false;
			if(props.remote - props.local > props.tolerance) return false;
			return true;
		});
		return {
			status,
		};
	},
});

</script>

