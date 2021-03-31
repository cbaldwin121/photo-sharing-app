<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Camera</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <div class="center">
        <ion-button size="large" fill="solid" @click="takePicture"></ion-button>
        <ion-icon :icon="camera"></ion-icon> &nbsp; Camera
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButton,
  IonIcon
} from '@ionic/vue';

import {camera} from 'ionicons/icons'

import {storage, auth, db} from '../main';
import {Plugins, CameraResultType} from '@capacitor/core';

const { Camera, Geolocation } = Plugins;




function uuidv4() {
  return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function (c) {
    const r = (Math.random() * 16) | 0,
      v = c == "x" ? r : (r & 0x3) | 0x8;
    return v.toString(16);
  });
}

export default  {
  name: 'Tab1',
  components: { IonHeader, IonToolbar, IonTitle, IonContent, IonPage, IonButton, IonIcon },
  setup(){
    const takePicture = async () => {
      const image = await Camera.getPhoto({
        quality: 90,
        allowEditing: false,
        resultType: CameraResultType.Base64
      });

      if(image?.base64String){
        const user = auth.currentUser;

        const filePath = '${user?.uid}/images/${guid}.${image.format}';

        const storageRef = storage.ref();

        await storageRef.child(filePath).putString(image.base64String, 'base64');
        const url = await storageRef.child(filePath).getDownloadURL();

        await db
          .collection("users")
          .doc(user?.uid)
          .collection("images")
          .add({
            image: url,
            //To Do
            //add text to image that you upload here
        });
      }
    };

    return {
      camera,
      takePicture
    }
  }
}
</script>

<style>
.center{
  display: flex;
  align-items: center;
  justify-content: center;
  height: 80hv;
}
</style>