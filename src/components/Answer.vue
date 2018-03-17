<template>
	<div>
		<button class="btn btn-primary btn-lg" v-for="sugg in sugestoes" @click="answered(sugg)" :disabled="questaoRespondida">{{ sugg }}</button>
	</div>
</template>

<script>
import { eventBus } from "../main.js"

export default {
	props:["sugestoes","questaoRespondida"],
	data () {
		return {
		}
	},methods:{
		answered: function(answer){
			eventBus.$emit("userAnswered", answer)
		}
	}
}
</script>

<style type="text/css" scoped>
	button{
		margin: 20px;
	}
</style>