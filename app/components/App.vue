<template>
  <Page>
    <ActionBar title="upload png" />
    <GridLayout columns="*" rows="*,60,60">
      <Image v-if="photoPath" :src="photoPath" stretch="none" />
      <Button text="takePicture" @tap="takePicture" col="0" row="1" />
      <Button text="uploadImage" @tap="uploadImage" col="0" row="2" />
    </GridLayout>
  </Page>
</template>

<script lang="ts">
import { HttpResponse } from "@nativescript/core/http";
import { request, HTTPFormData, HTTPFormDataEntry } from "@klippa/nativescript-http";
import { ImageSource, fromFile, fromResource } from "@nativescript/core/image-source";
import { Folder, path, knownFolders, File } from "@nativescript/core/file-system";
import * as camera from "nativescript-camera";
export default {
  data() {
    return {
      photoPath: "",
      fileName: "",
    };
  },
  methods: {
    takePicture() {
      if (!this.isViewOnly) {
        camera.requestPermissions().then(() => {
          camera
            .takePicture({
              width: 300,
              height: 300,
              keepAspectRatio: true,
            })
            .then((imageAsset) => {
              // START CLEAN
              this.cameraImage = null;
              this.photoPath = null;
              this.fileName = null;
              ImageSource.fromAsset(imageAsset).then((imageSource) => {
                const fileName = "patient.png";
                this.fileName = fileName;
                const folderPath = knownFolders.documents().path;
                console.log("ImageAsset  folderPath: " + folderPath);
                const filePath = path.join(folderPath, fileName);
                console.log("ImageAsset  filePath: " + filePath);
                // resize image
                const resizedImage = imageSource.resize(300);
                console.log("typeof(resizedImage) :>> ", typeof resizedImage);
                const saved = resizedImage.saveToFile(filePath, "png");
                console.log("ImageAsset  saved: " + saved);

                if (saved) {
                  const loadedImage = ImageSource.fromFileSync(filePath);
                  this.cameraImage = loadedImage;
                  console.log("Saved to: " + filePath);
                  this.photoPath = filePath;
                  console.log("ImageAsset  photoPath: " + this.photoPath);
                } else {
                  console.log("not saved");
                }
              });
            });
          () => alert("permissions rejected");
        });
      }
    },
    uploadImage() {
      const image: ImageSource = <ImageSource>ImageSource.fromFileSync(this.photoPath);
      console.log("image :>> ", image);
      const folder: Folder = knownFolders.documents();
      console.log("folder :>> ", folder);
      const filePath: string = path.join(folder.path, this.fileName);
      console.log("filePath :>> ", filePath);
      const saved = image.saveToFile(filePath, "png");
      console.log("saved :>> ", saved);
      if (saved) {
        const imageFile: File = File.fromPath(filePath);
        console.log("imageFile :>> ", imageFile);
        const binarySource = imageFile.readSync((err) => {
          console.log(err);
        });
        var form = new HTTPFormData();
        console.log("form :>> ", form);
        const formFile = new HTTPFormDataEntry(binarySource, this.fileName, "image/png");
        console.log("formFile :>> ", formFile);
        form.append("file", formFile);
      }
      request({
        url: "https://api.medsbridge.com/images/upload",
        // url: "https://e24f9aa74269c9e73a0e9cc34e8ad78a.m.pipedream.net",
        method: "POST",
        headers: { APIKey: "08292020", Patient: "aab3f9c6-bcf0-48f9-93f1-9c0b67bb5a39" },
        content: form,
      }).then(
        (response: HttpResponse) => {
          // Argument (response) is HttpResponse
          console.log("response :>> ", response);
          const file: File = <File>folder.getFile(filePath);
          file
            .remove()
            .then((res) => {
              alert("image is now on server");
              // Success removing the file.
              console.log("File successfully deleted!");
            })
            .catch((err) => {
              console.log(err.stack);
            });
        },
        (e) => {}
      );
    },
  },
  mounted() {
    this.photoPath = "https://api.medsbridge.com/images/download/aab3f9c6-bcf0-48f9-93f1-9c0b67bb5a39";
  },
};
</script>

<style scoped></style>
