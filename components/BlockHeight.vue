<template>
	{{ value < 0 ? 'unknown' : value.toLocaleString() }}
</template>

<script lang="ts">

import { defineComponent } from 'vue';

export default defineComponent({
	props: {
		value: Number,
	},
});

</script>

