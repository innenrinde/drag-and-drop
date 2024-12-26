<template>
	<div class="container">
		<drag-list
			:items="items"
			@order="getOrder"
		>
			<template #default="{ item }">
				{{ item.label }}
			</template>
		</drag-list>
	</div>
</template>

<script>
import DragList from "./components/DragList.vue";

export default {
  name: 'App',
  components: {
		DragList
  },
	data() {
		return {
			items: [{
				label: "item 1",
				id: 1,
			}, {
				label: "item 2",
				id: 2,
			}, {
				label: "item 3",
				id: 3,
			}, {
				label: "item 4",
				id: 4,
			}, {
				label: "item 5",
				id: 5,
			}, {
				label: "item 6",
				id: 6,
			}, {
				label: "item 7",
				id: 7,
			}, {
				label: "item 8",
				id: 8,
			}, {
				label: "item 9",
				id: 9,
			}, {
				label: "item 10",
				id: 10,
			}]
		}
	},
	methods: {
		getOrder(list) {
			console.log(list);
		}
	}
}
</script>

<style>
html, body {
	margin: 0;
	background-color: #efefef;
	height: 100%;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-size: 12px;
  color: #2c3e50;
	height: 100%;
  display: flex;
  flex-direction: column;
}

.container {
  padding: 20px;
	display: flex;
	flex-direction: column;
	align-items: center;
}
</style>


