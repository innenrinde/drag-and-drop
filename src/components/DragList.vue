<template>
  <div
      class="items-list"
      :class="{
        'items-list-vertical': direction === 'vertical',
        'items-list-horizontal': direction === 'horizontal'
      }"
      @dragover.prevent
  >
    <div
      v-for="item in localItems"
      :key="item"
      class="item"
      draggable="true"
      :ref="`item-${item.id}`"
      @dragstart="dragStart($event, item)"
      @dragend="dragEnd($event, item)"
      @dragover="dragOver($event, item)"
    >
      <slot :item="item" />
    </div>
  </div>
</template>

<script>
export default {
  name: "DragList",
  setup(props) {

    const animation = props.direction === "vertical" ?
        new AnimationVertical() :
        new AnimationHorizontal();

    return {
      animation
    }
  },
  props: {
    items: {
      type: Array,
      default: () => []
    },
    direction: {
      type: String,
      default: () => "vertical"
    },
  },
  data() {
    return {
      dragElementId: null,
      dropElementId: null,
      animationInProgress: false,
      localItems: structuredClone(this.items),
    }
  },
  methods: {
    /**
     * Start animation listener - by css class
     */
    animationStartListener() {
      this.animationInProgress = true;
    },
    /**
     * End animation listener - ended through css class
     */
    animationEndListener() {

      let dragTarget = this.getItem(this.dragElementId);
      let dropTarget = this.getItem(this.dropElementId);

      // trying to detect items to be switched
      let dragIndex = this.localItems.findIndex(row => String(row.id) === String(this.dragElementId));
      let dropIndex = this.localItems.findIndex(row => String(row.id) === String(this.dropElementId));

      // switch items
      let temp = this.localItems[dragIndex];
      this.localItems[dragIndex] = this.localItems[dropIndex];
      this.localItems[dropIndex] = temp;

      // clear animations css
      this.animation.clear(dragTarget, dropTarget);

      // clear listener... prepare to a new move
      dragTarget.removeEventListener("animationstart", this.animationStartListener);
      dropTarget.removeEventListener("animationend", this.animationEndListener);

      this.animationInProgress = false;
    },
    /**
     * Start dragging an item
     * @param {Event} event
     * @param {Object} item
     */
    dragStart(event, item) {
      this.dragElementId = item.id;
    },
    /**
     * Move item over another item
     * @param {Event} event
     * @param {Object} item
     */
    dragOver(event, item) {

      if (
          this.dragElementId &&
          item.id &&
          !this.animationInProgress &&
          String(this.dragElementId) !== String(item.id)
      ) {
        this.dropElementId = item.id;

        let dragTarget = this.getItem(this.dragElementId);
        let dropTarget = this.getItem(this.dropElementId);

        dragTarget.addEventListener("animationstart", this.animationStartListener);
        dragTarget.addEventListener("animationend", this.animationEndListener);

        this.animation.apply(dragTarget, dropTarget);
      }
    },
    /**
     * Stop dragging item
     */
    dragEnd() {
      this.animationInProgress = false;
      this.$emit("order", this.localItems);
    },
    /**
     * Get a list item by id
     * @param {String} id
     * @returns {HTMLElement}
     */
    getItem(id) {
      return (this.$refs[`item-${id}`] ?? []).at(0);
    }
  }
}

class AnimationVertical {

  /**
   * @param {HTMLElement} dragTarget
   * @param {HTMLElement} dropTarget
   */
  apply(dragTarget, dropTarget) {
    if (dragTarget.offsetTop < dropTarget.offsetTop) {
      dragTarget.classList.add("move-down");
      dropTarget.classList.add("move-up");
    } else {
      dragTarget.classList.add("move-up");
      dropTarget.classList.add("move-down");
    }
  }

  /**
   * @param {HTMLElement} dragTarget
   * @param {HTMLElement} dropTarget
   */
  clear(dragTarget, dropTarget) {
    dragTarget.classList.remove("move-down", "move-up");
    dropTarget.classList.remove("move-down", "move-up");
  }

}

class AnimationHorizontal {

  /**
   * @param {HTMLElement} dragTarget
   * @param {HTMLElement} dropTarget
   */
  apply(dragTarget, dropTarget) {
    if (dragTarget.offsetLeft < dropTarget.offsetLeft) {
      dragTarget.classList.add("move-right");
      dropTarget.classList.add("move-left");
    } else {
      dragTarget.classList.add("move-left");
      dropTarget.classList.add("move-right");
    }
  }

  /**
   * @param {HTMLElement} dragTarget
   * @param {HTMLElement} dropTarget
   */
  clear(dragTarget, dropTarget) {
    dragTarget.classList.remove("move-right", "move-left");
    dropTarget.classList.remove("move-right", "move-left");
  }

}

</script>

<style scoped>
.items-list {
  width: 100%;
  height: 100%;
  display: flex;
}

.items-list-vertical {
  flex-direction: column;
}

.items-list-horizontal {
  flex-direction: row;
}

.item {
  cursor: move;
  display: flex;
  flex-direction: column;
}

.items-list-horizontal > .item {
  flex-grow: 1;
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
