<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
    <main>
        <div class="container col-xxl-12 px-4 py-5">
            <header>
                <nav class="navbar navbar-expand-lg bg-body-tertiary">
                    <div class="container-fluid">
                        <a class="navbar-brand" href="#">
                            <img src="@/assets/logo.svg" alt="Bootstrap" width="30" height="24">
                        </a>
                        <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
                            data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent"
                            aria-expanded="false" aria-label="Toggle navigation">
                            <span class="navbar-toggler-icon"></span>
                        </button>
                        <div class="collapse navbar-collapse" id="navbarSupportedContent">
                            <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                                <li class="nav-item">
                                    <RouterLink to="/image-recognizer" class="nav-link">Image Recognizer</RouterLink>
                                </li>
                            </ul>
                        </div>
                    </div>
                </nav>
            </header>
            <RouterView />
        </div>
    </main>
</template>
