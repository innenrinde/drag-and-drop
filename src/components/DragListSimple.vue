<template>
	<div
		class="items-list"
		@dragover.prevent
	>
		<div
			v-for="item in localItems"
			:key="item"
			class="item"
			draggable="true"
			:data-id="item.id"
			@dragstart="dragStart"
			@dragend="dragEnd"
			@dragover="dragOver"
		>
			<slot :item="item" />
		</div>
	</div>
</template>

<script>
export default {
	name: "DragListSimple",
	props: {
		items: Array,
		default: () => []
	},
	data() {
		return {
			dragElementId: null,
			localItems: structuredClone(this.items),
		}
	},
	methods: {
		/**
		 * Start dragging element
		 * @param event
		 */
		dragStart(event) {
			this.dragElementId = event.target.dataset.id;
			event.target.classList.add("over");
		},
		/**
		 * Stop dragging an item
		 * @param event
		 */
		dragEnd(event) {
			event.target.classList.remove("over");
			this.$emit("order", this.localItems);
		},
		/**
		 * Move an item over another item
		 * @param event
		 */
		dragOver(event) {
			if (!this.dragElementId) {
				return;
			}

			let dragId = this.dragElementId;
			let dropId = event.target.dataset.id;

			if (dragId !== dropId) {
				let dragIndex = this.localItems.findIndex(item => item.id.toString() === dragId.toString());
				let dropIndex = this.localItems.findIndex(item => item.id.toString() === dropId.toString());

				let temp = this.localItems[dragIndex];
				this.localItems[dragIndex] = this.localItems[dropIndex];
				this.localItems[dropIndex] = temp;
			}
		},
	}
}
</script>

<style scoped>
.items-list {
	background-color: #efefef;
	display: flex;
	flex-direction: column;
	width: 300px;
	align-items: stretch;
	align-content: stretch;
	gap: 5px;
}

.item {
	background-color: white;
	border: solid 1px black;
	border-radius: 5px;
	padding: 10px;
	cursor: move;
	flex-grow: 1;
	height: 20px;
	width: 300px;
}

.over {
	opacity: 0.5;
	border: dashed 1px black;
	background-color: cornsilk;
}

</style>
