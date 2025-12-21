# <!DOCTYPE html>
<html>
<head>
    <title>PSFT Telescope Simulator</title>
    <script type="text/javascript" src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script type="text/javascript" src="https://aladin.cds.unistra.fr/AladinLite/api/v3/latest/aladin.js" charset="utf-8"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/p5.js"></script>
    <style>
        body { 
            margin: 0; 
            background: #000; 
            overflow: hidden; 
            color: #0cf; 
            font-family: sans-serif; 
        }
        #aladin-lite-div { 
            position: absolute; 
            top: 0; 
            left: 0; 
            width: 100%; 
            height: 100%; 
            z-index: 1; 
        }
        #p5-canvas { 
            position: absolute; 
            top: 0; 
            left: 0; 
            z-index: 2; 
            pointer-events: none; 
        }
        .ui-panel { 
            position: absolute; 
            top: 10px; 
            left: 10px; 
            background: rgba(0, 10, 20, 0.85); 
            padding: 15px; 
            border-radius: 8px; 
            border: 1px solid #0cf; 
            width: 260px; 
            z-index: 3; 
        }
        input[type="text"] { 
            width: 90%; 
            padding: 5px; 
            margin-bottom: 10px; 
            background: #002; 
            border: 1px solid #0cf; 
            color: #fff; 
        }
        button {
            padding: 5px 15px;
            background: #004466;
            color: #0cf;
            border: 1px solid #0cf;
            cursor: pointer;
            border-radius: 4px;
        }
        button:hover {
            background: #005588;
        }
        hr {
            border: none;
            height: 1px;
            background: #0cf;
            margin: 10px 0;
        }
    </style>
</head>
<body>
    <div id="aladin-lite-div"></div>
    <div id="p5-canvas"></div>
    <div class="ui-panel">
        <h3 style="margin-top:0;">PSFT Telescope View</h3>
        <input type="text" id="target" placeholder="Type Galaxy (e.g. M31, M51)" value="M31">
        <button onclick="gotoTarget()">GO TO GALAXY</button>
        <hr>
        <label>α (Coupling): <span id="alphaVal">1500</span></label><br><br>
        <input type="range" id="alphaSlider" min="100" max="5000" value="1500" style="width:100%;">
    </div>

    <script>
        // Global variables
        let aladin;
        let p5Canvas;
        let stars = [];
        
        // Initialize when page loads
        window.onload = function() {
            // Initialize Aladin Lite
            aladin = A.aladin('#aladin-lite-div', {
                survey: "P/DSS2/color",
                fov: 0.5,
                target: "M31",
                showZoomControl: false,
                showFullscreenControl: false
            });
            
            // Initialize P5.js
            initP5();
            
            // Update slider value display
            document.getElementById('alphaSlider').addEventListener('input', function() {
                document.getElementById('alphaVal').innerText = this.value;
            });
        };
        
        function gotoTarget() {
            let target = document.getElementById('target').value.trim();
            if (target) {
                aladin.gotoObject(target);
            }
        }
        
        function initP5() {
            // Create P5 sketch
            let sketch = function(p) {
                p.setup = function() {
                    let canvas = p.createCanvas(window.innerWidth, window.innerHeight);
                    canvas.id('p5-canvas');
                    canvas.position(0, 0);
                    canvas.style('pointer-events', 'none');
                    canvas.style('z-index', '2');
                    
                    // Create stars
                    for (let i = 0; i < 300; i++) {
                        stars.push(new Star(p));
                    }
                };
                
                p.draw = function() {
                    p.clear(); // Transparent background
                    p.translate(p.width / 2, p.height / 2);
                    
                    for (let star of stars) {
                        star.update();
                        star.show();
                    }
                };
                
                p.windowResized = function() {
                    p.resizeCanvas(window.innerWidth, window.innerHeight);
                };
            };
            
            new p5(sketch);
        }
        
        // Star class
        class Star {
            constructor(p) {
                this.p = p;
                this.angle = p.random(p.TWO_PI);
                this.r = p.random(50, 300);
                this.z = p.random(1, 3);
                this.speedFactor = 0.05;
            }
            
            update() {
                let a = document.getElementById('alphaSlider').value;
                let speed = (Math.sqrt(a / (this.r * 0.5))) / this.r;
                this.angle += speed * this.speedFactor;
            }
            
            show() {
                let x = this.r * this.p.cos(this.angle);
                let y = this.r * this.p.sin(this.angle);
                this.p.stroke(150, 220, 255, 180);
                this.p.strokeWeight(this.z);
                this.p.point(x, y);
            }
        }
    </script>
</body>
</html>