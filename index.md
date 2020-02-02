Dawid's game
<html>
<body>
<script>
console.log('Welcome to your world.Your goal is to defeat the robots because they are enemies.')

//TESTY

var komoda = [12,93,72,128,199,88,77];

for (var szuflada = 1; szuflada<komoda.length; szuflada++) {
console.log(komoda[szuflada]);
}

var szkola = {
klasy:8,
nauczyciele:16,
dzieci:132};

console.log(szkola);

szkola.klasy++;

console.log(szkola);

szkola.nauczyciele--;

console.log(szkola);

console.log(szkola);

console.log (szkola.nauczyciele + szkola.dzieci);

for (var i=0; i<10; i++){console.log(Math.random());
}
//*************************************\\


//CONSTANTS

var CANVAS_WIDTH = 800;
var CANVAS_HEIGHT = 600;
var NANONAUT_WIDTH = 181;
var NANONAUT_HEIGHT = 229;
var GROUND_Y = 540;
var NANONAUT_Y_ACCELERATION = 1;
var SPACE_KEYCODE = 32;
var NANONAUT_JUMP_SPEED = 20;
var NANONAUT_X_SPEED = 5;
var BACKGROUND_WIDTH = 1000;
//var NANONAUT_NR_FRAMES_PER_ROW = 5;
var NANONAUT_NR_ANIMATION_FRAMES = 7;
var NANONAUT_ANIMATION_SPEED = 3;
var ROBOT_WIDTH = 141;
var ROBOT_HEIGHT = 139;
var ROBOT_NR_ANIMATION_FRAMES = 9;
var ROBOT_ANIMATION_SPEED = 5;
var ROBOT_X_SPEED = 4;
var MIN_DISTANCE_BETWEEN_ROBOTS = 400;
var MAX_DISTANCE_BETWEEN_ROBOTS = 1200;
var MAX_ACTIVE_ROBOTS = 3;


//SETUP

var cameraX = 0;
var cameraY = 0;
var canvas = document.createElement('canvas');
var c = canvas.getContext('2d');
canvas.width = CANVAS_WIDTH;
canvas.height = CANVAS_HEIGHT;
document.body.appendChild(canvas);

var nanonautImage = new Image();
nanonautImage.src = 'animatedNanonaut.png';

var backgroundImage = new Image();
backgroundImage.src = 'background.png';

var bush1Image = new Image();
bush1Image.src = 'bush1.png';

var bush2Image = new Image();
bush2Image.src = 'bush2.png';

var robotImage = new Image();
robotImage.src = 'animatedRobot.png';

var nanonautSpriteSheet = {
nrFramesPerRow: 5,
spriteWidth: NANONAUT_WIDTH,
spriteHeight: NANONAUT_HEIGHT,
image: nanonautImage
};

var robotSpriteSheet = {
nrFramesPerRow: 3,
spriteWidth:ROBOT_WIDTH,
spriteHeight: ROBOT_HEIGHT,
image: robotImage
};

var robotData = [];




var nanonautX = CANVAS_WIDTH / 2;
var nanonautY = GROUND_Y - NANONAUT_HEIGHT;

var nanonautYSpeed = 0;
var nanonautIsInTheAir = false;

//window.addEventListener('touchstart',onKeyDown);
//window.addEventListener('touchend',onKeyUp);
//window.addEventListener('keydown',onKeyDown);
//window.addEventListener('keyup',onKeyUp);
window.addEventListener('load',start);

var spaceKeyIsPressed = false;
var nanonautFrameNr = 0;

var gameFrameCounter = 0;
var bushData = generateBushes();

function start() {
window.requestAnimationFrame(mainLoop);
document.body.addEventListener('touchstart', onKeyDown);
document.body.addEventListener('touchend', onKeyUp);
}

function generateBushes(){
var generatedBushData = [];
var bushX = 0
while (bushX<(2*CANVAS_WIDTH)){
var bushImage;
if(Math.random()>=0.5) {
bushImage = bush1Image;
} else {
bushImage = bush2Image;}
generatedBushData.push({x:bushX,
y:80+Math.random()*20,
image:bushImage});
bushX += 150+Math.random()*200;
}
return generatedBushData;}



//MAIN LOOP

function mainLoop(){
update();
draw();
window.requestAnimationFrame(mainLoop);
}


//PLAYER INPUT

function onKeyDown(event) {
//console.log(event.keyCode);
if (event.keyCode === SPACE_KEYCODE) {
spaceKeyIsPressed = true;
}
}

function onKeyUp(event) {
if (event.keyCode === SPACE_KEYCODE) {
spaceKeyIsPressed = false;
}
}


//UPDATING

function update(){
gameFrameCounter = gameFrameCounter + 1;
//update nanonaut
nanonautX = nanonautX + NANONAUT_X_SPEED;

if (spaceKeyIsPressed && !nanonautIsInTheAir) {
nanonautYSpeed = -NANONAUT_JUMP_SPEED;
nanonautIsInTheAir = true;
}
nanonautY = nanonautY + nanonautYSpeed;
nanonautYSpeed = nanonautYSpeed + NANONAUT_Y_ACCELERATION;
if (nanonautY > (GROUND_Y - NANONAUT_HEIGHT)) {
nanonautY = GROUND_Y - NANONAUT_HEIGHT;
nanonautYSpeed = 0;
nanonautIsInTheAir = false;
}
// update animation
if ((gameFrameCounter % NANONAUT_ANIMATION_SPEED) === 0){
nanonautFrameNr = nanonautFrameNr + 1;
if (nanonautFrameNr >= NANONAUT_NR_ANIMATION_FRAMES) {
nanonautFrameNr = 0;
}
}

//update camera
cameraX = nanonautX - 150;


//UPDATE bushes
for (var i = 0;i<bushData.length; i++) {
if ((bushData[i].x - cameraX)< - CANVAS_WIDTH){
bushData[i].x +=(2*CANVAS_WIDTH)+150;}}

//UPDATE ROBOTS
updateRobots();
}

function updateRobots() {
//move & animate robots
for (var i = 0;i<robotData.length;i++) {
robotData[i].x-=ROBOT_X_SPEED;
if ((gameFrameCounter % ROBOT_ANIMATION_SPEED) === 0) {
robotData[i].frameNr++;
if (robotData[i].frameNr>= ROBOT_NR_ANIMATION_FRAMES) {
robotData[i].frameNr = 0;
}
}
}
//REMOVE ROBOTS
var robotIndex = 0;
while(robotIndex<robotData.length) {
if (robotData[robotIndex].x<cameraX-ROBOT_WIDTH){
robotData.splice(robotIndex,1);
console.log("i removed a ROBOT");
}else {
robotIndex++;
}
}
if (robotData.length<MAX_ACTIVE_ROBOTS) {
var lastRobotX = CANVAS_WIDTH;
if (robotData.length>0) {
 lastRobotX = robotData[robotData.length-1].x;
 }
var newRobotX = lastRobotX+MIN_DISTANCE_BETWEEN_ROBOTS+Math.random()*(MAX_DISTANCE_BETWEEN_ROBOTS-MIN_DISTANCE_BETWEEN_ROBOTS);
robotData.push({
x:newRobotX,
y:GROUND_Y-ROBOT_HEIGHT,
frameNr:0
});
}
}


//DRAWING

function draw(){
//c.clearRect(0,0,CANVAS_WIDTH,CANVAS_HEIGHT);

//Draw the sky.
c.fillStyle = 'lightskyblue';
c.fillRect(0,0,CANVAS_WIDTH,GROUND_Y - 40);

//Draw the background.
var backgroundX = -(cameraX % BACKGROUND_WIDTH);
c.drawImage(backgroundImage,backgroundX,-210);
c.drawImage(backgroundImage,backgroundX + BACKGROUND_WIDTH,-210);
//Draw the ground.
c.fillStyle =  'rgb(56,255,43)';
c.fillRect(0,GROUND_Y - 40,CANVAS_WIDTH,CANVAS_HEIGHT - GROUND_Y + 40);

//draw the bush
for (var i = 0; i<bushData.length;i++){
c.drawImage(bushData[i].image,bushData[i].x - cameraX,GROUND_Y - bushData[i].y - cameraY);
}
//c.drawImage(bush1Image,750,GROUND_Y - 90);
//draw the nanonaut
//c.drawImage(nanonautImage,nanonautX - cameraX,nanonautY - cameraY);

//draw the robots
for (var i = 0; i<robotData.length; i++) {
drawAnimatedSprite(robotData[i].x-cameraX,
robotData[i].y-cameraY,robotData[i].frameNr,robotSpriteSheet);
}

//Draw the Nanonaut.
drawAnimatedSprite(nanonautX-cameraX,nanonautY-cameraY,
nanonautFrameNr,nanonautSpriteSheet);

//Draw the Nanonaut.
//var nanonautSpriteSheetRow = Math.floor(nanonautFrameNr/NANONAUT_NR_FRAMES_PER_ROW);
//var nanonautSpriteSheetColumn = nanonautFrameNr% NANONAUT_NR_FRAMES_PER_ROW;
//var nanonautSpriteSheetSheetX = nanonautSpriteSheetColumn * NANONAUT_WIDTH;
//var nanonautSpriteSheetSheetY = nanonautSpriteSheetRow * NANONAUT_HEIGHT;
//c.drawImage(nanonautImage,nanonautSpriteSheetSheetX,nanonautSpriteSheetSheetY,NANONAUT_WIDTH,NANONAUT_HEIGHT,nanonautX - cameraX,nanonautY - //cameraY,NANONAUT_WIDTH,NANONAUT_HEIGHT);
}

//draw animated sprite
function drawAnimatedSprite(screenX,screenY,frameNr,spriteSheet) {
var spriteSheetRow = Math.floor(frameNr / spriteSheet.nrFramesPerRow);
var spriteSheetColumn = frameNr % spriteSheet.nrFramesPerRow;
var spriteSheetX = spriteSheetColumn * spriteSheet.spriteWidth;
var spriteSheetY = spriteSheetRow * spriteSheet.spriteHeight;
c.drawImage(
spriteSheet.image,
spriteSheetX, spriteSheetY, 
spriteSheet.spriteWidth, spriteSheet.spriteHeight, screenX, screenY,
spriteSheet.spriteWidth, spriteSheet.spriteHeight
);
}


</script>
</body>
</html>
