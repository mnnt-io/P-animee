# Animee Technical Documentation

This document provides detailed technical information about the Animee platform's architecture, key components, and integration points for developers working with the codebase.

## Motion Tracking System

### MediaPipe Integration

Animee uses MediaPipe's vision tasks for real-time motion capture through the browser's webcam. The system initializes three key landmarkers:

```typescript
// From landmarkers.tsx
async function InitLandmarkers() {
	const filesetResolver = await FilesetResolver.forVisionTasks(
		"https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision/wasm"
	);

	const faceLandmarker = await InitFaceLandmarker(filesetResolver);
	const poseLandmarker = await InitPoseLandmarker(filesetResolver);
	const handLandmarker = await InitHandLandmarker(filesetResolver);

	return [faceLandmarker, poseLandmarker, handLandmarker];
}
```

Each landmarker is configured with specific options:

#### Face Landmarker

```typescript
// Configuration options for face tracking
const faceLandmarkerOptions = {
	baseOptions: {
		modelAssetPath: `https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/1/face_landmarker.task`,
		delegate: "GPU",
	},
	numFaces: 1,
	runningMode: "VIDEO",
	outputFaceBlendshapes: true,
	outputFacialTransformationMatrixes: true,
};
```

#### Pose Landmarker

```typescript
// Configuration options for pose tracking
const poseLandmarkerOptions = {
	baseOptions: {
		modelAssetPath: `https://storage.googleapis.com/mediapipe-models/pose_landmarker/pose_landmarker_lite/float16/latest/pose_landmarker_lite.task`,
		delegate: "GPU",
	},
	runningMode: "VIDEO",
	numPoses: 1,
	outputSegmentationMasks: true,
};
```

#### Hand Landmarker

```typescript
// Configuration options for hand tracking
const handLandmarkerOptions = {
	baseOptions: {
		modelAssetPath: `https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/latest/hand_landmarker.task`,
	},
	runningMode: "VIDEO",
	numHands: 2,
};
```

### Detection Pipeline

The detection process runs continuously in an animation frame loop:

1. **Video Input**: Captures frames from the user's webcam
2. **Landmark Detection**: Processes frames through MediaPipe models
3. **Data Transformation**: Converts landmarks to usable rotation/position data
4. **Rig Application**: Applies transformations to the 3D avatar

```typescript
// Detection loop from page.tsx
const detectFace = () => {
	// Process video frame for face landmarks
	const faceDetection = faceLandmarker.detectForVideo(
		videoRef.current,
		performance.now()
	);

	if (
		faceDetection.faceBlendshapes &&
		faceDetection.faceBlendshapes.length > 0
	) {
		// Store blendshapes for animation
		blendshapes.current = faceDetection.faceBlendshapes[0].categories;

		// Extract rotation matrix
		const matrix = new Matrix4().fromArray(
			faceDetection.facialTransformationMatrixes![0].data
		);
		rotation.current = new Euler().setFromRotationMatrix(matrix);

		riggedFace.current = Solve(faceDetection.faceLandmarks[0]);
	}

	// Similar process for pose and hand detection
	// ...

	// Continue the detection loop
	requestAnimationFrame(detectFace);
};
```

## Avatar Rigging System

### Rotation Rigging

The system uses various rigging functions to apply detected movements to the 3D model:
(Specialized versions exist for left and right-side rigging:)

```typescript
// Basic rotation rigging function
function rigRotation(
	part: any,
	rot: any = { x: 0, y: 0, z: 0 },
	dampener = 1,
	lerpAmount = 0.3
) {
	if (!part) return;

	// Create rotation from Euler angles
	let euler = new THREE.Euler(
		rot.x * dampener,
		rot.y * dampener,
		rot.z * dampener,
		"XYZ"
	);

	// Convert to quaternion for smooth interpolation
	let quaternion = new THREE.Quaternion().setFromEuler(euler);

	// Apply smoothed rotation to the part
	part.quaternion.slerp(quaternion, lerpAmount);
}
```

### Blendshape Application

Facial expressions are applied through blendshapes:

```typescript
// Apply detected blendshapes to the model's morphTargets
useFrame(() => {
	if (blendshapes.current.length > 0) {
		blendshapes.current.forEach((element) => {
			headMesh.forEach((mesh) => {
				let index = mesh.morphTargetDictionary[element.categoryName];
				if (index >= 0) {
					mesh.morphTargetInfluences[index] = element.score;
				}
			});
		});

		// Additional head rotation
		nodes.neckx.rotation.set(
			rotation.current.x + 0.3,
			rotation.current.y,
			-rotation.current.z
		);
	}
});
```

## 3D Model System

### Model Loading

3D models are loaded using React Three Fiber's useGLTF hook:

```typescript
function BodyModel({ folder, modelName, skin, ... }) {
   // Load the 3D model
   const { scene } = useGLTF(`https://mnnt.io/collections/outkasts/models/${folder}/${modelName}.glb`);
   const nodes = useGraph(scene).nodes;

   // Process and prepare the model
   // ...

   return <primitive object={scene} />;
}
```

### Shader System

Animee uses custom GLSL shaders for stylized rendering:

```typescript
// Creating a custom shader material
let zeroMaterial = new THREE.ShaderMaterial({
	uniforms: THREE.UniformsUtils.merge([
		shader.uniforms,
		{
			uTexture: { value: texture },
			uBaseColor: { value: new THREE.Color(skinDic[skin].base) },
			uShadingColor: { value: new THREE.Color(skinDic[skin].shadow) },
		},
	]),
	vertexShader: skinVertexShader,
	fragmentShader: skinFragmentShader,
	lights: true,
});
```

The shader system supports different skin tones and material types:

```typescript
// Skin tone dictionary
const skinDic = {
	tawny_amber: {
		base: 0xa86a43,
		shadow: 0x8d5633,
	},
	red_lava: {
		base: 0xec2b11,
		shadow: 0xa13939,
	},
	nova_cyan: {
		base: 0x73e4f5,
		shadow: 0x5ebbc8,
	},
	desert_tan: {
		base: 0xb8a898,
		shadow: 0x887868,
	},
    ...
};
```

## Avatar Customization

The avatar customization system combines multiple 3D models to create a complete avatar:

```typescript
// From AvatarCustomizationScene component
<group position={startingPosition}>
	<BodyModel
		folder="Head"
		modelName={"head"}
		skin={models.skin}
		riggedPose={riggedPose}
		rotation={rotation}
		blendshapes={blendshapes}
	/>
	<BodyModel
		folder="Clothes"
		modelName={models.cloth}
		skin={models.skin}
		riggedPose={riggedPose}
		rotation={rotation}
		blendshapes={blendshapes}
	/>
	<BodyModel
		folder="Eyes"
		modelName={models.eyes}
		skin={models.skin}
		riggedPose={riggedPose}
		rotation={rotation}
		blendshapes={blendshapes}
	/>
	<BodyModel
		folder="Mouth"
		modelName={models.mouth}
		skin={models.skin}
		riggedPose={riggedPose}
		rotation={rotation}
		blendshapes={blendshapes}
	/>
	<BodyModel
		folder="Hair"
		modelName={models.hairs}
		skin={models.skin}
		riggedPose={riggedPose}
		rotation={rotation}
		blendshapes={blendshapes}
	/>
</group>
```

## NFT Integration

NFTs are integrated through the following flow:

1. User connects wallet using WalletConnect/wagmi
2. App fetches owned NFTs from collections
3. NFT metadata determines available customization options
4. 3D models are loaded based on NFT traits

## Performance Optimization

Animee implements several optimization techniques:

1. **GPU Delegation**: MediaPipe tasks run on GPU when available
2. **Texture Management**: Optimized texture loading with NearestFilter
3. **Frame Rate Control**: Careful management of animation frame requests
4. **Quaternion Interpolation**: Smooth rotations with slerp
5. **Responsive Resolution**: Adapts to device capabilities
