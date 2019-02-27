<template>
   <div>
      <div class="div">
         <multipane class="horizontal-panes" layout="horizontal">
            <div class="pane" :style="{ minHeight: '100px', height: '200px', maxHeight: '300px' }">
               <div>
                  <h6 class="title is-6">Pane 1</h6>
                  <div style="width:100%;height:90%;color:red">内容一</div>
               </div>
            </div>
            <div
               class="multipane-resizer"
               :style="{width:'100%'}"
               @mousedown="onMouseDown($event)"
            ></div>
            <div class="pane" :style="{ height: '40%', maxHeight: '60%' }">
               <div>
                  <h6 class="title is-6">Pane 2</h6>
                  <div style="width:100%;height:90%;color:green">内容二</div>
               </div>
            </div>
            <div
               class="multipane-resizer"
               :style="{width:'100%'}"
               @mousedown="onMouseDown($event)"
            ></div>
            <div class="pane" :style="{ flexGrow: 1}">
               <div>
                  <h6 class="title is-6">Pane 3</h6>
                  <div style="width:100%;height:90%;color:pink">内容三</div>
               </div>
            </div>
         </multipane>
      </div>
   </div>
</template>

<script>
import { Multipane, MultipaneResizer } from "vue-multipane";
const LAYOUT_HORIZONTAL = "horizontal";
const LAYOUT_VERTICAL = "vertical";
export default {
   data() {
      return {
         isResizing: false
      };
   },
   components: {
      Multipane,
      MultipaneResizer
   },
   methods: {
      //pane demo元素 /
      onMouseDown({
         target: resizer,
         pageX: initialPageX,
         pageY: initialPageY
      }) {
         if (resizer.className && resizer.className.match("multipane-resizer")) {
            let self = this;
            let { $el: container, layout } = self;
            let pane = resizer.previousElementSibling; //事件加到线条上使用
            console.log(pane, "元素");
            let {
               offsetWidth: initialPaneWidth,
               offsetHeight: initialPaneHeight
            } = pane;

            let usePercentage = !!(pane.style.width + "").match("%"); //false
            console.log(usePercentage);
            const { addEventListener, removeEventListener } = window;
            const resize = (initialSize, offset = 0) => {
               if (layout == LAYOUT_VERTICAL) {
                  let containerWidth = container.clientWidth;
                  let paneWidth = initialSize + offset;

                  return (pane.style.width = usePercentage
                     ? (paneWidth / containerWidth) * 100 + "%"
                     : paneWidth + "px");
               }

               if (layout == LAYOUT_HORIZONTAL) {
                  let containerHeight = container.clientHeight;
                  let paneHeight = initialSize + offset;

                  return (pane.style.height = usePercentage
                     ? (paneHeight / containerHeight) * 100 + "%"
                     : paneHeight + "px");
               }
            };

            // This adds is-resizing class to container
            //这将is-resizing类添加到容器中
            self.isResizing = true;

            // Resize once to get current computed size
            //调整一次大小以获得当前计算的大小
            let size = resize();
            // Trigger paneResizeStart event
            //触发paneResizeStart事件
            self.$emit("paneResizeStart", pane, resizer, size);

            const onMouseMove = function({ pageX, pageY }) {
               size =
                  layout == LAYOUT_VERTICAL
                     ? resize(initialPaneWidth, pageX - initialPageX)
                     : resize(initialPaneHeight, pageY - initialPageY);

               self.$emit("paneResize", pane, resizer, size);
               console.log("onMouseMove");
            };

            const onMouseUp = function() {
               // Run resize one more time to set computed width/height.
               //再运行一次调整大小以设置计算的宽度/高度
               size =
                  layout == LAYOUT_VERTICAL
                     ? resize(pane.clientWidth)
                     : resize(pane.clientHeight);
               // This removes is-resizing class to container
               //这将把is-resizing类删除到container中
               self.isResizing = false;
               console.log(
                  "onMouseUp--高",
                  pane.clientHeight + "--宽",
                  pane.clientWidth
               );
               removeEventListener("mousemove", onMouseMove);
               removeEventListener("mouseup", onMouseUp);

               self.$emit("paneResizeStop", pane, resizer, size);
            };

            addEventListener("mousemove", onMouseMove);
            addEventListener("mouseup", onMouseUp);
         }
      }
   }
};
</script>

<style>
.horizontal-panes {
   width: 100%;
   height: 600px;
   border: 1px solid #ccc;
}
.horizontal-panes > .pane {
   text-align: left;
   padding: 15px;
   overflow: hidden;
   background: #eee;
}
.horizontal-panes > .pane ~ .pane {
   border-top: 1px solid #ccc;
}

</style>
