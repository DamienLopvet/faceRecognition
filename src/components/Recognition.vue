<template>
    <h1>{{ emoji }}</h1>
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
        data() {
            return {
                expression: "",
                emoji: "",
            };
        },
        methods: {
            loadModules() {
                faceapi.nets.tinyFaceDetector.loadFromUri("./weights");
                faceapi.nets.faceLandmark68Net.loadFromUri("./weights");
                faceapi.nets.faceRecognitionNet.loadFromUri("./weights");
                faceapi.nets.faceExpressionNet.loadFromUri("./weights");
                faceapi.nets.ageGenderNet.loadFromUri("./weights");
            },
            startVideo() {
                navigator.getUserMedia =
                    navigator.getUserMedia ||
                    navigator.webkitGetUserMedia ||
                    navigator.mozGetUserMedia ||
                    navigator.msGetUserMedia;
                const video = document.querySelector("video");
                navigator.getUserMedia(
                    {
                        video: {},
                    },
                    (stream) => (video.srcObject = stream),
                    (err) => console.error(err)
                );
            },
            getExpression(e) {
                let sortable = [];
                for (var expression in e) {
                    sortable.push([expression, e[expression]]);
                }
                sortable.sort(function (a, b) {
                    return b[1] - a[1];
                });
                this.expression = sortable[0][0];
            },
            faceDetection() {
                const video = document.querySelector("video");
                const container = document.querySelector(".container");
                const canvas = faceapi.createCanvasFromMedia(video);
                container.append(canvas);
                const displaySize = {
                    width: video.width,
                    height: video.height,
                };
                faceapi.matchDimensions(canvas, displaySize);
                setInterval(async () => {
                    const detections = await faceapi
                        .detectAllFaces(
                            video,
                            new faceapi.TinyFaceDetectorOptions()
                        )
                        .withFaceLandmarks()
                        .withFaceExpressions()
                        .withAgeAndGender();
                    console.log(detections);
                    this.getExpression(detections[0].expressions);

                    const resizedDetections = faceapi.resizeResults(
                        detections,
                        displaySize
                    );
                    canvas
                        .getContext("2d")
                        .clearRect(0, 0, canvas.width, canvas.height);
                    faceapi.draw.drawDetections(canvas, resizedDetections);
                    faceapi.draw.drawFaceExpressions(canvas, resizedDetections);
                    resizedDetections.forEach((detection) => {
                        const box = detection.detection.box;
                        const drawBox = new faceapi.draw.DrawBox(box, {
                            label:
                                detection.gender +
                                ", environ " +
                                Math.round(detection.age) +
                                " ans ",
                        });
                        drawBox.draw(canvas);
                    });
                }, 4000);
            },
        },
        watch: {
            expression: function (val) {
                console.log(val);
                switch (val) {
                    case "angry":
                        this.emoji = "😠";
                        break;
                    case "disgusted":
                        this.emoji = "😖";
                        break;
                    case "fearful":
                        this.emoji = "😨";
                        break;
                    case "happy":
                        this.emoji = "😃";
                        break;
                    case "neutral":
                        this.emoji = "😐";
                        break;
                    case "sad":
                        this.emoji = "🥺";
                        break;
                    case "surprised":
                        this.emoji = "😮";
                        break;

                    default:
                        this.emoji = "😐";
                        break;
                }
            },
        },
        mounted() {
            this.startVideo();
        },
        beforeMount() {
            this.loadModules();
        },
    };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
    h1 {
        position: absolute;
        top: 10%;
        left: 45%;
        font-size: 100px;
    }

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
