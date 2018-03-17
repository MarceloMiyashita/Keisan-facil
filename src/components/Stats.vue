<template>
	<div class="alert alert-info">
		<h4>Game Over</h4>
		<button type="button" class="btn btn-primary btn-lg text-center" @click="initGame">Iniciar</button>
		<ul class="list-group">
			<li class="list-group-item d-flex justify-content-between align-items-center">
				Pontuação
				<span class="badge badge-primary badge-pill">{{ score }}</span>
			</li>
			<li class="list-group-item d-flex justify-content-between align-items-center">
				Total do tempo
				<span class="badge badge-primary badge-pill">{{ totalTime }}s</span>
			</li>              
		</ul>

	</div>
</template>

<script>
export default {
	props:["score", "totalTime", "initGame"],
	data () {
		return {
		}
	}
}
</script>

<style type="text/css">
</style>