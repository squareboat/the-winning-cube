# The Winning Cube
I used three.js to build a 3D puzzle game in which you navigate through cubes to reach a goal. You can navigate through keyboard or hand gestures or simply using voice commands.

## Getting Started
In 3D world, the following things are necessarily needed :
- A Scene
- A Camera
- A Renderer
- An object or two (with materials)

Let’s see how I created my first three.js project, step-by-step.

First of all, we need to set the scene by defining our camera and renderer.
```
// Create a Scene
var scene = new THREE.Scene();

// Create a Camera
var camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );

// Create a Renderer
var renderer = new THREE.WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
document.body.appendChild( renderer.domElement );
renderer.setClearColor( 0x777777, 0.3);

// For creating responsive game on resizing
window.addEventListener( 'resize', function()
{
  renderer.setSize( window.innerWidth, window.innerHeight );
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix( );
} );

// Draw Scene
var render = function()
{
  renderer.render( scene, camera );
};

// Run Game Loop ( Update, render, Repeat )
var GameLoop = function()
{
  requestAnimationFrame( GameLoop );
  render();
};

GameLoop();
```

After creating the scene, we need to define our material and what object we are going to create. For a cubic puzzle I had to create various cubes of different sizes and colors.
```
// Create Outer Cube
var outerBoxGeometry = new THREE.BoxGeometry( 3, 3, 3 );

// Create a material, colour or image texture
var outerBoxMaterial = new THREE.MeshLambertMaterial( {
  color: 0xffffff,
  transparent: true,
  opacity: 0,
  wireframe: true,
} );
var cubeouter = new THREE.Mesh( outerBoxGeometry, outerBoxMaterial );
scene.add( cubeouter );
```

Now, we have our cubes in the place. But, without the light sources, the colors won’t reflect. Now, we need to define our light sources.
```
// Lighning
ambientLight = new THREE.AmbientLight( 0xFFFFFF, 0.2 );
scene.add( ambientLight );

hemisphereLight = new THREE.HemisphereLight(0xffffff, 0x000000, 0.2);
scene.add( hemisphereLight );
```

We have our puzzle ready. The only thing left is defining controls which we will be using to solve the puzzle. Now, I have provided three types of controls to help our cube reach the final position:
- One with our regular keyboard.
- Using our hand gestures
- Using voice commands

For controlling through keyboard, three.js already provides you with OrbitControls.js using which you can define your keyboard controls.
```
// Keyboard functions
function Keyboard() {
  var speed = 1;
  var old = {
    x: cube.position.x,
    y: cube.position.y,
    z: cube.position.z
  };

  if( event.keyCode == 65) { cube.position.x -= speed } // Left a
  else if( event.keyCode == 68) { cube.position.x += speed } // Right d
  else if( event.keyCode == 87) { cube.position.y += speed } // Up w
  else if( event.keyCode == 83) { cube.position.y -= speed } // Down s
  else if( event.keyCode == 80) { cube.position.z -= speed } // In p
  else if( event.keyCode == 76) { cube.position.z += speed } // Out l

  if ( cube.position.x > 1 || cube.position.y >1 || cube.position.z > 1 || cube.position.x < -1 || cube.position.y < -1 ||
  cube.position.z < -1 ){
        cube.position.x = old.x;
        cube.position.y = old.y;
        cube.position.z = old.z;
      }

  if ( cube.position.x == selectedCube.position.x && cube.position.y == selectedCube.position.y && cube.position.z == selectedCube.position.z ){
    cube.material.color.setHex(0x70A9A1);
  }

  fixedCubeCoordinates.map( box => {
    if (
      box.x === cube.position.x &&
      box.y === cube.position.y &&
      box.z === cube.position.z
    ) {
      cube.position.x = old.x;
      cube.position.y = old.y;
      cube.position.z = old.z;
    }
  });
}
```
      

For hand gestures, I used gest.js, which is the most basic and helpful library if you want to implement gesture controls in your project.
```
// Using Gest.js to control hand gestures
gest.options.subscribeWithCallback( function(gesture) {
  console.log( gesture.direction );

  // Old Position of the Cube
  var old = {
    x: cube.position.x,
    y: cube.position.y,
    z: cube.position.z
  };

  // Defining Controls for the gesture
  if( gesture.direction == 'Up' ) { cube.position.y +=1;}
  else if( gesture.direction == 'Long up' ) { cube.position.z -=1;}
  else if( gesture.direction == 'Down' ) { cube.position.y -=1;}
  else if( gesture.direction == 'Long down' ) { cube.position.z +=1;}
  else if( gesture.direction == 'Left') { cube.position.x -=1;}
  else if( gesture.direction == 'Right') { cube.position.x +=1;}

  if ( cube.position.x > 1 || cube.position.y >1 || cube.position.z > 1 || cube.position.x < -1 || cube.position.y < -1 || cube.position.z < -1 ){
    cube.position.x = old.x;
    cube.position.y = old.y;
    cube.position.z = old.z;
  }

  // When the cube reaches the final posiiton, change the color of the cube.
   if ( cube.position.x == selectedCube.position.x && cube.position.y == selectedCube.position.y && cube.position.z == selectedCube.position.z ){
    cube.material.color.setHex(0x70A9A1);
  }

  fixedCubeCoordinates.map( box => {
    if (
      box.x === cube.position.x &&
      box.y === cube.position.y &&
      box.z === cube.position.z
    ) {
      cube.position.x = old.x;
      cube.position.y = old.y;
      cube.position.z = old.z;
    }
  });

  window.setTimeout(function() {
  }, 3000);
});

gest.start();
```

For speech controls, I used WebSpeechApi by MDN which is already present in our modern browsers. We just need to write a function to use them.
```
function init() {
  if (!(window.webkitSpeechRecognition) && !(window.speechRecognition)) {
    upgrade();
  } else {
    var recognizing,
    transcription = document.getElementById('speech'),
    interim_span = document.getElementById('interim');

    interim_span.style.opacity = '0.5';


    function reset() {
      recognizing = false;
      interim_span.innerHTML = '';
      transcription.innerHTML = '';
      speech.start();
    }

    var speech = new webkitSpeechRecognition() || speechRecognition();

    speech.continuous = true;
    speech.interimResults = true;
    speech.lang = 'en-US'; // check google web speech example source for more lanuages
    speech.start(); //enables recognition on default

    speech.onstart = function() {
        // When recognition begins
        recognizing = true;
    };

    speech.onresult = function(event) {
      // When recognition produces result
      var interim_transcript = '';
      var final_transcript = '';

      // main for loop for final and interim results
      for (var i = event.resultIndex; i < event.results.length; ++i) {
        if (event.results[i].isFinal) {
          final_transcript += event.results[i][0].transcript;
        } else {
          interim_transcript += event.results[i][0].transcript;
        }
      }
      transcription.innerHTML = final_transcript;
      interim_span.innerHTML = interim_transcript;

      final_transcript = final_transcript.trim();

      console.log("[final_transcript] ", final_transcript, " => ", final_transcript.length);

      if( final_transcript.length > 0 ) {
        console.log("yay, I am here");
        var old = {
          x: cube.position.x,
          y: cube.position.y,
          z: cube.position.z
        };

        if( final_transcript == 'right') { cube.position.x += 1 } // Left a
        else if( final_transcript == 'left') { cube.position.x -= 1 } // Right d
        else if( final_transcript == 'up') { cube.position.y += 1 } // Up w
        else if( final_transcript == 'down') { cube.position.y -= 1 } // Down s
        else if( final_transcript == 'back') { cube.position.z -= 1 } // In p
        else if( final_transcript == 'front') { cube.position.z += 1 } // Out l

        if ( cube.position.x > 1 || cube.position.y >1 || cube.position.z > 1 || cube.position.x < -1 || cube.position.y < -1 || cube.position.z < -1 ){
          cube.position.x = old.x;
          cube.position.y = old.y;
          cube.position.z = old.z;
        }

         if ( cube.position.x == selectedCube.position.x && cube.position.y == selectedCube.position.y && cube.position.z == selectedCube.position.z ){
          cube.material.color.setHex(0x70A9A1);
          // selectedCube.selectedCubeMaterial.color.setHex(0xffffff);
        }

        fixedCubeCoordinates.map( box => {
          if (
            box.x === cube.position.x &&
            box.y === cube.position.y &&
            box.z === cube.position.z
          ) {
            cube.position.x = old.x;
            cube.position.y = old.y;
            cube.position.z = old.z;
          }
        });
      }
    };

    speech.onerror = function(event) {
        // Either 'No-speech' or 'Network connection error'
        console.error(event.error);
    };

    speech.onend = function() {
        // When recognition ends
        reset();
    };


  }
}
```

## Built With
- [Three.js](https://threejs.org/)
- [Gest.js](https://github.com/hdmchl/gest.js)
- [Web Speech API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API)

## Credits
- [Himanshu Grover](https://github.com/himanshugrover)
- [Proksh Luthra]
      

