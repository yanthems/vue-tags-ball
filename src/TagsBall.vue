<template>
  <canvas :width="width" :height="height" id="tags_ball">
  您的浏览器暂时不支持canvas
</canvas>
</template>

<style scoped>
#tags_ball {
  border-radius: 20px;
  border: 2px solid #66ccff;
}
</style>

<script>
var Color = require('color');
let Animation = function() {
  this.running = true;
};
Animation.prototype = {
  stop: function() {},
  start: function() {}
};
export default {
  name: "tags-ball",
  install:function(vue){
    vue.component(this.name,this)
  },
  data() {
    return {
      animation: new Animation()
    };
  },
  props: {
    stop: {
      type: Boolean,
      default: false
    },
    tags: {
      type: Array,
      default: []
    },
    width: {
      type: Number,
      default: 200
    },
    height: {
      type: Number,
      default: 200
    },
    radius: {
      type: Number,
      default: 100
    },
    fontMax:{
      type:Number,
      default:60
    },
    font: {
      type: String,
      default: "24px monaco"
    },
    color:{
      tupe:String,
      default:"#fc8457"
    }
  },
  methods: {
    init_ball: function() {
      let canvas = this.$el;
      let tags = this.tags;
      let Radius = this.radius;
      let fontMax=this.fontMax
      let color=Color(this.color)
      let ctx = canvas.getContext("2d");
      ctx.font = this.font;

      let angleX = Math.PI / 100 / 2;
      let angleY = Math.PI / 100 / 2;

      const hx = canvas.width / 2;
      const hy = canvas.height / 2;

      let count = tags.length;
      if (count == 0) {
        return;
      }
      canvas.addEventListener("mousemove", function(event) {
        let x = event.layerX - hx;
        let y = event.layerY - hy;
        // console.log("x y =",x,y)
        angleY = -x * 0.0001;
        angleX = -y * 0.0001;
      });

      let points = [];
      let ve3 = function(x, y, z) {
        this.x = x;
        this.y = y;
        this.z = z;
      };
      function create() {
        let alpha = Math.sqrt(1 - 8.0 / (count + 3));
        function getm(a) {
          return Math.sqrt(
            Math.pow(a.x, 2) + Math.pow(a.y, 2) + Math.pow(a.z, 2)
          );
        }

        function getv3cos(a, b) {
          let t1 = a.x * b.x + a.y * b.y + a.z * b.z;
          let t2 = getm(a) * getm(b);
          return t1 / t2;
        }

        function general() {
          let p = new ve3(
            Math.random() - 0.5,
            Math.random() - 0.5,
            Math.random() - 0.5
          );
          let pm = getm(p);
          p.x = p.x * (Radius / pm);
          p.y = p.y * (Radius / pm);
          p.z = p.z * (Radius / pm);
          return p;
        }
        let template = [];
        template.push(new ve3(Radius, 0, 0));
        template.push(new ve3(-Radius, 0, 0));
        template.push(new ve3(0, Radius, 0));
        template.push(new ve3(0, -Radius, 0));
        template.push(new ve3(0, 0, Radius));
        template.push(new ve3(0, 0, -Radius));
        if (count <= 6) {
          for (let i = 0; i < count; i++) {
            points.push(template[i]);
          }
        } else {
          for (; points.length < count; ) {
            let v = general();
            if (points.length == 0) {
              points.push(v);
            }
            // console.log(points.length)
            let success = true;
            for (let i = 0; i < points.length; i++) {
              if (getv3cos(v, points[i]) > alpha) {
                success = false;
                break;
              }
            }
            if (success) {
              points.push(v);
            }
          }
        }
      }
      create();

      let labels = [];
      let label = function(x, y, z, str, width, max) {
        this.x = x;
        this.y = y;
        this.z = z;

        this.str = str;
        this.width = width;
        this.max = max;
      };

      label.prototype = {
        paint: function() {
          ctx.save();
          let alpha = (this.z + Radius) / (2 * Radius);
          // ctx.fillStyle = "rgba("+color+ (alpha + 0.5) + ")";
          ctx.fillStyle=color.alpha(alpha+0.5).toString()
          ctx.fillText(
            this.str,
            hx + this.x - Math.min(this.max, this.width) / 2,
            hy + this.y,
            this.max
          );
          ctx.restore();
        }
      };
      function rotateX() {
        let cos = Math.cos(angleX);
        let sin = Math.sin(angleX);
        for (let i = 0; i < labels.length; i++) {
          let y1 = labels[i].y * cos - labels[i].z * sin;
          let z1 = labels[i].z * cos + labels[i].y * sin;
          labels[i].y = y1;
          labels[i].z = z1;
        }
      }

      function rotateY() {
        let cos = Math.cos(angleY);
        let sin = Math.sin(angleY);
        for (let i = 0; i < labels.length; i++) {
          let x1 = labels[i].x * cos - labels[i].z * sin;
          let z1 = labels[i].z * cos + labels[i].x * sin;
          labels[i].x = x1;
          labels[i].z = z1;
        }
      }

      function rotateZ() {
        let cos = Math.cos(angleY);
        let sin = Math.sin(angleY);
        for (let i = 0; i < labels.length; i++) {
          let x1 = labels[i].x * cos - labels[i].y * sin;
          let y1 = labels[i].y * cos + labels[i].x * sin;
          labels[i].x = x1;
          labels[i].y = y1;
        }
      }

      let init = function() {
        for (let i = 0; i < points.length; i++) {
          let t = ctx.measureText(tags[i]);
          labels.push(
            new label(
              points[i].x,
              points[i].y,
              points[i].z,
              tags[i],
              t.width,
              fontMax
            )
          );
        }
      };
      init();

      let animation = this.animation;
      let animate = function animate() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        rotateX();
        rotateY();
        rotateZ();
        for (let i = 0; i < labels.length; i++) {
          labels[i].paint();
        }
        if (animation.running) {
          requestAnimationFrame(animate);
        }
      };

      this.animation.start = function() {
        animation.running = true;
        animate();
      };
      this.animation.stop = function() {
        animation.running = false;
      };
      this.animation.start();
    }
  },

  mounted: function() {
    this.init_ball();
  },
  computed: {
    // tags:function(){
    // }
  },
  watch: {
    tags: function(t) {
      this.init_ball();
    },
    stop: function(t) {
      if (t) {
        this.animation.stop();
      } else {
        this.animation.start();
      }
    }
  }
};
</script>
