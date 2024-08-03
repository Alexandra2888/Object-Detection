# 📸 Object Detection App

Welcome to the Object Detection App! This project leverages **Next.js** and **TensorFlow's Coco-SSD model** to perform real-time object detection using your webcam. Detect various objects and visualize them with bounding boxes and labels.

## 🚀 Getting Started

Follow these instructions to set up and run the project locally.

### 📦 Prerequisites

Make sure you have the following installed on your system:

- Node.js
- npm

### 🛠 Installation

1. **Clone the repository:**

   ```
   git clone https://github.com/your-github-repo/object-detection-app.git
   cd object-detection-app
   ```

2. **Install the dependencies:**
 
```
npm install
```

🔧 Usage
Run the development server:

```
npm run dev
```

Open http://localhost:3000 with your browser to see the app in action.

🏗 Project Structure
Here's an overview of the project's structure:

```
object-detection-app/
├── components/
│   └── object-detection.js
├── pages/
│   └── index.js
├── styles/
│   └── globals.css
└── utils/
    └── render-predictions.js
```
📝 Code Explanation
Root Layout

```
// import { Inter } from "next/font/google";
// import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

export const metadata = {
  title: "Object Detection App",
  description: "Real-time object detection using TensorFlow and Next.js",
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

Home Page

```
// pages/index.js
import ObjectDetection from "../components/object-detection";

export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center p-8">
      <h1 className="gradient-title font-extrabold text-3xl md:text-6xl lg:text-8xl tracking-tighter md:px-6 text-center">
        Object Detection
      </h1>
      <ObjectDetection />
    </main>
  );
}
```

Object Detection Component

```
// components/object-detection.js
"use client";

import React, { useEffect, useRef, useState } from "react";
import Webcam from "react-webcam";
import { load as cocoSSDLoad } from "@tensorflow-models/coco-ssd";
import * as tf from "@tensorflow/tfjs";
import { renderPredictions } from "../utils/render-predictions";

let detectInterval;

const ObjectDetection = () => {
  const [isLoading, setIsLoading] = useState(true);

  const webcamRef = useRef(null);
  const canvasRef = useRef(null);

  async function runCoco() {
    setIsLoading(true);
    const net = await cocoSSDLoad();
    setIsLoading(false);

    detectInterval = setInterval(() => {
      runObjectDetection(net);
    }, 10);
  }

  async function runObjectDetection(net) {
    if (
      canvasRef.current &&
      webcamRef.current !== null &&
      webcamRef.current.video?.readyState === 4
    ) {
      canvasRef.current.width = webcamRef.current.video.videoWidth;
      canvasRef.current.height = webcamRef.current.video.videoHeight;

      const detectedObjects = await net.detect(
        webcamRef.current.video,
        undefined,
        0.6
      );

      const context = canvasRef.current.getContext("2d");
      renderPredictions(detectedObjects, context);
    }
  }

  useEffect(() => {
    runCoco();
  }, []);

  return (
    <div className="mt-8">
      {isLoading ? (
        <div className="gradient-text">Loading AI Model...</div>
      ) : (
        <div className="relative flex justify-center items-center gradient p-1.5 rounded-md">
          <Webcam
            ref={webcamRef}
            className="rounded-md w-full lg:h-[720px]"
            muted
          />
          <canvas
            ref={canvasRef}
            className="absolute top-0 left-0 z-99999 w-full lg:h-[720px]"
          />
        </div>
      )}
    </div>
  );
};

export default ObjectDetection;
```

Render Predictions Utility

```
// utils/render-predictions.js
import { throttle } from "lodash";

export const renderPredictions = (predictions, ctx) => {
  ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);

  const font = "16px sans-serif";
  ctx.font = font;
  ctx.textBaseline = "top";

  predictions.forEach((prediction) => {
    const [x, y, width, height] = prediction["bbox"];
    const isPerson = prediction.class === "person";

    ctx.strokeStyle = isPerson ? "#FF0000" : "#00FFFF";
    ctx.lineWidth = 4;
    ctx.strokeRect(x, y, width, height);

    ctx.fillStyle = `rgba(255, 0, 0, ${isPerson ? 0.2 : 0})`;
    ctx.fillRect(x, y, width, height);

    ctx.fillStyle = isPerson ? "#FF0000" : "#00FFFF";
    const textWidth = ctx.measureText(prediction.class).width;
    const textHeight = parseInt(font, 10);
    ctx.fillRect(x, y, textWidth + 4, textHeight + 4);

    ctx.fillStyle = "#000000";
    ctx.fillText(prediction.class, x, y);
  });
};
```

🌐 Live Demo

 See the app in action on Vercel: https://object-detection-zeta-flax.vercel.app/


🤝 Contributing

 Contributions, issues, and feature requests are welcome! Feel free to check the issues page if you want to contribute.

📄 License

This project is licensed under the MIT License.
