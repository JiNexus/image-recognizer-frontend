<script lang="ts">
import axios from 'axios';
import { ref } from 'vue';

declare const p5: any;
declare const ml5: any

const recursiveApi = import.meta.env.VITE_RECURSIVE_API;

export default {
    name: 'ImageRecognizerForm',
    setup() {
        let image = ref();
        let objects = ref();
        let relatedImages = ref();
        let uniqueLabels = ref();

        return {
            image,
            objects,
            relatedImages,
            uniqueLabels,
        };
    },
    data() {
        return {
            imageUrl: 'https://placehold.co/480x480',
            resizedHeight: 480,
            resizedWidth: 480,
            colors: {
                black: '#000000',
                blue: '#0000FF',
                cyan: '#00FFFF',
                green: '#00FF00',
                magenta: '#FF00FF',
                red: '#FF0000',
                white: '#FFFFFF',
                yellow: '#FFFF00',
            },
        }
    },
    mounted() { },
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
            this.relatedImages = null;
            this.uniqueLabels = new Array();
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
                    await detector.detect(img, async (error: any, result: any) => {
                        if (error) {
                            console.log(error);
                            return;
                        }

                        this.objects = result;

                        const labels = this.uniqueLabels.map((item: { label: any; }) => item.label);
                        await axios.get(recursiveApi + 'api/image-related?labels=' + labels.toString())
                        .then(response => {
                            // Handle successful response
                            this.relatedImages = response.data;

                            if (this.relatedImages?._total_items && this.relatedImages?._total_items > 0) {
                                this.relatedImages?._embedded.images.forEach(async (image: {
                                    url: string; width: number; height: number;
                                }) => {
                                    // Add width and height fields to the image object
                                    const resizeWidthAndHeight = await this.getResizeWidthAndHeight(image.url, 320, 320);
                                    image.width = resizeWidthAndHeight.width; // Example width value
                                    image.height = resizeWidthAndHeight.height; // Example height value
                                });
                            }
                        })
                        .catch(error => {
                            // Handle error
                            console.error('Error fetching data:', error);
                        });
                    });
                };

                sketch.draw = async () => {
                    // Draw bounding boxes for detected objects
                    if (await this.objects.length > 0) {
                        // Draw the image on the canvas
                        if (img) {
                            sketch.image(img, 0, 0, this.resizedWidth, this.resizedHeight);
                        }

                        // Create a Set to store unique labels
                        const uniqueLabelsSet = new Set();

                        // Loop through the data and add each label to the Set
                        this.objects.forEach((obj: { label: unknown; }) => {
                            uniqueLabelsSet.add(obj.label);
                        });

                        // Convert the Set to an array to get the unique labels
                        this.uniqueLabels = Array.from(uniqueLabelsSet);

                        const labelColors = await this.getLabelColors();
                        for (let i = 0; i < this.objects.length; i++) {
                            const object = this.objects[i];

                            const color = labelColors[object.label] as keyof typeof this.colors;
                            const rgb = this.colors[color];

                            sketch.stroke(rgb);
                            sketch.strokeWeight(4);
                            sketch.noFill();
                            sketch.rect(object.x, object.y, object.width, object.height);
                            sketch.noStroke();
                            sketch.fill(rgb);
                            sketch.textFont('Courier New');
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
            if (this.image.width >= this.image.height && this.image.width > maxWidth) {
                this.resizedWidth = maxWidth;
                this.resizedHeight = (maxWidth / this.image.width) * this.image.height;
            }

            // Resize the image if the height exceeds the maximum
            if (this.image.height >= this.image.width && this.image.height > maxHeight) {
                this.resizedHeight = maxHeight;
                this.resizedWidth = (maxHeight / this.image.height) * this.image.width;
            }

            // Set canvas dimensions to match resized image dimensions
            canvas.width = this.resizedWidth;
            canvas.height = this.resizedHeight;

            // Return resized image as data URL
            return canvas;
        },
        async getResizeWidthAndHeight(url: string, maxWidth: number, maxHeight: number): Promise<any> {
            return new Promise((resolve, reject) => {
                const img = new Image();
                img.src = url;

                img.onload = function() {
                    let resizedWidth = img.width; // Get the width of the loaded image
                    let resizedHeight = img.height; // Get the height of the loaded image

                    // Resize the image if the width exceeds the maximum
                    if (img.width >= img.height && img.width > maxWidth) {
                        resizedWidth = maxWidth;
                        resizedHeight = (maxWidth / img.width) * img.height;
                    }

                    // Resize the image if the height exceeds the maximum
                    if (img.height >= img.width && img.height > maxHeight) {
                        resizedHeight = maxHeight;
                        resizedWidth = (maxHeight / img.height) * img.width;
                    }

                    // Resolve the promise with the resized width and height
                    resolve({ width: Number(resizedWidth.toFixed(2)), height: Number(resizedHeight.toFixed(2)) });
                };

                // Handle errors
                img.onerror = function() {
                    reject(new Error('Failed to load image'));
                };
            });
        },
        async getLabelColors() {
            const labelColors: { [key: string]: string } = {};
            this.uniqueLabels.forEach((obj: any) => {
                const colorNames = Object.keys(this.colors);
                const randomColorName = colorNames[Math.floor(Math.random() * colorNames.length)] as keyof typeof this.colors;

                labelColors[obj] = randomColorName;
            });

            return labelColors;
        },
    }
}
</script>

<template>
    <div class="container text-center">
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
                    <ul v-if="uniqueLabels && uniqueLabels.length > 0" class="list-group list-group-flush">
                        <li v-for="(uniqueLabel, index) in uniqueLabels" :key="index" class="list-group-item">
                            {{ uniqueLabel.charAt(0).toUpperCase() + uniqueLabel.slice(1) }}
                        </li>
                    </ul>
                    <ul v-else class="list-group list-group-flush">
                        <li class="list-group-item">Looking for something?</li>
                    </ul>
                </div>
            </div>
        </div>
        <div class="row">
            <div class="col">
                <div class="col-lg-11 mx-auto mb-4" ref="refRelatedImages">
                    <div v-if="relatedImages?._total_items && relatedImages?._total_items > 0" class="row justify-content-center g-3">
                        <div v-for="(image, index) in relatedImages?._embedded?.images" :key="index" class="col-4">
                            <picture>
                                <div
                                    class="img-thumbnail img-thumbnail d-flex flex-wrap align-items-center justify-content-center mb-1"
                                    style="min-height: 350px; min-width: 350px;"
                                >
                                    <img :src="image.url" :alt="image.slug" :height="image.height + 'px'" :width="image.width + 'px'">
                                </div>
                            </picture>
                            <div class="alert alert-info" role="alert">
                                <p class="m-0">{{ image.label.charAt(0).toUpperCase() + image.label.slice(1) }}</p>
                            </div>
                        </div>
                    </div>
                    <div v-else>
                        <div class="alert alert-primary" role="alert">
                            No related images!
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
