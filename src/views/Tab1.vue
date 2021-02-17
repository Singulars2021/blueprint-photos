<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Camera</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <div class="center">
        <ion-button size="large" @click="takePicture"> Camera </ion-button>
        <ion-button size="large" @click="uploadPhotos">
          Upload Photos!
        </ion-button>
        <p v-if="error">{{ error }}</p>
      </div>
      <div class="preview" v-if="images.length > 0">
        <img v-for="image in images" :key="image.id" :src="image.preview" />
      </div>
      <div v-else>Upload photos to preview here</div>
    </ion-content>
  </ion-page>
</template>

<script>
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButton,
} from "@ionic/vue";

// import { camera } from "ionicons/icons";

import { storage, auth, db } from "../main";
import { Plugins, CameraResultType } from "@capacitor/core";

const { Camera } = Plugins;

function uuidv4() {
  return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function (c) {
    const r = (Math.random() * 16) | 0,
      v = c == "x" ? r : (r & 0x3) | 0x8;
    return v.toString(16);
  });
}

export default {
  name: "Tab1",
  components: {
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonPage,
    IonButton,
  },
  data() {
    return {
      images: [],
      error: null,
    };
  },
  methods: {
    async takePicture() {
      if (this.images.length < 5) {
        const image = await Camera.getPhoto({
          quality: 60,
          resultType: CameraResultType.Base64,
        });

        this.images.push({
          ...image,
          id: new Date().toISOString,
          preview: "data:image/jpeg;base64," + image.base64String,
        });
      } else {
        this.error = "You can only upload 5 pictures!";
      }
    },

    async uploadPhotos() {
      for (let i = 0; i < this.images.length; i++) {
        await this.uploadPhoto(i);
      }
    },
    async uploadPhoto(index) {
      const user = auth.currentUser;
      const guid = uuidv4();
      const filePath = `${user?.uid}/images/${guid}.${this.images[index].format}`;
      const storageRef = storage.ref();

      await storageRef
        .child(filePath)
        .putString(this.images[index].base64String, "base64");
      const url = await storageRef.child(filePath).getDownloadURL();

      await db.collection("users").doc(user?.uid).collection("images").add({
        image: url,
      });
    },
  },
};
</script>

<style>
.center {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 80vh;
}

.preview {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 2rem;
}

img {
  margin: 1rem;
}
</style>