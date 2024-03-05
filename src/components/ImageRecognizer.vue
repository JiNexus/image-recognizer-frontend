<script lang="ts">
import axios from 'axios';
import type { CanvasData } from '@/interfaces/CavasData';
import { ref } from 'vue';

export default {
    name: 'ImageRecognizerForm',
    setup() {
        let image = ref();

        return {
            image,
        };
    },
    data() {
        return {
            imageData: '',
        }
    },
    methods: {
        async handleFileChange(event: any) {
            // Retrieve the selected file
            const file = event.target.files[0];

            // Check if a file was selected
            if (file) {
                // Read the file as a data URL
                const reader = new FileReader();
                reader.onload = (e) => {
                    // Create a new image element
                    this.image = new Image();
                    if (typeof e.target?.result === 'string') {
                        this.image.src = e.target?.result || '';
                    }

                    // Draw the image on the canvas
                    this.image.onload = async () => {
                        const loadImageToCanvas = await this.loadImageToCanvas('canvas-input');

                        // Save the resized image data for further processing or upload
                        this.imageData = loadImageToCanvas.canvas.toDataURL();
                    };
                };

                reader.readAsDataURL(file);
            }
        },
        async loadImageToCanvas(canvasString: string): Promise<CanvasData> {
            const canvas = document.getElementById(canvasString) as HTMLCanvasElement;
            const ctx = canvas.getContext('2d');

            let resizedWidth = this.image.width;
            let resizedHeight = this.image.height;
            const maxWidth = 480;
            const maxHeight = 480;

            // Resize the image if the width exceeds the maximum
            if (this.image.width > this.image.height && this.image.width > maxWidth) {
                resizedWidth = maxWidth;
                resizedHeight = (maxWidth / this.image.width) * this.image.height;
            }

            // Resize the image if the height exceeds the maximum
            if (this.image.height > this.image.width && this.image.height > maxHeight) {
                resizedHeight = maxHeight;
                resizedWidth = (maxHeight / this.image.height) * this.image.width;
            }

            // Set canvas dimensions to match resized image dimensions
            canvas.width = resizedWidth;
            canvas.height = resizedHeight;

            // Clear previous content
            ctx?.clearRect(0, 0, canvas.width, canvas.height);

            // Draw the resized image on the canvas
            ctx?.drawImage(this.image, 0, 0, resizedWidth, resizedHeight);

            return {
                canvas: canvas,
                ctx: ctx
            };
        },
        async processImage() {
            await axios.post('http://localhost:7000/api/image-recognizer', { imageData: this.imageData })
            .then(async response => {
                console.log('Image uploaded successfully:', response);
                const loadImageToCanvas = await this.loadImageToCanvas('canvas-output');

                for (let i = 0; i < response.data.length; i++) {
                    let object = response.data[i];

                    if (loadImageToCanvas.ctx !== null) {
                        loadImageToCanvas.ctx?.beginPath();
                        loadImageToCanvas.ctx?.rect(
                            object.bbox[0],
                            object.bbox[1],
                            object.bbox[2],
                            object.bbox[3]
                        );
                        loadImageToCanvas.ctx.strokeStyle = '#00FF00';
                        loadImageToCanvas.ctx.lineWidth = 4;
                        loadImageToCanvas.ctx?.stroke();
                    }
                }
            })
            .catch(error => {
                console.error('Error uploading image:', error);
            });
        }
    }
}
</script>

<template>
    <div class="row">
        <div class="col">
            <div class="col-lg-10 mx-auto mb-4">
                <picture>
                    <div class="img-thumbnail d-flex flex-wrap align-items-center justify-content-center" style="min-height: 480px; min-width: 480px;">
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
                    <button @click="processImage" type="submit" class="btn btn-primary btn-lg px-5 mt-2 mb-2" name="btn-upload">
                        Process <font-awesome-icon icon="fa-solid fa-bolt-lightning" />
                    </button>
                </div>
            </div>
        </div>
        <div class="col">
            <div class="col-lg-10 mx-auto mb-4">
                <picture>
                    <div class="img-thumbnail d-flex flex-wrap align-items-center justify-content-center" style="min-height: 480px; min-width: 480px;">
                        <canvas id="canvas-output" ref="canvas" width="480" height="480"></canvas>
                    </div>
                </picture>
            </div>
            <div class="col-lg-10 mx-auto mb-4">
                <p class="lead">Objects found:</p>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item">An item</li>
                    <li class="list-group-item">A second item</li>
                    <li class="list-group-item">A third item</li>
                    <li class="list-group-item">A fourth item</li>
                    <li class="list-group-item">And a fifth one</li>
                </ul>
            </div>
        </div>
    </div>
</template>
