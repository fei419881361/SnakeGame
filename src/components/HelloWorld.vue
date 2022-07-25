<template>
  <div class="hello">
    <h1>
      {{ msg }}
      {{ eatCount}}
      <img id="apple" src="../assets/apple.jpg" width="30" height="30" />
    </h1>

    <canvas id="canvas" :width="mapWide" :height="mapHeight"></canvas>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      appleImgSrc: "../assets/apple.jpg",
      msg: "Welcome to Your Snake Game, You've already eaten ",
      eatCount:0,
      ctx: null,
      mySnakeNode: [],
      mapWide: 800,
      mapHeight: 500,
      snakeHeadR: 20,
      appleR: 30,
      // 输入的 x
      keyX: 0,
      // 输入的y
      keyY: 0,
      moveQueue: [],
      apples: [],
    };
  },
  methods: {
    init: function () {
      console.log("start init method");
      this.refreshSnake();
      this.addSnakeNode(30, 20);
      this.addSnakeNode(30, 20);
      var canvas = document.getElementById("canvas");
      this.ctx = canvas.getContext("2d");
      var w = this.mapWide;
      var h = this.mapHeight;
      var i = setInterval(() => {
        this.ctx.clearRect(0, 0, w, h);
        if (this.moveQueue != null && this.moveQueue.length != 0) {
          console.log("开始move");
          this.moveSnake();
          this.eatApple();
        }
        if (this.apples != null && this.apples.length != 0) {
          this.drawApple();
        }
        this.drawSnake();
      }, 80);
      var addApple = setInterval(() => {
        this.addApples();
      }, 2000);
    },
    drawSnake: function () {
      let index = 0;
      this.mySnakeNode.forEach((node) => {
        this.ctx.beginPath();
        index++;
        this.ctx.arc(node.x, node.y, this.snakeHeadR, 0, Math.PI * 2);
        if (index == 1) {
          this.ctx.fillStyle = "red";
        } else {
          this.ctx.fillStyle = "gold";
        }
        this.ctx.fill();
      });
      
    },
    drawApple: function () {
      var apple = document.getElementById("apple");
      this.apples.forEach((a) => {
        this.ctx.drawImage(apple, a.x, a.y, this.appleR, this.appleR);
      });
    },
    refreshSnake: function () {
      // 第一次进入地图，初始化头节点
      if (this.mySnakeNode == null || this.mySnakeNode.length == 0) {
        let snakeNode = {
          x: 20,
          y: 20,
        };
        this.mySnakeNode.push(snakeNode);
      }
    },
    // 加snakeNode, 前提mySnakeNode元素至少有一个哈
    addSnakeNode: function (x, y) {
      // 在第一个node的前面加一个node
      let snakeNode = {
        x: x,
        y: y,
      };
      this.mySnakeNode.push(snakeNode);
    },
    addApples: function () {
      let appleX = Math.round(Math.random() * (this.mapWide - this.appleR));
      let appleY = Math.round(Math.random() * (this.mapHeight - this.appleR));
      let apple = {
        x: appleX,
        y: appleY,
      };
      this.apples.push(apple);
    },
    /**
     * x: 外界输入的x偏移量
     * y: 外界输入的y偏移量
     * 方法根据画布边界，以及当前的snakeFirstNode在具体判断真实的偏移量(moveX, moveY)
     */
    moveController: function (x, y, snakeFirstNode) {
      let cx = snakeFirstNode.x;
      let cy = snakeFirstNode.y;
      let moveX = 0;
      let moveY = 0;
      if (x >= 0) {
        if (cx + x + this.snakeHeadR <= this.mapWide) {
          moveX = x;
          // console.log("moveX = x:" + x);
        } else {
          // 否则最多到达边界
          moveX = this.mapWide - cx - this.snakeHeadR;
          // console.log("moveX = this.mapWide - cx - snakeHeadR:" + moveX);
        }
      } else {
        if (cx + x - this.snakeHeadR >= 0) {
          moveX = x;
        } else {
          // 否则最多到达边界
          moveX = 0 - cx + this.snakeHeadR;
        }
      }
      if (y >= 0) {
        if (cy + y + this.snakeHeadR <= this.mapHeight) {
          moveY = y;
        } else {
          // 否则最多到达边界
          moveY = this.mapHeight - cy - this.snakeHeadR;
        }
      } else {
        if (cy + y - this.snakeHeadR >= 0) {
          moveY = y;
        } else {
          // 否则最多到达边界
          moveY = 0 - cy + this.snakeHeadR;
        }
      }
      let moveObj = {
        moveX: moveX,
        moveY: moveY,
      };
      this.moveQueue.push(moveObj);
    },
    // 设置snake的位置
    moveSnake() {
      // 拿到偏移量
      let moveObj = this.moveQueue.shift();
      //引用对象深拷贝
      let snakeTempNode = [];
      this.mySnakeNode.forEach((node) => {
        let newNode = {
          x: node.x,
          y: node.y,
        };
        snakeTempNode.push(newNode);
      });
      // console.log("头移动前：" + this.mySnakeNode[0].x + "-" + this.mySnakeNode[0].y)
      // 头根据偏移量移动，后面节点根据前面节点移动
      this.mySnakeNode[0].x = this.mySnakeNode[0].x + moveObj.moveX;
      this.mySnakeNode[0].y = this.mySnakeNode[0].y + moveObj.moveY;
      // console.log("头移动后：" + this.mySnakeNode[0].x + "-" + this.mySnakeNode[0].y)
      for (let i = 1; i < this.mySnakeNode.length; i++) {
        console.log(
          i + "移动前：" + this.mySnakeNode[i].x + "-" + this.mySnakeNode[i].y
        );
        this.mySnakeNode[i].x = snakeTempNode[i - 1].x;
        this.mySnakeNode[i].y = snakeTempNode[i - 1].y;
        // console.log(i+"移动后：" + this.mySnakeNode[i].x + "-" + this.mySnakeNode[i].y)
      }
    },
    // 监听键盘
    keyDown() {
      document.onkeydown = (e) => {
        //事件对象兼容
        let e1 =
          e || event || window.event || arguments.callee.caller.arguments[0];
        //键盘按键判断:左箭头-37;上箭头-38；右箭头-39;下箭头-40
        //左
        if (e1 && e1.keyCode == 37) {
          // 按下左箭头
          this.keyX = -this.snakeHeadR;
          this.keyY = 0;
          this.moveController(this.keyX, this.keyY, this.mySnakeNode[0]);
        } else if (e1 && e1.keyCode == 38) {
          // 按下上箭头
          this.keyY = -this.snakeHeadR;
          this.keyX = 0;
          this.moveController(this.keyX, this.keyY, this.mySnakeNode[0]);
        } else if (e1 && e1.keyCode == 39) {
          // 按下右箭头
          this.keyX = this.snakeHeadR;
          this.keyY = 0;
          this.moveController(this.keyX, this.keyY, this.mySnakeNode[0]);
        } else if (e1 && e1.keyCode == 40) {
          // 按下下箭头
          this.keyY = this.snakeHeadR;
          this.keyX = 0;
          this.moveController(this.keyX, this.keyY, this.mySnakeNode[0]);
        }
      };
    },
    eatApple() {
      let deleteIndex = [];
      // snakeNode 是否 碰到 apples
      this.mySnakeNode.forEach((node) => {
        // 每一个node的范围是否有apple
        for (let i = 0; i < this.apples.length; i++) {
          let apple = this.apples[i];
          let len = Math.sqrt(
            (apple.x - node.x) * (apple.x - node.x) +
              (apple.y - node.y) * (apple.y - node.y)
          );
          // let canEat =
          //   Math.abs(apple.x - node.x) <= 15 &&
          //   Math.abs(apple.y - node.y) <= 15;
          if ((len + this.snakeHeadR) <= (this.snakeHeadR + this.appleR)) {
            // 吃掉这个apple, snakeNode ++
            deleteIndex.push(i);
            this.eatCount++;
            this.addSnakeNode(
              this.mySnakeNode[this.mySnakeNode.length - 1].x,
              this.mySnakeNode[this.mySnakeNode.length - 1].y
            );
          }
        }
        deleteIndex.forEach((d) => {
          this.apples.splice(d, 1);
        });
        deleteIndex = [];
      });
    },
  },
  mounted() {
    this.init();
    this.keyDown();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
#canvas {
  margin: 0 auto;
  border: 2px solid;
  display: block;
}
</style>
