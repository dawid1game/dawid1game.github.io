<html>
<head>
<title>NINJA ACTION</title>

<body>
<script>

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
var SCREENSHAKE_RADIUS = 16;
var NANONAUT_MAX_HEALTH = 100;
var PLAY_GAME_MODE = 0;
var GAME_OVER_GAME_MODE = 1;
var SPLASH_SCREEN = 2;


//SETUP

var gameMode = SPLASH_SCREEN;

var cameraX = 0;
var cameraY = 0;
var canvas = document.createElement('canvas');
var c = canvas.getContext('2d');
canvas.width = CANVAS_WIDTH;
canvas.height = CANVAS_HEIGHT;
canvas.style.left = 5;
canvas.style.top = -70;
canvas.style.position = "absolute";
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

var nanonautCollisionRectangle = {
xOffset:60,
yOffset:20,
width:50,
height:200};

var robotCollisionRectangle = {
xOffset:50,
yOffset:20,
width:50,
height:100};



var nanonautX = CANVAS_WIDTH / 2;
var nanonautY = GROUND_Y - NANONAUT_HEIGHT;

var nanonautYSpeed = 0;
var nanonautIsInTheAir = false;

window.addEventListener('touchstart',onTouchDown);
window.addEventListener('touchend',onTouchUp);
window.addEventListener('keydown',onKeyDown);
window.addEventListener('keyup',onKeyUp);
window.addEventListener('load',start);

var spaceKeyIsPressed = false;
var nanonautFrameNr = 0;

var gameFrameCounter = 0;
var bushData = generateBushes();

var screenshake = false;
var nanonautHealth = NANONAUT_MAX_HEALTH;

function start() {
window.requestAnimationFrame(mainLoop);
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
if (gameMode === SPLASH_SCREEN) {
gameMode = PLAY_GAME_MODE;
}
}
}

function onTouchDown(event) {
spaceKeyIsPressed = true;
if (gameMode === SPLASH_SCREEN) {
gameMode = PLAY_GAME_MODE;
}
}

function onKeyUp(event) {
if (event.keyCode === SPACE_KEYCODE) {
spaceKeyIsPressed = false;
}
}

function onTouchUp(event) {
spaceKeyIsPressed = false;
}


//UPDATING

function update(){
if (gameMode != PLAY_GAME_MODE) return;
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
screenshake = false;
var nanonautTouchedARobot = updateRobots();
if (nanonautTouchedARobot) {
screenshake = true;
if (nanonautHealth > 0) nanonautHealth--;
}
if (nanonautHealth<=0) {
gameMode = GAME_OVER_GAME_MODE;
screenshake = false;
}
}

function updateRobots() {
//move & animate robots
var nanonautTouchedARobot = false;
for (var i = 0;i<robotData.length;i++) {
if (doesNanonautOverlapRobot(nanonautX+nanonautCollisionRectangle.xOffset,
nanonautY+nanonautCollisionRectangle.yOffset,
nanonautCollisionRectangle.width,
nanonautCollisionRectangle.height,
robotData[i].x+robotCollisionRectangle.xOffset,
robotData[i].y+robotCollisionRectangle.yOffset,
robotCollisionRectangle.width,
robotCollisionRectangle.height)) {
nanonautTouchedARobot = true;
console.log('OUCH!');
}
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
return nanonautTouchedARobot;

}

function doesNanonautOverlapRobotAlongOneAxis(nanonautNearX,nanonautFarX,robotNearX,robotFarX) {
var nanonautOverlapsNearRobotEdge = (nanonautFarX >= robotNearX) && (nanonautFarX<=robotFarX);
var nanonautOverlapsFarRobotEdge = (nanonautNearX >= robotNearX) && (nanonautNearX<=robotFarX);
var nanonautOverlapsEntireRobot = (nanonautNearX<=robotNearX) && (nanonautFarX >=robotFarX);
return nanonautOverlapsNearRobotEdge || nanonautOverlapsFarRobotEdge || nanonautOverlapsEntireRobot;
}

function doesNanonautOverlapRobot(nanonautX,nanonautY,nanonautWidth,nanonautHeight,robotX,robotY,robotWidth,robotHeight)
{
var nanonautOverlapsRobotOnXAxis = doesNanonautOverlapRobotAlongOneAxis(nanonautX,nanonautX+nanonautWidth,robotX,robotX+robotWidth);
var nanonautOverlapsRobotOnYAxis = doesNanonautOverlapRobotAlongOneAxis(nanonautY,nanonautY+nanonautHeight,robotY,robotY+robotHeight);
return nanonautOverlapsRobotOnXAxis && nanonautOverlapsRobotOnYAxis;
}


//DRAWING

function draw(){
//shake the screen if nes.

var shakenCameraX = cameraX;
var shakenCameraY = cameraY;

if (screenshake) {
shakenCameraX+=(Math.random()-0.5)*SCREENSHAKE_RADIUS;
shakenCameraY+=(Math.random()-0.5)*SCREENSHAKE_RADIUS;
}


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

// draw splash screen
if (gameMode == SPLASH_SCREEN) {
c.fillStyle = 'yellow';
c.fillRect(100, 100, 580, 450);
c.fillStyle = 'black';
c.font = '50px sans-serif';
c.fillText('        Dawid\'s game', 120, 160);
c.fillText('        in support of', 120, 220);
c.fillText('   Mary\'s Meals charity',120,290);
c.font = '25px sans-serif';
c.fillText('Press space bar or tap screen to start and jump', 120, 370);
c.font = '14px sans-serif';
c.fillText('         Game is based on a book "Create with code" by CoderDojo Foundation.', 120, 480);
//enable below when ads work
//c.fillText('Please consider clicking on the ad below - all proceeds will be donated to Mary\'s Meals.',120, 510);
return;
}

//draw the bush
for (var i = 0; i<bushData.length;i++){
c.drawImage(bushData[i].image,bushData[i].x - shakenCameraX,GROUND_Y - bushData[i].y - shakenCameraY);
}
//c.drawImage(bush1Image,750,GROUND_Y - 90);
//draw the nanonaut
//c.drawImage(nanonautImage,nanonautX - cameraX,nanonautY - cameraY);

//draw the robots
for (var i = 0; i<robotData.length; i++) {
drawAnimatedSprite(robotData[i].x-shakenCameraX,
robotData[i].y-shakenCameraY,robotData[i].frameNr,robotSpriteSheet);
}

//Draw the Nanonaut.
drawAnimatedSprite(nanonautX-shakenCameraX,nanonautY-shakenCameraY,
nanonautFrameNr,nanonautSpriteSheet);

//draw distance
var nanonautDistance = nanonautX/100;
c.fillStyle = 'black';
c.font = '48px sans-serif';
c.fillText(nanonautDistance.toFixed(0)+'m',20,40);

//draw Health bar
c.fillStyle = 'red';
c.fillRect(400,10,nanonautHealth / NANONAUT_MAX_HEALTH*380,20);
c.strokeStyle = 'red';
c.strokeRect(400,10,380,20);

// draw game over
if (gameMode == GAME_OVER_GAME_MODE) {
c.fillStyle = 'black';
c.font = '96px sans-serif';
c.fillText('GAME OVER',120,300);
}
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

<script data-ad-client="ca-pub-5113161158199844" async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>

</body>
</head>
</html>
