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
	name: "DragList",
	props: {
		items: Array,
		default: () => []
	},
	data() {
		return {
			dragElement: null,
			dropElement: null,
			animationInProgress: false,
			dragEventInProgress: false,
			localItems: structuredClone(this.items),
		}
	},
	methods: {
		/**
		 * Start animation listener -  by css class
		 */
		animationStartListener() {
			this.animationInProgress = true;
		},
		/**
		 * End animation listener - ended through css class
		 */
		animationEndListener() {

			// clear animation css
			this.dragElement.classList.remove("move-down");
			this.dropElement.classList.remove("move-up");
			this.dragElement.classList.remove("move-up");
			this.dropElement.classList.remove("move-down");

			// trying to detect items to be switched
			let dragIndex = this.localItems.findIndex(row => row.id.toString() === this.dragElement.dataset.id.toString());
			let dropIndex = this.localItems.findIndex(row => row.id.toString() === this.dropElement.dataset.id.toString());

			// switch items
			let temp = this.localItems[dragIndex];
			this.localItems[dragIndex] = this.localItems[dropIndex];
			this.localItems[dropIndex] = temp;

			// clear listener... prepare to a new move
			if (!this.dragEventInProgress) {
				this.dragElement.removeEventListener("animationstart", this.animationStartListener);
				this.dragElement.removeEventListener("animationend", this.animationEndListener);
			}

			this.animationInProgress = false;
		},
		/**
		 * Start dragging an item
		 * @param event
		 */
		dragStart(event) {
			this.dragElement = event.target;

			this.dragEventInProgress = true;

			this.dragElement.addEventListener("animationstart", this.animationStartListener);
			this.dragElement.addEventListener("animationend", this.animationEndListener);

			this.dragElement.classList.add("over");
		},
		/**
		 * Move item over another item
		 * @param event
		 */
		dragOver(event) {

			if (!this.animationInProgress && this.dragElement.dataset.id !== event.target.dataset.id) {
				this.dropElement = event.target;

				if (this.dragElement.offsetTop < this.dropElement.offsetTop) {
					this.dragElement.classList.add("move-down");
					this.dropElement.classList.add("move-up");
				} else {
					this.dragElement.classList.add("move-up");
					this.dropElement.classList.add("move-down");
				}

			}
		},
		/**
		 * Stop dragging item
		 * @param event
		 */
		dragEnd(event) {
			this.dragEventInProgress = false;
			event.target.classList.remove("over");

			this.$emit("order", this.localItems);
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
	padding: 10px;
	flex-grow: 1;
	cursor: move;
	width: 300px;
	border: solid 1px black;
	border-radius: 5px;
	background-color: white;
}

.over {
	opacity: 0.5;
	border: dashed 1px black;
	background-color: cornsilk;
}

.move-down {
	animation-duration: 0.3s;
	animation-name: slide-down;
}


@keyframes slide-down {
	from {
		transform: translateY(0);
	}

	to {
		transform: translateY(100%);
	}
}

.move-up {
	animation-duration: 0.3s;
	animation-name: slide-up;
}

@keyframes slide-up {
	from {
		transform: translateY(0);
	}

	to {
		transform: translateY(-100%);
	}
}

.move-left {
	animation-duration: 0.3s;
	animation-name: slide-left;
}

@keyframes slide-left {
	from {
		transform: translateX(0);
	}

	to {
		transform: translateX(-100%);
	}
}

.move-right {
	animation-duration: 0.3s;
	animation-name: slide-right;
}

@keyframes slide-right {
	from {
		transform: translateX(0);
	}

	to {
		transform: translateX(100%);
	}
}

</style>
