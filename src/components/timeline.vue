<template>
  <div class="component timeline" ref="component" 
    style="position: absolute;left:0;top:0;width:100%;height: 100%;overflow: hidden;background: rgb(223,223,223);cursor: default;"
  >

    <div class="global_wrap"
      v-pan="{fn:component_pan, args:[]}" 
      @mousewheel="onmousewheel($event)" 
      @mousemove="component_mousemove($event)" 
      @touchstart="component_mousemove($event)" 
      @touchmove="component_mousemove($event);is_show_pophover=false;"  
      style="position: absolute;left:0;top:0;width:100%;height:100%;"
    >
      <div class="global clearfix" :style="get_global_style()">
        <div class="marginfix" style="height: 1px;margin-bottom: -1px;"></div>
        <div class="area" v-for="(area, i) in act_areas" :style="get_area_style(area, i)" style="background: white;">
          <div class="row" v-for="(row, i) in area.rows" :style="get_row_style(row, i, area)">
            <div class="period" :class="{act:period_act===period}" v-for="(period, i) in row.periods" 
              @mouseenter="period_mouseenter(period, i)" 
              @touchstart="period_mouseenter(period, i)" 
              @mouseleave="period_mouseleave(period, i)" 
              @mousedown="period_mousedown($event)"
              v-hammer:press="period_press"
              :style="get_period_style(period, i, area)"
            >{{period.name}}</div>
          </div>
        </div>
      </div>
    </div>

    <div class="ruler"></div>
    <div class="v_bar" :style="{left: poin.x+'px'}"> </div>
    <div class="poin_time":style="{left:poin.x+'px'}">
      <span>{{poin_time}}</span>
      <span style="margin-left: 5px;color:gray;">距今: {{now_year-poin_time}}</span>
    </div>

    <div class="menu clearfix" style="width:100%;position: absolute;left: 0;bottom: 0;display: flex;align-items: flex-end;flex-wrap: wrap-reverse;justify-content: flex-end; background: rgb(190,190,190);">
      <!-- <div class="area" :class="{act:act_areas.includes(area)}" v-down="{fn:menu_area_click, args:[area, i]}" v-for="(area, i) in global.areas" style="">{{area.name}}</div> -->
      <!-- <a class="item" href="http://gonnavis.com/timeline_old2/" target="_blank">返回旧版</a> -->
      <div class="item area" :class="{act:act_areas.includes(area)}" v-hammer:tap="()=>menu_area_click(area, i)" v-for="(area, i) in global.areas" style="">{{area.name}}</div>
    </div>

    <div class="pophover" v-if="period_act&&is_show_pophover" :style="get_pophover_style()">
      <div>{{period_act.name}}  </div>
      <div>公元: {{period_act.from}} ~ {{period_act.to}}</div>
      <div style="color:rgb(160,160,160);">距今: {{now_year-period_act.from}} ~ {{now_year-period_act.to}}</div>
      <div>时长: {{period_act.to-period_act.from}}</div>

    </div>

    <div class="popmenu" v-if="period_act&&is_show_popmenu" @click="is_show_popmenu=false" :style="popmenu_style">
      <a :href="'https://baike.baidu.com/item/'+period_act.name"
        target="_blank" style="display: block;"
      >百度百科</a>
      <a :href="'https://www.baidu.com/s?wd='+period_act.name"
        target="_blank" style="display: block;"
      >百度搜索</a>
    </div>

  </div>
</template>

<script>
// import data from './data.js'
import global from './preprocess_data.js'
export default {
  name: 'timeline',
  data () {
    return {
      r: null,
      popmenu_style: null,
      is_show_popmenu: false,
      is_show_pophover: false,
      global: global,
      period_height: 30,
      zoom: .1,
      global_left: 0,
      global_top: 50,
      act_areas: [],
      poin: {x:0, y:0}, // pointer
      poin_time: 0,
      now_year: new Date().getFullYear(),
      zoom_fix: 2,
      zoom_min: .05,
      pan_speed: 2, // the larger the faster
      period_act: null,

    }
  },
  created(){
    let s=window.s=this
    console.log(global)
    s.act_areas.push(global.areas[0])
  },
  mounted(){
    let s=this
    s.r=s.$refs

    // let vh=new VSHammer(s.r.component);
    // vh.on('pan', function(ve){
    //   // console.log(ve)
    //   s.global_left+=ve.deltaX;
    //   s.global_top+=ve.deltaY;
    // })

    let hammer=new Hammer(s.r.component);
    hammer.get('pinch').set({ enable: true });
    hammer.on('pinchin', function(e){

      s.zoom_out(s.x_to_time(e.center.x));
    })
    hammer.on('pinchout', function(e){
      s.zoom_in(s.x_to_time(e.center.x));
    })
  },
  directives:{
    pan: {
      bind(el, binding){
        let vh=new VSHammer(el);
        vh.on('pan', function(ve){
          binding.value.fn(ve, ...binding.value.args )
        })
      }
    },
    down:{
      bind(el, binding){
        let vh=new VSHammer(el);
        vh.on('down', function(ve){
          binding.value.fn(ve, ...binding.value.args )
        })
      }
    }
  },
  methods:{
    period_contextmenu(ne){
      let s=this
      console.log('contextmenu', ne)
    },
    period_mousedown(ne){
      let s=this;
      if(ne.button===2){
        s.popmenu_style=s.get_popmenu_style();
        s.is_show_popmenu=true;
      }
    },
    period_press(){
      let s=this;
      s.popmenu_style=s.get_popmenu_style();
      s.is_show_popmenu=true;
    },
    get_popmenu_style(){
      let s=this;
      let style={
        left: s.poin.x-100+'px',
        top: s.poin.y-50+'px',
      }
      return style;
    },
    period_mouseenter(period, i){
      let s=this
      s.period_act=period;
      s.is_show_pophover=true;
    },
    period_mouseleave(period, i){
      let s=this
      s.is_show_pophover=false
    },
    get_pophover_style(){
      let s=this;
      let left=s.poin.x-320
      if(left<0) left=0
      let style={
        left: left+'px',
        // left: 0+'px',
        top: s.poin.y+30+'px',
      }
      return style;
    },
    zoom_in(current_time){
      let s=this;
      let prev_zoom = s.zoom;

      let zoom_step = s.zoom / 10;
      s.zoom = +(s.zoom + zoom_step).toFixed(s.zoom_fix);

      s.global_left += -((current_time - s.global.min) * s.zoom - (current_time - s.global.min) * prev_zoom);
    },
    zoom_out(current_time){
      let s=this;
      let prev_zoom = s.zoom;

      let zoom;
      let zoom_step = s.zoom / 10;
      zoom = +(s.zoom - zoom_step).toFixed(s.zoom_fix);
      if (zoom >= s.zoom_min) {
        s.zoom = zoom;
        s.global_left += ((current_time - s.global.min) * prev_zoom - (current_time - s.global.min) * s.zoom);
      }
    },
    component_mousemove(e){
      let s=this;
      // console.log(e);
      if(e.touches){
        s.poin.x=e.touches[0].clientX;
        s.poin.y=e.touches[0].clientY;
      }
      else{
        s.poin.x=e.clientX;
        s.poin.y=e.clientY;
      }
      s.poin_time = Math.floor((s.poin.x - s.global_left) / s.zoom + s.global.min);
      s.is_show_popmenu=false;
    },
    x_to_time(x){
      let s=this
      return Math.floor((x - s.global_left) / s.zoom + s.global.min);
    },
    component_pan(ve){
      let s=this
      s.global_left+=ve.deltaX*s.pan_speed;
      s.global_top+=ve.deltaY*s.pan_speed;
    },
    menu_area_click(area, i){
      let s=this;
      if(s.act_areas.includes(area)){
        s.act_areas.splice(s.act_areas.indexOf(area), 1);
      }else{
        s.act_areas.push(area);
      }
    },
    onmousewheel(e){
      let s=this;
      // console.log(e);
      // s.zoom+=-e.deltaY/10000;
      // if(s.zoom<.01) s.zoom=.01;
      if(e.deltaY<0){
        s.zoom_in(s.poin_time);
      }
      else{
        s.zoom_out(s.poin_time);
      }
    },
    get_global_style(){
      let s=this;
      let style={
        position: 'relative',
        width: s.global.span*s.zoom + 200 + 'px',
        left: s.global_left + 'px',
        top: s.global_top+'px',
      }
      return style;
    },
    get_area_style(area, i){
      let s=this
      let height=area.rows.length*s.period_height
      let style={
        position: 'relative',
        width: '100%',
        height: height+'px',
        border: 'solid 1px rgb(200,200,200)',
        margin: '10px 0',
      };
      return style;
    },
    get_row_style(row, i, area){
      let s=this
      let height=s.period_height;
      let style={
        position: 'relative',
        height: height+'px',
      };
      return style;
    },
    get_period_style(period, i, area){
      let s=this
      let height=s.period_height;
      let style={
        left: (period.from-s.global.min)*s.zoom+'px',
        width: period.span*s.zoom+'px',
        height: height+'px',
        'line-height': height-2+'px',
        'background-color': period.color,
      };
      return style;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .ruler{    height: 24px; background: #000; position: absolute; top: 0; left: 0; width: 100%;}
  .v_bar{    position: absolute; top: 0;    height: 100%; width: 1px; background: #000; pointer-events: none;}
  .poin_time{    position: absolute; top: 0;color: #fff; line-height: 24px;white-space: nowrap;}
  .pophover{border: 1px solid gray; background: rgba(255,255,255,.9); border-radius: 4px; text-align: left; padding: 10px; position: absolute; width: 300px; pointer-events: none;}
  .popmenu{border: 1px solid gray; background: white; border-radius: 4px;  padding: 10px; position: absolute;  background: white;width:200px;height:100px;line-height: 50px;}
  .menu .item{background: rgb(160,160,160);border: solid 1px;padding:6px 6px;cursor: pointer;}
  .menu .item.act{background: white;}
  .global .period{position: absolute;top: 0;box-sizing: border-box;border: solid 1px gray;color:black;text-shadow:rgb(255, 255, 255) 1px 1px 0px;word-break: keep-all;}
  .global .period.act{border-color:red;border-width: 2px;}
</style>
