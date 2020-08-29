<template>
    <div>
        <canvas ref="game" width="640" height="480" style="border: 1px solid black;"></canvas>
    </div>
</template> 

<script>
    import io from "socket.io-client";
    //var direction = "none";
    export default {
        name: 'BlockGame',
        data() {
            return {
                socket: {},  // initialisation
                context: {},
                position: {
                    x: 0,
                    y: 0
                },
                id: 0,
                nUsers: 0,
                imgURLs: [],
                images: []
            }
        },
        created() { 
            this.socket = io("http://localhost:3030");
        },
        mounted() {
            this.context = this.$refs.game.getContext("2d");
            this.socket.on("init", data => {  // Initial data, for each client
              console.log("Init");
              console.log(data.id);
              this.id = data.id;  // Unique client ID
              this.imgURLs = data.imgURLs;
            })
            this.socket.on("nUsers", data => {  // Received each time a user joins
              console.log("nUsers received: " + data);
              this.nUsers = data;
              this.images = [];
              for (var i = 0; i < this.nUsers; i++) {  // Re-generate user character models, probably replace
                this.images.push(new Image(5, 5));
              }
            })
            this.socket.on("position", data => {  // Received each time a player moves (position update)
                // Probably replace with position update every 1/60 second (server side)
              const ctx = this.context;

              console.log(this.nUsers);
              ctx.clearRect(0, 0, this.$refs.game.width, this.$refs.game.height);
              ctx.fillRect(100, 100, 100, 100)
              for (var i = 0; i < this.nUsers; i++) {
                var posx = data[i].x;
                var posy = data[i].y;
                
                //
                // Drawing code:
                //

                this.images[i].src = this.imgURLs[i];
                //this.images[i].style.height = '10px';
                //this.images[i].style.width = '10px';
                ctx.drawImage(this.images[i], posx, posy, 70, 70);
                //ctx.fillRect(posx, posy, 20, 20);
                // Add images
              }
              
            });
                document.addEventListener('keypress', event => {  // Movement code, Judd is working on improvements
                  console.log(event.keyCode);
                if (event.keyCode == 100) {
                    console.log("right");
                    console.log({id: this.id, direction: "right"});
                    this.socket.emit("move", {id: this.id, direction: "right"});
                    //move("right")
                }
                if (event.keyCode == 97) { 
                    //this.socket.emit("move", "left")
                    //move("left")
                    this.socket.emit("move", {id: this.id, direction: "left"});
                }
                if (event.keyCode == 119) {
                    //this.socket.emit("move", "up")
                    //move("up")
                    this.socket.emit("move", {id: this.id, direction: "up"});
                }
                if (event.keyCode == 115) {
                    //this.socket.emit("move", "down")
                    //move("down")
                    this.socket.emit("move", {id: this.id, direction: "down"});
                }
            })
        },
        methods: {
          // move(direction) {
          //   this.socket.emit("move", direction);
          // }



        }
    }

</script>

<style scoped></style>