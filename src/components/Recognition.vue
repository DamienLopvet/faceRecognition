<template>
    <div class="container">
        <video
            src=""
            id="video"
            width="720"
            height="560"
            autoplay
            @play="faceDetection"
        ></video>
    </div>
</template>

<script>
    import * as faceapi from "face-api.js";
    export default {
        name: "HelloWorld",

        methods: {
            loadModules() {
                faceapi.nets.tinyFaceDetector.loadFromUri("./weights");
                faceapi.nets.faceLandmark68Net.loadFromUri("./weights");
                faceapi.nets.faceRecognitionNet.loadFromUri("./weights");
                faceapi.nets.faceExpressionNet.loadFromUri("./weights");
                
            },
            startVideo() {
              navigator.getUserMedia = ( navigator.getUserMedia ||
                       navigator.webkitGetUserMedia ||
                       navigator.mozGetUserMedia ||
                       navigator.msGetUserMedia);
                const video = document.querySelector("video");
                navigator.getUserMedia(
                    {
                        video: {},
                    },
                    (stream) => (video.srcObject = stream),
                    (err) => console.error(err)
                );
            },
            faceDetection() {
                const video = document.querySelector("video");
                const container = document.querySelector(".container")
                const canvas = faceapi.createCanvasFromMedia(video)
                console.log(canvas);
                container.append(canvas)
                const displaySize = {width: video.width,height:video.height}
                faceapi.matchDimensions(canvas, displaySize)
                setInterval(async () => {
                    const detections =await faceapi
                        .detectAllFaces(
                            video,
                            new faceapi.TinyFaceDetectorOptions()
                        )
                        .withFaceLandmarks()
                        .withFaceExpressions();
                        const resizedDetections = faceapi.resizeResults(detections, displaySize);
                        canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height)
                    faceapi.draw.drawDetections(canvas, resizedDetections)
                    faceapi.draw.drawFaceLandmarks(canvas, resizedDetections)
                    faceapi.draw.drawFaceExpressions(canvas, resizedDetections)
                  
                }, 1000);
            },
        },
        mounted() {
          this.startVideo();
        },
        beforeMount(){
          this.loadModules();
        }
    };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
    .container {
        margin: 0;
        padding: 0;
        width: 100vw;
        height: 100vh;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    canvas {
        position: absolute;
    }
</style>
