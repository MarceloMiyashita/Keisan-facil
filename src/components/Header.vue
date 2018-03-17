<template>
	<nav class="navbar navbar-light bg-light">
		<button class="btn btn-warning my-2 my-sm-0 btn-lg" disabled>Sua pontuação <span class="badge badge-light">{{ pontos }}</span></button>
		<button class="btn btn-primary my-2 my-sm-0 btn-lg" disabled>Vida&nbsp<span class="badge badge-light">{{ health }}</span></button>
	</nav>
</template>

<script>
export default {
	props:["pontos","health"],
	data () {
		return {
		}
	}
}
</script>
