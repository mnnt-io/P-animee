# Animee - NFT-Powered VTuber Platform

![Andrometa Logo](/assets/images/logo.png)

Animee is a cutting-edge Next.js application that transforms NFTs into interactive VTuber avatars, enabling users to stream with personalized digital personas that respond to facial expressions, body movements, and hand gestures in real-time.

![Demo](/assets/videos/intro.mp4)

![characters](/assets/images/characters.png)
![stream](/assets/images/stream.png)
![profile](/assets/images/profile.png)
![multi](/assets/images/multi.png)

## ✨ Features

-   **NFT-Powered Avatars**: Use your NFT collections as fully-rigged VTuber avatars
-   **Real-time Motion Tracking**: Advanced facial, pose, and hand tracking using MediaPipe
-   **Customizable Appearance**: Mix and match traits from your NFT collection
-   **Streaming Integration**: Built-in screen recording capabilities for content creation
-   **Reactive 3D Models**: Avatars that respond naturally to your movements
-   **Cross-Platform Support**: Works on desktop and mobile browsers
-   **WebGL Rendering**: High-performance 3D rendering with Three.js and React Three Fiber

## 🔧 Tech Stack ![(Docs)](/Docs.md)

-   **Framework**: Next.js with TypeScript
-   **3D Rendering**: Three.js, React Three Fiber
-   **Motion Tracking**: MediaPipe (Face, Pose, and Hand Landmarkers)
-   **Animation**: GLSL Shaders
-   **Authentication**: Supabase Auth
-   **Storage**: Supabase Storage
-   **Styling**: Tailwind CSS
-   **State Management**: React Hooks
-   **Web3**: WalletConnect, wagmi

## 🚀 Getting Started

### Prerequisites

-   Node.js 16+
-   NPM or Yarn
-   Modern web browser with WebGL support
-   Webcam for motion tracking

### Installation

1. Clone the repository

```bash
git clone https://github.com/yourusername/animee.git
cd animee
```

2. Install dependencies

```bash
npm install
# or
yarn install
```

3. Create a `.env.local` file in the root directory with the following variables:

```
NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
PUSHER_APP_ID=your-pusher-app-id
PUSHER_KEY=your-pusher-key
PUSHER_SECRET=your-pusher-secret
S3_ACCESS_KEY=your-s3-access-key
S3_SECRET_KEY=your-s3-secret-key
```

4. Start the development server

```bash
npm run dev
# or
yarn dev
```

5. Access the application at [http://localhost:3000](http://localhost:3000)

## 🔍 How It Works

### Motion Tracking System

Animee uses MediaPipe's vision tasks to track facial expressions, body poses, and hand movements:

1. **Face Tracking**: Captures 468 facial landmarks and blendshapes for precise expression mapping
2. **Pose Detection**: Tracks 33 body landmarks for natural body movement
3. **Hand Tracking**: Recognizes 21 landmarks per hand for finger movements and gestures

The tracking is handled by three main components:

-   `FaceLandmarker`: Maps facial expressions to avatar blendshapes
-   `PoseLandmarker`: Translates body position to avatar skeleton
-   `HandLandmarker`: Converts hand gestures to avatar hand movements

### Avatar Customization

Users can customize their avatars by selecting different traits from their NFT collections:

-   Skin tones
-   Clothing
-   Eyes
-   Mouth
-   Hair styles
-   Accessories

Each trait is loaded as a separate 3D model and combined in the scene.

### Rendering Pipeline

1. Webcam feed is processed by MediaPipe to extract landmarks
2. Landmarks are converted to rotation and position data
3. Custom GLSL shaders apply stylized rendering
4. React Three Fiber manages the 3D scene graph
5. Three.js renders the final composite to the canvas

### Streaming Capabilities

Animee includes built-in screen recording functionality:

-   Captures both screen content and microphone audio
-   Combines streams for seamless recording
-   Supports various video formats based on browser capabilities
-   Provides download options for recorded content

### Shader Customization

Edit the shader files in `public/shaders/` to change the rendering style:

-   `vertex.glsl`: Vertex positioning
-   `fragment.glsl`: Color and lighting
