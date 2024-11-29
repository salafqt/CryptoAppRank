<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Profile</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Profile</ion-title>
        </ion-toolbar>
      </ion-header>
      <div class="profile-container">
        <ion-avatar class="orange-circle"></ion-avatar>
        <ion-list>
          <ion-item>
            <ion-label class="no-wrap">
              <h2>{{ name }}</h2>
            </ion-label>
          </ion-item>
          <ion-item>
            <ion-label class="no-wrap">
              <p>{{ email }}</p>
            </ion-label>
          </ion-item>
          <ion-item>
            <ion-label class="no-wrap">
              <p>{{ nim }}</p>
            </ion-label>
          </ion-item>
        </ion-list>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonAvatar,
  IonList,
  IonItem,
  IonLabel,
} from "@ionic/vue";

import { ref } from "vue";
const name = ref("Muhammad Salman Alfarisi");
const email = ref("salafqt@gmail.com");
const nim = ref("042246713 - UT Jember");
</script>

<style scoped>
.profile-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  margin-top: 20px;
}
ion-avatar.orange-circle {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  background-color: #ff7f50;
  margin-bottom: 10px;
}
ion-item {
  margin-bottom: 10px;
}
.no-wrap {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
