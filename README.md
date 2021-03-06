## DRACO AND CMAKE

#### Continuation

[Compressing 3D Model Files with Draco](https://spin.atomicobject.com/2018/09/30/compress-3d-files-draco/)

###### MY REPO in (HOW TO INSTALL DRACO / CMAKE)

[INSTALLING CMAKE](https://github.com/nadiamariduena/nm-final-three-scene)

 <br>
 <hr>
<br>

# 🌵

## SOLUTION FOR MY Draco PATH issue ../

#### I Had a hard time figuring out how to import draco to my scene, yes in three js there is a Boiler plate but they dont tell how to import it in "REACT".

- I WAS SO LUCKY to find this blog:

[Draco compression and Three.js](https://blog.jsbe.fun/2020/10/draco-compression-and-threejs.html)

Blog by:

#### Łukasz Lityński

 <br>

#### I copied and pasted the steps from the blog (just in case someone deleted it)

 <br>

- Let's [go into Three.js docs.](https://threejs.org/docs/#examples/en/loaders/DRACOLoader) They say that we can convert our normal GLTF files into Draco compressed GLTF files. In order to that we'd need to use something called [glTF-Pipeline.](https://github.com/CesiumGS/gltf-pipeline)

<br>

- So let's do this. First we need some GLTF. Then we install glTF-Pipeline.

#### Install glTF-Pipeline

```javascript
npm install --save-dev gltf-pipeline
```

#### Compress files

```javascript
npx gltf-pipeline -i scene.gltf -o compressed-scene.gltf -d
```

<br>

**Unfortunately this compression doesn't always shrink asset size**. Not sure if it depends on configuration or specific assets to compress but models with big textures are not getting smaller with Draco :( But AFAIK Draco is for compressing geometries rather than textures.

> But let's assume optimistic path that our asset did shrink. So let's go to the next path. Let's load an asset as usual, using GLTFLoader:

```javascript
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";

// ...

const loader = new GLTFLoader();

loader.load(
  "PATH_TO_OUR_GLTF",
  (d) => {
    scene.add(d.scene);
  },
  null,
  (e) => {
    console.error(e);
  }
);
```

<br>

#### And let's run.

##### result

```javascript
Error: THREE.GLTFLoader: No DRACOLoader instance provided.
    at new GLTFDracoMeshCompressionExtension (GLTFLoader.js:563)
    at GLTFLoader.parse (GLTFLoader.js:190)
    at Object.eval [as onLoad] (GLTFLoader.js:87)
    at XMLHttpRequest.eval (three.module.js:36838)
```

<br>

##### Yupi! We're almost at home. This error means that Three.js recognized that given GLTF is Draco compressed but we need to create DRACOLoader object and pass it to the GLTFLoader:

```javascript
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";

// ...

const loader = new GLTFLoader();
const dracoLoader = new DRACOLoader();
loader.setDRACOLoader(dracoLoader);

loader.load(
  "PATH_TO_OUR_GLTF",
  (d) => {
    scene.add(d.scene);
  },
  null,
  (e) => {
    console.error(e);
  }
);
```

<br>

#### Yet we missed something:

```javascript
GET http://localhost:8080/assets/create.svg 404 (Not Found)
three.module.js:36920 GET http://localhost:8080/draco_wasm_wrapper.js 404 (Not Found)
three.module.js:36920 GET http://localhost:8080/draco_decoder.wasm 404 (Not Found)
localhost/:1 Uncaught (in promise) ProgressEvent {isTrusted: true, lengthComputable: true, loaded: 160, total: 160, type: "load", …}
localhost/:1 Uncaught (in promise) ProgressEvent {isTrusted: true, lengthComputable: true, loaded: 160, total: 160, type: "load", …}
localhost/:1 Uncaught (in promise) ProgressEvent {isTrusted: true, lengthComputable: true, loaded: 160, total: 160, type: "load", …}
```

<br>

# ☁️ ☁️ ☁️ ☁️

#### And here's the weird part: it seems that you must to manually copy files from:

```javascript
// node_modules/three/examples/js/libs/draco/gltf;
```

#### into your public path on server:

- You can do this using the cp command native to Ubuntu, which is specifically **intended for copying files and directories from one location to another.**

```javascript
cp -r node_modules/three/examples/js/libs/draco/gltf HERE_YOUR_PUBLIC_PATH
//
// i added myDecoder
// cp -r node_modules/three/examples/js/libs/draco/gltf public/myDecoder
//  https://www.coderrocketfuel.com/article/copy-a-folder-to-a-new-location-via-the-terminal-on-ubuntu
```
<br>

- At one point it gets slow (I am trying to remember the steps :)

[<img src="./src/images/node_modules_three_examples_js_libs_draco_gltf_.gif"/>]()

 

<br>

##### AFTER typing that a folder containing the files is going to be created

- It should look like this:

[<img src="./src/images/decod.jpg"/>]()

<br>

#### Then you have to call

```javascript
dracoLoader.setDecoderPath("PATH_TO_DRACO_LIBRARY");
```

##### So our final code:

```javascript
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";

// ...

const loader = new GLTFLoader();
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("PATH_TO_DRACO_LIBS_ON_SERVER");
loader.setDRACOLoader(dracoLoader);
loader.load(
  "PATH_TO_OUR_GLTF",
  (d) => {
    scene.add(d.scene);
  },
  null,
  (e) => {
    console.error(e);
  }
);
```

<br>
<br>

##### MY CODE

```javascript
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
//
const loader = new GLTFLoader();
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("myDecoder/");
loader.setDRACOLoader(dracoLoader);
//
//
loader.load(
  "./models/beanstalk2_green-processed.gltf",
  (d) => {
    this.scene.add(d.scene);
  },
  null,
  (e) => {
    console.error(e);
  }
);
//
```

<br>
<br>
<br>
<hr>
<br>

# 🐖

## OBJ or PLY files

- ONCE YOU ARE DONE With the Draco and Cmake installation

- **Modify the first file** 🍦

> draco_encoder

- Will read **OBJ or PLY** files as input, and output Draco-encoded files **".drc"**

<br>

- GO TO **BLENDER** and create a geometry (a cube etc) **save it in a .ply format** inside the **draco-test folder**. (draco test is the name i choose while creating it)

<br>

- **Open your terminal** (inside the folder the "draco test folder" is, this is not the respository draco folder you downloaded from github)

<br>

- Here you will **convert a .ply file format to a drc. format**

```javascript
~/Documents/draco-test$ ./draco_encoder -i sphere.ply -o sphere.drc
```

<br>

#### result 🔴

```javascript
Encoder options:
  Compression level = 7
  Positions: Quantization = 11 bits
  Normals: Quantization = 8 bits

Encoded mesh saved to sphere.drc (1 ms to encode).

Encoded size = 2625 bytes

For better compression, increase the compression level up to '-cl 10' .
/// OPTIONS TO CHANGE THE SIZE OF THE FORMAT
// include -cl 10 in between the line
./draco_encoder -i sphere.ply -cl 10 -o sphere_1newcompress.drc


```

#### OPTIONS TO CHANGE THE SIZE OF THE FORMAT

- include -cl 10 in between the line

```javascript
./draco_encoder -i sphere.ply -cl 10 -o sphere_1newcompress.drc
// ./draco_decoder -i in.drc -o out.obj
```

<br>
<br>
<hr>
<br>
<br>

# 🍦 🍦 🍦

## GLTF AND DRACO

_**GLTF AND DRACO**_ Compression

- I ve been trying to use the GLTF Pipeline like I used to before I installed draco and the draco set up.

<br>

- Since then it doesn't seems to work in the way I want, I am going to do it manually by just using the blender draco compression.

<br>

- Here are few tests I did with a 1,2MB file (Model)
  It will serve like a Boilerplate for high res Models until i find the solution with the Pipeline.

```javascript
addCustomSceneObjects = () => {
  //

  //---------------------------
  //
  // ******** test ***** modifiers

  const loader = new GLTFLoader();
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath("myDecoder/");
  loader.setDRACOLoader(dracoLoader);
  // //
  // // WITH NO MODIFIERS AND NO TANGENTS = 1.2 MB
  // loader.load("./models/bird_paradiso_noModifiers.glb", (gltf) => {
  //   this.meshy = gltf.scene;

  //   gltf.scene.traverse((model) => {
  //     if (model.material) model.material.metalness = 0.08;

  //     model.receiveShadow = true;
  //     model.scale.set(1.5, 1.5, 1.5);
  //     // model.rotation.y = 1;
  //     // model.rotation.x += -0;
  //     // model.rotation.y += 0;
  //     //
  //     model.position.x = 0;
  //     model.position.y = 5;
  //     model.position.z = 0;
  //   });

  //   this.scene.add(gltf.scene);
  // });
  // //
  // //
  // // TANGENTS  29.4MB
  // loader.load("./models/bird_paradiso_tangents.glb", (gltf) => {
  //   this.meshy = gltf.scene;

  //   gltf.scene.traverse((model) => {
  //     if (model.material) model.material.metalness = 0.08;

  //     model.receiveShadow = true;
  //     model.scale.set(1.5, 1.5, 1.5);
  //     // model.rotation.y = 1;

  //     //
  //     model.position.x = 0;
  //     model.position.y = 0.7;
  //     model.position.z = 0;
  //   });

  //   this.scene.add(gltf.scene);
  // });
  // //
  // // WITH MODIFIERS 20.7MB
  // loader.load("./models/bird_paradiso_withmodifiers.glb", (gltf) => {
  //   this.meshy = gltf.scene;

  //   gltf.scene.traverse((model) => {
  //     if (model.material) model.material.metalness = 0.08;

  //     model.receiveShadow = true;
  //     model.scale.set(1.5, 1.5, 1.5);
  //     // model.rotation.y = 1;

  //     //
  //     model.position.x = -5;
  //     model.position.y = -5;
  //     model.position.z = 0;
  //   });

  //   this.scene.add(gltf.scene);
  // });

  //
  //
  // ---------------------------
  // COMPRESSION
  //without modifiers and tangents  192,3 Kb and WITH compression
  loader.load(
    "./models/bird_paradiso_compressed_without_mod-and-tangent.glb",
    (gltf) => {
      this.meshy = gltf.scene;

      gltf.scene.traverse((model) => {
        if (model.material) model.material.metalness = 0.08;

        model.receiveShadow = true;
        model.scale.set(0.5, 0.5, 0.5);
        // model.rotation.y = 1;

        //
        model.position.x = -2;
        model.position.y = 4;
        model.position.z = 6;
      });

      this.scene.add(gltf.scene);
    }
  );
  //
  //
  //
  // COMPRESSED
  // WITH MODIFIERS and NO tangents = 2.3Mb
  loader.load("./models/bird_compres_modi_notangetnts.glb", (gltf) => {
    this.meshy = gltf.scene;

    gltf.scene.traverse((model) => {
      if (model.material) model.material.metalness = 0.08;

      model.receiveShadow = true;
      model.scale.set(0.5, 0.5, 0.5);
      // model.rotation.y = 1;

      //
      model.position.x = 0;
      model.position.y = 2;
      model.position.z = 6;
    });

    this.scene.add(gltf.scene);
  });
  //
  //
  //

  // Compressed and with Modifiers and Tangents
  // 11.0 MB

  // loader.load(
  //   "./models/bird_paradisocompressed_WithModiAndTangents.glb",
  //   (gltf) => {
  //     this.meshy = gltf.scene;

  //     gltf.scene.traverse((model) => {
  //       if (model.material) model.material.metalness = 0.08;

  //       model.receiveShadow = true;
  //       model.scale.set(0.5, 0.5, 0.5);
  //       // model.rotation.y = 1;

  //       //
  //       model.position.x = 2;
  //       model.position.y = 3;
  //       model.position.z = 6;
  //     });

  //     this.scene.add(gltf.scene);
  //   }
  // );

  // COMPRESSED with draco pipeline (working and not working) still dont know why!!
  /*

nm-draco-obj-compressor/public/models$ gltf-pipeline -i bird_compres_modi_notangetnts.glb -o bird_compres_modi_notangetntsDraco.gltf -d7
Total: 61.186ms

-d7 is how much compression do you want , 7
*/
  // bird_compres_modi_notangetntsDraco.gltf  196.6 Kb

  loader.load(
    "./models/bird_paradisocompressed_WithModiAndTangents.glb",
    (gltf) => {
      this.meshy = gltf.scene;

      gltf.scene.traverse((model) => {
        if (model.material) model.material.metalness = 0.08;

        model.receiveShadow = true;
        model.scale.set(0.5, 0.5, 0.5);
        // model.rotation.y = 1;

        //
        model.position.x = 2;
        model.position.y = 3;
        model.position.z = 6;
      });

      this.scene.add(gltf.scene);
    }
  );

  //---------------------------
  //
  // ******** test ***** modifiers

  // First compress it in Blender
  // Then use the gltf-pipeline to compress it further (at your own risk)
  // Beanstalk-compress.glb is the name of the file i want to compress
  // You have to be in the right directory
  // myProject/public/models$   gltf-pipeline -i Beanstalk-compress.glb -d
  //  result
  // Beanstalk-processed.gltf or glb depending of your file
  //
};
```

<br>
<br>
<hr>
<br>
<br>

# 🌵

## AFTER DEFAULT BLENDER COMPRESSION

- I decided to give a last try to the gltf-pipeline compression

#### USING -d7

```javascript
//
 /nm-draco-obj-compressor/public/models$
 gltf-pipeline -i bird_compres_modi_notangetnts.glb -o bird_compres_modi_notangetntsDraco.gltf -d7
//
Total: 61.186ms
```

#### USING -d10

```javascript
/nm-draco-obj-compressor/public/models$ gltf-pipeline -i bird_compres_modi_notangetnts.glb -o bird_compres_modi_notangetntsDraco2.gltf -d10
Total: 41.537ms
```

[<img src="./src/images/draco_and_Pipeline-results.jpg"/>]()

<br>
<br>

## 🌵

# AFTER THE PIPELINE TESTS

##### I DECIDED TO MANIPULATE THE SUBDIVISION Modifier to see if there was a possibilty to recude the size, and yes in fact "there was a way"

## what i've learned so far 🐖

<br>

- **NEVER ADD SUBDIVISION as it (makes it smooth by adding more vertices which is bad in our case) to each part of a model** (yes its cool) BUT ITS painful when using it in a scene due to the cpu/gpu, I am still trying to figure that out.

[CPU vs GPU (What's the Difference?) - Computerphile](https://www.youtube.com/watch?v=_cyVDoyI6NE)

<br>
<br>

##### SO JUST APPLY 1 subdivision to the whole model and thats all!

- AlSO VERY IMPORTANT don't change this inside the (circle), if you dont know what you are doing:

[<img src="./src/images/compression_fields.jpg"/>]()

> 6 14 10 12 12 (**Blender Default**)

<br>

#### I DONT KNOW WHAT I AM DOING, but i am curious 💃

- **LOOK AT** the picture above and then read the following:

> 14 is good as if you use less than that, it will be the opposite of smooth, it will look very quadratic, of course it will pass from 658,8kb (14 quantize), to 401,4 kb (5 quantize)

#### result of changing to (5 quantize)

[<img src="./src/images/quantisiz5.jpg"/>]()

<br>
<br>
<hr>
<br>
<br>

## GOOD NEWS 🍦 🍦 🍦

#### I FOUND a way to add smooth without using Subdivision, and its " Shade Smooth" inside of the Object

<br>

- So **WHAT IS** THIS **"Shade Smooth"**?
  <br>

- Its just a soft way to add smooth **without using a Modifier** which makes our models really high res, smooth is like air.

[<img src="./src/images/bird_smooth_inside_object.jpg"/>]()

#### Result of **"Shade Smooth"**

- I Dont like the extrem quadratisch of the beak, yes it s 64,1kb WONDERFUL !! but it s ugly 🔴

```javascript
// birdblleu_ohnesubdivision_and_compression_with_shadesmooth.glb
 64,1Kb  // (extrem quadratic)
```

[<img src="./src/images/bird-smooth-shading_and-compression-No_modifier.gif"/>]()

<br>
<br>

### Smooth Options

#### Smooth Modifier 🍦

> The **Smooth modifier** smooths a mesh by **flattening the angles between adjacent faces in it**, just like the Smooth tool in Edit Mode. It smooths without subdividing the mesh, the number of vertices remains the same.

- This modifier is not limited to smoothing, though. Its control factor can be configured outside the (0.0 to 1.0) range (including negative values), which can result in interesting deformations.

[<img src="./src/images/modeling_modifiers_deform_smooth_panel.png"/>]()

[docs/Blender: Smooth Modifier](https://docs.blender.org/manual/en/latest/modeling/modifiers/deform/smooth.html)

<br>

### Result 🔴

- The result of the **smooth Modifier** is good, but since its a Modifier it will of course add it to the size of the model, so here we have **192,5 Kb**

```javascript
//birdblleu_smoothModifier_Withcompression_modifer.glb
 192,5Kb  // (a bit quadratic)
```

[<img src="./src/images/smooth-modifier_withCompression_result.gif"/>]()

<br>

- Its a Bit quadratisch but **its not that bad**, of course **my goal is to have the models as smooth as possible**, but until i find a way to reduce the size of the model, its okay.

- SO what i am going to do, is to add a modifier to the beak/bec and then add a simple **smooth shading** to the rest of the body.

### Here I added smooth Modifier + Smooth Shading to a Join Model

- I said to a "join" Model, because I tried to add a strong smooth to the beak like the Smooth Modifier and then when join to the rest of the body , apply the smooth shading **It didnt work** as somehow the whole thing get distorted.

- WHEN I SAID "join" i make reference to "MERGE" , take the gerometries and join them to one.

#### As you can see, its a mess

[<img src="./src/images/smooth-modifier_and_smooth-shading_TESTS.gif"/>]()

#### So this is what worked

- I added a Smooth Modifier ( 1622 FACTOR AND 2 of REPEAT )then i added a smooth shading to the whole "join" model, isn't perfect but can work...

<br>

- And as you can see in the image, it makes a total of **63,9Kb**

[<img src="./src/images/smooth-modifier_and_smooth-shading.jpg"/>]()

<br>
<br>

```javascript
import * as THREE from "three";
//
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
//
//----------------------------------
//         BLENDER  MODELS
//----------------------------------
//

//---------------------------
//
// ******** test ***** modifiers

const loader = new GLTFLoader();
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("myDecoder/");
loader.setDRACOLoader(dracoLoader);

// COMPRESSED
//
// 63,9 Kb

loader.load(
  "./models/bird-bleu/bird_with_smoothShading_and_with_smooth-modifier-and_with_compression_default.glb",
  (gltf) => {
    this.meshy = gltf.scene;

    gltf.scene.traverse((model) => {
      if (model.material) model.material.metalness = 0.08;

      model.receiveShadow = true;
      model.scale.set(0.5, 0.5, 0.5);
      // model.rotation.y = 1;

      //
      model.position.x = 2;
      model.position.y = 3;
      model.position.z = 6;
    });

    this.scene.add(gltf.scene);
  }
);
```

<br>
<br>

- **ANOTHER option would be to add a GUI panel**, with an option to make it smooth on the screen, this can be a choice that will give the user the possibility to add as much as smooth as they want.

<br>
<br>

## More Options 🌵

#### GUI Panel example

- The Gui Panel is the box with the blue bars, that let you manipulate the model.

[<img src="./src/images/GuiPanel-example.png"/>]()

#### ANOTHER EXAMPLE

- Here you can see how i can manipulate the vertices of someone else models

##### TRY IT

[<img src="./src/images/guipanel_example_onpage.gif"/>](https://threejs.org/examples/?q=subd#webgl_modifier_subdivision)
