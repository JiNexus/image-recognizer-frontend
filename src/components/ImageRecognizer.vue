<script lang="ts">
import { ref } from 'vue';

declare const p5: any;
declare const ml5: any

export default {
    name: 'ImageRecognizerForm',
    setup() {
        let image = ref();
        let objects = ref();

        return {
            image,
            objects,
        };
    },
    data() {
        return {
            imageUrl: 'https://placehold.co/480x480',
            resizedHeight: 480,
            resizedWidth: 480,
        }
    },
    methods: {
        async clearCanvas(canvasString: string) {
            const canvas = document.getElementById(canvasString) as HTMLCanvasElement;
            const ctx = canvas.getContext('2d');

            const image = new Image();
            let width = 480;
            let height = 480;

            // Set canvas dimensions to match resized image dimensions
            canvas.width = width;
            canvas.height = height;

            // Clear previous content
            ctx?.clearRect(0, 0, canvas.width, canvas.height);

            // Draw the resized image on the canvas
            ctx?.drawImage(image, 0, 0, width, height);

            const canvasOutputWrapper = document.getElementById('canvas-output-wrapper') as HTMLElement;
            canvasOutputWrapper.innerHTML = '';
        },
        async clearObjects() {
            this.objects = new Array();
        },
        async detect() {
            new p5((sketch: any) => {
                let img: HTMLElement | null;
                let detector: any;

                sketch.preload = () => {
                    detector = ml5.objectDetector('cocossd');
                    img = sketch.loadImage(this.imageUrl);
                };

                sketch.setup = async () => {
                    sketch.createCanvas(this.resizedWidth, this.resizedHeight);

                    // Pass the canvas element to detect
                    await detector.detect(img, (error: any, result: any) => {
                        if (error) {
                            console.log(error);
                            return;
                        }

                        this.objects = result;
                    });
                };

                sketch.draw = async () => {
                    // Draw bounding boxes for detected objects
                    if (await this.objects.length > 0) {
                        // Draw the image on the canvas
                        if (img) {
                            sketch.image(img, 0, 0, this.resizedWidth, this.resizedHeight);
                        }

                        for (let i = 0; i < this.objects.length; i++) {
                            const object = this.objects[i];

                            sketch.stroke(0, 255, 0);
                            sketch.strokeWeight(4);
                            sketch.noFill();
                            sketch.rect(object.x, object.y, object.width, object.height);
                            sketch.noStroke();
                            sketch.fill(255);
                            sketch.textSize(24);
                            sketch.text(object.label, object.x + 10, object.y + 24);
                        }

                        sketch.noLoop();
                    }
                };
            }, document.getElementById('canvas-output-wrapper'));
        },
        async handleFileChange(event: any) {
            // Clear objects
            this.clearObjects();

            // Clear canvas
            this.clearCanvas('canvas-input');

            // Retrieve the selected file
            const file = event.target.files[0];

            // Check if a file was selected
            if (file) {
                // Read the file as a data URL
                const reader = new FileReader();
                reader.onload = (e) => {
                    // Create an image element
                    this.image = new Image();

                    this.image.onload = async () => {
                        // Resize the image
                        const canvas = await this.resizeCanvas('canvas-input');
                        this.loadImageToCanvas(canvas);

                        this.imageUrl = canvas.toDataURL();
                    };

                    // Set the image URL to the data URL
                    this.image.src = e.target?.result || '';
                };

                reader.readAsDataURL(file);
            }
        },
        async loadImageToCanvas(canvas: HTMLCanvasElement) {
            const ctx = canvas.getContext('2d');

            // Clear previous content
            ctx?.clearRect(0, 0, canvas.width, canvas.height);

            // Draw image on canvas with resized dimensions
            ctx?.drawImage(this.image, 0, 0, this.resizedWidth, this.resizedHeight);
        },
        async resizeCanvas(canvasString: string): Promise<HTMLCanvasElement> {
            const canvas = document.getElementById(canvasString) as HTMLCanvasElement;

            this.resizedWidth = this.image.width;
            this.resizedHeight = this.image.height;
            const maxWidth = 480;
            const maxHeight = 480;

            // Resize the image if the width exceeds the maximum
            if (this.image.width > this.image.height && this.image.width > maxWidth) {
                this.resizedWidth = maxWidth;
                this.resizedHeight = (maxWidth / this.image.width) * this.image.height;
            }

            // Resize the image if the height exceeds the maximum
            if (this.image.height > this.image.width && this.image.height > maxHeight) {
                this.resizedHeight = maxHeight;
                this.resizedWidth = (maxHeight / this.image.height) * this.image.width;
            }

            // Set canvas dimensions to match resized image dimensions
            canvas.width = this.resizedWidth;
            canvas.height = this.resizedHeight;

            // Return resized image as data URL
            return canvas;
        },
    }
}
</script>

<template>
    <div class="row">
        <div class="col">
            <div class="col-lg-10 mx-auto mb-4">
                <picture>
                    <div id="canvas-input-wrapper" class="img-thumbnail d-flex flex-wrap align-items-center justify-content-center" style="min-height: 480px; min-width: 480px;">
                        <canvas id="canvas-input" ref="canvas" width="480" height="480"></canvas>
                    </div>
                </picture>
            </div>
            <div class="col-lg-10 mx-auto mb-2">
                <label for="image-file" class="form-label visually-hidden">Image File</label>
                <input @change="handleFileChange" class="form-control form-control-lg" id="image-file" type="file" accept="image/*" />
            </div>
            <div class="col-lg-10 mx-auto mb-2">
                <div class="d-grid gap-2">
                    <button @click="detect" type="submit" class="btn btn-primary btn-lg px-5 mt-2 mb-2" name="btn-upload">
                        Detect <font-awesome-icon icon="fa-solid fa-eye" />
                    </button>
                </div>
            </div>
        </div>
        <div class="col">
            <div class="col-lg-10 mx-auto mb-4">
                <picture>
                    <div id="canvas-output-wrapper" class="img-thumbnail d-flex flex-wrap align-items-center justify-content-center" style="min-height: 480px; min-width: 480px;"></div>
                </picture>
            </div>
            <div class="col-lg-10 mx-auto mb-4">
                <p class="lead">Objects found:</p>
                <ul v-if="objects && objects.length > 0" class="list-group list-group-flush">
                    <li v-for="(object, index) in objects" :key="index" class="list-group-item">
                        {{ object.label.charAt(0).toUpperCase() + object.label.slice(1) }}
                    </li>
                </ul>
                <ul v-else class="list-group list-group-flush">
                    <li class="list-group-item">Looking for something?</li>
                </ul>
            </div>
        </div>
    </div>
</template>
