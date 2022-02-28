# Openpoint Potree  #

## Basics ##

Potree is a webgl based javascript library for vieweing pointclouds, in web browser. It uses three.js under the hood for 3d  functionalities. Since potree is not available as npm package, it is included in the head of the application html (currently added dynamically in vue app). 

### Viewer ###

Potree implements a `Potree` object which contains all the functions for pointcloud loading and manipulation. The core of potree is the `Viewer` which is available as a class in potree . Instantiate the class to get a viewer. All other functionality need the pointer to the viewer, like loading pointclouds, loading models etc.

```javascript
const potreeViewer = new Potree.Viewer(document.getElementById("element_to_render_viewer"))
```

Under the hood the viewer is the main threejs class which composes the 3d scene and renders it. Viewer can contain one or more 
Scenes, which are a 3d organisation of camera, lighting and 3d objects. It implements `THREE.Scene` class to achieve this.
For more info regarding scenes refer [Three.js Scene](https://threejs.org/docs/#manual/en/introduction/Creating-a-scene).

Potree renders the pointcloud into one scene. When loading 3d models we can have seperate scenes with different lighting.

The default scene is available at `Viewer.scene`. If you want to add new scene, create a new `THREE.Scene` and add to viewer using `setScene` function.

```javascript
let modelScene = new Potree.Scene();		
viewer.setScene(modelScene);
```

### Loading Pointclouds ###

Pointclouds can be loaded using the `Potree.loadPointCloud` function.

```javascript
Potree.loadPointCloud("pointcloudPath", function(e){
//process loaded pointcloud and add to viewer scene
let scene = viewer.scene;
let pointcloud = e.pointcloud;
			
let material = pointcloud.material;
material.size = 1;
material.pointSizeType = Potree.PointSizeType.ADAPTIVE;
material.shape = Potree.PointShape.SQUARE;			
scene.addPointCloud(pointcloud);			
viewer.fitToScreen();
})
```
The above method can load a point cloud from any open web resource. But in openpoint, due to security reasons, we have to serve the pointclouds and other resources from secured server.To achieve this , the `potree.js` and `OctreeLoader.js` has been modified to accept object rather than a simple url.  The `Storage Module` of openpoint gets the files of pointcloud and passes it as an object to loadPointcloud function. The object also includes a utility function which will fetch the secure file and return as a array buffer , which is internally used by potree to create pointcloud.

```javascript
Potree.loadPointCloud(
	{ metadata:"metadatafile_path_s3", //metadata.json
	hierarchy:"hierarchy_filepath_s3,octree", //hierarchy.bin
	octree:"octree_filepath_s3",//octree.bin
	fetchObject:signedObjectFunction},// file blob function
	(e)=>{});
```

### Loading Models ###

Potree can load all 3d model formats supported by Three.js. You can find available loaders in [Three.js Examples](https://github.com/mrdoob/three.js/tree/dev/examples).


### Creating Measurements and Annotations ###

Measurements can be created using potree class `Potree.Measure`

```javascript
{ // DISTANCE MEASURE
	let measure = new Potree.Measure();
	measure.closed = false;
	measure.addMarker(new THREE.Vector3(589803.18, 231357.35, 745.38));
	measure.addMarker(new THREE.Vector3(589795.74, 231323.42, 746.21));
	measure.addMarker(new THREE.Vector3(589822.50, 231315.90, 744.45));
	
	scene.addMeasurement(measure);
}

{ // ANGLE MEASURE
	let measure = new Potree.Measure();
	measure.name = "Angle Sample";
	measure.closed = true;
	measure.showAngles = true;
	measure.showDistances = false;
	measure.addMarker(new THREE.Vector3(589866.11, 231372.25, 737.41));
	measure.addMarker(new THREE.Vector3(589842.15, 231366.82, 743.61));
	measure.addMarker(new THREE.Vector3(589860.61, 231348.01, 740.33));
	
	scene.addMeasurement(measure);
}

{ // SINGLE POINT MEASURE
	let measure = new Potree.Measure();
	measure.name = "Canopy";
	measure.showDistances = false;
	measure.showCoordinates = true;
	measure.maxMarkers = 1;
	measure.addMarker(new THREE.Vector3(589853.73, 231300.24, 775.48));
	
	scene.addMeasurement(measure);
}

{ // HEIGHT MEASURE
	let measure = new Potree.Measure();
	measure.name = "Tree Height";
	measure.closed = false;
	measure.showDistances = false;
	measure.showHeight = true;
	measure.addMarker(new THREE.Vector3(589849.69, 231327.26, 766.32));
	measure.addMarker(new THREE.Vector3(589840.96, 231329.53, 744.52));
	
	scene.addMeasurement(measure);
}

{ // AREA MEASURE
	let measure = new Potree.Measure();
	measure.name = "Area";
	measure.closed = true;
	measure.showArea = true;
	measure.addMarker(new THREE.Vector3(589899.37, 231300.16, 750.25));
	measure.addMarker(new THREE.Vector3(589874.60, 231326.06, 743.40));
	measure.addMarker(new THREE.Vector3(589911.61, 231352.57, 743.58));
	measure.addMarker(new THREE.Vector3(589943.50, 231300.08, 754.62));
	
	scene.addMeasurement(measure);
}
```

Annotations can be added using `Potree.annotation` class

``` javascript
let annotation = new Potree.Annotation({
	position: [589850.15, 231300.10, 770.94],
	title: "My title",
	description: `My description`,
});
```