<template>
  <Page>
    <ActionBar title="upload png" />
    <GridLayout columns="*" rows="*,60,60">
      <StackLayout col="0" row="0" v-if="cameraImage || patientImage">
        <Image :src="cameraImage" stretch="aspectFill" width="100%" height="100" @tap="takePicture" />
        <Image :src="patientImage" stretch="aspectFill" width="100%" height="100" @tap="takePicture" />
      </StackLayout>
      <Label v-else text="" col="0" row="0" class="fas" fontSize="30" @tap="takePicture" />
      <Image col="0" row="0" v-if="cameraImage" :src="cameraImage" stretch="none" />
      <Button text="takePicture" @tap="takePicture" col="0" row="1" />
      <Button text="uploadImage" @tap="uploadImage" col="0" row="2" />
    </GridLayout>
  </Page>
</template>

<script lang="ts">
import { HttpResponse } from "@nativescript/core/http";
import { request, HTTPFormData, HTTPFormDataEntry } from "@klippa/nativescript-http";
import { ImageSource } from "@nativescript/core";
import { Folder, path, knownFolders, File } from "@nativescript/core/file-system";
import * as camera from "@nativescript/camera";
export default {
  data() {
    return {
      cameraImage: "",
      fileName: "",
      patientImage: null,
    };
  },
  methods: {
    takePicture() {
      console.log("ImageAsset  path  cameraImage: " + this.cameraImage);
      console.log("this.fileName :>> ", this.fileName);
    },
    uploadImage() {
      const image = ImageSource.fromFileSync(this.cameraImage);
      console.log("image :>> ", image);
      const folder = knownFolders.documents();
      console.log("folder :>> ", folder);
      const filePath = path.join(folder.path, this.fileName);
      console.log("filePath :>> ", filePath);
    },
    getImage() {
      this.patientImage = this.findPatientImage(this.patientFob);
    },
    findPatientImage(personId) {},
  },
};
</script>

<style>
.fas {
  font-family: "Font Awesome 5 Free", "Font Awesome 5 Free Regular", "fa-solid-900";
  font-weight: 900;
}
</style>
