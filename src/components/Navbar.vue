<template>
    <div class="app-title">
        <b-button size="sm" class="return d-none d-md-block">Return To Field Medical Connect</b-button> 
        <b-button size="sm" class="return d-md-none d-lg-none d-xl-none">Back</b-button>
        <h1 style="margin: 0">Custom Handout Builder</h1>
        <b-button class="d-md-none" v-b-toggle.sidebar-1>Toggle Sidebar</b-button>
    </div>
</template>

<style>
.app-title {
    padding: 1rem;
    background: rgb(0,128,89);
    background: radial-gradient(circle, rgba(0,128,89,1) 21%, rgba(0,64,45,1) 100%);
    position: fixed;
    top: 0;
    width: 100%;
    z-index: 2;
}
.app-title .return {
    position: absolute;
    left: 20px;
    top: 14px;
    background: rgb(255,255,255);
    box-shadow: inset 0px 0px 10px 5px rgb(205,205,205);
    /* background: radial-gradient(circle, rgba(255,255,255,1) 0%, rgba(205,205,205,1) 40%, rgba(205,205,205,1) 100%); */
    color: black;
    text-transform: uppercase;
    font-size: 14px;
    font-weight: 700;
    border-radius: 8px;
}
h1 {
    font-size: 24px;
    color: white;
    font-weight: bold;
}
</style>