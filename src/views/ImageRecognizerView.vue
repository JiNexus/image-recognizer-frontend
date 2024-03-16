<script lang="ts">
import ImageRecognizer from '@/components/ImageRecognizer.vue';

export default {
    name: 'ImageRecognizerView',
    components: { ImageRecognizer },
    mounted() {
        const scriptP5 = document.createElement('script');
        scriptP5.src = 'https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.1/p5.min.js';
        scriptP5.async = true;

        document.head.appendChild(scriptP5);

        const scriptMl5 = document.createElement('script');
        scriptMl5.src = 'https://unpkg.com/ml5@0.12.2/dist/ml5.min.js';
        scriptMl5.async = true;

        document.head.appendChild(scriptMl5);
    },
}
</script>

<template>
    <section>
        <div class="container my-5">
            <div class="position-relative p-3 text-center text-muted bg-body border border-dashed rounded-5">
                <div class="bi mt-3 mb-3">
                    <font-awesome-icon icon="fa-solid fa-file-import" class="fa-2xl" />
                </div>
                <h1 class="text-body-emphasis">Image Recognizer</h1>
                <p class="col-lg-8 mx-auto mb-5">
                    Utilizes ML5 - machine learning object detection model that aims to localize and identify multiple objects in a single image.
                </p>
                <ImageRecognizer />
            </div>
        </div>
    </section>
</template>
