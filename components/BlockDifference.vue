<template>
	{{ prev < 0 || current < 0 ? '0' : ((current-prev > 0 ? '+' : '') + (current-prev).toLocaleString()) }}
</template>

<script lang="ts">

import { defineComponent } from 'vue';

export default defineComponent({
	props: {
		prev: Number,
		current: Number,
	},
});

</script>

