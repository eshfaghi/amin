<template>
  <div :style="getStyle()">
    <div class="rowc" :id="page.name+properties.name">
      <drop class="drop visual" @drop="onVisualDrop" :accepts-data="(n) => isVisual(n)"></drop>
      <div :class="'content '+getClass()" :style="'padding:'+(15 * scale)+'px;'+getStyleMain()">
        <child-simulator :style="getStyleChild(child)" class="flex-child" v-for="(child,k) in properties.children" :type="child.type"
                         :properties="child" :scale="scale"
                         :page="page" @click.native.capture="setProperty(child)" :key="k"></child-simulator>

      </div>

      <div class="control" title="Children control" @click="modalOpen">
        <i class="fa fa-list"></i>
      </div>
    </div>

    <!--    <vue-final-modal v-model="showChildrenModal" @before-open="modalOpen" @before-close="modalClose" name="code-modal">-->
    <!--      <div class="terminal" >-->
    <!--        <i class="fa fa-circle red-text" @click="showChildrenModal = false"></i>-->
    <!--        <i class="fa fa-circle yellow-text text-darken-2"></i>-->
    <!--        <i class="fa fa-circle green-text text-lighten-2"></i>-->
    <!--        <div class="content" id="container">-->
    <!--           hello-->
    <!--        </div>-->
    <!--      </div>-->
    <!--    </vue-final-modal>-->
  </div>
</template>

<script>
import {fnc} from "@/assets/js/functions";
import {Drop} from "vue-easy-dnd";
// import Sortable from '@/assets/js/Sortable.min';

export default {
  name: "Row",
  data: function () {
    return {
      showChildrenModal: false,
    }
  },
  props: ['properties', 'scale', 'page'],
  components: {
    Drop,
    'child-simulator': () => import('../elements/Simulator')
  },
  methods: {
    getClass: function () {
      if (this.properties.axis === 'MainAxisAlignment.spaceAround' || this.properties.axis === 'MainAxisAlignment.spaceBetween') {
        if (this.properties.scrollable) {
          return 'axis-flex.scroll';
        } else {
          return 'axis-flex';
        }
      }

      if (this.properties.axis === 'MainAxisAlignment.start') {
        return 'axis-start';
      }
      if (this.properties.axis === 'MainAxisAlignment.end') {
        if (window.appData.project.isRTL) {
          return 'axis-end rtl';
        } else {
          return 'axis-end';
        }
      }

    },
    getStyleMain: function () {
      let style = '';
      if (this.properties.axis === 'MainAxisAlignment.spaceAround' || this.properties.axis === 'MainAxisAlignment.spaceBetween') {
        style += 'display:flex;flex-flow: row wrap; flex-basis: 100%;';
      }
      return style;
    },
    modalOpen: function () {
      let n = this;
      do {
        n = n.$parent;
      } while (n.showRowModal === undefined);
      n.showRowModal = true;
      n.rowData = this.properties.children;
      // console.log(n.rowData);
    },
    setProperty: function (prop) {
      var propcont = prop;
      var n = this;
      let counter = 0;
      do {
        n = n.$parent;
        counter++;
      } while (n.currentProperties === undefined);
      setTimeout(function () {
        n.currentProperties = propcont;
      },counter * 3);
    },
    capitalizeFirstLetter: function (string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    },
    onVisualDrop(event) {
      try {

        // find component
        let component = window.components[event.data];
        // load default value
        let properties = eval(component.data);
        // if can't loaded
        if (properties === undefined) {
          window.alertify.warning('Invalid component', 15);
        } else {
          // when loaded add to page with validate
          this.visualValidator(properties, this.properties.children);
          // ***!*** need sort and rename here
        }
      } catch (e) {
        console.log(e.message);
        window.alertify.warning('unknown error on load component');
      }
    },
    visualValidator: function (component, visuals) {
      var self = this;
      if (component.type === 'appbar' || component.type === 'nav' ) {
        // check non appbar or row
        window.alertify.error("You can't drop appbar or nav into row");
        return false;
      }
      // add component
      var newComponent = fnc.clone(component);
      // // choose name
      // // check not duplicate name add number to name
      let names = [];
      newComponent.name = this.properties.name + this.capitalizeFirstLetter(component.type);
      for (const comp of visuals) {
        names.push(comp.name);
      }
      let i = 0;
      let nextName = false;
      do {
        nextName = false;
        i++;
        if (names.indexOf(this.properties.name + this.capitalizeFirstLetter(component.type) + i.toString()) > -1) {
          nextName = true;
        }
      } while (nextName);
      newComponent.name = this.properties.name + this.capitalizeFirstLetter(component.type) + i;
      visuals.push(newComponent);
// update page preview
      if (!window.ide.settings.performanceMode) {
        setTimeout(function () {
          fnc.takeScreenShot("#preview", function (e) {
            self.page.image = e;
          });
        }, 1000);
      }
      return true;
    },
    isVisual: function (n) { // check is visual component or not
      return window.components[n].visual;
    },
    getStyle: function () {
      let style = '';
      // style += 'background-color:' + this.color2web(this.properties.color, false) + ';';

      if (this.properties.width != 'null') {
        style += 'width:' + fnc.getSize(this.properties.width, this.scale, 3) + ';'
      }
      if (this.properties.height != 'null') {
        style += 'height:' + fnc.getSize(this.properties.height , this.scale , 3, true) + ';'
      }
      if (this.properties.scrollable == true) {
        style += 'overflow-y:hidden ;overflow-x: scroll;';
      }

      if (this.properties.bgColor != 'null') {
        style += 'background:' + this.color2web(this.properties.bgColor) + ';';
      }
      // console.log('bgRow',this.properties.bgColor,style);
      style += 'border-style: solid;';
      style += 'border-width:' + fnc.calcPadding(this.properties.border) + ';';
      if (this.properties.borderColor != 'null') {
        style += 'border-color:' + this.color2web(this.properties.borderColor) + ';';
      }
      if (this.properties.padding != 'null' && this.properties.padding != '') {
        style += 'padding:' + this.calcPadding(this.properties.padding) + ';'
      }
      if (this.properties.margin != 'null' && this.properties.margin != '') {
        style += 'margin:' + this.calcPadding(this.properties.margin) + ';'
      }
      if (this.properties.borderRadius != 'null' && this.properties.borderRadius != '') {
        style += 'border-radius:' + this.calcPadding(this.properties.borderRadius) + ';'
      }


      return style;
    }
    ,
    getStyleChild: function (child) {
      if (child.type === 'column' || child.type === 'row' || child.type === 'container'){
        return 'min-width:75px;' // width:'+fnc.getSize( child.width, this.scale, 2.5) +';';
      }
    },
    color2web: function (clr, b = false) {
      return fnc.color2web(clr, b);
    }
    ,
    calcPadding: function (pad) {
      return fnc.calcPadding(pad);
    }
  }
}
</script>

<style scoped>
.rowc {
  background: transparent;
  min-height: 25px;
  border: 1px dotted silver;
  position: relative;
  white-space: nowrap;
  display: flex;
  flex-wrap: nowrap;
  flex-direction: row;
}

.control {
  position: absolute;
  right: 0;
  bottom: 0;
  padding: 5px;
  color: white;
  width: 35px;
  text-align: center;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 3px;
  transition: .3s;
}

.control:hover {
  background: rgba(0, 0, 0, 0.7);
}

.modal {
  color: white;
}


.terminal {
  width: 80%;
  margin: 50px auto 0 auto;
  max-width: 1000px;
  border-radius: 7px;
  background: #272c34;
  padding: 15px;
  line-height: 1.2em;

}


.clear:hover {
  background: rgba(0, 0, 0, .5);
}

.terminal .content {
  padding: 0 20px;
  color: #eee;
  font-family: VazirCodeX;
  height: 600px;
  overflow-x: hidden;
  overflow-y: scroll;
  scroll-behavior: smooth;
  -webkit-user-select: text; /* Chrome 49+ */
  -moz-user-select: text; /* Firefox 43+ */
  -ms-user-select: text; /* No support yet */
  user-select: text; /* Likely future */
  /*display: none;*/
}

.terminal .content ul li {
  white-space: pre;
}


.fa-circle {
  margin-right: 10px;
}

.row {
  border: 0;
}

.row .col {
  height: 580px;
}

.row .s3 {
  /*border-left: 1px solid silver;*/
  text-align: center;
}

.rowc .content{
  align-items: center;
}
</style>