<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Images</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content v-if="loading">
      <div class="center">
        <ion-spinner color="primary"></ion-spinner>  
      </div>  
    </ion-content>
    <ion-content v-else> 
      <div v-if="photos.length > 0">
        <ion-card v-for="(photo, index) in photos" :key="index" >
          <ion-img :src="photo" />
        </ion-card>
      </div>
      <div class="center" v-else>
        <ion-label>There are no photos to display</ion-label>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { reactive, toRefs } from "vue";
import { db, auth } from "../main";
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonSpinner,
  IonLabel,
  IonCard,
  IonImg,
} from "@ionic/vue";

export default {
  name: "Tab2",
  components: {
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonPage,
    IonSpinner,
    IonLabel,
    IonCard,
    IonImg,
  },
  setup() {
    const state = reactive({
      photos: [] as string[],
      loading: false,
    });

    const fetchPhotos = async () => {
      state.loading = true;

      const user = auth.currentUser;

      const snap = await db
        .collection("users")
        .doc(user?.uid)
        .collection("images")
        .get();

      if (!snap.empty) {
        snap.docs.forEach((doc) => {
          const data = doc.data();
          if (data.image) {
            state.photos.push(data.image);
          }
        });
      }

      state.loading = false;
    };

    fetchPhotos();
    
    return {
      ...toRefs(state)
    }
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
</style>