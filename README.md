# Noel
Truy cập link: editor.p5js.org
# Mục Html nhập
<!DOCTYPE html>
<html lang="en">
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/addons/p5.sound.min.js"></script>
    <!-- Thư viện nhận diện tay ml5.js -->
    <script src="https://unpkg.com/ml5@1/dist/ml5.min.js"></script>
    <link rel="stylesheet" type="text/css" href="style.css">
    <meta charset="utf-8" />

  </head>
  <body>
    <main>
    </main>
    <script src="sketch.js"></script>
  </body>
</html>

# Mục sketch.js nhập

function setup() {
  createCanvas(400, 400);
}

function draw() {
  background(220);
}
let video;
let handPose;
let hands = [];
let particles = [];
let treeScale = 1; 
let isFist = false; 
let textImg; // Biến chứa ảnh chữ

const NUM_PARTICLES = 600; 

function setup() {
  createCanvas(windowWidth, windowHeight, WEBGL);
  
  video = createCapture(VIDEO);
  video.size(640, 480);
  video.hide();

  // Khởi tạo handpose model và bắt sự kiện predict
  handPose = ml5.handpose(video, modelReady);
  handPose.on('predict', gotHands);

  for (let i = 0; i < NUM_PARTICLES; i++) {
    particles.push(new Particle(i));
  }

  // --- TẠO HÌNH ẢNH CHỨA CHỮ ---
  textImg = createGraphics(500, 150); // Tạo khung ảo
  textImg.textAlign(CENTER, CENTER);
  textImg.textSize(50);             // Kích cỡ chữ
  textImg.textStyle(BOLD);          // Chữ đậm
  textImg.fill(255, 223, 0);        // Màu vàng Gold
  textImg.text("MERRY CHRISTMAS", 250, 75); // Vẽ chữ vào giữa khung ảo
}

function modelReady() {
  console.log('Handpose model ready!');
}

function draw() {
  background(10, 10, 30); 
  
  // Ánh sáng
  ambientLight(100);
  pointLight(255, 255, 255, 0, 0, 200);

  // Xử lý tay
  if (hands.length > 0) {
    let hand = hands[0];
    
    // Đo độ to nhỏ
    let wrist = hand.landmarks[0];
    let middleFinger = hand.landmarks[9];
    let d = dist(wrist[0], wrist[1], middleFinger[0], middleFinger[1]);
    treeScale = map(d, 50, 200, 0.5, 2.5, true);

    // Đo nắm/thả
    let indexTip = hand.landmarks[8];
    let thumbTip = hand.landmarks[4];
    let pinchDist = dist(indexTip[0], indexTip[1], thumbTip[0], thumbTip[1]);
    
    if (pinchDist < 40) isFist = true;
    else isFist = false;

  } else {
    isFist = false;
    treeScale = 1;
  }

  rotateY(frameCount * 0.01);

  // --- VẼ CHỮ DƯỚI GỐC CÂY ---
  push();
  // Di chuyển vị trí chữ xuống dưới gốc cây (220 đơn vị * tỷ lệ)
  translate(0, 180 * treeScale, 0); 
  
  // Dán hình chữ lên
  texture(textImg); 
  noStroke();
  
  // Vẽ tấm bảng chứa chữ (kích thước to nhỏ theo cây)
  plane(250 * treeScale, 75 * treeScale);
  pop();
  // ---------------------------

  // Vẽ hạt
  for (let p of particles) {
    p.update(isFist, treeScale);
    p.show(treeScale);
  }
}

function gotHands(results) {
  hands = results;
}

class Particle {
  constructor(index) {
    this.pos = createVector(random(-500, 500), random(-500, 500), random(-500, 500));
    this.vel = p5.Vector.random3D().mult(random(2, 5));
    
    // Tạo hình xoắn ốc nón
    let t = index / NUM_PARTICLES; 
    let angle = t * 30; 
    let radius = map(t, 0, 1, 150, 0); 
    let y = map(t, 0, 1, 150, -200); // Điều chỉnh lại chiều cao một chút cho đẹp
    
    let x = radius * cos(angle);
    let z = radius * sin(angle);
    
    this.target = createVector(x, y, z);
    
    if (random() < 0.1) this.color = color(255, 50, 50); 
    else if (random() < 0.1) this.color = color(255, 215, 0); 
    else this.color = color(50, 200, 100); 
  }

  update(isFist, scaleFactor) {
    if (isFist) {
      let scaledTarget = this.target.copy().mult(scaleFactor);
      this.pos.lerp(scaledTarget, 0.05); 
    } else {
      this.pos.add(this.vel);
      if (this.pos.mag() > 800 * scaleFactor) {
         this.pos = p5.Vector.random3D().mult(100);
      }
    }
  }

  show(scale) {
    push();
    translate(this.pos.x, this.pos.y, this.pos.z);
    noStroke();
    fill(this.color);
    sphere(3 * scale); // Hạt cũng to nhỏ theo scale
    pop();
  }
}

