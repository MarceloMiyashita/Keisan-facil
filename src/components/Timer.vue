<template>
	<div>
		<div>Tempo restante : {{ temporestante }}</div>
		<div class="progress">
			<div
			class="progress-bar progress-bar-striped progress-bar-animated"
			:class="getProgressClass()"
			:style="{ width: ( 100 - ( 100 * temporestante/settings.timeOut ) ) + '%'}"
			:aria-valuenow="temporestante"
			aria-valuemin="0"
			:aria-valuemax="settings.timeOut"></div>
		</div>

		<div class="messages alert" :class="messageClass()" v-if="notification != ''">{{ notification }} <strong v-if="getBonus()>0 && result == 'Bom'">Bonus + {{ getBonus() }}</strong></div>
	</div>
</template>

<script>
export default {
	props:["temporestante", "settings", "getProgressClass", "messageClass", "getBonus" ,"notification", "result"],
	data () {
		return {
		}
	}
}
</script>

<style type="text/css">
	.progress{
		height: 3px;
	}
</style>